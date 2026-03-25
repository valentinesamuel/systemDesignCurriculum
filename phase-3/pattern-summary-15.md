---
**Title**: Pattern Summary 15 — Staff Patterns V: Geospatial Systems, Seasonal Planning, Multi-Tenancy at Staff Level
**Type**: Pattern Summary
**Covers**: Ch. 216–229 (PharmaSync, HarvestAI)
---

### From Dr. Adaeze Obi (Slack — annotated whiteboard diagram)

---

**#dm — Dr. Adaeze Obi → You**
**2026-03-21 10:07**

```
adaeze.obi: Morning. I was reviewing the PharmaSync LIMS migration retrospective
            and the HarvestAI postmortem you shared with me.

adaeze.obi: I drew something. Let me describe it — I can't send an image here
            that will render properly, but picture this:

adaeze.obi: A whiteboard divided into two columns.
            Left column header: "Where we thought the boundary was."
            Right column header: "Where the boundary actually was."

adaeze.obi: Left column has a clean horizontal line between "our system" and
            "their data." One box on each side. Arrow going left to right
            labeled "API call." Tidy.

adaeze.obi: Right column: the boundary is jagged, diagonal. It cuts through
            the middle of three different boxes — the LIMS database, the
            regulatory submission package, and the audit trail. Little
            question marks sitting on the boundary line.

adaeze.obi: At the bottom of the right column I wrote: "Whoever controls the
            data controls the compliance risk."

adaeze.obi: My question for you: in both PharmaSync (the LIMS deprecation,
            Ch. 220) and HarvestAI (the multi-tenant farm management, Ch. 227),
            the hardest architectural decisions weren't about how data flows.
            They were about who controls the data.

adaeze.obi: In PharmaSync, the FDA doesn't care which system holds the audit
            trail — they care that someone is accountable for its integrity.
            You picked where the boundary was. That was a compliance decision,
            not a technical one.

adaeze.obi: In HarvestAI, the farm cooperative owns the agronomic data, but
            the platform holds the carbon credit submission records. Who can
            delete what? Who can audit what? Those questions have legal answers,
            and the architecture either encodes those legal answers correctly
            or it creates liability.

adaeze.obi: What would the right column of my diagram look like if you were
            wrong about where you drew the boundary?
```

---

### Patterns Covered (Ch. 216–229)

**Pattern 1 — 21 CFR Part 11 Compliance: Electronic Records, Audit Trails, Electronic Signatures** *(Ch. 217, PharmaSync)*

21 CFR Part 11 requires that electronic records in regulated pharmaceutical environments be: attributable (linked to the individual who created them), legible (human-readable and durable), contemporaneous (timestamped at creation, not retroactively), original (not overwritten), and accurate. Practically, this means append-only audit logs (no `UPDATE` or `DELETE` on audit rows), cryptographic signing of records at write time, and retention periods that exceed FDA inspection cycles (often 10+ years for clinical data). Electronic signatures require: the signer's printed name, date and time of signing, and the meaning of the signature (e.g., "reviewed," "approved," "verified"). An e-signature is not a checkbox — it is a legally binding assertion. Design your audit schema to store all five components as first-class columns, not as a JSON blob.

**Pattern 2 — FDA Validation Architecture: IQ/OQ/PQ, Validation Evidence Packages, Retroactive Validation** *(Ch. 218, PharmaSync)*

FDA software validation requires evidence of three phases: Installation Qualification (IQ — the software was installed correctly in the target environment), Operational Qualification (OQ — the software performs its specified functions under normal and boundary conditions), and Performance Qualification (PQ — the software performs correctly under production-representative load over time). Validation evidence packages are documents, not code: test execution records, deviation logs, and approval signatures. The architectural implication: every environment configuration that could affect software behavior must be version-controlled and traceable. "Works on my laptop" is not a validation statement. The production environment configuration is part of the validated system.

**Pattern 3 — Strangler Fig for Legacy LIMS Migration: Facade API, Traffic Shadowing, Gradual Cutover** *(Ch. 220, PharmaSync)*

Migrating a live Laboratory Information Management System (LIMS) in a regulated environment cannot involve a big-bang cutover — the new system must be validated before it receives production traffic, but validation requires production-representative data. The strangler fig pattern resolves this: build a facade API in front of both the legacy and the new system; route all reads to the legacy system initially; shadow-write all mutations to the new system silently (no response returned to the caller); run the new system in shadow mode for a validation period; compare outputs; then gradually shift read traffic from legacy to new. The facade is the migration control plane. It must log every routing decision for the validation evidence package.

