---
**Title**: SentinelOps — Chapter 96: Advanced Observability — Distributed Tracing at 50M Spans/Day
**Level**: Staff
**Difficulty**: 8
**Tags**: #observability #distributed-tracing #opentelemetry #jaeger #grafana-tempo #cost-engineering #sampling
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 92 (Zero-Trust Architecture), Ch. 55 (LuminaryAI Observability), Ch. 56 (LuminaryAI Billing)
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**Date**: Wednesday, June 18 — 2:15 PM
**Channel**: #platform-observability

---

**Slack Thread — #platform-observability**

**[14:15] Priya Suresh (Platform Lead):** Monthly cloud cost review just finished. Distributed tracing infrastructure: $280,000/month. That's our single largest line item after compute. For context, our entire database fleet is $190k/month.

**[14:18] Nadia Cho (Engineering Manager):** What?? We deployed OpenTelemetry two quarters ago and nobody flagged this?

**[14:19] Priya Suresh:** The cost crept up. We started with 3M spans/day at launch. We're now at 50M spans/day because every new service that got added just had `OTEL_TRACES_SAMPLER=always_on` set by default. Nobody turned it down. 50M spans × average 2KB = 100GB of trace data per day. Jaeger's storage backend is an AWS OpenSearch cluster with 30 days retention.

**[14:22] Kenji Watanabe (Head of Engineering):** Can we turn it off? Keep metrics and logs, drop traces?

**[14:24] Yusuf Okafor (Head of Product Security):** No. We cannot turn it off. Three reasons. One: our SOC 2 Type II auditors specifically asked about our ability to reconstruct the execution path of any authorization decision within 72 hours. Traces are how we do that. Two: two of our FedRAMP-adjacent customers require it contractually. Three: last month, we had a P1 where a threat detection rule was silently dropping events. We found the root cause in 40 minutes because we had full trace context. Without traces, that investigation is 6-8 hours minimum.

**[14:27] Kenji Watanabe:** I'm not saying turn it off completely. I'm saying $280k/month is not survivable. That's $3.36M/year on tracing alone. I have to justify that to the CFO in two weeks. What can we do?

**[14:30] You:** The problem isn't that we have traces. The problem is that we're storing all of them at equal priority. A health check trace and a nation-state threat detection trace cost the same amount of storage. They should not.

**[14:32] Priya Suresh:** Tail-based sampling. I've been meaning to propose this for two months.

**[14:33] Kenji Watanabe:** How much does it save?

**[14:34] You:** Depends on the strategy. Let me model it and come back.

**[14:35] Kenji Watanabe:** You have 48 hours. I need numbers before the CFO prep.

---

**Email Chain**

**From**: Kenji Watanabe
**To**: You
**Subject**: Tracing Cost Analysis — CFO Meeting Prep
**Date**: Wednesday, June 18 — 3:00 PM

I need a written cost model by EOD Friday. The CFO will ask:

1. What is the current cost and why is it this high?
2. What is the proposed solution and what does it save?
3. What are we giving up? (Specifically: will we miss incidents?)
4. What is the implementation risk?

She is not technical but she is sharp. Be honest about the tradeoffs. If tail-based sampling means we miss 0.5% of slow requests, say that. Don't oversell.

— Kenji

---

**Slack DM — Priya Suresh → You**
*Wednesday, June 18 — 4:45 PM*

**Priya:** I did some math. Our 50M spans/day breaks down roughly like this based on what I can see in the OTel Collector pipeline:

- Health check / liveness probes: ~18M spans/day (36%)
- Background jobs / scheduled tasks: ~12M spans/day (24%)
- Internal service-to-service calls (non-critical path): ~8M spans/day (16%)
- User-facing API requests (normal): ~7M spans/day (14%)
- Threat detection critical path: ~4M spans/day (8%)
- Error traces / slow traces (>500ms): ~1M spans/day (2%)

The bottom two categories — the ones we actually care about — are 10% of total volume. The top three categories are 76% of volume and almost none of it has ever been opened in Jaeger by a human being.

**You:** So if we sample the top three at 1% and keep 100% of everything else...

