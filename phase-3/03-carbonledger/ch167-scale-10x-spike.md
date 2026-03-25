---
**Title**: CarbonLedger — Chapter 167: SPIKE — Scale 10x by Friday
**Level**: Strong Senior
**Difficulty**: 9
**Tags**: #distributed #database #caching #messaging #spike-scale-10x #cost-engineering
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 165 (Immutable Registry), Ch. 166 (Article 6), Ch. 57 (LuminaryAI capacity planning), Ch. 149 (VertexCloud FinOps)
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**#incidents — Slack, Thursday 7:43 AM**

**Yara Osei**: @channel — everyone stop what you're doing and read this

**Yara Osei**: [link: UN Environment Programme Press Release DRAFT — EMBARGOED UNTIL FRIDAY 9AM GMT]

**Yara Osei**: they gave us 18 hours notice. the announcement goes live tomorrow morning. CarbonLedger is the Article 6 registry.

**Kwesi Ampah**: holy

**Priya Nair** [Staff Engineer — joining from Stratum as advisor]: i just got on a plane in Nairobi. landing in 6 hours. do not touch the database until i'm there

**Yara Osei**: @you you need to start the scaling analysis NOW. i need to know: what breaks first, and what does it cost to fix it before tomorrow 9am

---

**#incidents — Slack, Thursday 8:01 AM**

**You**: okay. current baseline:
- PostgreSQL primary: ~400 writes/min (10K/day)
- Redis cache: 12GB used, 20GB allocated
- Kafka: 3 broker cluster, 12 partitions, ingestion ~8 MB/s
- API servers: 4x t3.xlarge behind ALB
- RDS instance: db.r5.2xlarge (8 vCPU, 64GB)

immediate after announcement (conservative): 10x in 24 hours = 100K txns/day = 4,000 writes/min
6-month horizon: 10M/day = 400K writes/min = ~6,700 writes/sec

**Kwesi Ampah**: our RDS max connections at current size is ~400. at 6700 writes/sec we are absolutely cooked

**You**: yep. and that's before we account for the CA fan-out — each cross-border transfer is 3 writes minimum

**Priya Nair**: [from airplane wifi] don't over-engineer the next 24h. focus on what breaks *tomorrow morning*. the 6-month architecture is a different conversation.

---

**#incidents — Slack, Thursday 9:22 AM**

**Yara Osei**: just got off the phone with AWS. they can have db.r5.8xlarge provisioned in 4 hours. that buys us time. what else

**You**: the event store is the bottleneck, not just the database size. we need to talk about write path architecture

**Kwesi Ampah**: also — heads up — i just found something in the UNFCCC integration spec that nobody told us about

**You**: what

**Kwesi Ampah**: Article 6.4, paragraph 3: "The Supervisory Body shall maintain an independent copy of all transactions registered in the Article 6.4 mechanism registry." that means the UN's own system — separate from the ITL-successor — needs a full replica of every transaction we process. that's a new integration nobody has scoped.

**Yara Osei**: ...nobody told us about 6.4 Supervisory Body

**Kwesi Ampah**: it's in the spec. page 47.

**You**: so we have 18 hours to handle 10x traffic AND add a new replication target AND not break the existing system

**Yara Osei**: correct. also the CFO just asked me for a cost estimate. current: $45K/month. what does the scaled system cost?

---

**#incidents — Slack, Thursday 10:15 AM**

**Priya Nair**: [still on plane, bless airplane wifi] okay here's the 72-hour playbook i'd use. you don't have to follow it but it's what i'd do:

hour 0-6: vertical scale the DB. add read replicas. enable PgBouncer connection pooling. that handles tomorrow morning.

hour 6-24: shard the event store by credit serial number. 256 shards minimum. move writes off the monolith DB and onto the event store.

hour 24-72: kafka partition expansion. add the 6.4 supervisory body replication consumer. cost model the 6-month state.

**Priya Nair**: the cost question is real. yara, at 10M/day, if you're not careful this is a $500K/month infrastructure bill. that kills the margin.

**Yara Osei**: what's the target

**Priya Nair**: $45K x 2.5 = $112.5K/month. that's the line to hold.

---

**Email — Thursday 11:30 AM**

**From**: David Nkrumah, CFO
**To**: Yara Osei; You
**Subject**: Cost modeling needed BEFORE announcement

Yara,

Before we accept the UN contract terms tomorrow, I need a number. The contract pays us $0.0008 per transaction for registry services. At 10M transactions/day that's $8,000/day revenue = $240K/month.

If the infrastructure costs $200K/month, we have a $40K margin before any engineering salaries. That is not sustainable.

I need: (1) current monthly infrastructure cost broken down by service, (2) projected cost at 10M/day, (3) minimum cost to run this at 10M/day with acceptable reliability.

I need this by 4pm today.

David

---

**Slack DM — Marcus Webb → You, Thursday 12:47 PM**

**Marcus Webb**: i heard about the UN announcement. congratulations, i think.

**You**: 72 hours to scale 10x. any wisdom?

**Marcus Webb**: yeah. don't try to solve the 6-month problem tonight. solve tomorrow's problem tonight. the mistake everyone makes in a 10x sprint is they redesign the whole system when they should just stabilize the immediate failure mode.

**Marcus Webb**: also: cost modeling under pressure is where junior engineers make mistakes. they forget that data transfer costs don't scale linearly with transactions. at global scale, cross-region replication is often your biggest bill. not compute.

