---
**Title**: OrbitCore — Chapter 152: Sovereignty in Orbit
**Level**: Staff
**Difficulty**: 9
**Tags**: #data-sovereignty #compliance #gdpr #itar #data-residency #security #distributed
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 8 (CivicOS data sovereignty), Ch. 38 (CloudStack GDPR), Ch. 152 (OrbitCore cell architecture)
**Exercise Type**: System Design
---

### Story Context

It is March 18th. Nine days into the job.

**[Email chain]**

**From:** Miriam Schultz <miriam.schultz@vantage-dynamics.eu> [Vantage Dynamics — EU Constellation Operator]
**To:** James Whitfield <james.whitfield@orbitcore.io>
**CC:** Legal <legal@orbitcore.io>, Ingrid Solberg <ingrid@orbitcore.io>
**Subject:** RE: Data Processing Agreement Review — URGENT
**Date:** March 18, 08:14am CET

> James,
>
> Our DPA review (initiated by our internal compliance team following the Schrems II guidance updates) has identified a concern we need to resolve before our contract renewal on April 1st.
>
> Article 14 of our agreement states that all telemetry data from the Vantage constellation must be processed exclusively within EEA-sovereign infrastructure. Our IT audit team has flagged log records indicating that telemetry processing jobs for VAT-07 through VAT-11 have been executed on infrastructure located in us-east-1 (AWS Virginia).
>
> This appears to be a violation of GDPR Article 44 (restriction on transfers) as well as our contractual DPA terms.
>
> We require:
> 1. A written explanation of how this occurred and the duration of the violation
> 2. A remediation timeline
> 3. Technical architecture documentation demonstrating ongoing compliance
>
> Our contract renewal is April 1st. If we cannot confirm remediation by March 28th, Vantage Dynamics will be required to initiate contract termination proceedings.
>
> This is not a negotiating position. This is a compliance requirement.
>
> — Miriam Schultz, DPA Compliance Officer, Vantage Dynamics

---

**[Slack DM — Ingrid Solberg → You, 9:17am]**

> **Ingrid Solberg**
> Have you seen the Vantage email?

> **You**
> Just read it.

> **Ingrid Solberg**
> This is not a surprise. Tomás flagged this 8 months ago. We never fixed it.
> You now own the fix. March 28th.
> Also: Legal is going to ask you to be in a call with Miriam Schultz's team Thursday.

> **You**
> Ten days.

> **Ingrid Solberg**
> Nine. I need the architecture doc by Wednesday so Legal can review before Thursday's call.

---

