---
**Title**: Checkpoint 3 — Senior → Strong Senior: Apex Financial Systems
**Level**: Staff
**Difficulty**: 10
**Tags**: #checkpoint #system-design-interview #distributed-systems #real-time #consensus
**Estimated Time**: 4 hours
**Exercise Type**: Promotion Checkpoint
---

### Story Context

**LinkedIn Message — Apex Financial Systems Recruiter**

> Hi — I'm a technical recruiter at Apex Financial Systems. We're the global risk management platform that tier-1 banks use for real-time counterparty risk calculation. We've been watching your work closely — the GigGrid CRDT design was shared in our engineering slack by someone who knows your network.
>
> We're hiring a Principal Systems Architect for a project that I can't describe in detail until we're further along in the process. The technical brief is attached. If it looks interesting, let me know.

**Attached**: `apex_principal_architect_brief.pdf`

---

**Apex Financial — Technical Brief (4 sentences)**

Apex Financial Systems processes real-time market data and computes risk exposure for 200 global counterparties. The system must handle unpredictable volume spikes and maintain consistency under load. Sub-100ms P99 risk scores are required for regulatory reporting. The system operates in an adversarial clock environment.

That's it. Four sentences.

---

**On-Site Interview — Apex Financial Systems, London**

*Conference Room 3C. You arrive early. Three engineers are already seated.*

**Dr. Soo-Jin Park** *(Principal Architect)*: I'm Soo-Jin. This is Omar Hassan, VP Engineering. And this is Priya Nair.

You look up. Priya Nair. Stratum Systems. She's been at Apex Financial for 18 months.

**Priya**: *(smiling slightly)* I recommended you. Don't make me regret it.

**Dr. Park**: We have four hours. The brief was intentionally sparse. You know why?

**You**: Because the right questions are as important as the right answers.

**Dr. Park**: Correct. Let's begin. Design the Apex risk calculation engine.

---

**Clarifying Questions Session** *(first 20 minutes)*

**You**: I have questions before I draw anything.

**Omar**: Good. Ask.

**You**: First: "sub-100ms P99 risk scores." What's the user of that score? Is it a human analyst reading a dashboard, or is it an automated system making trading decisions?

**Dr. Park**: Automated system. Basel III regulatory requirement: risk scores must update within 100ms of a market event that changes a counterparty's exposure.

**You**: Second: "200 global counterparties." Are these static — always the same 200 — or dynamic, with new counterparties added daily?

**Omar**: Semi-static. Maybe 2-3 new counterparties per week. But each existing counterparty can have thousands of individual positions.

**You**: Third: "500k market events/sec." Is that the peak or the average?

**Dr. Park**: Peak. At market open and during volatility events. Average is ~50k/sec.

**You**: Fourth: "adversarial clock environment." You mean clock skew between market data feeds?

**Priya**: *(leaning forward)* Different market data vendors have different clock precision. Some are nanosecond-accurate. Some are millisecond-accurate with 200ms skew. And the clock skew isn't stable — it drifts under load.

**You**: *(making a note)* That changes the event ordering design significantly. Are market events causally ordered — does event A cause event B — or are they independent parallel events from different markets?

**Dr. Park**: Both. An equity price change on NYSE can trigger a derivative position change on CBOE. The risk engine must compute the net exposure correctly even when the causally-dependent events arrive out of order.

**You**: Fifth question — probably the most important: multi-tenant or single-tenant? Are all 200 counterparties sharing one risk engine, or is each bank running their own instance?

**Omar**: Multi-tenant. The 200 counterparties are the banks that use our platform. They're separate tenants. Tenant isolation is a hard requirement — one bank cannot see another bank's risk data.

**You**: That's actually six requirements I'm going to design around. I need 2 minutes to sketch the architecture.

---

**Design Session** *(60 minutes)*

[The interview continues for 60 minutes. You design, they challenge, you defend or redesign.]

**Key design decisions you must justify:**

1. **Event ordering**: You propose Lamport timestamps at the ingestion layer with a causal ordering buffer. Priya challenges: "What's the buffer window? If you wait too long for out-of-order events, you miss the 100ms SLA. If you don't wait long enough, you miss causally-dependent events."

