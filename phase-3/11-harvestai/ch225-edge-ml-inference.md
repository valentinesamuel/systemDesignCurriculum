---
**Title**: HarvestAI — Chapter 225: The Model in the Field
**Level**: Staff
**Difficulty**: 8
**Tags**: #edge-ml #iot #model-serving #ota #bandwidth-constrained
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 205 (ForgeSense SCADA edge processing), Ch. 195 (AutoMesh OTA firmware updates), Ch. 14 (AgroSense IoT sensor pipeline)
**Exercise Type**: System Design
---

### Story Context

**Direct Message — Dr. Aisha Kamara → You, Wednesday 11:17 AM**

*(A voice note transcript appears in the DM thread. Aisha recorded it while walking between her desk and the coffee machine — you can tell from the slightly uneven audio quality she described when she sent it.)*

---

**Dr. Aisha Kamara** [11:17 AM]
Hey — I'm sending you a voice note because this is easier to explain out loud. Transcript below.

---

*[Voice note transcript — 3m 42s]*

"Okay so. The drone imaging pipeline. Currently everything runs in the cloud — a farmer's drone uploads imagery, we run it through our ResNet-50 fine-tuned model, we generate a crop health map, the farmer sees it on their dashboard. That pipeline works fine when you have connectivity. The problem is: farmers don't always have connectivity.

"We started a pilot six months ago with John Deere. They have this thing called the Operations Center, and some of their tractors — the X-series — have an NVIDIA Jetson AGX Orin module on board. Basically a 32-TOPS AI accelerator, 16GB RAM, can run our full inference pipeline on-device. We put our model on 50 tractors for the pilot.

"Here's where it gets complicated. The model is 2.3 gigabytes. When we update it — and we update it roughly weekly because we're constantly fine-tuning on new seasonal data — we need to push the new model to the tractor. The tractor has WiFi connectivity only when it's in the equipment barn, usually once a day in the evening for maybe 2 to 4 hours. Upload bandwidth is consumer-grade — maybe 15 to 20 megabits per second on a good day. Sometimes it's 5 megabits because the router is in the corner of the barn and the tractor is at the back.

"So a 2.3GB model update takes between 15 minutes and 8 minutes depending on signal. That's okay — it fits in the connectivity window. But here's the problem nobody thought about until a farmer called us:

"He ran the tractor all day doing soil sampling. The tractor was making predictions about crop health — nitrogen deficiency, early blight indicators, that kind of thing. He saw the predictions on the in-cab display. He made planting decisions based on them. He came home, tractor connected to WiFi, we pushed a model update. The next morning the model version was different.

"He asked me: 'Is the analysis I did yesterday still valid? The model changed. Was yesterday's model good or bad? Should I redo the analysis?'

"I didn't have a good answer for him. I said the new model is better. He said: 'Better than what? How much better? And what if the field I analyzed yesterday is the one I'm planting tomorrow?'

"That's the staleness problem. We have no way to tell a farmer: 'This analysis was done with model version X, and model version Y is now live, and here is the confidence delta between them.' We just... overwrite.

"The second problem is delta updates. We don't do them. Every week we push the full 2.3GB model. We have 50 tractors now. If we scale to 500 tractors and the model updates weekly, that's 500 × 2.3GB × 52 weeks = 59.8 terabytes per year in model transfer alone. At current WiFi bandwidth that's... a lot.

"The third problem is model corruption. Last month one of the tractors lost power during a model update. The local SQLite database that stores the model metadata got partially written. The model loader tried to load a corrupt model, failed silently, and fell back to... nothing. The tractor showed no predictions for a week. The farmer didn't notice because the UI just showed blank instead of an error. He thought the feature was disabled.

"So: three problems. Staleness disclosure, delta updates, corruption recovery. I need a system that solves all three. I've been thinking about it for weeks and I keep getting stuck on the bandwidth-constrained update strategy. There's a way to do this that looks like how mobile apps do OTA updates but for gigabyte-scale ML models. I just don't know the right architecture.

"Anyway. That's the problem. I'll be in the design meeting Thursday. Bring ideas."

---

**Dr. Aisha Kamara** [11:18 AM]
voice note transcript above — AI transcribed, should be accurate
the three problems are real and we're hitting them now with 50 tractors
at 500 tractors they become critical
at 5,000 tractors they become company-threatening
Priya wants a design by Thursday

---

