---
**Title**: CarbonLedger — Chapter 169: The Cost of Green
**Level**: Staff
**Difficulty**: 7
**Tags**: #cost-engineering #finops #database #storage #distributed
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 149 (VertexCloud FinOps), Ch. 109 (Crestline Energy capacity planning), Ch. 167 (CarbonLedger Scale 10x)
**Exercise Type**: Performance Optimization
---

### Story Context

**Email — Wednesday 8:15 AM**

**From**: David Nkrumah, CFO
**To**: You; Yara Osei
**Subject**: Infrastructure Cost Analysis — URGENT before Series B

Both,

I've attached the AWS cost explorer export for the last 30 days. I'll give you the headline number: $187,000 last month.

When we raised Series A, we modeled infrastructure at $0.0015 per transaction. At our current volume (approximately 450,000 transactions in October), that implies ~$675 in infrastructure costs. We spent $187,000.

I understand we've been scaling rapidly since the UN announcement. But $187K/month for 450K transactions is $0.42 per transaction — 280x our model. At 10M transactions/day (300M/month), extrapolating linearly gives us $126M/month. That is more than our total projected annual revenue.

I need three things from engineering by end of week:
1. A breakdown of where the $187K is actually going
2. The unit economics at 10M/day — what does one transaction *actually* cost to process?
3. A credible plan to get the per-transaction cost to $0.003 or below (our break-even with the UN contract pricing)

I'm meeting with the Series B lead investor on Friday. If I can't answer these questions, the round may not close.

David

---

**Slack DM — You → Yara Osei, Wednesday 8:47 AM**

**You**: have you seen this email

**Yara Osei**: yes. i was going to call you. this is bad.

**You**: $187K for 450K transactions. that's... almost all of it is infrastructure overhead, not variable cost. right?

**Yara Osei**: that's my guess. the scaling we did post-UN announcement was not designed for cost efficiency. it was designed to survive. we over-provisioned everything.

**You**: what's the breakdown?

**Yara Osei**: i don't know exactly. can you pull it from cost explorer?

---

**Slack Thread — #engineering-cost-review, Wednesday 9:30 AM**

**You** [after pulling cost explorer]:

Okay. Here's the actual breakdown for October:

| Service | Monthly Cost | % of Total |
|---|---|---|
| RDS (db.r5.8xlarge, 2 replicas) | $52,400 | 28% |
| ElastiCache (Redis, r6g.2xlarge cluster) | $31,200 | 17% |
| MSK (Kafka, 6-broker cluster) | $28,800 | 15% |
| EC2 (API servers, 12x t3.2xlarge) | $18,600 | 10% |
| S3 (event archive, 18TB) | $3,200 | 2% |
| S3 Glacier (cold archive, 140TB) | $1,120 | 1% |
| CloudFront | $4,200 | 2% |
| Data transfer (cross-region, CA fan-out) | $38,900 | 21% |
| EKS, misc | $8,580 | 5% |

**You**: data transfer is 21% of our bill. the CA fan-out across 3 regions for cross-border transfers is killing us.

**Kwesi Ampah**: and the RDS instance — we're using maybe 15% of its capacity. we over-provisioned for the peak we never hit yet.

**Priya Nair**: the kafka cluster is 6 brokers for 450K txns/day. that's... aggressive.

**You**: we provisioned for 10M/day peak. we're running at 0.005% utilization on kafka.

---

**Slack DM — You → Priya Nair, Wednesday 10:05 AM**

**You**: how do i think about this? david wants $0.003 per transaction. we're at $0.42.

**Priya Nair**: start with the fixed vs variable split. most of your $187K is fixed cost — it doesn't move with transaction volume. your data transfer cost is the variable part.

**Priya Nair**: the RDS, kafka, elasticache, EC2 — those are all sized for your peak target, not your current volume. if you rightsized them for TODAY, you'd save maybe $70-80K/month immediately.

**Priya Nair**: the harder question is: what does the system cost at 10M/day? that's where the math gets interesting. your data transfer cost at 10M/day with 15% cross-border fan-out across 3 regions... let me think about that.

**You**: i calculated it. at 10M/day, 1.5M cross-border × 3 regions × 2KB per event = 9GB/day cross-region = $25/day = $750/month on data transfer. that seems fine.

**Priya Nair**: wait. you're counting the data size of one event. but the CA fan-out isn't one event. it's one event replicated to 3 endpoints. and each endpoint is a full TLS-encrypted HTTP POST with headers. the actual payload per CA transfer is closer to 8-10KB with headers and retry metadata. and you're paying for each direction.

**You**: so it's more like 1.5M × 3 × 10KB = 45GB/day = $4/day = $130/month. that's still fine.

**Priya Nair**: is it? what about the UNFCCC supervisory body replication — that's a 4th copy. and what about your audit log writes to S3? and what about read replicas synchronizing across availability zones?

**You**: ...okay i need to redo this calculation

**Priya Nair**: unit economics are where Series B investors probe. David needs to walk in that room with a believable number, not a rough estimate. show your work.

---

**1:1 — You and David Nkrumah (CFO), Thursday 2:00 PM**

**David**: "Before you start — I want to understand the model, not just the number. Help me understand what makes infrastructure expensive for a registry like ours."

**You**: "Three things. First, durability guarantees — every transaction must be written to at least 3 places before we acknowledge it. That's 3x the I/O cost. Second, compliance overhead — the bi-temporal event log, the hash chain verification, the audit exports — those aren't free. They add ~40% to storage costs compared to a simple database. Third, the cross-border replication — every international transfer fans out across multiple regions, and AWS charges for data leaving a region."

