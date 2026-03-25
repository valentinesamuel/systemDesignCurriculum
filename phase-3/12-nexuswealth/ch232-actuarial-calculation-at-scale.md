---
**Title**: NexusWealth — Chapter 232: The March 31st Problem
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #batch-processing #distributed-computing #compliance #financial-systems #actuarial
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 230, Ch. 231, Ch. 235
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**Email chain — Subject: URGENT: Annual Benefit Statement Processing — 2024 Cycle**

---

**From**: Renata Kowalski, VP Actuarial Services
**To**: Engineering Team DL; Priya Nair, CTO
**CC**: Compliance@nexuswealth.com
**Date**: January 14, 2025 — 8:47 AM

Team,

I need to brief everyone on the scope of the 2024 annual benefit statement cycle before we kick off planning. The regulatory deadline is March 31st. ERISA Section 105(a) requires that every participant receives their individual benefit statement within 90 days of plan year end (December 31st). Failure to deliver is a $50 per-participant per-day penalty, which at our scale is not theoretical — it's existential.

This year is different from prior years in three ways:

1. We have 12 million active participants. Last year was 9.8 million and we barely made the deadline — the final batch ran until 11:58 PM on March 31st.
2. The Heartland Pension Fund acquisition closed December 20th. That is 2 million new participants with entirely different plan types (multi-employer defined benefit, not the single-employer DB plans we normally process). Their calculation rules are materially different.
3. We are being audited this cycle by the Department of Labor. Every calculation must be individually auditable — they can ask us to reproduce the exact result for any participant, with the exact data and the exact formula version that was used, at any point in the future.

We have 6 weeks. I am available for a call today.

Renata

---

**Reply from**: Marcus Delgado, Principal Engineer
**Date**: January 14, 2025 — 9:22 AM

Renata,

I ran some rough numbers. With 12M participants and our current batch throughput of 8,000 calculations/hour, we're looking at 1,500 hours of compute. That's 62.5 days. We have 77 days total. That leaves exactly 14.5 days of slack for errors, reruns, and the regulatory review process — which itself takes 5 business days minimum.

The Heartland plans are the real problem. I pulled their plan documents last night. They have:
- 3 different early retirement factor tables depending on hire date
- Spousal benefit elections that require a separate calculation thread
- A COLA (cost-of-living adjustment) provision tied to CPI-W that no one on our team has implemented before

The good news: we could scale horizontally. The bad news: our current architecture doesn't support it cleanly — the calculation engine is a Node.js monolith that processes participants sequentially within each worker. We can run multiple workers, but they all share the same actuarial parameter database connection, and at scale that becomes a bottleneck.

I need to know: do we want to build a proper distributed calculation engine now, or do we just add more workers and accept the database bottleneck? One is the right answer. The other is the answer we can ship in 6 weeks.

Marcus

---

**Reply from**: Priya Nair, CTO
**Date**: January 14, 2025 — 9:51 AM

Marcus, I need both. I need a plan that gets us through March 31st AND moves us toward an architecture that won't create this same crisis next year when we have 15M participants.

One thing Renata didn't mention in her email but told me on the phone: the DOL audit specifically asked about our "calculation reproducibility" — they want to know if we can reproduce the exact statement for participant ID 77234891 using the data as it existed on December 31st, 2024, not today's data. That's a different problem from just running the calculation. That's a point-in-time replay problem.

Can we discuss at 2pm?

Priya

---

**Slack — #eng-benefits-platform**

**@you** [9:58 AM]: Just read the chain. The point-in-time replay requirement Priya mentioned — that's not a logging problem, that's a data model problem. If we've been updating participant records in-place (which I strongly suspect we have been), we have no way to reconstruct "what did this participant's record look like on December 31st, 2024?"

**@marcus.delgado** [10:02 AM]: Yeah. We've been updating in-place. Single `participants` table with `updated_at`. No history.

**@you** [10:04 AM]: So the DOL could ask us to reproduce a calculation we have no way of reproducing.

**@marcus.delgado** [10:06 AM]: Correct. And the penalty for failing a DOL audit is not $50/day. It's plan disqualification. Which means the entire pension fund loses its tax-exempt status.

**@you** [10:07 AM]: I'll redesign both problems. Give me until end of day.

---

By 2 PM you have sketched two systems that need to exist simultaneously: a distributed calculation engine that can process 12 million participants in under 6 weeks with full horizontal scale, and an immutable participant data model that preserves point-in-time snapshots for every plan year. These are not separate projects. They are one architecture.

### Problem Statement