**#eng-ml — Slack, Wednesday 2:30 PM**

**Carlos Mendes** [2:30 PM]
quick thought on the bandwidth problem
in Brazil the barn WiFi is even worse — 3-5 Mbps at some of the more remote fazendas
a 2.3GB model at 3 Mbps is 100 minutes
that doesn't fit in a 2-4 hour window once you account for other sync operations (telemetry upload, prescription maps download)
we need delta updates or we need model compression or we need both

**Raj Patel** [2:35 PM]
or we need to rethink whether 2.3GB is the right model size for the edge
can we distill it?

**Dr. Aisha Kamara** [2:37 PM]
we tried distillation last quarter
got it to 800MB with 94% of the accuracy on nitrogen deficiency detection
but early blight accuracy dropped to 78% which is below our contractual SLA for premium-tier farms
the 2.3GB model is 2.3GB for a reason

**Raj Patel** [2:38 PM]
what if we split the model?
fast small model on-device for common cases
slow big model in cloud for rare/uncertain cases
and the tractor decides which path to take based on confidence score?

**Dr. Aisha Kamara** [2:40 PM]
that's... actually interesting
I hadn't thought about confidence-gated routing
let me think about accuracy implications

---

### Problem Statement

HarvestAI runs ML inference for crop health analysis (nitrogen deficiency, early blight, moisture stress) using a 2.3GB fine-tuned ResNet-50 model. A pilot program with 50 John Deere tractors deploys this model to NVIDIA Jetson AGX Orin edge modules, enabling offline inference in fields without connectivity. The model is updated weekly with seasonal fine-tuning data. Three critical problems have emerged: (1) farmers have no visibility into model versioning and cannot assess whether analyses made on old model versions are still valid; (2) full model replacement pushes 2.3GB per tractor per week, creating bandwidth bottlenecks at scale; (3) interrupted model updates corrupt the local model store, causing silent failure with no user-visible error.

At 500 tractors (the near-term growth target), these problems multiply: 59.8TB/year in model transfers, daily silent failures across the fleet, and farmers making high-stakes planting decisions on potentially stale predictions without knowing it. The system must scale to 5,000 tractors within 3 years while maintaining prediction trustworthiness and update reliability under severe bandwidth constraints.

---

### Explicit Requirements

1. Model updates must be delivered using binary delta patching — only changed weights are transmitted, not the full 2.3GB model
2. Every prediction shown to a farmer must include the model version that generated it, the age of that model version, and a confidence-staleness indicator if a newer model is available
3. Model updates must be atomic: a tractor must never be in a state where it has a partially-applied model; if an update fails mid-transfer, the previous model must remain operational
4. The system must support confidence-gated routing: predictions with confidence below a threshold should be offloaded to cloud inference when connectivity is available
5. Update scheduling must respect the tractor's connectivity window (typically 2–4 hours/day in barn); updates must be prioritized and scheduled within available bandwidth
6. The cloud system must know the model version running on every tractor at all times; a tractor that has not reported its model version in 48 hours should generate an alert
7. Model updates must be tested and rolled out progressively (canary rollout to 5% of fleet, then 25%, then 100%), with automatic rollback if error rates increase post-update

---

### Hidden Requirements

1. **Hint: re-read Raj's question "can we distill it?"** Dr. Aisha says distillation gets to 800MB but early blight accuracy drops to 78%. What if you do not choose between the full model and the distilled model — what if you deploy both? A small fast model for common disease patterns (nitrogen deficiency: ~60% of cases) and the full model only for cases the small model is uncertain about. What does this architecture look like, and what is the combined effective model transfer size if you can ship the distilled model (800MB) weekly and the full model only when major retraining happens (monthly)?

2. **Hint: re-read the farmer's question: "Was yesterday's model good or bad?"** He is not asking about model accuracy in the abstract — he is asking about a specific field, on a specific day, for a specific decision. This implies that the system needs to store not just the current prediction, but a prediction audit log tied to model version, so a farmer can retroactively assess confidence. Where does this audit log live — on the tractor (local SQLite), in the cloud, or both?

3. **Hint: re-read the silent failure scenario.** The model loader failed silently. This is not just a UX bug — it is an architectural failure. The edge device has no dead-man's switch: no mechanism that says "if I have not made a successful prediction in N hours, emit a health alert." What does a heartbeat/health reporting system for 5,000 edge devices look like, and what is the data volume of that telemetry?

