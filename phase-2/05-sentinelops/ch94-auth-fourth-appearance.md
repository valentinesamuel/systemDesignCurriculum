---
**Title**: SentinelOps — Chapter 94: Advanced Authorization — RBAC vs ABAC and Policy-as-Code
**Level**: Staff
**Difficulty**: 8
**Tags**: #auth #rbac #abac #opa #policy-as-code #multi-tenant #security #compliance
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 6 (MeridianHealth Auth), Ch. 30 (CloudStack Auth Federation), Ch. 61 (SkyRoute OAuth2 Delegation), Ch. 92 (Zero-Trust Architecture)
**Exercise Type**: System Design / Security Incident Response
---

### Story Context

**Date**: Thursday, April 3 — 7:22 AM
**Channel**: #incidents-priority1

---

**#incidents-priority1 — Thursday, April 3**

**[07:22] Yusuf Okafor (Head of Product Security):** CRITICAL. Just got a call from NexaShield (customer tenant: `nxs-0041`). One of their SOC analysts — Gabriela Ferreira — was able to view threat intelligence reports belonging to Arcturus Defense (`arc-0017`). Different customers. Completely separate contracts. Gabriela confirmed she saw Arcturus data. She screenshot it. NexaShield's CISO is on the phone with our CEO right now.

**[07:24] You:** How did this happen? What's the access path?

**[07:26] Yusuf Okafor:** Gabriela's role is `SOC_ANALYST` in NexaShield's tenant. She navigated to `/api/v1/intel/reports?severity=CRITICAL` — no tenant filter in the query. The intel-reports service does RBAC check (does user have `SOC_ANALYST` role?) but does NOT verify that the returned records belong to her tenant. It trusts the frontend to send the right filters.

**[07:28] You:** So the authorization check is "can this role access this endpoint" but not "can this role access this resource owned by this tenant."

**[07:29] Yusuf Okafor:** Correct. And it's not just intel-reports. I'm pulling the code now. At least six other endpoints have the same pattern. The `tenant_id` is in the JWT but nothing enforces it at the authorization layer. We've been doing RBAC wrong for two years.

**[07:31] Nadia Cho (Engineering Manager):** Do we know the blast radius? How many tenants were potentially exposed?

**[07:33] Yusuf Okafor:** I need to query the access logs. Give me 20 minutes.

**[07:55] Yusuf Okafor:** Okay. Worst case: 14 tenants have overlapping `SOC_ANALYST` access windows over the past 6 months where cross-tenant data could have been returned. We can't confirm actual exposure without customer cooperation. We need to notify all 14.

**[08:10] Dr. Amara Kamara (CTO):** This is a contractual and regulatory breach. MSSP agreements require strict data isolation. Yusuf, I want a full incident report by noon. Engineering — I want a root cause write-up and a remediation plan by EOD. Not a patch. A design. We are fixing authorization properly.

---

**1:1 Transcript — You and Yusuf Okafor**
*Thursday, April 3 — 10:30 AM*
*Location: Yusuf's office (door closed)*

**Yusuf:** I want you to design the replacement. Not just a hotfix. I've been saying for eight months that pure RBAC is not sufficient for a multi-tenant security platform. You can't capture "SOC_ANALYST at tenant NexaShield is allowed to view threat intel reports WHERE tenant_id = nxs-0041" in a role assignment alone. Roles don't carry resource context.

**You:** That's an attribute-based access control problem. ABAC instead of RBAC.

**Yusuf:** Or a hybrid. I don't want to throw out roles entirely — ops teams understand roles. But the resource context has to be enforced at the authorization layer, not trusted from the API caller. That's the gap.

**You:** What's your take on Open Policy Agent?

**Yusuf:** I like it conceptually. Policy as code, testable, auditable. The Rego learning curve is real. But we're a security company. We should be comfortable with the complexity. What I care about is: can we express "this principal is allowed to perform this action on this resource if and only if the resource's tenant_id matches the principal's tenant claim"? And can we audit every decision?

**You:** Yes. OPA can express that. The challenge is integrating the decision call into 340 services without each team reimplementing it.

**Yusuf:** That's your design problem. I want a centralized policy store with a local sidecar evaluation model — fast decisions, no single point of failure. And I want every DENY decision logged. Not just errors. Denies. Because if Gabriela had done this intentionally, I want to know, and I want the evidence to hold up.

**You:** Authorization decision logging. That complicates the data model.

**Yusuf:** It complicates the data model. It also makes us the most defensible MSSP on the market. Frame it that way to Dr. Kamara.

---

**Slack DM — Marcus Webb → You**
*Thursday, April 3 — 2:14 PM*

**Marcus:** Heard about the NexaShield incident through the grapevine. You know what I find interesting? This is the fourth time in your career you've had to design an auth system from scratch. MeridianHealth, CloudStack, SkyRoute, now here. Each one was more complex than the last. You kept reaching for RBAC because it's familiar. But at some point you hit the wall where roles don't carry enough context. You've hit that wall. What does the design look like on the other side of it?

**You:** ABAC with OPA. Policy as code. Tenant claims enforced at every resource decision.

**Marcus:** Good. Now ask yourself: who writes the policies? Who reviews them? Who owns them when you leave? Policy-as-code only works if the governance model around it is as rigorous as the code itself. I've seen OPA deployments where the policy repo had three contributors and no PR review process. The tech was right. The process was wrong. Same outcome as no policy at all.

**You:** So the design includes a policy governance model.

**Marcus:** The design IS a policy governance model. The OPA config is just the implementation.

---

