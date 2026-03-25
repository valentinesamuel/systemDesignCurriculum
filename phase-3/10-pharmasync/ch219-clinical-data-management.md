---
**Title**: PharmaSync — Chapter 219: The Six-Week Submission
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #compliance #data-pipeline #distributed #database #api-design
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 216, Ch. 217, Ch. 218, Ch. 221
**Exercise Type**: System Design
---

### Story Context

**From:** Dr. Priya Chandrasekaran, VP Medical Affairs
**To:** Engineering Leadership, Regulatory Affairs, Clinical Data Management
**CC:** Olusegun Adeyemi (CEO), Sandra Kowalczyk (Head of Regulatory)
**Subject:** URGENT — NDA Submission: 6-Week Clock Is Running
**Date:** Tuesday, 06:47

Team,

The FDA accepted our pre-NDA meeting request. We submit our New Drug Application for PR-7714 in 42 days. This is not a soft deadline. The submission window was 18 months in the making.

I need clinical data in CDISC SDTM v3.3 format, validated, locked, and ready to package by Day 38. That gives EDC and engineering exactly 38 days from this morning.

Current status per site:

| Site | Country | Format | CDISC Version | Status |
|------|---------|--------|---------------|--------|
| Site 01 — Boston General | USA | REDCap export | SDTM v3.3 | Clean |
| Site 02 — Toronto Western | Canada | REDCap export | SDTM v3.3 | Clean |
| Site 03 — Charité Berlin | Germany | Oracle Clinical | SDTM v3.1.2 | **DEPRECATED** |
| Site 04 — Paris Lariboisière | France | OpenClinica | SDTM v3.2 | **DEPRECATED** |
| Site 05 — São Paulo USP | Brazil | Medidata Rave | SDTM v3.3 | Clean |
| Site 06 — Mumbai AIIMS | India | Local CSV export | SDTM v3.1.3 | **DEPRECATED** |
| Site 07 — Sydney RPA | Australia | REDCap export | SDTM v3.3 | Clean |
| Site 08 — Cape Town Groote Schuur | South Africa | OpenClinica | SDTM v3.3 | Clean |
| Site 09 — Seoul Samsung Medical | South Korea | Medidata Rave | SDTM v3.3 | Clean |
| Site 10 — Tokyo NCNP | Japan | Local system | SDTM v3.2.1 | **DEPRECATED** |
| Site 11 — Mexico City INNN | Mexico | REDCap export | SDTM v3.3 | Clean |
| Site 12 — Cairo Kasr Al Ainy | Egypt | OpenClinica | SDTM v3.3 | Clean |
| Site 13 — Warsaw WUM | Poland | Local CSV export | SDTM v3.2 | **DEPRECATED** |
| Site 14 — Stockholm Karolinska | Sweden | REDCap export | SDTM v3.3 | Clean |
| Site 15 — Singapore SGH | Singapore | Medidata Rave | SDTM v3.3 | Clean |

That's 5 sites on deprecated CDISC versions. Sandra — I need a remediation plan in 48 hours.

One more thing. Site 03 (Charité Berlin) flagged something with their DPO last week that I didn't escalate promptly enough — I apologize for the delay. Their DPO is concerned about patient identifiers in the DM domain export. I'll let Sandra and engineering sort out the details, but this needs to move fast. I'll forward the email thread separately.

We do not miss this submission window.

— Priya

---

**From:** Sandra Kowalczyk, Head of Regulatory Affairs
**To:** [You], Tobi Nwachukwu (Head of Clinical Data Management)
**CC:** Dr. Priya Chandrasekaran
**Subject:** RE: URGENT — NDA Submission: 6-Week Clock Is Running
**Date:** Tuesday, 09:14

I'll cut to it. The deprecated CDISC versions are annoying but solvable. Mappings exist for v3.1.2 → v3.3 and v3.2/3.2.1 → v3.3 but they're not clean — there are deprecated variables that must be handled and some new mandatory variables in 3.3 that simply don't exist in the older exports. For each deprecated site we need:

1. A transformation specification document
2. The transformation code with unit tests
3. A validation report comparing pre/post-transform record counts and variable distributions
4. An audit trail showing every transformation applied per record

The audit trail is not optional. The FDA reviewer will ask to see it.

Regarding Charité Berlin: I read their DPO's email. The problem is real. Their patient identifiers in the SUBJID field contain a combination of site code + date-of-birth hash. The DPO considers this pseudonymized personal data under GDPR. The FDA requires SUBJID in the submission. There is no carve-out.

