# Pattern Summary — Security Architecture, Service Mesh & Deployment Patterns

**Covers**: Chapters 92–104 (SentinelOps + LightspeedRetail arcs)
**Type**: Pattern Summary (no deliverables — reflection only)
**Related Chapters**: Ch. 92, Ch. 93, Ch. 94, Ch. 95, Ch. 96, Ch. 99, Ch. 100, Ch. 101, Ch. 102

---

## How This Summary Arrived

You open your laptop on a Sunday morning to find an unusually long document in your inbox. It's from Marcus Webb. The subject line is "pre-retirement knowledge dump — read this." There is no greeting. The document is 14 pages, single-spaced. His annotations are in bold. Some of them are blunt opinions. Some read like confessions. All of them are useful.

His cover note, in its entirety:

> "I've been writing this for three years. Every time I saw someone make a mistake I'd already made, I added a paragraph. You've now seen enough to understand all of it. Don't just read the patterns. Read the annotations. That's where the real education is. — M."

What follows is a distillation.

---

## Pattern 50: Zero-Trust Network Design

**The Concept**: In a zero-trust model, the network perimeter does not determine trust. Every service, every call, every connection is explicitly authenticated and authorized regardless of whether it originates inside or outside the corporate network. The default posture is deny; trust is granted through explicit policy, not network location.

**The Design**: Zone-based network segmentation with explicit allowlists replaces implicit perimeter trust. Every service communicates only with services it has a declared relationship with. Unknown paths are blocked at the network layer before they reach the application layer.

**Where You Saw It**: SentinelOps — designing the security platform for a fintech client. The attacker in that scenario had already breached the network perimeter. Zero-trust meant the attacker still could not reach the payment processing service because no allowlist entry existed for the path they tried to traverse.

**Marcus Webb's Annotation**: *"I've seen companies spend $50 million on perimeter security and nothing on lateral movement detection. A breach gets through the wall — that's survivable. A breach that can traverse freely once inside — that's catastrophic. Zero-trust is not about making the wall thicker. It's about assuming the wall is already breached."*

**When to Apply**: Any system with a sensitive data tier, any system with external contractors or third-party integrations, any system handling financial or health data. The cost of zero-trust setup is high. The cost of not having it after a breach is higher.

---

## Pattern 51: mTLS End-to-End — Istio STRICT Mode Design

**The Concept**: Mutual TLS (mTLS) means both sides of a connection authenticate each other via certificates, not just the server authenticating to the client. In a service mesh, every service-to-service call is mTLS by default, with certificates managed and rotated by the mesh control plane (Istio, Linkerd).

**The Design**: Istio STRICT mode means the mesh rejects all non-mTLS traffic. Every pod receives an identity (SPIFFE/SVID). The control plane (istiod) issues short-lived certificates — default 24-hour rotation. Services do not manage their own certs. Certificate rotation is invisible to application code.

**Where You Saw It**: SentinelOps — enabling mTLS across a microservices platform that had grown organically without service identity. The migration path: PERMISSIVE mode (accepts both mTLS and plaintext) → STRICT mode (rejects plaintext) → enforce via PeerAuthentication policy.

**Marcus Webb's Annotation**: *"The migration from PERMISSIVE to STRICT will find every service that was quietly talking plaintext across the network. There are always more of them than you think. Run PERMISSIVE for two weeks before flipping to STRICT. Watch the Grafana dashboard. Every red line on 'rejected plaintext connections' is a service that needs to be enrolled."*

**When to Apply**: Regulated industries (fintech, health, energy) where service-to-service authentication is a compliance requirement. Also in any microservices platform where you cannot trust that every engineer has correctly implemented TLS in their service.

---

## Pattern 52: RBAC vs ABAC Selection Criteria

**The Concept**: Role-Based Access Control (RBAC) grants permissions to roles; users inherit permissions through role membership. Attribute-Based Access Control (ABAC) evaluates policies against attributes of the subject, resource, and environment at the time of each request.

**The Selection Criteria**:
- RBAC is correct when: access is determined by organizational function, roles are stable and finite, you can enumerate all permission sets in advance.
- ABAC is correct when: access depends on context (time, location, data classification, resource attributes), roles would proliferate to hundreds to capture all combinations, or data-level access control is required (row-level, field-level).

**The RBAC Breakdown Point**: When the number of roles grows to express all permission combinations — "UK-readonly-analyst", "DE-readonly-analyst", "FR-readwrite-analyst" — you have effectively implemented ABAC using RBAC syntax. This is the signal to switch.

**Where You Saw It**: SentinelOps and CloudStack (Phase 1). SentinelOps had 340 custom roles in a system designed for 12. The design review revealed this was ABAC in disguise.

