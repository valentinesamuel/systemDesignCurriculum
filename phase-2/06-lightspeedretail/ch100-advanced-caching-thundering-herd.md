---
**Title**: LightspeedRetail — Chapter 100: Advanced Caching — Black Friday Thundering Herd
**Level**: Staff
**Difficulty**: 9
**Tags**: #caching #thundering-herd #redis #black-friday #distributed-lock #cache-stampede
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 82 (TradeSpark cache stampede), Ch. 37 (PulseCommerce thundering herd), Ch. 99, Ch. 101
**Exercise Type**: Performance Optimization / System Design
---

### Story Context

**#load-testing** — Slack, Tuesday October 8, 09:47

**[09:47] Marcus Delgado** (SRE Lead): Black Friday pre-mortem load test complete. Results are bad. Tagging @[you] and @Priya Nair.

**[09:48] Marcus Delgado**: Summary: we simulated 20x normal peak (280k TPS). Product catalog cache survived until 09:00 in the test scenario — that's when we simulated "market open" for European terminals. At 09:01, cache miss rate went from 0.3% to 98.7% in under 90 seconds.

**[09:51] [you]**: 98.7%? That's not degraded performance, that's total collapse.

**[09:53] Marcus Delgado**: Correct. Postgres received 47,000 concurrent product catalog queries in the first 60 seconds. Max connections is 500. The connection pool exhausted in 8 seconds. After that, terminals started returning HTTP 503. In the simulation that means 120,000 terminals showing "System Unavailable" to cashiers.

**[09:55] Priya Nair**: Why does this happen at 09:00 specifically?

**[09:57] Marcus Delgado**: Every terminal does a daily product sync at startup. UK terminals boot at 08:45 for 09:00 open. France boots at 08:30 for their 09:00. Germany at 08:55. They all hit cache simultaneously. The cache TTL is 24 hours. Last night's cache was set at 09:03 yesterday. They all expire within 3 minutes of each other.

**[10:01] [you]**: Thundering herd. Every terminal sees an expired cache entry, tries to be the one to repopulate it, hammers the DB. Classic.

**[10:03] Marcus Delgado**: Right. And because the product catalog is 4.2GB for our full SKU set, the DB query to rebuild it takes 8-12 seconds. 47,000 concurrent executions of an 8-12 second query on a 500-connection pool.

**[10:07] Beatriz Santos**: @[you] — Black Friday is in 40 days. I need a working solution before the next load test in 2 weeks. What do we have?

---

**Slack DM — [you] → Marcus Webb** — Tuesday October 8, 10:22

**[you]**: Pre-mortem load test failed. 120k terminals all request product cache at market open. Cache expiry clustering is the root cause. Need to solve thundering herd at this scale.

**Marcus Webb**: How big is the cached object?

**[you]**: 4.2GB full SKU set. But terminals only need their store's catalog — maybe 2,000-8,000 SKUs each. We partition by store.

**Marcus Webb**: Okay, that's different. Per-store cache or global cache?

**[you]**: Currently global, partitioned by region in Redis. 12 regional keyspaces.

**Marcus Webb**: Bad design. 12 keyspaces means 12 thundering herds, not one. What's your TTL strategy?

**[you]**: Fixed 24h TTL set at write time.

**Marcus Webb**: There's your problem. Fixed TTL on high-read, low-write data means all your readers expire together. You know about XFetch?

**[you]**: Probabilistic early expiration. I used it at TradeSpark but the data was financial, not retail SKUs.

**Marcus Webb**: Same algorithm, bigger data. The key insight: don't wait for expiry to recompute. Compute early, probabilistically, before the TTL hits. Early recomputation by one worker prevents the stampede. But at 120k terminals you also need a distributed lock so only ONE recomputation happens per store catalog, not one per terminal.

**Marcus Webb**: Redlock or SETNX. Don't overthink it. If you use SETNX, put a TTL on the lock itself — you don't want a dead worker to hold the lock forever during a Black Friday incident.

**[you]**: What about pre-warming before market open?

**Marcus Webb**: Yes. Scheduled job at 05:00 UTC, before any terminal boots. Warm the cache for every store that opens in the next 4 hours. Stagger by region. But your pre-warming job itself can cause a stampede if it's not careful. What's the query pattern?