> **Auth System — 4th Appearance**
> 1st: MeridianHealth (Ch. 6) — HIPAA, OIDC SSO, MFA, client credentials. Simple role-based, single-tenant.
> 2nd: CloudStack (Ch. 30) — SAML + OIDC federation, SCIM, multi-tenant RBAC. Roles per tenant.
> 3rd: SkyRoute (Ch. 61) — OAuth2 delegation (RFC 8693), adversarial accounts, behavioral detection.
> 4th (here): RBAC at scale breaks down. Resource-level tenant isolation requires ABAC. Policy-as-code with OPA. Authorization decision audit trail.

### Problem Statement

SentinelOps's existing authorization system is pure RBAC: roles are assigned to users and roles grant access to endpoints. There is no enforcement of resource ownership or tenant context at the authorization layer. The NexaShield incident exposed a systematic gap: a `SOC_ANALYST` in one tenant can retrieve resources belonging to another tenant if the API layer does not apply tenant filters.

You must design a replacement authorization architecture that enforces tenant-scoped resource access using Attribute-Based Access Control (ABAC), implemented via Open Policy Agent as the central policy decision point, with a sidecar deployment model for performance, and a comprehensive authorization decision audit log.

### Explicit Requirements

1. Replace endpoint-level RBAC with resource-level ABAC that includes tenant context in every authorization decision
2. Centralized policy store (OPA) with local sidecar deployment for sub-5ms decision latency
3. All authorization decisions (ALLOW and DENY) logged to an immutable audit store
4. Policies expressed in Rego (OPA policy language), stored in version-controlled repository
5. Policy changes require review and approval before production deployment (policy CI/CD pipeline)
6. JWT claims must include `tenant_id`, `user_id`, `roles[]`, and `permissions[]`; OPA policies consume these claims
7. Backward-compatible migration path: existing services can migrate to OPA-enforced authorization without a flag day
8. Policy decision log retention: minimum 2 years (MSSP contractual obligation)

### Hidden Requirements

- **Hint**: Re-read Yusuf's line "I want every DENY decision logged. Not just errors. Denies. Because if Gabriela had done this intentionally, I want to know, and I want the evidence to hold up." What does this imply about the audit log's tamper-evidence properties? A SOC platform's own authorization log is a high-value target.
- **Hint**: Re-read Marcus Webb's Slack DM: "who writes the policies? Who reviews them? Who owns them when you leave?" The design requires a policy governance model — not just OPA configuration. What does the SDLC for a Rego policy look like, and who are the named approvers?
- **Hint**: Re-read Yusuf's blast radius analysis: "14 tenants have overlapping access windows over the past 6 months." For the new system to detect this proactively, what kind of cross-tenant access anomaly detection needs to be built on top of the authorization decision log?
- **Hint**: SentinelOps has 620 enterprise tenants. Each tenant can define custom roles. The OPA policy must be parameterized by tenant without requiring a separate policy file per tenant. What is the data model for tenant-specific role definitions inside OPA's data document?

### Constraints

- **Tenants**: 620 enterprise customers, each with custom role definitions
- **Services**: 340 microservices, each making authorization decisions
- **Authorization RPS**: ~180,000 service-to-service RPS (from Ch. 93) + ~45,000 user-to-service RPS = ~225,000 total authorization decisions/second at peak
- **Decision latency SLA**: p99 < 5ms (must not materially impact threat detection pipeline)
- **Audit log retention**: 2 years minimum, immutable
- **Policy deployment**: changes must not require service restarts; OPA bundle updates < 30 seconds propagation
- **Migration timeline**: hotfix (tenant_id filter enforcement) within 24 hours; full OPA rollout within 60 days
- **Compliance**: SOC 2 Type II, MSSP contractual data isolation guarantees
- **Team**: 3 engineers own the authorization platform long-term

### Your Task

Design SentinelOps's replacement authorization architecture. You must address: the OPA deployment model (centralized vs sidecar), the Rego policy data model for multi-tenant RBAC+ABAC, the JWT claims structure, the authorization decision audit log, the policy CI/CD pipeline, and the migration path from the current broken RBAC system.

### Deliverables

- [ ] **Mermaid architecture diagram** — Full authorization flow: user request → API gateway → OPA sidecar decision → service handler → resource fetch → response. Show the policy bundle update flow from the policy store to all OPA sidecars.
- [ ] **Database schema** — Authorization decision audit log table (decision_id, principal_id, tenant_id, action, resource_type, resource_id, policy_version, decision: ALLOW/DENY, reason, request_metadata JSONB, decided_at) with indexes for compliance query patterns (tenant + time range, principal + resource_type)
- [ ] **Scaling estimation** — At 225,000 decisions/second: (a) OPA sidecar CPU requirements per pod (OPA benchmarks ~10,000 simple decisions/sec per core — show the fleet sizing math), (b) audit log write throughput (225k decisions/sec × average record size = MB/sec), (c) 2-year audit log storage size
- [ ] **Tradeoff analysis** — Minimum 3 explicit tradeoffs:
  - Centralized OPA cluster vs OPA sidecar per service
  - RBAC preserved as a layer within ABAC vs full ABAC replacement
  - Synchronous authorization call vs middleware enforcement
- [ ] **Rego policy sample** — Write the core Rego rule that implements: "a principal with role SOC_ANALYST may perform action READ on resource intel_report if and only if the resource's tenant_id equals the principal's tenant_id claim." Show the data document structure for tenant-specific role definitions.
- [ ] **TypeScript interface** — `AuthorizationRequest`, `AuthorizationDecision`, and `PolicyBundle` interfaces
- [ ] **Migration plan** — 24-hour hotfix (what exactly changes in the existing code), 60-day OPA rollout phasing, and rollback procedure