We cannot de-identify SUBJID further — we need it for adverse event follow-up. But we cannot submit GDPR-protected data to a US regulatory agency without the patient's GDPR-compliant consent covering FDA submission, which the Charité consent form does not explicitly include.

I'm reaching out to our GDPR counsel. But engineering needs to design a system where: the FDA submission uses a new pseudonymized ID (let's call it FDA_SUBJID), the mapping from SUBJID → FDA_SUBJID is stored in a system that is: (a) not exported to FDA, (b) accessible for adverse event follow-up, and (c) auditable to demonstrate the mapping was consistent.

We have 38 days. Start now.

— Sandra

---

**#clinical-data-eng | Slack**

**tobi.nwachukwu [09:45]:** Sandra's email is in your inbox. Berlin is the hard one. The rest are mapping problems we've solved before.

**[you] [09:52]:** Reading it now. The FDA_SUBJID thing is a pseudonymization layer on top of pseudonymization. We need a key management story.

**tobi.nwachukwu [09:54]:** And the mapping has to survive the submission process. If a patient has an adverse event 2 years from now, we need to trace FDA_SUBJID back to the original patient record at Charité.

**[you] [09:57]:** Okay. So the mapping table cannot be in the same database as the export pipeline. It needs access controls. And it needs to outlive the submission by... how long?

**tobi.nwachukwu [09:58]:** FDA adverse event follow-up obligations are 10 years post-approval. Minimum.

**[you] [10:01]:** This is key management with a 10+ year retention horizon. That's not a database problem, it's a secrets problem.

**tobi.nwachukwu [10:03]:** There's another thing I haven't told Priya yet. The transformation pipeline for v3.1.2 (Charité is on that) has a known issue with the EPOCH variable. It derives EPOCH from RFSTDTC (reference start date) but when RFSTDTC is missing for screen-fail patients, the derivation fails silently. We have 47 screen-fail patients at Charité. I need to know what happens to those records in your new pipeline.

**[you] [10:06]:** Silent failure in a clinical data pipeline is not acceptable. We need validation gates that fail loudly and stop the pipeline.

**tobi.nwachukwu [10:07]:** Correct. Design it that way.

---

### Problem Statement

PharmaSync is 38 days from FDA submission of a New Drug Application for PR-7714. Clinical data from 15 trial sites across 7 countries must be transformed into a single unified CDISC SDTM v3.3 dataset and submitted via the FDA Electronic Submission Gateway. Five sites are on deprecated CDISC versions requiring non-trivial transformation. One site (Charité Berlin, Germany) has a GDPR conflict: patient identifiers in the SUBJID field contain pseudonymized personal data that cannot be directly exported to a US regulatory agency, but must be traceable for future adverse event follow-up.

You must design the CDISC data management platform: ingestion from 15 heterogeneous sources, transformation with audit trails, the GDPR-safe pseudonymization layer, validation gates, data lock, and FDA submission packaging.

### Explicit Requirements

1. Ingest clinical data from 15 sites in 6 different source formats (REDCap, Oracle Clinical, OpenClinica, Medidata Rave, local CSV, local proprietary system)
2. Transform 5 deprecated CDISC version exports (v3.1.2, v3.2, v3.2.1) to SDTM v3.3 with documented transformation specifications
3. Generate an audit trail for every transformation at the record level (before/after values, transformation rule applied, timestamp, operator)
4. Implement a GDPR-safe pseudonymization layer for Site 03 (Charité Berlin): FDA submission uses FDA_SUBJID, mapping to original SUBJID is stored separately and access-controlled
5. Validation gates must fail loudly and stop the pipeline on: missing mandatory SDTM v3.3 variables, record count mismatches between source and transformed output, silent transformation failures (e.g., the EPOCH/RFSTDTC issue)
6. Data lock: once triggered, no record modifications permitted — only audit trail appends
7. Submission package must conform to FDA ESG technical specifications (eCTD format, DEFINE-XML v2.1)
8. Transformation specifications must be human-readable documents that a regulatory reviewer can audit

### Hidden Requirements

- **Hint: re-read the Tobi Nwachukwu Slack thread.** Tobi mentions 47 screen-fail patients at Charité with missing RFSTDTC. What happens to those records if the EPOCH derivation fails silently? The system must handle screen-fail patients as a distinct record class with explicit null-handling rules, not let them fall through the transformation pipeline undetected.

