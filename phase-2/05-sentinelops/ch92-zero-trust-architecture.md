---
**Title**: SentinelOps — Chapter 92: Zero-Trust Architecture
**Level**: Staff
**Difficulty**: 9
**Tags**: #security #zero-trust #network-segmentation #blast-radius #mtls #kubernetes
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 83, Ch. 93, Ch. 139
**Exercise Type**: Incident Response / System Design
---

### Story Context

**Security Researcher Disclosure — Responsible Disclosure**

**From**: sec-research@trustsec.io
**To**: security@sentinelops.io
**Subject**: Responsible Disclosure — Internal Lateral Movement via Compromised Pod
**Date**: [Week 2 at SentinelOps]

SentinelOps Security Team,

During authorized penetration testing of our own SentinelOps tenant environment, our red team discovered a vulnerability in SentinelOps's internal infrastructure that we believe you would want to know about.

Specifically: after obtaining remote code execution within one of our tenant's sandboxed analysis pods (via a known deserialization vulnerability in a third-party library), our red team was able to make internal HTTP calls to SentinelOps's core API services — including the threat intelligence aggregation API and the alert enrichment service — from inside the compromised pod.

The attack path:
1. Exploit deserialization vulnerability in analysis-runner pod
2. From compromised pod: `curl http://threat-intel-api.sentinelops.svc.cluster.local/v1/internal/indicators`
3. Response received: 200 OK with full threat intel database response

There is no mTLS, no network policy, and no service authentication between pods. Any pod can call any service.

The irony of a security platform with no internal security is not lost on us. We're sharing this because we believe in the work you're doing and want you to fix it.

Marcus Rivera, Chief Red Team Officer, TrustSec

---

**War Room — SentinelOps, 09:00 AM**

**Dr. Aisha Kamara**: *(one word on Slack)* "Design."

She means: redesign the internal security architecture. You have a week to present the plan.

**#security-response**
`09:15` **you**: I've read the disclosure. the attack is correct and the path is exactly as described. the root cause is: flat network topology with no internal trust model.
`09:17` **dmitri.volkov** *(Staff Engineer)*: we've been saying this for 2 years. every quarterly review I put "implement network policies" on the roadmap. it keeps getting bumped.
`09:18` **neha.gupta** *(Principal Engineer)*: I've been saying it too. the reason it gets bumped is: it requires a complete re-architecture of service-to-service communication. it's not a one-week fix.
`09:20` **you**: it's a 12-week fix with the right design. but we need to prevent the attack surface from getting worse immediately, while we do the proper redesign.
`09:22` **dr.kamara**: what's the immediate mitigation?
`09:23` **you**: Kubernetes NetworkPolicy. default-deny-all for ingress. every service explicitly allows only its known callers. takes 2 days to implement if we have the caller map. do we have a caller map?
`09:25` **dmitri.volkov**: ...no
`09:26` **neha.gupta**: I have an unofficial one from 2023. it's out of date.
`09:27` **you**: we'll build one from service mesh traffic logs. another reason we need a service mesh.
`09:28` **dr.kamara**: you have 2 days for the immediate mitigation. 12 weeks for the full zero-trust redesign. go.

---

### Problem Statement

SentinelOps's internal network has no segmentation, no service authentication, and no trust boundaries. A compromised pod can reach any internal service. For a security platform with 8,000 enterprise customers, this is a critical vulnerability — if an attacker pivots from a customer-facing component to internal services, they could access threat intelligence data for all 8,000 customers.

You need to design a zero-trust architecture that eliminates lateral movement by requiring explicit authentication and authorization for all service-to-service communication.

---

### Explicit Requirements

1. Default-deny-all networking: services cannot communicate unless explicitly permitted
2. All service-to-service communication must be mutually authenticated (each side proves its identity)
3. Authorization must be based on service identity, not network location
4. Analysis runner pods (tenant-accessible) must be strictly isolated from core platform services
5. Blast radius of any single compromised pod must be limited to services that pod is explicitly authorized to call
6. The redesign must be deployable in 12 weeks without production downtime

---

### Hidden Requirements

- **Hint**: Re-read the attack path: `curl http://threat-intel-api.sentinelops.svc.cluster.local/v1/internal/indicators`. The URL has "internal" in the path — suggesting this endpoint was not intended to be publicly accessible. But in a flat network, there's no enforcement. What's the correct principle: should "internal" APIs rely on network access control to protect them, or should they require authentication regardless of network origin?

- **Hint**: The disclosure mentions "analysis-runner pod." This is a tenant-isolation boundary — each customer's analysis runs in their own pod (or namespace). If you implement zero-trust correctly, the analysis-runner pod should only be able to call services it needs for its specific function. What are those services? And what services should it be explicitly forbidden from calling?

---

### Constraints

- 47 microservices in the Kubernetes cluster
- 8,000 enterprise customers, each with isolated analysis runner pods
- Analysis runners are by far the highest-risk pods (customer-accessible, run arbitrary analysis workloads)
- 12-week implementation timeline
- Zero production downtime during migration
- The immediate mitigation (Kubernetes NetworkPolicy) must be in place within 48 hours

---

### Your Task

Design the zero-trust architecture for SentinelOps. Include the immediate mitigation, the full zero-trust model, and the phased migration plan.

---

### Deliverables

- [ ] Zero-trust principles: Define the 5 principles of zero-trust and how each applies to SentinelOps's specific architecture
- [ ] Immediate mitigation: Kubernetes NetworkPolicy YAML (default-deny, with explicit allows for known service communication paths)
- [ ] Service identity design: SPIFFE/SPIRE or Kubernetes service accounts — how each service gets a cryptographic identity
- [ ] Trust boundary map: Mermaid diagram showing which services can call which, and the isolation boundary between tenant-space and platform-space
- [ ] Blast radius analysis: Before and after zero-trust — if analysis-runner is compromised, what can the attacker reach?
- [ ] Phased migration: 12-week implementation plan with no downtime
- [ ] Tradeoff analysis (minimum 3):
  - Network segmentation (Kubernetes NetworkPolicy) vs application-layer authentication (mTLS) — are both needed?
  - SPIFFE/SPIRE vs Kubernetes service account tokens for service identity
  - Default-deny with explicit allowlist vs default-allow with explicit denylist (why one is catastrophically wrong)
