---
**Title**: BuildRight — Chapter 130: Data Pipeline Schema Evolution
**Level**: Staff
**Difficulty**: 8
**Tags**: #schema-evolution #avro #schema-registry #kafka #contract-testing #data-pipeline
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 26 (OmniLogix CDC), Ch. 50 (LuminaryAI ETL), Ch. 130 (this chapter builds toward ch131 GDPR)
**Exercise Type**: System Design / Inherited Disaster
---

### Story Context

The email arrives on a Monday morning with no warning.

---

**From**: Liam Kowalski
**To**: You
**Subject**: Autodesk broke us again
**Date**: February 14, 2026

Third time in 12 months. This is on me — I knew this was fragile and didn't fix it.

Autodesk released Revit 2026.1 API last night. New required field in the BIM element schema: `classification_system` (IFC standard reference). Our ingestion service doesn't know this field exists. It's failing to parse roughly 60% of incoming BIM sync payloads.

The bad part: this is the third time this has happened. January 2025 they renamed `element_category` to `element_type`. June 2025 they added `parametric_constraints` as a nested object where there used to be a flat field. Both times, we patched the ingestion service manually and reprocessed the backlog.

The worse part: this morning's failure hit during the Al-Barsha project milestone sync (yes, the one with the subpoena). I've paused ingestion for that project and manually flagged it in the DLQ. Legal is going to want to know why there's a gap.

The worst part: Bentley and Trimble also version their schemas. We have no process for any of this.

I'm sorry.
— Liam

---

You forward the email to Thomas Adeyemi with a single line: *"We need a schema governance conversation."*

His response: *"We need a schema governance solution."*

---

**#platform-eng** *(Slack — same day)*

**You**: "Liam — walk me through the current ingestion flow for BIM vendor data."

**Liam Kowalski**: "Vendor sends a webhook with a JSON payload. We have a Node.js service that parses it with hardcoded field mappings. It writes to PostgreSQL. That's it."

**You**: "No schema validation?"

**Liam Kowalski**: "There's a Joi schema, but we wrote it 18 months ago and it doesn't get updated when vendors change their API."

**You**: "What happened to the data during the January and June failures?"

**Liam Kowalski**: "January: we caught it within 2 hours. Manually patched ingestion service, reprocessed ~3,000 payloads from the vendor's retry queue. Data gap: 2 hours. June: caught it in 4 hours. Data gap: 4 hours. This time: caught it in 40 minutes. But the specific project that got hit..."

**You**: "The subpoena project."

**Liam Kowalski**: "Yes."

**You**: "What does the Bentley and Trimble situation look like?"

**Liam Kowalski**: "Bentley: we have one enterprise customer using it. They update quarterly. We haven't had a failure yet but only because their schema hasn't changed in a way that breaks us. Trimble: two customers. Same situation."

**You**: "How do we know when vendors are going to change their schema?"

**Liam Kowalski**: "We don't. Autodesk emails their developer list. I'm on it. I missed the February one because it came in on a Friday and I was at my kid's birthday party."

---

**#bim-integrations** *(Slack — after you've drafted a plan)*

**You**: "I want to move all BIM ingestion to a schema-first approach. Producers declare their schema. We validate against it. Breaking changes require version bumps. Non-breaking changes are auto-compatible."

**Priya Mehta**: "What's a 'breaking change' in this context? From a data science perspective, adding a required field that we don't use in any of our models is technically breaking for you but irrelevant to me."

**You**: "That's exactly the right question. A breaking change for the ingestion pipeline is different from a breaking change for downstream consumers. We need to model both."

**Priya Mehta**: "So when Autodesk adds a field I care about, I want to be able to start consuming it without waiting for a pipeline change?"

**You**: "That's the goal. Schema evolution should let consumers opt into new fields without requiring a full pipeline redeploy."

**Kenji Mori**: "Can we get advance warning? Like a 'schema changelog' feed from vendors?"

**You**: "We can request it. Autodesk has a schema versioning API we're not using. Bentley probably has something similar. Trimble — I'll find out."

**Kenji Mori**: "And when a vendor breaks us with no warning, like this morning?"

**You**: "Dead-letter queue. The payload lands in DLQ with the schema violation tagged. We don't lose it. We reprocess it when we've updated our schema handler. No data gaps."

**Kenji Mori**: "That would have saved us this morning."

**You**: "And the last three times."

---

**From**: Marcus Webb
**To**: You
**Date**: February 15, 2026

Quick thought before you design this. There's a tension most people miss: schema registries enforce contracts *between* producer and consumer, but in this case the producers (Autodesk, Bentley, Trimble) don't use your registry — they have their own release cycle and you have no leverage over them. Your schema registry is actually modeling *your expectations* of their schema, not a contract they've agreed to.

That changes the design. Think about the difference between:
(a) A schema registry that enforces compatibility for systems you control
(b) A schema registry that acts as a translation/adaptation layer for external systems you don't control

Which one are you building? Or both?

— M.W.

---

### Problem Statement

BuildRight ingests BIM data from three external vendors (Autodesk Revit, Bentley, Trimble), each of which evolves their schema independently, without coordinating with BuildRight. Three breaking schema changes in 12 months have caused data pipeline failures ranging from 2 to 4 hours. The current approach — hardcoded field mappings in a Node.js service — cannot absorb schema drift without manual intervention.

The new data pipeline must enforce schema validation at ingestion, route schema-violating payloads to a dead-letter queue for deferred processing, support backward and forward compatibility for non-breaking changes, and provide a schema registry that tracks vendor schema versions and maps them to internal canonical schemas — even though the external vendors do not participate in the registry.

