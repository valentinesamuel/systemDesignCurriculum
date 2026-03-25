---
**Title**: CarbonLedger — Chapter 165: The Ledger That Cannot Lie
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #database #compliance #distributed #immutability #event-sourcing
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 168 (Event Sourcing for Regulators), Ch. 52 (OmniLogix append-only inventory), Ch. 49 (CAP theorem consistency tiers)
**Exercise Type**: System Design
---

### Story Context

**#engineering — Slack, Monday 9:14 AM**

**Yara Osei** [VP Engineering]: @here before anyone asks — yes, the UN eval team is in the building today. Please do not approach them in the kitchen. Let the demo team handle it.

**Kwesi Ampah** [Lead Backend]: noted. also @new-hire your onboarding doc is in Notion but heads up — skip the "architecture overview" section. it's outdated by about 18 months.

**You**: got it, thanks. anything I should actually read?

**Kwesi Ampah**: read the incident report from March. the one titled "Voluntary Registry Incident — Root Cause." that'll tell you more about where we are than any doc.

---

You find the incident report in Notion. It's 34 pages. The executive summary is three sentences:

> *On March 14th, a batch processing error caused 847,000 carbon credits to be recorded with incorrect project IDs. The credits had already been transferred to four corporate buyers. The error was discovered 11 days later by an external auditor. Full remediation required manual database corrections across six tables.*

You read that last sentence twice. *Manual database corrections.* In a registry that's supposed to be immutable.

You close the laptop and walk to Yara's office.

---

**1:1 Transcript — Yara Osei / You, Monday 2:30 PM**

**You**: "I read the March incident report."

**Yara**: "Good. That's the one I wanted you to see."

**You**: "The remediation required direct database mutations. On what's supposed to be an immutable ledger."

**Yara**: "Yes."

**You**: "How did that pass the audit?"

**Yara** [pause]: "The auditor didn't ask to see the database change logs. They asked to see the credit records, and the records looked correct after remediation."

**You**: "That means the audit trail has a gap."

**Yara**: "Yes."

**You**: "And the UN eval team is here today."

**Yara**: "Yes. Which is why you're here. I need someone who can tell me: what does a registry that *actually* cannot lie look like? Not a registry that says it's immutable. One that *is* immutable. And I need you to design it in time for the technical review in three weeks."

**You**: "What's the current architecture?"

**Yara**: "PostgreSQL. Mutable rows. Soft deletes. A 'status' column. Audit log table that we write to... sometimes."

**You** [long pause]: "Okay."

**Yara**: "One more thing. You need to read the CEO's notes from the Nairobi pre-brief. Specifically the part about what happened with the voluntary market registries last year." She slides a printed document across the desk.

---

**CarbonLedger CEO Pre-Brief Notes — Nairobi Summit**

*...The voluntary market incident of 2024 demonstrated the existential risk of registry fragmentation. Three separate registries — Verra, Gold Standard, and a regional African registry — all issued credits referencing overlapping geographic boundaries in the Amazon. Satellite verification confirmed the rainforest credits were counted three times. Estimated double-counted volume: 14 million tonnes CO2e. The scandal set voluntary market prices back 40%. If this happens in the Article 6 compliance market, it doesn't just damage the market — it gives signatories legal grounds to withdraw from their NDC commitments...*

---

You put the document down. The problem isn't just "make the database immutable." The problem is that carbon credit double-counting is a climate fraud vector, and the registry is the last line of defense.

---

**Slack DM — Marcus Webb → You, Monday 6:51 PM**

**Marcus Webb**: heard you joined the ledger people. smart move. or insane. probably both.

**You**: three weeks to redesign the core registry before a UN audit. so... insane.

**Marcus Webb**: good. comfortable means you're not learning. one question before you start designing: what's the difference between a database that *records* immutability and a ledger that *enforces* it?

**You**: the enforcement mechanism is structural, not procedural?

**Marcus Webb**: closer. think about who has the power to lie. and then make sure that power doesn't exist. talk later.

---

### Problem Statement

