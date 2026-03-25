---
**Title**: TeleNova — Chapter 121: 200 Million Spans Per Day — Tail Sampling and Cost Control
**Level**: Staff
**Difficulty**: 8
**Tags**: #opentelemetry #distributed-tracing #tempo #tail-sampling #cost-engineering #observability #sampling
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 69 (Stratum tracing intro), Ch. 96 (SentinelOps tracing), Ch. 106 (Crestline observability)
**Exercise Type**: System Design
---

### Story Context

**Slack Thread — #platform-engineering**
**Tuesday 14:30**

**Marcus Lin (Finance Business Partner)**: Monthly cloud cost review just completed. Infrastructure team, I'm going to need someone to explain a line item to me.

**Marcus Lin**: Observability infrastructure (Grafana Cloud + Tempo storage): €340,000/month. That's our 3rd largest line item behind compute and database. Last year it was €80,000/month. That's a 4x increase in 12 months.

**14:35**
**Jin-ho Park**: That's because we deployed OpenTelemetry across all 60 services last year. Traces go everywhere.

**Marcus Lin**: I see that. I'm asking: is €340,000/month the right number for traces?

**14:38**
**Yuki Tanaka (Head of Security)**: Before anyone talks about reducing tracing, I want to flag something. During last month's network security incident, the traces for the affected call path were not available. I looked into why. Those traces had been sampled away by head-based sampling. We had a 1% sample rate. The exact requests I needed to investigate were in the 99% that got dropped.

**14:41**
**Marcus Lin**: So we're paying €340k/month and still missing the traces we actually need?

**Yuki Tanaka**: That is correct.

**14:43**
**Alejandro Reyes**: Someone fix this. I want a design by end of week.

---

**DM — Yuki Tanaka → [you] — Wednesday 09:00**

**Yuki Tanaka**: I want to be specific about my requirements before you design anything.

**[you]**: Please.

**Yuki Tanaka**: I need complete trace retention for three categories of requests:
1. Any request that results in an HTTP 5xx or 4xx response
2. Any request touching the SIM authentication, eSIM activation, or billing services (these are our highest-security paths)
3. Any request with p99 latency > 2x baseline (anomalous slow requests are attack vectors)

For everything else, I'm flexible on sampling rates.

**[you]**: Can you quantify what percentage of requests fall into those three categories?

**Yuki Tanaka**: I pulled the last 30 days. Errors: 0.3% of requests. Security-critical paths: 8% of request volume. Slow requests (>2x baseline): 2.1%. There's some overlap. Total: roughly 10% of requests must be retained at full resolution.

**[you]**: So 10% must be retained, 90% can be sampled. But the decision about which 10% cannot be made at the head of the request — you don't know it's an error or slow request until it completes.

**Yuki Tanaka**: Correct. That's the problem with head-based sampling.

**[you]**: You need tail-based sampling. The sampling decision is made after the trace is complete, based on what actually happened.

**Yuki Tanaka**: Will that reduce costs?

**[you]**: Significantly. 90% of traffic at low sampling rates, 100% retention for the 10% that matters. Projected cost reduction: 65-70%.

**Yuki Tanaka**: Design it.

---

**DM — Marcus Lin → [you] — Wednesday 14:15**

**Marcus Lin**: I saw the Slack conversation. What's the projected cost after the redesign?

**[you]**: Rough math: 200M spans/day × 2KB average = 400GB/day. Current setup retains 100% in Grafana Cloud Tempo: €340k/month. Target: retain 10% fully (40GB/day) + 1% sample of remaining (3.6GB/day) = 43.6GB/day. Tempo storage for 43.6GB/day × 30-day retention = 1.3TB. At Grafana Cloud pricing, roughly €85-100k/month.

**Marcus Lin**: €240,000/month in savings?

**[you]**: Give or take. Plus reducing OTel Collector CPU costs because we're not shipping 90% of spans to Tempo at all.

**Marcus Lin**: That's a meaningful number. Get me the detailed design.

---

### Problem Statement

TeleNova's observability costs have grown 4x in 12 months due to head-based sampling that retains 100% of 200M daily spans. Simultaneously, the security team cannot investigate incidents because the relevant traces are being sampled away. A tail-based sampling architecture must reduce cost by 65% while guaranteeing 100% retention for error traces, security-critical paths, and anomalous slow requests.

---

### Explicit Requirements

1. 100% trace retention for: error responses (4xx/5xx), security-critical services (SIM auth/billing), requests with p99 > 2x baseline
2. ≤ 1% sampling for all other traffic
3. Sampling decision must be made after request completion (tail-based, not head-based)
4. Reduce monthly observability cost from €340k to ≤ €110k
5. Traces must be queryable within 60 seconds of request completion (current SLA for security investigations)
6. The OTel Collector pipeline must survive a collector node failure without losing security-critical traces

---

### Hidden Requirements

- **Hint**: Tail-based sampling requires buffering the full trace somewhere until the sampling decision can be made. For a 200M span/day pipeline, that buffer is substantial. Re-read the constraint: "queryable within 60 seconds." How long must the buffer hold traces before making a sampling decision? And what happens if the tail sampling processor crashes with a full buffer?

- **Hint**: Yuki identified that 8% of requests touch security-critical paths. But TeleNova has 120M subscribers. If a network security incident triggers correlated 5xx errors across many of those subscribers simultaneously, your tail-based sampler must retain 100% of a sudden spike in errors without dropping traces or overwhelming storage. What is your overflow strategy?

---

### Constraints

- 200M spans/day across 60 services
- Average span size: 2KB → 400GB/day raw
- Current retention: 30 days in Grafana Cloud Tempo
- Tail sampling buffer requirement: must hold ~30 seconds of spans in memory (to await trace completion)
- OTel Collector fleet: 12 nodes, must handle node failure gracefully
- Target cost: ≤ €110k/month

---

### Your Task

Design the tail-based sampling architecture for TeleNova's distributed tracing pipeline.

---

### Deliverables

- [ ] Mermaid architecture: services → OTel Collector (head) → tail sampling processor → Tempo + long-term archive
- [ ] Tail sampling rules: sampling policy configuration for each retention category
- [ ] Buffer design: trace buffer sizing math (spans/second × expected trace duration × retention period = buffer size)
- [ ] Cost model: projected storage volume after tail sampling, cost at Grafana Tempo pricing
- [ ] Collector fleet design: how to handle trace affinity (all spans for a trace must go to the same tail sampler)
- [ ] Overflow strategy: what happens when errors spike and the 10% retention budget is exceeded temporarily?
- [ ] Tradeoff analysis (minimum 3):
  - Tail-based vs head-based sampling — complexity vs observability guarantees
  - Grafana Tempo vs Jaeger vs self-hosted Zipkin — cost and operational overhead at this scale
  - 30-day retention vs tiered retention (30 days hot + 1 year cold) — security investigation requirements vs cost
