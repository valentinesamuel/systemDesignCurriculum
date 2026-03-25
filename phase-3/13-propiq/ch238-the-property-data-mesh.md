---
**Title**: PropIQ — Chapter 238: Twelve Sources, One Truth
**Level**: Staff
**Difficulty**: 7
**Tags**: #data-mesh #data-quality #etl #distributed-systems #real-estate
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 239, Ch. 240, Ch. 242
**Exercise Type**: System Design
---

### Story Context

**Meeting transcript — PropIQ Data Quality Review — Week 1**

*Attendees: you (new), Siyanda Dlamini (Head of Data Engineering), Jerome Ashworth (VP Product), Tara Kowalczyk (Senior Data Engineer), Felipe Mendonça (Client Analytics Lead)*

---

**Jerome**: For the new person's benefit — welcome — we do this review monthly. We look at data freshness by source, flag quality issues, and prioritize fixes. Siyanda runs it. Siyanda?

**Siyanda**: Thanks Jerome. Okay. Status by source. County Assessor feeds: we aggregate from 3,114 counties. Update frequency varies. Some counties push updates every 90 days. Some once a year. Eight counties in Texas have not updated since February 2023. I believe their portal changed format and our parser broke.

**Tara**: I flagged that in Jira six months ago.

**Siyanda**: Noted. MLS feeds: daily. Quality is decent but we see structural mismatches — some MLS systems report square footage as "gross" and some as "net leasable." We're mixing them.

**You**: How often does that affect client data?

**Siyanda**: Tara, what's the error rate?

**Tara**: About 4% of commercial listings in the Western US have square footage figures that are gross vs. net inconsistently. For a 100,000 sqft building, that's a 15-20% difference in the core metric clients use for price-per-sqft calculations.

**You**: That's not a data quality footnote. That's a material error in the most important metric.

**Felipe**: Our clients don't know this. When Meridian Capital queries pricing analytics, they get a price-per-sqft figure that for some buildings includes a 4% sample with the wrong denominator. The number looks right. It's not.

**Jerome**: We have a disclosure in our terms of service that data accuracy may vary by source.

**You**: Terms of service disclosures don't protect us from a client making a $100M acquisition based on our bad data.

*[Brief silence]*

**Jerome**: Let's continue the review and come back to that.

**Siyanda**: Building permit data: we get this from 12 municipal APIs and one licensed aggregator. The licensed aggregator covers 80% of volume but their data is 30 days delayed by contract. Municipal APIs are real-time but incomplete — they only cover jurisdictions with digital permit portals. Rural counties often still file paper.

**Tara**: The satellite imagery is the freshest we have but also the least structured. We get 4-band imagery every 30 days. Our computer vision model extracts construction stage, visible parking occupancy, and approximate rooftop equipment changes. That's a guess with 78% accuracy.

**You**: 78% accuracy on construction stage detection — do clients know that's a model estimate?

**Tara**: It's displayed the same as county assessor data.

**You**: It absolutely cannot be displayed the same as county assessor data. One is a government record. The other is a 78%-accurate machine learning model. Those are different epistemic claims.

**Siyanda**: CoStar licensing feed: we have a data sharing agreement that requires we do not display CoStar-originated data in a way that enables clients to substitute CoStar's product with ours. Our contract says we can use CoStar data for "enrichment" but not as primary data. Half our pricing analytics is built on CoStar occupancy figures.

**Jerome**: Let me stop you there. The CoStar licensing issue has been raised before. Legal says we're compliant.

**Felipe**: Legal says we're probably compliant. That's different.

**Siyanda**: Environmental data: we license from three aggregators. EPA ECHO database directly, a commercial aggregator called EnviroRisk, and county-level hazmat filings from a startup that crawls public records. The three sources contradict each other for 12% of properties. We don't have a resolution strategy — we show all three.

**Jerome**: The contradiction display has confused clients before. They see conflicting environmental flags and don't know which one to trust.

**You**: What's the resolution strategy today?

**Siyanda**: There isn't one. We display all three and let the client decide.

**You**: So a client looking at a property with a potential environmental flag has to be their own data scientist to interpret conflicting sources that we don't explain. And if they make the wrong call and there's actually a remediation liability, that's on them.

**Jerome**: That's... essentially correct.

**You**: I need a week to map all 12 sources, their update patterns, their quality levels, their licensing constraints, and their conflict rates. Then I want to come back with a data architecture proposal.

---

After the meeting you stay late, open a spreadsheet, and begin cataloging. By midnight you have the first honest picture anyone at PropIQ has drawn of what their data actually is versus what the product claims it is. The gap is larger than you expected. And the Meridian Capital contract renewal is in 6 weeks.

### Problem Statement

Design a data mesh architecture for PropIQ's 12-source commercial property intelligence platform. The system must: enforce data quality standards per source, track data provenance and freshness per data point, resolve conflicts between authoritative sources with documented resolution strategy, enforce licensing constraints per source (particularly the CoStar limitation), and provide clients with confidence-scored data rather than uniformly presented output. The platform covers 5 million commercial properties across the United States.

### Explicit Requirements

