---
**Title**: Stratum Systems — Chapter 64: Distributed Consensus and Raft
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #distributed-systems #consensus #raft #leader-election #fencing-tokens
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 65, Ch. 68
**Exercise Type**: System Design / Incident Response
---

### Story Context

**#incidents — Stratum Engineering**
`09:14` **pagerduty-bot**: 🔴 P1 ALERT — customs_event_processor: event sequence validation failures > 5% threshold. 14 countries affected.
`09:15` **lars.eriksson**: what's the blast radius
`09:15` **priya.nair**: customs clearance is failing for anything touching EU ports. if this isn't fixed by 11am we hit SLA breach for 3 enterprise accounts
`09:16` **you**: just joined. reading runbook now
`09:16` **priya.nair**: runbook is useless. that's why you're here

**DM — Priya Nair → You**
`09:18` **priya.nair**: The short version: we have 4 regional DCs. Each one processes container scan events from its region. The DCs vote on "canonical" event order before publishing to the customs system. The voting is broken.
`09:19` **you**: broken how?
`09:19` **priya.nair**: two DCs are claiming to be the leader simultaneously. classic split-brain. the customs system is receiving contradictory event sequences.
`09:20` **priya.nair**: I already know the root cause. I want to see if YOU know. That's your first test here.
`09:20` **priya.nair**: hint: what did you say in your interview about fencing tokens?

The customs clearance system for Stratum's EU operations processes container scan events from 67 countries flowing into 4 regional data centers: Frankfurt, Amsterdam, Lisbon, and Warsaw. Each scan event — "container C-447821 passed through Rotterdam terminal gate 7 at 14:32:07 UTC" — must be sequenced correctly before being submitted to national customs authorities. A container that appears to have arrived before it was loaded is rejected by EU customs systems automatically.

Three weeks ago, the Warsaw DC experienced a 4-minute network partition due to a fiber cut. During that window, both Warsaw and Frankfurt believed they were the primary leader. Both published event sequences to the customs system. The customs system, receiving two conflicting sequences, began randomly accepting one and rejecting the other. The result: 847 customs declarations across 14 countries contain impossible event sequences.

DHL's operations team noticed first. Their automated clearance system started throwing `ERR_SEQUENCE_INVALID` at 09:09. By 09:14, Maersk and Hapag-Lloyd's systems reported the same. Three of Stratum's enterprise accounts are now past the 2-hour notification SLA.

The underlying problem isn't the fiber cut. Fiber cuts happen. The problem is that Stratum's leader election mechanism has no way to prevent two nodes from simultaneously believing they hold leadership authority.

---

### Problem Statement

Stratum's event ordering infrastructure uses a custom consensus mechanism that was built before the engineers understood Raft. It uses heartbeats for failure detection but lacks the term-based leadership model that prevents split-brain scenarios. When a network partition isolates a DC for more than 10 seconds, the remaining DCs can elect a new leader — but the isolated DC doesn't know it's been replaced, and continues acting as leader until it reconnects.

You need to redesign the consensus layer for Stratum's regional DC cluster to prevent split-brain scenarios and ensure that event sequence decisions are always made by a single authoritative leader, even during network partitions.

---

### Explicit Requirements

1. Exactly one DC must act as event sequencing leader at any time
2. Leader elections must complete within 5 seconds of failure detection
3. Previously-elected leaders must be unable to publish events after a new leader is elected (even if they believe they're still leader)
4. The system must tolerate any single DC going offline without losing sequencing capability
5. Event sequence decisions must be durable — a decision that's been acknowledged must survive any single node failure
6. All 4 DCs must be able to serve read queries (event history) regardless of leader status

---

### Hidden Requirements

- **Hint**: Re-read Priya's DM. She mentions "3 enterprise accounts hitting SLA breach." Re-read the incident log timestamps. The customs authorities in different countries have different submission windows. What does that imply about the sequencing system's latency requirements beyond just "correctness"?

- **Hint**: The runbook mentions "Q3 weather incidents — typhoon season partition protocols." What does Stratum's infrastructure in Southeast Asia need that a European 4-DC cluster doesn't? What happens to consensus when you have 6 DCs instead of 4, with variable network latency between them?

---

### Constraints

- 4 regional DCs (Frankfurt, Amsterdam, Lisbon, Warsaw) — expanding to 14 in Chapter 68
- 18M shipment events/day across all DCs = ~208 events/second average, 850/sec peak
- Event sequencing latency must remain under 200ms P99 (customs systems reject submissions older than the sequence window)
- Maximum 3 engineers can be dedicated to this work (rest are on other product teams)
- Existing event store is Kafka — the consensus layer sits above Kafka
- Network latency between EU DCs: 8–35ms; Southeast Asia: 80–180ms

---

### Your Task

Design a Raft-based consensus layer for Stratum's regional DC cluster. Focus on leader election safety, fencing token generation, and the protocol for preventing previously-elected leaders from publishing stale decisions.

---

### Deliverables

- [ ] Mermaid diagram: Raft state machine (Follower → Candidate → Leader transitions)
- [ ] Mermaid diagram: Fencing token flow (how a stale leader is rejected by the event store)
- [ ] Database/state schema: What each Raft node persists (current_term, voted_for, log entries)
- [ ] Scaling estimation: At 850 events/sec peak, what's the throughput of a single-leader Raft cluster with 4 nodes? Show the math for replication lag.
- [ ] Tradeoff analysis (minimum 3):
  - Single-leader sequencing vs multi-leader with conflict resolution
  - Raft vs Paxos vs ZAB for this use case
  - Fencing tokens vs epoch-based leader validity checks
- [ ] TypeScript interface sketch: `RaftNode` with methods for `requestVote()`, `appendEntries()`, and `publishWithFencingToken()`
