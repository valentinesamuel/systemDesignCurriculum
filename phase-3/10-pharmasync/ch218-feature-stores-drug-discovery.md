---
**Title**: PharmaSync — Chapter 218: Feature Stores for Drug Discovery
**Level**: Staff
**Difficulty**: 8
**Tags**: #ml-systems #data-pipeline #storage #database #distributed
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 53 (LuminaryAI feature stores), Ch. 217, Ch. 108 (Axiom Labs genomic data)
**Exercise Type**: System Design
---

### Story Context

**Week 5. Wednesday afternoon. The smaller of the two conference rooms, 3:15pm.**

Dr. Ingrid Walsh has a whiteboard marker in one hand and a printout in the other. She is PharmaSync's head of computational biology and she has the focused energy of someone who has been waiting for this conversation for six months.

"I want to show you something," she says, and draws a horizontal line on the whiteboard. She marks it with five points.

"This is the lifecycle of a feature computation for compound screening. Point one: protein structure prediction — 8 hours on a GPU cluster. Point two: molecular dynamics simulation — 2 hours. Point three: docking score computation against a target protein — 45 minutes. Point four: assay activity prediction — 15 minutes. Point five: ADMET prediction — toxicity, absorption, metabolism — 20 minutes."

She circles point one.

"Our team has recomputed protein structure features for the same 847 compounds eleven times in the last six months. Different models, different team members, different experiments — but the same compounds. Eleven times. At 8 hours each."

She does the math on the whiteboard. "847 compounds × 11 runs × 8 hours = 74,536 GPU-hours. At $3.50/GPU-hour on AWS p3.2xlarge instances, that's $260,875. Wasted."

---

**Slack DM — Dr. Ingrid Walsh → You — Wednesday, 4:01pm**

> **Ingrid**: I sent Tariq a proposal 6 months ago for a feature store. He forwarded it to the previous platform team. Nothing happened. I'm told you're the person who actually builds things.

> **You**: I've worked on feature stores before. What makes drug discovery different from a typical ML feature store?

> **Ingrid**: Several things. First, versioning is not optional. A model trained on AlphaFold2 protein structures gives different predictions than one trained on AlphaFold3 structures. When a regulatory agency asks "what features did you use to train the model that ranked compound GX-441?", you need to know exactly which version of each feature, from which method, with which parameters. This is not a nice-to-have. This is part of the submission package.

> **Ingrid**: Second, features in drug discovery are expensive in an unusual way. It's not expensive to serve them — it's expensive to compute them. The cost asymmetry is the opposite of recommendation systems. In ads, features are cheap to compute and expensive to serve at scale. Here, features are cheap to serve and brutally expensive to compute.

> **Ingrid**: Third, our features span wildly different timescales. Protein structures are computed once per compound and rarely change (unless a new prediction method is released). Assay results come in continuously as experiments are run. Weather and batch conditions for synthesis are logged in real-time. They all need to be accessible together at training time.

> **You**: What about the regulatory angle? Do features fall under 21 CFR Part 11?

> **Ingrid**: Great question. Feature vectors used in models that inform clinical decisions — yes. We've had that debate with Preethi. Her position: any feature that's used to rank compounds that go into a clinical trial must have a traceable audit trail. The FDA may ask: "what data was this AI recommendation based on?"

---

**#data-science — Slack thread**

**Thursday, 10:45am — Dr. Yusuf Al-Farsi [Senior Computational Biologist]**
> @ingrid just ran the ADMET batch for the NEXO series again. Took 6 hours because someone deleted the cached predictions from February. Who has access to the prediction cache?

**10:47am — Ingrid Walsh**
> @platform-team this is the third time this month. We have no governance on computed features. Anyone with S3 access can delete the cache.

**10:50am — Priya Singh [ML Engineer]**
> Also — I just found out that the model Yusuf trained last month was trained on v1 protein structures, but we've since updated to v2 for 200 of those compounds. The model results might be inconsistent. There's no way to know which feature version was used because the feature files don't have version metadata.

**10:52am — Ingrid Walsh**
> This is exactly what I've been saying for 6 months. We need a feature store with immutable versioned storage and access controls.

**11:03am — Tariq Osei**
> @you - can you join the data science team call at 2pm?

---

**2pm team call — Notes (You, Ingrid, Priya, Yusuf, Tariq)**

Key requirements surfaced during the call:

- **Feature types in drug discovery**:
  - **Molecular features**: SMILES strings, Morgan fingerprints, molecular weight, logP, rotatable bonds — computed from chemical structure, essentially static per compound
  - **Structural features**: Protein 3D coordinates from AlphaFold, surface area calculations — 8h compute, version-sensitive
  - **Dynamics features**: MD simulation trajectories, binding free energy estimates — 2h compute, extremely large (GB per compound)
  - **Assay features**: IC50, EC50, selectivity ratios — from lab experiments (see Ch. 217), continuously updated as assays run
  - **Biological context features**: gene expression profiles, pathway enrichment scores — updated as new RNA-seq data arrives

- **The "point-in-time" training problem**: When training a model on a dataset of 10,000 compounds, every feature must come from the same point in time. If compound A's assay result was updated last week, using the new result would introduce label leakage if the model was supposed to predict that result. The feature store must support "give me a snapshot of all features for this compound set as of date X."

- **Storage challenge**: MD simulation trajectories average 4GB per compound. 10M compounds in the discovery library × even 0.1% computed = 40TB of trajectory data. This cannot live in a feature table. It must be referenced by pointer.

**[Hidden clue: Tariq mentions "10M compounds in the discovery library." Even at 0.1% coverage, storage is enormous. The feature store must use tiered storage: hot (recently computed, frequently accessed), warm (older computed features still referenced by active models), cold (archive). The compound coverage will grow — budget for 10x storage growth over 3 years.]**