**[you]**: Good point. I'll use a queue-based pre-warmer, one store at a time with bounded parallelism.

**Marcus Webb**: Now you're thinking. One more thing: circuit breaker for stale data. If the DB is unavailable during cache rebuild, serve stale rather than 503. A terminal with yesterday's product catalog is infinitely better than a terminal with no catalog at all.

---

**#load-testing** — Slack, October 8, 14:31

**[14:31] Marcus Delgado**: Follow-up data for @[you]. Redis Cluster config: 6 nodes, 3 primary / 3 replica. Each store's catalog key averages 180KB after gzip. 120k stores × 180KB = 21.6GB total catalog data in cache. We're currently at 87% memory utilization.

**[14:34] [you]**: So we also have a memory pressure problem. 87% is too high — when Redis starts evicting keys, it'll trigger more stampedes.

**[14:36] Marcus Delgado**: Correct. And the eviction policy is currently `allkeys-lru`. Under memory pressure, it evicts the least recently used — which might be store catalogs for stores that haven't opened yet today. When they open: cache miss for all their terminals.

**[14:39] [you]**: We need to rethink the entire caching strategy, not just the TTL.

---

This is a direct evolution from Chapter 82 (TradeSpark), where you first encountered XFetch on a financial data feed. At TradeSpark the data was small and the stakes were high-frequency trading latency. Here the data is large (180KB per store), the stakes are $8.7M revenue per hour, and the thundering herd vector is not expiry alone but the correlated boot behavior of 120,000 physical devices with predictable market-open timing.

### Problem Statement

LightspeedRetail's product catalog cache collapses every morning when 120,000 POS terminals simultaneously request their store catalogs at market open. Fixed 24-hour TTLs cause correlated expiry across all regional cache keyspaces. The resulting thundering herd exhausts the Postgres connection pool in under 10 seconds, causing HTTP 503 errors across all terminals. With Black Friday 40 days away — when terminal count and query volume will be 5x normal — this architecture will fail catastrophically at the worst possible time.

You must redesign the caching strategy using probabilistic early expiration (XFetch), distributed locking for single-writer cache rebuild, pre-market warming with bounded parallelism, memory pressure management, and a stale-data circuit breaker that degrades gracefully rather than returning errors.

### Explicit Requirements

1. Product catalog cache must survive 20x normal peak load without DB exhaustion.
2. Cache expiry must be staggered — no correlated mass-expiry across stores.
3. Only one worker per store catalog may execute a cache rebuild at a time (distributed lock).
4. Pre-market warming must run before terminal boot times, using staggered regional scheduling.
5. Redis Cluster memory utilization must stay below 70% under peak load.
6. If DB is unavailable during cache rebuild, serve stale data rather than returning an error.
7. Terminals must receive a response within 500ms for product catalog requests.
8. Cache warming job must complete for all 120,000 stores before the earliest regional market open (05:00 UTC).

### Hidden Requirements

- **Hint**: Re-read Marcus Delgado's message about Redis eviction policy `allkeys-lru`. The hidden requirement is that catalog keys for stores not-yet-opened must be protected from LRU eviction. What eviction policy or key design prevents pre-warmed keys from being evicted before the store opens?

- **Hint**: Re-read the message about "120k stores × 180KB = 21.6GB total." At 70% memory target that is 15.12GB. 21.6GB of catalog data does not fit. There is a hidden requirement: compress or tier the catalog data. Stores with no activity in 12 hours should not hold full catalog in hot cache.

- **Hint**: Re-read the sequence of regional boot times (UK 08:45, France 08:30, Germany 08:55). These are not random — they are timezone-correlated. The hidden requirement is a time-zone-aware cache warming schedule, not a single global job.

- **Hint**: Marcus Webb said "pre-warming job itself can cause a stampede." There is a hidden requirement: the pre-warming job's own DB queries must be rate-limited and queued, otherwise the warming job itself becomes the thundering herd.

### Constraints

