---
**Title**: ForgeSense — Chapter 204: The Webb Keynote
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #ml-systems #data-quality #feature-stores #predictive-maintenance #observability
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 22 (AgroSense), Ch. 55 (LuminaryAI), Ch. 202, Ch. 203
**Exercise Type**: System Design
---

### Story Context

**Design review — ForgeSense conference room "Bessemer" — Thursday, 9 AM**

The whiteboard is already covered in diagram fragments when you arrive. Svetlana Volkov, Nikolaj Brandt, and two engineers you haven't formally met yet are clustered around one end of the table. Tariq Hussain sits at the head. On the screen is a slide titled: "ForgeMind Predictive Maintenance — Q3 Roadmap."

**Nikolaj**: "Before we start — has everyone seen the Marcus Webb keynote from StratechConf 2023? The one about AgroSense?"

A few people nod. One of the newer engineers, Chidi Obi (ML Engineer, joined 3 months ago), pulls up a laptop.

**Chidi**: "I have the link. Hang on."

The talk is 42 minutes. Nikolaj plays the relevant clip — about 8 minutes, starting around the 22-minute mark.

On screen: Marcus Webb, standing at a podium, white shirt, sleeves rolled up. He looks older than you'd imagined from his Slack messages. His voice is the same, though — flat and precise.

---

*Clip transcript (reconstructed from your notes):*

**Marcus Webb**: "AgroSense. I'm not going to name them, because they're a client, and they know who they are, and they've given me permission to tell this story because they think it might stop someone else from making the same mistake."

"They had fifty thousand sensors deployed across agricultural fields in California and the Central Valley. They built a beautiful ML pipeline — feature engineering, model serving, real-time inference, the works. They spent eight months on the ML layer. They spent three weeks on data quality."

"On the first day of real production, the model started generating alerts. A lot of alerts. Alerts for fields that were fine. Alerts for sensor readings that were impossible — negative soil temperature in July. Alerts for events that happened twelve hours ago, delivered as if they were happening now."

"The model wasn't wrong. The model was doing exactly what it was trained to do. The problem was the data it was operating on. The sensors had a firmware bug that introduced a systematic offset in moisture readings. The edge processing layer had a timestamp bug that delayed delivery by up to twelve hours. The feature engineering pipeline had no data validation — it accepted whatever came in and treated it as ground truth."

[pause]

"The ML team spent six weeks debugging what they thought was a model problem. It wasn't. It was a data problem. By the time they found it, the customer had disabled half the alert rules because 'the model cries wolf.' That reputational damage — operators ignoring alerts — that's the real failure. That's the failure that's hard to undo."

"If you are building a predictive maintenance pipeline — for agriculture, for manufacturing, for anything — here is the order of operations. First: instrument your data quality. Second: build your feature pipeline. Third: train your model. Do not do these in parallel. Do not skip step one because the deadline is close. A model trained on bad data is worse than no model, because it gives you false confidence."

[end clip]

---

**Back in the conference room:**

Silence for a moment. Then Chidi looks up from his laptop. "That's... exactly what our current pipeline does. We have no data quality layer. We pass raw sensor readings straight into the feature extraction job."

Svetlana has the red marker out. She draws a circle around the ML box on the whiteboard and writes: "NO QUALITY GATE."

**Tariq**: "How long have we been running it like this?"

**Chidi**: "Since the first pilot. About fourteen months."

**Tariq**: "And our false positive rate?"

**Chidi**: "Officially, 12%. Informally — if you talk to the operators at the Detroit pilot factory — they say closer to 40%. They've stopped acknowledging about a third of the alerts. They call it 'ghost alarms.'"

Tariq sets down his coffee. "Ghost alarms. On a system that's supposed to prevent $2M/week in downtime."

**You**: "Webb's point was specific: the quality problem needs to be solved before the model problem. What does 'data quality layer' actually mean in an IIoT context?"

**Chidi**: "At minimum: range validation per sensor type, timestamp monotonicity checks, drift detection comparing against recent historical baseline, and sensor health status from the OPC-UA diagnostics. Probably also cross-sensor correlation — if the press is running, the temperature sensor on the press die should be above ambient."

**Svetlana**: "None of that exists today."

**Tariq** [to you]: "You're redesigning this pipeline. Webb's keynote just became your design brief. Data quality first. Do it right."

---

**Slack — DM from you to Marcus Webb — Thursday, 8 PM**

```
You [8:12 PM]
Just watched your AgroSense StratechConf talk.
We have the same problem. Different industry.

Marcus Webb [8:34 PM]
How many ghost alarms?

You [8:35 PM]
40% by operator estimate. 12% by our official metrics.
Classic problem — our metrics don't measure what operators
actually do, just what the model fires.

Marcus Webb [8:37 PM]
The gap between official and informal is always the truth.
Fix the quality layer before you touch the model.
You already know this.
```

---

### Problem Statement

ForgeSense's predictive maintenance pipeline passes raw sensor readings directly into a feature extraction job with no data quality validation layer. The result: a 40% effective false positive rate (by operator estimate), causing operators to disable alert rules and manually ignore alerts — defeating the purpose of the entire system.