CarbonLedger's carbon credit registry is currently built on a mutable PostgreSQL database with a soft-delete pattern and an inconsistently maintained audit log. A production incident in March demonstrated that direct database mutations were used to "fix" records — silently breaking the immutability guarantee that the platform's entire value proposition rests on.

You must design an immutable carbon credit registry where: (1) no record can ever be deleted or modified, only superseded by new events; (2) double-counting of credits is structurally impossible, not just procedurally prohibited; (3) every state transition is cryptographically auditable; and (4) the system can handle the UN Article 6 mandate which would increase transaction volume by 1,000x.

### Explicit Requirements

1. Every carbon credit issuance, transfer, and retirement must be recorded as an immutable event — no record can be updated or deleted
2. Each carbon credit must have a globally unique serial number; duplicate serial numbers must be rejected at the database constraint level
3. Double-counting prevention: a credit that has been transferred cannot be transferred again by the sender; a retired credit cannot be transferred or re-issued
4. Full audit trail: any credit's complete lifecycle must be reconstructable from events alone
5. GDPR compliance: project developer PII (operator identity) must be deletable on request while preserving the credit transaction record
6. System must handle 10K transactions/day today, with architecture that supports 10M/day without redesign
7. All writes must be acknowledged within 500ms at p99

### Hidden Requirements

- **Hint**: Re-read the CEO's pre-brief notes. What happened with the three registries and the Amazon rainforest? The system must prevent *cross-registry* double-counting, not just intra-registry. CarbonLedger needs to record which external registry a credit originated from and flag potential cross-registry conflicts.
- **Hint**: Re-read the March incident report carefully. "Manual database corrections across six tables." Why six tables? The current schema has denormalized credit state across multiple tables. A properly immutable design must be the single source of truth — no derived state stored in separate mutable tables.
- **Hint**: Yara said "the audit log table that we write to... *sometimes*." What does that mean for the current audit log's completeness? Your design must make audit logging *structural* — it cannot be an optional side effect.
- **Hint**: Marcus Webb asked "who has the power to lie?" Database administrators with direct SQL access can mutate any row. The design must address the privileged access problem — not just application-level immutability.

### Constraints

- **Current scale**: 50M credits under management, 10K transactions/day, 600 project developers, 1,400 corporate buyers
- **Target scale (UN adoption)**: 10M transactions/day, 196 country registries, 50K+ participants
- **Credit serial number format**: CR-{year}-{sequence} — globally unique, never reused
- **Latency SLA**: Write acknowledgment < 500ms p99; Credit history query < 2s p95
- **Storage**: Each event ~2KB; at 10K/day = ~20MB/day today; at 10M/day = ~20GB/day
- **GDPR**: Right to erasure applies to operator PII, NOT to transaction records
- **Team**: 14 engineers total; 4 backend engineers available for this work
- **Timeline**: 3 weeks to production-ready, UN technical review
- **Budget**: Current infrastructure ~$45K/month

### Your Task

Design an immutable carbon credit registry architecture. The design must address structural immutability (not just policy immutability), double-counting prevention, GDPR-compliant PII handling, and a path to 10M transactions/day.

### Deliverables

- [ ] Mermaid architecture diagram showing the immutable ledger architecture, including how state is derived from events
- [ ] Database schema: events table, credit serial number registry, state projection table, PII pseudonymization approach (with column types and indexes)
- [ ] Scaling estimation: storage growth at 10K/day and 10M/day, read/write RPS at each scale, partition strategy
- [ ] Tradeoff analysis (minimum 3): event sourcing vs. mutable state; append-only vs. traditional ACID; on-chain vs. off-chain immutability proof
- [ ] Double-counting prevention design: explain the structural mechanism that makes it impossible, not just the policy that prohibits it
- [ ] Cost modeling: storage cost at 10M/day events for 5 years (use S3 Glacier pricing for cold storage, RDS for hot)
- [ ] Capacity planning: 6-month horizon assuming linear growth from 10K to 10M transactions/day after UN announcement

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

```
Example structure to adapt:
graph TD
    A[Client API] --> B[Command Handler]
    B --> C[Event Store - append only]
    C --> D[State Projector]
    D --> E[Read Model - PostgreSQL]
    C --> F[Audit Log - WORM S3]
```