- **Terminals**: 120,000 globally; 13,400 UK, ~18,000 France/Germany combined
- **Catalog size**: 180KB per store (gzipped), 21.6GB total
- **Redis Cluster**: 6 nodes (3 primary/3 replica), current 87% memory utilization; target max 70%
- **DB connection pool**: 500 connections max (PgBouncer to Postgres RDS)
- **Cache miss DB query time**: 8-12 seconds per store catalog rebuild
- **Terminal boot window**: Staggered by region, earliest 08:30 UTC
- **Pre-warming job must complete**: By 05:00 UTC at latest to cover first regional boot
- **Cache response SLA**: 500ms p99
- **Black Friday multiplier**: 5x terminal activity, 5x query volume
- **XFetch parameter beta**: Calibrate for 8-12 second rebuild time
- **Redis eviction policy**: Must be changed from `allkeys-lru`
- **Team**: 3 SRE engineers, 2 platform engineers

### Your Task

Design the complete caching architecture to eliminate the thundering herd for Black Friday. Include the XFetch implementation design, distributed lock strategy, pre-warming pipeline, memory management plan, and circuit breaker for stale data.

### Deliverables

- [ ] **Mermaid architecture diagram** showing: terminal request flow → Redis Cluster → XFetch evaluation → distributed lock (SETNX) → DB catalog query → cache write → stale-data fallback path. Include pre-warming job pipeline.

- [ ] **XFetch implementation design** (TypeScript interface sketch):
  ```typescript
  interface XFetchConfig {
    beta: number;          // calibration parameter (suggested: 1.0 for 10s rebuild)
    ttlSeconds: number;    // base TTL
    deltaSeconds: number;  // recompute time estimate
  }

  function shouldRecompute(
    ttlRemaining: number,
    delta: number,
    beta: number
  ): boolean;
  ```
  Explain why the formula `current_time - delta * beta * log(random())` works. Show example values for your catalog rebuild scenario.

- [ ] **Distributed lock design**: SETNX key format, TTL value, lock acquisition retry strategy, lock release on success/failure/timeout. What happens if the worker holding the lock crashes mid-rebuild?

- [ ] **Pre-warming pipeline design**:
  - Queue-based architecture (BullMQ or equivalent)
  - Regional batches with time-zone scheduling
  - Bounded parallelism (max N concurrent DB queries)
  - Show the math: 120,000 stores ÷ max 50 concurrent queries × 10 sec/query = minimum warm time. Does it fit in the 05:00 UTC window?

- [ ] **Memory management plan**:
  - Change eviction policy from `allkeys-lru` to `volatile-lru` or custom
  - Define TTL tiers: active stores (booted today) vs inactive stores (no activity 12h)
  - Show: active store memory footprint vs inactive, and how total stays below 70%

- [ ] **Circuit breaker design for stale data**: Two states (normal, degraded). In degraded state: return cached data with `X-Cache-Stale: true` header, log staleness age, alert if stale age > 4 hours. Do NOT return 503 unless cache is empty AND DB is down for > 30 seconds.

- [ ] **Scaling estimation**:
  - Normal peak: 120,000 terminals × 1 catalog request at boot ÷ 30-min stagger = 67 requests/sec average. Peak burst (UK market open 09:00): 13,400 requests in 60 seconds = 223 RPS.
  - After XFetch: estimate what % of those 223 will hit cache vs trigger rebuild. Show assumptions.
  - Black Friday: 5x multiplier. Recalculate.
  - Pre-warming throughput: calculate job runtime with 50 concurrent workers and 10s per query.

- [ ] **Tradeoff analysis** (minimum 3):
  1. XFetch probabilistic recomputation vs explicit TTL jitter: XFetch is adaptive; jitter is simpler. When is jitter sufficient and when do you need XFetch?
  2. Distributed lock (Redlock) vs single-writer design: Redlock has failure modes on network partition. What are they and how do you mitigate?
  3. Serving stale catalog vs 503: A stale catalog means a cashier might scan a product that doesn't exist in inventory. What is the business impact compared to a full terminal outage?

- [ ] **Redis Cluster key sharding strategy**: How do you distribute 120,000 store catalog keys across 6 primary nodes? Show key naming convention and hash slot distribution. Ensure hot stores do not cluster on one node.
