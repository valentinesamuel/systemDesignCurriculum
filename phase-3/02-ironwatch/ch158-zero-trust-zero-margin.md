---
**Title**: IronWatch — Chapter 158: Zero Trust, Zero Margin
**Level**: Staff
**Difficulty**: 9
**Tags**: #zero-trust #auth #FIDO2 #access-control #audit #air-gapped #compliance
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 157, Ch. 159, Ch. 162
**Exercise Type**: System Design
---

### Story Context

IronWatch runs an internal air-gapped communication system called IRONCOMM — think Slack, but physically isolated on the SECRET network, no cloud dependency, no external APIs. It's clunky, the UI hasn't changed since 2017, but it's there. You've been given a read-only account on the UNCLAS instance so you can receive messages from the team without going through the SCIF. The SECRET and TS/SCI instances require physical presence.

Three days after the briefing room meeting, a message lands in IRONCOMM from Randall Tuck:

---

**[IRONCOMM — #infra-alerts — SECRET tier]**

**Tuck [09:14]:** Audit results are in from the external review last month. Sharing the summary — full report is in the secure document system.

**Tuck [09:14]:** Short version: 3 analysts flagged for over-broad access. Two were SECRET-tier analysts with read access to 14 TS/SCI compartment indexes — not the data, but the *existence* of those compartments. One UNCLAS analyst had been given a SECRET service account credential "temporarily" in March. It expired in... never. It was never set to expire.

**Tuck [09:15]:** CISO wants a complete access redesign. She used the phrase "zero-trust" four times in the debrief. I counted.

**Marguerite Osei [09:22]:** I'll be more precise: what we have now is perimeter-based access. Once you're on the SECRET network, you're broadly trusted. That's wrong. Every access decision — every query, every document open, every export — needs to be authorized against that specific user's current clearance, their current assignment, the classification of the specific resource, AND the device they're on. If any of those four things don't match, the request is denied. And logged. Always logged.

**Tuck [09:24]:** The device posture piece is going to be hard. We have 200 analyst workstations and most of them haven't had a compliance scan since Q3.

**Osei [09:26]:** That's why we're doing it now and not after the next audit.

**Tuck [09:27]:** One more thing from the audit report — worth flagging for the architect. The auditors found that password-based MFA (TOTP apps) can be phished even in an air-gapped environment via social engineering. They're recommending FIDO2-only. Hardware keys. No exceptions.

**Osei [09:28]:** Not a recommendation. A requirement. [The architect] — you have 2 weeks to produce an architecture.

---

You print the IRONCOMM thread (you can print from the UNCLAS terminal) and take it to SCIF room 3. The whiteboard still has the three-tier diagram from the briefing. You add a new column: **Identity & Access**.

The problem isn't exotic. You've built zero-trust systems before — at CloudStack (Ch. 30) you did SAML federation across multi-tenant enterprise customers; at SkyRoute (Ch. 61) you dealt with adversarial account types and OAuth2 delegation. But those systems had cloud infrastructure, managed identity providers, external IdPs. Here, you have no internet. The IdP is air-gapped. FIDO2 keys need an air-gapped registration ceremony. The policy engine can't phone home to any SaaS product.

You sketch the four factors Osei enumerated: user identity, current clearance level, resource classification label, device posture. Every access decision is an intersection of these four dimensions. Any mismatch is a deny. Every decision is an audit record.

You write on the whiteboard: **Never trust, always verify, always record.**

Then you add, because Marcus Webb's voice is still in your head: **And make sure the audit survives if the system doing the auditing is compromised.**

---

### Problem Statement

IronWatch's current access model trusts network perimeter as a primary authorization mechanism. An analyst on the SECRET network has broad read access to SECRET-tier resources by virtue of network adjacency alone. A recent audit found three cases of over-broad access, including a non-expiring service account credential in the wrong tier.

You must design a zero-trust access architecture for the SECRET and TS/SCI tiers. Every access decision must be evaluated against four factors: user identity + current clearance, resource classification label, device posture, and access context (time, location within facility, operational assignment). FIDO2 hardware keys are mandated for phishing-resistant MFA. The identity provider and policy engine must be fully air-gapped, with no external dependencies. All access decisions — grant and deny — must be logged with immutable, tamper-evident records.

---

### Explicit Requirements

1. FIDO2 hardware security keys only — no TOTP, no password-only paths
2. Policy engine evaluates four factors per request: user identity/clearance, resource classification, device posture, access context
3. Air-gapped IdP: all authentication and authorization infrastructure is physically on-premises with no internet dependency
4. Every access decision (grant and deny) produces an immutable audit record written to append-only storage
5. Service account credentials must have mandatory expiry enforced at the platform level — no non-expiring credentials
6. Device posture checks: OS patch level, disk encryption status, certificate validity must be evaluated per session
7. Existing IRONCOMM internal system must integrate with new auth layer without architectural replacement
8. Support 500 analysts: ~300 UNCLAS, ~150 SECRET, ~50 TS/SCI

---

### Hidden Requirements

- **Hint**: Re-read Tuck's line about 200 analyst workstations not having a compliance scan since Q3. Device posture must be evaluated dynamically, not at enrollment time only. What happens to active sessions when a device falls out of compliance mid-session?
- **Hint**: Osei says "every export" must be authorized. Export is a different action class than read. What does this imply about the policy model — is RBAC sufficient, or does this require ABAC with action-level granularity?
- **Hint**: The audit found analysts with access to "the existence of compartments" — the indexes — not the data itself. What does this imply about classification labels? Must metadata (document titles, compartment names) carry classification labels independently from their content?
- **Hint**: Re-read Osei's four factors: "current assignment." Analysts rotate between operational assignments. What does "current assignment" mean for a policy engine — is it static RBAC, or must it be dynamically evaluated against an operational assignment record that changes?

---

### Constraints

- **Analysts**: 500 (SECRET: 150 concurrent peak, TS/SCI: 50 concurrent peak)
- **Access decisions/sec**: estimated 50 RPS peak (document reads, queries, exports)
- **Latency SLA**: access decision < 50ms p99 (analysts are doing time-sensitive work)
- **Device fleet**: 200 workstations (SECRET), 40 workstations (TS/SCI) — all Windows, domain-joined
- **Air-gapped constraint**: zero external DNS, zero external APIs, zero cloud IdP
- **FIDO2 keys**: hardware procurement 2-3 week lead time
- **Audit log retention**: 7 years minimum (federal requirement)
- **Team**: 4 infrastructure engineers; 2-week design deadline
- **Compliance**: NIST SP 800-207 (Zero Trust Architecture), DoD Zero Trust Reference Architecture

---

### Your Task

Design the zero-trust access architecture for IronWatch. Include the air-gapped IdP, ABAC policy engine, device posture integration, FIDO2 enrollment ceremony, session management, and audit log pipeline. Address the credential lifecycle management failure from the audit explicitly.

---

### Deliverables

- [ ] Mermaid architecture diagram: air-gapped IdP, policy engine, device posture service, IRONCOMM integration, audit log pipeline
- [ ] Database schema for: user identity + clearance record, operational assignment record, device posture record, access policy, audit event — with column types, indexes, and partition strategy
- [ ] Scaling estimation:
  - Access decisions/sec: 500 analysts × 6 decisions/min average = X RPS
  - Audit log write rate: X events/sec × record size = storage/day, storage/7 years
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3):
  - ABAC policy engine vs. simpler RBAC
  - Session-persistent device posture vs. per-request re-evaluation
  - Air-gapped IdP operational complexity vs. SaaS IdP convenience (not available here but note the tradeoff for the analysis)
- [ ] Cost modeling: air-gapped IdP server hardware, FIDO2 key procurement, infrastructure operational overhead ($X/month estimate)
- [ ] Capacity planning: 12-month horizon (analyst count growth, new compartment addition rate)
- [ ] FIDO2 enrollment ceremony design: how are keys registered in an air-gapped environment? What is the recovery path if a key is lost?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
