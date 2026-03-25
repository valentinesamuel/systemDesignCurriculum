---
**Title**: PharmaSync — Chapter 217: Lab Data Integration
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #data-pipeline #integration #etl #database #api-design
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 216, Ch. 51 (OmniLogix CDC), Ch. 56 (LuminaryAI ETL)
**Exercise Type**: System Design
---

### Story Context

**Week 3. The PharmaSync open floor plan. Tuesday, 10:20am.**

You've been quietly working through the Part 11 architecture when a Slack message from Danielle Osei — no relation to Tariq — the head of customer success, lands in the engineering channel. Danielle has the particular tone of someone who is professionally cheerful but has been carrying a fire for three weeks.

---

**#engineering — Slack**

**10:22am — Danielle Osei [Customer Success]**
> Quick question for the team — Crestwood BioSciences has been asking about the lab data unification feature for 6 months. They're one of our three largest accounts. Where does that stand?

**10:24am — Tariq Osei**
> Still in design. No ETA.

**10:25am — Danielle Osei**
> Their VP just sent me this:

> *"We have 12 labs across the UK, US, and Germany. Each lab runs a different LIMS — LabVantage in the US, Thermo SampleManager in the UK, Benchling in one of our German labs, and a custom Oracle-based system in the other two. Getting a unified view of experimental results from a single drug candidate across all labs takes my data team 3 weeks of manual consolidation every time we want a cross-lab analysis. This is why we're paying PharmaSync. If you can't solve this in Q3, we are evaluating competitors."*

**10:26am — Tariq Osei**
> @channel — who has capacity?

**10:28am — Kiran Mehta [Staff Engineer]**
> I've been saying we need a proper integration layer for 18 months. Not "I told you so" — just context. The problem is gnarly. Every LIMS vendor uses a different data model, different API protocols (some are REST, some are SOAP, some have no API and need direct DB access), and different schemas for the same concept. What a "sample" means in LabVantage is structurally different from what it means in Benchling.

**10:31am — Danielle Osei**
> Can someone talk to Crestwood this week?

**10:33am — Tariq Osei**
> @you — I know you just got here, but I need someone without preconceptions on this. Can you talk to Crestwood's lab data team and come back with a design?

---

**Video call — Crestwood BioSciences Lab Data Team — Thursday, 2:00pm**
**Attendees: Dr. Amara Kovacs (Head of Informatics), Sam Lin (Lab Data Engineer), You**

> **Dr. Kovacs**: "I want to show you what our consolidation process looks like today."

She shares her screen. An Excel spreadsheet. 47 columns. 12 color-coded tabs, one per lab. Manual formulas linking cells across tabs.

> **Dr. Kovacs**: "This is the consolidated view for compound GX-441. It took Sam four days to build. We run drug candidates through 15-20 different assays across the labs. Each assay generates results in different units, different precision, different reference ranges. Some labs run duplicates, some run triplicates. Reconciling that is not just a data problem — it's a scientific problem."

> **Sam Lin**: "The other thing is — this is read-only. When we want to trace back to the original instrument data, we have to log into each LIMS separately. There's no way to go from the consolidated view to the source record without switching systems."

> **Dr. Kovacs**: "And the LIMS vendors hate each other. LabVantage's API documentation is 800 pages long and hasn't been updated since 2018. Benchling has a decent REST API but their webhooks are unreliable — we've had events just... not arrive. The German Oracle system has no API at all. The previous vendor had a direct database connection with read access to the Oracle schema."

> **You**: "What does 'unified view' mean to you? Are you looking for a search interface, a reporting dashboard, an API you can query programmatically?"

> **Dr. Kovacs**: "All three. But the most important thing is scientific reproducibility. If I run a model on consolidated data today and the same model on the same data in 6 months, I should get identical results. That means I need point-in-time snapshots. If a lab re-analyzes a sample and updates the result, I need to know the original value."

> **Sam Lin**: "Also — and this is critical — the data coming in is not always clean. Labs make mistakes. There are outliers that are legitimate scientific anomalies and outliers that are data entry errors. We need to be able to annotate the data — 'this result excluded from analysis, reason: instrument calibration failure on 2024-03-15' — and those annotations need to be auditable."

**[Hidden clue: Sam Lin says "point-in-time snapshots" and "original value." Dr. Kovacs says "scientific reproducibility." This is a requirement for bi-temporal data storage — you need both the valid time (when the sample was taken) and the transaction time (when the data was entered into the system). Updating a result cannot overwrite the original — it must be a new version.]**

---

**Slack DM — Kiran Mehta → You — Friday, 4:30pm**

> **Kiran**: I worked on the first attempt at this 18 months ago. We got killed by schema heterogeneity. Let me give you the graveyard tour.

> **Kiran**: LabVantage calls it a "sample container." SampleManager calls it a "aliquot." Benchling calls it a "sample." Oracle LIMS calls it a "specimen." They're all the same thing — a physical tube of material derived from an experiment. But the metadata attached is completely different. LabVantage has 40 fields. Benchling has 12. Oracle has 8.

> **Kiran**: First attempt, we tried to build a universal schema. We got to 200 columns and gave up. Second attempt, we tried EAV (Entity-Attribute-Value). It was queryable but the performance was terrible and it was impossible to enforce scientific constraints.

> **Kiran**: What I think we actually need is a hub-and-spoke model with a canonical ontology. Not a universal schema — a translation layer. Each LIMS vendor has an adapter that maps their data to a canonical intermediate format. The canonical format only captures what's scientifically meaningful across all vendors, plus a JSON blob for vendor-specific fields.

