---
**Title**: Pattern Summary 1 — Data Layer & Caching Patterns
**Level**: Reference
**Difficulty**: N/A
**Tags**: #pattern-summary #caching #database #idempotency #performance
**Estimated Time**: 1 hour (review + reflection)
**Related Chapters**: Ch. 1–15
**Exercise Type**: Pattern Summary
---

> No deliverables. This is a reflection chapter.

---

### How This Arrived

**Slack DM — Marcus Webb → You**

**Marcus Webb**
You've been through NovaPay, MeridianHealth, and VeloTrack. Different industries.
Different teams. Different crises. But you've been solving variations of the same
handful of problems over and over.

I'm sending you a doc I wrote for the engineers I mentor. It's not a textbook.
It's a distillation. Read it. The reflection questions at the end are the ones
I ask in every senior engineering interview.

[Attachment: data-layer-caching-patterns.md]

---

### Patterns Covered: Ch. 1–15

---

#### Pattern 1: Idempotency Keys

**What it is**: A unique key attached to a request that lets the server detect and
deduplicate duplicate requests. If the server has already processed a request with
a given key, it returns the previous result rather than processing again.

**Where it appeared**:
- Ch. 1 (NovaPay): Idempotency keys on payment initiation — merchants retry safely
- Ch. 2 (NovaPay): The dual-write failure showed what happens when idempotency
  is applied at the wrong boundary (the queue job was idempotent, but the bank API
  call inside the job was not)
- Ch. 15 (VeloTrack): Delivery state transitions — idempotent writes via optimistic
  locking and sequence numbers

**The core insight**: Idempotency must be applied at *every* boundary where
re-execution is possible. Protecting the queue entry doesn't protect the
side effects that happen inside the job.

**When to use**: Any write operation that can be retried: payments, state transitions,
email/notification sends, external API calls with side effects.

---

#### Pattern 2: Write-Through Cache

**What it is**: When data is written to the database, it is also written to the
cache simultaneously. Reads are served from cache. The cache is always in sync
with the database (within a transaction boundary).

**Where it appeared**:
- Ch. 4 (NovaPay): Merchant balance cache — balance is expensive to recompute
  on every read; write-through keeps it current without requiring a full DB query

**Failure mode**: If the cache write fails after the DB write succeeds, cache is
stale until evicted or updated. Requires a compensation strategy.

**Contrast with**:
- **Cache-aside**: Application checks cache, misses → queries DB, populates cache.
  Cache can be stale during the miss window.
- **Write-behind**: Write goes to cache first, DB write is async. Risk of data loss
  if cache fails before DB write.

---

#### Pattern 3: Read Replica Routing

**What it is**: Direct heavy read queries (analytics, reporting, batch jobs) to
read replicas, preserving the primary database for writes and latency-sensitive reads.

**Where it appeared**:
- Ch. 9 (MeridianHealth): Clinical analytics routed to replica; real-time clinical
  queries on primary or replica-with-lag-check

**Key concerns**:
- **Replication lag**: Replicas are behind. The application must know which reads
  tolerate lag and which don't.
- **Read-your-own-writes**: A user who just wrote something must read from the
  primary or wait for replication to complete before reading from replica.
- **Connection pool math**: Don't over-connect. Total desired connections must not
  exceed `max_connections` without a pooling proxy.

---

#### Pattern 4: Database Indexing Strategy

**What it is**: Creating the right indexes to support query patterns without over-indexing
(which slows writes). Composite indexes must be ordered correctly; the leading column
determines index selectivity.

**Where it appeared**:
- Ch. 3 (NovaPay): Payments table had no indexes beyond PK; sequential scans on 14M rows
- Ch. 9 (MeridianHealth): Analytics queries competing for the same indexes as clinical reads

**Rules of thumb**:
- Put the highest-cardinality column first in composite indexes (unless range scan needed)
- Offset-based pagination degrades with large offsets; prefer cursor-based pagination
- `CREATE INDEX CONCURRENTLY` for production tables — it takes longer but doesn't lock writes
- Every index is a write overhead — create only what queries actually need

