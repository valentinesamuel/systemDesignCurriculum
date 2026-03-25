---
**Title**: IronWatch — Chapter 163: The Exit Brief
**Level**: Staff
**Difficulty**: 10
**Tags**: #RFC #design-review #compliance #audit-escrow #key-management #legal
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 162, Ch. 157, Ch. 158
**Exercise Type**: System Design / Design Review Ambush (SPIKE)
---

### Story Context

It is a Tuesday afternoon. You are scheduled to present the RFC for the audit system architecture transition to four people: Colonel Osei (CISO), Randall Tuck (Lead Infrastructure Engineer), Barrington Forde (Chief Technology Officer, IronWatch), and Dr. Lena Bauer, making a second appearance in her NSA advisory capacity.

There is a fifth person in the room you were not told about. She is introduced as Carolyn Marsh, General Counsel. She has a yellow legal pad and a pen that she does not put down for the entire meeting.

The RFC presentation takes twenty-two minutes. You cover the cryptographic chaining design, the per-tier WORM storage, the cross-tier correlation procedure, the hash verification architecture. Osei asks three technical questions. Bauer asks one, about the hash algorithm selection. Tuck nods throughout. Forde seems satisfied.

Then Carolyn Marsh speaks for the first time.

"Before we proceed — I want to ask about the audit log escrow scenario."

You ask her to clarify.

"IronWatch is a defense contractor. Our contracts can be terminated for convenience. Our company could be acquired. Theoretically — and I want to be very clear I'm speaking theoretically — IronWatch could cease to exist as an operating entity. When that happens, the government audit logs that IronWatch has been holding in its infrastructure for the duration of these contracts — what happens to them? Who holds the decryption keys? Who takes custody of seven years of classified audit records?"

The room is very quiet.

She continues: "The regulation I'm thinking of is DFARS 252.204-7012. When a contractor stores government data, there are custodial obligations that survive the contractor's operating life. If IronWatch goes dark tomorrow — acquisition, bankruptcy, whatever — those audit logs need to be readable by government auditors five years from now. Right now, the decryption keys live in IronWatch's infrastructure. If that infrastructure goes dark, so do the keys."

Forde: "Carolyn, why are we discussing bankruptcy—"

Marsh: "I'm discussing it because it's a contractual obligation we haven't addressed in the architecture. I'm also discussing it because our parent company's acquisition talks became active last week. I thought the architect should know."

The air in the room changes. Osei looks at Forde. Forde looks at his hands. Bauer writes something on her notepad and underlines it.

You buy yourself six seconds by uncapping your marker and walking to the whiteboard. What Marsh is describing is an audit log escrow architecture: a design where the government — specifically, a designated government custodian — holds a separate copy of the decryption keys (or the audit data itself) that cannot be lost regardless of IronWatch's operational status. The architecture you've just presented has the keys in IronWatch's key management system. That's the problem.

Forde: "How long do you need to revise the RFC?"

You think about the OrbitCore cell-based architecture. The CarbonLedger permanence problem. The NovaPay outbox pattern. Every complex system problem comes down to the same thing: who holds state, and what happens when the holder goes away?

"One week," you say. "And I need a written legal brief from General Counsel on the specific regulatory requirements so I'm designing against the actual obligation, not my interpretation of it."

Marsh: "You'll have it by Thursday."

Bauer nods slightly and writes something else on her notepad.

---

**[IRONCOMM — DM — Randall Tuck → You]**

**Tuck [18:42]:** That was not a normal RFC review.

**Tuck [18:43]:** For what it's worth — you handled it well. Better than the previous architect would have. He would have argued with Carolyn for twenty minutes and then sulked for a week.

**You [18:45]:** I almost argued. The acquisition thing was a surprise.

**Tuck [18:46]:** Yeah. I found out yesterday. Welcome to government contracting.

**Tuck [18:47]:** One thing Carolyn didn't say out loud: the government custodian for key escrow in DFARS 252.204-7012 scenarios is typically a designated Defense Contract Audit Agency (DCAA) officer. Which means your escrow design has to be readable by a civilian government auditor, not a cleared contractor. Think about what that means for the classification of the escrow keys themselves.

You stare at that last message for a long time.

---

### Problem Statement

During the RFC review for IronWatch's audit log transition, General Counsel revealed a regulatory requirement that was absent from the existing design: under DFARS 252.204-7012 and related contractor data custodial obligations, the government must retain the ability to read audit logs even if IronWatch ceases to exist as an operating entity. The current design stores decryption keys within IronWatch's own key management infrastructure. An acquisition or dissolution event could render seven years of classified audit logs permanently inaccessible to government auditors.

You must redesign the audit log architecture to include an escrow mechanism: government-held decryption keys (or equivalent custody design) that survives IronWatch's operational lifecycle. The escrow mechanism must be usable by a DCAA auditor who may not hold a TS/SCI clearance, which creates a classification handling challenge for the escrow keys themselves.

