---
**Title**: TeleNova — Chapter 125: Five Country Launches in One Quarter
**Level**: Staff
**Difficulty**: 9
**Tags**: #multi-region #active-active #crdt #anycast #traffic-steering #spike-scale-10x #global-expansion
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 68 (Stratum active-active intro), Ch. 84 (GigGrid multi-region conflicts), Ch. 111 (Crestline data residency)
**Exercise Type**: Spike — Scale 10x
---

### Story Context

**Monday All-Hands — Alejandro Reyes speaking**

"I have news. Some of you have heard rumors. I want to address them directly.

We have received regulatory approval in five new countries simultaneously: Poland, Czech Republic, Romania, Portugal, and Croatia. The approvals came through on the same day — Friday. We had been expecting one or two approvals this quarter. We got five.

The combined projected subscriber count at launch: 8 million. Growing to 18 million in year one.

Launch date, per the regulatory agreements: 90 days from today.

I'm going to let that settle for a moment.

90 days. Five countries. 8 million subscribers.

The engineering team has work to do. *(looks directly at you)* Starting immediately."

---

**Whiteboard session — Monday afternoon**
**Attendees**: [you], Fatima Al-Rashid, Jin-ho Park (recovered from illness), Kenji Watanabe

You've drawn TeleNova's current region topology on the whiteboard. 9 regions: UK, Germany, France, Netherlands, Belgium, Austria, Switzerland, Sweden, Denmark. Full-mesh active-active replication.

**Jin-ho Park**: Here's the problem. Full-mesh replication means every region talks to every other region. 9 regions: 9×8/2 = 36 replication channels. We add 5 regions: 14 regions: 14×13/2 = 91 replication channels. We go from 36 to 91 replication channels. That's 2.5x the cross-region bandwidth, and it grows quadratically with every new region we add.

**Kenji Watanabe**: And the replication lag? Currently p99 inter-region sync is 180ms. At 14 regions, with 91 channels competing for bandwidth, that number goes where?

**[you]**: Up. Significantly. The current 9-region topology was designed in 2021 when we had 6 regions. We've been adding regions to a full-mesh that doesn't scale beyond about 10.

**Fatima Al-Rashid**: What's the alternative?

**[you]**: Hierarchical topology. Cluster the 14 regions into 3-4 "super-regions." Within a super-region: full-mesh sync (3-4 regions, manageable). Between super-regions: hub-spoke or ring. This reduces the number of cross-super-region channels from O(n²) to O(n).

**Jin-ho Park**: But that increases replication lag between super-regions.

**[you]**: Yes. The question is whether our SLA requires global sync within 180ms or regional sync within 180ms. Let me ask: what data types actually need global sync vs regional sync?

**Kenji Watanabe**: Subscriber presence (is this SIM active globally?) — global. Subscriber usage counters (for billing) — regional, with daily global rollup. Network routing tables — regional. Device registry — global.

**[you]**: So two of four major data types only need regional sync. We're paying for full-mesh global sync when we only need it for 50% of our data.

**Fatima Al-Rashid**: *(writing on the whiteboard)* That's the design. Regional data stays regional. Global data uses a lighter synchronization path. And the 5 new countries fit into existing super-regions geographically: Poland/Czech/Romania join the Central European cluster; Portugal/Croatia join Mediterranean. Minimal new topology changes.

**[you]**: *(nodding)* We can have this deployed in 90 days. It won't be perfect. But it'll work.

**Alejandro Reyes**: *(who has been listening from the doorway)* "It'll work" is fine. "Work" is what I need.

---

### Problem Statement

TeleNova's full-mesh active-active replication topology scales quadratically (from 36 to 91 replication channels adding 5 regions), making 5 simultaneous country launches in 90 days infeasible with the current architecture. A hierarchical regional topology must be designed that keeps cross-region bandwidth manageable and enables launch within the regulatory deadline.

---

### Explicit Requirements

1. Support 14-region topology (current 9 + 5 new) without quadratic bandwidth growth
2. Regional data (usage counters, network routing) must sync within region in ≤ 200ms
3. Global data (subscriber presence, device registry) must sync globally in ≤ 5 seconds (not 180ms — relaxed from current)
4. Launch deadline: 90 days from approval
5. No downtime for existing 9 regions during topology migration
6. New country regulatory requirements: each country's subscriber data must not leave that country (data residency from ch111 pattern)

---

### Hidden Requirements

- **Hint**: Re-read Kenji's data classification: "subscriber usage counters for billing — regional, with daily global rollup." If a subscriber travels from Germany to Poland and uses data in both countries on the same day — the billing calculation requires aggregating usage from both regional counters. What is the correct aggregation pattern for cross-regional billing consistency without requiring real-time global sync?

- **Hint**: Alejandro said "8 million subscribers at launch, growing to 18 million in year one." The super-region topology you're designing must accommodate this growth without requiring another architectural redesign. What headroom does the hierarchical topology provide?

---

### Constraints

- Current: 9 regions, 36 full-mesh replication channels
- Target: 14 regions, must not exceed ~50 replication channels
- Current inter-region sync p99: 180ms
- Acceptable global sync latency: ≤ 5 seconds (relaxed for non-critical global data)
- Launch deadline: 90 days (3 months)
- Data residency: each country's subscriber data cannot leave that country
- Migration constraint: 9 existing regions must continue operating during migration

---

### Your Task

Design the hierarchical multi-region topology for TeleNova's 14-region expansion.

---

### Deliverables

- [ ] Topology diagram: current full-mesh vs proposed hierarchical super-regions (Mermaid)
- [ ] Super-region assignments: which of the 14 regions belong to which super-region, and why
- [ ] Replication channel count: before (36) vs after (target ≤ 50) with math
- [ ] Data classification: which data types sync intra-region vs inter-super-region vs globally
- [ ] CRDT usage: which global data types use CRDT for conflict-free eventual consistency vs require coordination
- [ ] Cross-regional billing aggregation: how usage counters from multiple regions are aggregated for monthly billing without real-time global sync
- [ ] Migration plan: how to migrate existing 9 regions from full-mesh to hierarchical without downtime (phased approach)
- [ ] Launch sequencing: which of the 5 new countries to launch first, and why
- [ ] Tradeoff analysis (minimum 3):
  - Full-mesh vs hub-spoke vs hierarchical for inter-region sync — when each is appropriate
  - Tight global sync (180ms) vs relaxed global sync (5s) — what breaks at relaxed consistency?
  - Data residency enforcement at infrastructure level vs application level — reliability and auditability
