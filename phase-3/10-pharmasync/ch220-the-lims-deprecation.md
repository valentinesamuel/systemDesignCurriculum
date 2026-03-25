---
**Title**: PharmaSync — Chapter 220: Gerald Must Die
**Level**: Staff
**Difficulty**: 8
**Tags**: #system-evolution #compliance #database #distributed #api-design
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 216, Ch. 219, Ch. 222
**Exercise Type**: System Design
---

### Story Context

**From:** Dr. Ravinder Singh, Head of Analytical Chemistry
**To:** All PharmaSync Staff
**CC:** Olusegun Adeyemi (CEO), Dr. Miriam Osei (CIO), Sandra Kowalczyk (Head of Regulatory)
**Subject:** Opposition to LIMS Migration — Formal Notice
**Date:** Thursday, 08:12

Dear colleagues,

I am writing to formally oppose the proposed migration of our Laboratory Information Management System, known internally as "Gerald," to the new Thermo Fisher Scientific SampleManager LIMS platform.

Gerald has been our primary system for laboratory data management for 15 years. Every assay method, every analytical result, every instrument calibration record in this company's history lives in Gerald. In that time, our laboratory has generated 847,000 analytical results that are referenced in 23 IND filings, 4 Phase I trial reports, 2 Phase II trial reports, and our current NDA package for PR-7714.

The CIO's office has characterized this migration as "necessary for modernization." I would characterize it as a CIO vanity project that puts 15 years of regulatory history at unnecessary risk.

My specific concerns:

1. **Regulatory continuity**: Gerald's data is referenced in active regulatory submissions. A migration that changes data structure or storage location creates a chain-of-custody question the FDA can and will ask.

2. **Instrument qualification records**: Our HPLC, LC-MS, and dissolution testing instruments are qualified against Gerald's instrument module. Re-qualification against a new LIMS costs us 6-12 months of instrument downtime per instrument class.

3. **Method validation packages**: Our ICH Q2(R1) validation packages for 47 analytical methods reference Gerald's result storage format. I do not know what "migration" means for those references.

4. **Human factors**: My team of 12 analytical scientists has used Gerald for an average of 9 years. I am not willing to put them through a LIMS migration during an active NDA submission cycle.

I am not saying Gerald is perfect. I am saying the migration risk has been catastrophically underestimated. If assay data integrity is compromised during migration, I will personally escalate to the FDA's Office of Pharmaceutical Quality.

I expect a response from the CIO's office and engineering leadership within 48 hours.

Dr. Ravinder Singh
Head of Analytical Chemistry, PharmaSync

---

**From:** Dr. Miriam Osei, CIO
**To:** [You]
**CC:** Olusegun Adeyemi (CEO)
**Subject:** RE: Dr. Singh's email — please call me
**Date:** Thursday, 08:44

I need you on a call in 20 minutes. Dr. Singh has also pulled two of his senior scientists — Dr. Keiko Hayashi and Dr. Adebayo Falade — from the migration working group. Without their cooperation, we have no access to Gerald's data dictionary. Gerald's original developer, a contractor named Phil Mercer, left in 2017. There is no documentation.

The CEO wants this migration done before fiscal year end for the ERP integration project to proceed. Budget is committed.

But Dr. Singh is not wrong about the regulatory risk. We need him on our side.

Your job is not to prove Gerald is bad. Your job is to prove the new system respects his workflows and his data. Figure out how to do that.

— Miriam

---

**1:1 Transcript — [You] and Dr. Ravinder Singh**
**Location:** Lab C-4 Conference Room
**Date:** Thursday, 14:00

**[You]:** Dr. Singh, thank you for making time. I want to start by saying — I read your email and I think most of your concerns are technically valid.

**Dr. Singh:** [pause] I wasn't expecting that.

**[You]:** The instrument qualification issue alone — that's a real problem. If we migrate Gerald and then have to re-qualify 8 instrument classes, we're looking at 12 months of disruption. I don't think the CIO's office factored that in.

**Dr. Singh:** They didn't. The presentation I saw showed a "data migration" as a single line item. Eleven words. Eleven words to describe moving 15 years of GxP data.

**[You]:** Here's what I want to propose. Not a migration — a coexistence strategy. Gerald stays running. We build the new system around Gerald, not replacing it. Strangler fig pattern — do you know it?

**Dr. Singh:** I'm a chemist. Explain it.

**[You]:** A strangler fig is a vine that grows around a tree. Over time it takes over more and more functions until the tree eventually dies naturally — but it never kills the tree abruptly. New assays go into SampleManager. Old assays stay in Gerald. We build a read layer on top of both that makes them look like one system. After 3 years, when Gerald has organically transferred its workload, we sunset it.

**Dr. Singh:** [long pause] What about the historical data? The 847,000 results?

**[You]:** They stay in Gerald. Gerald becomes a read-only archive. We build a query federation layer. Any system that needs an old result queries the federation layer — it goes to Gerald. Any new result goes to SampleManager.

**Dr. Singh:** If Gerald is read-only, who maintains it? Phil Mercer's contract ended 7 years ago. I've been running Gerald on tribal knowledge since then.

