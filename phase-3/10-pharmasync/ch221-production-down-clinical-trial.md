---
**Title**: PharmaSync — Chapter 221: The Two Phantom Patients
**Level**: Staff
**Difficulty**: 9
**Tags**: #incident-response #data-pipeline #compliance #distributed #database
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 219, Ch. 222
**Exercise Type**: Incident Response
---

### Story Context

**#incidents | Slack — Thursday 02:04**

```
🚨 INCIDENT DECLARED — SEV-1
Incident Commander: [you]
Bridge: https://meet.pharmasync.io/incidents/INC-2024-11-14

Summary: Clinical data discrepancy in PR-7714 NDA dataset.
DM domain export shows 849 patient records. Source EDC shows 847.
2 extra records of unknown origin.
FDA submission T-18 days.
```

---

**Incident Bridge Log — INC-2024-11-14**

**[02:04] [you]:** I'm declaring SEV-1. Who's on the bridge? Sound off.

**[02:05] tobi.nwachukwu:** On. I'm the one who found it — running the pre-submission QC check. We do a record count reconciliation every night. Tonight: EDC = 847, SDTM export = 849. I've never seen this before.

**[02:05] fatima.alrashid (Clinical Data Manager):** On. I reviewed the DM domain manually. The two extra records have valid-looking SUBJID values — PR7714-003-0241 and PR7714-003-0247. But these SUBJIDs do not exist in the EDC system.

**[02:06] kofi.adebiyi (Data Engineer):** On. Just pulled up the ETL logs. Last run completed at 23:47. No errors reported.

**[02:07] dr.priya.chandrasekaran:** On. I need to say this clearly for the log: if there are patient records in our submission dataset that do not correspond to real patients, this is a data integrity incident. At minimum this triggers a clinical hold review. At worst — if these records contain fabricated clinical data — this is research fraud.

**[02:08] [you]:** Understood. First hypothesis: ETL bug. Second hypothesis: data breach (external insertion). Third: data breach (internal). We work hypothesis one first because it's most likely. Kofi, can you pull the full ETL log for last night's run?

**[02:09] kofi.adebiyi:** Pulling now.

**[02:11] fatima.alrashid:** I'm looking at the two phantom records more closely. PR7714-003-0241 has: SUBJID = PR7714-003-0241, SITEID = 003, SEX = M, AGE = 47, COUNTRY = DEU, RFSTDTC = 2023-04-12. Site 003 is Charité Berlin.

**[02:12] tobi.nwachukwu:** Both phantom records are from Site 003. Berlin.

**[02:12] dr.priya.chandrasekaran:** Berlin again. They've had data quality issues before. Tobi, what do we know about their last data transmission?

**[02:13] tobi.nwachukwu:** Site 03 sent their final data package on Tuesday. We confirmed receipt. The package had exactly 78 patient records — matching their enrollment numbers.

**[02:15] kofi.adebiyi:** I have the full ETL log. Starting at the Berlin data connector. Input: 78 records. Output stage 1 (raw ingest): 78 records. Output stage 2 (SDTM v3.1.2 normalization): 78 records. Output stage 3 (v3.1.2 → v3.3 transformation): 80 records. Source of the extras is in stage 3.

**[02:16] [you]:** The transformation step is generating 2 extra records from 78 inputs. That's the bug. Kofi, can you diff the input and output of stage 3 for the Berlin dataset?

**[02:19] kofi.adebiyi:** Running diff now. Looking at the 80-record output vs the 78-record input. The two extra records are... hm. They're duplicates of records 0241 and 0247 from the input. But not exact duplicates — some fields are different.

**[02:21] kofi.adebiyi:** I see it. The SUBJID field in the source data for records 0241 and 0247 contains special characters. 0241 has "Müller" in the INVNAM field (investigator name). 0247 has "Rodríguez" in the RACEOTH field. The transformation script is generating a secondary record for any record containing non-ASCII characters. It's a character encoding bug.

**[02:22] fatima.alrashid:** So these aren't phantom patients. They're duplicates of real patients with names containing umlauts and accents.

**[02:22] tobi.nwachukwu:** The transformation script must be splitting the record at the encoding boundary, treating the multi-byte character as a record delimiter.

**[02:23] dr.priya.chandrasekaran:** [exhales audibly on bridge] Okay. That's... that's a bug. Not a breach. I need confirmation before I stand down.

**[02:24] [you]:** Kofi, can you manually run the transformation on just those two records with proper UTF-8 handling and confirm the output is 2 records, not 4?

