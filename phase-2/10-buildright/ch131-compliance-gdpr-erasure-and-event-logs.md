---
**Title**: BuildRight — Chapter 131: GDPR Erasure in Event-Sourced Systems
**Level**: Staff
**Difficulty**: 9
**Tags**: #gdpr #event-sourcing #compliance #cryptographic-erasure #data-retention #legal
**Estimated Time**: 6 hours
**Related Chapters**: Ch. 70 (NexaCare GDPR, simpler version), Ch. 126 (BuildRight event sourcing), Ch. 131 (this chapter — hardest GDPR chapter)
**Exercise Type**: System Design / Compliance Emergency
---

### Story Context

The request is polite, formal, and arrives via registered mail.

---

**From**: Dr. Klaus Brandt, Data Protection Attorney
**To**: BuildRight Legal (via registered post + email)
**Subject**: DSGVO / GDPR Art. 17 — Erasure Request — Client: Heinrich Vogel, Subcontractor DE-4482
**Date**: March 5, 2026

Dear BuildRight GmbH (European Entity),

I represent Heinrich Vogel, a German national and registered subcontractor on the BuildRight platform (account ID: BR-DE-4482). Mr. Vogel ceased operations in December 2025 and formally requests erasure of all personal data under Article 17 of the EU General Data Protection Regulation (GDPR) / DSGVO.

Mr. Vogel's personal data includes: full name, contact information, professional certifications, and contribution records associated with the following projects between 2021 and 2025: [list of 47 project IDs attached].

Please confirm receipt and provide a timeline for erasure. The statutory deadline is 30 days from the date of this letter.

Regards,
Dr. Klaus Brandt

---

Fatima Al-Rashid calls you within an hour of receiving the letter.

**Fatima**: "We have a problem. Tell me we have a process for this."

**You**: "We have a process for user account deletion. That removes the profile, the login, the contact information."

**Fatima**: "And the event logs?"

Pause.

**You**: "The event logs are append-only. That was the design decision from Ch. 126 — immutable, hash-chained, legally defensible."

**Fatima**: "I know. I helped design that requirement. So here's the contradiction: GDPR says we must erase Heinrich Vogel's personal data within 30 days. NSW construction regulations say we must retain project records — including the event log — for 7 years. Some of those projects are in New South Wales. The event logs are the same data."

Long silence.

**You**: "What projects are in NSW?"

**Fatima**: "Twelve of the forty-seven."

**You**: "And Heinrich's personal data is in those event logs as..."

**Fatima**: "Submitter name on 340 approval events. Signatory on 12 permit forms. His professional certification number appears in 89 events — required by NSW law to be recorded."

You pull up the event log schema. Every event has a `submitted_by` field — a JSON object containing `user_id`, `display_name`, `email`, and `certification_number`. It was denormalized into the event payload at creation time for legal defensibility.

**You** (quietly): "The personal data is inside the events. The events are the legal record."

**Fatima**: "Yes. That's the problem. I need you to tell me there's a solution that satisfies both. If there isn't, I need to know that too, because we're going to have to negotiate with the NSW regulator."

---

**#arch-platform** *(Slack — the same afternoon)*

**You**: "Liam — I need to understand what 'deleting' a user currently does in the system."

**Liam Kowalski**: "Marks `users.deleted_at` as current timestamp. Sets `users.email` to a hash. Removes profile data. Login stops working. That's it."

**You**: "And the event log references?"

**Liam Kowalski**: "Not touched. The display_name and email in the event payload are still there verbatim."

**You**: "Kafka topics?"

**Liam Kowalski**: "We have `project.events` topic, retention: 30 days. After 30 days they're in S3 (Parquet, partitioned by project + month). S3 lifecycle: no deletion policy."

**You**: "BI warehouse?"

**Liam Kowalski**: "Redshift. Contains denormalized event exports. Heinrich Vogel's name appears in at least 4 Redshift tables."

**You**: "Backup?"

**Liam Kowalski**: "Daily PostgreSQL snapshots. 90-day retention. His data is in all of them."

---

