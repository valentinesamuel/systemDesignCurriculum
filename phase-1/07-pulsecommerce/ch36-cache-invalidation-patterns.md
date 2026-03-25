---
**Title**: PulseCommerce — Chapter 36: Cache Invalidation Hell
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #caching #cache-invalidation #redis #consistency #distributed
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 4, Ch. 22, Ch. 35, Ch. 38
**Exercise Type**: System Design
---

### Story Context

**Merchant support ticket escalation — Nnamdi forwards to you, Monday**

```
From: Support Team → Nnamdi Okafor (escalated)
Subject: URGENT — Merchant stale pricing issue (3 merchants, 12-hour window)

Three merchants have escalated an issue affecting real customer orders:

1. SportZone: Updated a product price from $89.99 to $79.99 at 9:00 AM.
   Customers saw $89.99 in search results and cart until 9:48 PM — 12 hours.
   23 customers paid $89.99; 11 requested refunds citing the $79.99 search price.
   SportZone is requesting compensation.

2. TechAccessories: Marked 340 products as out-of-stock at 2:00 PM.
   Products continued showing "In Stock" in search results until 9:00 PM.
   18 orders were placed for out-of-stock items. All had to be cancelled.

3. HomeDecor: Updated product description to remove a claim that turned out
   to be inaccurate (material specification). The old description remained
   in search results for 6+ hours. Merchant received a customer complaint.

All three merchants are asking what our cache TTL is and why it's 12 hours.
```

---

**Root cause investigation — Monday afternoon**

You trace through the caching layers:

```
Cache architecture (current state):

Layer 1: Elasticsearch — product search index
  - Products indexed when created. Never updated unless reindex job runs.
  - Reindex job: runs nightly at 2:00 AM. That's the 12-hour window.
  - Inventory sync from Postgres → ES: daily. NOT real-time.

Layer 2: Redis cache — product API responses
  - Cache key: product:{id}
  - TTL: 24 hours (set once, never invalidated)
  - Cache set on first request. Never cleared on update.

Layer 3: CDN — product listing pages (server-side rendered)
  - TTL: 1 hour
  - No cache invalidation mechanism

Layer 4: Local memory cache — in API server
  - product metadata cached for 5 minutes in Node.js memory
  - No invalidation possible (multiple servers, no shared state)

Result: A price update at 9:00 AM:
  - Hits Postgres immediately
  - Redis cache: stale for up to 24 hours
  - Elasticsearch: stale until 2:00 AM next day
  - CDN: stale for up to 1 hour
  - Local memory: stale for up to 5 minutes

Worst case: 24 hours of stale data (Redis).
```

---

**Team discussion — Tuesday 10:00 AM**

**Priya Menon**: The naive fix is to just reduce all TTLs. Set Redis to 60 seconds,
Elasticsearch sync to 60 seconds, CDN to 60 seconds.

**You**: That would work for correctness. But reducing cache TTLs increases cache
miss rate. More misses means more DB queries. We calculated that at 2% cache miss
rate with current traffic, we're handling about 160 DB queries/second. If TTL goes
to 60 seconds and miss rate jumps to 25%, that's 2,000 DB queries/second. We'd need
to over-provision the DB significantly.

**Priya**: So we can't just reduce TTLs.

**You**: We need event-driven cache invalidation. When a product is updated in
Postgres, invalidate the cache entries immediately — not by waiting for TTL expiry.

**Priya**: But we have 4 cache layers. Do we invalidate all 4?

**You**: Different layers need different strategies. Local memory is easy — short TTL
is fine because it's small scope. Redis — invalidate on write. ES — sync the specific
product within 60 seconds. CDN — this is the hard one. CDN invalidation is expensive
and slow.

---

**Slack DM — Marcus Webb → You, Tuesday afternoon**

**Marcus Webb**
Cache invalidation. Famous problem. Phil Karlton's quote and all that.

The specific challenge you're describing: you have caches at multiple layers, each
with different invalidation mechanisms and costs. You can't treat them all the same.

Here's the framework: for each cache layer, answer two questions:
1. What is the cost of a stale read? (Customer-facing? Back-office? Billing-critical?)
2. What is the invalidation mechanism? (TTL, event-driven, manual, or none?)

Price staleness is the highest risk — it causes financial disputes. Inventory
staleness causes cancelled orders. Description staleness causes complaints.
Different costs → different urgency of invalidation.

And the CDN problem: CDN invalidation via CloudFront takes 5-30 seconds and costs
money ($0.005 per 1,000 invalidation paths). At 100k product updates/day, that's
$0.50/day — trivial. But you can't invalidate per-product-per-merchant URL efficiently
because the URL space is enormous (150M products × N pages per product).

