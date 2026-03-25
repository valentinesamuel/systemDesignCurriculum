---
**Title**: Pattern Summary 2 — Async, Queues & Event-Driven Patterns
**Level**: Reference
**Difficulty**: N/A
**Tags**: #pattern-summary #kafka #queues #async #event-driven #stream-processing
**Estimated Time**: 1 hour (review + reflection)
**Related Chapters**: Ch. 11–26
**Exercise Type**: Pattern Summary
---

> No deliverables. This is a reflection chapter.

---

### How This Arrived

**Slack DM — Marcus Webb → You**

**Marcus Webb**
You've now been through a logistics company running Kafka at scale, a media
company with massive fan-out, and an IoT company processing sensor events from
the edge. All three of them taught you the same six lessons about asynchronous
systems, and you probably didn't realize you were learning the same lesson
three times.

I wrote this doc for engineers transitioning from synchronous thinking to
async thinking. It's the mental model I wish I'd had in year three.

[Attachment: async-queue-event-driven-patterns.md]

---

### Patterns Covered: Ch. 11–26

---

#### Pattern 9: Consumer Group Isolation (continued from Pattern 6)

**Deep dive**: After VeloTrack, you saw this pattern in a production disaster.
The evolution of your understanding:

- **Level 1** (Pattern Summary 1): Separate groups prevent slow consumers from blocking fast ones.
- **Level 2** (this summary): Consumer groups also define your *fault isolation boundary*.
  A crashed consumer in Group A does not affect Group B. When you define consumer groups,
  you're defining your system's blast radius for consumer failures.
- **Level 3** (coming): Consumer groups also affect your replay/backfill strategy.
  If you need to replay historical events for one use case, you don't want to
  reprocess events for all other use cases.

---

#### Pattern 10: Dead-Letter Queue (DLQ) — The Full Pattern

**What it is**: An overflow queue for messages that fail processing after all retries.

**Full lifecycle** (beyond what Ch. 12 covered):

1. **Detection**: Message fails N times → routed to DLQ with error metadata
2. **Storage**: DLQ message carries original message + retry history + error details
3. **Alerting**: DLQ size > threshold → on-call notified
4. **Investigation**: On-call inspects DLQ; determines cause (bad message vs system bug)
5. **Resolution paths**:
   - Fix the message, replay to main queue (data fix)
   - Fix the consumer code, replay all DLQ messages (code fix)
   - Discard the message after investigation (intentional drop)
6. **Time-based expiry**: DLQ messages older than N days are archived or discarded

**Key insight**: A DLQ is not a failure — it's an intentional checkpoint. The
failure is a DLQ that fills up and no one looks at it.

---

#### Pattern 11: Fan-out on Write vs Fan-out on Read

**What it is**: Two strategies for distributing one event to many recipients.

**Fan-out on write**:
- When event occurs (e.g., new episode), immediately write to the inbox of every subscriber
- Reader queries: "what's in my inbox?" — fast, simple
- Problem: 10 million subscribers = 10 million writes at event time. Write amplification.

**Fan-out on read**:
- When event occurs, write once to the event store
- Reader queries: "what events from sources I follow?" — join at query time
- Problem: heavy read joins at query time. Slow for readers with many follows.

**Where it appeared**:
- Ch. 20 (Beacon Media): 8.5M push notification fan-out

**Hybrid approach**: Fan-out on write for "active" users (seen in last 24h);
fan-out on read for inactive users. Beacon Media's recommendation caching
(Ch. 22) used a similar hot/cold split.

---

#### Pattern 12: Event Time vs Processing Time

**What it is**: Two different clocks an event can be measured against.

- **Processing time**: when the event arrived in your system
- **Event time**: when the event actually occurred (recorded in the event itself)

**Why it matters**: For correctness, use event time. For simplicity, use processing time.

**Where it appeared**:
- Ch. 26 (AgroSense): sensors offline for 4 hours, then burst-replay with original timestamps.
  Processing-time windows incorrectly treat these as current events.
  Event-time windows correctly place them in their historical time buckets.

**The watermark**: Stream processors (Flink, Kafka Streams) use watermarks to track
event-time progress. "Watermark at T" means: "all events before time T have arrived
(or we've given up waiting)." Watermarks involve a tradeoff: aggressive watermarks
(advance fast) → lower latency but some late events dropped. Conservative watermarks
(wait longer) → higher latency but fewer events dropped.

---

#### Pattern 13: Kafka Partition Key Design

**What it is**: The partition key determines which partition (and thus which consumer)
processes a given event. Events with the same key always go to the same partition
and are processed in order by the same consumer.

**The rule**: Events that must be processed in order must share a partition key.

**Corollary**: Events that don't need ordered processing can use any key (including
round-robin) — this maximizes parallelism.

