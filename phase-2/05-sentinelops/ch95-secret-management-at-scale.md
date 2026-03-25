---
**Title**: SentinelOps — Chapter 95: Secret Management at Scale
**Level**: Staff
**Difficulty**: 8
**Tags**: #vault #secret-management #security #distributed-systems #database #compliance #zero-trust
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 92 (Zero-Trust Architecture), Ch. 93 (Service Mesh), Ch. 94 (Auth/ABAC)
**Exercise Type**: System Design
---

### Story Context

**Date**: Monday, May 5 — 11:00 AM
**Meeting**: Vault Architecture Review — "How We Store Secrets at a Security Company"
**Attendees**: Dr. Amara Kamara (CTO), Yusuf Okafor (Head of Product Security), Priya Suresh (Platform Lead), Tomás Vidal (SRE), You

---

**Email Chain**

**From**: Dr. Amara Kamara
**To**: Engineering Leadership
**Subject**: Vault Design — This Is Not NexaCare
**Date**: Friday, May 2 — 5:47 PM

Team,

I reviewed the current Vault deployment plan that Tomás drafted last month. It is a reasonable starting point for a healthcare company or a mid-market SaaS. We are not those things.

We are a managed security service provider. Our customers trust us to detect nation-state actors inside their networks. If our own secret management infrastructure is compromised, we become the attack vector. This happened to a major SIEM vendor in 2020. It will not happen to us.

I want Monday's design session to produce something different. Not a Vault deployment. A security posture showcase. The kind of Vault design that we would recommend to a Fortune 100 customer and be comfortable publishing in a SANS whitepaper.

Specifically I want to hear:
- How does Vault stay available at 99.999%? That is 5 minutes of downtime per year.
- How do we auto-unseal without creating a key escrow problem?
- How do we eliminate ALL static database credentials from ALL 340 services?
- How does the Vault audit log become part of our SIEM pipeline, not an afterthought?
- What happens when Vault itself is unavailable? How do 340 services continue operating?

Come prepared to defend your answers.

— Amara

---

**Reply from Tomás Vidal**
**Date**: Saturday, May 3 — 10:22 AM

Dr. Kamara,

Point taken. I'll revise before Monday. Two questions I want the group to resolve:

1. Auto-unseal with AWS KMS vs HSM. KMS is operationally simpler but creates a dependency on AWS. An HSM is more defensible but requires physical access for initial setup and is expensive (~$40k for a CloudHSM cluster). For a security company, which story do we tell?

2. Vault HA with Raft vs Consul backend. HashiCorp has been pushing Raft integrated storage as the default for new deployments. I've had bad experiences with Vault+Consul version drift. But Raft is newer and I don't have 3-year production data on it. Do we trust it at 99.999%?

— Tomás

---

**Slack DM — Marcus Webb → You**
*Sunday, May 4 — 8:31 PM*

**Marcus:** I heard you're designing the Vault architecture for SentinelOps. I've seen Vault deployments that created more risk than they solved. Not because Vault is bad. Because people treat it like a password manager at scale and don't think through the failure modes. Let me ask you three questions. Don't answer now — think about them for the meeting tomorrow.

**Marcus:** One: if Vault is unavailable for 10 minutes, what exactly breaks? List the services. Are any of them in the threat detection critical path?

**Marcus:** Two: you have 340 services and 12 database clusters. Every service gets a dynamic secret with a 1-hour TTL. That's 340 secret leases being renewed every hour. What happens to Vault's storage engine at that renewal rate? Does anyone actually benchmark this before going to production?

**Marcus:** Three: break-glass. Your on-call SRE wakes up at 3AM and Vault is down hard — Raft leader election failed, no quorum. Cluster is dead. What's the procedure? Write it down. If it takes more than 10 minutes to describe, it's too complicated.

**You:** All three of those are things nobody has asked yet.

**Marcus:** That's why I'm asking them. The design is for the normal case. The runbook is for when the design fails. At a security company, the runbook IS the security posture.

---

**Meeting Transcript — Monday, May 5, 11:00 AM**

