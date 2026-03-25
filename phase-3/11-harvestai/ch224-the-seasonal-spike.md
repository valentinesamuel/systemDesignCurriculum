---
**Title**: HarvestAI — Chapter 224: Design for 100x, Spend 2x
**Level**: Staff
**Difficulty**: 8
**Tags**: #seasonal-scaling #auto-scaling #finops #cost-engineering #capacity-planning
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 196 (AutoMesh fleet doubles), Ch. 234 (NexusWealth market volatility), Ch. 57 (LuminaryAI capacity planning sprint)
**Exercise Type**: Performance Optimization / System Evolution
---

### Story Context

**HarvestAI Engineering — All-Hands Planning Meeting, February 6th, 10:00 AM PST**
*(Conference room. Priya Sundaram has connected a laptop to the projector. Raj Patel has a printed spreadsheet. Carlos Mendes is on a video call from São Paulo. Dr. Aisha Kamara has a coffee and a skeptical expression.)*

---

**Priya Sundaram:**
"Planting season starts in sixty days. Last year we buckled. I want to do a full post-mortem on the cost model before we talk architecture, so Raj — show them."

**Raj Patel:**
*(clicks through to a bar chart)*
"Okay. This is our AWS bill for last year by month."

*(The chart shows a nearly flat line from August through January — roughly $12,000/month — then a sharp cliff-edge ascent starting mid-March, peaking at $178,000 in May, staying high through June, then falling off a cliff again in July.)*

"Peak month was $178K. Off-peak average was $12K. We ran 15x more EC2 capacity in May than in January and we still had performance problems. The geospatial query layer — which you're already fixing — was one piece. The bigger problem was that we were running our full production fleet year-round and then trying to scale on top of it."

**Carlos Mendes** *(from São Paulo, audio crackling slightly)*:
"The Brazil side is worse. The Southern Hemisphere season is offset by six months — we have load in October through December when North America is quiet. But we share the same infrastructure, so there's no real off-peak. We're always paying for one hemisphere's peak while the other is idle."

**Dr. Aisha Kamara:**
"The ML inference compute is the worst offender. During planting season we're running 400 GPU-hours a day on crop health models. During winter we run maybe 20. But the GPU instances take 8 to 12 minutes to cold-start. We can't afford to cold-start in the middle of a farmer's request."

**Raj Patel:**
"I did a breakdown."

*(He passes around a printed sheet. You look at it.)*

```
HarvestAI Monthly Infrastructure Cost — Peak vs Off-Peak

                          OFF-PEAK ($)    PEAK ($)    RATIO
API layer (EC2)              2,100         28,000      13x
Database (RDS)               3,200         42,000      13x
ML inference (GPU EC2)         800         68,000      85x
Worker fleet (BullMQ)          900         24,000      27x
Redis cluster                  600          8,000      13x
S3 / data transfer           1,400          5,000       4x
Monitoring / misc            3,000          3,000       1x
------------------------------------------------------------
TOTAL                       12,000        178,000      15x
```

"The ML inference GPU line is the killer. We pay for those instances whether they're running models or sitting idle. Last year we pre-provisioned 12 GPU instances in March and ran them through July. That's 120 days × 12 instances × $12/hour = roughly $172,000 just in GPU compute. A third of our annual cloud bill."

*(Silence in the room.)*

**Priya Sundaram:**
"This year we have more farms, more sensors, more data. North America alone is 20x normal load for three months. Brazil is 8x normal load for three months at a different time. If we do nothing, I project we hit $280K/month at peak."

**Carlos Mendes:**
"We can't raise $280K/month from our current ARR without cutting other things."

**Dr. Aisha Kamara:**
"Can we move the ML inference to batch? If we're not doing real-time inference during the spike, we could queue everything and process in bulk at off-peak prices."

**Raj Patel:**
"Some of it, yes. But farmers paying for the premium tier expect results in under 2 minutes. You can't tell them 'come back in 6 hours.' The latency SLA is in their contracts."

**Priya Sundaram:**
"Here's the constraint I want to design around."

*(She writes on the whiteboard in large letters.)*

> "Design for 100x growth. Spend only 2x more."

"That means by end of next year, if we have 100x the load, our peak cloud bill should be $24K/month, not $178K. I know that sounds impossible. I want to understand exactly what would have to be true for it to be achievable. That's your job."

