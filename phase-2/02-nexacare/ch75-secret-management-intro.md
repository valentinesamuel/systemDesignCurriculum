---
**Title**: NexaCare — Chapter 75: Secret Management with HashiCorp Vault
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #security #secret-management #vault #hashicorp #secret-rotation #audit
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 95
**Exercise Type**: Incident Response / System Design
---

### Story Context

**#security-alerts — NexaCare**
`Saturday 11:34` **github-bot**: 🚨 SECRET DETECTED — repository nexacare/trial-ingestion-service — file: config/database.yml — possible PostgreSQL credential — commit: a7f3c2b — author: dev.lucas.zhang
`11:35` **pagerduty-bot**: 🔴 P1 ALERT — CISO notification required: credential exposed in public repository

**#incidents**
`11:37` **rachel.kim** *(CISO)*: @fatima.al-rashid @you — I need both of you on a call in 5 minutes. We have a credential exposure incident.
`11:38` **you**: joining now
`11:39` **rachel.kim**: Lucas Zhang committed a database credential to the trial-ingestion-service repository at 11:17 AM. GitHub's secret scanning detected it at 11:34. The repository was PUBLIC for 47 minutes.
`11:40` **fatima.al-rashid**: 47 minutes. was it indexed by any search engines?
`11:41` **rachel.kim**: unknown. GitHub doesn't tell us if bots crawled it. we're treating it as compromised.
`11:42` **you**: I'm rotating the credential now.
`11:43` **rachel.kim**: do it. and then I need a full audit. this is not the first time.

**Incident Report — Rachel Kim to Fatima Al-Rashid (1 hour later)**

> Rachel Kim: I've completed the initial audit. The exposed credential was for the trial_observations database replica. I've confirmed it was rotated within 8 minutes of detection.
>
> The deeper problem: I found 340 other secrets that are not managed correctly. Specifically:
> - 47 database credentials hardcoded in environment variables across 12 services
> - 89 API keys in Kubernetes ConfigMaps (not Secrets)
> - 112 secrets in Kubernetes Secrets (which are base64-encoded, not encrypted — not the same as secure)
> - 91 secrets in .env files in developer home directories
> - 1 production private key in a Google Doc titled "Backup Keys — DO NOT SHARE"
>
> We do not have a secret management system. We have a secret chaos system.

**Slack DM — You → Rachel Kim**
`12:47`
**you**: Rachel — I'm looking at the audit results. I've seen this problem before (not at NexaCare level, but similar). The answer is a dedicated secret management platform. HashiCorp Vault is the industry standard. But before I propose it — can I ask: what's the most urgent risk to remediate first?
**rachel.kim**: the 47 hardcoded database credentials. if any of those services are compromised, an attacker has direct database access. patient health data. HIPAA.
**you**: agreed. the Vault proposal would eliminate ALL static credentials with dynamic secrets — short-lived credentials generated per request. but that's a 4-week migration. what do we do in the next 48 hours?
**rachel.kim**: in the next 48 hours, we rotate all 47. then we implement Vault over 4 weeks. I'll help you write the architecture proposal.
**you**: I'm going to need you to explain Vault to the DBA team. Sasha is going to push back on dynamic credentials — she's going to say "this is too complex."
**rachel.kim**: leave Sasha to me. I have one word for her: HIPAA.

---

### Problem Statement

NexaCare's production infrastructure has 340 unmanaged secrets distributed across environment variables, Kubernetes ConfigMaps, hardcoded configs, and developer machines. A credential was accidentally exposed publicly for 47 minutes. The root cause is architectural: there is no centralized secret management system. Static credentials persist indefinitely, there is no audit trail for secret access, and no rotation mechanism.

You need to design and implement a HashiCorp Vault-based secret management system that eliminates all static credentials through dynamic secrets, provides a complete audit trail of secret access, and includes break-glass procedures for emergency access.

---

### Explicit Requirements

1. All database credentials must be dynamic (short-lived, generated per service request, not static)
2. All API keys and external service credentials must be stored in Vault (not in environment variables or configs)
3. Secret access must generate an audit log entry per access (accessible for HIPAA audit)
4. Vault must be highly available (cannot be a single point of failure)
5. Secret rotation must be possible without service downtime
6. Break-glass procedures must exist for emergency access when Vault is unavailable
7. The Vault audit log must be immutable and streamed to the SIEM

---

### Hidden Requirements

- **Hint**: Re-read Rachel's audit: "91 secrets in .env files in developer home directories." These are on developer laptops, not servers. Developer laptop secrets are almost certainly not rotated when a developer leaves the company. How does your Vault design handle the offboarding problem?

- **Hint**: The 340 secrets are spread across 12 services. The migration to dynamic credentials requires each service to authenticate with Vault before getting credentials. What's the bootstrapping problem — how does a service prove its identity to Vault before it has any credentials?

---

### Constraints

- 12 services, 340 secrets, 47 developers
- 3 database clusters (trial_observations, patient_records, audit_log)
- Vault must have 99.99% availability (HIPAA requires audit trail access; Vault unavailability = no new DB connections)
- Secret rotation SLA: credentials must be rotatable within 5 minutes without service restart
- 4-week migration window
- Team: You + Rachel Kim (CISO) + Sasha Petrov (DBA)

---

### Your Task

Design the HashiCorp Vault deployment and secret management architecture for NexaCare. Include the dynamic secrets design for databases, the service authentication model, the break-glass procedure, and the 4-week migration plan.

---

### Deliverables

- [ ] Mermaid diagram: Vault architecture (Vault cluster, auth methods, secrets engines, services)
- [ ] Dynamic secrets design: How a service requests a database credential from Vault, what the credential lifetime is, how it's automatically revoked
- [ ] Service authentication: AppRole auth design — how each service gets its initial Vault token without hardcoded credentials (the bootstrapping problem)
- [ ] Vault HA design: Active/standby nodes, auto-unseal with AWS KMS, storage backend (Consul vs integrated Raft)
- [ ] Break-glass procedure: Step-by-step emergency access protocol when Vault is unavailable, with mandatory review afterward
- [ ] Migration plan: 4-week timeline for migrating 340 secrets from 12 services
- [ ] Tradeoff analysis (minimum 3):
  - Dynamic secrets (short-lived) vs static secrets (rotated) for database credentials
  - Vault agent sidecar vs Vault SDK injection for secret delivery to services
  - AppRole vs Kubernetes auth method for Kubernetes-deployed services
