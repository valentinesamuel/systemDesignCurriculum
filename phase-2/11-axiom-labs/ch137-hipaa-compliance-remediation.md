---
**Title**: Axiom Labs — Chapter 137: Genomic PII in the Application Logs
**Level**: Staff
**Difficulty**: 9
**Tags**: #hipaa #compliance #pii #log-remediation #breach-notification #spike-inherited-disaster #genomics
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 72 (NexaCare audit trails), Ch. 75 (NexaCare secret management), Ch. 131 (BuildRight GDPR)
**Exercise Type**: Spike — Inherited Disaster
---

### Story Context

**Monday 07:30**

You're at your desk. The building is mostly empty. Marcus Thompson, Head of Security & Compliance, appears at your doorway. He looks like he didn't sleep. He is carrying a laptop and a printed spreadsheet.

**Marcus Thompson**: Can you come to my office? Now.

His office. He closes the door. Sets the laptop in front of you.

**Marcus Thompson**: I ran a log analysis tool over the weekend. I wrote it myself — we don't have a commercial tool that does what I needed. I was looking for PII leakage in our application logs. I found it.

The screen shows rows of log entries. You scan them. Application INFO logs. Debug output from a backend service. But embedded in the log lines: strings that look like `patient_id=PAT-7723849-MAN`, `cohort=AF-IMMUNE-2022`, `variant_flag=BRCA2_pathogenic`.

**[you]**: When did this start?

**Marcus Thompson**: The logs go back 140 days. The leakage starts at day 1 of this log retention window. I don't know when it started before that — logs before 140 days are purged.

**[you]**: Who wrote this logging code?

**Marcus Thompson**: A junior engineer named Thierry Dubois who left 3 months ago. He added verbose debug logging to trace a performance issue and never removed it. The logs contain partial patient identifiers — not full names, but combinations of institution code, cohort identifier, and genetic variant flags that, in combination, could re-identify specific patients.

**[you]**: How many records?

**Marcus Thompson**: I've counted approximately 4,200 distinct patient identifier strings across 140 days of logs.

HIPAA requires covered entities to notify the HHS Office for Civil Rights of breaches affecting 500 or more individuals within 60 days of discovery. You have just discovered a breach.

**[you]**: Have you told Dr. Sharma?

**Marcus Thompson**: Not yet. I wanted to understand the full scope before I did. I have one more number to show you.

He flips to the second page of the spreadsheet.

**Marcus Thompson**: The logging system sends logs to our centralized Grafana Loki instance. Loki is accessible to every engineer at Axiom — approximately 140 people. I pulled the access logs. 23 engineers have queried logs from the service in question in the last 140 days.

**[you]**: So potentially 23 people have seen these logs.

**Marcus Thompson**: Yes. I don't know if any of them noticed the patient identifiers or not.

You take a breath.

**[you]**: We need to tell Dr. Sharma. Right now. And we need to call our HIPAA counsel by 9 AM.

---

**09:00 — Dr. Priya Sharma's office**
**Present**: Dr. Sharma, Marcus Thompson, [you], Kavita Rao (General Counsel, joining by video)

**Kavita Rao**: The 60-day clock started this morning at 07:30. We have until March 25th to file with OCR. The notification must include: scope of the breach, description of the PHI affected, how the breach occurred, steps taken to mitigate, and what we're doing to prevent recurrence.

**Dr. Priya Sharma**: *(very quietly)* 4,200 patients.

**Kavita Rao**: The HIPAA definition of a breach is "unauthorized access to or disclosure of PHI." 140 engineers having access to logs containing PHI qualifies. Severity is somewhat mitigated because the identifiers are partial — not full patient records. But we cannot avoid notification.

**[you]**: Three things need to happen simultaneously. One: secure the current logs — revoke broad access, move to restricted access for security team only. Two: identify and scrub all logs containing PHI — not just in Loki, but in any backups, S3 archives, or monitoring dashboards that may have cached log snippets. Three: build the technical remediation so this can't happen again.

**Dr. Priya Sharma**: How long for three?

**[you]**: Secure access: today. Scrub existing logs: 5 days. Technical remediation (automated PII detection in log pipeline): 3 weeks.

**Dr. Priya Sharma**: Do all three in parallel. Today is day zero.

---

### Problem Statement

Axiom Labs has discovered 140 days of application logs containing partial patient identifiers accessible to 140 engineers, constituting a HIPAA breach affecting approximately 4,200 research subjects. The 60-day OCR notification clock has started. Three parallel workstreams are required: immediate access restriction, retroactive log scrubbing, and a permanent technical remediation.

---

### Explicit Requirements

1. OCR breach notification filed within 60 days of discovery (March 25th deadline)
2. All logs containing PHI must be identified and rendered inaccessible within 5 days
3. Log scrubbing must extend to all storage tiers: Loki, S3 log archives, monitoring dashboards
4. Technical remediation: automated PII detection in the log pipeline before logs reach storage
5. Access audit: determine which of the 23 engineers who accessed the logs viewed PHI-containing entries
6. Prevention: logging guidelines and automated enforcement so future engineers cannot log PHI

---

### Hidden Requirements

- **Hint**: Marcus Thompson mentioned "logs before 140 days are purged." But HIPAA audit logs must be retained for 6 years. If you purge application logs after 140 days, you're also purging evidence of your own access audit trail. Your log retention policy redesign must distinguish between application debug logs (short retention, strict PII filtering) and audit logs (long retention, immutable). These are different things and were previously mixed.

- **Hint**: The breach notification to OCR is required — but OCR also evaluates the quality of the breach response when deciding on penalties. A company that discovers a breach, acts immediately, cooperates with OCR, and implements strong technical controls typically receives a corrective action plan rather than a fine. Your incident response plan must be designed to demonstrate responsiveness.

---

### Constraints

- 60-day OCR notification deadline (hard wall)
- 140 days of affected logs × average log volume
- 23 engineers who accessed affected logs (need access audit)
- Loki log retention: currently 140 days (must redesign)
- S3 cold log archive: 18 months of logs stored in Glacier (need to identify and scrub PHI entries)
- Engineering team: 140 engineers who can currently access all logs

---

### Your Task

Design the breach response and technical remediation for Axiom Labs' HIPAA log exposure incident.

---

### Deliverables

- [ ] Day 1 actions: specific steps to secure logs and revoke broad access within hours of discovery
- [ ] PHI detection methodology: how to scan 140 days of logs to identify all PII-containing entries
- [ ] Log scrubbing pipeline: approach for retroactive PII removal from Loki, S3 archives, dashboards
- [ ] Access audit design: how to determine whether the 23 engineers who accessed logs viewed PHI entries
- [ ] OCR notification package: what must be included in the HHS submission (use HIPAA breach notification template)
- [ ] Log pipeline redesign: automated PII detection and redaction before log storage (regex + ML-based scanner)
- [ ] Log retention policy redesign: application debug logs (short, strict PII filter) vs audit logs (6-year immutable) separation
- [ ] Prevention controls: structural changes that make PHI logging structurally impossible for future engineers
- [ ] Tradeoff analysis (minimum 3):
  - Regex-based PII detection vs ML-based detection in the log pipeline — false positive rate vs coverage
  - Delete-in-place scrubbing vs cryptographic erasure for log archives — HIPAA compliance equivalence
  - Immediate broad log access revocation vs phased access review — security speed vs operational disruption