**Dr. Kamara [11:00 AM]:** I want to start differently. Before we talk about what we're building, I want to talk about what we're defending against. The threat model for our Vault deployment. Tomás, what are the top three attack surfaces?

**Tomás Vidal [11:02 AM]:** First: the unseal keys. If an attacker gets the unseal keys, they can decrypt the Vault storage and read everything. Second: AppRole credentials. If a malicious actor gets a service's RoleID and SecretID, they can authenticate to Vault as that service and retrieve all its secrets. Third: the Vault root token. It should never exist after initialization, but in practice teams keep it "just in case."

**Dr. Kamara [11:04 AM]:** Good. Those three things. I want the design to make each of them either impossible or immediately detectable. That's the bar.

**Yusuf Okafor [11:05 AM]:** Auto-unseal with KMS eliminates the unseal key exposure — the key never exists as a file on disk. AppRole with short-lived SecretIDs and response wrapping eliminates static credentials. Root token — destroy it after initialization, period. Store the recovery keys in Shamir shares held by five named executives, require three to reconstruct.

**Priya Suresh [11:07 AM]:** I want to flag the operational complexity. Dynamic secrets for 12 database clusters means Vault needs a database secrets engine configured for each cluster with a service account that can CREATE ROLE in the database. That service account is itself a high-value secret. We're bootstrapping secrets with secrets.

**You [11:09 AM]:** That's the Vault agent problem. The agent running in each service pod needs to authenticate to Vault to get the dynamic secret. That authentication needs to be bootstrapped. In Kubernetes, we use the K8s auth method — the pod's service account JWT, signed by the cluster's API server, is the authentication credential. It's verifiable without any out-of-band secret distribution. That's the answer to Priya's concern.

**Dr. Kamara [11:11 AM]:** Show me that on the whiteboard.

---

### Problem Statement

SentinelOps currently stores database credentials as environment variables in Kubernetes Secrets (base64 encoded, not encrypted at rest). The zero-trust initiative requires eliminating all static credentials from all 340 services across 3 Kubernetes clusters. HashiCorp Vault will be deployed as the central secret management platform, but the deployment must meet a 99.999% availability SLA, integrate with the existing SIEM pipeline for audit log streaming, support dynamic credentials for all 12 database clusters, and include a tested break-glass procedure for complete Vault cluster failure.

### Explicit Requirements

1. Vault HA cluster using Raft integrated storage (3 nodes minimum per cluster, deployed in each of the 3 regions)
2. Auto-unseal using cloud KMS (AWS KMS in us-west-2, aws:kms EU region in eu-west-1, GCP KMS in ap-southeast-1 as a deliberate multi-cloud unseal dependency mitigation)
3. Dynamic secrets for all 12 PostgreSQL/MySQL database clusters; no service may hold a static database password
4. Dynamic secret TTL: 1 hour for application credentials, 15 minutes for CI/CD pipeline credentials
5. Kubernetes auth method for all service pods (no AppRole SecretID distribution required)
6. Vault audit log streamed to SIEM (Splunk) in real-time; log tampering must be detectable
7. Vault agent sidecar injected by the agent injector (not manual configuration per service)
8. Root token destroyed after initialization; recovery keys held in 5-of-3 Shamir shares
9. Break-glass procedure documented and tested quarterly; recovery time objective (RTO) < 15 minutes for Vault cluster rebuild
10. Secret rotation for all dynamic leases must survive a Vault leader re-election without lease expiry

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's second question: "340 services and 12 database clusters, dynamic secrets with 1-hour TTL — what happens to Vault's storage engine at that renewal rate?" At 340 leases renewed every 60 minutes, what is the write throughput to Vault's Raft log? Does Vault's default `lease_count_max` of 300,000 become a constraint?
- **Hint**: Re-read Dr. Kamara's email: "What happens when Vault itself is unavailable? How do 340 services continue operating?" This implies a local secret caching strategy in the Vault agent — but cached secrets with hour-long TTLs mean a compromised pod retains valid credentials for up to an hour after detection. What is the maximum acceptable cache TTL for a security platform?
- **Hint**: Re-read Tomás's email question about HSM vs KMS: "For a security company, which story do we tell?" The decision has compliance implications. PCI-DSS and FedRAMP moderate both have specific requirements around HSM for key management. Do any of SentinelOps's customers operate in FedRAMP moderate environments? (Yes — three defense contractors were mentioned in the intro.)
- **Hint**: Re-read Priya's comment: "That service account is itself a high-value secret. We're bootstrapping secrets with secrets." The Kubernetes auth method solves the bootstrap problem for application pods — but what about the Vault database secrets engine service account in PostgreSQL? That account has `CREATE ROLE` permission. How is it rotated, and who has access to it?