You spend three days researching. You find a technique called *cryptographic erasure*: instead of deleting the data itself, you encrypt it with a per-person key and then destroy the key. The encrypted data remains in the event log — the bytes are still there — but they are unrecoverable without the key. For GDPR purposes, encrypted-and-key-destroyed is considered "erased."

But then you find the complexity: this only works if the data was encrypted with a per-person key *before* it was written to the log. Heinrich Vogel's data was written in plaintext from 2021 to 2025. There is no key to destroy.

You write a two-page memo and send it to Fatima.

**Your memo, key paragraph**: *"For historical data (pre-encryption design), we have three options: (1) full field redaction — overwrite PII fields with '[REDACTED]' in all storage systems, breaking hash-chain integrity; (2) negotiated retention notice — demonstrate to the GDPR regulator that regulatory retention requirements in NSW override Article 17 for the specific events involving certification numbers; (3) pseudonymization — replace real PII with a stable pseudonym that cannot be reverse-mapped, preserving event semantics while removing identifiability. None of these are clean. They each have a different legal exposure profile."*

**Fatima's reply**: "Which one do you recommend?"

**You**: "A combination. And we need a lawyer in both Germany and New South Wales on the phone this week."

---

**Marcus Webb [DM]**

**Marcus Webb**: "NexaCare's GDPR deletion was straightforward — mutable database, delete the rows. This is different. A few things to think through. What's the difference between 'erased' and 'unreadable'? Are they legally equivalent? And: if you break the hash chain to redact a field, what does that do to the legal admissibility of the events around it? The events that SHOULD be immutable."

---

### Problem Statement

BuildRight's event-sourced project log (designed in Ch. 126 to be immutable and hash-chained for legal defensibility) now contains five years of personal data belonging to a German subcontractor who has invoked GDPR Art. 17 erasure. The data is embedded inside the event payloads in plaintext and appears across the Kafka topic, S3 Parquet archive, PostgreSQL snapshots, and Redshift warehouse. Twelve of the forty-seven affected projects are in NSW, Australia, where construction records must be retained for 7 years under local regulation.

The platform must build a GDPR erasure pipeline that satisfies European erasure requirements without destroying the regulatory retention obligations in other jurisdictions — while also implementing a cryptographic erasure design for future data that prevents this conflict from recurring.

---

### Explicit Requirements

1. GDPR erasure pipeline: given a `user_id`, identify all event log entries, Kafka messages, S3 files, and Redshift records containing that user's PII.
2. PII classification map: maintain a data catalog mapping which fields in which schemas contain PII, updated when schemas change.
3. Cryptographic erasure for future data: all new event payloads must encrypt PII fields with a per-person AES-256 key. Key stored in a dedicated key vault (AWS KMS per-person CMK). Erasure = key deletion.
4. Retroactive handling for historical data: implement pseudonymization for historical plaintext PII where regulatory retention prevents full erasure. Pseudonymization mapping must itself be erasable.
5. Hash chain integrity handling: when redacting historical event fields, the redaction must be documented in the event log (a `redaction_event` appended) without altering existing event hashes. The auditor can verify: the event existed, the PII was redacted on a specific date, by a specific request.
6. S3 Parquet remediation: for events in S3, rewrite affected Parquet files with PII fields replaced. Maintain file-level audit log of which files were rewritten and when.
7. Redshift remediation: update all denormalized Redshift tables containing the user's PII.
8. Regulatory retention carve-out: for NSW-regulated projects, emit a legal hold flag on events containing certification numbers. GDPR erasure pipeline must acknowledge the hold and produce a "partial erasure" report for the data controller explaining which fields could not be erased under NSW law 9.
9. Erasure confirmation: within 30 days, produce a signed GDPR Art. 17 compliance report: which data was erased, which data was pseudonymized, which data was retained under regulatory override (with citation).
10. Forward design: new user accounts must have an encryption key provisioned at registration. PII fields in event payloads encrypted by default.