> **You**: What about the bi-temporal problem? Dr. Kovacs needs point-in-time reproducibility.

> **Kiran**: That's why EAV failed us. You can't do bi-temporal queries efficiently on an EAV model. You need a proper versioned record store. Every result is immutable once committed. Updates create new versions. We keep all versions forever.

> **Kiran**: The other graveyard problem: the German Oracle system. We had a direct DB connection. When Oracle released a patch, the schema changed without notice. Our queries broke silently. We didn't notice for 11 days. Crestwood lost 11 days of data ingestion.

---

**Email — From: Tariq Osei → You — Monday, 9:03am**
**Subject: Crestwood Q3 Commitment**

> I've told Danielle we'll have a design proposal ready in 5 business days to share with Crestwood. The design needs to cover:
>
> 1. How we connect to 4 different LIMS systems (including the no-API Oracle one)
> 2. How we handle schema heterogeneity without a 200-column nightmare
> 3. How we guarantee data immutability and point-in-time reproducibility
> 4. How we handle lab annotation/exclusion workflows with audit trail
>
> Crestwood isn't the only customer with this problem. If we design this right, it becomes a product for all 8 of our pharma customers. Design for that scale.
>
> — Tariq

---

### Problem Statement

PharmaSync needs a laboratory data integration platform that can ingest experimental data from heterogeneous LIMS systems across multiple vendors, normalize it to a canonical format, and present a unified queryable view that guarantees point-in-time reproducibility, data immutability, and full audit trail of all annotations and exclusions.

The system must handle at least 4 LIMS vendors with fundamentally different data models, APIs ranging from modern REST to no-API direct database access, and scientific data that requires bi-temporal versioning.

### Explicit Requirements

1. Ingest data from: LabVantage (REST API), Thermo SampleManager (SOAP API), Benchling (REST + webhooks), Oracle-based custom LIMS (no API — direct read-only DB connection)
2. Normalize data to a canonical schema representing samples, experiments, assays, and results
3. Guarantee data immutability: once committed, a result cannot be overwritten — only superseded by a new version
4. Support bi-temporal queries: "give me the result for sample GX-441-003 as of experiment date 2024-01-15 as it was recorded on 2024-02-20"
5. Support annotation and exclusion workflow with audit trail (who excluded what, when, why)
6. Handle schema evolution: if a LIMS vendor adds or removes fields, the integration must not break silently
7. Provide a unified search/query API across all labs and LIMS systems

### Hidden Requirements

- **Hint**: Re-read Dr. Kovacs — "point-in-time snapshots" and "if a lab re-analyzes a sample and updates the result, I need to know the original value." This is bi-temporal modeling (valid time vs. transaction time). Your schema must track both. See Ch. 76 Pattern Summary for bi-temporal patterns.

- **Hint**: Re-read Kiran's message about the Oracle patch — "the schema changed without notice, our queries broke silently, we didn't notice for 11 days." Your Oracle connector needs schema version detection and alerting. A schema change in the source should trigger a human review, not silent data loss.

- **Hint**: Re-read Sam Lin — "annotations need to be auditable." This is a Part 11 requirement (see Ch. 216). Annotation/exclusion records are electronic records that modify clinical data — they fall under 21 CFR Part 11 audit trail requirements, not just application-level logging.

### Constraints

- **Labs**: 12 labs × 4 countries (US, UK, Germany)
- **LIMS vendors**: 4 (LabVantage, SampleManager, Benchling, Oracle custom)
- **Instrument types**: 50+ (mass spectrometers, sequencers, chromatography, NMR)
- **Data volume**: Crestwood: ~500K sample results/year; all 8 customers: ~4M results/year
- **Query SLA**: Cross-lab consolidated view query < 5 seconds; audit trail query < 30 seconds
- **Immutability**: Results must be retained for 15 years (regulatory requirement)
- **Team**: 3 engineers assigned to this project
- **Oracle connector**: read-only access to the Oracle schema, no API, JDBC/ODBC connection

### Your Task

Design the laboratory data integration platform. Your design must cover the ingestion layer (adapters per vendor), the canonical data model, the bi-temporal storage layer, the annotation/exclusion workflow, and the unified query API.

### Deliverables

- [ ] Mermaid architecture diagram showing: 4 LIMS adapters, canonical hub, bi-temporal storage layer, annotation service, query API, and audit trail integration (from Ch. 216)
- [ ] Database schema for the canonical data model:
  - `canonical_samples` table (with valid_from, valid_to, recorded_at, recorded_by for bi-temporal support)
  - `canonical_results` table (versioned, immutable)
  - `data_annotations` table (with audit trail FK to Ch. 216 audit system)
  - `lims_source_registry` table (vendor, connection type, schema version)
  - Include column types and indexes
- [ ] Scaling estimation:
  - 4M results/year × 15-year retention = storage math
  - Schema version tracking overhead
  - Query performance at 60M result rows (bi-temporal index strategy)
- [ ] Tradeoff analysis (minimum 3):
  - Bi-temporal PostgreSQL vs. append-only event log vs. Apache Iceberg time-travel
  - Hub-and-spoke canonical model vs. federated query (query each LIMS live)
  - Pull-based polling vs. push-based webhooks/CDC per vendor
- [ ] Cost modeling: monthly AWS cost for ingestion infrastructure + canonical data store at current and 12-month projected scale
- [ ] Capacity planning: 18-month horizon assuming 3 new pharmaceutical customers per quarter
- [ ] Schema evolution strategy: how to handle LabVantage adding a new field without breaking the integration

### Diagram Format

All architecture diagrams in Mermaid syntax. Include the Oracle schema-change detection mechanism explicitly in the diagram.