What's the CDN strategy for content that can change without a new URL?

---

### Problem Statement

PulseCommerce has four cache layers with no event-driven invalidation, resulting
in stale product data (price, inventory, description) persisting for up to 24 hours
after updates. Three merchant incidents have caused customer-facing financial disputes
and cancelled orders. You must design an event-driven cache invalidation architecture
that propagates updates through all four cache layers within defined SLAs.

### Explicit Requirements

1. Price updates must be reflected in search results and cart within 60 seconds
2. Inventory status updates must be reflected within 15 seconds (flash sales)
3. Description updates may take up to 5 minutes to propagate
4. All four cache layers (ES, Redis, CDN, local memory) must have defined invalidation strategies
5. Cache miss rate must not increase by more than 5% after implementing event-driven invalidation
   (preserving DB headroom)
6. Invalidation events must be durable — a restart of the invalidation processor
   must not permanently skip any pending invalidations

### Hidden Requirements

- **Hint**: Marcus Webb raised the CDN invalidation cost for "per-product URLs."
  The CDN caches server-rendered pages at paths like `/merchant/sportzone/products/running-shoes`.
  If you can't efficiently invalidate per-product CDN cache, what's an alternative?
  (Hint: ETag headers, stale-while-revalidate, or... don't render the price
  server-side. Render it client-side from the API instead.)
- **Hint**: "Invalidation events must be durable." If the Redis invalidation worker
  crashes after receiving a product update event but before invalidating the cache,
  the cache entry remains stale permanently (until TTL). How do you ensure that
  a crashed worker's pending invalidations are processed after restart?
  (Hint: Kafka consumer with at-least-once delivery + idempotent cache delete)
- **Hint**: "Cache miss rate must not increase by more than 5%." Removing a cached
  entry doesn't mean the next request will miss — it means the next request fetches
  fresh data and re-populates the cache. If price updates happen 100,000 times/day
  and each invalidation causes 1 cache miss (the request immediately after invalidation),
  what is the miss rate impact? Show the math.

### Constraints

- **Price update frequency**: ~100,000/day (including flash sale bulk updates)
- **Inventory update frequency**: ~500,000/day (high during sales)
- **Description update frequency**: ~10,000/day
- **Current Redis TTL**: 24 hours (to be replaced with event-driven)
- **CDN invalidation cost**: $0.005 per 1,000 paths; max 3,000 invalidations/hour before rate limiting
- **ES sync latency**: Debezium CDC from Postgres → Kafka → ES (already available from Ch. 33)
- **SLAs**: Price: 60s; Inventory: 15s; Description: 5min

### Your Task

Design the event-driven cache invalidation architecture for PulseCommerce across
all four cache layers.

### Deliverables

- [ ] **Cache invalidation architecture diagram** (Mermaid) — product update in Postgres
  → CDC event → invalidation processor → per-layer invalidation (Redis, ES, CDN, local)
- [ ] **Per-layer invalidation strategy table** — for each layer: current approach, proposed
  approach, invalidation trigger, latency to fresh data, durability mechanism
- [ ] **CDN strategy design** — since per-product CDN path invalidation is expensive
  and rate-limited, what alternative approaches make prices and inventory always fresh
  on the CDN-cached page?
- [ ] **Kafka-based durable invalidation** — show how the invalidation processor reads
  product update events from Kafka, performs cache invalidations, and handles restart
  without re-processing or skipping events
- [ ] **Cache miss rate impact analysis** — at 600,000 total updates/day across all
  update types, what is the expected increase in daily cache misses? What is the
  corresponding increase in DB queries/second?
- [ ] **Tradeoff analysis** — minimum 3 tradeoffs:
  1. TTL-based expiry (simple) vs event-driven invalidation (correct but complex)
  2. Cache-aside with short TTL vs write-through with immediate invalidation
  3. Server-side rendered prices (CDN-cacheable but potentially stale) vs
     client-side price fetching (always fresh but extra API call)

### Diagram Format

```mermaid
graph LR
  PG[(Postgres)] -->|CDC| KF[Kafka\nproduct-updates]
  KF --> IP[Invalidation Processor]
  IP -->|DEL product:{id}| RD[(Redis)]
  IP -->|update document| ES[Elasticsearch]
  IP -->|POST /invalidation| CDN[CDN API]
  IP -->|broadcast| LM[Local Memory\nno-op: wait for TTL]
  PG -->|write-through| RD
```
