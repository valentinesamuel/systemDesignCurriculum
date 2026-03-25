---
**Title**: TradeSpark — Chapter 79: Advanced Kafka — Log Compaction for Order Book State
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #kafka #log-compaction #consumer-lag #partitioning #trading #order-book
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 98, Ch. 122
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**1:1 Session — You and Marco Bellini**
*TradeSpark War Room, Post-Incident Week*

**Marco**: The order book recovery problem is next on my list. When the trading engine restarts — for any reason — it takes 4 hours to replay the full Kafka topic and rebuild the order book state. 4 hours. The market doesn't wait.

**You**: What's in that topic?

**Marco**: Every order event since we started using Kafka. 18 months of events. 2 million events per minute at peak. At any given time, the topic has about 380 billion records.

**You**: 380 billion records replayed on restart. That's the problem.

**Marco**: I know the solution. Log compaction. Keep only the latest state for each order_id. But...

**You**: But what?

**Marco**: *(pauses)* I tried this at Citadel in 2019. The compaction deleted records that were still active — orders that were in the process of filling. The downstream consumer read a compacted record and missed the intermediate fill events. The position tracking was wrong. We had a $4M loss before we noticed. I shut it off immediately and nobody has touched it since.

**You**: Walk me through exactly what happened. The fill events for an in-flight order — what was the key?

**Marco**: The key was order_id.

**You**: And you set `cleanup.policy=compact` on the topic?

**Marco**: Yes.

**You**: So Kafka kept only the latest record per order_id. For an order that was 50% filled when the compaction ran, the latest record was the "50% fill" event. The earlier "submitted" and "25% fill" events were deleted.

**Marco**: Right. And the downstream consumer — the position tracker — needed all three events in sequence to correctly calculate the position.

**You**: That's a key design problem, not a log compaction problem. You were using an order_id as the key and compacting event log records — but those records weren't state records, they were event records. You need a different key design.

**Marco**: *(leaning forward)* What key?

**You**: What's the *state* you want to retain for the order book? Not the event history — just the current state of each order.

**Marco**: Current order state: open, partially filled, cancelled, fully filled.

**You**: So the compaction topic key should be `order_id` AND the record value should be the *current state*, not an event. You publish a "current state" record after each event — a state snapshot, not an event delta. Compaction keeps the latest state snapshot. The event log is a separate topic, with no compaction, for audit purposes.

**Marco**: Two topics.

**You**: Two topics. One for audit (event log, no compaction, 7-year retention). One for state recovery (current state snapshots, compacted, keeps only latest per key). Recovery replays only the state snapshot topic — which contains one record per open order, not 380 billion records.

**Marco**: *(slowly)* How long would recovery take?

**You**: How many open orders are there at any given time?

**Marco**: About 200,000.

**You**: 200,000 records. At 100k records/sec consumer throughput — 2 seconds.

**Marco**: *(long pause)* That's... 2 seconds versus 4 hours.

---

### Problem Statement

TradeSpark's trading engine takes 4 hours to recover after a restart because it replays 380 billion Kafka records to rebuild order book state. A previous attempt to use Kafka log compaction caused incorrect state by deleting intermediate event records that consumers needed. You need to design a Kafka topology that enables sub-10-second order book recovery while preserving the complete event log for SEC audit compliance.

---

### Explicit Requirements

1. Order book recovery time after restart must be < 10 seconds
2. The complete event history must be preserved (7-year retention) for SEC Rule 17a-4
3. Log compaction must not delete records that are required for event replay or position calculation
4. Partition strategy must support efficient consumer parallel processing
5. Consumer lag monitoring must alert if the state snapshot topic falls more than 30 seconds behind the event log
6. The solution must work with existing Kafka 3.x infrastructure

---

### Hidden Requirements

- **Hint**: Re-read Marco's Citadel story. The compaction deleted events for "in-flight" orders — orders that were in the process of filling. Your two-topic design publishes a state snapshot after every event. What happens if the process publishes the event but *crashes before publishing the state snapshot*? You now have an event in topic A that has no corresponding state update in topic B. The next recovery will start from a stale state snapshot. How do you handle this exactly-once guarantee between the two topics?

- **Hint**: The partition strategy for the event log uses `order_id` as the key. But 40 trading strategies trade different symbols. Some symbols (AAPL, TSLA) have 100x the order volume of other symbols. Does using order_id as the partition key create hot partitions? What's a better partition strategy?

---

### Constraints

- 380 billion historical records in the current event topic
- 200,000 open orders at any given time
- 2M events/minute peak
- 12 exchanges, 40 trading strategies, varying symbol volume distribution
- Recovery target: < 10 seconds
- Kafka 3.x with 96 partitions on the current event topic
- Consumer throughput: ~100k records/sec per consumer group instance

---

### Your Task

Design the dual-topic Kafka architecture (event log + state snapshot) for TradeSpark's order book, including partition strategy, log compaction configuration, and the exactly-once guarantee between topics.

---

### Deliverables

- [ ] Mermaid diagram: Dual-topic architecture (event log + state snapshot topic) with producer and consumer flows
- [ ] Log compaction configuration: `cleanup.policy`, `min.cleanable.dirty.ratio`, `min.compaction.lag.ms` for the state snapshot topic
- [ ] Partition strategy: How to partition the state snapshot topic to balance load across 40 strategies and 12 exchanges
- [ ] Exactly-once guarantee: How to ensure the state snapshot is published atomically with the event record (Kafka transactions)
- [ ] Consumer lag monitoring: What metric to watch, what threshold triggers an alert
- [ ] Recovery time calculation: At 200k open orders × 1KB avg state record size, what's the state snapshot topic size and recovery time? Show the math.
- [ ] Tradeoff analysis (minimum 3):
  - Log compaction vs consumer-side state snapshots (rebuilding state in application vs relying on Kafka compaction)
  - `order_id` as partition key vs `symbol` as partition key
  - Kafka transactions for exactly-once vs idempotent producer (performance trade-off)
