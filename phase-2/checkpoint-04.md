---
**Title**: Checkpoint 4 — Strong Senior Interview @ Meridian Exchange
**Level**: Strong Senior
**Difficulty**: 9
**Tags**: #promotion-checkpoint #market-data #real-time #fanout #distributed-systems #compliance #multi-region
**Estimated Time**: 4 hours
**Exercise Type**: Promotion Checkpoint
---

### The Night Before

**Slack DM — Marcus Webb → [you] — Sunday 22:14**

You almost don't see it. You're reviewing your notes on market data systems — tick-by-tick dissemination, competing multicast vs unicast architectures, OPRA vs proprietary protocols.

The notification appears. Marcus Webb. You haven't heard from him since VertexCloud.

**Marcus Webb**: I heard you're at Meridian tomorrow.

A pause. Then:

**Marcus Webb**: Design review or interview?

**[you]**: Both, I think.

**Marcus Webb**: Good. Those are the honest ones. They know what they need. They want to see if you know it too.

Another pause. Longer this time. When he types again, it's different from his usual style. No challenge. No war story.

**Marcus Webb**: I've watched you work through 14 companies. You've gotten to the point where you don't need me to tell you what you missed. You already see it before the postmortem.

**Marcus Webb**: One piece of advice for tomorrow: when you don't know something, say so and reason through it out loud. That's what Staff engineers do. They don't pretend. They think in public.

**Marcus Webb**: Good luck. You won't need it.

He goes offline. You stare at the message for a moment.

You close your notes.

---

### The Interview

**Meridian Exchange — Monday 09:00**

The room has four people you've never met:

- **Isabela Ferreira** — Principal Engineer, Distribution Systems (20+ years in financial data)
- **Tariq Osei** — Staff Engineer, Infrastructure (infrastructure background, cost-obsessed)
- **Dr. Renata Kowalski** — Head of Compliance and Regulatory Affairs (financial regulation, FINRA, MiFID II)
- **James Whitfield** — VP of Engineering (your potential manager, strategic, mostly listens)

Isabela opens a blank whiteboard and sets a timer for 5 minutes.

**Isabela Ferreira**: "Before we begin — 5 minutes to write down any clarifying questions you'd want to ask about the problem. Don't start designing. Just ask questions."

The problem statement she's given you:

> *Design Meridian Exchange's next-generation market data distribution system.*
> *Current scale: 50 million events/second peak, growing to 150M by Q4.*
> *10,000 institutional subscribers across 200 countries.*
> *Latency SLA: sub-millisecond fanout to top-tier subscribers.*
> *The system must support regional failover with no subscriber disruption.*

That's all you have.

---

### Problem Statement

Meridian Exchange distributes real-time market data — stock ticks, order book updates, trade confirmations — to 10,000 institutional subscribers (investment banks, hedge funds, sovereign wealth funds, algorithmic trading firms). The current system was designed for 5M events/second and is now handling 50M at peak. A Singapore sovereign wealth fund client requires regional failover guarantees that the current architecture cannot provide. Design the next-generation distribution system.

---

### Clarifying Questions You Should Ask

The panel will evaluate the quality of your questions before your design. These are the questions a Strong Senior / Staff candidate asks:

1. **What is the event shape?** Are all 50M/second events the same size? Tick data is typically 64-200 bytes. Full order book snapshots can be kilobytes. The answer affects bandwidth math.

2. **What does "sub-millisecond fanout" mean precisely?** Is this 99th percentile? 99.9th? Is it measured from event ingestion to subscriber receipt, or from exchange feed to subscriber receipt? The SLA contract matters.

3. **Are all 10,000 subscribers equal?** Tier 1 subscribers (investment banks, sovereign wealth funds) likely have stricter latency SLAs than Tier 3 (retail API consumers). The architecture for each tier may differ.

4. **What is the subscriber subscription model?** Do subscribers subscribe to all symbols, specific symbols, specific exchanges? Fan-out factor depends on this. If 10,000 subscribers each want 500 symbols from 3,000 available, the math is different from full-feed subscribers.

5. **What is the regulatory obligation?** MiFID II requires a Consolidated Tape for European subscribers. FINRA has NMS obligations in the US. Does the system need to produce a regulatory audit trail of what data was delivered to whom, at what timestamp?

6. **What does "no subscriber disruption" mean for failover?** Does the subscriber need to not drop a single event? Or is "no disruption" defined as connection reestablishment within N seconds with a gap-fill mechanism?

7. **Where are the 10,000 subscribers located?** If they're geographically distributed, the sub-millisecond SLA implies edge infrastructure, not centralized.

8. **What are the current bottlenecks?** Is the current system failing on ingestion, fan-out, or last-mile delivery? Understanding where it's breaking shapes the redesign.

---

### Explicit Requirements

1. Ingest 50M events/second today, design for 150M/second within 12 months
2. Sub-millisecond P99 fanout latency for Tier 1 subscribers (top 500 firms by contract value)
3. < 10ms P99 fanout latency for Tier 2 subscribers (next 2,000 firms)
4. < 100ms P99 for Tier 3 subscribers (remaining 7,500)
5. Regional failover: Tier 1 subscribers must receive uninterrupted data flow on regional failure (gap-fill acceptable within 200ms)
6. Regulatory audit trail: every event delivered to every subscriber must be logged with nanosecond timestamp for FINRA/MiFID II compliance
7. Subscription granularity: per-symbol, per-exchange, per-asset-class filtering
8. Back-pressure handling: slow subscribers cannot affect fast subscribers