**Priya:** We go from 50M to roughly 7.5M spans/day. That's an 85% volume reduction. Storage drops from ~100GB/day to ~15GB/day. Jaeger cost goes from $280k to roughly $42k/month. We save $238k/month.

**You:** Tail-based sampling or head-based?

**Priya:** Has to be tail-based. Head-based doesn't know if a request will be an error or slow until it's done. We'd end up dropping some errors if we sample at the head. For a security platform, that's not acceptable.

**You:** Tail-based sampling means we need to buffer the full trace before making a sampling decision. At 50M spans/day that's a non-trivial buffer. What collector are we using?

**Priya:** OTel Collector. The tail-sampling processor exists but it's not battle-tested at our volume. That's the implementation risk.

---

**Slack DM — You → Kenji Watanabe**
*Thursday, June 19 — 9:00 AM*

**You:** Modeling complete. Short version: we can get from $280k/month to approximately $40-50k/month through tail-based sampling without losing any error, slow request, or threat detection traces. The health check and background job traces — which nobody reads — are 60% of our cost.

**Kenji:** What's the catch?

**You:** The OTel Collector tail-sampling processor needs to buffer traces in memory before making a sampling decision. At 50M spans/day with an average trace duration of 2 seconds, we need roughly 50-100GB of in-memory buffer across the collector fleet. That's real infrastructure. The Collector fleet itself will cost maybe $8-12k/month. Net saving is still $220-230k/month.

**Kenji:** That's a good story for the CFO. What about Jaeger? Is it the right backend at 15GB/day sustained?

**You:** Honestly, probably not. Jaeger on OpenSearch was a reasonable choice at 3M spans/day. At 15GB/day with 30-day retention, we need 450GB of hot storage in OpenSearch. Grafana Tempo with S3 backend is significantly cheaper for that volume and has better query performance for our use case. Migration is a 2-month project.

**Kenji:** Can we start the migration after the CFO meeting?

**You:** Yes. The sampling changes can go in this sprint. The Tempo migration is a separate track.

---

### Problem Statement

SentinelOps's distributed tracing infrastructure has grown to 50 million spans per day at a cost of $280,000/month. The current implementation uses the OpenTelemetry SDK with `always_on` sampling across all services, storing everything in a Jaeger cluster backed by AWS OpenSearch. The CFO requires a cost reduction plan. Compliance and security requirements mean traces cannot be eliminated — only intelligently sampled.

You must design a tail-based sampling pipeline using the OTel Collector that reduces trace volume by 80-85% while preserving 100% of error traces, slow traces (>500ms), and all traces on the threat detection critical path. You must also evaluate Jaeger vs Grafana Tempo as the storage backend for the reduced volume.

### Explicit Requirements

1. Tail-based sampling strategy: 100% retention for errors, latency outliers (>500ms), and threat detection path traces; configurable sample rate (default 1%) for all other trace categories
2. OTel Collector pipeline redesign: collector agents on each node forward to a collector gateway tier that performs tail-sampling; gateway tier must buffer complete traces before making sampling decisions
3. Trace categorization at collection time: traces must be tagged with category (health-check, background-job, user-api, threat-detection, etc.) before sampling decisions are applied
4. Storage backend evaluation: Jaeger+OpenSearch vs Grafana Tempo+S3 for the reduced 15GB/day volume; cost comparison required
5. Exemplar linking: metrics (Prometheus) must link to representative trace IDs so engineers can jump from a metric spike to a trace without searching
6. Service graph generation: trace data must feed a service dependency graph (service calls, latency, error rates) that updates every 5 minutes
7. Trace retention: error traces and threat detection traces retained 90 days; all other sampled traces retained 30 days
8. Zero trace loss for the threat detection pipeline during collector fleet upgrades or restarts

### Hidden Requirements

