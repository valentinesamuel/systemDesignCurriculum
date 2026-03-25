---
**Title**: PrismHealth — Chapter 139: mTLS for HIPAA — Every Service Call Is a PHI Flow
**Level**: Staff
**Difficulty**: 8
**Tags**: #service-mesh #mtls #hipaa #phi #audit-trail #compliance #healthcare #istio
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 83 (TradeSpark mTLS), Ch. 93 (SentinelOps service mesh), Ch. 120 (TeleNova service mesh)
**Exercise Type**: System Design
---

### Story Context

**Pre-Audit Assessment — Question 23**

Dr. Sarah Kim, CIO at Massachusetts General Hospital (MGH), is conducting a pre-audit review of PrismHealth before the hospital renews their enterprise contract. She has sent a 47-question security questionnaire. You are working through it with Priya Nair and Jake Huang.

Question 23: "Provide evidence that all service-to-service communications involving Protected Health Information (PHI) are encrypted in transit, and that access to PHI-containing services is logged with the requesting service's identity."

**Priya Nair**: We have mTLS. Istio deployed 8 months ago.

**Jake Huang**: Most services are on Istio. We have a few that opted out.

**Priya Nair**: How many?

**Jake Huang**: *(looking at his laptop)* Eleven services opted out. The reason cited in every case is "sidecar latency overhead." The alert evaluation service specifically — which processes all the vital sign readings and makes the alert decisions — opted out because the Envoy overhead was affecting their sub-100ms evaluation SLA.

**[you]**: Does the alert evaluation service handle PHI?

**Jake Huang**: *(pause)* Yes. It receives patient readings with patient IDs. It queries patient history. It writes alert records with patient identifiers.

**Priya Nair**: So our highest-criticality service, which processes PHI for every patient, is not behind mTLS.

**Jake Huang**: It's behind network ACLs. Traffic is still encrypted — we use TLS at the application layer for external calls.

**Priya Nair**: HIPAA requires demonstrable encryption in transit AND access logging with identity. Network ACLs don't give us service-level identity logging. We can show "this IP address accessed that service" but not "the alert-evaluation service accessed the patient-history service."

**[you]**: Dr. Kim can't sign off on Question 23 with that answer.

---

**Slack DM — Jake Huang → [you] — Tuesday 16:00**

**Jake Huang**: I've been thinking about the mTLS problem. The reason we opted out of Istio for the alert evaluation service is real — the sidecar adds 12ms and our SLA is 100ms. After the TradeSpark work I read in the shared library, I think the issue is connection reuse, not mTLS itself. But I've never profiled it properly.

**[you]**: What's the latency breakdown?

**Jake Huang**: I don't have it. Nobody profiled it when they made the opt-out decision. They just measured the end-to-end increase and made the call.

**[you]**: Let me spend a day profiling Envoy on a staging version of the service before we decide anything. The opt-out may have been the right call. Or it may have been based on incomplete data.

**Jake Huang**: If it turns out the overhead is primarily new connections — and we can fix it with HTTP/2 keepalive — we could get mTLS back on the alert service in a week.

**[you]**: That's the hypothesis. Let's test it.

---

**Profiling results — Wednesday afternoon**

Envoy sidecar overhead on the alert evaluation service: 11ms. Breakdown:
- New TLS connection handshake: 8.1ms
- Envoy filter evaluation: 2.4ms
- Connection pool overhead: 0.5ms

The alert evaluation service makes 14 calls per alert evaluation — all to the same 3 downstream services. And all 14 calls open new TLS connections. No keepalive configured.

**Jake Huang**: *(reading the profiling output)* The fix is `idleTimeout` and `h2UpgradePolicy`. With HTTP/2 multiplexing on persistent connections, those 14 downstream calls share one connection. The 8.1ms handshake happens once, not 14 times.

**[you]**: Correct. Projected overhead after fix: ~1.8ms total instead of 11ms. Well within the 100ms SLA.

**Jake Huang**: So the opt-out decision was made based on a configuration mistake, not an inherent Istio limitation.

**[you]**: Yes. But the broader problem remains: we have 11 services that opted out, and we don't know if all of them had the same root cause. We need an audit.

---

### Problem Statement

Eleven PrismHealth services are excluded from the Istio service mesh, including the alert evaluation service that processes PHI for every patient. MGH's pre-audit review requires evidence that all PHI-handling service-to-service calls are encrypted with service identity logging. The exclusion was based on incomplete latency profiling. A systematic mTLS compliance program is needed.

---

### Explicit Requirements

1. All services handling PHI must be enrolled in Istio mTLS with STRICT mode
2. Every service-to-service call involving PHI must be logged with source service identity and target service identity
3. Alert evaluation service must maintain ≤ 100ms p99 alert evaluation latency with mTLS enabled
4. Mesh enrollment of all 11 opted-out services must complete before MGH's audit in 6 weeks
5. PHI data flow map: document which services handle PHI vs which handle non-PHI (used for audit response)
6. HIPAA access audit log: queryable record of PHI service access by service identity for the last 12 months

---

### Hidden Requirements

- **Hint**: Re-read Jake's original opt-out reason: "sidecar latency overhead affecting sub-100ms evaluation SLA." The profiling revealed this was a connection reuse configuration mistake. But what about the other 10 opted-out services? Some may have legitimate latency constraints. Your audit process must profile each one before assuming the fix is the same. What's your approach when you find a service where connection reuse doesn't solve the problem?

- **Hint**: HIPAA audit logging requires demonstrating "who accessed what when." For service mesh access logs, "who" is the source service's mTLS identity (SPIFFE URI). But if the HIPAA auditor asks "show me all accesses to patient history records by the alert evaluation service in the last 90 days" — the mesh access log gives you service-to-service calls, but not which specific patient records were accessed. What additional logging is required?

---

### Constraints

- 47 services total in the platform, 11 currently opted out of mTLS
- Alert evaluation service SLA: ≤ 100ms p99 (was 100ms before mTLS, was 111ms with naive Istio enrollment)
- Projected overhead after HTTP/2 keepalive fix: ≤ 2ms per service (acceptable)
- MGH audit: 6 weeks away — all 11 services must be enrolled before then
- HIPAA audit log retention: 6 years
- PHI surface: must identify which of 47 services handle PHI vs which are purely operational

---

### Your Task

Design PrismHealth's mTLS compliance remediation program and HIPAA service audit trail.

---

### Deliverables

- [ ] PHI data flow map: list of all 47 services with PHI vs non-PHI classification and justification
- [ ] Istio enrollment plan: per-service latency profiling approach, fix for HTTP/2 connection reuse, enrollment sequencing
- [ ] Alert evaluation service configuration: specific Istio DestinationRule settings for HTTP/2 keepalive
- [ ] Service mesh audit log design: what fields are captured for each service call (source identity, target, timestamp, response code)
- [ ] HIPAA access audit layer: application-level logging that records patient record access (not just service-to-service calls)
- [ ] MGH Question 23 answer: draft the technical response to the audit questionnaire question
- [ ] AuthorizationPolicy design: which services are authorized to call which PHI-handling services (least-privilege)
- [ ] Tradeoff analysis (minimum 3):
  - STRICT mTLS mode vs PERMISSIVE mode during enrollment — security gap vs migration risk
  - Service mesh access logs vs application-level HIPAA audit logs — what each covers, what each misses
  - Ambient mesh (no sidecar) vs sidecar for latency-sensitive PHI services — the HIPAA compliance equivalence question
