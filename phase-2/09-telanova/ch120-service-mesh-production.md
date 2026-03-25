---
**Title**: TeleNova — Chapter 120: Service Mesh at Production Scale — 60 Services, 18 Months of Istio
**Level**: Staff
**Difficulty**: 8
**Tags**: #service-mesh #istio #mtls #traffic-management #sidecar #envoy #latency #production
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 83 (TradeSpark mTLS intro), Ch. 93 (SentinelOps service mesh), Ch. 139 (PrismHealth mTLS)
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**Slack DM — Alejandro Reyes (CTO) → [you] — Wednesday 09:14**

**Alejandro Reyes**: I have a recurring meeting on my calendar called "Istio Death Watch." I did not create this meeting. Do you know who did?

**[you]**: I didn't know this meeting existed.

**Alejandro Reyes**: It's been running for 6 months. 8 attendees, all engineers. They meet weekly to discuss whether to rip out Istio. I found it by accident when I was looking at calendar conflicts.

**[you]**: Who's the organizer?

**Alejandro Reyes**: Jin-ho Park.

---

**Meeting: Istio Death Watch — Thursday 14:00**
**Attendees**: Jin-ho Park (Staff Engineer), 7 senior engineers

Jin-ho has been running this meeting since Istio caused a SIM activation latency spike 6 months ago. You ask to join. He accepts, warily.

**Jin-ho Park**: *(presenting data)* The Envoy sidecar adds 12ms to every internal hop. Our SIM activation path makes 14 internal service calls. That's 168ms in added overhead. ETSI TS 102 221 requires SIM activation to complete in under 200ms. We had a two-week period last quarter where our p99 was 216ms. We were in spec violation.

**[you]**: What's the failure mode when you're over 200ms?

**Jin-ho Park**: eSIM activations timeout and retry. If we're marginally above 200ms, a device will retry activation 3 times. So we go from one activation call to four. This is how a 10% latency increase becomes a 400% call volume increase.

**[you]**: Did anyone instrument which part of the 12ms is which?

Jin-ho looks at you. *(pause)* "No. We just know it's Envoy."

**[you]**: I want to see the detailed Envoy metrics before we decide anything.

---

**Analysis — Friday, posted in #platform-architecture**

You pull the Envoy stats from Prometheus. The 12ms breaks down as:

- mTLS handshake per connection: 8.4ms (new connections; persistent connections cost 0.2ms)
- Wasm filter evaluation (3 custom filters deployed): 2.1ms each, but called sequentially = 3.1ms total
- Circuit breaker evaluation overhead: 0.4ms
- Connection pool overhead: negligible

**Key finding**: The 8.4ms is from new connections per request. TeleNova's services are not reusing connections — they're opening new TLS connections for every service call. At 14 hops in the SIM activation path, if any of those hops creates a new connection, the handshake cost accumulates.

You also discover a secondary problem: TeleNova deployed 3 Wasm filters from a vendor 8 months ago for security telemetry. Two of them are no longer needed (the vendor was replaced). They're still running on every request.

**[you]** *(posting in Slack)*: Before anyone rips out Istio, I'd like to propose 3 configuration changes. Projected latency savings: 9ms of the 12ms overhead. The 3ms that remains is unavoidable mTLS cost.

**Jin-ho Park**: *(5 minutes later)* What are the 3 changes?

---

### Problem Statement

TeleNova's Istio service mesh is adding 12ms per service hop, causing SIM activation p99 latency to exceed the ETSI 200ms spec during peak periods. The mesh was deployed 18 months ago without detailed performance profiling. A systematic analysis reveals the overhead is primarily from connection reuse failures and unnecessary Wasm filters — both fixable without removing Istio.

---

### Explicit Requirements

1. Reduce per-hop Envoy overhead from 12ms to ≤ 4ms without removing Istio
2. SIM activation p99 must remain under 200ms under normal load and 220ms at peak (20% above SLA)
3. mTLS must remain enabled on all internal service communication (GDPR and ETSI security requirements)
4. Wasm filter audit: remove unused filters, minimize active filter evaluation overhead
5. Design must handle 120M subscribers × peak activation events (e.g., Black Friday SIM flash sales)
6. Istio control plane stability: xDS push frequency must not cause control plane instability during configuration changes

---

### Hidden Requirements

- **Hint**: Re-read Jin-ho's escalation: "a 10% latency increase becomes a 400% call volume increase" through retries. This is a retry amplification problem. Your mesh configuration must include retry budgets that prevent retry storms from turning a marginal SLA breach into a cascading failure. This is not mentioned in the problem statement.

- **Hint**: Alejandro Reyes created this situation indirectly — he approved deploying Istio but never approved a "mesh governance" process. The vendor Wasm filters were deployed by one team without organization-wide awareness. Your solution must include a governance proposal, not just technical fixes.

---

### Constraints

- 60 microservices in the mesh
- SIM activation path: 14 internal service hops
- Current overhead: 12ms per hop (168ms total on the critical path)
- ETSI TS 102 221: SIM activation ≤ 200ms p99
- Peak activation throughput: 50,000 activations/second (major SIM flash sales)
- Control plane: Istio 1.18, istiod, 3 worker nodes for the control plane
- Current Wasm filters: 3 deployed, 2 confirmed unused

---

### Your Task

Diagnose and resolve TeleNova's Istio latency problem without removing the service mesh.

---

### Deliverables

- [ ] Envoy latency breakdown: annotated breakdown of the 12ms per hop with sources
- [ ] Connection pool configuration: DestinationRule settings for HTTP/1.1 keepalive vs HTTP/2 multiplexing on the SIM activation path
- [ ] Wasm filter audit results and remediation: which to remove, which to optimize
- [ ] Retry budget design: retry policy that limits amplification to ≤ 2x volume under latency degradation
- [ ] Circuit breaker tuning: threshold configuration for the 14-hop activation path
- [ ] Mesh governance proposal: process for approving new Wasm filters, configuration changes, and mesh policy updates
- [ ] Post-remediation latency model: projected p99 after changes, with breakdown of remaining 3ms unavoidable overhead
- [ ] Tradeoff analysis (minimum 3):
  - HTTP/2 multiplexing vs HTTP/1.1 keepalive for high-frequency internal calls
  - Ambient mesh (no sidecar) vs sidecar for latency-sensitive paths
  - Wasm filters vs external policy enforcement — the tradeoff between filter chain flexibility and latency