*(She looks at you specifically.)*

"You're going to design the seasonal architecture. You've seen what the bill looks like. Now show me what 2x spend looks like at 100x scale."

---

**Slack DM — Raj Patel → You, that afternoon**

**Raj Patel** [2:47 PM]
btw I looked at spot instance pricing for our GPU workloads
the g4dn.xlarge we use is $0.526/hr on-demand, $0.1578/hr spot
that's a 70% discount
but spot instances can be interrupted with 2 minutes notice
for batch crop analysis jobs that's probably fine
for the real-time premium tier inference it's definitely not fine
the question is: can we architect the ML pipeline so the spot instances handle 80% of the work and the small on-demand fleet handles just the urgent jobs?

**Raj Patel** [2:48 PM]
also
our RDS instance is a db.r6g.8xlarge
$3.84/hr on-demand
we run it year-round
there's a 1-year reserved instance price of $2.11/hr
if we commit to 1 year, we save $14K/year just on the database
but then we're locked in
if we want to downscale the DB off-peak we lose the reservation benefit
it's a classic commitment vs flexibility tradeoff

**Raj Patel** [2:49 PM]
I've been building a spreadsheet model
want me to share it?

---

### Problem Statement

HarvestAI's infrastructure bill follows an extreme seasonal curve: $12K/month in the off-season, $178K/month at peak, driven primarily by ML inference GPU compute (85x swing), API/worker fleets (13-27x swing), and database resources (13x swing). The business cannot sustain this cost model at growth: projecting 100x load over the next 18 months would push peak bills to over $1.7M/month at current architecture efficiency. The business constraint is to reach that 100x load ceiling while spending no more than 2x the current peak — approximately $360K/month, ideally much less.

This requires a fundamental rethinking of how HarvestAI provisions, scales, and tears down infrastructure seasonally. The design must handle: hemisphere-offset seasonality (North America and Brazil have opposite peak seasons), SLA-tiered compute (premium-tier farmers need real-time ML inference; standard-tier can tolerate batch), burst ingestion patterns (sensor flush at top of hour), and a small engineering team (12 engineers total, including 1 infra engineer) who cannot operate a complex scaling system without significant automation.

---

### Explicit Requirements

1. The system must automatically scale from off-peak baseline (~$12K/month equivalent) to peak capacity within 30 minutes of load onset — not within hours
2. ML inference must be split into two tiers: real-time (SLA: < 2 minutes, on-demand compute required) and batch (SLA: < 6 hours, spot/preemptible compute acceptable)
3. The database layer must handle 20x normal query load without manual intervention; connection pooling must be part of the design
4. Worker fleets (BullMQ job processors) must scale based on queue depth, not time-based schedules
5. Pre-warming must be automated: the system should begin scaling up based on calendar signals (planting season start dates are predictable to within ±7 days) combined with live load signals
6. North American and Brazilian infrastructure must share underlying resources where possible to exploit hemisphere offset and reduce idle capacity
7. The system must produce a monthly cost estimate (not just an architecture) — the design is not complete without a cost model showing the 2x-spend target

---

### Hidden Requirements

1. **Hint: re-read Carlos's line about Brazil sharing the same infrastructure.** He says "there's no real off-peak — we're always paying for one hemisphere's peak while the other is idle." This implies a specific architecture decision: should North American and Brazilian workloads share a single global cluster, or should they be separate regional deployments that can independently scale? What does "sharing underlying resources" actually mean at the infrastructure layer — is it the same RDS instance, the same EC2 autoscaling group, or something else?

2. **Hint: re-read Dr. Aisha's line about GPU cold-start: "8 to 12 minutes."** A cold-start of 12 minutes is catastrophic for a 2-minute SLA. What does this imply about the minimum standing fleet that must be maintained even in off-peak? If you keep 1 warm GPU instance year-round vs 12 instances seasonally, what is the cost difference, and what is the maximum burst capacity you can serve from that 1-instance baseline before latency SLAs breach?

3. **Hint: re-read Raj's note about the 1-year RDS reservation.** He frames it as "commitment vs flexibility." But there is a third option: Reserved Instances for the baseline load, on-demand for the seasonal burst delta. If off-peak baseline is 1 RDS instance and peak requires 4, you can reserve 1 and provision 3 on-demand only during peak. What is the math on this mixed strategy vs full on-demand vs full reservation?

