# Pattern Summary — Multi-Region Systems, Real-Time Scale & Advanced Caching

**Type**: Pattern Summary Chapter
**Covers**: Chapters 77–89 (TradeSpark, GigGrid)
**Chapter**: 91

---

**Email — Marcus Webb**
*Subject: (no subject)*
*Sent to: you@[current-company].io*

I don't normally do this in email. But I'm traveling and I don't have time for a call, and I wanted to get this to you before you go into the next arc.

Attached is a diagram I drew after reading your GigGrid CRDT write-up. I annotated it. Read my notes.

The pattern across TradeSpark and GigGrid is: **state is expensive at scale, so the design must define what "state" actually means**.

At TradeSpark: trade execution state (the order book) is 33k events/sec. You can't afford to compute it from scratch on every query. The state snapshot topic is a materialized view of the event log — cached state. XFetch is a caching strategy. The mTLS service mesh is protecting state in transit.

At GigGrid: worker availability is state. Job assignments are state. Real-time connections ARE state. The CRDT work was about what "correct availability state" means when two replicas disagree.

The patterns are the same idea at different scales with different correctness requirements.

— Marcus

P.S. You did well at Apex. Priya told me. I didn't tell you that.

---

## Patterns 43–49

### Pattern 43: Active-Active vs Active-Passive Selection Criteria

**Active-active** (all nodes write): Higher availability, better write throughput, requires conflict resolution strategy. Appropriate when: each region has local write volume, network partition is acceptable, conflict resolution is well-defined.

**Active-passive** (one primary writes, replicas serve reads): Simpler consistency model, no conflicts, but write bottleneck at the primary and failover latency during primary loss.

**The decision criterion**: Can you define a correct conflict resolution strategy for your data? If yes: active-active. If your data has "exactly one winner" semantics (like job assignment): you need a reservation protocol at the application layer even in active-active, which approaches active-passive in terms of coordination overhead for that specific operation.

---

### Pattern 44: CRDT Types and When to Use Each

| CRDT Type | Use When | GigGrid Example |
|-----------|----------|----------------|
| G-Counter | Counts that only increase | Worker job completions |
| PN-Counter | Counts that increase and decrease | Platform-wide job pool count |
| G-Set | Set that only adds | Worker certifications (can add, never remove) |
| 2P-Set | Set with removals (removed items cannot be re-added) | Banned job categories |
| OR-Set | Set with removals and re-adds | Worker availability job type preferences |
| LWW-Register | Single value, latest timestamp wins | Worker profile (name, contact) |
| MV-Register | Single value, conflict visible as multiple versions | Controversial preference settings |

**The key insight**: CRDTs don't solve all distributed state problems. They solve problems where the data type has mathematical convergence properties. Job assignment doesn't have convergence properties — it requires a winner. Worker availability does.

---

### Pattern 45: WebSocket vs SSE vs Long Polling Decision Matrix

| Criterion | WebSocket | SSE | Long Polling |
|-----------|-----------|-----|--------------|
| Directionality | Bidirectional | Server → Client only | Server → Client only |
| HTTP/2 support | Separate protocol | Native HTTP/2 | HTTP/1.1 and HTTP/2 |
| Corporate proxy support | Often blocked | Works (HTTP) | Works (HTTP) |
| Reconnection | Manual | Built-in (EventSource) | Client-initiated |
| Multiplexing | Per-connection | HTTP/2 multiplexed | One request per poll |
| Best for | Interactive (chat, gaming) | Unidirectional push (feeds, notifications) | Simple, lowest common denominator |

**GigGrid decision**: SSE for job dispatch (unidirectional, proxy-safe, HTTP/2 native). WebSocket only if workers need to send real-time data back to servers (currently not needed).

---

### Pattern 46: Pub/Sub Connection Sharding Strategy

**The problem**: At 2M connections, no single server can hold all connections. You need to shard workers to connection servers.

**The strategy**: Consistent hash ring over connection servers. `server = hashRing.get(worker_id)`. Dispatch service maintains the same ring — knows which server holds worker X without querying a registry.

