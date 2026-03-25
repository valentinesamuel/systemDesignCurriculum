---
**Title**: Axiom Labs — Chapter 136: 200 Universities, 200 Data Silos
**Level**: Staff
**Difficulty**: 8
**Tags**: #data-mesh #data-products #federated-governance #data-catalog #domain-ownership #discoverability
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 133 (Axiom federated computation), Ch. 135 (Axiom columnar storage), Ch. 148 (VertexCloud API governance)
**Exercise Type**: System Design
---

### Story Context

**Platform Governance Meeting — Wednesday 10:00**
**Attendees**: Dr. Priya Sharma, Kofi Mensah, [you], Dr. Elena Sokolova, 3 university research coordinators (invited)

Dr. Priya Sharma opens: "I've invited three research coordinators to give us direct feedback on the platform's data discoverability. Please speak frankly."

**Dr. Yuki Hashimoto (Kyoto University)**: I spent three weeks looking for a specific cohort dataset. It was the "JPN-Rare-Disease-2022" study that Osaka University contributed to the platform in 2022. I found it eventually in a folder named `OU_2022_RDST_FINAL_v3`. When I queried the data, I discovered the column names were in Japanese with no English translation and no data dictionary. I had to contact the contributing researcher directly — who is now at a different institution — to understand the schema.

**Dr. Amara Diallo (University of Dakar)**: I want to share genomic data from our Malaria studies with collaborating institutions. I cannot find the upload process. I cannot find who to ask. I submitted a support ticket 6 weeks ago and received one automated response.

**Dr. Sophia Keller (ETH Zürich)**: Every time I want to use a dataset I didn't contribute myself, I don't know: Is this data current? When was it last updated? Who is responsible for it? Can I cite it in a paper? What is the data quality? I have to contact the original institution and wait weeks for a response.

---

**Post-meeting — Kofi Mensah and [you]**

**Kofi Mensah**: This is the central problem. We have 200 universities contributing data. We have a data lake with 12 petabytes. And none of the 4,000 researchers can find what they need without contacting the people who contributed it.

**[you]**: It's a governance problem, not a storage problem. The ch135 work fixes the query performance. But a researcher can't query data they can't find or don't understand.

**Kofi Mensah**: The obvious solution is a centralized data catalog.

**[you]**: A centralized catalog with Axiom as owner of all the metadata? That requires 200 institutions to submit metadata to us and trust us to maintain it. Universities don't work that way. Each institution has their own data governance policies, their own metadata standards, their own naming conventions.

**Kofi Mensah**: So what does work?

**[you]**: Data mesh. Each university owns their data products. We provide the platform (storage, compute, catalog infrastructure, governance standards). They produce data products with defined schemas, SLAs, documentation, and contact owners. Researchers discover data through a federated catalog that aggregates metadata from each institution's data products — without Axiom owning the content.

**Kofi Mensah**: That's a significant culture change. Universities are contributing data to a shared resource, not thinking of themselves as "data product owners."

**[you]**: Yes. The technical architecture is the easier part. The governance adoption is the harder part.

---

**Email — [you] → Dr. Yuki Hashimoto, Dr. Amara Diallo, Dr. Sophia Keller**
**Subject**: Data mesh proposal — input requested**

Thank you for your candid feedback at Wednesday's meeting. I'm designing a data governance architecture that addresses the issues you raised. Before I finalize the design, I'd like to ask three questions:

1. If Axiom required each contributing institution to provide a standard metadata package with each dataset (schema, description, update frequency, contact owner, usage license) — would your institution be willing to provide this? Would it be a burden?

2. If a dataset were poorly documented and a researcher couldn't use it — what should happen? Downrank it? Require documentation before it's discoverable? Allow but flag?

3. If Axiom provided a "data steward" role — someone at each institution responsible for keeping their datasets current — would your institution designate someone for this role?

---

### Problem Statement

Axiom Labs' 200-university data platform is a data lake with excellent storage (after ch135) but zero discoverability. 70% of researchers can't find the data they need without contacting the contributing institution directly. A data mesh architecture — domain-owned data products with federated governance and a self-serve catalog — addresses this at the cultural and technical level simultaneously.

---

### Explicit Requirements

1. Data catalog: every dataset on the platform must be discoverable through a search interface with schema, description, provenance, and contact owner
2. Domain ownership: each contributing institution controls their own data product metadata
3. Federated governance: Axiom sets standards (metadata schema, quality minimums) but doesn't own the content
4. Data quality scoring: each dataset has a computed quality score visible to researchers
5. Self-serve publishing: a researcher at University of Dakar can publish a dataset without contacting Axiom support
6. Access control integration: data discovery is separate from data access (you can see a dataset exists without having access to the data)

---

### Hidden Requirements

- **Hint**: Dr. Keller's question: "Can I cite it in a paper?" This is a data *provenance* requirement. Academic publications require persistent, citable data references — ideally DOIs (Digital Object Identifiers). Your data catalog must support DOI registration for published datasets. This is a non-obvious requirement buried in the researcher feedback.

- **Hint**: Dr. Hashimoto's experience with Japanese-language columns with no translation suggests a deeper problem: Axiom's platform has no *metadata standards* requirement. Universities contribute data in whatever format they use internally. A data mesh with federated governance needs a minimum metadata standard without which a dataset is unpublishable on the platform. Define this standard.

---

### Constraints

- 200 contributing institutions, 14 countries, multiple languages
- 12 petabytes of existing data with no metadata catalog
- 4,000 active researchers performing discovery searches
- Data steward capacity: each institution can allocate ~2 hours/week for data governance work
- Platform integration: catalog must work with existing Iceberg tables (from ch135) and federated compute (from ch133)

---

### Your Task

Design Axiom Labs' data mesh governance architecture and catalog platform.

---

### Deliverables

- [ ] Data product interface definition: minimum required fields (schema, description, update frequency, quality SLA, contact, license, provenance DOI)
- [ ] Federated catalog architecture: how 200 institution catalogs are aggregated into a searchable platform catalog
- [ ] Data steward role definition: responsibilities, tooling, time commitment per institution
- [ ] Data quality scoring model: how datasets are scored and when low-quality datasets are flagged vs hidden
- [ ] Self-serve publishing workflow: the path from "I have a dataset" to "it's discoverable in the catalog"
- [ ] DOI integration: how publishable datasets get citable persistent identifiers
- [ ] Adoption strategy: how to migrate 12PB of existing uncatalogued data into the data mesh without requiring 200 institutions to do months of retroactive documentation
- [ ] Tradeoff analysis (minimum 3):
  - Centralized catalog (Axiom owns metadata) vs federated catalog (institutions own metadata) — operational burden vs ownership scalability
  - Apache Atlas vs DataHub vs Amundsen vs custom catalog — build vs buy for academic research context
  - Strict metadata requirements (block publishing without documentation) vs soft requirements (publish but flag incomplete) — adoption rate vs catalog quality
