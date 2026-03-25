---
**Title**: Stratum Systems — Chapter 68: Multi-Region Active-Active Architecture
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #multi-region #active-active #distributed-systems #data-residency #conflict-resolution
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 64, Ch. 84, Ch. 125
**Exercise Type**: System Design / Scale 10x
---

### Story Context

**Slack DM — Marcus Webb → You**
`Wednesday 08:15`
**marcus.webb**: heard you're working on the Stratum DC expansion. 4 to 14.
**you**: news travels fast. how do you know Priya?
**marcus.webb**: I've known Priya since 2009. I taught a workshop at her grad school. she's one of the three people on earth I trust with a distributed systems problem.
**marcus.webb**: she told me you're about to make a classic mistake.
**you**: what mistake?
**marcus.webb**: I have three questions before I tell you. answer them first.
**marcus.webb**: 1. What's the difference between active-active and active-passive in terms of what gets replicated where?
**marcus.webb**: 2. What happens when two DCs in your active-active cluster simultaneously accept a write to the same shipment record with conflicting state?
**marcus.webb**: 3. Who owns conflict resolution — the application layer or the storage layer?
**you**: 1. active-active: all nodes accept writes and reads. active-passive: one primary accepts writes, replicas serve reads. 2. you get a conflict — two versions of the same record that need to be merged or one needs to win. 3. depends on whether your storage supports CRDTs or LWW natively.
**marcus.webb**: good. now I'll tell you the mistake.
**marcus.webb**: you're going to want to make all 14 DCs equal peers because it feels architecturally clean. don't. last-write-wins is fine for some of your data. it will destroy you for the rest.
**marcus.webb**: I've seen this exact design kill a $200M company. not the network partition. the conflict resolution policy.

---

**All-Hands Engineering — Stratum Systems**
`Thursday 09:00`

CEO Klaus Reinholt opens the meeting: "I have good news and I have news that will keep this team busy for the next 72 hours. A strategic contract with the Asian Infrastructure Investment Bank requires us to process freight events from 10 additional countries in Southeast and Central Asia. We go live in 90 days. We need to expand from 4 to 14 regional data centers immediately."

VP Engineering Anton Keller: "To be specific: we need 10 new DCs operational within 90 days. The contract specifies that event processing must occur within the country of origin for regulatory reasons in Vietnam, India, Indonesia, Malaysia, Thailand, Kazakhstan, and UAE. That's not a preference — it's in the contract."

*The room goes quiet.*

Priya, next to you, writes on her notepad and slides it over: *"Data residency requirement. Active-active is now a regulatory constraint, not just a performance choice. We have to get this right."*

**#engineering-planning**
`09:47` **lars.eriksson**: 10 new DCs in 90 days. are we building or buying?
`09:48` **priya.nair**: combination. AWS regions for most. we already have VMs in ap-southeast-1, ap-south-1. we need to add me-central-1, kz-astana (private DC), and 6 others.
`09:49` **you**: the latency between Kazakhstan and Germany is going to be 180-200ms. that changes the consensus math.
`09:50` **priya.nair**: @you — that's exactly right. which is why your Raft design from chapter 64 doesn't scale to 14 nodes globally. what's your plan?
`09:51` **marcus.webb** *(just joined the channel)*: @you — remember my third question. who owns conflict resolution?

---

### Problem Statement

Stratum must expand from 4 EU data centers to 14 global data centers in 90 days, driven by a regulatory requirement that shipment events be processed within the country of origin for 7 specific jurisdictions. The new architecture must handle active-active replication across 14 DCs with widely varying latency (8ms within EU; up to 200ms for Central Asia), while maintaining event ordering guarantees and resolving write conflicts correctly.

---

### Explicit Requirements

1. All 14 DCs can accept writes from local partner APIs (active-active)
2. 7 DCs must process and store events locally before replicating (data residency)
3. Replication lag across all DCs must be < 5 seconds under normal conditions
4. Write conflicts (two DCs accepting conflicting state updates for the same shipment) must be resolved deterministically
5. The solution must tolerate any single DC going offline without affecting the remaining 13
6. Global event ordering (cross-DC sequencing) must still satisfy the Lamport timestamp guarantees from Chapter 65

---

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's DM: "last-write-wins is fine for some of your data. it will destroy you for the rest." Your shipment data has at least two categories. What are they, and why does LWW work for one but not the other?

- **Hint**: The contract specifies regulatory data residency. The CEO mentioned "Asian Infrastructure Investment Bank." International finance contracts often include audit trail requirements. What does that imply about the replication of the *compliance history* (not the current-state projection)?

---

### Constraints

- 14 DCs across EU (4 existing), SE Asia (4 new), South Asia (2 new), Central Asia (2 new), Middle East (2 new)
- EU-to-EU latency: 8–35ms; EU-to-SE Asia: 120–160ms; EU-to-Central Asia: 180–200ms
- 18M events/day now; projected 40M/day with new regions
- 90-day launch deadline
- 7 DCs have mandatory local-first processing (data residency)
- Existing infrastructure: Kafka per DC (cannot be replaced), Postgres per DC for projections
- Team: 5 engineers for this expansion

---

### Your Task

Design the active-active multi-region architecture for 14 DCs. Define the replication topology, conflict resolution strategy per data type, and the consistency guarantees each DC can provide.

---

### Deliverables

- [ ] Mermaid diagram: 14-DC topology with replication topology overlay (hub-and-spoke vs mesh vs hierarchical)
- [ ] Conflict resolution matrix: For each data type (current shipment state, compliance event log, projection metadata), specify the conflict resolution strategy and justify it
- [ ] Replication lag model: Show the math for worst-case replication lag with 200ms EU→Central Asia RTT and 40M events/day
- [ ] Data residency enforcement: Architecture for ensuring data in Vietnam DC is processed locally before replication
- [ ] Scaling estimation: At 40M events/day across 14 DCs, what's the per-DC event volume and storage requirement? Show the math.
- [ ] Tradeoff analysis (minimum 3):
  - Active-active vs active-passive for high-latency DCs (Central Asia, Middle East)
  - LWW vs application-level merge for shipment current-state conflicts
  - Full mesh replication vs hub-and-spoke for 14 DCs
- [ ] Migration plan: How do you add 10 DCs without downtime to the existing 4-DC system?
