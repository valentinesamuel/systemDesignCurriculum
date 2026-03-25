---
**Title**: VertexCloud — Chapter 149: The Game Day That Became a Real Incident
**Level**: Staff
**Difficulty**: 9
**Tags**: #chaos-engineering #game-day #blast-radius #resilience #spike-design-review #incident-response #gremlin
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 88 (GigGrid feature flags rollback), Ch. 129 (BuildRight canary), Ch. 147 (VertexCloud GitOps)
**Exercise Type**: Spike — Design Review Ambush
---

### Story Context

**Design Review — Thursday 10:00**
**Presenter**: [you]
**Audience**: Yael Cohen, Reza Tehrani, Amara Chen, 3 platform engineers

You're presenting the chaos engineering game day design. The proposal: a controlled blast radius test of three non-critical services to validate circuit breaker behavior.

**Your proposal**: inject 500ms latency into the database connection pool of the `config-cache-service`, `metrics-aggregator`, and `feature-flag-service`. These three services are "non-critical" per the service classification in the IDP catalog — they don't handle customer data and don't sit on any revenue-critical path.

The game day is scheduled for today's 14:00 window.

---

**14:00 — Game Day begins**

Reza Tehrani, Yael Cohen, and two platform engineers are watching the dashboards. You initiate the latency injection through Gremlin. The 500ms latency starts hitting all three services simultaneously.

**14:03** — The `config-cache-service` circuit breaker opens. Requests start failing with fallback responses. Expected behavior.

**14:04** — The `metrics-aggregator` starts returning 503s. Expected.

**14:04:47** — Something unexpected: the authentication service starts showing elevated error rates. 8% of auth requests returning 500.

**[you]**: Pause the injection.

**Amara Chen**: *(already looking)* The auth service is making a synchronous call to `config-cache-service` to load authorization policy configs. The call is failing because `config-cache-service` is circuit-broken. Auth service has no fallback.

**[you]**: Why is the auth service calling config-cache? Auth is classified as critical. Config-cache is classified as non-critical.

**Amara Chen**: It's in the service map. Auth service was updated 3 months ago to load ABAC policies from a central config store. The config store is backed by config-cache-service. Nobody updated the service classification.

**14:06** — Eight production services are returning errors. Auth errors cascade: services that require auth on every request are now degrading.

**Reza Tehrani**: *(to you, very quietly)* This is a production incident.

**[you]**: Yes. Terminate the injection. Now.

**Amara Chen**: Injection terminated.

**14:08** — Auth service recovers. All 8 cascaded services recover.

**14:09** — Reza Tehrani sits back. "We just had a production incident during a controlled chaos engineering test. I want to know two things. First: what did the game day design miss? Second: what did the production incident tell us about our architecture?"

---

**Post-game-day review — Friday 09:00**

You present the analysis.

**[you]**: The game day design missed one thing: the dependency graph was wrong. The IDP's service catalog showed config-cache-service as non-critical. It didn't show that auth-service depended on config-cache-service. An undocumented synchronous dependency in a critical-to-non-critical direction.

**Yael Cohen**: Why was the dependency undocumented?

**[you]**: The auth service engineer who made the change 3 months ago updated the code and the deployment config but not the service catalog. No process enforced catalog update as part of the deployment.

**Reza Tehrani**: And the second question — what did the real incident tell us?

**[you]**: The auth service has no fallback for a failed policy config load. Every auth-required request fails completely if the config cache is unavailable for even 30 seconds. That's a design problem that chaos engineering revealed — and that we would not have found without the game day.

**[you]**: The game day succeeded at its actual purpose: finding architectural weaknesses. The problem is we found one we didn't intend to find, in a way that affected production. The right response is: improve the game day design process AND fix the auth service fallback.

**Amara Chen**: The game day design process improvement is: never inject into a service without verifying its full upstream and downstream dependency graph. Not just the documented graph — the live traffic graph from the API gateway and service mesh.

---

### Problem Statement

VertexCloud's first chaos engineering game day triggered a real production incident by injecting latency into a service that had an undocumented synchronous dependency from the authentication service. The game day succeeded at finding an architectural weakness (auth has no fallback for config failures) but failed to control blast radius. Design the correct chaos engineering methodology and fix the underlying architectural weakness.

---

### Explicit Requirements

1. Pre-game-day checklist: dependency graph verification using live traffic data (not just service catalog)
2. Service classification validation: automated check that service catalog is consistent with live service-to-service traffic
3. Auth service fallback design: auth service must handle config-cache-service unavailability without total auth failure
4. Chaos experiments must have defined abort criteria: specific metrics that automatically stop the experiment
5. Game day runbook: documented pre-game-day, during, and post-game-day procedures any engineer can follow
6. Recurring game days: once per quarter, rotating services, documented learning from each

---

### Hidden Requirements

- **Hint**: Re-read Amara Chen's finding: "the auth service engineer updated the code but not the service catalog." This is a process gap — deployment doesn't require catalog update. Your game day process improvement must include a catalog validation step in CI: when a service is deployed with a new external call (new HTTP client, new Kafka consumer), the CI pipeline should detect it and require a service catalog update. This is preventive, not reactive.

- **Hint**: Reza Tehrani's second question was the more important one: "what did the production incident tell us about our architecture?" The auth service having no fallback for a configuration load failure is a critical resilience gap that would have caused a production incident eventually — with or without a game day. Your postmortem should reflect this: the game day was successful because it found this weakness in a controlled setting. The design must support this framing.

---

### Constraints

- Game day blast radius: limited to services classified as "non-critical" in the IDP catalog
- Service catalog accuracy: currently 60% accurate (40% of services have undocumented dependencies)
- Auth service: handles 100% of authentication for all VertexCloud products; zero-downtime SLA
- Circuit breaker coverage: 65% of services have circuit breakers; auth service does not
- Post-incident RCA: 5 calendar days to produce postmortem (VertexCloud standard)

---

### Your Task

Design the correct chaos engineering methodology for VertexCloud and fix the auth service's architectural weakness.

---

### Deliverables

- [ ] Game day design checklist: pre-game-day dependency graph validation, abort criteria definition, rollback plan
- [ ] Dependency graph validation: how to extract live service-to-service call graph from service mesh + API gateway (vs relying on static catalog)
- [ ] Blast radius estimation tool: given an injection target, identify all services that have a dependency path to that target
- [ ] Auth service fallback design: stale config cache (last known good policy) + policy evaluation failure mode (fail open vs fail closed — with justification)
- [ ] Circuit breaker coverage plan: identify services without circuit breakers, prioritized by criticality
- [ ] Service catalog accuracy improvement: CI check for undocumented external calls
- [ ] Postmortem template: the 5-section postmortem for the game day incident (timeline, contributing factors, impact, corrective actions, lessons learned)
- [ ] Quarterly game day program: structure, rotation strategy, learning documentation
- [ ] Tradeoff analysis (minimum 3):
  - Fail open (allow requests even without policy) vs fail closed (deny all) for auth fallback — security vs availability
  - Chaos injection in production vs staging — production finds real dependencies, staging is safe — the right answer for VertexCloud
  - Automated chaos (Chaos Monkey-style, always on) vs scheduled game days — continuous resilience testing vs controlled learning
