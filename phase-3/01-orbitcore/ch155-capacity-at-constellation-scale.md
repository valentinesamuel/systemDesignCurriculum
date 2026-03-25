---
**Title**: OrbitCore — Chapter 155: Capacity at Constellation Scale
**Level**: Staff
**Difficulty**: 8
**Tags**: #capacity-planning #scaling #cost-engineering #finops #distributed #infrastructure
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 57 (LuminaryAI capacity planning), Ch. 105 (Crestline Energy capacity), Ch. 150 (OrbitCore ingestion), Ch. 151 (cell architecture)
**Exercise Type**: Performance Optimization
---

### Story Context

It is April 8th. One month in.

**[Slack DM — James Whitfield (CEO) → You, 7:02am]**

> **James Whitfield**
> Good morning. I know this is early.
> Board closed Series C Friday night. $140M. I'm sure Ingrid will announce today.
> I'm sending this to you directly because you're going to get a lot of questions about what this means for engineering.
> The answer: we're expanding from 47 satellites to 200. In 18 months.
> Batch 1: 60 additional satellites (Kepler expansion + new Meridian constellation), launches in 6 months.
> Batch 2: 45 more (ArcticLens expansion + new PolarGrid constellation), launches in 12 months.
> Batch 3: 48 more (TerraWatch expansion + Vantage expansion + new customer TBD), launches in 18 months.
> I need a capacity plan for the board by Tuesday. That's 72 hours from now.
> Not a vague roadmap. Real numbers. Infrastructure cost, team growth, and a clear statement of what breaks and when if we don't invest ahead of the curve.
> I know this is asking a lot. But you're Staff now. This is the work.

---

