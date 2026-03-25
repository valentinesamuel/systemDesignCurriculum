---
**Title**: NexaCare — Chapter 70: GDPR Deletion Pipelines
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #gdpr #compliance #event-sourcing #data-deletion #privacy #cryptographic-erasure
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 66, Ch. 131
**Exercise Type**: System Design / Compliance
---

### Story Context

**Email Chain**

---
**From**: dr.lisa.baumann@nexacare.io
**To**: legal@nexacare.io; fatima.al-rashid@nexacare.io
**Subject**: URGENT — GDPR Right to Erasure Request — Patient ID EU-2024-08847
**Date**: Friday 16:44

Team,

We have received a formal GDPR Article 17 erasure request from a patient enrolled in Trial ARM-EU-2019-B. The patient's legal representative submitted this through the trial sponsor's data subject rights portal at 16:31 today.

GDPR mandates a response within 30 days. However, I am told publicly that NexaCare guarantees "24-hour data deletion." I did not know about this commitment. Can someone explain?

Additionally, I need to understand the technical scope. Patient data in our system includes: direct trial participation records, de-identified data that was previously identified, event logs in Kafka, Redshift analytics tables, S3-based audit archives, and DR backups in Glacier. Are all of these covered?

Please advise urgently.

Dr. Lisa Baumann
Chief Compliance Officer

---
**From**: fatima.al-rashid@nexacare.io
**To**: you@nexacare.io
**Subject**: FWD: GDPR request — your first real project
**Date**: Friday 16:52

The "24-hour deletion" commitment was made by a salesperson two years ago and is now in 14 enterprise contracts. I found out today.

I need you to:
1. Figure out what "full deletion" technically means across our data estate
2. Tell me if we can actually do it in 24 hours
3. Design the system that makes it possible going forward

Monday 9am. My office.

— Fatima

---

**Monday 09:00 — Fatima's Office**

**You**: I spent the weekend mapping the data. Patient data exists in 12 locations: the primary Postgres database, the Kafka event topics, the CQRS projection tables, the Redshift analytics warehouse, the S3 audit archive, the Glacier backup copies, and 6 microservice databases that receive copies via CDC.

**Fatima**: And?

**You**: The 24-hour commitment is technically impossible with our current architecture. Glacier restores alone take 12 hours. The Kafka event log — our event sourcing layer — is set to 30-day retention with no deletion capability. And there's a conflict I need to surface immediately.

**Fatima**: What conflict?

**You**: Patient EU-2024-08847 is enrolled in an *active* trial. Under FDA ICH E6 Good Clinical Practice guidelines, trial data must be retained for the duration of the trial plus at minimum 2 years after completion. The trial ARM-EU-2019-B runs until Q3 2025.

**Fatima**: So GDPR says delete. FDA says retain.

**You**: That's the conflict. They're both legally binding and they directly contradict each other.

**Fatima**: *(long pause)* I need you to design a system that satisfies both, or design an escalation path for the cases where that's impossible. Which data can we delete without violating FDA retention? Which data must we retain, and how do we do so in a GDPR-compliant way?

**You**: I have some ideas about cryptographic erasure.

**Fatima**: Tell me at 2pm. Bring Dr. Baumann.

---

**Slack DM — Marcus Webb → You**
`Monday 11:30`
**marcus.webb**: heard about the NexaCare situation through Priya. GDPR vs FDA is a classic compliance trap.
**you**: how do other platforms handle it?
**marcus.webb**: the good ones separate "right to erasure" from "right to be forgotten." erasure doesn't always mean deletion. it can mean rendering data unreadable. that's where cryptographic erasure comes in.
**you**: encrypt with a patient-specific key, then destroy the key?
**marcus.webb**: you already know the answer. the question is whether your event log is designed to support per-record encryption. is it?
**you**: ...not currently
**marcus.webb**: that's what 14 months of pending deletion requests look like

---

### Problem Statement

NexaCare's 14 enterprise contracts include a 24-hour data deletion guarantee. Patient data exists across 12 storage locations in a distributed architecture that includes event-sourced Kafka logs, Postgres databases, S3 archives, and Glacier backups. Additionally, some patients requesting deletion are enrolled in active clinical trials, creating a direct conflict between GDPR Article 17 (right to erasure) and FDA ICH E6 (trial data retention requirements).

You need to design a GDPR deletion pipeline that satisfies the 24-hour SLA where technically possible, handles the FDA retention conflict correctly, and introduces cryptographic erasure as the technical mechanism for data in systems where hard deletion is architecturally infeasible.

---

### Explicit Requirements

1. Soft delete cascade must complete within 24 hours for all mutable storage systems
2. Event log data (Kafka, S3 archive) must be rendered unreadable for the requesting patient within 24 hours
3. FDA-retained data must remain technically present but must be flagged as "erasure-requested — retained for regulatory obligation"
4. Cryptographic erasure: patient data encrypted with patient-specific key; key destruction must be auditable
5. Backup copies (Glacier) must be scheduled for deletion on their next restore cycle
6. The deletion pipeline must generate an auditable erasure certificate suitable for GDPR Article 5(2) accountability

---

### Hidden Requirements

- **Hint**: Re-read Dr. Baumann's email. She lists "de-identified data that was previously identified" as a separate category. De-identification is not permanent if re-identification is possible. If NexaCare's de-identified analytics data can be re-linked to Patient EU-2024-08847 using the trial participation record, does the deletion obligation extend to the de-identified data?

- **Hint**: The event log has 14 months of pending deletion requests. Those 847 patients who requested deletion — some of them may be in Redshift analytics tables that feed downstream reports that were already shared with trial sponsors. What's the obligation for downstream data copies that have already left NexaCare's control?

---

### Constraints

- 12 data storage locations (Postgres, Kafka x3 topics, S3 audit, Glacier, Redshift, 6 microservice DBs)
- 24-hour SLA for erasure (contractual — 14 enterprise contracts)
- Glacier restore time: 4–12 hours depending on retrieval tier
- Kafka topic retention: 30 days (hard-coded, would require re-architecture to support deletion)
- FDA ICH E6 retention: trial duration + 2 years minimum
- 847 existing pending deletion requests, oldest 14 months old

---

### Your Task

Design the GDPR deletion pipeline for NexaCare's platform. Include the cryptographic erasure approach for event logs, the cascade deletion mechanism for mutable storage, the FDA conflict resolution framework, and the erasure certificate generation.

---

### Deliverables

- [ ] Mermaid diagram: Deletion pipeline showing all 12 data locations and the deletion/erasure action for each
- [ ] Cryptographic erasure design: Per-patient encryption key management (where keys are stored, how destruction is recorded, what "erased" records look like)
- [ ] FDA conflict resolution framework: Decision tree for "can this patient record be deleted?" with explicit handling for active trial participants
- [ ] Erasure certificate schema: What fields constitute a legally defensible GDPR erasure certificate
- [ ] Scaling estimation: At 50 deletion requests/week, what's the processing time for the cascade? Can you hit 24 hours?
- [ ] Tradeoff analysis (minimum 3):
  - Soft delete vs hard delete for mutable Postgres tables
  - Cryptographic erasure vs log compaction for Kafka event streams
  - 24-hour SLA vs tiered SLA (24hr for mutable, 72hr for archival)
- [ ] Glacier backup strategy: How do you handle the 4–12 hour restore time within the 24-hour SLA?