**Marcus Webb's Annotation**: *"Sixty roles is RBAC. Six hundred roles is broken ABAC. The engineers who built that system weren't lazy — they were using the only tool they had. The lesson is: choose the right abstraction before you need to migrate away from the wrong one."*

---

## Pattern 53: OPA Policy-as-Code

**The Concept**: Open Policy Agent (OPA) externalizes authorization decisions from application code into declarative policies written in Rego. The application sends a structured query to OPA; OPA evaluates the policy and returns an allow/deny decision with a structured reason. Authorization logic is versioned, tested, and deployed independently of the application.

**The Key Behaviors**:
- Policies are data, not code — they can be updated without redeploying the application.
- Every authorization decision is logged with the input context and the policy that was evaluated.
- Policy unit tests are written in Rego and run in CI. Authorization regressions are caught before production.

**Where You Saw It**: SentinelOps — replacing 340 hardcoded role checks with OPA policies. The authorization decision log became an audit trail: who accessed what data, under which policy, at which timestamp.

**Marcus Webb's Annotation**: *"The authorization decision log is underrated. Most teams treat OPA as just a faster way to write if-statements. It is that. But the decision log is what saves you during a compliance audit. 'Show me every user who accessed PHI in Q3' becomes a SQL query on the decision log instead of a two-week code archaeology exercise."*

---

## Pattern 54: Vault Dynamic Secrets and Rotation Pipeline

**The Concept**: HashiCorp Vault generates short-lived credentials on demand (dynamic secrets) rather than storing long-lived secrets that require manual rotation. A database credential issued by Vault expires in 15 minutes. If it is compromised, the damage window is bounded. The application requests a new credential on expiry — transparently, without redeployment.

**The Rotation Pipeline**: Vault database secrets engine generates a Postgres role with a 15-minute TTL. The application uses the Vault agent sidecar to automatically refresh credentials before expiry. The Vault agent handles retry and caching. The application never touches a long-lived password.

**The Failure Mode to Design Around**: Vault unavailability. If Vault is down and a credential expires, the application loses database access. The mitigation: a Vault HA cluster (Raft backend, 3-5 nodes), a credential cache in the Vault agent with a grace period, and a circuit breaker that falls back to the last valid cached credential for up to 1 hour.

**Where You Saw It**: SentinelOps security architecture chapter. The client had 47 static database passwords embedded in environment variables, some of which had not been rotated in 4 years.

**Marcus Webb's Annotation**: *"A 4-year-old password in an environment variable is not a credential — it's a liability waiting to appear in a breach disclosure. I've written the disclosure letters. I prefer not to."*

---

## Pattern 55: Distributed Tracing Tail-Based Sampling Strategy

**The Concept**: Head-based sampling decides whether to trace a request at the start, before the outcome is known. Tail-based sampling buffers spans for a request and makes the sampling decision at the tail — after the outcome is known. This means: slow requests, error requests, and requests with anomalous behavior are always sampled. Healthy fast requests are sampled at a low rate (1-5%).

**The Design**: OpenTelemetry collector with a tail-sampling processor. Sampling rules (in priority order): always sample if error, always sample if latency > p95 threshold, always sample if trace contains a security event, sample 5% of all other traces.

**The Storage Impact**: Tail-based sampling at 5% of healthy traffic + 100% of error/slow traffic produces roughly 8-15% of total trace volume stored. At high RPS, this is still significant — size your trace backend (Jaeger, Grafana Tempo) for this rate, not 100%.

**Where You Saw It**: SentinelOps observability chapter — designing trace collection for a security platform where every audit-relevant request must be fully sampled.

**Marcus Webb's Annotation**: *"Head-based sampling is a lie you tell yourself. 'I'm sampling 10%.' Sure. But when production breaks at 2AM, the broken request is in the 90% you didn't sample. Tail-based sampling collects exactly what you need: the evidence for what went wrong."*

---

## Pattern 56: Blue/Green with Expand/Contract Database Migrations

**The Concept**: Blue/green deployments require that two versions of the application (old and new) can run simultaneously against the same database schema. This means the schema must be forward-compatible: the old binary must tolerate new columns; the new binary must tolerate missing data in new columns.

**The Three-Phase Expand/Contract Pattern**:
1. **Expand**: Add the new column as nullable (no lock, no downtime). Old binary ignores it. New binary writes to it.
2. **Deploy**: Promote green fleet. Both binaries handle the schema correctly. Verify in canary.
3. **Contract**: Backfill historical data. Add NOT NULL constraint. Remove old column (separate deploy). Old binary is now retired.

**The Migration Safety Gate**: A CI/CD pipeline step that parses migration SQL and blocks merges containing: `ADD COLUMN ... NOT NULL` without DEFAULT, `DROP TABLE`, full-table `UPDATE` without batch sizing, `ALTER TABLE ... LOCK`. These patterns cause downtime in a live system.

