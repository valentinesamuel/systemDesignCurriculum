---
**Title**: Pattern Summary 3 — Caching, Consistency & Cache Invalidation
**Level**: Reference
**Difficulty**: N/A
**Tags**: #pattern-summary #caching #consistency #cache-invalidation #redis
**Estimated Time**: 1 hour (review + reflection)
**Related Chapters**: Ch. 27–37
**Exercise Type**: Pattern Summary
---

> No deliverables. This is a reflection chapter.

---

### How This Arrived

**Slack DM — Marcus Webb → You**

**Marcus Webb**
You've now designed or debugged caching systems at NovaPay (write-through balance
cache), Beacon Media (CDN + recommendation cache), CloudStack (not directly, but
the billing aggregate is a form of cache), and PulseCommerce (catalog cache with
invalidation hell).

Notice how every caching conversation ends up at the same question: "How stale is
too stale for this specific piece of data?"

The answer changes the entire design. A merchant balance that's 3 seconds stale is
a financial liability. A product description that's 3 seconds stale is invisible
to any user. And yet, both are "cached data."

I'm sending you a framework I've been refining for 15 years.

[Attachment: caching-consistency-patterns.md]

---

### Patterns Covered: Ch. 27–37 (plus retrospective from Ch. 4)

---

#### Pattern 17: The Staleness Tolerance Spectrum

**The fundamental question before any cache design:**
> "What is the maximum acceptable age of this data before a stale read causes a measurable business harm?"

| Data Type | Max Staleness | Business Impact of Stale Read |
|-----------|--------------|------------------------------|
| Payment authorization status | 0s (no cache) | Double charge or fraud |
| Merchant balance (payout decision) | 0s (no cache) | Incorrect payout |
| Product price (checkout) | 60s | Customer dispute, refund demand |
| Inventory (checkout) | 15s | Oversell, cancelled orders |
| Product description | 5 min | Customer complaint, rarely actionable |
| Search relevance scores | 24h | Slightly worse results, invisible |
| CDN-cached static images | 30 days | Zero impact |
| Recommendation results (inactive users) | 24h | Slightly less personalized |

**The rule**: set your cache TTL to the maximum acceptable staleness for that data type.
Never cache "until we need to change it."

---

#### Pattern 18: Write-Through vs Write-Behind vs Cache-Aside

**Three write strategies, different guarantees:**

**Write-Through** (Ch. 4, NovaPay):
- Write to DB and cache simultaneously
- Cache is always in sync with DB (if both writes succeed)
- Failure case: if cache write fails, cache is stale until next write or TTL
- Best for: data that must be fresh after writes (balance, inventory)

**Write-Behind** (theoretical — appeared as a tradeoff):
- Write to cache immediately; DB write is async
- Very low write latency; risk of data loss if cache fails before DB write
- Best for: high-volume, low-criticality writes (activity tracking, click counts)
- Worst for: financial data, anything that must survive a cache failure

**Cache-Aside** (lazy cache):
- Read: check cache → miss → query DB → populate cache → return
- Write: write to DB → invalidate cache (or let TTL expire)
- Simple; cache is populated by demand
- Weakness: cold start; thundering herd on cache miss
- Best for: read-heavy data with occasional writes

---

#### Pattern 19: Event-Driven Cache Invalidation

**What it is**: Instead of waiting for TTL expiry, invalidate cache entries immediately
when the underlying data changes.

**Full pattern**:
1. Application writes to DB
2. CDC event (Debezium) or application-level event emitted
3. Cache invalidation processor subscribes to events
4. Processor deletes/updates relevant cache keys
5. Next read: cache miss → fresh DB read → repopulate cache

**Where it appeared**:
- Ch. 36 (PulseCommerce): Price and inventory changes not propagating to 4 cache layers

**Key concerns**:
- **Durability**: The invalidation event must be durable. If the processor crashes,
  events must not be lost. Use Kafka with at-least-once delivery.
- **Idempotency**: Processing the same invalidation event twice must be safe
  (deleting an already-deleted cache key is idempotent).
- **Fan-out**: One product update may need to invalidate many cache keys
  (product page, search index, cart cached copy, CDN page).

---

#### Pattern 20: The Thundering Herd

**What it is**: When a cached item expires or is invalidated, many concurrent requests
simultaneously discover the cache miss and all query the database simultaneously —
causing a spike.