**[#engineering — Slack, 9:00am — Company-wide announcement]**

> **Ingrid Solberg**
> Team — we just closed our Series C. $140M. This is the result of every one of you doing excellent work over the past two years.
> What this means in practice: we're on a path to 200 satellites by end of next year.
> That's 4x our current capacity. The platform needs to grow with it.
> Details in the all-hands at 2pm. Questions in #series-c-q-and-a.

---

**[#series-c-q-and-a — Slack, 9:17am]**

> **Niamh Okafor**
> Does this mean we need more ground stations? 12 seems low for 200 satellites.

> **Dr. Yusuf Adeyemi**
> Yes. Rule of thumb: you need roughly 1 ground station per 10-12 LEO satellites to maintain continuous coverage. At 200 satellites, 12 ground stations is insufficient. You'll have coverage gaps.

> **Niamh Okafor**
> How many do we need?

> **Dr. Yusuf Adeyemi**
> Depends on orbital inclination of the new constellations. Polar orbits (PolarGrid) need high-latitude stations. We have none.

---

**[1:1 — Ingrid Solberg + You, April 8, 11:00am]**

> **Ingrid Solberg:** James wants the board deck by Tuesday. I need you to own the infrastructure capacity section. Preet will help with current baselines, Yusuf will handle orbital coverage modeling. You synthesize it.

> **You:** What level of confidence does the board expect?

> **Ingrid Solberg:** Order of magnitude for cost. But the growth triggers need to be precise — when do we add a ground station, when do we add a Kafka cluster, when do we hit storage limits. Those need to be engineering-defensible numbers, not educated guesses.

> **You:** And the "Design for 100x, spend 2x" question — is that real?

> **Ingrid Solberg:** That's exactly how the Series C lead framed it. Their portfolio companies have all over-provisioned at this stage and wasted 30-40% of their capital. They want us to grow just ahead of demand, not years ahead. But "just ahead" with satellite launches is non-trivial — you can't un-launch a satellite if the infrastructure isn't ready.

---

**[Slack DM — Preet Kapila → You, 1:30pm]**

> **Preet Kapila**
> Here's the current baseline for your capacity model:
>
> Ground station compute (per station, current):
> - 8x c5.4xlarge EC2 (ingest workers): $0.68/hr each = $5.44/hr per station
> - 3x r5.2xlarge EC2 (Kafka brokers): $0.504/hr each = $1.51/hr per station
> - 2x m5.xlarge (coordination + routing): $0.192/hr each = $0.38/hr per station
> Total per station: ~$7.33/hr = ~$5,280/month per station
> 12 stations: ~$63,360/month
>
> Kafka (shared cluster, separate from ground stations):
> - 9 brokers, m5.2xlarge: $0.384/hr each = ~$25,000/month total
>
> Storage (S3, hot tier):
> - Current consumption: 4.8 TB/day (200 bytes × 2M events/sec × 86400 sec / 1e9)
> - Actually we're at 3.1 TB/day observed — compression helps
> - 6-month hot tier: ~560 TB
> - S3 standard: $0.023/GB = ~$12,880/month
>
> Cold storage (Glacier):
> - 7 year retention, last 5.5 years
> - ~4 PB estimated
> - Glacier: $0.004/GB = ~$16,000/month
>
> Network egress:
> - ~800 GB/day cross-region (sovereignty splits)
> - AWS egress: $0.09/GB = ~$2,160/month
>
> Total current monthly infra: ~$120,000/month
>
> These are real numbers. Use them.

---

**[Email — Dr. Yusuf Adeyemi → You, 3:45pm]**

**From:** Yusuf Adeyemi
**To:** [You]
**Subject:** Ground Station Coverage Model — Inputs for Capacity Plan

> Quick notes for your model:
>
> **Current coverage efficiency (47 satellites, 12 stations):** ~73%
> (27% of satellite passes have no ground station contact — we buffer onboard)
>
> **At 107 satellites (6 months), 12 stations:** coverage efficiency drops to ~51%
> This is below our contractual commitment of 65% minimum coverage.
> We need 4-5 new ground stations before Batch 1 launches.
> Recommended locations: Iceland (polar coverage), Brazil, Japan, UAE.
>
> **At 152 satellites (12 months), 16-17 stations:** coverage ~68% (marginally acceptable)
> Add 1-2 more stations by month 12. South Africa (upgrade Joburg) and India.
>
> **At 200 satellites (18 months), 18-20 stations:** coverage ~72-75% (target: 70%+)
> Final 2-3 stations based on new constellation orbital parameters.
>
> Each new ground station: 8-12 week lead time for equipment + site contracts.
> Iceland and Brazil are greenfield sites — 16 week lead time minimum.
>
> You need to start the Iceland and Brazil procurement in the next 60 days.

### Problem Statement

OrbitCore has closed a $140M Series C and committed to expanding from 47 satellites to 200 satellites across three launch batches over 18 months. The board requires a capacity plan with infrastructure cost modeling, growth triggers, and a clear statement of what fails when without investment.

Produce OrbitCore's 18-month capacity plan covering: ground station scaling (compute and coverage), Kafka/messaging infrastructure, storage tiering, network egress, and team growth. The plan must show step-by-step math, identify the critical path items (what must be ordered NOW), and model costs at each 6-month milestone. Design for 100x eventual scale while justifying spending only what is needed for the next 18 months.

### Explicit Requirements

1. Model infrastructure at three milestones: M+6 (107 satellites), M+12 (152 satellites), M+18 (200 satellites)
2. Identify critical-path procurement items that must begin immediately (Iceland and Brazil ground stations are flagged by Yusuf as 16-week lead time items)
3. Produce monthly infrastructure cost at each milestone
4. Identify the first point at which each major system component (Kafka, storage, compute per station, network) hits a scaling limit under current architecture
5. Model the "just ahead of demand" provisioning strategy vs. over-provisioning — show the cost difference
6. Account for new ground station procurement in the capacity plan timeline
7. Include team growth model — infrastructure this complex requires headcount

### Hidden Requirements

1. **Hint: re-read Yusuf's email carefully.** He says coverage efficiency at 107 satellites with 12 ground stations drops to 51%, below the contractual minimum of 65%. He says you need 4-5 new stations before Batch 1 launches in 6 months. But Iceland and Brazil have a 16-week lead time. Six months minus 16 weeks = 8 weeks. You have 8 weeks to start procurement or you miss Batch 1's coverage SLA. This is the most urgent finding in the capacity plan.

2. **Hint: re-read James Whitfield's message.** He says "Batch 3 includes a new customer TBD." This new customer has unknown sovereignty requirements and unknown satellite orbital parameters. Your capacity plan must include a provisioning buffer for an unknown constellation — what is the right buffer size, and how do you model unknown sovereign requirements?

3. **Hint: re-read Preet's storage numbers.** He gives both a theoretical storage number (4.8 TB/day) and an observed number (3.1 TB/day, due to compression). At 200 satellites (4.25x current fleet), does the compression ratio hold? Satellite telemetry from new constellations (Meridian, PolarGrid) has unknown compression characteristics. Your storage model should include a compression assumption range.

### Constraints

- **Current**: 47 satellites, 12 ground stations, ~$120K/month infra
- **M+6**: 107 satellites, 12 (current) + 4-5 new ground stations needed
- **M+12**: 152 satellites, ~17 ground stations
- **M+18**: 200 satellites, ~19-20 ground stations
- **Coverage SLA**: ≥ 65% minimum coverage efficiency (contractual)
- **New ground station lead time**: 8-12 weeks (existing site types), 16 weeks (greenfield: Iceland, Brazil)
- **Critical procurement window**: Iceland and Brazil must be started within 8 weeks
- **Kafka scaling limit**: current 9-broker cluster is the limiting factor at ~3M events/sec sustained
- **Storage compression ratio**: 3.1/4.8 = 0.65 observed. Model range: 0.6–0.75 for new constellations
- **Team budget signal**: Series C allows for aggressive headcount growth, but board expects justification
- **FinOps constraint**: "Design for 100x, spend 2x" — the board's framing

### Your Task

Produce OrbitCore's 18-month infrastructure capacity plan. Show the math for every major cost component at each milestone. Identify critical-path items. Model two scenarios: just-ahead-of-demand provisioning vs. over-provisioning. Present the board-level cost summary and the engineering-level detail that supports it.

### Deliverables

- [ ] Mermaid timeline diagram showing:
  - Satellite launch batches (M+6, M+12, M+18)
  - Ground station procurement deadlines (backwards from launch dates)
  - Infrastructure scaling trigger points
- [ ] Scaling estimation (show math step by step) for each milestone:
  - Events/sec → MB/sec → GB/day → TB/month → TB/6months
  - Use compression ratio range (0.6–0.75)
  - Storage cost at S3 standard + Glacier at each milestone
  - Network egress growth curve
- [ ] Ground station capacity model:
  - Current: 12 stations at $5,280/month each
  - M+6: +4-5 stations, cost per new station (estimate for greenfield vs. managed co-lo)
  - M+12: +2-3 more stations
  - M+18: +2-3 more stations
  - Total ground station cost curve from $63K/month to projected M+18 number
- [ ] Kafka/messaging scaling triggers:
  - At what event/sec rate does the current 9-broker cluster hit its limit?
  - What does the next scaling step look like (more brokers? partition strategy change? different message bus?)
  - Cost of Kafka scaling at each milestone
- [ ] Monthly infrastructure cost model:
  - Line-item breakdown at current, M+6, M+12, M+18
  - Total infra cost trajectory
  - "Just ahead of demand" vs. "over-provisioned" cost comparison
- [ ] Critical path analysis:
  - What must be procured/started in the next 30 days?
  - 60 days? 90 days?
  - What is the cost of missing each deadline?
- [ ] Team growth model:
  - Current: 2 SREs, 4 platform engineers, you (Staff)
  - At 200 satellites across 20 ground stations, what team size is needed?
  - Headcount cost modeling (rough: $150K-$200K fully-loaded per engineer)
- [ ] Tradeoff analysis (minimum 3):
  - Greenfield ground stations (owned) vs. managed ground-station-as-a-service (e.g., AWS Ground Station, Leaf Space)
  - Kafka vs. Pulsar at 10M events/sec — cost and operational complexity tradeoff
  - Just-ahead-of-demand provisioning risk vs. over-provisioning cost
- [ ] "Design for 100x, spend 2x" analysis:
  - Current spend: ~$120K/month
  - 2x = $240K/month budget constraint
  - At M+18 (200 satellites), can you stay at or below $240K/month?
  - If not, what is the minimum spend at M+18 with the current architecture?
  - What architectural changes would reduce M+18 cost to $240K?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

Key diagram: a Gantt chart showing the capacity plan timeline.

```
gantt
  title OrbitCore Capacity Plan — 18 Month Horizon
  dateFormat YYYY-MM
  section Ground Stations
    Iceland Procurement    :crit, 2026-04, 4M
    Brazil Procurement     :crit, 2026-04, 4M
    ...
  section Kafka
    Cluster Scale Phase 2  :2026-09, 2M
    ...
```