Following Marcus Webb's lesson from the AgroSense incident, redesign the predictive maintenance pipeline with data quality as the foundational layer. The data quality layer must run in real-time (cannot add more than 500ms to the prediction path), must be observable (quality metrics must be chartable over time), and must degrade gracefully (a quality check failure should route readings to a quarantine queue, not halt the pipeline).

---

### Explicit Requirements

1. Data quality layer must validate: range bounds, timestamp monotonicity, sensor health status from OPC-UA diagnostics, cross-sensor correlation rules
2. Quality failures must route to a quarantine queue with full context (raw reading, failed rule, factory ID, timestamp)
3. The quality layer must emit metrics: quality pass rate per sensor, quality pass rate per factory, quality degradation alerts
4. Feature extraction must only consume quality-passed readings (no raw sensor bypass)
5. Model serving: sub-500ms end-to-end latency (sensor reading → maintenance prediction delivered)
6. False positive rate < 5% (measured against operator feedback, not just model confidence threshold)
7. Model versioning: support A/B testing between model versions with traffic splitting
8. Feature store: features must be reusable for both real-time inference and offline model training

---

### Hidden Requirements

- **Hint**: Re-read Chidi's list of quality checks: "cross-sensor correlation — if the press is running, the temperature sensor on the press die should be above ambient." This is not a simple threshold check — it requires knowing the machine's current operational state. Where does that state come from, and how does the quality layer access it without creating a circular dependency (quality layer needs machine state → machine state comes from the same pipeline the quality layer guards)?

- **Hint**: Tariq asked about the false positive rate. The official metric is 12%; the informal operator estimate is 40%. What does this gap reveal about the current measurement system? What new observability data needs to be collected to measure "operator-effective false positive rate" — and who provides that data?

- **Hint**: Svetlana circled the "ML box" and wrote "NO QUALITY GATE." But she didn't say anything about model retraining. If the quality layer now filters out 30% of sensor readings (the bad ones), the model needs to be retrained on the quality-filtered dataset. Otherwise the model was trained on data with different quality characteristics than what it will receive in production. What does this mean for the training pipeline?

---

### Constraints

- **Sensor throughput**: 800K msg/sec per factory (Group A + Group B combined)
- **Quality layer latency budget**: < 50ms (sensor reading → quality-passed reading out)
- **Feature extraction latency**: < 200ms (quality-passed reading → feature vector)
- **Model serving latency**: < 250ms (feature vector → prediction)
- **End-to-end latency target**: < 500ms (sensor reading → prediction delivered)
- **False positive SLA**: < 5% (operator-measured, not model confidence threshold)
- **Quarantine queue retention**: 7 days (for debug analysis)
- **Feature store**: must serve both real-time (< 5ms feature lookup) and batch (offline training)
- **Model versions**: support 2 concurrent versions with traffic split (90/10, 80/20, 50/50)
- **Team**: 2 ML engineers, 2 backend engineers
- **Timeline**: data quality layer live in 30 days, full pipeline redesign in 90 days

---

### Your Task

Design the redesigned predictive maintenance pipeline with data quality as the foundation. Your design must show:

1. The three-layer pipeline: data quality → feature engineering → model serving
2. The quarantine queue: what triggers routing, what is stored, how it is used for debugging
3. The feature store: architecture for both real-time feature serving and offline training data export
4. Model versioning: how A/B traffic splitting works in the inference path
5. Observability: what metrics are emitted at each layer, and what does the "operator-effective false positive rate" dashboard look like?

---

### Deliverables

- [ ] Mermaid architecture diagram: full three-layer pipeline (quality → features → model serving) including quarantine path, feature store, and model registry
- [ ] Data quality rule schema (rule definitions stored externally — rule_id, sensor_type, rule_type, parameters, version — with types and indexes)
- [ ] Scaling estimation:
  - Throughput capacity needed for the quality layer at 12 factories
  - Feature store read latency math (how many IOPS needed for sub-5ms feature lookup at 12-factory scale)
  - Quarantine queue storage estimate (7 days retention at 30% failure rate)
  - Show your math
- [ ] Tradeoff analysis:
  - Tradeoff 1: Rule-based quality validation vs. ML-based anomaly detection for the quality layer (accuracy, latency, maintenance burden)
  - Tradeoff 2: Centralized feature store (one store, all factories) vs. per-factory feature store (isolation vs. cross-factory correlation)
  - Tradeoff 3: Online learning (model updates in real-time from operator feedback) vs. periodic batch retraining (simplicity vs. freshness)
- [ ] Cost modeling: monthly cost for the full pipeline at 3 factories and 12 factories (feature store, model serving infrastructure, quality layer compute)
- [ ] Capacity planning: plan for adding 2 new ML models (vibration analysis, thermal imaging) to the same pipeline in 6 months — what changes?
- [ ] Operator feedback loop: describe in 1-2 paragraphs how operator alert acknowledgements (dismiss, confirm, escalate) are captured and fed back into model evaluation metrics

### Diagram Format

All architecture diagrams must use Mermaid syntax.
