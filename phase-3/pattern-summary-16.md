---
**Title**: Pattern Summary 16 — Staff Patterns VI: System Retirement, Strangler Fig, Deprecation Strategy, Organizational Change
**Type**: Pattern Summary
**Covers**: Ch. 230–244 (NexusWealth, PropIQ)
**Related Chapters**: Ch. 230–244
---

### From Dr. Adaeze Obi (Voice Note Transcript — Slack)

*Transcribed from voice note, 11:34 AM*
*Duration: 6 minutes, 22 seconds*

---

Hey. I was going to write this up but I think it's easier to just talk through it. So. Here we go.

I've been sitting with the NexusWealth arc and the PropIQ arc for a few days, and there's a through-line I want to name before you move on. Both of those situations — the CFO presentation at NexusWealth, and the inherited disaster at PropIQ — involved systems that weren't broken. Not really. The NexusWealth pension calculation engine had been running for eleven years. PropIQ's data ingestion LIMS had been running for seven. Neither of them was a disaster on its terms. They were disasters on your terms. On the terms of the people who came after.

I want to ask you something, and I'm not going to answer it. When you deprecated the LIMS system at PropIQ — when you stood in that room and made the case for replacing it — who lost power? Not access rights. Not system permissions. Power. Whose expertise became less relevant the moment the new system went live? And who gained something? A title, a budget line, a seat at the architecture table? I'm asking because the resistance you encountered wasn't irrational. It was rational. People were defending something real.

And here's the harder question: did the architecture change reflect that power shift? When you drew the strangler fig diagram — the facade API, the shadow writes, the gradual traffic cutover — did the migration plan account for the people who knew the old system in their bones? Or did the migration plan assume that the old system was wrong, and that replacing it was therefore simply a matter of correctness?

Because here's what I've noticed, reviewing systems across a lot of organizations: the most dangerous assumption in system retirement is that the old system was wrong. Usually it wasn't wrong. It was right, for its time, for its constraints, for the team that built it. The people who built it made good decisions with the information they had. The system lasted seven years. Eleven years. That's not failure. That's success. The world changed and the system didn't change with it, which is a different thing.

What changes when you can say that out loud? In the room, I mean. Not in the postmortem document. In the actual meeting, when someone who built the old system is sitting across the table from you. What changes in your design conversation when you start from "this system was right once" instead of "this system is wrong now"?

I'm not going to answer that for you. But I want you to feel the difference between those two starting points before you run your next deprecation.

One more thing. At the end of the PropIQ arc you ran the data quality crisis — the quarantine before fix, the downstream impact analysis, the lineage trace. You did those things in the right order. And I noticed that you did something that most engineers skip: you told the downstream teams what you found before you fixed it. You gave them bad news before you had a solution. That is a trust-building action. It costs you credibility in the short term and builds it over years. I just want to name that, because in system work it's easy to only talk about the technical decisions and miss the relational ones. The relational decisions are also architectural decisions. They determine whether your next migration goes smoothly or whether it's fought at every step.

Last question. Not about NexusWealth or PropIQ. About you. What would this look like if you were wrong about why the old system was built?

Don't answer me. Write it down somewhere. You'll need it.

---

*— Dr. Adaeze Obi*
*[voice note ends]*

---

### Patterns Covered (Ch. 230–244)

**Pattern 1 — Long-Term Data Retention Architecture (50+ Year Horizon)**
*(Ch. 231 — NexusWealth Pension Records)*

Immutable storage tiers designed for multi-decade retention require explicit separation of hot (queryable), warm (archival, slower), and cold (regulatory hold, write-once) layers. Schema evolution over decades is the central design challenge: columns added, types changed, entire record formats obsoleted. Solutions include schema versioning in the record itself (a `schema_version` column read by a versioned deserializer), Avro or Protobuf for binary-compatible evolution, and event sourcing to preserve the original event regardless of current schema. The key insight: for 50-year retention, the biggest risk is not storage cost — it is the inability to read a record written in 2025 by software running in 2060.

**Pattern 2 — SEC / ERISA Compliance: Audit Trail Reproducibility**
*(Ch. 232 — NexusWealth Regulatory Audit)*

Financial and pension regulations (SEC Rule 17a-4, ERISA) require not just retention of records but reproducibility of calculations. An auditor asking "what was this customer's account value on March 15, 2019" must receive the answer that the system would have produced on that date — not the answer the current calculation engine produces when run against that date's data. This is the bi-temporal data model applied to regulatory compliance: valid time (when the fact was true in the world) and transaction time (when the system recorded it) are both preserved. Calculation logic is versioned alongside data, and the version in effect at a given transaction time is retrievable. The pattern is sometimes called "calculation reproducibility" and it is distinct from standard audit trails.

