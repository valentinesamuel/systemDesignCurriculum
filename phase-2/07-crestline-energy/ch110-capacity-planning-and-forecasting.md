---
**Title**: Crestline Energy — Chapter 110: Three Years of Growth, Two Years of Budget
**Level**: Staff
**Difficulty**: 8
**Tags**: #capacity-planning #cost-modeling #finops #forecasting #reserved-instances #spot-instances #infrastructure
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 109 (SLO design), Ch. 146 (VertexCloud FinOps), Ch. 56 (LuminaryAI billing)
**Exercise Type**: System Design
---

### Story Context

**Calendar invite — Thursday 14:00**
**Subject**: Infrastructure Cost Review — Q3
**Organizer**: Elena Vasquez
**Attendees**: Tobias Klein (CFO), [you], Dr. Marcus Thompson, Amara Diallo

You accept the invite and spend 20 minutes pulling the numbers before the meeting.

Current monthly cloud spend: €680,000. Three years ago: €200,000. Revenue growth over the same period: 2.4x. Infrastructure cost growth: 3.4x. Infrastructure as a percentage of revenue: climbing.

---

**Meeting Room — Thursday 14:00**

Tobias Klein opens a spreadsheet on the projector. He is precise, unhurried, and has clearly prepared. He does not say hello. He shows the infrastructure cost trend line and says: "This is unsustainable. Infrastructure costs are growing 1.4x faster than revenue. At current trajectory, we cross 30% of revenue in 18 months. The board will not accept that."

**Elena Vasquez**: *(to you)* That's why you're here.

**Tobias Klein**: I need three things. One: what does the next three years of infrastructure cost look like if we do nothing? Two: what levers do we have to reduce or control cost without degrading reliability? Three: what does "right-sized" infrastructure look like for our expected growth?

**[you]**: What's the expected growth model?

**Tobias Klein**: Conservative: 15% meter growth per year, 20% query volume growth (analytics usage is growing faster than meters). Aggressive: 25% meter growth if we win the Nordic Grid contract.

**[you]**: And the reliability floor?

**Elena Vasquez**: 99.9% SLA is contractual. We can't go below it.

**[you]**: I'll need a week to model this properly.

**Tobias Klein**: *(closing his laptop)* I need the analysis before the board meeting on the 20th. That gives you 8 days.

---

**Slack DM — Amara Diallo → [you] — Friday 09:15**

**Amara Diallo**: I pulled the utilization metrics you asked for. Here's the summary:

- Kafka brokers: average CPU 23%, peak 71%. Currently 24 brokers, m5.2xlarge.
- TimescaleDB primary: average CPU 67%, peak 94%. This is too high.
- Iceberg compaction nodes: run only during off-peak (02:00-06:00). Average utilization during compaction window: 89%.
- Flink processing nodes: average CPU 38%, peak 82%.
- Analytics query nodes (Presto): average CPU 12%, peak 48%. These are severely over-provisioned.

**[you]**: What's the on-demand vs reserved split?

**Amara Diallo**: 100% on-demand. Everything.

**[you]**: Everything?

**Amara Diallo**: Sanjay set it up that way in 2021 because "we weren't sure about growth." Three years later, nobody switched it.

**[you]**: That's €180,000 in annual waste right there. Reserved instances for baseline load would save 40% on that cost.

**Amara Diallo**: Nobody knew what "baseline load" looked like. Now we have 3 years of data.

---

**From your analysis notes:**

Conservative 3-year infrastructure forecast (do nothing):
- Year 1: €816,000/month (+20%)
- Year 2: €979,000/month (+20%)
- Year 3: €1.17M/month (+20%)
- 3-year total: ~€35M

Optimized forecast (reserved instances + spot for batch + rightsizing):
- Year 1: €510,000/month (-25% from current)
- Year 2: €612,000/month (+20% growth, but from optimized baseline)
- Year 3: €734,000/month (+20%)
- 3-year total: ~€22M

Savings potential: €13M over 3 years.

Tobias Klein is going to like this.

---

### Problem Statement

Crestline Energy's infrastructure costs are growing 1.4x faster than revenue. All infrastructure runs on-demand with no reserved instance commitments and poor utilization rates. A 3-year capacity plan must project growth, identify optimization levers, and present a cost model that reduces infrastructure as a percentage of revenue while maintaining 99.9% SLA.

---

### Explicit Requirements

1. 3-year capacity plan for two growth scenarios (15% vs 25% meter growth/year)
2. Cost model must break down by workload type (ingestion, storage, analytics, Flink processing)
3. Reserved instance strategy: identify baseline vs variable load for each workload
4. Spot instance strategy: identify which workloads are safe for spot (batch, compaction) vs unsafe (ingestion, alerting)
5. Rightsizing recommendations based on utilization data
6. Cost per meter read as a unit economic metric (current and projected)

---

### Hidden Requirements

- **Hint**: The Presto analytics nodes show 12% average CPU but 48% peak. This pattern (low average, high peak) has specific implications for reserved vs spot strategy. What's the right infrastructure model for workloads with high peak-to-average ratios?

- **Hint**: Tobias Klein asked about "right-sized infrastructure for expected growth." The TimescaleDB primary at 67% average CPU is a risk: at 20% growth, it hits 80% average and 100%+ peak within 6 months. A capacity plan that only shows cost without flagging this time-bomb is incomplete.

---

### Constraints

- Current spend: €680,000/month (100% on-demand)
- Target: reduce infrastructure as % of revenue from current to ≤ 20% over 3 years
- Growth models: conservative (15%/year meters, 20%/year queries), aggressive (25%/year meters)
- Board presentation: 8 days, slide format, cost model in €/month
- Reliability floor: 99.9% SLA non-negotiable
- Spot instance risk tolerance: zero for ingestion and alerting paths; flexible for batch and compaction

---

### Your Task

Build a 3-year infrastructure capacity plan with cost optimization for Crestline Energy.

---

### Deliverables

- [ ] Capacity growth model: show math for both conservative and aggressive growth scenarios
- [ ] Current-state utilization analysis: per workload — current utilization, peak-to-average ratio, rightsizing recommendation
- [ ] Reserved instance analysis: which instances to commit to 1-year reserved (with break-even calculation)
- [ ] Spot instance strategy: which batch workloads can migrate to spot, estimated savings, fallback design
- [ ] 3-year cost model: monthly cost projection for both scenarios, with and without optimization
- [ ] Cost per meter read: unit economic trend over 3 years
- [ ] Executive summary slide: 3 bullets, 1 number (€M savings over 3 years)
- [ ] Tradeoff analysis (minimum 3):
  - 1-year reserved vs 3-year reserved — commitment vs flexibility for uncertain growth
  - Cost optimization vs reliability floor — where the tradeoff exists and where it doesn't
  - Rightsizing now vs growing into current capacity — the engineering org overhead of frequent resizing