---

### Explicit Requirements

1. Schema registry (Confluent Schema Registry or equivalent): all vendor schemas registered and versioned. Internal canonical schemas registered separately.
2. Schema validation at ingestion boundary: every inbound BIM payload validated against the registered schema for that vendor + API version. Invalid payloads → DLQ, never dropped.
3. Compatibility enforcement: define backward compatibility, forward compatibility, and full compatibility rules per vendor. Auto-reject schemas that violate declared compatibility mode.
4. Schema translation layer: map vendor-specific field names to internal canonical schema (e.g., Autodesk `element_type` → internal `bim_element_category`). Translation rules versioned alongside schemas.
5. Dead-letter queue with schema violation tagging: DLQ entries must record the specific field(s) that caused the violation, the vendor schema version, and the expected schema version.
6. DLQ reprocessing: when the internal schema is updated to handle a new vendor schema version, DLQ entries for that violation type must be replayable without manual intervention.
7. Schema change notification: monitor vendor schema version feeds (where available) and trigger an alert when a new vendor schema version is detected, before it reaches production ingestion.
8. Contract testing: internal services that consume ingested BIM data must have registered consumer contracts; schema changes that break a consumer contract must fail CI/CD.

---

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's email. He distinguishes between a schema registry for systems you control (where producers agree to compatibility rules) vs a schema registry for external systems you don't control (where you're modeling your expectations). Autodesk, Bentley, and Trimble will not register their schemas in your Confluent Schema Registry. How does your schema registry work when the producer doesn't participate? Who registers the "expected" schema for Autodesk's API, and how is that schema kept up to date?
- **Hint**: Re-read Priya Mehta's question: "Adding a required field we don't use in any of our models is technically breaking for you but irrelevant to me." The compatibility determination depends on which downstream consumer you're considering. A field that breaks the ingestion service might be irrelevant to the analytics pipeline. Your compatibility model must be consumer-specific, not just schema-level. How does the schema registry track which consumers depend on which fields?
- **Hint**: Re-read the incident context. The subpoena project (BR-48291 from Ch. 126) has a data gap during the Autodesk failure. DLQ reprocessing will fill this gap — but the event-sourced log from Ch. 126 cares about `transaction_time` (when the event was recorded). Reprocessed events from DLQ will have a `transaction_time` different from their `valid_time`. How does the DLQ reprocessing pipeline interact with the bi-temporal event log from Ch. 126?
- **Hint**: Re-read Liam's email. He says "I missed the February one because it came in on a Friday." The schema change notification system is alerting humans, who can miss it. What is the automated response to detecting a new vendor schema version — before it hits production? Can the pipeline auto-degrade to a more lenient parsing mode when a new unknown version is detected?

---

### Constraints

- **Vendors**: Autodesk Revit (primary: 35,000 projects), Bentley (1 enterprise customer: 500 projects), Trimble (2 customers: 300 projects).
- **Ingestion volume**: 180,000 BIM sync payloads/day (Autodesk), 8,000/day (Bentley + Trimble combined).
- **Schema change frequency**: Autodesk: ~4 per year. Bentley: ~2 per year. Trimble: ~1 per year.
- **DLQ retention**: 30 days (long enough to reprocess after a schema update).
- **Reprocessing SLA**: DLQ entries must be reprocessable within 4 hours of schema update deployment.
- **Schema registry**: Confluent Schema Registry (managed, Confluent Cloud).
- **Message format**: moving from JSON (current) to Avro (target). JSON payloads from vendors must be adapted at the ingestion boundary.
- **Team**: 2 engineers. Six weeks.
- **Legal requirement**: no data loss. All payloads that arrive must be either successfully processed or stored in DLQ for future processing. Drops are not acceptable.

---

### Your Task

Design the schema evolution infrastructure for BuildRight's BIM vendor ingestion pipeline. Address the schema registry design for external producers, the ingestion validation and DLQ flow, the translation layer (vendor schema → internal canonical schema), the consumer contract testing integration, and the DLQ reprocessing pipeline.

---

### Deliverables

- [ ] **Mermaid architecture diagram** — show: vendor webhook → ingestion service → schema validator → Kafka topic (valid) / DLQ (invalid); schema registry; translation layer; downstream consumers; DLQ reprocessor
- [ ] **Schema registry design** — describe how "external vendor schemas" are registered (who registers them, what triggers an update, how versions are tagged); define the compatibility mode per vendor and the rationale
- [ ] **Avro schema for internal canonical BIM element** — define the `bim_element_v1` Avro schema with key fields, nullable fields for optional vendor data, and schema evolution annotations; show the evolution from v1 to v2 adding `classification_system`
- [ ] **DLQ schema** — `bim_dlq` Kafka topic message format: include `payload`, `vendor`, `vendor_schema_version`, `expected_schema_version`, `violation_fields`, `received_at`, `retry_count`
- [ ] **Scaling estimation** — calculate: DLQ storage at 10% failure rate during a schema incident lasting 4 hours (180,000 payloads/day); schema registry storage for 3 vendors × 10 schema versions × average schema size
- [ ] **Tradeoff analysis** — minimum 3: (1) Avro vs Protobuf vs JSON Schema (expressiveness vs tooling vs vendor compatibility), (2) strict validation (reject on unknown fields) vs lenient parsing (ignore unknown fields), (3) consumer-driven contracts (Pact) vs schema registry compatibility checks
- [ ] **DLQ reprocessing design** — describe the reprocessor service: how it detects that a new schema update enables reprocessing of existing DLQ entries; how it handles ordering (DLQ entries must be reprocessed in original arrival order per project)