**[You]:** That's... that's actually the most important thing you've said to me today. Can I ask — have you documented that tribal knowledge anywhere?

**Dr. Singh:** [quietly] No. It's in my head and Keiko's head. And some of Adebayo's.

**[You]:** Then part of this project — before we do anything else — is getting that knowledge out of your team's heads and into a system. Call it a Gerald Rosetta Stone.

**Dr. Singh:** [slight smile] Gerald is not a dead language yet.

**[You]:** Not yet. But the thing I need to tell you, and I need you to sit with this — Gerald has never been validated to 21 CFR Part 11 standards. Not fully. I reviewed the system inventory last week.

**Dr. Singh:** [long silence] What does that mean for our submissions?

**[You]:** It means every historical result in Gerald is in a gray area. Not invalid — the FDA isn't going to retroactively challenge 15 years of submissions. But if they do a systems inspection and ask "show me your Part 11 validation for the system that generated these results," we cannot currently do that. The migration — done right — is actually our opportunity to fix that. We create an audit trail for the migration itself that documents the provenance of every historical record. That becomes our Part 11 remediation.

**Dr. Singh:** [very long pause] I want Keiko and Adebayo in the next meeting. And I want Sandra Kowalczyk there.

**[You]:** Done.

---

**#lims-migration | Slack**

**keiko.hayashi [16:34]:** Ravinder forwarded me notes from your meeting. I have one question before I agree to cooperate: how are you going to handle the custom Gerald fields? We have 23 fields that have no equivalent in SampleManager. These are not standard LIMS fields — they're fields Ravinder's team invented for dissolution kinetics modeling over 12 years.

**[you] [16:41]:** That's a schema archaeology project. We need to document every custom field, understand what it stores, and either: find an equivalent in SampleManager, create a custom field in SampleManager, or leave it in Gerald permanently.

**keiko.hayashi [16:44]:** "Leave it in Gerald permanently" is fine as long as the query federation layer can surface it alongside new results.

**adebayo.falade [16:47]:** The real problem is going to be Gerald's batch numbering. Gerald uses a composite batch ID format that encodes equipment ID, analyst ID, and date in a non-standard way. Every reference to a batch number in every report we've ever written uses Gerald's format. If SampleManager generates different batch IDs for new batches, we're going to have two incompatible batch numbering systems.

**[you] [16:52]:** The federation layer needs to normalize batch numbering across both systems. That means we need a canonical batch ID registry that maps Gerald batch IDs and SampleManager batch IDs to a single canonical format.

**adebayo.falade [16:54]:** And that canonical format needs to be in every future report. Because if someone opens a 2009 report and a 2027 report side by side, the batch numbering needs to be traceable across the transition.

**[you] [16:57]:** This is getting more complex. I'm going to need you both in a working session tomorrow. Can you bring Gerald's data dictionary — whatever documentation exists?

**keiko.hayashi [16:58]:** There's no formal data dictionary. But I have a 47-page Word document I wrote in 2019 explaining how Gerald works. I've never shown it to the CIO's office because I was worried they'd use it to accelerate the migration.

**[you] [16:59]:** That document is gold. Please bring it.

---

### Problem Statement

PharmaSync must migrate from Gerald — a 15-year-old Oracle Forms LIMS system containing 847,000 analytical results referenced in active FDA submissions — to Thermo Fisher SampleManager LIMS, without interrupting ongoing NDA submission activities and without creating a regulatory chain-of-custody gap. The migration cannot be a big-bang replacement: Gerald cannot go offline, its historical data cannot be structurally altered, and instrument qualifications cannot be invalidated. You must design a strangler fig migration architecture that lets Gerald and SampleManager coexist indefinitely, with a query federation layer that makes both appear as one system to upstream consumers (regulatory reporting, QC dashboards, ERP integration). Additionally, Gerald's historical data has never been validated to 21 CFR Part 11 standards — the migration must produce a Part 11 remediation audit trail for historical records.

### Explicit Requirements

1. Zero downtime migration — Gerald must keep serving live traffic throughout
2. Strangler fig pattern: new assays go to SampleManager, old assays stay in Gerald
3. Query federation layer: single API that makes Gerald + SampleManager appear as one LIMS
4. Canonical batch ID registry: maps Gerald batch IDs and SampleManager batch IDs to a single canonical format, with full lineage
5. Custom field support: Gerald's 23 custom fields must remain queryable through the federation layer even if they have no SampleManager equivalent
6. Part 11 remediation audit trail: for every historical Gerald record, produce an audit entry documenting: data source (Gerald), record content hash at time of audit, that no migration transformation was applied (read-only archival)
7. Instrument qualification continuity: instrument qualification records must be queryable through the federation layer without requiring re-qualification
8. Data lock for historical records: Gerald's historical data becomes read-only, enforced at the database level, after the federation layer is deployed

### Hidden Requirements

- **Hint: re-read Dr. Keiko Hayashi's Slack message about custom fields.** She mentions 23 custom fields, some of which have "no equivalent in SampleManager." But she says "leave it in Gerald permanently is fine as long as the query federation layer can surface it." This implies the federation layer's query schema must be extensible — new custom fields from SampleManager will also need to be mapped into the canonical schema over time. Design for a schema registry, not a hardcoded field map.