**David**: "So the compliance requirements make us expensive."

**You**: "They make us *trustworthy*. The question is whether we can be trustworthy cheaply. And I think we can — but it requires some architectural changes."

**David**: "What's the target?"

**You**: "$0.003 per transaction at 10M/day is achievable. But not with the current architecture. We need to change how we handle the CA fan-out, right-size the provisioned capacity, and move cold data aggressively to cheaper storage tiers."

**David**: "Can you put that in a spreadsheet?"

**You**: "I can put it in something better than a spreadsheet."

---

### Problem Statement

CarbonLedger is spending $187,000/month to process 450,000 transactions — a unit cost of $0.42/transaction that is 140x the break-even target ($0.003) for the UN contract to be profitable. The Series B round depends on demonstrating a credible path to $0.003/transaction at 10M/day. This requires a full FinOps analysis: understanding where the money goes today, identifying over-provisioned resources, redesigning expensive architectural patterns (especially cross-region data transfer), and building a cost model that shows unit economics at scale.

### Explicit Requirements

1. Produce a complete cost breakdown: identify which services are over-provisioned and by how much
2. Model unit cost at three volume points: current (450K/day), target-near (1M/day), target-scale (10M/day)
3. Identify the top 3 cost drivers and propose specific optimizations for each
4. Cross-region data transfer cost must be explicitly modeled (it is currently the second-largest cost driver)
5. Reserved instance / savings plan analysis: what 1-year and 3-year commitments make financial sense at projected growth?
6. The compliance requirements (bi-temporal event log, hash chains, multi-copy durability) must remain intact — cost optimization must not compromise these
7. Capacity right-sizing: identify over-provisioned resources that can be downsized immediately without affecting reliability

### Hidden Requirements

- **Hint**: Re-read Priya Nair's Slack message about the CA fan-out calculation. She corrected your initial math — actual payload per CA transfer is 8-10KB with headers, not 2KB. What other places in your cost model might you be underestimating? (Hint: what about retry traffic? What percentage of CA notifications fail and require retry — and how does that change the data transfer bill?)
- **Hint**: David asked "so the compliance requirements make us expensive?" and you said they add ~40% to storage costs. But you haven't quantified the compliance overhead on *compute*. Hash chain verification is CPU-intensive. At 10M events/day, how much CPU time is spent computing SHA-256 hashes? Is that a meaningful cost?
- **Hint**: The cost table shows S3 Standard: $3,200 for 18TB and S3 Glacier: $1,120 for 140TB. But the table doesn't show S3 API call costs. At 10M events/day writing to S3, what do the PUT request costs look like? (S3 Standard PUT: $0.005/1,000 requests)
- **Hint**: Kwesi said "we're using maybe 15% of [RDS] capacity." But right-sizing RDS carries a risk: what if volume spikes unexpectedly? The cost optimization plan must include a scaling trigger strategy — at what utilization level do you scale back up?

### Constraints

- **Current monthly cost**: $187,000 (detailed breakdown in story)
- **Current volume**: 450K transactions/day (post-UN announcement ramp)
- **Target volume**: 10M transactions/day (6-month horizon)
- **Break-even per transaction**: $0.003 (UN contract price: $0.0008; product price to corporate buyers: $0.004; blended: ~$0.003 break-even)
- **Compliance non-negotiables**: bi-temporal event log, SHA-256 hash chains, 3-copy durability for hot data, 5-year retention, UNFCCC replication
- **Right-sizing constraints**: RDS cannot be downsized below db.r5.2xlarge (minimum for event store workload); Kafka cannot go below 3 brokers
- **AWS region footprint**: us-east-1 (primary), eu-west-1 (UNFCCC/Swiss data residency), sa-east-1 (Brazil), ap-northeast-2 (South Korea/APAC)
- **Reserved instance budget**: CFO will approve 1-year reserved commitments for stable baseline; 3-year requires board approval

### Your Task

Produce a FinOps analysis and cost optimization plan for CarbonLedger. The output should be presentable to the CFO and Series B investor — show your math, be specific about optimizations, and model unit economics at scale.

### Deliverables

- [ ] Mermaid diagram: cost attribution architecture — which services serve which functions (compliance layer, transaction processing, CA fan-out, cold storage), annotated with monthly cost
- [ ] Cost model spreadsheet (as a markdown table): current costs, right-sized costs (immediate wins), and projected costs at 1M/day and 10M/day — with per-transaction unit cost at each level
- [ ] Right-sizing analysis: for each over-provisioned service, state current size, recommended size, monthly savings, and risk of downsizing
- [ ] Cross-region data transfer deep dive: model the actual data transfer cost at 10M/day including CA fan-out (4 copies), retry traffic (assume 3% failure rate requiring 1 retry), audit log writes, and read replica sync
- [ ] Top 3 optimization opportunities: specific architectural changes (not just resizing) that reduce per-transaction cost — e.g., batching CA notifications, tiered storage migration, spot instances for non-critical workloads
- [ ] Reserved instance analysis: which services to commit on 1-year vs. 3-year vs. keep on-demand; monthly savings from each commitment tier
- [ ] Scaling estimation (show math): at 10M/day with optimizations applied, what is the total monthly cost? What is the per-transaction cost? Does it hit $0.003?
- [ ] Tradeoff analysis (minimum 3): right-sizing immediately vs. waiting for growth to "fill" provisioned capacity; batching CA notifications for cost vs. 4-hour SLA for large transfers; on-demand vs. reserved for event store (RDS)

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
