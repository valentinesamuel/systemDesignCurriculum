---
**Title**: TradeSpark — Chapter 80: Distributed Transactions — 2PC vs Saga
**Level**: Staff
**Difficulty**: 9
**Tags**: #distributed-transactions #2pc #saga #settlement #trading #compensating-transactions
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 77, Ch. 81
**Exercise Type**: System Design / Debugging
---

### Story Context

**Email Chain**

---
**From**: yuki.tanaka@tradespark.io
**To**: alex.hartmann@tradespark.io; you@tradespark.io
**Subject**: Settlement Failures — Halting Trading Recommendation
**Date**: [Week 5 at TradeSpark]

Alex,

Three settlement failures this week. I need to explain the severity clearly.

Failure #1 (Monday): Strategy AAPL-MOMENTUM executed a buy order on NASDAQ. The execution was confirmed. The corresponding position update in the risk system failed due to a database timeout. Result: we hold a long position in AAPL that the risk system doesn't know about. The risk dashboard shows incorrect exposure.

Failure #2 (Wednesday): Multi-leg strategy ARBS-SPREAD executed simultaneously on NYSE and CBOE. NYSE confirmed. CBOE failed. We're now in a position where we're long on NYSE but have no offsetting short on CBOE. Unintended directional risk.

Failure #3 (Friday): Settlement batch to prime broker (Morgan Stanley) included a trade that our internal records show as cancelled. The prime broker processed it. We are now arguing about whether the trade happened.

Three phantom positions in one week. I am recommending we halt discretionary trading immediately until this is resolved.

Yuki Tanaka, Head of Risk

---
**From**: alex.hartmann@tradespark.io
**To**: you@tradespark.io
**Subject**: FWD: Yuki's email

Yuki is right. We're pausing discretionary trading effective today. You have 72 hours.

---

**#engineering — Emergency Session**
`[Same afternoon]`
**you**: I've looked at all three failure modes. they all have the same root cause: the trade execution and the risk position update are two separate database writes with no coordination. if either fails, we have inconsistent state.
**marco.bellini**: the obvious solution is a distributed transaction. 2-phase commit across both databases.
**you**: 2PC works in theory. in practice, 2PC with a coordinator failure leaves both databases in a locked state until the coordinator recovers. at trading latency requirements, a 2PC coordinator failure means seconds of write blocking. that's unacceptable.
**marco.bellini**: so 2PC won't work?
**you**: 2PC is correct but it introduces coordinator failure as a new risk. I want to model a saga instead.
**diana.voss**: I don't know what a saga is
**you**: a saga is a sequence of local transactions with compensating transactions for rollback. no distributed coordinator. if step 2 fails, step 2's compensating transaction undoes step 1.
**diana.voss**: what's the compensating transaction for "we bought 10,000 AAPL at $183.50"?
**you**: *(pauses)* ...you sell 10,000 AAPL.
**diana.voss**: at whatever the market price is now. which might be different.
**you**: ...yes.
**diana.voss**: so the compensating transaction for a failed trade settlement is... a new trade. that has its own execution risk.
**yuki.tanaka** *(who just joined)*: this is why trade settlement is hard

---

### Problem Statement

TradeSpark's trade settlement process involves multiple independent services: trade execution engine, risk management system, position tracker, and prime broker settlement queue. Currently these services update independently with no coordination. When any service fails, the system enters an inconsistent state (phantom positions, missing risk exposure). You need to design a distributed transaction mechanism for trade settlement that maintains consistency without introducing coordinator bottlenecks that violate latency requirements.

---

### Explicit Requirements

1. All legs of a trade must either all complete or all roll back (no partial settlement)
2. The coordinator (if any) must not become a bottleneck — trade execution latency budget: 500µs P99
3. Failed settlement legs must generate a visible alert to the Risk team within 5 seconds
4. Multi-leg trades (cross-exchange arbitrage) must settle atomically across all exchanges
5. Compensating transactions must be tracked and auditable (SEC audit trail)
6. The solution must handle the prime broker integration (external system — no distributed locks possible)

---

### Hidden Requirements

- **Hint**: Re-read Yuki's Failure #3: "Settlement batch to prime broker included a trade our internal records show as cancelled." The prime broker is an external system with its own idempotency rules. Your saga has a compensating transaction design — but what happens when the compensating transaction to the prime broker fails? You've now issued a trade, tried to cancel it, and the cancel failed. You're now arguing with Morgan Stanley. How does your design prevent this scenario?

- **Hint**: Re-read Diana's question: "the compensating transaction for a failed trade settlement is... a new trade." Diana has identified a fundamental asymmetry: compensating a failed financial transaction may create a new financial transaction with different market risk. How does your saga design document this risk and ensure the Risk team is notified when a compensating transaction is executed?

---

### Constraints

- 4 services in the settlement chain: execution engine, risk system, position tracker, prime broker queue
- Prime broker is an external system (no distributed transactions possible with it)
- Trade execution latency budget: 500 microseconds (cannot use synchronous 2PC coordinator in the hot path)
- 5,000 trades/minute at peak
- Failed settlement visible to Risk team within 5 seconds
- All settlement activity must be auditable for SEC Rule 17a-4

---

### Your Task

Design the distributed transaction mechanism for TradeSpark's trade settlement process. Compare 2PC and saga explicitly, choose the appropriate mechanism, and design the full saga with compensating transactions.

---

### Deliverables

- [ ] 2PC analysis: Draw the 2PC protocol for TradeSpark's settlement. Identify the coordinator failure scenario and quantify its impact on trading latency.
- [ ] Saga design: Step-by-step saga for a single-leg trade settlement (4 steps with compensating transactions for each)
- [ ] Saga design: Extended saga for multi-leg arbitrage trade (cross-exchange, 6+ steps)
- [ ] Compensating transaction risk table: For each compensating transaction, document the financial risk (e.g., "sell 10k AAPL at market price — price risk: ±$5k depending on market movement")
- [ ] Prime broker integration: How to make the external prime broker integration idempotent and how to handle a failed compensating transaction to the prime broker
- [ ] Monitoring design: How the Risk team is alerted within 5 seconds of a failed settlement leg
- [ ] Tradeoff analysis (minimum 3):
  - 2PC vs choreography-based saga vs orchestration-based saga
  - Synchronous saga (blocking until complete) vs asynchronous saga (event-driven)
  - "Too late to compensate" scenarios — when is saga not appropriate?
