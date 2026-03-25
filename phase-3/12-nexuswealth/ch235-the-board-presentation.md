---
**Title**: NexusWealth — Chapter 235: The Trustees Want Answers
**Level**: Staff
**Difficulty**: 7
**Tags**: #cost-engineering #finops #capacity-planning #compliance #staff-engineering
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 230, Ch. 232, Ch. 233, Ch. 234
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**Email chain — Subject: Board of Trustees — Q1 Meeting Agenda Item 4: Infrastructure Cost Overrun**

---

**From**: Franklin Drummond, Chairman, NexusWealth Board of Trustees
**To**: Priya Nair, CTO; Harold Beckford, CFO
**Date**: February 3, 2025 — 8:15 AM

Priya, Harold,

The Finance Committee reviewed the 2024 annual infrastructure spend report this morning. I want to be direct: the numbers are not acceptable.

In 2021 we spent $8.1M on infrastructure. In 2024 we spent $22.3M. That is a 175% increase in three years. In the same period, assets under management grew from $41B to $57B — a 39% increase. Our participant count grew from 7.2M to 12M — a 67% increase.

Infrastructure costs grew nearly three times faster than the business. That is a material misalignment that the Board has a fiduciary obligation to investigate. These are pension fund participant assets we are ultimately accountable for.

I am placing this on the agenda for the February 18th Trustees meeting. I would ask that you present a complete explanation of where the money went, why it was necessary, and a credible plan to realign infrastructure cost growth with business growth.

This is not a witch hunt. But the Board needs to understand what it paid for.

Franklin Drummond

---

**Reply from**: Priya Nair, CTO
**To**: Franklin Drummond; Harold Beckford; you (direct add)
**Date**: February 3, 2025 — 10:41 AM

Franklin,

Noted. We will be prepared for February 18th.

I am adding our Staff Engineer directly to this thread. They will lead the infrastructure cost analysis and present alongside me. They know the architecture in the most granular detail.

Priya

---

**1:1 transcript — Priya Nair (CTO) + you — February 3, 2025 — 2:00 PM**

**Priya**: Franklin Drummond has been Chairman of the Board for 19 years. He was CFO of a Fortune 100 before that. He understands numbers. Do not bring slides with percentages and call it an explanation. He will ask for the underlying data.

**You**: Understood. I'll build the cost attribution from raw AWS billing data. But I want to flag something before I start: I think a significant portion of that $22M is not waste — it's the cost of capability we acquired. The risk platform we built after the VIX 80 incident, the actuarial calculation engine, the ERISA compliance infrastructure. None of those existed in 2021.

**Priya**: The Board will not accept "we got more capable" as a complete answer. They will ask: what would the system have cost if you'd designed it well from the start? You need to have an answer for that.

**You**: That's fair. There's also waste. I know there is. We have resources that were provisioned for the March 31st batch window that are sitting idle the other 10 months of the year. We have some over-provisioned RDS instances that haven't been right-sized since 2022.

**Priya**: Put a number on the waste. Trustees are fiduciaries. They are protecting participant assets. If we have $4M of idle infrastructure, they will want to know why it still exists.

**You**: What tone should I take with them? They're not engineers.

**Priya**: They are smart, patient people who have been managing large institutions for decades. Respect their intelligence. Don't use jargon. Don't be defensive. The ones who've been around the longest have sat through presentations from engineers who thought they were the only person in the room who understood the problem. That's the fastest way to lose them.

**You**: Any guidance on the forward-looking plan?

**Priya**: Harold told me privately: the Board will accept cost growth that is tied to clear business outcomes. They will not accept unexplained growth. Tie every dollar to either AUM growth, participant growth, regulatory requirements, or incident response. If you can't tie it to one of those four, you need to explain why it exists.

---

**Slack DM — @harold.beckford (CFO) → @you — February 4, 2025 — 9:02 AM**

**Harold**: One more thing Priya may not have mentioned. The Board has a specific concern about the March 2024 infrastructure spike. We spent $1.2M in a single day during a market volatility event. That's visible in the monthly statement and they noticed it. Be ready to explain what happened and why we can't have that exposure again. A $1.2M unplanned spend in a single day is a budget governance question, not just a technology question.

---

You pull the AWS Cost Explorer data for 2021–2024. The story it tells is complicated. Some of the growth is capacity that should have been there from day one. Some is waste. Some is the result of incidents that revealed architectural gaps. Some is legitimate investment in compliance infrastructure that the DOL audit demanded. The Board needs to understand all of it — and they need to believe you're being honest, not spin-doctoring.

You open a blank presentation. Tuesday is 13 days away.

### Problem Statement

Prepare a Board of Trustees infrastructure cost presentation covering FY2021–2024. The Board is a governing body of fiduciaries responsible for participant assets; they are not technologists but are financially sophisticated and deeply accountability-oriented. The presentation must: explain where the $22.3M went with dollar-level attribution, diagnose the cost-to-business misalignment, defend legitimate investment, acknowledge and quantify waste, and present a credible 12-month cost optimization plan and 5-year forecast.

This chapter is a **cost analysis and communication exercise**, not a pure architecture exercise. The deliverables are structured accordingly.

### Explicit Requirements