- **Hint: re-read Sandra's email about FDA_SUBJID.** She says the mapping must "survive the submission process" and adverse event follow-up is 10 years post-approval. The FDA_SUBJID mapping is not just a database table — it is a regulated record with its own retention, access control, and audit requirements. It must be stored in a system that can be demonstrated to auditors as tamper-evident and access-logged.

- **Hint: re-read Dr. Priya's site status table.** Sites 04 (France), 10 (Japan), and 13 (Poland) are also on deprecated versions. France has GDPR obligations. Poland has GDPR obligations. Japan has APPI (Act on Protection of Personal Information) obligations. The GDPR problem at Charité may not be unique — you need to audit the patient identifier fields at all non-US sites.

- **Hint: re-read Sandra's message about the transformation specification document.** She says "the FDA reviewer will ask to see it." This means the transformation specification is itself a regulatory document — it must be version-controlled, signed, and linked to the transformation code that implements it. The specification and the code must be demonstrably in sync.

### Constraints

- **Sites**: 15 trial sites, 7 countries
- **Patients**: 5,000 enrolled, ~12% screen-fails (600 patients), 47 screen-fails at Charité specifically
- **Data volume**: ~200K clinical observations across all domains (DM, AE, CM, EX, LB, VS, etc.)
- **Deadline**: 38 days to data lock; 42 days to FDA submission
- **CDISC standard**: SDTM v3.3 (target), SDTM v3.1.2/3.2/3.2.1 (source for 5 sites)
- **Compliance**: 21 CFR Part 11 (audit trail, electronic signatures), GDPR (Site 03, 04, 13), APPI (Site 10)
- **Retention**: Submission records and FDA_SUBJID mapping retained minimum 10 years post-approval
- **FDA ESG**: eCTD format, DEFINE-XML v2.1, file size limits per domain (max 500MB per dataset file)
- **Team**: 4 clinical data managers, 3 engineers, 2 biostatisticians
- **Zero downtime requirement**: Pipeline must support iterative runs (not a one-shot batch)

### Your Task

Design the end-to-end CDISC clinical data management platform for PharmaSync's NDA submission. Your design must address: multi-source ingestion, deprecated version transformation with audit trails, the GDPR pseudonymization architecture for Site 03 and equivalent sites, validation gate design, data lock mechanics, and FDA submission packaging.

### Deliverables

- [ ] Mermaid architecture diagram showing the full pipeline: source connectors → staging → transformation → validation → pseudonymization layer → submission assembly
- [ ] Database schema for: `clinical_records` (staging), `transformation_audit_log`, `fda_subjid_mapping` (separate schema/database), `validation_failures`, `submission_manifest`
- [ ] Transformation specification document template (the human-readable format Sandra requires) — show the structure with example entries for the EPOCH/RFSTDTC null-handling rule
- [ ] Pseudonymization architecture: how FDA_SUBJID is generated (deterministic vs random), where the mapping is stored, access control model, audit log for mapping access, retention policy
- [ ] Validation gate design: list all gates, what triggers a pipeline halt vs a warning, how screen-fail patients (missing RFSTDTC) are handled explicitly
- [ ] Scaling estimation:
  - 200K observations × average 50 bytes per observation = 10MB raw clinical data
  - Transformation audit log: 200K records × 3 transformations average × 200 bytes = 120MB
  - Show total storage estimate for the submission package including DEFINE-XML
- [ ] Tradeoff analysis (minimum 3):
  - Deterministic vs random FDA_SUBJID generation (auditability vs unguessability)
  - Per-record audit trail vs per-batch audit trail (granularity vs storage)
  - Fail-fast validation vs best-effort with flagging (speed vs completeness of output)
- [ ] Cost modeling: estimate pipeline infrastructure cost for the 38-day submission sprint ($X/month, then how to scale down post-submission)
- [ ] Data lock implementation: how does the system enforce immutability post-lock while still allowing audit trail appends?

### Diagram Format

Mermaid syntax. Show: SourceConnector nodes for each format type → StagingDatabase → TransformationEngine (with AuditLogger sidecar) → ValidationGate → two paths: (a) non-Berlin sites direct to SubmissionAssembly, (b) Berlin sites through PseudonymizationService → SubmissionAssembly → FDASubmissionGateway. Show FDASubjIDMapping as a separate isolated store with AccessControlLayer.