**Pattern 4 — Clinical Data Discrepancy Detection: Hash-Based Record Integrity, Discrepancy Alerting** *(Ch. 221, PharmaSync)*

In clinical trial data, a discrepancy between what a site recorded in a CRF (Case Report Form) and what the central system holds is a regulatory event, not just a data quality issue. Hash-based integrity: at the time of original data entry, compute a cryptographic hash of the record (SHA-256 of the canonical JSON representation) and store it alongside the record. On every subsequent read for regulatory submission, recompute the hash and compare. Any mismatch triggers a discrepancy alert routed to a data manager, not silently overwritten. The alert must include: the record identifier, the timestamp of the original hash, the timestamp of the mismatch detection, and the field-level diff. Discrepancy resolution is a documented process, not a database `UPDATE`.

**Pattern 5 — Geospatial Indexing: GiST Indexes in PostGIS, Spatial Partitioning, Tile-Based Queries** *(Ch. 223, HarvestAI)*

PostgreSQL with PostGIS supports GiST (Generalized Search Tree) indexes on geometry columns. A GiST index on a polygon column (`field_geometries.polygon`) enables spatial predicate pushdown: `WHERE ST_Within(point, polygon)` uses the index rather than scanning all rows. At 38M rows, a full table scan for a spatial query runs in 45 seconds (as HarvestAI discovered); the GiST-indexed query runs in 180ms. Spatial partitioning at the table level (partitioning `field_geometries` by a geohash prefix or administrative region) reduces index scan scope for region-specific queries. Tile-based queries (dividing the query area into fixed-size tiles and querying each tile independently) improve cache hit rate for dashboard queries that repeatedly load the same geographic viewport. Always store geometry in a single canonical CRS (EPSG:4326 for global data, or the local national grid standard) — mixing CRS within a table produces silent geometric errors that are extremely difficult to debug.

**Pattern 6 — Seasonal Spike Architecture: Pre-Warming, Auto-Scaling, Spot/Reserved Hybrid, Queue-Based Leveling** *(Ch. 224, HarvestAI)*

Agricultural platforms have the most predictable traffic seasonality of almost any domain: planting season and harvest season are calendar events, not random spikes. This predictability is an architectural advantage that most teams fail to fully exploit. Pre-warming: scale up compute, pre-populate caches, and run `ANALYZE` on all large tables 48 hours before the expected spike — not when the spike arrives. Auto-scaling alone is insufficient for database-heavy workloads because RDS scaling lag (5–15 minutes) is too slow for a sudden 10x traffic event. Spot/reserved hybrid: reserve baseline capacity for the season's expected average; use spot instances for burst capacity above that baseline. Queue-based leveling: for batch workloads (model inference runs, carbon credit submissions), route jobs through a queue and use a configurable concurrency limit to prevent the spike from directly hitting downstream APIs or the database.

**Pattern 7 — Edge ML Model Deployment: OTA Model Updates, Versioning at the Edge, Staleness Flags** *(Ch. 225, HarvestAI)*

Deploying an ML model to an edge device (tractor, sensor node, gateway) requires versioning discipline that cloud deployments rarely enforce. Every model artifact must carry: model version, feature schema version, SQLite schema version (if the model requires schema changes), minimum runtime version (Python, SQLite, firmware), and a compatibility matrix. The OTA delivery system must enforce this matrix before delivering an update: if the device's runtime version does not satisfy the minimum, do not push the update. Include a staleness flag in the model's output: if the model has not been refreshed within its configured staleness window (e.g., 7 days for a predictive maintenance model), the output should include a `staleness_warning: true` field that the device HMI surfaces to the operator. Silent staleness is more dangerous than visible staleness.

**Pattern 8 — Coordinate Reference System Pitfalls: Always Store CRS Metadata with Geospatial Data** *(Ch. 223, HarvestAI)*

This is one of the most common and most silent failure modes in geospatial systems. A coordinate pair `(-23.5505, -46.6333)` is meaningless without knowing its CRS. In EPSG:4326 (WGS 84), that is São Paulo, Brazil. In a local Brazilian national grid projection (SIRGAS 2000), the same numbers refer to a different location hundreds of kilometers away. Every geospatial record must store its CRS as first-class metadata, not as a convention that "everyone knows." PostGIS enforces this via the geometry type's SRID parameter — use it. When ingesting data from external sources (government land registries, machinery manufacturer APIs), validate the CRS against expected values before inserting. A mismatched CRS that silently inserts will produce planting-zone polygons that are geographically correct in one system and completely wrong in another.