**The catch**: When a connection server is added or removed, the ring remaps some workers to new servers. Those workers need to reconnect. Design your reconnection flow to handle this gracefully.

---

### Pattern 47: Feature Flag Lifecycle

Every feature flag has 4 stages:
1. **Definition**: Flag created, default OFF, code path gated
2. **Rollout**: Flag enabled for increasing % of users (1% → 5% → 25% → 100%)
3. **Stable**: Flag at 100% for 30+ days, no issues, code path validated
4. **Removal**: Flag removed from code, dead code path deleted

Flags that skip stage 4 become permanent feature flag debt. Teams that maintain 200 "stable" flags have a maintenance problem. Enforce automated warnings for flags older than 90 days at 100%.

---

### Pattern 48: Canary Deployment Automated Rollback Triggers

Not all metrics are equal for rollback decisions:

- **Error rate** (best trigger): Directly indicates user impact. Threshold: > 2× baseline error rate.
- **Latency P99** (good trigger): Indicates performance regression. Threshold: > 1.5× baseline P99.
- **CPU/memory** (weak trigger): Infrastructure metrics. Useful for capacity, not correctness.
- **Business metrics** (lagging trigger): Conversion rate, job acceptance rate. Real impact but slow to surface.

**The rule**: Rollback on error rate or latency P99. Never rollback on CPU/memory alone. Never wait for business metrics to rollback.

---

### Pattern 49: Probabilistic Cache Early Expiration (XFetch Algorithm)

**The formula**: `current_time - (beta × delta × log(random()))`

When this value exceeds the cache expiration time, the current consumer recomputes the value (even though it hasn't expired yet). Other consumers get the cached value.

**Parameters**:
- `beta`: Tuning parameter (default 1.0). Higher = more aggressive early expiration.
- `delta`: Expected recomputation time in seconds. Longer recomputation → triggers earlier.
- `random()`: Uniform random [0,1]. Ensures only one consumer in N triggers the early recomputation.

**Why this works**: The probability of triggering early recomputation increases as the cache approaches expiration. In expectation, one consumer triggers a recompute before the cache expires, and serves the new value to all subsequent consumers with no cache miss.

---

## Cross-Industry Observation

Marcus Webb's annotation on the diagram: *"TradeSpark needed microsecond state consistency. GigGrid needed eventually-consistent state correctness. These sound like different problems. They're both asking: 'what does it mean for distributed state to be correct?' The answer is always specific to the data type and its semantic requirements. There is no universal answer. Anybody who tells you otherwise is selling you a database."*

Real-time requirements across the curriculum are consistently underspecified. "Real-time" has meant: 50 microseconds (trading), 500ms (job dispatch), 30 seconds (CQRS projection consistency), 60 minutes (batch window). When a product manager says "real-time," your first question should always be: "Real-time as in 50 microseconds, or real-time as in 30 seconds? Because those are the same word for very different engineering problems."

---

## Reflection Questions

1. In Pattern 44 (CRDTs), OR-Set allows worker availability job types to be added and removed freely. But what if two regions simultaneously add and remove the same job type — region A adds "forklift," region B removes "forklift" at the same moment? OR-Set has a specific resolution for this concurrent conflict. What is it, and is it the right semantic for GigGrid's use case?

2. Pattern 49 (XFetch) prevents cache stampede by probabilistically pre-computing cache values before expiry. But XFetch assumes you know `delta` (recomputation time) in advance. What happens if recomputation time is highly variable — sometimes 2 seconds, sometimes 60 seconds depending on system load? How do you set `beta` when `delta` is uncertain?

3. In Pattern 43 (Active-Active vs Active-Passive), job assignment requires a "reservation protocol" even in an active-active system. This protocol introduces coordination that effectively makes job assignment active-passive for that operation. Identify two other data types in GigGrid that appear to fit the active-active model but actually require similar reservation semantics. What's the general principle for detecting when "active-active with conflict resolution" is insufficient?