2. **Risk calculation consistency**: 500k events/sec fan out to 200 counterparty risk calculators. Omar challenges: "What happens when two events for the same counterparty arrive 5ms apart? Do you process them sequentially (correct but slow) or in parallel (fast but potentially wrong)?"

3. **Clock skew handling**: Dr. Park asks: "You mentioned clock skew in your Stratum work. How does your event ordering handle a market data feed that's 200ms behind? If a NYSE event at T=0 and a CBOE event at T=+5ms (but received 200ms later due to feed delay) — do you risk computing an incorrect net exposure during the 200ms gap?"

4. **Tenant isolation at risk calculation layer**: You propose a tenant-scoped event partition. Priya challenges: "But market events affect multiple tenants. An AAPL price drop affects every bank that has AAPL exposure. You can't partition market events by tenant — they're shared. You can only partition risk calculations by tenant."

5. **Regulatory audit trail**: Dr. Park: "Basel III requires us to prove, retroactively, what the risk score was at 14:32:07.001 on March 3rd for a specific counterparty. Your system is stateless — you compute risk from market events. How do you reconstruct historical state?"

---

### Problem Statement

Design the Apex Financial Systems real-time risk calculation engine. Requirements: 500k market events/sec peak, 200 global counterparties (multi-tenant), sub-100ms P99 risk scores, clock skew handling for adversarial feed environments, and a complete audit trail for retroactive risk score reconstruction.

---

### Explicit Requirements

1. Process up to 500k market events/sec at peak
2. Compute risk scores for 200 counterparties within 100ms P99 of the triggering market event
3. Handle causally-dependent events that arrive out of order due to clock skew
4. Tenant isolation: no cross-tenant risk data visibility
5. Regulatory audit trail: retroactive reconstruction of any counterparty's risk score at any historical timestamp
6. System must handle 2-3 new counterparty onboardings per week without architectural changes

---

### Hidden Requirements

- **Hint**: Priya's challenge about the buffer window is a hint about the real requirement. Re-read Dr. Park's answer: "causally-dependent events" can arrive up to 200ms out of order. How does this interact with the 100ms SLA? If a causally-dependent event arrives 200ms late, the "correct" risk score requires waiting for it — but your SLA says 100ms. This is an irresolvable conflict. How do you design a system that produces a "best available" risk score within 100ms and a "confirmed" score when causally-dependent events arrive?

- **Hint**: Omar said "each existing counterparty can have thousands of individual positions." At 500k events/sec, each event may affect multiple positions across multiple counterparties. If an AAPL price drop at 09:30:00 affects 80 counterparties each with 400 AAPL positions — that's 32,000 position recalculations from a single event. At 500k events/sec, what's the total calculation throughput required? Show the math.

---

### Constraints

- 500k events/sec peak, 50k/sec average
- 200 counterparties, each with up to 5,000 positions
- Sub-100ms P99 risk score update
- Clock skew: up to ±200ms across market data feeds
- Multi-tenant: 200 tenants (banks)
- Regulatory: Basel III, MiFID II (EU)
- Budget: can use up to 200 compute nodes

---

### Your Task

Design the complete Apex risk calculation engine. This is a Staff-level interview. You are expected to handle ambiguity, ask clarifying questions, defend tradeoffs under pressure, and model the math.

---

### Deliverables

- [ ] Clarifying questions: List the questions you would ask before designing (minimum 6, with justification for each)
- [ ] Mermaid architecture diagram: Full system with event ingestion, ordering, distribution, risk calculation, tenant isolation, and audit trail
- [ ] Event ordering design: Lamport timestamp + causal buffer design, with explicit buffer window calculation
- [ ] Risk calculation design: How 500k events/sec fan out to 200 counterparty calculators — show the math
- [ ] "Best available" vs "confirmed" risk score: Design the two-tier scoring model for handling out-of-order events
- [ ] Tenant isolation: How market events (shared) are separated from risk calculations (tenant-scoped)
- [ ] Audit trail: How to reconstruct historical risk scores from the event log
- [ ] Scaling math: At 500k events/sec × avg 10 affected counterparties per event × 400 positions per counterparty = ? calculations/sec. Can 200 nodes handle this? Show the work.
- [ ] Tradeoff defense: Pick your three most controversial design choices and defend them against a skeptical panel