**Where You Saw It**: LightspeedRetail — Chapter 102. A NOT NULL column addition locked the `transactions` table for 9 minutes, causing a 12-minute outage across 120,000 POS terminals. $144,000 in revenue lost.

**Marcus Webb's Annotation**: *"The $144k lesson is one of the cheaper ways to learn expand/contract. I've seen companies lose $12M to a migration that ran against a table with 8 billion rows. The DBA said it would take 10 minutes. It took 6 hours. They ran it at 2PM on a Tuesday."*

**Cross-Industry Application**: Every system with a live database that receives deployments — which is every system. The pattern does not change whether the domain is retail, energy, finance, or health. The discipline is universal.

---

## Pattern 57: Immutable Infrastructure Pipeline (Packer → AMI → Auto Scaling)

**The Concept**: Immutable infrastructure means servers are never modified after deployment. A code change produces a new artifact (AMI, container image). The old fleet is replaced by a new fleet from the new artifact. No SSH. No in-place updates. No configuration drift.

**The Pipeline**:
1. Code merge → CI build → artifact (binary or Docker image)
2. Packer bakes a new AMI: base OS + application artifact + configuration
3. AMI is tagged (version, git SHA, build date) and stored in AMI registry
4. Terraform Launch Template updated to reference new AMI
5. Auto Scaling Group performs rolling replacement (or blue/green swap)
6. Old AMI retained for 30 days for rollback

**The Snowflake Drift Problem**: Servers that have been manually SSH-modified accumulate configuration state that is not captured anywhere. When they fail, they cannot be reproduced. When they are scaled, the new instances behave differently from the old ones. Immutable infrastructure eliminates drift by design — the AMI is the complete, reproducible artifact.

**Where You Saw It**: LightspeedRetail — Chapter 101. 47 unique EC2 configurations, 3 with unknown SSH-applied changes, no IaC. The burn-and-rebuild drill forced the organization to confront that they could not reproduce their own infrastructure.

**Marcus Webb's Annotation**: *"'Cattle not pets' is the phrase. Your servers should be indistinguishable from each other and replaceable without ceremony. If someone on your team knows the 'personality' of a specific server — its quirks, its manual tweaks, the thing you have to restart in a particular order — that server is a pet. And pets are liabilities."*

**Cost Impact**: Immutable infrastructure enables spot instance strategies (ephemeral by design), auto-scaling (scale to zero in off-hours), and fast incident recovery (replace a failing instance in minutes, not hours). LightspeedRetail's Black Friday cost dropped from $340,000 to $80,000 by switching from permanent over-provisioning to auto-scaling immutable fleets.

---

## Cross-Industry Observation: Security and Deployment Are Converging

A pattern emerged across the SentinelOps and LightspeedRetail arcs that is worth naming explicitly.

Security architecture and deployment architecture are solving the same underlying problem: **how do you maintain trust in a system that changes continuously?**

Zero-trust networks and immutable infrastructure share the same principle: assume the current state is compromised or stale. Do not inherit trust from previous state. Re-verify. Re-provision. Re-authenticate.

mTLS certificate rotation and Vault dynamic secrets share the same principle: short-lived credentials issued from a verifiable source are safer than long-lived credentials that accumulate exposure over time.

Blue/green deployments and expand/contract migrations share the same principle: never put a system in a state where rollback is impossible. Preserve the ability to return to a known-good state.

These are not coincidentally similar. They are expressions of the same architectural philosophy applied to different failure domains.

**Marcus Webb's final annotation on this document**: *"I've seen companies spend $50M fixing what 2 weeks of deployment discipline would have prevented. The security team and the platform team were in different buildings and never talked. The security team built a beautiful zero-trust architecture. The platform team deployed it via SSH to a mutable fleet with 3-year-old AMIs. The attackers found the fleet. The zero-trust network did not protect unmanaged servers. Build both. Make the teams talk."*

---

## Reflection Questions

1. You have now seen blue/green deployments described as a deployment strategy and zero-trust described as a security strategy. They share a common underlying principle. Articulate that principle in one sentence. Now name two other systems you have designed in this curriculum that also embody it — even if they were not described using that language.

2. Marcus Webb says "the authorization decision log is underrated." Think about the systems you have designed that involve access control: MeridianHealth, CloudStack, SentinelOps, SkyRoute. In which of those systems would an OPA-style decision log have provided the most operational value? What would you have been able to answer with it that you cannot currently answer with the existing designs?

3. The expand/contract migration pattern requires three separate deploys to add a single NOT NULL column. A simpler team would call this over-engineering for a small table. A large team would call it minimum viable discipline for a 4.2-billion-row table. Where is the threshold — at what table size, traffic volume, or deployment frequency does expand/contract become non-negotiable? Define your criteria.