---

#### Pattern 5: Dead-Letter Queue (DLQ)

**What it is**: A secondary queue that receives messages that have failed processing
after all retries. Prevents "poison pill" messages from blocking the main processing
queue forever.

**Where it appeared**:
- Ch. 12 (VeloTrack): A single malformed notification message blocked 2.3M messages
  for 9 hours because there was no DLQ

**Design principles**:
- DLQ messages must carry original message content, error reason, retry count, timestamps
- DLQ is not a trash can — every message represents a business event that needs review
- Replay must be idempotent
- Alert when DLQ accumulates above a threshold (not after days)

---

#### Pattern 6: Consumer Group Isolation

**What it is**: Each independent processing use case should have its own Kafka consumer
group, so that a slow consumer in one use case doesn't block other use cases.

**Where it appeared**:
- Ch. 11 (VeloTrack): Four different consumers in one group; analytics slowness
  caused ETA lag and 4-hour incident

**Key rule**: Consumer groups that have different SLAs, different processing speeds,
or different failure modes must be separate. A real-time ETA consumer and a batch
analytics consumer sharing a group is a category error.

---

#### Pattern 7: Partition Key Design

**What it is**: Choosing the Kafka partition key so that all events that must be
processed in order land on the same partition (and therefore are consumed in order
by the same consumer).

**Where it appeared**:
- Ch. 11 (VeloTrack): Round-robin partitioning caused delivery events for the same
  delivery to be processed out of order, breaking the state machine
- Ch. 15 (VeloTrack): Correct partition key (`delivery_id`) ensures sequential processing

**Rule**: If two events have a causal ordering relationship (A must happen before B),
they must share a partition key.

---

#### Pattern 8: Compliance-First Architecture

**What it is**: Building audit trails, access controls, and data residency into the
system architecture from the start, not retrofitting them after a compliance failure.

**Where it appeared**:
- Ch. 5 (NovaPay): PCI-DSS design review ambush revealed absent CDE boundary and audit logs
- Ch. 7 (MeridianHealth): HIPAA audit trail design — tamper-evident, append-only, queryable
- Ch. 8 (MeridianHealth): Data residency failure — PHI replicated to EU without consent

**The pattern**: Compliance requirements (PCI, HIPAA, GDPR) are not features to add later.
They define architectural constraints that affect every layer: storage, networking, access
control, and logging. Discover them early.

---

### Cross-Industry Observations

These patterns appeared across industries because they solve fundamental problems that
exist wherever data is written, stored, and read at scale:

| Problem | Pattern | Companies |
|---------|---------|-----------|
| Duplicate writes | Idempotency keys | NovaPay, VeloTrack |
| Slow reads | Caching, read replicas, indexes | NovaPay, MeridianHealth |
| Queue blockage | DLQ, consumer group isolation | VeloTrack |
| Out-of-order processing | Partition key design, sequence numbers | VeloTrack |
| Compliance gaps | Audit trails, CDE boundaries | NovaPay, MeridianHealth |

---

### Reflection Questions

No answers are provided. Sit with these.

1. You've seen idempotency applied at three layers: the HTTP API (idempotency key
   header), the queue job (job ID deduplication), and the database write (optimistic
   locking). For a given system, how do you decide *which* layer is the right place
   to enforce idempotency? Can you have too many idempotency layers? What does that cost?

2. Write-through cache keeps the cache and DB in sync. But "in sync" assumes the write
   to both succeeds atomically — and it never truly does. A cache write and a DB write
   are two separate operations. What is the maximum acceptable inconsistency window
   for financial data? For medical data? For delivery tracking data? How does the answer
   change your cache design in each domain?

3. Compliance was retrofitted in every company you've worked at so far — it was never
   designed in from day one. Is that avoidable? Or is it an inherent property of startups
   that compliance architecture always lags the product? If you were the first engineer
   at a company, what three compliance-related decisions would you make on day one?
