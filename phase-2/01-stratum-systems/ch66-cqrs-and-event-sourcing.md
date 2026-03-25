---
**Title**: Stratum Systems — Chapter 66: CQRS and Event Sourcing
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #cqrs #event-sourcing #projections #distributed-systems #eventual-consistency
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 65, Ch. 67, Ch. 104, Ch. 112
**Exercise Type**: System Design / System Evolution
---

### Story Context

**1:1 Session Transcript — You and Priya Nair**
*Stratum Systems, Hamburg Office, Conference Room B*
*Two weeks after the Lamport timestamp fix shipped*

**Priya**: The ordering fix is working. Consumer lag is down. Customs rejections are at 0.03% — that's within acceptable range. Well done.

**You**: Thank you. But I'm noticing something. The read side can't keep up anymore. We fixed write correctness, but now the query service is timing out under read load. The customs integration team is getting 504s when they try to fetch shipment history.

**Priya**: Define "can't keep up."

**You**: The shipment_states table is getting hit by both writes and reads on the same Postgres instance. During peak hours — 08:00 to 11:00 UTC when Asian shipments land in European ports simultaneously — read latency is hitting 4 seconds. The customs SLA requires sub-500ms.

**Priya**: What does Yemi want?

**You**: That's the other thing. VP of Product [Yemi Okafor] sent me a requirements doc this morning. He wants two things: real-time current shipment location for the customer portal — sub-second, highly available — AND a full 5-year immutable audit history for compliance. "Full" means every state transition, every timestamp, the complete provenance chain.

**Priya**: Those are two different query patterns.

**You**: Completely different. One is "where is container C-447821 right now?" — current state, high read volume, tolerance for 2-second staleness. The other is "show me every event for container C-447821 since January 2020" — full history, low read volume, must be immutable, needs to survive GDPR scrutiny and customs audits.

**Priya**: You already know what the solution is.

**You**: CQRS. Command side writes events to the log. Query side builds projections optimized for each read pattern.

**Priya**: *(nods)* Tell me where you'd put the projection rebuild logic. And tell me what happens to the current-location projection when a Lamport timestamp correction causes event reordering.

**You**: *(pause)* That's a good question.

**Priya**: That's why I asked it.

---

**Slack DM — Yemi Okafor → You**
`Thursday 16:22`
**yemi.okafor**: hey — quick question for the compliance team. they're asking: if we source the audit history from the same event log as the customer portal, what's the consistency guarantee? can a customer see their shipment as "delivered" before the compliance system records it as delivered?

**you**: that depends on the projection update lag. right now, yes — there's a window of eventual consistency.

**yemi.okafor**: how long a window?

**you**: up to 90 seconds under current load

**yemi.okafor**: the EU customs directive requires that the compliance record and the customer-visible record are consistent within 30 seconds. did anyone tell you that?

**you**: ...no

**yemi.okafor**: it was in appendix C of the legal update. the 34-page one.

**you**: I really need to read that document.

**yemi.okafor**: yes you do

---

### Problem Statement

Stratum's shipment state system uses a single Postgres table as both the write target and the read source. This creates a read/write contention problem at scale. More importantly, the system serves two fundamentally different read patterns — real-time current state queries and long-form compliance history queries — with the same data model, leading to poor performance for both.

You need to redesign the shipment state system using CQRS (Command Query Responsibility Segregation) and event sourcing to separate the write path from the read paths and optimize each independently.

---

### Explicit Requirements

1. Write path: all shipment state changes are recorded as immutable events in an append-only log
2. Read path 1 (Current Location): current shipment state available in < 500ms, tolerates up to 30-second staleness
3. Read path 2 (Compliance History): full 5-year event history, immutable, with complete provenance chain
4. Projection consistency window: current-state and compliance record must converge within 30 seconds
5. If a Lamport timestamp correction causes event reordering, projections must be re-computable from the event log
6. The event log must be the single source of truth — projections are derived, never primary

---

### Hidden Requirements

- **Hint**: Re-read Priya's question about "what happens when a Lamport correction causes reordering." The projection rebuild problem is harder than it sounds. What's the maximum window in which a late-arriving event can change the "final" state of a closed shipment (one that's been delivered)? How does this affect projection design?

- **Hint**: Yemi mentioned "GDPR scrutiny." If a customer invokes right to erasure (Chapter 70 will cover this in detail), you need to be able to delete their data from the event log. But the event log is "immutable." How does your event sourcing design accommodate this future requirement?

---

### Constraints

- 18M events/day = ~208/sec avg write throughput
- Peak read volume: 15,000 current-state queries/minute during peak hours (08:00–11:00 UTC)
- Compliance history: ~500 queries/day, mostly range scans over years of data
- 5-year event retention required for EU customs compliance
- Current-state projection must support 30-second SLA consistency guarantee
- Team size: 3 engineers for this work

---

### Your Task

Design the CQRS architecture for Stratum's shipment state system. Define the event schema, the command side (write path), and both query-side projections (current location and compliance history). Explain the projection update mechanism and consistency guarantees.

---

### Deliverables

- [ ] Mermaid diagram: CQRS architecture (command side → event log → projection consumers → read models)
- [ ] Event schema: `ShipmentEvent` with all fields (event_id, type, shipment_id, payload, lamport_ts, received_at, source_dc)
- [ ] Projection schema: Current Location read model (optimized for sub-500ms single-shipment lookup)
- [ ] Projection schema: Compliance History read model (optimized for range scans and audit export)
- [ ] Scaling estimation: At 208 events/sec, what's the projection consumer lag under normal load? What's maximum lag during peak (850 events/sec)?
- [ ] Tradeoff analysis (minimum 3):
  - Eventual consistency in projections vs synchronous projection updates
  - Single event log vs partitioned event log per shipping region
  - Snapshot optimization (when to snapshot vs always replay from origin)
- [ ] Projection rebuild algorithm: Step-by-step procedure for rebuilding the current-location projection after a Lamport correction causes historical reordering