**[Slack channel — #legal-eng-sync, March 18, 11:03am]**

> **Petra Hoffmann** [General Counsel]
> @you — I've been briefed by Ingrid. Before you design the solution, please read Article 46 SCC (Standard Contractual Clauses) and our existing DPA with Vantage. Also: the UK satellite arc (ArcticLens constellation) has a different legal basis post-Brexit. UK GDPR is equivalent but not identical. Don't conflate them.

> **You**
> Understood. Any other constellations with sovereignty requirements I should know about?

> **Petra Hoffmann**
> TerraWatch is unclassified US government. Technically falls under the Cloud Act. The contract says data must remain on US soil.
> And there's one more: SV-23. It's a dual-use payload on the Kepler constellation. The payload is ITAR-controlled. That satellite's telemetry has to be treated as export-controlled material.
> We haven't documented any of this formally. Which is its own compliance problem.

> **You**
> I'm going to need a list of every satellite and its data classification.

> **Petra Hoffmann**
> I'll send you what we have. It's not complete. Some of the Kepler satellites have payload amendments that engineering never received copies of.

---

**[Email — Petra Hoffmann → You, 12:47pm]**

**From:** Petra Hoffmann
**To:** [You]
**Subject:** Satellite Data Classification Inventory (PARTIAL)
**Attachments:** satellite-classification-draft-v2.xlsx

> Attached is our best current inventory. Notes:
>
> - Highlighted in yellow: classification status UNCERTAIN — needs confirmation with constellation operator
> - SV-23 (Kepler): ITAR-restricted payload. Data classification: ITAR EAR99-adjacent. Requires US-only processing. Air-gap equivalent for payload channel telemetry. Do NOT route to any non-US infrastructure under any circumstances, including during failover.
> - VAT-07 through VAT-17 (Vantage): GDPR-covered. EEA-only processing. Standard SCC applies.
> - ARC-01 through ARC-08 (ArcticLens): UK GDPR. UK-only processing.
> - TWC-01 through TWC-14 (TerraWatch): Cloud Act. US-only processing.
> - KEP-01 through KEP-22 except SV-23: No current restrictions, but contractual data residency preference is US.
>
> The yellow rows are my concern. There are 7 satellites with uncertain classification. I need a design that handles re-classification gracefully — if a satellite's status changes, the routing must update without downtime.

---

**[#eng-leads Slack, March 18, 3:30pm]**

> **Tomás Reyes**
> I want to flag a technical constraint on the SV-23 ITAR requirement. The dual-use payload emits on a separate telemetry channel (channel 4) alongside the standard housekeeping telemetry (channels 1-3). Currently ALL channels from SV-23 route together. We'd need to demultiplex at the ground station level before the data touches any non-US infrastructure.
> Also — during a ground station failover, if the US ground station handling SV-23 goes offline, the current system would route to the nearest available station. That might be Singapore or Sydney. That would be an ITAR violation.
> The failover logic doesn't know about ITAR classifications.

You stare at that message for a long time.

### Problem Statement

OrbitCore's satellite telemetry pipeline routes all data through a shared global infrastructure without enforcing data sovereignty at the ingestion layer. This has resulted in GDPR-covered EU constellation telemetry being processed on US infrastructure, and creates ongoing risk of ITAR violations during ground station failover events.

Design OrbitCore's data sovereignty architecture: a system that enforces per-satellite data residency requirements at the ingestion layer, handles re-classification of satellite data status without downtime, and ensures that sovereignty constraints are respected even during ground station failover — including the ITAR-restricted SV-23 scenario where failover to non-US infrastructure is categorically prohibited.

### Explicit Requirements

1. EU constellation telemetry (Vantage, 11 satellites, VAT-07 to VAT-17) must be processed exclusively within EEA infrastructure — enforced at ingestion, not just at rest
2. UK constellation telemetry (ArcticLens, 8 satellites) must be processed exclusively on UK infrastructure (UK GDPR, post-Brexit)
3. TerraWatch (14 satellites) must remain on US infrastructure (Cloud Act)
4. SV-23 ITAR payload channel (channel 4) must never touch non-US infrastructure, including during failover
5. Satellite classification status must be changeable without pipeline downtime (hot re-routing)
6. Classification enforcement must be auditable — every routing decision must be logged
7. The system must handle 7 currently-unclassified satellites gracefully (default: most restrictive routing, flag for review)
8. Remediation evidence must be producible for Vantage Dynamics by March 28th

### Hidden Requirements

1. **Hint: re-read Tomás's message about SV-23 channel demultiplexing.** The dual-use payload emits on channel 4, while housekeeping telemetry is on channels 1-3. SV-23's housekeeping data has no ITAR restriction — only channel 4. Your architecture must demultiplex at the ground station before routing. What does this mean for your ground station agent design?

2. **Hint: re-read Petra's note about SV-23 failover.** She says "do NOT route to any non-US infrastructure under any circumstances, including during failover." This means if the US ground station handling SV-23 goes dark and there is no US alternative in SV-23's visibility window, the correct behavior is to DROP or BUFFER the ITAR-channel telemetry until a US station becomes available. Design this explicitly.

3. **Hint: re-read Petra's email about yellow-row satellites.** She says "if a satellite's status changes, the routing must update without downtime." This implies a live routing policy update mechanism. What is the propagation latency of a policy change, and what happens to in-flight events during a policy update?

4. **Hint: re-read the email thread from Miriam Schultz.** She mentions "log records indicating that telemetry processing jobs for VAT-07 through VAT-11 have been executed on infrastructure located in us-east-1." This means there are EXISTING audit records of the violation. Your remediation architecture needs to produce evidence of compliance from the fix-date forward — but you should also consider what happens to historical data that was processed in violation.

### Constraints

- **Satellites with sovereignty requirements**: 40 of 47 (85%) — this is not a rare edge case
- **SV-23 ITAR channel failover behavior**: DROP and BUFFER until US station available (no exceptions)
- **Policy update propagation latency**: < 5 seconds from policy change to enforcement
- **In-flight events during policy update**: must be handled (not silently re-routed)
- **Audit log requirement**: every routing decision, with satellite ID, classification used, infrastructure location, timestamp
- **Remediation deadline**: March 28th (10 days from story "now")
- **Vantage contract renewal**: April 1st
- **Jurisdictions**: EEA (GDPR), UK (UK GDPR), US (Cloud Act), ITAR (EAR)
- **Ground station sovereignty**: US (3 stations), EU (3 stations), UK (2 stations), Kenya/Singapore/Australia (outside any constellation's jurisdiction — these can only handle unclassified traffic)

### Your Task

Design OrbitCore's data sovereignty enforcement architecture. Cover: the satellite classification registry (the source of truth for routing policy), the ground station routing agent (how it reads and enforces policy), the channel demultiplexing layer for SV-23, the ITAR-channel buffer design for failover scenarios, the audit logging system for routing decisions, and the policy hot-reload mechanism.

### Deliverables

- [ ] Mermaid architecture diagram showing:
  - Sovereignty enforcement zones (EU, UK, US, ITAR-restricted)
  - Classification registry and its propagation to ground station agents
  - SV-23 channel demultiplex flow
  - ITAR-channel buffer and conditional routing
- [ ] Satellite classification registry schema:
  - Table definition with column types and indexes
  - How policy changes are versioned and propagated
  - What "uncertain" classification maps to in routing logic
- [ ] Ground station routing agent design:
  - How it reads current policy
  - How it enforces sovereignty without adding > 10ms to the ingestion path
  - How it handles policy updates for in-flight events
- [ ] SV-23 failover design:
  - Buffer design for ITAR channel during US-station outage
  - Buffer capacity sizing (how long can SV-23 ITAR data be held? what's the orbital period before SV-23 re-enters a US ground station's visibility window?)
  - What triggers drain of the buffer
- [ ] Audit log schema and architecture:
  - Per-routing-decision log entry (fields, types, indexes)
  - Tamper-evidence mechanism
  - How to produce compliance evidence for Vantage Dynamics
- [ ] Remediation plan for existing violation:
  - What happened to EU telemetry that was processed in us-east-1?
  - Can it be retroactively re-processed in EU? Should it be?
  - What do you tell Miriam Schultz on Thursday's call?
- [ ] Tradeoff analysis (minimum 3):
  - Per-ground-station policy enforcement vs. central routing proxy
  - Eager drop (ITAR data dropped immediately if no valid station) vs. buffer-and-wait
  - Strict-enforcement-by-default vs. permissive-with-alert for unclassified satellites
- [ ] Cost modeling:
  - Additional infrastructure cost of sovereignty-segregated processing vs. shared global pipeline
  - Cost of buffering ITAR channel data during US-station outages

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

Key diagram: sovereignty enforcement decision tree at the ground station agent level.