---

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's DM. He asks if "erased" and "unreadable" are legally equivalent. The GDPR Working Party guidance (now EDPB) says encrypted data where the key is destroyed can be considered personal data no longer accessible — but only if the encryption was strong and the key management is robust. For data encrypted with a shared key (e.g., a per-table key), destroying the key erases everyone's data, not one person's. What is the minimum granularity for key management: per-user? Per-user-per-project? Per-event?
- **Hint**: Re-read Fatima's statement about Heinrich Vogel's certification number appearing in 89 events, "required by NSW law to be recorded." The certification number is a professional identifier — it may be PII under GDPR but it is also a legally mandated field under NSW construction law. If you pseudonymize it, does the pseudonym satisfy the NSW requirement to record a valid certification number? What is the legal analysis here?
- **Hint**: Re-read the system inventory Liam provides: Kafka (30-day retention), S3 (Parquet, no deletion policy), PostgreSQL snapshots (90-day), Redshift (indefinite). The 90-day PostgreSQL snapshots contain Heinrich's data even after erasure from the live database. Restoring a backup during a disaster recovery scenario would reintroduce the "erased" data. What is your backup erasure strategy?
- **Hint**: Re-read your memo's three options. Option 2 — "negotiated retention notice" — requires demonstrating a conflict between GDPR Art. 17 and a third-country regulatory requirement. GDPR Recital 65 acknowledges retention exceptions for legal obligations. But NSW law is not an EU legal obligation. Which party (the data controller) is responsible for demonstrating this exception, and what documentation must accompany the partial erasure report?

---

### Constraints

- **Affected user**: Heinrich Vogel, 47 projects, 5 years of event history.
- **Event count**: ~340 approval events + 12 permit forms + 89 certification events = ~441 affected events.
- **NSW projects**: 12 of 47. 7-year retention requirement.
- **GDPR deadline**: 30 days from March 5, 2026 = April 4, 2026.
- **Storage systems to remediate**: PostgreSQL (live), PostgreSQL snapshots (90 days, ~90 files), Kafka (`project.events`, 30-day retention), S3 (Parquet archive, 2021–2025), Redshift (4 tables).
- **Key management**: AWS KMS. Per-user CMK provisioning for new design.
- **Encryption**: AES-256-GCM for PII fields in event payloads.
- **Pseudonymization**: HMAC-SHA256 with a per-tenant secret. Deterministic (same input → same pseudonym) so relational integrity is preserved.
- **Team**: you + Fatima (legal) + 1 engineer. Four weeks.
- **Legal counsel**: German GDPR attorney + NSW construction law specialist engaged.

---

### Your Task

Design the GDPR erasure pipeline for BuildRight's event-sourced system. Address both the retroactive erasure of Heinrich Vogel's historical data and the forward-looking cryptographic erasure architecture for all new event data. Explicitly address the GDPR vs NSW regulatory conflict, the hash chain integrity problem, the backup erasure strategy, and the signed compliance report format.

---

### Deliverables

- [ ] **Mermaid architecture diagram** — show: erasure request intake → PII scanner (all storage systems) → erasure orchestrator → per-system erasure workers (PostgreSQL, S3 rewriter, Redshift, Kafka); key vault (KMS); compliance report generator
- [ ] **Cryptographic erasure key design** — define key granularity (per-user vs per-user-per-project), key provisioning flow at user registration, key deletion flow at erasure request, key rotation policy
- [ ] **Redaction event schema** — define the `redaction_event` record appended to the event log when historical PII is redacted: include `original_event_id`, `redacted_fields`, `redaction_reason` (GDPR Art. 17 / regulatory hold), `request_id`, `executed_at`, `executed_by`
- [ ] **Scaling estimation** — calculate: S3 Parquet rewrite cost for 5 years of event data for one user (estimate files affected, rewrite time, cost); KMS key provisioning cost for 500,000 users at $0.03/key/month
- [ ] **Tradeoff analysis** — minimum 3: (1) cryptographic erasure vs field redaction for historical data (legal equivalence vs implementation cost), (2) pseudonymization vs full erasure under regulatory hold (data utility vs GDPR compliance risk), (3) per-user CMK vs per-user-per-project CMK (cost vs granularity)
- [ ] **Regulatory conflict resolution memo** — two paragraphs: the legal basis for partial erasure under GDPR Recital 65 when NSW retention law applies; what the compliance report must say to satisfy both regulators
- [ ] **Backup erasure strategy** — describe the approach for the 90-day PostgreSQL snapshot window: can you delete individual records from a snapshot without restoring it? What is the practical approach and its limitations?