1. Full cost breakdown for FY2024: attribute $22.3M to specific categories (compute, storage, networking, managed services, data transfer, support tiers)
2. Cost-per-participant trend 2021–2024: what did NexusWealth spend per participant per year, and how has that changed?
3. Cost-per-dollar-AUM trend 2021–2024: infrastructure spend as basis points of AUM
4. Growth attribution: for each major cost category, which of the four drivers applies — AUM growth, participant growth, regulatory requirements, or incident response?
5. Waste identification: idle resources, over-provisioned instances, underutilized services — total dollar value of avoidable waste
6. The March 2024 VIX 80 incident cost ($1.2M single day): full explanation and prevention plan
7. 12-month optimization plan: specific initiatives with projected savings and implementation risk
8. 5-year forecast: two scenarios — status quo trajectory vs. optimized trajectory
9. Budget governance proposal: what controls prevent a $1.2M unplanned spend from recurring without Board notification?
10. Presentation framing: written in language accessible to a Board of Trustees, not an engineering audience

### Hidden Requirements

- **Hint**: Re-read Harold's Slack message. He says the March 2024 incident cost "$1.2M in a single day" and asks why "we can't have that exposure again." The implicit question is: was there a budget approval process that should have caught this? When Priya said "remove the 20-worker limit, we figure out the cost later" during the incident, she made a real-time financial decision that bypassed any CFO approval. The Board wants to know: is there a governance process for emergency financial decisions, or was the $1.2M purely ad-hoc?
- **Hint**: Re-read Priya's guidance: "Tie every dollar to either AUM growth, participant growth, regulatory requirements, or incident response." The DOL audit (Chapter 231) required a specific compliance architecture — audit trails, calculation reproducibility, immutable records. What did that compliance work actually cost in infrastructure terms? The Board needs to understand this is not waste — it's the cost of regulatory survival.
- **Hint**: Re-read your own note in the 1:1: "resources provisioned for the March 31st batch window that are sitting idle the other 10 months of the year." This is the actuarial calculation infrastructure from Chapter 232. If the batch calculation engine runs for 6 weeks per year and sits idle for 46 weeks, the annual compute cost is severely misallocated. What would spot instances + hibernation have cost vs. always-on provisioned capacity? Calculate the waste in dollar terms.
- **Hint**: The Board is a fiduciary body. Franklin Drummond specifically says "these are pension fund participant assets." There is a hidden subtext in his letter: if infrastructure is over-provisioned, those excess costs ultimately come from plan returns — participants' retirement money. This frames the conversation not as "IT budget management" but as "fiduciary duty to participants." Your presentation needs to acknowledge this framing, not avoid it.

### Constraints

- **Infrastructure spend**: $8.1M (2021) → $11.4M (2022) → $17.8M (2023) → $22.3M (2024)
- **AUM**: $41B (2021) → $45B (2022) → $51B (2023) → $57B (2024)
- **Participants**: 7.2M (2021) → 8.1M (2022) → 9.8M (2023) → 12M (2024, including Heartland acquisition)
- **Single-day peak spend**: $1.2M (March 2024 VIX 80 incident)
- **Identified idle infrastructure**: ~$2.8M/year estimated (batch compute, over-provisioned RDS)
- **Audience**: Board of Trustees — financially sophisticated, not engineers, fiduciary perspective
- **Presentation date**: February 18th (13 days)
- **Presentation time**: 30 minutes + 15 minutes Q&A

### Your Task

Prepare the full cost analysis and Board presentation. This includes: cost attribution analysis, narrative explanation for the Board, waste identification with dollar figures, the March 2024 incident explanation, budget governance proposal, 12-month optimization plan with projected savings, and 5-year forecast.

### Deliverables

- [ ] **Cost attribution table**: FY2024 $22.3M broken into categories (compute, storage, DB, networking, managed services, support) with growth driver label for each
- [ ] **Cost-per-participant trend**: table showing $/participant/year for 2021–2024; calculate whether the per-participant cost is trending in the right direction
- [ ] **Cost-per-AUM trend**: infrastructure spend as basis points of AUM (e.g., $22.3M ÷ $57B AUM = 0.039 bps); is this acceptable for a pension fund administrator?
- [ ] **Waste analysis**: identify and quantify idle/over-provisioned resources in dollar terms; total avoidable waste as a % of total spend
- [ ] **March 2024 incident narrative**: what happened, what it cost, what architectural change prevents recurrence, what budget governance change prevents recurrence
- [ ] **12-month optimization plan**: at least 4 specific initiatives, each with: projected savings, implementation complexity (Low/Medium/High), and risk of reducing capability
- [ ] **5-year forecast table**: two columns — status quo (extrapolated trend) vs. optimized (with plan implemented); show total savings over 5 years
- [ ] **Budget governance proposal**: at least 3 controls (e.g., emergency spend threshold requiring CFO notification, monthly cost anomaly alerts, reserved-vs-on-demand ratio policy)
- [ ] **Mermaid diagram**: cost attribution by system/platform (infrastructure cost breakdown as architecture-linked diagram — which systems drive which costs)
- [ ] **Scaling estimation**: per-participant cost at 20M participants in 3 years — what does the optimized architecture cost vs. status quo?

### Diagram Format

Mermaid syntax. (Note: for this chapter, the Mermaid diagram should show infrastructure cost by system — e.g., a flowchart where each major system block is labeled with its approximate annual cost contribution.)