**You**: the CA fan-out — 3 copies per cross-border transfer across 3 regions

**Marcus Webb**: now you're thinking about the right problem. go build it.

---

### Problem Statement

CarbonLedger has 18 hours before a public UN announcement that will immediately increase transaction volume by 10x (100K/day) with a 6-month trajectory to 1,000x (10M/day). The current architecture — a single PostgreSQL RDS instance, a 3-broker Kafka cluster, and 4 API servers — will not survive the announcement. Additionally, a previously unknown requirement has surfaced: Article 6.4 of the Paris Agreement requires the UN Supervisory Body to maintain an independent replica of all transactions, adding a new replication target to every write path.

The CFO needs a cost model by 4pm. The architecture must handle the immediate 10x increase by morning AND provide a credible path to 10M/day at ≤2.5x the current monthly cost ($112.5K/month target, current: $45K/month).

### Explicit Requirements

1. System must survive 10x traffic (100K transactions/day) by 9 AM tomorrow — approximately 18 hours from now
2. Architecture must provide a credible path to 10M transactions/day within 6 months
3. Article 6.4 Supervisory Body replication must be added: every transaction must be replicated to the UNFCCC Supervisory Body system
4. Cost at 10M/day must not exceed $112.5K/month (2.5x current)
5. No downtime during the scaling operations — the registry must remain operational
6. The immutability guarantee from Ch. 165 must not be compromised by scaling shortcuts
7. Write acknowledgment SLA remains < 500ms p99 at 10M/day

### Hidden Requirements

- **Hint**: Re-read Kwesi's Slack message about Article 6.4. "The Supervisory Body shall maintain an *independent* copy." Independent means it cannot be the same database cluster. What does this mean for the write path? Every transaction write now has a mandatory second write to an external system — this changes the write path latency budget and introduces a new failure mode.
- **Hint**: Marcus Webb said "data transfer costs don't scale linearly with transactions. At global scale, cross-region replication is often your biggest bill." The 3-copy CA architecture from Ch. 166 means every cross-border transfer generates data transfer costs across 3 AWS regions. Calculate this separately in the cost model.
- **Hint**: Priya Nair said "256 shards minimum" for the event store. Why 256? Think about the credit serial number format (CR-{year}-{sequence}) and how you'd partition it. What happens to your shard count if you choose a bad partition key?
- **Hint**: The CFO's math: $0.0008 per transaction × 10M/day = $8,000/day = $240K/month revenue. If infrastructure is $112.5K/month, the engineering team budget is ~$127.5K/month. That's 2-3 engineers at market rate. The architecture must minimize ongoing operational burden — not just cost.

### Constraints

- **Immediate deadline**: 18 hours to handle 10x traffic (100K/day)
- **6-month target**: 10M transactions/day
- **Current cost**: $45K/month (breakdown: RDS $8K, EC2 $6K, Kafka MSK $4K, S3/storage $3K, data transfer $2K, misc $22K)
- **Cost target**: ≤$112.5K/month at 10M/day
- **Immutability**: Cannot be compromised — the UN contract specifies cryptographic audit trail
- **Current write latency**: ~180ms p99 (single region, 10K/day)
- **Target write latency**: <500ms p99 at 10M/day (includes Supervisory Body replication)
- **Data transfer**: Cross-region transfer costs $0.09/GB (AWS inter-region); each CA event ~2KB; 15% of 10M = 1.5M cross-border/day = 3GB/day = $0.27/day = $100/month per region pair
- **Team**: 4 backend engineers, 18-hour window for immediate work, 72-hour window for sprint

### Your Task

Produce a two-phase scaling plan: Phase 1 (18-hour stabilization for tomorrow's announcement) and Phase 2 (6-month architecture for 10M/day). For Phase 1, be specific about the exact commands/steps. For Phase 2, produce the full architecture with cost modeling and the Supervisory Body replication design.

### Deliverables

- [ ] Mermaid architecture diagram: Phase 2 target architecture showing event store sharding, Kafka expansion, read replica tier, Supervisory Body replication path, and regional CA storage
- [ ] Phase 1 playbook (18-hour stabilization): ordered list of specific steps with estimated time per step and risk level
- [ ] Phase 2 architecture description: event store sharding strategy (partition key choice, shard count rationale), Kafka partition expansion plan
- [ ] Supervisory Body replication design: sync vs. async, failure handling, data format for UNFCCC API
- [ ] Scaling estimation (show math):
  - Current: 10K/day = N writes/sec = X RPS on DB
  - Phase 1: 100K/day = ?
  - Phase 2: 10M/day = ? — including CA fan-out writes
  - Connection pool math: at 6,700 writes/sec, how many DB connections do you need? What pool configuration?
- [ ] Cost model (show math, line by line):
  - Current $45K/month breakdown
  - Phase 2 projected costs at 10M/day: compute, storage, Kafka, data transfer (cross-region), Supervisory Body replication overhead
  - Total: can you hit $112.5K/month? What optimizations get you there?
- [ ] Tradeoff analysis (minimum 3): vertical scaling vs. horizontal sharding for Phase 1; synchronous vs. asynchronous Supervisory Body replication; cost vs. durability on event storage tier selection
- [ ] Capacity planning: month-by-month growth forecast from 100K/day (Day 1) to 10M/day (Month 6), with infrastructure expansion trigger points

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