**Pattern 3 — Real-Time Portfolio Risk at Institutional Scale**
*(Ch. 233 — NexusWealth Risk Engine)*

Institutional-scale portfolio risk computation (VaR, stress tests, Greeks) cannot run as a full recalculation on every price tick — the math is too expensive. The pattern is snapshot plus delta: a full risk snapshot is computed at market open (expensive, batch) and cached. Intraday, only the delta — the incremental change from the latest price — is computed and applied to the cached snapshot. This reduces per-tick computation from O(portfolio size × market factors) to O(changed positions × affected risk factors). The architectural challenge is cache invalidation: when is the snapshot stale enough to require a full recompute rather than delta application? The answer is typically a staleness threshold plus a circuit breaker: if delta drift exceeds X%, force a full recompute.

**Pattern 4 — CFO Cost Presentation Structure**
*(Ch. 235 — NexusWealth Infrastructure Review)*

Translating architecture into CFO-readable language requires a specific structure: current state cost (what we spend), risk of inaction (what it costs if we don't change, in business terms, not technical terms), proposed change cost (what the migration costs), and ongoing savings (what the new state costs per month, per year). The most common mistake is presenting the technical risk ("the LIMS is a single point of failure") without translating it into business risk ("a LIMS outage during the regulatory filing window means a missed SEC deadline, which carries penalties of up to X"). The CFO is not uninterested in architecture. The CFO is interested in architecture when it is expressed as risk and money. Give them that, and they become an ally. Give them topology diagrams, and you get five minutes instead of fifty.

**Pattern 5 — Strangler Fig for High-Stakes Migration**
*(Ch. 238 — PropIQ LIMS Migration)*

The strangler fig pattern for high-stakes systems (those serving live production traffic with no maintenance window) proceeds in three phases: facade, shadow, and cutover. In the facade phase, a new API layer routes all traffic through to the old system while providing the new system's interface contract — no behavior changes, just interface normalization. In the shadow phase, writes go to both systems; reads come from the old system but are reconciled against the new system's output in the background. Divergences are logged and investigated. In the cutover phase, reads are gradually shifted to the new system while the old system remains available for rollback. The key discipline: do not cut over until shadow divergence rate reaches zero, not "near zero." Near zero is not zero in a financial system.

**Pattern 6 — Deprecation with Political Fallout**
*(Ch. 239 — PropIQ LIMS Sunset)*

System deprecation in an organization is a change management problem wearing an engineering costume. The technical work is often the smallest part. The patterns that work: a champions program (identify power users of the old system and invest in migrating them first, turning them from resistors into advocates), migration incentives (teams that migrate early get engineering support; teams that miss the deadline get migrated for them, without customization), and a hard cutoff date with executive backing announced early. "Hard cutoff" means the old system will not be running on date X. This date is not negotiable. Negotiable cutoff dates are not cutoff dates; they are indefinitely extended coexistence arrangements. The strangler fig becomes the permanent architecture. Set the date. Hold the date.

**Pattern 7 — Data Quality Crisis Response**
*(Ch. 240 — PropIQ Inherited Disaster)*

When a data quality incident is discovered in a production system, the correct order of operations is: quarantine before fix. Do not fix the data and then tell downstream consumers. Quarantine the affected records (mark as QUALITY_HOLD, exclude from downstream reads), notify downstream consumers of the quarantine with scope and estimated impact, then fix. This order is counterintuitive — engineers want to fix first — but it prevents a worse outcome: a downstream consumer acting on bad data during the window between discovery and fix. Data lineage tracing is the prerequisite: you cannot scope the quarantine without knowing which downstream systems consumed the affected records. If you don't have lineage, build it before you fix the data.

**Pattern 8 — Multi-Party Data Room Access Control**
*(Ch. 241 — PropIQ Due Diligence Data Room)*

A data room for multi-party access (M&A due diligence, regulatory review, academic consortium) requires ABAC (attribute-based access control) rather than RBAC. The reason: access is not determined by role alone but by the intersection of role, organization, time, and purpose. An ABAC policy engine evaluates a policy triple: (subject attributes, resource attributes, environment conditions). Time-bounded access tokens are issued per access grant, not per user — a user may have different access levels to different documents within the same data room, and each level expires independently. Every access event (including views, downloads, failed attempts) is logged to a per-resource audit trail. The audit trail is the product for regulatory data rooms: the access log may be subpoenaed.

**Pattern 9 — Hybrid Ingestion Pipeline**
*(Ch. 242 — PropIQ Property Data Ingestion)*

Real-world data ingestion pipelines rarely receive data in a single format. The pattern that emerges at scale is a tiered ingestion hierarchy: structured webhook (preferred, immediate), then streaming file upload (CSV/JSON, near-real-time), then batch SFTP (daily, legacy partners), then OCR fallback (PDFs, scanned documents, lowest trust). Each tier has different SLAs, different validation logic, and different error handling. The common mistake is designing the pipeline for the preferred tier and treating fallback tiers as special cases. In practice, 30–40% of data volume arrives via fallback tiers in any sufficiently diverse data ecosystem. Design the fallback tiers as first-class pipeline branches with their own monitoring, dead-letter queues, and SLAs.

**Pattern 10 — Tenant-Specific Relevance Tuning Without Model Leakage**
*(Ch. 243 — PropIQ Search Relevance)*

In a multi-tenant search platform, tenants want relevance tuned to their data and their users' behavior. But training a relevance model on one tenant's data and exposing it to another creates a model leakage risk: the model's weights implicitly encode the tenant's proprietary data distribution. The pattern is tenant-isolated model versioning: each tenant has a separate relevance model (or a separate fine-tuned layer on a shared base model), trained only on that tenant's interaction data. The base model is shared and updated globally; the tenant-specific layer is updated only from that tenant's signals. This gives tenants customized relevance without cross-contamination. The operational challenge is model proliferation: 500 tenants means 500 fine-tuned layers. Lifecycle management (when to prune stale models, minimum data threshold before fine-tuning is useful) is the engineering problem, not the ML problem.

---

### Cross-Industry Recurrence

**System retirement is universal.** Every industry eventually faces the moment when a system that was right when it was built needs to be replaced. The strangler fig + facade + shadow + cutover sequence appeared at PropIQ (property data LIMS), and variations of it have appeared at NovaPay (payment processor migration), CloudStack (legacy auth provider replacement), and OmniLogix (route optimizer rebuild). The sequence is not specific to any technology. It is a general pattern for changing a system while it is running. The discipline that varies by industry is the acceptable shadow divergence window: in fintech, zero divergence is required before cutover; in property data, a small divergence window may be acceptable while root causes are investigated.

**Data quality as a governance problem.** The quarantine-before-fix pattern from PropIQ recurs in any system where multiple consumers share a data source. It appeared at AgroSense (sensor data quality), Axiom Labs (genomic data integrity), and MindScale (research dataset re-identification). In each case, the instinct was to fix first and communicate later. In each case, the correct order — communicate scope, then quarantine, then fix — is counterintuitive and requires organizational support to execute. The architecture that enables this is data lineage: without knowing who consumed the affected data, you cannot scope the quarantine. Data lineage is not an optimization. It is the prerequisite for responsible data operations.

**Political fallout is load-bearing architecture.** The pattern of champions programs, migration incentives, and hard cutoff dates for system deprecation is not specific to PropIQ or NexusWealth. It is the pattern of any deprecation in an organization where the old system has institutional knowledge embedded in it — and they all do. The engineers who understand the old system deeply are the most important people in the migration. They are also the people most threatened by it. Your deprecation strategy succeeds or fails based on whether you convert them or bypass them. Bypassing them is faster in the short term and catastrophically expensive in the medium term when the migration hits the edge cases only they know about.

---

### Three Reflection Questions

1. Dr. Obi asks: "What changes when you can say, in the room, that the old system was right for its time?" What would change in your next deprecation conversation if you opened with that acknowledgment instead of a technical critique? What would you need to be true about the organization for that opening to be safe to make?

2. The bi-temporal data model for calculation reproducibility (Pattern 2) and the purpose-bound storage model from MindScale (Ch. 257) are both responses to the same underlying question: "What does it mean to know what the system knew, and when?" How would you explain the difference between these two patterns to an engineer who has only worked with point-in-time data models? When does each pattern apply?

3. Tenant-specific relevance tuning (Pattern 10) creates a proliferation problem: 500 tenants, 500 models, 500 model lifecycles. This is a microcosm of a broader pattern in platform engineering — per-tenant customization creates operational complexity that eventually consumes the efficiency gains from sharing infrastructure. Where is the inflection point? How would you design the lifecycle governance (minimum data threshold, staleness pruning, merge-back criteria) for 500 tenant models? And what organizational structure — centralized ML platform team vs distributed tenant ownership — does your answer imply?