**Where it appeared**:
- Ch. 11 (VeloTrack): Delivery events without delivery_id as key → out-of-order state machine
- Ch. 15 (VeloTrack): Partition key = delivery_id → sequential processing per delivery

**Watch out for**: Hot partitions. If your partition key is not uniformly distributed
(e.g., one mega-merchant generates 50% of events), one partition gets 50% of the load.
Solution: add a suffix to the key for large entities (e.g., `delivery_id + shard_suffix`).

---

#### Pattern 14: Idempotent Stream Processing

**What it is**: Making stream consumers safe to run multiple times on the same event.

**Kafka's delivery semantics**:
- At-most-once: commit offset before processing. Fast, but can lose events on crash.
- At-least-once: commit offset after processing. Events may be processed twice.
- Exactly-once: Kafka transactions. Most complex, highest overhead.

**At-least-once + idempotent consumers = practical exactly-once**. This is the
pattern used in every production system in this curriculum.

**Idempotency key for stream consumers**:
- Use a natural key from the event: `delivery_id + event_type + event_timestamp`
- Check if the derived effect (state transition, row write) already exists before applying
- If already applied: skip. If not: apply.

**Where it appeared**:
- Ch. 15 (VeloTrack): Delivery state transitions with optimistic locking
- Ch. 12 (VeloTrack): DLQ replay must be idempotent

---

#### Pattern 15: ETL → Streaming Evolution

**What it is**: The evolution path from batch ETL to real-time streaming, and when each is appropriate.

**Stages**:
1. **Batch ETL**: Run periodically (hourly/daily). Simple. Works until latency requirement
   drops below batch interval.
2. **Micro-batch**: Batch every few minutes. Better latency, but still not real-time.
3. **Stream processing**: Continuous. Event-driven. Complex but low latency.

**You do NOT always need real-time**. Analytics dashboards can often tolerate
5-minute latency. Only real-time alerting and live displays need sub-minute latency.

**Where it appeared**:
- Ch. 24 (AgroSense): Hourly batch (6-hour alert latency) → real-time stream (3-min SLA)

**The cost of real-time**: State management. In a batch job, your database is
always the source of truth. In a stream processor, state lives in the processor
(in-memory or local RocksDB). This state must be: checkpointed (to survive restarts),
consistent with reference data (CDC), and sized appropriately.

---

#### Pattern 16: CDC — Change Data Capture

**What it is**: A technique for detecting and reacting to changes in a database
without polling — by reading the database's write-ahead log (WAL).

**Tools**: Debezium (for Postgres, MySQL, MongoDB), AWS DMS, Kafka Connect JDBC.

**Use cases**:
1. Sync reference data into stream processors (Ch. 24 — sensor configs → stream)
2. Replicate data between databases (primary → replica, or DB → data warehouse)
3. Event sourcing — treat every DB change as a published event
4. Cache invalidation — react to DB changes to invalidate relevant cache keys

**Key properties**:
- Low overhead on source DB (reads WAL, not main tables)
- Eventually consistent (small lag between DB write and CDC event)
- Bootstrap problem: CDC only sees changes; new consumers need a full snapshot first

---

### Cross-Industry Observations

Asynchronous patterns appear everywhere because synchronous systems cannot scale
to the throughput or failure tolerance required in production. The key question
is not "should I use async?" — the answer is almost always yes. The key question
is "where exactly is the sync/async boundary, and what are the consistency
guarantees at that boundary?"

| Problem | Pattern | Companies |
|---------|---------|-----------|
| Consumer isolation | Separate consumer groups | VeloTrack, Beacon |
| Poison pill resilience | Dead-letter queue | VeloTrack |
| Mass notification | Fan-out on write/read | Beacon Media |
| Physical sensor reliability | Event-time processing | AgroSense |
| Reference data in streams | CDC (Debezium) | AgroSense |
| Ordered event processing | Partition key design | VeloTrack |

---

### Reflection Questions

No answers are provided.

1. You've seen fan-out on write at Beacon Media (8.5M pushes). At what follower
   count does fan-out on write become unsustainable, and what's the architectural
   trigger to switch to fan-out on read? Is there a number, or is it more nuanced?

2. Kafka's exactly-once semantics (Kafka transactions) were not used in any system
   in this curriculum — instead, at-least-once + idempotent consumers was the pattern.
   When, if ever, would you choose Kafka transactions over idempotent consumers?
   What would the system need to look like for the additional complexity to be justified?

3. AgroSense's 6-hour batch lag was unacceptable for agricultural alerts. But
   the batch-to-streaming migration added significant complexity (state management,
   CDC, watermarks). Imagine you're advising a startup with a 10-person engineering
   team and 15-minute alert latency requirement. Do you recommend streaming or
   micro-batch? What's the minimum team size and infrastructure maturity required
   for streaming to be the right choice?