- **Hint**: Re-read Priya's breakdown of trace categories. Health check / liveness probes are 36% of total volume (18M spans/day). What is generating health check traces? The Kubernetes liveness and readiness probes hit HTTP endpoints. Why are those generating OTel traces at all? Is there a simpler fix before tail-based sampling is even deployed?
- **Hint**: Re-read Yusuf's response: "our SOC 2 Type II auditors specifically asked about our ability to reconstruct the execution path of any authorization decision within 72 hours." This creates a requirement that authorization-path traces must be retained with forensic integrity — not just stored cheaply in S3. What metadata must be preserved on those traces to meet the 72-hour reconstruction SLA?
- **Hint**: Re-read the tail-based sampling buffer math: "50-100GB of in-memory buffer across the collector fleet." A collector gateway pod that holds 10GB of in-memory trace data and then crashes loses that data. What happens to the traces buffered in a crashed collector pod, and how does this interact with the "zero trace loss for threat detection" requirement?
- **Hint**: Re-read Priya's note: "The OTel Collector tail-sampling processor exists but it's not battle-tested at our volume." The tail sampling processor in the OTel Collector requires that all spans for a given trace land on the same collector instance (because the sampling decision is per-trace). At 50M spans/day across a collector fleet, how do you route spans from the same trace to the same collector gateway instance?

### Constraints

- **Current volume**: 50M spans/day, ~100GB/day
- **Target volume**: ~7-8M spans/day, ~15GB/day (after sampling)
- **Current cost**: $280,000/month
- **Target cost**: < $60,000/month (CFO requirement)
- **Trace duration**: average 2 seconds; max 30 seconds for complex threat detection pipeline traces
- **Collector buffer requirement**: ~50-100GB in-memory buffer across gateway tier
- **Retention**: 90 days for errors/security traces; 30 days for sampled normal traces
- **Services**: 340 microservices, each emitting OTel spans
- **Compliance**: SOC 2 Type II — authorization traces must be reconstructible within 72 hours
- **Latency impact**: sampling pipeline must not add more than 1ms of latency to the trace export path (async export, non-blocking)
- **Availability**: collector pipeline must have 99.9% availability; traces lost during collector downtime must not include threat detection path traces

### Your Task

Design the revised observability pipeline for SentinelOps. You must produce a tail-based sampling strategy with specific sample rates per trace category, an OTel Collector fleet architecture that supports tail sampling at scale, a storage backend recommendation with cost model, and an exemplar linking design that connects Prometheus metrics to representative traces.

### Deliverables

- [ ] **Mermaid architecture diagram** — OTel Collector pipeline: service SDK → node-level collector agent → gateway collector tier (tail sampling processor) → storage backends (Tempo/Jaeger) + exemplar sink (Prometheus remote write). Show the trace routing mechanism that ensures spans from the same trace_id land on the same gateway instance.
- [ ] **Sampling strategy table** — For each trace category (health-check, background-job, internal-service, user-api, threat-detection, error/slow): sample rate, retention period, storage tier (hot/warm/cold), and rationale
- [ ] **Scaling estimation** — Show full math for: (a) collector gateway fleet sizing (50M spans/day arriving, average span size 2KB, buffer required for 30-second max trace duration), (b) Grafana Tempo storage cost at 15GB/day × 90-day retention on S3 vs Jaeger+OpenSearch at same volume (use AWS S3 $0.023/GB-month vs OpenSearch i3.2xlarge nodes), (c) net monthly cost after migration
- [ ] **Tradeoff analysis** — Minimum 3 explicit tradeoffs:
  - Tail-based sampling vs head-based sampling for a security platform
  - Grafana Tempo (object storage backend) vs Jaeger (OpenSearch backend) at 15GB/day
  - OTel Collector gateway tier (stateful, buffering) vs per-service sampling (stateless, no buffer)
- [ ] **Exemplar linking design** — How a Prometheus metric (e.g., `threat_detection_latency_p99`) includes a `trace_id` exemplar pointing to a representative slow trace; include the Prometheus metric definition and the OTel span attribute that gets promoted to exemplar
- [ ] **Database schema** — Trace metadata index table for compliance queries (trace_id, service_name, tenant_id, category, sampled: bool, retention_tier, started_at, duration_ms, has_error: bool, span_count) — separate from raw trace storage, enables fast compliance queries without loading full trace data