### Constraints

- **Services**: 340 microservices across 3 Kubernetes clusters (us-west-2, eu-west-1, ap-southeast-1)
- **Database clusters**: 12 (mix of PostgreSQL 15 and MySQL 8.0)
- **Vault availability SLA**: 99.999% (5.26 minutes downtime/year)
- **Dynamic secret TTL**: 1 hour (app), 15 minutes (CI/CD)
- **Lease renewal rate**: 340 leases × (60 min / 1 hr TTL) = 340 renewals/hour at steady state; add lease creation on pod restarts
- **Audit log volume**: estimated 50,000 Vault API operations/hour; must stream to Splunk without loss
- **Break-glass RTO**: < 15 minutes for full Vault cluster rebuild from snapshot
- **Vault storage**: Raft integrated storage; snapshot backup every 15 minutes to S3 (encrypted)
- **Compliance**: SOC 2 Type II, PCI-DSS (3 customers), FedRAMP Moderate consideration (3 defense contractor customers)
- **Node sizing**: Vault nodes run on dedicated m6i.xlarge instances (4 vCPU, 16GB RAM) — not shared with application workloads
- **Secret sprawl baseline**: current state has ~1,200 static secrets in Kubernetes Secrets objects across all clusters

### Your Task

Design the production Vault deployment for SentinelOps. Your design must cover the Raft cluster topology, auto-unseal configuration, Kubernetes auth method integration, Vault agent sidecar injection model, dynamic database secrets engine configuration, audit log SIEM streaming pipeline, and break-glass runbook. You must address the lease renewal throughput at scale and the local caching strategy given that SentinelOps is itself a security platform.

### Deliverables

- [ ] **Mermaid architecture diagram** — Full secret lifecycle: pod startup → K8s auth method → Vault token issuance → dynamic DB credential generation → Vault agent cache → application use → TTL expiry → renewal. Show Raft cluster topology across 3 regions and the KMS auto-unseal dependency for each region.
- [ ] **Database schema** — Secret lease tracking table (lease_id, service_name, cluster, secret_type, issued_at, expires_at, renewed_count, last_renewed_at, status: ACTIVE/EXPIRED/REVOKED, vault_accessor) with indexes optimized for expiry sweeps and per-service lease audits
- [ ] **Scaling estimation** — Show math for: (a) Vault Raft log write throughput at 340 leases × renewal rate + pod churn (assume 10% daily pod restarts = 34 new lease issuances/day from pod restarts), (b) audit log SIEM throughput in MB/hour at 50,000 operations/hour with average 512-byte audit record, (c) Raft snapshot size given 1,200 secrets migrated + 340 dynamic leases in steady state
- [ ] **Tradeoff analysis** — Minimum 3 explicit tradeoffs:
  - KMS auto-unseal vs HSM (Vault Enterprise CloudHSM) — security posture vs operational complexity for a FedRAMP customer
  - Vault agent cache TTL: 1 hour (matches lease TTL, minimal Vault load) vs 5 minutes (rapid revocation, higher Vault load)
  - Single global Vault cluster vs per-region Vault clusters — latency vs operational complexity vs data residency
- [ ] **Break-glass runbook** — Step-by-step procedure for complete Vault cluster failure: detection, snapshot restore, re-unseal, verification, service recovery. Must fit in < 15 minutes of execution time.
- [ ] **Dynamic secrets migration plan** — How to migrate 12 database clusters from static credentials to dynamic secrets without service downtime: phasing, rollback triggers, and validation criteria per cluster
