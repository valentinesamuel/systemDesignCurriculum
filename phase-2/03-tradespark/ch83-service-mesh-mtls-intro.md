---
**Title**: TradeSpark — Chapter 83: Service Mesh and mTLS Introduction
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #service-mesh #mtls #istio #security #circuit-breakers #linkerd
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 93, Ch. 120, Ch. 139
**Exercise Type**: System Design / Security
---

### Story Context

**Email — Rachel Kim (CISO) → You**
*(Rachel Kim, NexaCare's former CISO, joined TradeSpark 3 months ago. Small world.)*

Subject: Annual Security Audit — Finding #1 — URGENT

Finding #1 of 7: Service-to-service communication within TradeSpark's trading cluster is **unencrypted and unauthenticated**.

Specifically:
- The trading engine communicates with the risk system over plaintext HTTP
- The settlement service communicates with the execution database over unencrypted TCP
- Any pod within the Kubernetes cluster can call any service without authentication

Impact classification: **CRITICAL**

If any pod within the cluster is compromised by an adversary (via dependency vulnerability, misconfigured container, or supply chain attack), the attacker has network access to:
- All trading strategy parameters (sensitive intellectual property)
- All settlement data (financial data, regulatory impact)
- The risk management system (could be manipulated to show incorrect exposure)

SEC Regulation S-P and SEC Rule 17a-4 both require that electronic records containing trading data be protected from unauthorized access.

Required remediation: Mutual TLS authentication for all service-to-service communication.
Required timeline: Before next SEC examination (scheduled in 90 days).

Rachel Kim, CISO

---

**#engineering — TradeSpark**
`Friday 09:15` **you**: Rachel's security audit is in. her top finding is mutual TLS for service-to-service calls. I want to discuss the implementation approach.
`09:17` **marco.bellini**: we talked about this in 2022. it's a lot of work to add mTLS to every service. certificates, rotation, the PKI...
`09:18` **you**: that's why I'm proposing a service mesh. the mTLS is transparent to the application code. the sidecar proxy handles it.
`09:19` **marco.bellini**: I've seen service meshes add 10-15ms to every request. at our latency requirements, that's a problem.
`09:20` **you**: the latency overhead depends on the mesh implementation and the traffic pattern. Linkerd's overhead is typically 1-2ms. Istio's Envoy sidecar is 5-10ms. for our management plane calls (risk, settlement, monitoring) — those aren't latency-sensitive. for the hot path (order execution) — we'd exempt those from the mesh.
`09:21` **rachel.kim** *(who just joined)*: I'm okay with exempting the nanosecond-sensitive hot path. what I need is mTLS for everything that touches financial data at human-readable timescales.
`09:22` **marco.bellini**: "management plane mTLS" — I can work with that
`09:23` **you**: the bigger benefit for us beyond security is circuit breakers at the infrastructure level. right now, when the risk system is slow, it takes down the settlement service because there's no circuit breaker. the mesh can enforce timeouts and circuit breaking without code changes.
`09:24` **marco.bellini**: ...that would have prevented the settlement cascade from three weeks ago?
`09:25` **you**: yes. the risk system timeout would have tripped the circuit breaker at the mesh level. settlement would have failed fast instead of blocking.
`09:26` **marco.bellini**: okay. I'm listening.

---

### Problem Statement

TradeSpark's service-to-service communication is unencrypted and unauthenticated, creating a critical security vulnerability. A service mesh with mutual TLS will address the security finding and provide infrastructure-level circuit breaking to prevent the cascade failures that have caused recent settlement incidents. The hot path (nanosecond-sensitive order execution) must remain outside the mesh for latency reasons.

---

### Explicit Requirements

1. All service-to-service communication touching financial data must use mTLS within 90 days
2. Certificate rotation must be automatic (no manual certificate management)
3. The mesh must enforce timeout policies and circuit breakers for risk and settlement services
4. Latency overhead for mTLS must be < 2ms P99 for management plane calls
5. The order execution hot path must be exempted from mesh overhead
6. All mTLS connections must be auditable (connection logs for SEC compliance)

---

### Hidden Requirements

- **Hint**: Re-read Marco's latency concern: "service meshes add 10-15ms." He's right for some implementations. But there's a hidden concern he didn't raise: the sidecar proxy pattern adds a separate process to every pod. In TradeSpark's Kubernetes cluster, this doubles the number of running containers. What does this do to resource consumption (CPU, memory), and does TradeSpark have headroom?

- **Hint**: Rachel said "mTLS for everything that touches financial data." mTLS provides authentication (you are who you claim to be) and encryption (traffic is private). But it doesn't provide authorization (you're allowed to call this endpoint). In the future, when TradeSpark adds a new microservice, how does the mesh ensure new services can only call what they're authorized to call?

---

### Constraints

- 12 services in TradeSpark's Kubernetes cluster
- Order execution hot path: < 500µs latency budget (cannot add any overhead)
- Management plane calls: 10–50ms baseline (2ms overhead acceptable)
- Certificate rotation: must be automatic, every 24 hours maximum TTL
- 90-day remediation deadline for SEC examination
- Existing infrastructure: Kubernetes 1.28, Calico CNI

---

### Your Task

Design the service mesh implementation for TradeSpark's Kubernetes cluster. Include the mTLS architecture, circuit breaker policies for risk and settlement services, and the traffic segmentation between hot path and management plane.

---

### Deliverables

- [ ] Mermaid diagram: Service mesh topology showing which services are in-mesh and which are hot-path exempt
- [ ] mTLS design: Certificate authority design (Istio Citadel / SPIRE), certificate lifetime, automatic rotation
- [ ] Circuit breaker policies: For the risk service and settlement service, define timeout thresholds and circuit breaker open/half-open/closed behavior
- [ ] Linkerd vs Istio comparison: For TradeSpark's specific constraints (latency sensitivity, SEC audit logging requirement)
- [ ] Hot path isolation: How to exempt the order execution hot path from mesh overhead while maintaining security
- [ ] Audit logging: How to configure mesh access logs for SEC compliance (what fields, what retention)
- [ ] Tradeoff analysis (minimum 3):
  - Istio (Envoy sidecar) vs Linkerd (Rust-based proxy) for this use case
  - STRICT mTLS (reject all non-mTLS) vs PERMISSIVE mTLS (allow both during migration)
  - Service mesh circuit breakers vs application-level circuit breakers (Resilience4j/Hystrix)
