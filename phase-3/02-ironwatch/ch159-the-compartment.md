---
**Title**: IronWatch — Chapter 159: The Compartment
**Level**: Staff
**Difficulty**: 9
**Tags**: #ABAC #access-control #compartmentalization #policy-engine #compliance #audit
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 157, Ch. 158, Ch. 162
**Exercise Type**: System Design
---

### Story Context

The TS/SCI tier has twelve active compartments. You know this because Randall Tuck wrote it on a sticky note that he then peeled off and fed into a shredder. He said the compartment names themselves are sensitive, which is a sentence that recalibrated your threat model considerably.

Compartments are sub-divisions within TS/SCI. An analyst might hold TS/SCI clearance but only have access to compartments ALPHA and DELTA, not BRAVO, GAMMA, or any of the other eight. The problem is that the current system enforcing this is a manual process: one sysadmin — a contractor named Devon Marsh, who also manages the SECRET network and the UNCLAS helpdesk — manually adjusts Active Directory group membership when analysts are added to or removed from a compartment. Devon has a spreadsheet. It lives on his laptop. His laptop is not backed up to any central system because "the backup server is in the UNCLAS wing and I can't put TS/SCI stuff there."

You ask Tuck how many times Devon's spreadsheet has been wrong.

Tuck looks at the ceiling. "Definitively wrong? Twice that we caught. Unknown number of times we didn't."

The Inspector General review in 90 days will include a specific audit of compartment access: every analyst, every compartment, every grant and revocation. The IG will want to see an automated, auditable system. Devon's spreadsheet will not satisfy them. Devon is also, as Tuck mentions almost in passing, planning to retire in five months.

Colonel Osei pulls you aside in the corridor after your meeting. She has a specific concern: "The policy engine itself — whoever controls the policy engine controls access to everything. I want the policy engine to be auditable independently. I want to know who changed a policy, when, and why, and I want that audit to be immutable and separate from the policy engine's own storage." She lowers her voice. "I've seen insider threats work by compromising the access control system itself. The policy system has to be outside its own blast radius."

You write two things on your notepad: *ABAC* and *policy audit != data audit*.

Then you write a third thing: *single points of failure: Devon's laptop, Devon's retirement.*

Back in SCIF room 3, you map the twelve compartments (labeled C1 through C12 in your working documents) against the analyst roster. The access matrix has 50 rows and 12 columns. 600 cells. Each one is either "authorized" or "not authorized." It changes whenever an analyst is read-in to a new compartment program or read-out. These changes must be tracked with the same rigor as the access events they enable.

The challenge is structural: ABAC policy for compartments is not just "does user U have attribute COMPARTMENT_ALPHA?" It's "does user U have attribute COMPARTMENT_ALPHA *and* is that authorization currently active *and* was it granted through an approved read-in ceremony *and* has it not been suspended pending review?" Each of those conjuncts is a separate data point with its own lifecycle.

You think about the IRT model from NeuroLearn, the Saga pattern from OmniLogix, the MVCC internals you worked through at Stratum. None of them directly apply. This is something different: access state that has its own temporal history and must be queryable not just for current state but for historical state at any past timestamp.

Bi-temporal. Valid time and transaction time. You draw a small diagram, annotate it, and feel the shape of the solution begin to form.

---

### Problem Statement

IronWatch's TS/SCI tier hosts 12 active compartments. Compartment access is currently managed manually by a single sysadmin via an unbacked spreadsheet. This is a single point of failure (both technically and in terms of personnel), and it cannot produce the auditable, automated access history required by the IG review.

You must design an Attribute-Based Access Control (ABAC) system for TS/SCI compartment boundary enforcement. The policy engine must be independently auditable — changes to policies themselves must be logged in a tamper-evident system separate from the policy engine's operational storage. Compartment authorization must be modeled bi-temporally: every grant and revocation must be queryable at any historical timestamp with both valid-time and transaction-time semantics. Devon's spreadsheet must be eliminated. Devon's retirement must not be a bus factor.

---

### Explicit Requirements

