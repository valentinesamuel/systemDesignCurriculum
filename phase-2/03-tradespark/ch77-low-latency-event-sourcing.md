---
**Title**: TradeSpark — Chapter 77: Low-Latency Event Sourcing for Trade Execution
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #event-sourcing #cqrs #trading #low-latency #append-only #temporal-queries
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 66, Ch. 80, Ch. 126
**Exercise Type**: System Design / Compliance
---

### Story Context

**#engineering — TradeSpark**
`Monday 09:14` **diana.voss** *(Compliance Officer)*: @you — I need to talk to engineering today. SEC inquiry dropped this morning.
`09:15` **you**: what's the inquiry?
`09:16` **diana.voss**: they want to reconstruct the exact state of TradeSpark's order book for AAPL at 14:32:07.443 EST on March 3rd. specifically: what orders were open, what executions had occurred, what was the bid-ask spread at that exact millisecond.
`09:17` **you**: ...how precise is "exact millisecond"?
`09:18` **diana.voss**: the SEC's market surveillance system has nanosecond timestamps. they want our records to match. to the millisecond at minimum.
`09:19` **you**: let me check if the current system can produce this
`09:23` **you**: okay. the current system stores mutable state. order records get UPDATED in place as they fill. we have a timestamp on the last modification. we do NOT have a full history of state transitions.
`09:24` **diana.voss**: so you can't reconstruct the order book at 14:32:07.443?
`09:25` **you**: not accurately. we can show you what it looked like *eventually* — after all the modifications settled. not what it looked like at that exact moment.
`09:26` **diana.voss**: that's a regulatory gap. SEC Rule 17a-4 requires that we be able to reconstruct all trading activity from electronic records. "mutable state with last-modification timestamp" does not satisfy that requirement.
`09:27` **you**: understood. how much time do we have?
`09:28` **diana.voss**: the inquiry response deadline is 60 days. but the underlying system needs to be fixed permanently. not just for this inquiry.
`09:29` **alex.hartmann**: @you — this is now your top priority. everything else waits.

**1:1 — You and Diana Voss**
*TradeSpark Offices, 2:00 PM*

**Diana**: Let me explain what the SEC actually needs. Not just for this inquiry — for ongoing compliance.

**You**: Please.

**Diana**: Every order event must be recorded with: the order ID, the event type (submitted, modified, partial_fill, cancelled, fully_filled), the timestamp in nanoseconds, the state of the order at that moment, and the identity of the trading strategy that issued the event. All of this must be immutable. And it must be reproducible — if I ask you to replay the order book from 09:30:00 to 14:32:07.443, you must produce the identical order book state, every time I run the replay.

**You**: That's an event-sourced append-only log. I've built this before.

**Diana**: In pharma, yes — Fatima Al-Rashid mentioned you. But this is different. Your pharma audit trail had 100,000 writes per day. Our order management system has up to 2 million order events per *minute* during active trading. And the latency budget for recording an event to the audit log is 50 microseconds — not milliseconds. Microseconds.

**You**: *(pause)* 50 microseconds is tighter than most network round trips.

**Diana**: Welcome to trading infrastructure. The log write has to be in-process, not a network call.

---

### Problem Statement

TradeSpark's order management system uses mutable state — orders are updated in place as they fill. This prevents temporal reconstruction ("what was the order book state at timestamp T?") required by SEC Rule 17a-4. A complete event-sourced audit trail is needed that handles 2 million order events per minute with sub-50-microsecond write latency. The current CQRS implementation (if any) must be redesigned to use the event log as source of truth.

---

### Explicit Requirements

1. Every order state transition must be recorded as an immutable event with nanosecond timestamp
2. The event log must support temporal queries: "what was the complete order book state at time T?"
3. Write latency for recording an event must be < 50 microseconds P99
4. The CQRS read side must serve current order book state with < 1ms latency
5. The event log must retain all events for minimum 7 years (SEC Rule 17a-4)
6. Replay must be deterministic — replaying from T to T+N always produces identical order book state

---

### Hidden Requirements

- **Hint**: Re-read Diana's requirement: "the identity of the trading strategy that issued the event." TradeSpark runs 40 trading strategies simultaneously. Some strategies may trade the same symbol at the same time. How does your event schema distinguish events from two different strategies operating on the same order book?

- **Hint**: Diana said "50 microseconds is tighter than most network round trips." If the log write can't be a network call, what storage medium is the event log backed by? And if it's in-process, how do you ensure durability (events survive process restart)?

---

### Constraints

- 2M order events/minute peak = 33,333 events/second
- Write latency budget: < 50 microseconds P99
- Read latency (current order book): < 1ms
- 7-year retention for SEC compliance
- 40 trading strategies operating simultaneously across 12 exchanges
- Current language/runtime: C++ for low-latency hot path, TypeScript for management plane

---

### Your Task

Design the event-sourced order management system for TradeSpark. Include the event schema, the in-process write path for sub-50µs latency, the CQRS read model for current order book state, and the temporal query design.

---

### Deliverables

- [ ] Mermaid diagram: Event-sourced order management architecture (write path → event log → projections → read models)
- [ ] Event schema: `OrderEvent` with all fields including nanosecond timestamp, strategy_id, sequence_number
- [ ] Write path design: How to achieve < 50µs write latency (in-process journal, memory-mapped files, or other approach)
- [ ] Durability guarantee: How in-process writes survive process restarts
- [ ] CQRS read model: Order book projection schema and update mechanism
- [ ] Temporal query design: How to reconstruct order book state at an arbitrary nanosecond timestamp
- [ ] Scaling estimation: At 33,333 events/sec, what's the storage growth rate? Over 7 years?
- [ ] Tradeoff analysis (minimum 3):
  - In-process journal vs Kafka for event log durability
  - Nanosecond-precision timestamps vs millisecond (storage overhead trade-off)
  - Full replay vs snapshot+replay for temporal queries (performance trade-off)