**Pattern 9 — Multi-Tenant Hierarchy Design: Nested Orgs (Account → Sub-Org → Resource), RBAC Inheritance** *(Ch. 227, HarvestAI)*

HarvestAI's farm management model has three tiers: the agricultural cooperative (account), the individual farm within the cooperative (sub-org), and the field or tractor (resource). Access control must respect this hierarchy: a cooperative administrator can see all farms in their cooperative; a farm manager can see only their farm's resources; a tractor operator can see only their assigned tractor's data. RBAC inheritance pattern: roles are defined at the account tier (admin, manager, operator); permissions are scoped to the sub-org or resource tier at assignment time. A cooperative admin has `admin` role with scope `account:123`; a farm manager has `manager` role with scope `account:123/suborg:456`. The permission check walks up the hierarchy: a request to access tractor `789` checks permissions at `resource:789`, then `suborg:456`, then `account:123`. Row-level security in PostgreSQL (RLS) is the correct enforcement point for data isolation within this hierarchy — not application-layer filtering, which can be bypassed by a missing `WHERE` clause.

---

### Cross-Industry Recurrence

Geospatial indexing challenges appear across multiple domains in Phase 3 wherever the underlying data is fundamentally geographic. HarvestAI's field polygon queries (Ch. 223), DeepOcean's vessel position stream (Ch. 210), and any mapping or routing platform share the same core problem: spatial predicates on large datasets are catastrophically slow without the right index, and the right index requires the query planner to have accurate statistics. The recurring lesson is not "use a GiST index" — that is the easy part. The recurring lesson is that geospatial systems require the same operational discipline as any other high-cardinality index: monitor statistics staleness, test query plans against production-scale data, and treat the query planner as a component that requires maintenance, not a black box that always makes the right choice.

The seasonal spike pattern at HarvestAI is structurally identical to patterns from earlier in the curriculum — the Champions League notification spike at Beacon Media (Phase 1, Ch. 20), the election-day traffic surge at CivicOS (Phase 1, Ch. 39), and the ticket on-sale event at VenueFlow (Phase 2, Ch. 111). What distinguishes HarvestAI is the predictability: planting season does not arrive without warning. Systems with predictable seasonal spikes have an obligation to pre-warm rather than react. The architectural difference between a reactive auto-scaling strategy and a pre-warming strategy is the difference between a 5-minute degradation window and zero degradation. For agricultural systems, this is particularly consequential: a farmer who cannot access their planting zone data during the first week of planting season has a business-critical outage, not an inconvenience.

Multi-tenant hierarchy design recurs in every SaaS product in the curriculum, but the PharmaSync and HarvestAI arcs reveal a dimension that earlier chapters did not surface: in regulated industries, the tenant hierarchy is not merely an access control concern — it is a compliance boundary. At PharmaSync, the question of who owns the audit trail (the CRO, the sponsor, or the platform) has FDA regulatory implications. At HarvestAI, the question of who can delete carbon credit submission records (the farm, the cooperative, or the platform) has legal implications under the voluntary carbon market's registry rules. Dr. Adaeze Obi's whiteboard diagram captures this precisely: the boundary between "your system" and "their data" is not a technical decision — it is a legal and compliance decision that the architecture must encode correctly.

---

### Three Reflection Questions

1. Pattern 2 describes FDA validation requiring an IQ/OQ/PQ evidence package for every environment configuration that could affect software behavior. If HarvestAI's tractor firmware is a validated system under 21 CFR Part 11 (hypothetically), what does the OTA model update process look like under that constraint? Which parts of the update process would require re-validation, and which could be covered by a "delta validation" approach?

2. Pattern 9 describes RBAC inheritance walking up the account → sub-org → resource hierarchy to resolve permissions. In HarvestAI's carbon credit program, a third party (the carbon registry auditor) needs read access to submission records across multiple cooperatives — but only for records related to carbon credits, not for agronomic data. This cross-tenant, cross-hierarchy access pattern is not described by the standard RBAC inheritance model. How would you extend the model to support it without breaking tenant isolation?

3. Dr. Adaeze Obi's diagram distinguishes between where you think the data ownership boundary is and where it actually is. Consider the HarvestAI postmortem's Incident 2: 40 farms lost carbon credit revenue because a submission job failed silently. Who owns the obligation to ensure that submission? The platform (HarvestAI), the cooperative, or the individual farm? How does your answer to that question change the architecture of the submission job, the DLQ, and the alert routing?
