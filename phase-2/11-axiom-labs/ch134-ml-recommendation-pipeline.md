---
**Title**: Axiom Labs — Chapter 134: Drug Target Discovery — ML Pipelines at Research Scale
**Level**: Staff
**Difficulty**: 8
**Tags**: #ml-pipeline #model-versioning #shadow-mode #feature-store #drug-discovery #mlops #model-monitoring
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 55 (LuminaryAI ML), Ch. 57 (LuminaryAI feature stores), Ch. 133 (Axiom privacy)
**Exercise Type**: System Design
---

### Story Context

**Slack — #platform-engineering — Thursday 15:20**

**Dr. Priya Sharma**: I need everyone to stop what they're doing for 5 minutes. Read this.

She pastes a screenshot of an internal Slack message from the research team:

**Dr. Marcus Osei (Computational Drug Discovery)**: URGENT — the drug target model produced anomalous predictions for batch job 4471 processed yesterday. 12 of the 14 compounds it flagged as "high probability targets" for the KRAS inhibitor study have never appeared in any similar model output before. The research team used these predictions to guide their next 3 months of wet lab experiments. We need to know: was this a model error or a genuine discovery?

**[you]**: *(responding in Slack)* Can you reproduce the issue? Run batch 4471 again?

**Dr. Marcus Osei**: The model has been updated since yesterday. The version that ran batch 4471 no longer exists. We can't reproduce it.

**[you]**: There's no version tracking?

**Dr. Marcus Osei**: There's a git repository. But the model weights aren't in git — they're on a GPU server. Whoever trained the model yesterday apparently trained a new version this morning without tagging the old one.

---

**War room — 30 minutes later**

**Dr. Priya Sharma**: Walk me through the severity.

**Dr. Marcus Osei**: Three months of wet lab work is beginning tomorrow based on yesterday's predictions. Each month of wet lab time costs approximately $240,000 in reagents, equipment, and researcher hours. If the model had an error, we're potentially sending researchers down a $720,000 wrong path. If the model made a genuine discovery, we need to understand why, reproduce it, and publish it. Either way, we can't answer the question because we can't reproduce the run.

**Kofi Mensah**: Where's the model currently running?

**Dr. Marcus Osei**: On one GPU server. One GPU. One engineer — Priya Iyer — who built it and has been maintaining it solo for 8 months. She's currently at a conference in Tokyo.

**Dr. Priya Sharma**: *(closing her eyes briefly)* [you] — how long to build a proper ML pipeline?

**[you]**: The foundation — model versioning, reproducibility, basic monitoring — four weeks. Shadow mode deployment, feature store, full MLOps: eight to ten weeks.

**Dr. Priya Sharma**: Four weeks for the foundation. Start tomorrow. And find out what's in batch 4471.

---

**The batch 4471 investigation (conducted over two days):**

Through careful reconstruction of Priya Iyer's git history, training data snapshots, and conda environment exports found in her home directory, you piece together what happened: she had updated the training data with a new batch of genomic sequences from three European biobanks. The new data contained a cluster of sequences from a Scandinavian population cohort that had never been in the training set. The model found a genuine biological signal in that cohort.

The predictions are likely valid. But the research team can't use that finding in a publication because there's no reproducibility trail. The discovery is scientifically valuable and scientifically unreproducible simultaneously.

Dr. Marcus Osei: "This is every researcher's nightmare. A result you can't trust and can't discard."

---

### Problem Statement

Axiom Labs' drug target discovery ML model runs on a single GPU server with no versioning, no reproducibility tracking, and no monitoring. A model update caused predictions that may be a genuine discovery or a model error — but the run cannot be reproduced because the model version no longer exists. Build a production ML pipeline with versioning, shadow mode deployment, and model monitoring.

---

### Explicit Requirements

1. Model versioning: every model version must be uniquely identified, with training data snapshot, hyperparameters, and environment logged
2. Reproducibility: any historical batch must be re-runnable against the exact model version that produced it
3. Shadow mode: new model versions run in parallel with production, outputs compared before promotion
4. Feature store: training features must be version-locked to model versions (same features used in training must be used in inference)
5. Model monitoring: prediction distribution monitoring — alert if output distribution shifts significantly from historical baseline
6. Rollback: any model version can be reverted within 5 minutes

---

### Hidden Requirements

- **Hint**: Re-read the batch 4471 story. The model found a genuine signal in new training data from Scandinavian biobanks. This is a **data drift** event — the training distribution changed, causing the model to find patterns it hadn't found before. Your monitoring system must distinguish between "model error" (bad prediction) and "data distribution change" (legitimate new signal). These require different responses.

- **Hint**: "Reproducibility" in academic research has a specific meaning beyond technical reproducibility — a published finding must be independently verifiable. Your pipeline must be able to export a reproducibility package (model weights, code, data snapshot) that another institution's researcher can use to verify results independently. This is different from internal rollback capability.

---

### Constraints

- Current model: running on 1 GPU (A100, 80GB VRAM), training takes 6-8 hours
- Inference batch size: 5,000-20,000 compound sequences per batch
- Prediction rate: ~40 batch jobs/day across all research teams
- Feature dimensionality: 2,048 features per compound (learned from genomic sequences)
- Regulatory context: GCP Part 11 (FDA) for any findings used in clinical drug development
- Researchers: 4,000 active, 200 submitting ML batch jobs weekly

---

### Your Task

Design the production ML pipeline for Axiom Labs' drug target discovery model.

---

### Deliverables

- [ ] Model versioning design: MLflow vs DVC vs custom registry — selection and implementation
- [ ] Feature store design: offline (training) vs online (inference) feature pipelines, version locking to model versions
- [ ] Shadow mode architecture: traffic splitting between production and candidate models, comparison framework
- [ ] Model monitoring: prediction distribution metrics (mean, variance, percentile shifts), alert thresholds
- [ ] Reproducibility package: what a published research finding must include for independent verification
- [ ] Data drift vs model error detection: how to distinguish a distribution shift (valid new signal) from a model error
- [ ] Rollback procedure: 5-minute rollback process with zero disruption to in-flight batch jobs
- [ ] Tradeoff analysis (minimum 3):
  - MLflow vs DVC vs Weights & Biases for academic ML reproducibility requirements
  - Shadow mode (parallel inference) vs canary (traffic split) for ML model promotion
  - Centralized feature store vs per-model feature snapshots — consistency vs operational overhead