1. ABAC policy engine evaluating user attributes (clearance, compartment authorizations, assignment, device posture) against resource attributes (classification, compartment label, data sensitivity)
2. 12 active compartments; compartment names are themselves sensitive metadata
3. All compartment authorization changes (read-in, read-out, suspension) must be recorded with immutable audit trails
4. Policy engine changes (new policies, policy modifications, policy deletions) must be logged in a separate audit system with cryptographic chaining
5. Bi-temporal authorization records: valid time (when the authorization was operationally in effect) and transaction time (when it was recorded in the system)
6. Support for retroactive queries: "Was analyst X authorized for compartment C7 on 14 March at 11:00 UTC?" must be answerable
7. Eliminate single-person dependency for compartment access management
8. Policy engine must evaluate authorization requests in < 20ms p99 (inline in request path)

---

### Hidden Requirements

- **Hint**: Re-read Osei's requirement that "the policy system has to be outside its own blast radius." What happens to existing authorization grants if a compromised admin attempts to delete all policy records? The bi-temporal model is relevant here — what does it mean for policy records to be immutable?
- **Hint**: Tuck mentions Devon also manages SECRET and UNCLAS. What does this mean for the blast radius of Devon's account compromise? Should the policy engine for TS/SCI compartments share any administrative path with the lower-tier systems?
- **Hint**: Re-read "pending review" in the compartment authorization lifecycle. Suspension is a distinct state from revocation. What does a suspended authorization mean for active sessions? Must a session be terminated immediately on suspension, or at next session boundary?
- **Hint**: The IG review wants historical access records. What does bi-temporal modeling mean for the database schema — specifically, how do you represent the difference between "the authorization was actually in effect on date X" vs. "we recorded on date X that the authorization was in effect"?

---

### Constraints

- **TS/SCI analysts**: 50 total; peak concurrent: 30
- **Compartments**: 12 active; estimated 2-3 new per year; deactivations rare
- **Authorization change events**: estimated 100-300/month (read-ins + read-outs + suspensions)
- **Policy evaluation latency**: < 20ms p99 (inline access path)
- **Policy audit retention**: 7 years minimum, WORM storage preferred
- **Team**: 2 engineers available for TS/SCI systems (clearance-gated)
- **Compliance**: DCSA DAAPM, ICD 704 (Personnel Security), DoD 5205.07 (SAP Policy)
- **Constraint**: Policy engine must operate with zero external network dependencies

---

### Your Task

Design the ABAC policy engine for IronWatch TS/SCI compartment enforcement. Produce the data model (bi-temporal authorization records), policy evaluation architecture (inline request path), policy audit system (independent append-only store with cryptographic chaining), and administrative workflow for compartment read-in/read-out ceremonies.

---

### Deliverables

- [ ] Mermaid architecture diagram: ABAC policy engine, authorization store (bi-temporal), policy audit store, request evaluation flow, admin workflow
- [ ] Database schema (with column types and indexes):
  - `compartment_authorization` (bi-temporal: valid_from, valid_to, transaction_from, transaction_to)
  - `policy_rule` (ABAC policy table with WORM constraint design)
  - `policy_audit_event` (cryptographic chain: hash of previous entry)
  - `access_evaluation_log` (per-request decision log)
- [ ] Scaling estimation:
  - Policy evaluations/sec: 50 analysts × 6 accesses/min = X RPS
  - Authorization record volume: 50 analysts × 12 compartments × bi-temporal entries over 7 years
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3):
  - Bi-temporal modeling vs. simple audit log (performance vs. queryability)
  - Inline policy evaluation vs. pre-computed authorization cache (latency vs. staleness)
  - Separate policy audit store vs. unified audit store (blast radius vs. operational complexity)
- [ ] Cost modeling: policy engine server, WORM audit storage, admin tooling ($X/month estimate)
- [ ] Capacity planning: 12-month horizon (compartment growth, analyst roster changes, IG review readiness)
- [ ] Read-in ceremony workflow: step-by-step process for granting new compartment authorization with dual-control approval and audit trail

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
