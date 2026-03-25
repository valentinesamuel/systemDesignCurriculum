---
**Title**: Pattern Summary 5 — Distributed Systems Fundamentals
**Level**: Reference
**Difficulty**: N/A
**Tags**: #pattern-summary #distributed #cap-theorem #saga #idempotency #exactly-once #feature-store
**Estimated Time**: 1 hour (review + reflection)
**Related Chapters**: Ch. 49–58
**Exercise Type**: Pattern Summary
---

> No deliverables. This is a reflection chapter.

---

### How This Arrived

**Slack DM — Marcus Webb → You**

**Marcus Webb**
You've been through two of the hardest distributed systems problems in practice:
OmniLogix's consistency problem (14 data centers, CAP theorem made real) and
LuminaryAI's data pipeline correctness problem (silently wrong ML predictions).

Different companies. Different domains. Same family of problems underneath.

Let me give you the synthesis before you head into SkyRoute.

[Attachment: distributed-systems-fundamentals.md]

---

### Patterns Covered: Ch. 49–58

---

#### Pattern 29: CAP Theorem Is a Per-Operation Decision

**The misconception**: "Our system is CP" or "our system is AP." Systems are
not uniformly one or the other.

**The reality**: Every operation has a consistency requirement. You make the
CAP tradeoff per operation, not per system.

| Operation | Consistency | Why |
|-----------|-------------|-----|
| Inventory allocation | Strong | Double-allocation costs $240k |
| Catalog read | Eventual | Stale price by $0.01 doesn't matter |
| Payment processing | Strong | Cannot charge twice |
| Order history view | Eventual | 2-second lag is acceptable |
| Recommendation serving | Eventual | Slightly stale is acceptable |

**The architecture implication**: Per-operation consistency requires routing
infrastructure. Allocation requests go to a quorum write path. Read requests
go to a local replica path. The routing is explicit, documented, and monitored.

**Where it appeared**: OmniLogix (Ch. 49) — $240k BMW double-allocation incident.

---

#### Pattern 30: The Saga Pattern — Distributed Transactions Without 2PC

**The problem 2PC solves**: atomic commit across multiple services.
**Why 2PC is dangerous at scale**: the coordinator becomes a single point of
failure; blocking during coordinator failure; doesn't work with external third-party APIs.

**The saga alternative**:
- Each step has a forward action and a compensating action (the undo)
- Steps execute in sequence; on failure, compensating actions run in reverse
- No coordinator holds a lock; each step commits independently