1. Data source catalog: each of 12 sources has a documented schema, update frequency, quality score, and licensing constraint
2. Per-data-point provenance: every data attribute displayed to clients must be traceable to its source, ingestion timestamp, and version
3. Data quality scoring: automated quality checks per source (schema validation, value range checks, cross-source consistency checks); quality score visible to clients
4. Conflict resolution: for the same data attribute from multiple sources, a documented priority hierarchy with fallback logic (e.g., county assessor overrides CoStar for assessed value; CoStar overrides satellite for occupancy if CoStar data is less than 30 days old)
5. CoStar licensing enforcement: CoStar-originated data used for enrichment only, not as primary display; architecture must make this distinction enforceable, not just contractual
6. Data freshness SLAs: define maximum staleness tolerance per data type (e.g., MLS listings: 48 hours; county assessor: 90 days; satellite: 60 days) and alert when exceeded
7. Schema drift detection: county assessor format changes (like the Texas parser break) must be detected within 24 hours and quarantined, not silently corrupted
8. Client-facing confidence labels: display confidence tier (Government Record / Licensed Data / Model Estimate) alongside each data attribute; do not display model estimates as equivalent to records
9. Source-level licensing quarantine: data from sources with licensing ambiguity (CoStar) must be flagged and isolatable without disrupting the rest of the data pipeline
10. Historical data versioning: when source data is corrected or updated, preserve the previous version for audit and recalculation

### Hidden Requirements

- **Hint**: Re-read Siyanda's comment about 8 Texas counties that haven't updated since February 2023 due to a broken parser. The parser broke because the county changed their format. This is a schema drift problem — but it went undetected for 6+ months because there was no automated detection. What does a schema drift detection system look like for 3,114 independent county APIs, each of which can change format at any time without notice? The answer is not "just add a parser test" — it's a pattern for detecting that a source has stopped producing valid output.
- **Hint**: Re-read Felipe's comment: "Our clients don't know this." The 4% MLS square footage error is not disclosed to clients at the data level — it's buried in terms of service. Now consider: if Meridian Capital (the enterprise client with the renewal coming up) made an acquisition in the Western US in the last 6 months, there's a probability they used a price-per-sqft figure that was calculated with the wrong square footage. This is a latent legal exposure issue — the same pattern as the PropIQ data quality crisis in Chapter 240. The data quality architecture you build here directly determines the legal exposure in that future chapter.
- **Hint**: Re-read the CoStar licensing discussion. Jerome says "legal says we're probably compliant." Felipe says "that's different." The architectural implication: "probably compliant" is not compliant. If CoStar audits PropIQ's data usage and finds that CoStar occupancy data is the primary display source for 50% of pricing analytics, PropIQ faces a licensing violation. The architecture must enforce the primary-vs-enrichment boundary technically — not just document it in a data dictionary. How do you tag data lineage so that a CoStar-origin attribute can never reach the "primary display" path?
- **Hint**: Re-read Tara's note about satellite imagery: "displayed the same as county assessor data." At 78% accuracy, the satellite-derived construction stage estimate is a probabilistic output. If PropIQ presents a "construction stage: Active" flag derived from satellite ML that is wrong 22% of the time, and a client relies on that flag in a valuation, PropIQ has a data accuracy liability that is distinct from the county assessor data liability. The confidence label system you design must distinguish between "this is factually wrong 4% of the time" (MLS sqft error) vs. "this is a probabilistic estimate that is correct 78% of the time" (satellite ML). These are different failure modes.

### Constraints

- **Properties**: 5M commercial properties (US)
- **Data sources**: 12 (county assessors ×3,114 counties, MLS, building permits, satellite imagery, environmental ×3, CoStar, CMBS, FEMA, building permits startup)
- **Update frequencies**: quarterly (county assessor) to real-time (MLS)
- **Quality variance**: 78% accuracy (satellite ML) to government record (county assessor)
- **Licensing constraints**: CoStar — enrichment only, not primary; environmental aggregator — no redistribution of raw records
- **Conflict rate**: 12% of properties have conflicting environmental data across 3 sources
- **Current architecture**: monolithic ETL pipeline; all sources treated identically; no provenance tracking
- **Team**: 3 data engineers, 2 platform engineers
- **Timeline**: data architecture proposal in 1 week; Meridian Capital renewal in 6 weeks

### Your Task

Design the PropIQ data mesh. Address: source domain model (each source as a data domain with schema, quality score, and licensing tag), conflict resolution hierarchy, confidence-tier display system, schema drift detection for 3,114 county APIs, CoStar licensing enforcement at the architecture level, and historical versioning for corrected data.

### Deliverables

- [ ] Mermaid architecture diagram — 12 sources → domain ingestion layer → conflict resolution → property knowledge graph → client API, with licensing quarantine boundary clearly shown
- [ ] Database schema — property record with provenance metadata, source version table, conflict resolution log, data quality score table
- [ ] Data quality scoring model: define the scoring formula for a data attribute (components: source authority level, data age, validation pass/fail, cross-source agreement); show the score for a sample property with 3 conflicting environmental flags
- [ ] Conflict resolution priority hierarchy: table defining which source wins for each major data category (assessed value, occupancy, square footage, construction date, environmental status) with rationale
- [ ] Scaling estimation:
  - Schema drift detection: 3,114 county APIs × 4 checks/day = how many daily parser validation runs? Storage for validation history?
  - Provenance metadata overhead: for 5M properties × average 30 attributes × provenance record size — total storage overhead for provenance layer
  - CoStar licensing query: how do you run an audit query to find all client-facing displays where CoStar is the primary source? Query design + estimated execution time
- [ ] Tradeoff analysis (minimum 3):
  - Conflict resolution: deterministic hierarchy (faster, less accurate) vs. ML-based confidence scoring (more accurate, complex, explainability challenges)
  - Data mesh vs. centralized lake: domain ownership benefits vs. cross-domain query complexity
  - CoStar technical enforcement: data lineage tagging in pipeline vs. separate licensed-data store with explicit API boundary
- [ ] Cost modeling: 12 source ingestion pipelines, provenance storage for 5M properties, schema drift monitoring — $X/month estimate
- [ ] Capacity planning: what does this architecture look like at 15M properties (US + Canada) and 20 data sources in 18 months?

### Diagram Format

Mermaid syntax.
