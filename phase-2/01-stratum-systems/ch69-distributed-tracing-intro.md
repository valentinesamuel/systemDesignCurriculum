---
**Title**: Stratum Systems — Chapter 69: Distributed Tracing with OpenTelemetry
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #observability #distributed-tracing #opentelemetry #jaeger #sampling
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 96, Ch. 121
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**Email Chain**

---
**From**: operations-team@dhl-enterprise.com
**To**: enterprise-support@stratum.io; priya.nair@stratum.io
**Subject**: RE: RE: RE: 47 Clearance Delays — Escalation to Executive Level
**Date**: [Week 6 at Stratum]

Priya,

I have been patient. Your team has told me "the system is operating within SLA" three times in the past two weeks. I am looking at my operations dashboard right now. It shows 47 delayed customs clearances in 72 hours for shipments routed through your Hamburg processing pipeline.

Your status dashboard says "All systems operational."

These two facts cannot both be true.

I am going to be very direct: DHL processes €2.4 billion in freight through your platform per year. If this is not resolved with a root cause explanation by Friday, I will be escalating to our procurement team for contract review.

Werner Hoffmann
Head of Digital Operations, DHL Enterprise

---
**From**: priya.nair@stratum.io
**To**: you@stratum.io
**Subject**: FWD: Werner's email
**Date**: [same day]

This is yours to fix. Werner is right — our dashboard is lying to us.

The problem: we have 14 DCs, 240 partner APIs, and 8 internal microservices in the clearance pipeline. When something slows down, we can't tell which service is actually slow. Our logging shows us what happened. It doesn't show us *why* or *where*.

We need distributed tracing. I should have done this 18 months ago.

— P

---

**#engineering-general**
`Thursday 11:23` **you**: I've been looking at the clearance delay pattern. I traced one specific shipment (DHL container C-883421) through our logs manually. It took 4 hours from "events received" to "clearance submitted." The pipeline should take 3 minutes.
`11:24` **lars.eriksson**: where's the time going?
`11:25` **you**: I can't tell. I have timestamps in each service's logs but they're not correlated. I can see that the Lisbon DC received the event, that the Frankfurt DC processed it, that the customs API gateway submitted it. But the 4-hour gap is... somewhere in the middle. Four services, no correlation IDs.
`11:26` **priya.nair**: that's the problem I've been waiting for someone to say out loud for 18 months
`11:27` **amara.chen**: we tried to add tracing in 2022. gave up because partner APIs in Malaysia and Indonesia don't accept W3C Trace-Context headers. they strip them.
`11:28` **you**: that's a known problem. there are fallback strategies.
`11:29` **amara.chen**: tell us one
`11:30` **you**: you use a baggage propagation fallback — encode the trace context in a query parameter or a vendor-specific header that partners tolerate. you lose some correlation fidelity but you keep the trace chain.
`11:31` **priya.nair**: I knew there was a reason we hired you

**DM — Marcus Webb → You**
`Friday 09:15`
**marcus.webb**: distributed tracing is one of those things where 80% of the value comes from getting the first 20% right. don't let perfect be the enemy of useful.
**you**: what's the 20%?
**marcus.webb**: trace context propagation across service boundaries. everything else — sampling, storage, visualization — is downstream of that. if you can't propagate context, you have correlated logs, not traces. those are different things.
**you**: what's the difference?
**marcus.webb**: correlated logs: you find the log entries for one request. distributed traces: you understand the *causal structure* of why that request behaved the way it did. one is search. the other is understanding.

---

### Problem Statement

Stratum's clearance pipeline passes through 8 internal services and 240 partner APIs before submitting to national customs systems. When a clearance is delayed, the team cannot identify which service caused the delay because there is no trace correlation across service boundaries. Partner APIs in some countries don't support W3C TraceContext headers, creating gaps in the trace chain.

You need to design and implement a distributed tracing system using OpenTelemetry that provides end-to-end visibility across the clearance pipeline, including partner APIs that don't support standard trace propagation.

---

### Explicit Requirements

1. All 8 internal services must emit spans with the same trace context
2. Cross-service trace context must propagate via W3C TraceContext where supported
3. Partner APIs that don't support W3C TraceContext must have a fallback propagation strategy
4. Sampling strategy must reduce trace storage costs while preserving 100% of traces for failed/slow clearances
5. Traces must be queryable by: shipment_id, partner_id, DC region, and error type
6. The tracing system must not add more than 5ms to the clearance pipeline P99 latency

---

### Hidden Requirements

- **Hint**: Re-read Amara's message about "partner APIs in Malaysia and Indonesia." Some partner APIs are not HTTP — they use EDIFACT (a structured text format from the 1980s used in international trade). How do you propagate trace context through a non-HTTP protocol?

- **Hint**: Marcus Webb said "you lose some correlation fidelity but you keep the trace chain" for the fallback strategy. What happens to trace analysis when 30% of spans have fallback propagation vs W3C propagation? How does your sampling strategy need to account for incomplete traces?

---

### Constraints

- 8 internal services across 14 DCs
- 240 partner API integrations: ~80% support HTTP/REST, ~15% use SOAP, ~5% use EDIFACT
- ~60% of partner APIs support W3C Trace-Context; ~40% do not
- Clearance pipeline: ~85,000 clearances/day = ~1 trace/sec average
- Estimated trace size: 8 internal spans + variable partner spans = ~2KB per trace
- Trace storage budget: $15k/month maximum (at $0.10/GB/month compressed)
- Jaeger or Grafana Tempo (existing infrastructure preference)

---

### Your Task

Design the OpenTelemetry-based distributed tracing system for Stratum's clearance pipeline. Include context propagation strategy for non-compliant partner APIs, sampling design, and the trace storage architecture.

---

### Deliverables

- [ ] Mermaid diagram: Trace propagation flow through all 8 services, showing W3C path and fallback path
- [ ] Sampling strategy: Tail-based sampling configuration — what triggers 100% sampling vs probabilistic sampling? Define the slow/error thresholds.
- [ ] Fallback propagation: Design for non-W3C partner APIs (query parameter? custom header? EDIFACT segment?)
- [ ] Storage calculation: At 85k clearances/day × 2KB/trace, what's the monthly storage cost? After tail-based sampling (assume 5% base rate + 100% error rate), what does storage drop to?
- [ ] Jaeger/Tempo schema: Index design for querying by shipment_id, partner_id, DC region
- [ ] Tradeoff analysis (minimum 3):
  - Head-based sampling vs tail-based sampling for this use case
  - Jaeger vs Grafana Tempo at 85k traces/day scale
  - In-process instrumentation vs auto-instrumentation for the 8 internal services
- [ ] TypeScript interface: `TraceContext` with W3C fields and Stratum-specific baggage (shipment_id, dc_region, partner_id)