**The three compensation states** (Marcus Webb's taxonomy):
1. **Fully compensatable**: compensation undoes the step completely (inventory release)
2. **Partially compensatable**: compensation happens with a cost (10% cancellation fee)
3. **Non-compensatable**: compensation is impossible; human intervention required

**The key design rule**: Compensation operations inherit the consistency requirements
of their forward operations. If inventory allocation requires a quorum write,
inventory release also requires a quorum write.

**Where it appeared**: OmniLogix (Ch. 50).

---

#### Pattern 31: Idempotency Has Two Requirements

**Requirement 1 — safe execution**: calling the same operation twice produces
the same result. The server rejects or deduplicates the second call.

**Requirement 2 — queryable state**: the caller can ask "did this operation
complete?" without re-executing. The server stores (idempotency_key → result)
and exposes a GET endpoint.

**Why Requirement 2 matters**: when a call times out, the caller doesn't
know if it succeeded or failed. Without Requirement 2, the only option is
to retry (risking double-execution) or give up (risking lost operations).
With Requirement 2, the caller queries the status of the previous attempt
before deciding to retry.

**The write-first pattern**: write the operation record as "pending" BEFORE
calling the external API. This guarantees the record exists even if the caller
crashes between the call and the response. On restart: check the record. If
pending and old, query the external API for status.

**The deterministic key rule**: idempotency keys must be generated from the
operation parameters (not random UUIDs), so they are reproducible on retry
after a crash.

**Where it appeared**: NovaPay (Ch. 2), OmniLogix (Ch. 51) — PM-1442 triple-billing.

---

#### Pattern 32: Kafka Exactly-Once — Three Separate Layers

Engineers often believe "Kafka gives me exactly-once delivery." This is
imprecise to the point of being dangerous.

| Layer | Mechanism | What It Guarantees |
|-------|-----------|-------------------|
| Producer → Broker | `enable.idempotence = true` + `acks=all` | No duplicate messages in the partition |
| Broker → Consumer | At-least-once delivery | Message will be delivered; may be delivered more than once |
| Consumer processing | Application idempotency | YOUR code must be idempotent |

**The outbox pattern** is how you make DB + Kafka writes atomic:
1. Write to DB and insert into `outbox` table in ONE transaction
2. Relay process reads outbox and publishes to Kafka
3. If relay crashes after publish but before delete: duplicate publish
4. Consumer idempotency handles the duplicate

**The consumer idempotency contract**:
```
receive event
→ INSERT INTO processed_events WHERE (event_id, consumer_name) UNIQUE
→ if conflict: already processed, skip
→ else: execute business logic + INSERT to outbox (for next step)
```

**Where it appeared**: VeloTrack (Ch. 15), OmniLogix (Ch. 52).

---

#### Pattern 33: Feature Stores — Two Write Paths, One Read Path

ML systems that serve real-time predictions have a fundamental tension:
- **Batch accuracy**: features computed over 30-day windows are statistically
  rich but stale
- **Real-time freshness**: streaming updates are fresh but lack historical context

**The solution**: two write paths converging on one read path.

- **Batch write path**: nightly full-vector recompute with historical context.
  Writes authoritative values for all fields.
- **Real-time write path**: streaming delta updates for high-priority signals
  (inventory status, recent purchases). Writes only the specific fields.

**Conflict resolution rule**: batch writes are authoritative (have full context);
real-time writes are best-effort. If batch and real-time write to the same
field concurrently, batch wins. Implement with versioned Redis keys and Lua
scripts for atomic compare-and-set.

**Training-serving skew**: the features served at prediction time must match
the distribution the model was trained on. If real-time features diverge
significantly from the training distribution, the model degrades silently.

**Where it appeared**: LuminaryAI (Ch. 54, Ch. 55).

---

#### Pattern 34: Three Layers of ML Observability

Infrastructure monitoring ("is the system up?") is insufficient for ML systems
because ML systems can be "up" and produce wrong outputs simultaneously.

**Layer 1 — Data quality** (cheapest, catches most problems):
- Validate inputs before they reach the model
- Schema validation, null checks, distribution checks on input data
- Halt ingestion on failure; never silently ingest corrupt data

**Layer 2 — Feature distribution**:
- Monitor the statistical distribution of each feature against the training baseline
- Drift detection: KL divergence, 2-sigma threshold
- Alert when features shift significantly relative to what the model was trained on

**Layer 3 — Prediction quality** (most latency, most business-relevant):
- Monitor proxy metrics (CTR, conversion rate, bounce rate)
- Requires feedback from downstream systems (client engagement webhooks)
- Latency: hours to days; catches problems that Layers 1-2 miss

**The key insight**: build all three layers. In that order. Skipping Layer 1
and building Layer 3 first is backwards — it catches problems after they've
propagated to the model output.

**Where it appeared**: LuminaryAI (Ch. 57) — FashionHub 10-day invisible degradation.

---

#### Pattern 35: Batch Pipeline Failure = Single Point of Failure

Any system that depends on a single nightly batch job for correctness has
exactly one failure mode: the job fails. The system then degrades silently
until someone notices.

**The resilience pattern**:
1. Streaming pipeline provides continuous updates for high-priority signals
2. Batch pipeline provides correctness baseline (full recompute)
3. Neither pipeline is the sole source of truth
4. Monitoring validates the output of both pipelines independently

**The danger of coexistence**: two pipelines writing to the same store can
produce inconsistent state. Design explicit merge semantics from the start.

**Where it appeared**: AgroSense (Ch. 24) — ETL to streaming evolution;
LuminaryAI (Ch. 55) — inventory pipeline failure caused 10-day degradation.

---

### Cross-Domain Observations

Distributed systems problems appear in every domain, but the cost of failure differs:

| Problem | Supply Chain | ML Platform | Finance |
|---------|-------------|-------------|---------|
| Consistency failure | Double-allocation ($240k) | Wrong recommendation (CTR drop) | Double-charge (fraud) |
| Stale data | Wrong inventory (shipment failure) | Out-of-stock recommendation (UX) | Stale price (arbitrage) |
| Non-idempotent retry | Triple-billing ($84k) | Duplicate training run (wasted compute) | Duplicate payment |
| Pipeline failure | Order stuck in saga | 10-day invisible degradation | Missed transaction |

**The meta-pattern**: the same underlying failure mode (stale data, non-idempotency,
consistency violation) has domain-specific consequences — but the solution is
usually the same pattern, adapted to the domain's constraints.

---

### Reflection Questions

No answers provided.

1. The saga pattern requires each step to have a compensating transaction. But
   for some operations — sending a physical shipment, charging a credit card —
   compensation is partial or impossible. Marcus Webb described three compensation
   states. How would you design a saga for a system where 20% of steps are
   non-compensatable? What is the minimum saga design that handles these cases
   without requiring manual intervention for every failure?

2. Kafka exactly-once semantics require three separate layers. The outbox pattern
   handles the DB + Kafka atomicity problem. But the outbox relay is itself a
   service that can fail. If your organization has 20 services, each with its own
   outbox relay, you now have 20 relay processes to operate and monitor. At what
   scale does a shared outbox relay service make sense? What are the failure
   modes of a shared relay that per-service relays avoid?

3. Feature distribution monitoring compares today's features to the training
   baseline. But models are periodically retrained — on purpose, to improve.
   After retraining, the new "normal" distribution is different. When the model
   is retrained, should the feature distribution baseline automatically update?
   If yes: you can't detect drift that occurred before the retraining.
   If no: you'll false-positive on every retrain. How do you resolve this?