---

**Slack DM — Priya Singh → You — Thursday, 5:30pm**

> **Priya**: One more thing I forgot to mention in the call. We sometimes need to re-run old experiments with new feature versions to compare model performance. This requires: (1) knowing which compounds were in the original training set, (2) knowing which feature version each compound used, (3) being able to retrieve those exact features. If the old features are gone because we only kept the latest version, we can't reproduce the experiment. We need to keep ALL versions of ALL features for ALL compounds that were ever used in a submitted model.

> **Priya**: "Used in a submitted model" is the key phrase. Preethi has a list of every model we've submitted as part of a drug application. Those features are sacred. Everything else can have a retention policy.

---

### Problem Statement

PharmaSync's computational biology team is recomputing expensive molecular and structural features repeatedly because there is no shared feature store. Feature files have no version metadata, no access controls, and can be deleted by anyone with S3 access. Models trained on features with no version tracking cannot be reproduced for regulatory inspection.

Design a pharmaceutical feature store that handles the unique economics of drug discovery (expensive to compute, cheap to serve), supports bi-temporal point-in-time training snapshots, maintains immutable versioned storage for regulated models, and handles the extreme storage requirements of MD simulation trajectories through tiered storage.

### Explicit Requirements

1. Store and version all feature types: molecular, structural (AlphaFold), dynamics (MD trajectories), assay (from LIMS via Ch. 217), biological context
2. Immutable versioning: once a feature version is written, it cannot be overwritten — only a new version can be added
3. Point-in-time snapshots: "give me all features for compound set X as of date Y"
4. Features used in FDA-submitted models must be retained indefinitely with full audit trail (21 CFR Part 11 — see Ch. 216)
5. Tiered storage for large features: hot (SSD-backed), warm (HDD-backed), cold (S3 Glacier) based on access recency and model registration status
6. Access controls: read/write/delete permissions per feature type and team
7. Feature lineage: track which features were used in which model training run

### Hidden Requirements

- **Hint**: Re-read Ingrid's message — "Any feature that's used to rank compounds that go into a clinical trial must have a traceable audit trail." FDA may ask "what data was this AI recommendation based on?" The feature store must integrate with the Ch. 216 audit system for features that influence clinical decisions.

- **Hint**: Re-read the hidden clue about 10M compounds × 0.1% = 40TB trajectory data. The store must use tiered storage with automatic promotion/demotion policies. Cold features for active models must be promotable to warm on model re-evaluation.

- **Hint**: Re-read Priya Singh's message — "features used in a submitted model are sacred." You need a model registration system that pins specific feature versions. Pinned features cannot be demoted to cold storage or deleted, regardless of access recency.

- **Hint**: Re-read Dr. Al-Farsi's Slack message — "someone deleted the cached predictions." Deletion of computed features must require a two-person approval workflow, with full audit trail. The previous engineer left no access controls at all.

### Constraints

- **Discovery library**: 10M compounds
- **Feature computation costs**: protein structure: 8h GPU ($28/compute), MD simulation: 2h GPU ($7/compute), assay features: continuous from LIMS
- **Storage volumes**: molecular features: ~2KB/compound × 10M = 20GB; structural features: ~50MB/compound × computed fraction; MD trajectories: ~4GB/compound × computed fraction
- **Existing infrastructure**: AWS S3, RDS PostgreSQL, existing GPU cluster (100× p3.2xlarge instances)
- **Team**: Ingrid + Priya + Yusuf + 1 platform engineer (you)
- **Query SLA**: feature retrieval for training batch (10K compounds) < 5 minutes; single compound feature retrieval < 2 seconds

### Your Task

Design the pharmaceutical feature store. Cover the ingestion and computation pipeline, the versioned storage layer (including tiered storage for large features), the point-in-time snapshot mechanism, the model registration and feature pinning system, and the audit trail integration.

### Deliverables

- [ ] Mermaid architecture diagram showing: feature computation jobs, feature ingestion pipeline, tiered storage layers (hot/warm/cold), model registry with feature pinning, audit trail integration, access control layer, and the training snapshot service
- [ ] Database schema:
  - `feature_definitions` table (feature_type, version, computation_method, parameters)
  - `compound_features` table (compound_id, feature_def_id, storage_tier, storage_pointer, computed_at, computed_by_job_id)
  - `model_registrations` table (model_id, training_snapshot_id, status: experimental/submitted)
  - `training_snapshots` table (snapshot_id, compound_set_id, as_of_date, feature_versions_json)
  - `feature_pins` table (compound_id, feature_def_id, pinned_by_model_id, protected_until)
  - Include column types and indexes
- [ ] Scaling estimation:
  - Storage growth: 10M compounds × feature type coverage percentages × size per type = total storage
  - Tiering math: hot/warm/cold distribution at 12 months
  - Cost savings: show computation avoided by caching at current recompute rate
- [ ] Tradeoff analysis (minimum 3):
  - Custom feature store (PharmaSync-built) vs. Feast (open source) vs. Tecton (managed)
  - Tiered storage auto-demotion (access-recency based) vs. manual curation vs. model-registration-based pinning
  - Feature computation as part of feature store pipeline vs. external compute job writing to feature store
- [ ] Cost modeling: monthly cost for storage (S3 tiers), compute (avoided GPU recomputation), and infrastructure. Show ROI vs. current state.
- [ ] Capacity planning: 18-month horizon assuming discovery library doubles to 20M compounds, 5 new FDA-submitted models/year

### Diagram Format

All architecture diagrams in Mermaid syntax. The diagram must show the distinction between "regulated features" (audit trail required) and "experimental features" (standard access controls only).