**[02:27] kofi.adebiyi:** Done. With UTF-8 force-set in the locale: input 2 records, output 2 records. Confirmed encoding bug.

**[02:28] dr.priya.chandrasekaran:** I'm standing down on the breach hypothesis. This is an engineering defect. But Tobi — what other records in the full dataset might have been affected? If Berlin had 2 affected records, what about the other 14 sites?

**[02:29] tobi.nwachukwu:** Scanning now for non-ASCII characters across all 15 sites. Sites 04 (Paris), 06 (Mumbai), 10 (Tokyo), 13 (Warsaw) — all submitted data. French accents, Devanagari transliterations, Japanese romaji transliterations, Polish diacritics.

**[02:31] tobi.nwachukwu:** Full scan complete. 23 records with non-ASCII characters across 8 sites. The transformation bug could have affected all of them. I need to re-run the transformation for all affected records.

**[02:32] fatima.alrashid:** And then I need to manually verify the re-run output. 23 records. We're 18 days from submission.

**[02:33] [you]:** This is manageable but we need to do this properly. We need to: fix the bug, re-run the affected records, verify the output, update the audit trail, and most importantly — understand why our validation gates didn't catch this. The record count check caught it. But we should have caught it before the full export even ran.

**[02:35] dr.priya.chandrasekaran:** And I need a full written incident report by 08:00 for the CEO. If the FDA asks — and they may ask — I need to be able to say we detected this through our own QC process, investigated it, identified a root cause that was not a data integrity issue, and remediated it with an audit trail.

**[02:36] [you]:** Understood. Bridge stays open. Let's work.

---

**[06:47] — Bridge Update**

**kofi.adebiyi:** Bug fixed. UTF-8 locale explicitly set in the transformation runtime. All 23 affected records re-processed. Output verified by Fatima: all 23 produce clean output, no duplicates.

**fatima.alrashid:** I've verified all 23 manually. DM domain is now 847 records — matching EDC.

**tobi.nwachukwu:** QC reconciliation passing. 847 in, 847 out, across all 15 sites.

**[you]:** Good. I'm declaring incident resolved. Fatima — I need the audit trail for those 23 records showing the original run timestamp, the error detection timestamp, the bug fix timestamp, the re-run timestamp, and your manual verification signature. That goes into the submission package.

**fatima.alrashid:** Will have it by 10:00.

**[you]:** Closing the bridge. Thanks everyone. Incident report due 08:00. I'm writing it now.

---

### Problem Statement

A character encoding bug in PharmaSync's CDISC transformation pipeline has been creating duplicate records for patients whose data fields contain non-ASCII characters (umlauts, accents, diacritics). The bug was detected by a nightly QC record count reconciliation 18 days before FDA submission. The 2-record discrepancy was initially investigated as a potential data breach (phantom patients) before being identified as an ETL defect. You must design a data integrity architecture that: (a) prevents this class of bug from reaching the production export, (b) detects it faster when it does occur, (c) produces a forensically credible audit trail for the FDA, and (d) handles the specific complexity of FDA-regulated clinical data — where a "re-run" is not simply a technical operation but a regulated event that requires documentation.

### Explicit Requirements

1. Input validation gate: detect non-ASCII characters in all clinical data fields before transformation begins; log affected records to a dedicated `non_ascii_flag` table
2. Character encoding normalization: enforce UTF-8 throughout the transformation pipeline; explicit locale setting at each stage transition
3. Post-transformation reconciliation: after every transformation stage, compare input record count to output record count; any discrepancy halts the pipeline immediately
4. Duplicate detection: post-transformation, scan for records where SUBJID appears more than once per site; flag as `DUPLICATE_SUSPECTED`
5. Transformation re-run tracking: when records are re-processed after a bug fix, the audit trail must record: original run (with error), bug fix event, re-run (with operator signature), and manual verification event
6. Incident audit package: produce a structured audit document for FDA submission explaining the detection, investigation, root cause, and remediation of the discrepancy
7. Automated pre-submission QC suite: expand the nightly reconciliation to include all the checks that would have caught this bug earlier

### Hidden Requirements

- **Hint: re-read the bridge log at 02:28.** Dr. Priya asks "what other records in the full dataset might have been affected?" The encoding bug was detected in the Berlin data because Berlin was the first site processed. But the full scan found 23 affected records across 8 sites. This means the bug had been silently running for multiple processing cycles — not just last night. The audit trail needs to cover every historical run of the transformation pipeline, not just the most recent one.