4. **Hint: re-read Carlos's bandwidth numbers for Brazil.** At 3 Mbps with a 2-4 hour window, you have 2.7–5.4GB of bandwidth budget per night. A 2.3GB model fills most of it. But the tractor also needs to upload telemetry (sensor readings, prediction logs, error logs) and download prescription maps. The model update is competing with operational data transfers. What is the priority ordering, and how does the update scheduler negotiate with other data transfer operations for bandwidth?

---

### Constraints

- **Fleet size**: 50 tractors today; 500 in 12 months; 5,000 in 36 months
- **Edge hardware**: NVIDIA Jetson AGX Orin, 32 TOPS, 16GB RAM, 64GB local NVMe storage per unit
- **Model size**: full model 2.3GB; distilled model 800GB (94% accuracy for nitrogen, 78% for early blight)
- **Update frequency**: weekly fine-tuning releases; major retraining quarterly
- **Connectivity window**: 2–4 hours/day in equipment barn (WiFi); bandwidth 3–20 Mbps depending on location
- **Bandwidth budget per night**: 2.7GB (3 Mbps × 2hr) to 36GB (25 Mbps × 4hr) — but shared with telemetry upload and prescription map download
- **Telemetry upload**: ~150MB/day per tractor (sensor readings, predictions, logs)
- **Prescription map download**: ~50MB/day per tractor (planting recommendations, irrigation schedules)
- **Prediction SLA (premium tier)**: < 2 minutes end-to-end, including inference time
- **Inference time on Jetson (full model)**: ~8 seconds per field image; ~25 images per field analysis = ~3.5 minutes total — this already slightly exceeds the SLA, which is why confidence-gated routing to cloud may be needed for some cases
- **Cloud inference latency** (when connectivity available): 12–18 seconds via API call
- **Model update atomic requirement**: rollback to previous model must complete within 30 seconds of failed update detection
- **Fleet management budget**: $0.50/tractor/month at 5,000 tractors = $2,500/month maximum for OTA infrastructure

---

### Your Task

Design the edge ML deployment pipeline for HarvestAI's tractor fleet. The design must cover: delta update generation and delivery, atomic update application with rollback, confidence-gated cloud routing, staleness disclosure to farmers, fleet health monitoring, progressive rollout (canary deployment), and bandwidth scheduling across competing data transfers.

This is a distributed systems problem at the edge: you have 5,000 devices, each with intermittent connectivity, running a critical ML workload, in an environment where failure affects physical-world decisions. Design accordingly.

---

### Deliverables

- [ ] **Mermaid architecture diagram**: full system from cloud model registry → delta generation service → tractor OTA client → local model store → inference engine → farmer dashboard; include the confidence-gated cloud routing path
- [ ] **Database schema**:
  - Cloud side: `model_versions`, `model_deltas`, `tractor_fleet`, `tractor_model_assignments`, `fleet_health_events`
  - Edge side (SQLite on tractor): `local_model_registry`, `prediction_log`, `pending_updates`, `bandwidth_schedule`
- [ ] **Scaling estimation** (show math):
  - Delta size estimate: if ResNet-50 weekly fine-tuning changes ~15% of weights, what is the expected delta size vs full model?
  - Transfer time at P50 bandwidth (8 Mbps): delta vs full model — time to update 500 tractors vs 5,000 tractors
  - Fleet telemetry volume: 5,000 tractors × 150MB/day = X TB/year; what ingestion rate does this require on the cloud side?
  - OTA infrastructure cost at 5,000 tractors: S3 storage for model versions and deltas, CloudFront transfer costs, fleet management service compute
- [ ] **Tradeoff analysis** (minimum 3):
  - Full model replacement vs binary delta patching: implementation complexity vs bandwidth savings vs corruption risk surface
  - Local SQLite audit log vs cloud-only prediction history: privacy, storage, query capability at farmer dashboard layer
  - Confidence-gated routing (hybrid inference) vs full on-device only: accuracy improvement vs latency variance vs connectivity dependency
- [ ] **Atomic update protocol**: write the step-by-step protocol (not code, but precise specification) for how a tractor applies a model update atomically, including the checksum verification, staging area, and rollback procedure
- [ ] **Bandwidth scheduler pseudocode**: TypeScript pseudocode for the nightly bandwidth allocation function that prioritizes critical telemetry upload, schedules model updates, and leaves headroom for prescription map downloads

### Diagram Format
All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