- **Hint: re-read Dr. Adebayo Falade's message about batch numbering.** He says "every reference to a batch number in every report we've ever written uses Gerald's format." This means the canonical batch ID registry cannot simply supersede Gerald's format — it must preserve Gerald's original batch IDs as a queryable alias. Reports generated years ago must still resolve when batch IDs are looked up.

- **Hint: re-read the 1:1 transcript.** Dr. Singh says "Phil Mercer's contract ended 7 years ago" and admits the tribal knowledge is undocumented. The 47-page Word document Dr. Hayashi reveals is not just a migration aid — it is the closest thing to a Gerald data dictionary that exists. Before any migration work begins, this document must be ingested and structured into a schema registry. The order of operations matters.

- **Hint: re-read Dr. Singh's formal email.** He mentions "ICH Q2(R1) validation packages for 47 analytical methods reference Gerald's result storage format." If the federation layer changes the effective data model that these method validation packages reference, it creates a regulatory document update requirement — 47 method validation packages may need to be updated to reference the federation layer as the authoritative data source, not Gerald directly.

### Constraints

- **Gerald**: Oracle Forms 2009, Oracle DB 11g, no formal API, accessible only via JDBC and direct SQL queries
- **Historical records**: 847,000 analytical results, 23 custom fields, composite batch ID format, 15 years
- **SampleManager**: Thermo Fisher, REST API, PostgreSQL backend, LIMS-standard field schema
- **Part 11 history**: Gerald has never been Part 11 validated — audit trail for historical records is remediation, not original documentation
- **Team**: Dr. Singh's team of 12 scientists (now cooperating), 3 engineers, 1 regulatory consultant
- **Timeline**: Migration to coexistence mode within 90 days (before fiscal year end); full Gerald sunset in 3 years
- **Budget**: $180K for the 90-day sprint (engineering + SampleManager licenses)
- **Zero re-qualification**: Instrument qualifications against Gerald must transfer without re-qualification events
- **Tribal knowledge**: Keiko's 47-page Word document is the only data dictionary
- **Active submission**: NDA submission (Ch. 219) is in parallel — no disruption permitted

### Your Task

Design the strangler fig migration architecture for Gerald → SampleManager coexistence. Your design must address: the query federation layer, the canonical batch ID registry, the custom field schema registry, the Part 11 remediation audit trail for historical data, the data lock enforcement on Gerald's historical records, and the 3-year sunset plan.

### Deliverables

- [ ] Mermaid architecture diagram showing: Gerald (read-only, Oracle DB 11g) and SampleManager (active write, PostgreSQL) both behind a QueryFederationLayer; BatchIDRegistry as a shared service; SchemaRegistry for custom fields; upstream consumers (RegReporting, QCDashboard, ERP) all hitting the federation layer — never Gerald or SampleManager directly
- [ ] Database schema for: `canonical_batch_ids` (Gerald alias, SM alias, canonical ID, lineage), `schema_registry` (field name, source system, canonical name, type, description), `part11_remediation_audit` (gerald_record_id, content_hash, audit_timestamp, auditor)
- [ ] Strangler fig migration plan: which Gerald modules migrate in which order, over the 3-year timeline, with milestones and rollback criteria
- [ ] Part 11 remediation strategy: how you produce a compliant audit trail for records that were never Part 11-compliant, without altering the records themselves
- [ ] Custom field handling: schema registry design, how new custom fields from SampleManager are added without requiring federation layer code changes
- [ ] Scaling estimation:
  - Gerald: 847K records × 2KB avg = 1.7GB historical data
  - Federation layer: 100 queries/day × 365 days × 3 years = 109,500 queries — query plan for cross-system joins
  - SampleManager growth: estimate 50K new records/year × 2KB = 100MB/year
- [ ] Tradeoff analysis (minimum 3):
  - Strangler fig vs parallel-run vs big-bang migration (risk vs speed vs complexity)
  - Synchronous federation (real-time queries to both systems) vs async replication (copy Gerald data to a read replica) — consistency vs latency
  - Part 11 remediation by record-level hash audit vs full re-validation of Gerald (completeness vs cost)
- [ ] Cost modeling: federation layer + audit trail infrastructure ($X/month for 3-year coexistence period); estimate Gerald Oracle DB license cost savings at sunset
- [ ] Capacity planning: 3-year plan — what triggers Gerald sunset? Define measurable criteria (e.g., "< 5% of queries touch Gerald-only data for 6 consecutive months")

### Diagram Format

Mermaid syntax. Show: Oracle DB 11g (Gerald, read-only) and SampleManager PostgreSQL as two separate data stores. QueryFederationLayer in the middle with a BatchIDRegistry service and SchemaRegistry service. Upstream consumers (RegReporting, QCDashboard, ERP, InstrumentQual) connect only to QueryFederationLayer. New assay writes go directly to SampleManager. Part11AuditService runs as a separate process, reading from Gerald, writing to Part11RemediationStore.