- **Hint: re-read Tobi's statement at 02:13.** He says "Site 03 sent their final data package on Tuesday. We confirmed receipt. The package had exactly 78 patient records." But he also says the input to stage 3 was 78 records. This means the bug was introduced in the v3.1.2 → v3.3 transformation step specifically. The validation gate design must include per-stage input/output counts — not just a start-to-finish count — because the bug was invisible at the start and end but visible between stages.

- **Hint: re-read Dr. Priya's statement at 02:37 (bridge closing).** She says the incident report must describe the detection as occurring "through our own QC process." This is important for FDA purposes: a company that detects its own data integrity issues and documents them demonstrates a functioning quality system. The design of the QC system — including what checks ran and when — must be documented as part of the system architecture, not just the incident report.

- **Hint: re-read Fatima's 02:11 message.** She examines the phantom records and notes they have "valid-looking SUBJID values." The fact that the phantom records looked legitimate (valid format, plausible demographic data) means automated duplicate detection by SUBJID alone would have caught it — but only if the duplicate detection ran before the merge with other sites' data. Per-site duplicate detection must run before cross-site merge.

### Constraints

- **Dataset**: 5,000 patients across 15 sites; 23 records with non-ASCII characters across 8 sites confirmed affected
- **Timeline**: 18 days to FDA submission; fix and re-verification must complete within 48 hours (FDA notification SLA: 72 hours for potential data integrity events)
- **Regulatory context**: 21 CFR Part 11; every re-processing event is a regulated event requiring an electronic signature
- **Transformation pipeline**: 5 stages (raw ingest, SDTM normalization, version upgrade, domain derivation, QC export); bug was in stage 3
- **Audit trail retention**: minimum 10 years post-approval
- **FDA concern**: the incident report must be included in the submission as a data integrity disclosure if the FDA reviewer requests it (proactive disclosure is the safer regulatory strategy)
- **Historical runs**: need to audit every prior transformation run to determine if the bug affected any earlier QC checkpoints

### Your Task

Design a data integrity architecture for the CDISC transformation pipeline that would have prevented the phantom record incident, detected it earlier if not prevented, and produced the forensic documentation package required for FDA disclosure. Your design should also include a post-incident remediation plan covering historical run audit.

### Deliverables

- [ ] Mermaid architecture diagram showing the 5-stage transformation pipeline with validation gates between each stage, non-ASCII detection as a pre-stage-1 gate, per-stage input/output count reconciliation, and the duplicate detection service running after per-site processing and after cross-site merge
- [ ] Validation gate specification: a table listing every gate in the pipeline, the check performed, the action on failure (halt / warn / flag), and the regulatory rationale
- [ ] Database schema for: `pipeline_run_log` (run_id, stage, input_count, output_count, status), `non_ascii_flag` (record_id, field_name, character, run_id), `duplicate_suspected` (subjid, site_id, run_id, record_ids), `reprocessing_event` (original_run_id, bug_fix_description, new_run_id, operator_signature, verification_signature)
- [ ] Incident audit package template: the structure of the document you would submit to FDA — sections, what each section contains, what constitutes "sufficient evidence" for each section
- [ ] Historical run audit plan: how do you audit every prior pipeline run to determine scope of the encoding bug? What do you look for? What evidence do you preserve?
- [ ] Scaling estimation:
  - 5,000 patients × 20 domains × 10 records/domain average = 1M clinical observation records
  - Validation gate overhead per record: < 1ms per check × 6 checks × 1M records = 6 seconds total gate time
  - Audit log growth: 1M records × 5 stages × 200 bytes = 1GB per full pipeline run
- [ ] Tradeoff analysis (minimum 3):
  - Halt pipeline on first duplicate detected vs process all records and report all anomalies at end (safety vs informativeness)
  - Per-character encoding validation vs character set whitelist (completeness vs performance)
  - Proactive FDA disclosure vs silent remediation (regulatory credibility vs risk of drawing attention)
- [ ] Cost modeling: validation gate infrastructure added to existing pipeline ($X/month incremental); cost of the 48-hour incident remediation sprint in engineering time
- [ ] Capacity planning: what does the validation architecture look like for Phase III trials (5× the patient count, 50 sites)?

### Diagram Format

Mermaid syntax. Pipeline stages as sequential nodes with validation gate nodes between each. Show non-ASCII detection as a red gate before Stage 1. Show per-stage count reconciliation gates between stages 1-2, 2-3, 3-4, 4-5. Show duplicate detection as a gate after Stage 3 (per-site) and after Stage 4 (cross-site merge). Show AuditLogger as a sidecar writing to AuditStore at every stage. Show QCExport as the final stage, with a human verification step before submission packaging.