4. **Hint: Priya writes "Design for 100x growth. Spend only 2x more."** She is not asking you to design for 100x today. She is asking you to prove that the architecture is capable of reaching 100x without a linear cost relationship. The key insight is: which components of the cost model have fixed costs, which have linear scaling costs, and which can be made sublinear through design choices? A Mermaid diagram alone does not answer this — only a cost model does.

---

### Constraints

- **Current peak load**: 20x normal for North America (March–June); 8x normal for Brazil (October–December)
- **Growth target**: 100x normal load within 18 months (new farm acquisitions, sensor density increase)
- **Cost target**: peak cloud bill ≤ $24,000/month at 100x load (2x current off-peak baseline)
- **ML inference SLA**: premium tier < 2 minutes (on-demand required); standard tier < 6 hours (spot acceptable); 70% of farms are standard tier
- **GPU cold-start time**: 8–12 minutes (g4dn.xlarge)
- **Spot instance discount**: 70% vs on-demand; average interruption rate: ~5% per hour during high-demand periods (March–May)
- **Database**: RDS PostgreSQL, currently db.r6g.8xlarge ($3.84/hr on-demand, $2.11/hr 1-year reserved); connection limit: 5,000
- **Application servers**: Node.js/TypeScript on EC2 t3.xlarge ($0.1664/hr); autoscaling group currently min=4, max=40
- **Worker fleet**: BullMQ on EC2 t3.large ($0.0832/hr); currently 8–80 instances seasonal range
- **Pre-warning signal**: planting season start dates are known ±7 days in advance; earliest signal is soil temperature reaching 50°F (from sensor data)
- **Team constraint**: 1 infra engineer (Raj) must be able to operate the seasonal scale-up/down without a dedicated SRE team; must be heavily automated

---

### Your Task

Design HarvestAI's seasonal infrastructure architecture. This is not just an architecture diagram — it is an architecture plus a cost model. Prove mathematically that the design can hit the 2x cost target at 100x load. Show the cost savings from each design choice. Identify which cost line items are fixed, which scale linearly, and which your design makes sublinear.

The design must address: the ML inference tier split (real-time vs batch, spot vs on-demand, warm pool sizing), the database scaling strategy (read replicas, connection pooling under burst, reserved vs on-demand), the hemisphere offset resource-sharing architecture, the pre-warming automation (calendar + sensor signal triggers), and the queue-based load leveling strategy for non-urgent work.

---

### Deliverables

- [ ] **Mermaid architecture diagram**: full seasonal architecture showing normal baseline, pre-warm phase, peak phase, and scale-down phase; include ML inference tier split and hemisphere regions
- [ ] **Database schema** (for the auto-scaling coordination layer): `scaling_events`, `capacity_targets`, `hemisphere_load_signals` tables — the metadata layer that drives automated scaling decisions
- [ ] **Seasonal cost model** (the core deliverable — show all math):
  - Off-peak baseline cost per component
  - Peak cost per component at current 20x load
  - Peak cost per component at projected 100x load with new architecture
  - Prove the 2x target is achievable (or prove it is not, and explain what the realistic floor is)
  - Include spot instance interruption risk modeling: if 5% of spot instances are interrupted per hour, what is the expected impact on batch job completion rate?
- [ ] **Scaling estimation**:
  - How many GPU instances are needed at 100x load for premium-tier real-time inference?
  - What is the minimum warm-pool size that keeps cold-start latency within the 2-minute SLA?
  - At what queue depth should the worker fleet trigger an auto-scaling event?
- [ ] **Tradeoff analysis** (minimum 3):
  - Spot instances for ML inference: cost savings vs reliability risk vs SLA tier design
  - Shared global infrastructure vs hemisphere-isolated deployments: cost vs operational complexity vs data residency
  - Reserved instances for baseline vs full on-demand: break-even analysis (at what utilization percentage does reservation become cheaper?)
- [ ] **Pre-warming automation design**: pseudocode for the scaling trigger function that combines calendar signals (planting date calendar) with live soil temperature sensor data to initiate pre-warm 48 hours before predicted load onset

### Diagram Format
All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