**Where it appeared**:
- Ch. 22 (Beacon Media): 6.8M users hit cold recommendation cache simultaneously after
  Champions League match ended
- Ch. 36 (PulseCommerce): Mass invalidation of product cache causing DB spike

**Solutions**:
1. **Probabilistic early expiration**: Begin refreshing a cache entry before it expires
   (e.g., when 80% of TTL has elapsed, a small probability triggers early refresh)
2. **Mutex/single-flight**: When a cache miss occurs, only one request goes to DB;
   all other concurrent requests for the same key wait for the first request to complete
3. **Pre-warming**: Pre-populate cache before an expected traffic event (live event ending)
4. **Stale-while-revalidate**: Return stale data immediately, refresh in background

---

#### Pattern 21: Cache Invalidation Granularity

**The spectrum of invalidation strategies:**

| Strategy | Granularity | Cost | Staleness Risk |
|----------|------------|------|----------------|
| Full cache flush | Everything | Very high (rebuilds from scratch) | Low after flush |
| Tag-based invalidation | All keys with tag X | Medium | Medium |
| Key-based invalidation | Specific key | Low | Low |
| TTL expiry only | None (passive) | Zero | High (= TTL) |
| Field-level invalidation | Specific field within a document | Very low | Very low |

**Where it appeared**:
- Ch. 22 (Beacon Media): Model update required invalidating all 28M user recommendation
  caches simultaneously — threatening thundering herd. Solution: tag-based + staggered
- Ch. 36 (PulseCommerce): Product updates needed key-level invalidation per product

---

#### Pattern 22: CDN Cache and the "Stale Content" Problem

**Specific challenge**: CDN nodes are distributed globally. Invalidating a CDN cache
entry requires calling the CDN invalidation API, which propagates to all edge nodes.
This takes 5-30 seconds and may be rate-limited.

**Solutions when per-URL invalidation is infeasible**:
1. **Content hash in URL**: `/static/product-image-abc123def456.jpg` — change the hash
   on update, old URL expires naturally, new URL is immediately fresh
2. **Short CDN TTL for dynamic content**: 60 seconds is "cached" but never dangerously stale
3. **Edge-side includes (ESI)**: Render the static frame via CDN; include dynamic data
   (price, inventory) via ESI that bypasses cache
4. **Client-side fetching**: Don't render dynamic data in server-side templates at all;
   fetch price/inventory from API in the browser (always fresh, one extra round-trip)

**Where it appeared**:
- Ch. 18 (Beacon Media): CDN hit rate 54% — wrong cache keys from session parameters
- Ch. 36 (PulseCommerce): CDN-cached product pages showing stale prices

---

### Cross-Industry Observations

Caching appears everywhere because reads vastly outnumber writes in almost every
system. The challenge is not caching — it's the guarantee you make about how fresh
the cache is.

| Incident | Root Cause | Correct Pattern |
|----------|-----------|----------------|
| NovaPay balance errors | No write-through consistency | Write-through with fallback |
| Beacon Media recommendation meltdown | No thundering herd protection | Pre-warm + stale-while-revalidate |
| PulseCommerce stale prices (12 hours) | TTL-only invalidation | Event-driven invalidation |
| PulseCommerce inventory oversell | No cache at all for the reservation step | Don't cache what must be consistent |

---

### Reflection Questions

No answers are provided.

1. Cache-aside is the simplest pattern, and it's used widely. But "simple" comes
   with the thundering herd risk. At what scale does cache-aside become unsafe, and
   what's the trigger to upgrade to single-flight or pre-warming? Is there a traffic
   level, a data sensitivity level, or a business risk level that should cause the switch?

2. PulseCommerce's inventory was not cached (the cache invalidity in Ch. 36 was about
   product descriptions and prices, not inventory). But the flash sale in Ch. 37 was
   also a consistency problem — just at the database write level, not the cache level.
   Is there a meaningful difference between "cache consistency" and "database write
   consistency" from a system design perspective? Where is the conceptual boundary?

3. You've seen three companies get burned by "overly long TTLs" — data cached for longer
   than the business can accept being stale. Yet reducing TTLs increases DB load.
   If you were starting a new company today and could only set one rule about caching,
   what would it be?