Design a distributed actuarial calculation engine for annual benefit statements. The system must process 12 million pension plan participants within a 6-week window, support point-in-time data replay (reconstruct any participant's record as it existed on any historical date), handle heterogeneous plan types including multi-employer defined benefit plans with COLA provisions, and produce individually auditable results that the Department of Labor can request at any time in the future.

The system must also handle the 2 million Heartland Pension Fund participants whose plan rules — multi-employer DB, multiple early retirement factor tables, spousal benefit calculation threads — have never been implemented in this codebase.

### Explicit Requirements

1. Process 12M participant benefit statements within 77 calendar days (March 31st deadline)
2. Support at least 4 plan types: single-employer DB, multi-employer DB, defined contribution (DC), and hybrid (cash balance)
3. Each calculation must be individually reproducible: given participant ID + plan year, reproduce the exact result
4. Support point-in-time data snapshots: reconstruct participant record as it existed on any historical date
5. Heartland multi-employer plans require: 3 early retirement factor tables (by hire date range), spousal benefit calculation (separate thread), COLA provision tied to CPI-W index
6. DOL-auditable: every calculation must store the formula version, input data version, and actuarial parameter version used
7. Results must be reviewed by Renata's actuarial team before statements are mailed — workflow for actuary sign-off on batch results
8. Statement generation (PDF) must complete within 5 business days of calculation completion
9. Failed calculations must be retried with no risk of incorrect results being used

### Hidden Requirements

- **Hint**: Re-read Marcus Delgado's 10:06 AM Slack message. He says the penalty for a failed DOL audit is plan disqualification — not a fine. What does that mean for data retention? If the DOL can audit any year, any participant, indefinitely, how long must you retain calculation inputs, formula versions, and actuarial parameters?
- **Hint**: Re-read Priya's email. She specifically says "the data as it existed on December 31st, 2024, not today's data." What happens if a participant's compensation history is corrected after December 31st? The correction is legitimate — but the December 31st statement should reflect the data at that moment, not the corrected data. How do you handle retroactive corrections vs. point-in-time statements?
- **Hint**: Renata mentions a 5-business-day regulatory review before statements can be mailed. What is the implication for the batch schedule? If review takes 5 days and mailing takes 3 days, you need calculation complete by approximately March 21st — not March 31st. The real deadline is 10 days earlier than stated.
- **Hint**: The Heartland multi-employer plans have "spousal benefit elections that require a separate calculation thread." This means some participants require two calculations that must be correlated — participant benefit + spousal survivor benefit. What does this mean for your parallelization model?

### Constraints

- **Participants**: 12M total (10M legacy, 2M Heartland acquisition)
- **Plan types**: 4 (single-employer DB, multi-employer DB, DC, hybrid/cash balance)
- **Variables per participant**: 15-25 (compensation history, years of service, hire date, plan type, early retirement factors, beneficiary elections, COLA parameters, etc.)
- **Processing window**: 77 calendar days (January 14 – March 31), effective 67 days (real deadline ~March 21 with review + mailing buffer)
- **Calculation throughput required**: 12M ÷ 67 days = 179,000 participants/day minimum; target 300K/day for margin
- **DOL audit retention**: indefinite (plan disqualification risk — treat as permanent)
- **Regulatory review workflow**: actuarial team must approve batch results before mailing
- **Formula versioning**: multiple formula versions must coexist (Heartland plans use pre-acquisition formula rules)
- **Infrastructure budget**: existing AWS infrastructure; can expand horizontally
- **Team**: 4 engineers, 2 actuarial analysts (for formula validation)
- **Spousal benefit**: ~15% of participants have spousal benefit elections (1.8M correlated pair calculations)

### Your Task

Design the distributed actuarial calculation engine. Address: worker architecture for horizontal scale, the point-in-time participant data model, formula versioning and auditability, the spousal benefit correlation problem, and the actuarial review workflow.

### Deliverables

- [ ] Mermaid architecture diagram — full calculation pipeline from job scheduling through statement generation
- [ ] Database schema — participant snapshot model (point-in-time replay), calculation audit record, formula version registry
- [ ] Scaling estimation:
  - Workers needed to process 12M participants in 67 days
  - Database connection pool sizing for shared actuarial parameters
  - Storage for calculation audit records (inputs + outputs + metadata) for 12M participants × indefinite retention
  - Spousal benefit correlation: how does pairing 1.8M participant pairs affect throughput?
- [ ] Tradeoff analysis (minimum 3):
  - Point-in-time snapshots: event-sourced history vs. temporal tables vs. snapshot-on-close
  - Worker isolation: shared actuarial parameter DB vs. parameter cache per worker vs. parameter broadcast at batch start
  - Spousal benefit: sequential (calculate participant first, then spouse) vs. parallel (calculate both, then merge)
- [ ] Cost modeling: EC2 compute for 67-day window (on-demand vs. spot), RDS storage for audit records at 12M × indefinite retention ($X/month estimate)
- [ ] Capacity planning: what does the architecture look like when NexusWealth reaches 20M participants in 18 months?

### Diagram Format

Mermaid syntax.