---

### Explicit Requirements

1. Government key escrow: designated government custodian (DCAA/DoD) must hold a copy of decryption keys for all audit logs, independent of IronWatch's infrastructure
2. Escrow activation: escrow keys must be usable by government auditors in an escrow activation scenario (contractor dissolution, contract termination, acquisition)
3. Regular escrow updates: escrow must be updated whenever encryption keys are rotated (keys rotate per policy; escrow must not lag more than 24 hours)
4. Audit log integrity: the escrow copy of keys must be verifiable against the live audit logs — an auditor must be able to confirm the escrow keys are current and correct without decrypting the entire archive
5. Classification handling: DCAA auditors may not hold TS/SCI clearances — escrow key material for TS/SCI tier audit logs requires a specific handling design
6. Audit of the escrow itself: key escrow transfers must be logged in a tamper-evident system; the escrow process must itself be auditable
7. The existing cryptographic chaining and WORM storage design (Ch. 162) must be preserved and extended — not replaced

---

### Hidden Requirements

- **Hint**: Re-read Tuck's final IRONCOMM message about the classification of escrow keys. The escrow keys for TS/SCI audit logs are themselves TS/SCI material. A DCAA auditor without TS/SCI clearance cannot hold them. But the regulatory requirement says the government must be able to access the logs. What design pattern resolves this contradiction — is the answer in how you structure the data, or how you structure the access ceremony?
- **Hint**: Re-read Marsh's framing: "acquisition talks became active last week." The architecture must be graceful under a time-pressured acquisition scenario. What is the operational procedure for transferring key escrow custody when there are 72 hours' notice? Is this a system design problem, a process design problem, or both?
- **Hint**: Re-read the requirement that escrow must be updated within 24 hours of key rotation. The TS/SCI tier is air-gapped. Key rotation events on the TS/SCI network cannot trigger outbound notifications. How does the escrow update process work for an air-gapped tier?
- **Hint**: The requirement says an auditor can "verify the escrow keys are current and correct without decrypting the entire archive." This is a cryptographic commitment problem. What technique (e.g., key commitment scheme, key fingerprint registry) satisfies this requirement without requiring full decryption?

---

### Constraints

- **Regulatory basis**: DFARS 252.204-7012, NIST SP 800-57 (key management), DoD 5015.02 (records management)
- **Escrow custodian**: DCAA or designated DoD representative; may or may not hold TS/SCI clearance
- **Key rotation cadence**: 90-day rotation for all tiers (NIST recommendation)
- **Escrow update SLA**: within 24 hours of any key rotation event
- **Air-gap constraint**: TS/SCI tier key rotation events cannot trigger outbound network activity
- **Audit log volume**: 7 years × ~120K events/day × 500 bytes = significant compressed archive
- **Timeline**: 1 week to produce revised RFC; legal brief from General Counsel by Thursday
- **Budget signal**: escrow infrastructure is a compliance cost; cost is secondary to correctness
- **Team**: you, Tuck, and legal guidance from Carolyn Marsh

---

### Your Task

Redesign the IronWatch audit log architecture to include government key escrow. Produce the revised RFC structure, the escrow design, the classification handling approach for TS/SCI escrow keys, the air-gap-compatible key rotation + escrow update process, and the cryptographic commitment scheme for escrow verification.

---

### Deliverables

- [ ] Mermaid architecture diagram: full audit system including escrow path, government custodian interface, per-tier key management, air-gapped TS/SCI escrow update procedure
- [ ] Revised RFC document structure (as a deliverable, not just the system):
  - Title, Problem Statement, Proposed Solution, Alternatives Considered, Risks, Compliance References, Approval Required
- [ ] Database schema for key escrow registry: key_id, tier, effective_date, expiry_date, escrow_transfer_timestamp, escrow_custodian_id, key_fingerprint, verification_hash
- [ ] Scaling estimation:
  - Escrow update frequency: 3 tiers × 90-day rotation = X key escrow transfers/year
  - Key archive size: key material × 7 years × 3 tiers
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3):
  - Split-knowledge key escrow vs. full key escrow (security vs. operational simplicity for government custodian)
  - Pull-based escrow update (government polls) vs. push-based (IronWatch delivers) — given air-gap constraints
  - Centralized escrow custodian vs. distributed government custodian (single point of failure vs. coordination overhead)
- [ ] Cost modeling: escrow infrastructure, HSM hardware for key management, operational transfer ceremonies ($X/month + capital)
- [ ] Capacity planning: 12-month horizon (key rotation events, new contract additions, potential acquisition timeline)
- [ ] Classification handling design: step-by-step procedure for how a DCAA auditor accesses TS/SCI audit log escrow in an activation scenario without violating classification handling requirements

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
