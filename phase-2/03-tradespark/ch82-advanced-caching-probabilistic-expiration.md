---
**Title**: TradeSpark — Chapter 82: Advanced Caching — Probabilistic Early Expiration
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #caching #cache-stampede #probabilistic-expiration #redis #market-data #thundering-herd
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 100
**Exercise Type**: Incident Response / System Design
---

### Story Context

**#incidents — TradeSpark (Monday post-mortem)**
`08:45` **marco.bellini**: going to document what happened this morning
`08:46` **marco.bellini**: at 09:29:58 — 2 seconds before market open — all 40 trading strategies simultaneously requested the "current market snapshot" cache entry. it expired at exactly 09:30. redis received 40 simultaneous cache-miss requests. all 40 hit the market data feed API at once.
`08:47` **marco.bellini**: the market data feed API rate-limits at 30 requests/second per client. we hit 40 simultaneous requests in under 100ms. all 40 got rate-limited. all 40 cached nothing. all 40 retried. redis fell over from the retry storm.
`08:48` **marco.bellini**: this is the 4th monday in a row
`08:49` **you**: fourth monday specifically?
`08:50` **marco.bellini**: the market snapshot cache TTL is exactly 604800 seconds — one week. it was last set on a monday morning. it expires monday morning. every week.
`08:51` **you**: ah
`08:52` **diana.voss**: were trading strategies operating without market data during this window?
`08:53` **marco.bellini**: they were queuing orders and waiting for the cache to refresh. we didn't execute any trades during the 94-second window.
`08:54` **diana.voss**: 94 seconds of trading suspension. I need to know if this is a material disruption.
`08:55` **alex.hartmann**: @you — fix this by wednesday

**Slack DM — You → Marco Bellini**
`09:30`
**you**: I know what this is. have you heard of XFetch?
**marco.bellini**: no
**you**: probabilistic early expiration. instead of all consumers discovering the cache has expired at the same time, you probabilistically recompute the cache BEFORE it expires, based on how much time is left and how expensive the recomputation is.
**marco.bellini**: how does the probability work?
**you**: there's a formula: `current_time - (beta * delta * log(random()))`. when this value exceeds the cache expiration time, the current consumer recomputes the cache — even though it hasn't expired yet. other consumers still get the cached value. only the "winner" triggers a recompute.
**marco.bellini**: how early does it trigger?
**you**: depends on beta (a tuning parameter, usually 1.0) and delta (recomputation time in seconds). if recomputing the market snapshot takes 5 seconds (delta=5), a consumer might trigger early recomputation up to ~30 seconds before expiry on average. by the time the cache actually expires, the new value is already loaded.
**marco.bellini**: so the cache never actually expires from the consumers' perspective?
**you**: for any individual consumer: they either get the cached value (not early-expired) or they get the freshly computed value (early-expired). they never get a cache miss. the stampede never happens.
**marco.bellini**: that's elegant
**you**: it's a 20-line change. the hard part is the cache hierarchy design.

---

### Problem Statement

TradeSpark's market data cache stampede occurs every Monday morning when the weekly snapshot TTL expires exactly at market open. 40 trading strategies simultaneously hit the downstream market data API, triggering rate limiting and a 94-second trading suspension. You need to implement probabilistic early expiration (XFetch algorithm) and redesign the cache hierarchy to prevent any future cache stampede.

---

### Explicit Requirements

1. No cache stampede at market open (zero simultaneous cache-miss requests to downstream API)
2. Market data cache must serve < 1ms P99 to all 40 trading strategies
3. Cache warm-up must be complete before market open (09:30 EST)
4. The solution must handle the market data feed API's 30 req/sec rate limit
5. Cache hierarchy: in-process (L1) + Redis (L2) + market data feed (L3)
6. TTL jitter must prevent any future synchronized expiration across multiple cache entries

---

### Hidden Requirements

- **Hint**: Re-read the incident: "TTL is exactly 604800 seconds — one week. Set on a Monday morning, expires Monday morning." The root cause isn't just that 40 strategies hit simultaneously — it's that TTL was set to an exact round number that created a synchronized expiration. What's the TTL jitter strategy that prevents this, and why does it need to be applied at write time, not read time?

- **Hint**: The market data feed API rate-limits at 30 requests/second. With 40 trading strategies potentially needing different market data segments (AAPL, TSLA, SPY, etc.), the cache hierarchy isn't just "one cache entry." There are hundreds of symbol-specific entries. How does XFetch interact with a large number of cache entries that all have approximately the same TTL?

---

### Constraints

- 40 trading strategies, each potentially reading different symbol data
- Redis rate: can handle 500k ops/sec without degradation
- Market data feed API: 30 req/sec rate limit (external provider, cannot change)
- In-process L1 cache: each trading strategy process has its own JVM/Node.js heap
- Recomputation time (delta): 2–15 seconds depending on how many symbols are requested
- Market open: 09:30 EST daily, Monday–Friday

---

### Your Task

Implement the XFetch probabilistic early expiration algorithm and design the three-tier cache hierarchy for TradeSpark's market data.

---

### Deliverables

- [ ] XFetch algorithm explanation: Formula `current_time - (beta * delta * log(rand()))`, explain each parameter
- [ ] TypeScript implementation sketch: `getCachedWithEarlyExpiration(key, recomputeFn, ttl, beta)` function
- [ ] Cache hierarchy design: L1 (in-process, per-strategy process) → L2 (Redis Cluster) → L3 (market data API)
- [ ] TTL jitter strategy: How to add jitter to TTLs at write time to prevent synchronized expiration
- [ ] Redis Cluster sharding: How to shard market data by symbol across Redis nodes
- [ ] Scaling estimation: 40 strategies × 500 symbols × 1KB avg entry = how much Redis memory? Show the math.
- [ ] Tradeoff analysis (minimum 3):
  - XFetch vs distributed lock for cache warming (performance vs simplicity)
  - Cache-aside vs refresh-ahead for market data
  - Per-strategy L1 cache vs shared L1 cache (consistency vs memory efficiency)