---

### Hidden Requirements

- **Hint**: Dr. Renata Kowalski is in the room. She hasn't spoken yet, but she will. MiFID II's RTS 17 requires that market data be made available on a "reasonable commercial basis" — but the relevant compliance detail is this: consolidated tape providers must timestamp market data at the point of receipt and at the point of dissemination. Two timestamps. If your architecture loses one of them (e.g., by timestamping only at dissemination), you fail the audit. Your audit trail design must capture both.

- **Hint**: Tariq Osei is cost-obsessed. He'll ask about the cost of your design. A naive implementation of sub-millisecond fanout to 10,000 subscribers requires dedicated hardware or co-location. The hidden question is: what is the cost of the P99 latency guarantee at scale, and is there a tiered architecture that meets the SLA for each tier without paying Tier 1 costs for Tier 3 subscribers?

- **Hint**: "No subscriber disruption" has a trap. You can design a perfect failover system that routes around regional failures, but if you don't design for *event ordering* across the failover boundary, algorithmic trading subscribers will receive out-of-order ticks. An out-of-order tick that updates an order book in the wrong sequence is a compliance and financial risk event. Your failover design must include sequence number assignment and gap-fill with ordering guarantees.

---

### Constraints

- Current peak: 50M events/second; 12-month target: 150M events/second
- 10,000 subscribers; top 500 (Tier 1) have contractual sub-millisecond SLAs
- Event size: P50 = 128 bytes (tick data), P99 = 8KB (order book snapshot)
- Bandwidth: 50M × 128 bytes = 6.4 GB/s ingestion; 10,000 subscribers × 6.4 GB/s fanout = 64 TB/s if full-feed (impractical — subscription filtering required)
- Regulatory retention: 7 years for FINRA, 5 years for MiFID II (keep 7 years)
- Current architecture: single-region, unicast TCP to all subscribers, single fan-out tier
- Budget: Tariq will ask. Estimate required.

---

### Your Task

Design Meridian Exchange's next-generation market data distribution system. You have a whiteboard, a panel of four, and no prepared slides.

---

### Deliverables

- [ ] Clarifying questions — list the 5 most important questions before designing
- [ ] Architecture layers: ingestion tier, fan-out tier, last-mile delivery tier, audit tier
- [ ] Tiered subscriber model: Tier 1 (co-location / kernel-bypass), Tier 2 (regional PoP), Tier 3 (cloud delivery)
- [ ] Fan-out math: bandwidth reduction via subscription filtering (show your calculation)
- [ ] Sequence number assignment: how events get globally ordered sequence numbers for gap-fill
- [ ] Regional failover design: active-passive vs active-active, failover trigger, gap-fill protocol
- [ ] Regulatory audit trail: dual-timestamp architecture (receipt + dissemination), 7-year retention design
- [ ] Back-pressure isolation: how slow Tier 3 subscribers cannot affect Tier 1 latency
- [ ] Cost model: estimate infrastructure cost for Tier 1 co-location, Tier 2 PoPs, Tier 3 cloud (approximate)
- [ ] Tradeoff analysis (minimum 3):
  - Multicast vs unicast for Tier 1 — bandwidth efficiency vs subscriber isolation and filtering granularity
  - Kernel-bypass networking (DPDK / RDMA) vs standard TCP for sub-millisecond latency — latency vs operational complexity
  - Active-active vs active-passive multi-region — zero-gap failover vs cost and conflict resolution complexity
  - Centralized sequence assignment vs distributed (per-exchange) sequence numbering — global ordering vs horizontal scale

---

### Panel Behavior

The panel will interrupt you. Plan for it.

**Isabela Ferreira** will ask about ordering guarantees at the failover boundary — "if Region A fails mid-sequence and Region B takes over, what sequence number does Region B start from?"

**Tariq Osei** will ask for cost estimates at the 150M events/second scale — "what does it cost to deliver sub-millisecond to 500 firms across 200 countries?"

**Dr. Renata Kowalski** will ask about the audit trail gap — "what happens if an event is received but your audit logger is down? Do we lose the receipt timestamp?"

**James Whitfield** will say almost nothing for 90 minutes. In the final 10 minutes, he will ask one question: "If I gave you this project and unlimited budget, what would you NOT build? What's the thing you'd explicitly decide not to do?"

That last question is the most important one.

---

### What Passing Looks Like

A passing candidate at Strong Senior level:
- Asks clarifying questions before drawing anything
- Builds a tiered architecture that explicitly serves different SLAs with different infrastructure
- Identifies the ordering problem at failover without being prompted
- Has a coherent answer (not perfect, coherent) for the regulatory audit trail
- Gives a rough cost estimate when asked rather than deflecting

A passing candidate at Staff level additionally:
- Identifies the dual-timestamp compliance requirement unprompted
- Explicitly discusses what they'd deprioritize and why
- Handles panel interruptions without losing the thread of the design
- Answers Whitfield's final question with a specific, justified "we would not build X because Y"
