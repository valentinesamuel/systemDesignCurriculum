---
**Title**: MindScale — Chapter 253: The Precision-Recall Dilemma
**Level**: Staff
**Difficulty**: 9
**Tags**: #ml-systems #crisis-detection #mental-health #precision-recall #human-in-the-loop #system-design
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 252 (MindScale most sensitive data), Ch. 255 (notification system final evolution)
**Exercise Type**: System Design
---

### Story Context

**1:1 Transcript — Priscilla Tang (Head of Data Science) and you**
_MindScale Conference Room C, Monday 2:00 PM_
_Week 2 at MindScale_

---

**Priscilla Tang:** Close the door. I want to walk you through the crisis model before you start building anything. Because I need you to understand what we're actually dealing with before you touch the infrastructure.

**you:** Of course. Go ahead.

**Priscilla Tang:** (opens laptop) The model is an LSTM — Long Short-Term Memory network. We trained it on 18 months of de-identified therapy session notes, medication adherence records, sleep pattern data from wearables, and social media signals from patients who gave consent. 140,000 training examples. We define a "crisis event" as a patient contacting a crisis line, being admitted to an emergency department for psychiatric reasons, or a therapist filing a welfare concern note within 72 hours of a session.

**you:** What's the current performance?

**Priscilla Tang:** 78% precision. 65% recall. I'll explain what that means in human terms, not just numbers.

**you:** Please.

**Priscilla Tang:** At 78% precision: if the model fires an alert, there's a 22% chance the patient is not actually in crisis. We call those false positives. Right now, out of 4,000 alerts per day, 880 are false positives. Those 880 alerts go to counselors who investigate them, find nothing wrong, and then have to document why they dismissed the alert. That documentation takes an average of 7 minutes per alert. That's 103 hours of counselor time per day — wasted. Across our network, we pay crisis counselors $28/hour. That's $2,884 per day. Per day.

**you:** And 65% recall means —

**Priscilla Tang:** 35% of real crisis events are missed. The model doesn't fire. Nobody is alerted. (pause) Last month, there were two missed crises. We know about them because they showed up in incident reports afterward. One patient self-harmed. One patient was hospitalized. Both had behavioral signals in the 72 hours prior that, in retrospect, the model should have caught. (she closes her laptop) I've been staring at those two incidents for three weeks.

**you:** What happens if you lower the threshold to improve recall?

**Priscilla Tang:** I modeled it. If I lower the decision threshold from 0.62 to 0.45, recall goes from 65% to 81%. We'd catch more crises. But precision drops from 78% to 54%. That means false positives nearly double — from 880 per day to roughly 1,820 per day. We'd be drowning counselors. And here's what the research shows about alert fatigue: when clinicians receive too many false positives, they start dismissing alerts faster. They develop a pattern of "probably nothing." And then they miss a real one.

**you:** So lowering the threshold to save more patients might paradoxically lead to more missed crises.

**Priscilla Tang:** Exactly. It's not a threshold problem. It's a system design problem. The model is one component. The human attention system is another component. And right now, we're treating the model output as the final word. The model fires → counselor is alerted → counselor either investigates or ignores. That's a one-step pipeline. It needs to be a multi-step pipeline.

**you:** What does a multi-step pipeline look like in your mind?

**Priscilla Tang:** I don't know. That's why I'm talking to you. I'm a data scientist. I can tune the model. I can retrain it. I can give you confidence intervals on every prediction. But I can't design the system around the human beings who receive the alerts. That's systems architecture and organizational design, and I've reached the edge of what I know.

**you:** (writing on notepad) Let me think out loud for a minute. The core insight you've given me is: human attention is a finite, exhaustible resource. Right now, the model is the only filter. If the model is uncertain — say, a score of 0.55, well above threshold but not high-confidence — it gets the same alert as a score of 0.95. Counselors can't tell the difference.

**Priscilla Tang:** Correct. The alert they receive just says "CRISIS ALERT — Patient [ID]." No confidence score. No explanation of which signals triggered it.

**you:** So we're asking counselors to treat a 55%-confident prediction with the same urgency as a 95%-confident prediction. And we're surprised they're burning out.

**Priscilla Tang:** (quietly) I wrote the alert system in 2021 when we had 200 patients. It made sense then. We have 2 million now.

**you:** Here's what I'm thinking. We need to separate the model's job from the human's job. The model's job is to detect anomalies and score them. The human's job is to make a clinical judgment on the highest-confidence, highest-urgency cases. But between those two, we need a triage layer. And above that, an escalation layer for when a counselor doesn't respond within a time window.

**Priscilla Tang:** A pipeline.

**you:** A state machine, really. Each alert starts in a "Detected" state. Depending on the model score and the patient's risk profile, it routes differently. High confidence goes directly to a counselor with full context — not just "alert" but "here's what the model saw, here are the last 3 session notes, here's the medication adherence graph." Medium confidence goes to a softer review queue. Low confidence — which right now is being filtered out at threshold — maybe it goes to a weekly risk review, not an immediate page.

**Priscilla Tang:** And the escalation?

**you:** If a counselor receives a high-confidence alert and doesn't acknowledge within 15 minutes, it escalates to their supervisor. If the supervisor doesn't respond in 15 more minutes, it escalates to the on-call clinical director. And if the patient has previously consented to emergency contact, there's a final escalation to emergency services.

**Priscilla Tang:** You're designing a human escalation system on top of an ML detection system.

**you:** I'm designing a system where the model handles scale and the humans handle judgment. The model can process 2 million patients. The humans can handle the 50 highest-confidence alerts per day — not 4,000.

**Priscilla Tang:** (staring at her coffee) If we could reduce counselor alerts to 200 per day with 90% precision, and use a secondary queue for the next 500 medium-confidence cases... the counselors would actually read the alerts.

**you:** And if they're reading them, they're more likely to catch the edge cases.

**Priscilla Tang:** There's a clinical term for what you're describing. "Collaborative decision support." The model provides decision support. The human makes the decision. The system ensures the human is never overwhelmed to the point of defaulting to dismissal.

**you:** (nods) We need Dr. Wells in this conversation. Because the routing logic — who gets notified about what — has 42 CFR Part 2 implications. A patient in a substance abuse treatment program may have a crisis, but we can't route that alert to their primary care physician without specific written consent. The escalation path has to be consent-aware.

**Priscilla Tang:** I'll get us 30 minutes with Samara tomorrow.

**you:** One last question. The two missed crises last month — you said the behavioral signals were there in retrospect. Do you know why the model missed them?

**Priscilla Tang:** (long pause) One was a patient who had been stable for four months. The model had very high confidence they were not in crisis. It had learned that stability. The crisis came out of nowhere — an acute life event. The model wasn't trained on acute life events because we don't have ground truth labels for them. The other... the patient's therapist had marked the session notes as "protected" under 42 CFR Part 2. The model never saw those notes. It was flying blind.

**you:** So the model's blind spot is exactly the patients we're most legally obligated to protect.

**Priscilla Tang:** Yes. (quietly) That's the problem I haven't been able to solve.

---

### Problem Statement

MindScale's crisis detection LSTM achieves 78% precision and 65% recall — meaning 22% of alerts are false positives (burning out counselors) and 35% of real crises are missed (a patient safety failure). At 2 million patients, the system generates 4,000 alerts per day, 880 of which are false positives. Counselor alert fatigue is causing real crises to be missed: counselors default to dismissing alerts because they've been conditioned to expect mostly noise.

Design the crisis detection pipeline as a human-in-the-loop system: an architecture that uses the ML model's confidence scores, patient risk profiles, and clinical context to route alerts intelligently — delivering the right alert to the right human at the right urgency level — while staying within the bounds of 42 CFR Part 2 consent requirements and preventing alert fatigue.

### Explicit Requirements

1. **Tiered alert routing**: Route alerts based on model confidence score bands (e.g., high ≥ 0.85, medium 0.65–0.85, low 0.45–0.65) to different human queues with different urgency levels
2. **Escalation state machine**: High-confidence alerts that go unacknowledged must escalate through a defined chain: counselor → supervisor → clinical director → emergency services (if consent given)
3. **Acknowledgment SLA**: High-confidence alerts must be acknowledged within 15 minutes; medium-confidence within 60 minutes
4. **Clinical context in alerts**: Alerts must include model confidence score, which behavioral signals triggered the alert, the last 3 session notes summary, and the patient's medication adherence trend — not just a patient ID
5. **Daily alert volume target**: Design the system to deliver ≤ 200 immediate-action alerts per day per counselor (currently 4,000 system-wide → distribute across counselor pool)
6. **Consent-aware routing**: Alerts for 42 CFR Part 2 patients (substance abuse records) may only be routed to providers with active, specific written consent for that routing path
7. **False positive feedback loop**: Counselors who dismiss an alert must select a dismissal reason; this feedback is used to retrain the model monthly
8. **Audit trail**: Every alert generated, routed, received, acknowledged, dismissed, and escalated must be logged with immutable timestamp and actor ID
9. **Model explainability in UI**: The counselor alert UI must show which specific signals contributed to the score (not just the score itself)
10. **Quiet hours handling**: Non-emergency alerts (medium/low confidence) must not be pushed to counselors outside their working hours; they enter a morning review queue

### Hidden Requirements

1. **Hint**: Priscilla mentions "one patient's therapist had marked the session notes as 'protected' under 42 CFR Part 2 — the model never saw those notes." This means the model has a systematic blind spot: the highest-risk, most legally protected patients may be invisible to the detection system. The architecture must account for a class of patients where the ML model cannot receive full signal. What is the alternative detection pathway for these patients? Is there a non-NLP signal (behavioral patterns, appointment gaps, medication adherence) that doesn't require note access?

2. **Hint**: Priscilla says "the acute life event — the model wasn't trained on acute life events because we don't have ground truth labels for them." This is a training data problem that has architectural implications. If therapists can flag session notes with a tag like "acute life event disclosed" at the time of documentation, that creates future training labels AND is an early signal that the real-time system could use immediately (before the LSTM processes anything). Is there a fast-path alert trigger for therapist-tagged events that bypasses the ML model latency entirely?

3. **Hint**: Re-read: "103 hours of counselor time per day — wasted." The false positive cost is not just emotional but financial ($2,884/day). The architecture you design has a direct, measurable ROI. You should include a before/after cost model in your design: at the target precision of 90%, how many false positive hours are saved? What is the annual dollar value of that improvement? This justifies engineering investment to the CFO (Nathan Cole's domain).

### Constraints

- **Patient population**: 2M patients across 500 clinics
- **Current alert volume**: 4,000 crisis alerts/day; 880 false positives/day
- **Crisis counselor pool**: ~1,200 counselors system-wide; typical counselor handles 30-40 patients
- **Alert SLA**: High-confidence alerts: 15-minute acknowledgment; escalation if missed
- **LSTM inference latency**: Current model inference: ~400ms per patient batch; runs every 15 minutes on updated signals
- **Inference compute**: GPU cluster (4× A10G); cost ~$1,200/month; runs ~96 inference jobs/day
- **42 CFR Part 2 patients**: approximately 340,000 of 2M patients (~17%) — substance abuse treatment records
- **HIPAA audit retention**: 6 years minimum for all access and alert logs
- **Available signals**: session notes (NLP), medication adherence (pharmacy API), sleep data (wearable consent), social media sentiment (explicit consent), appointment attendance/cancellation history
- **Budget**: $50K/year incremental for infrastructure changes
- **Team**: you + Yusuf Adeyemi (Staff Eng) + 2 backend engineers; 8-week implementation window
- **Database**: PostgreSQL (patient records, audit logs), Redis (alert state machine, session data), Kafka (event streaming)

### Your Task

Design the crisis detection pipeline as a human-in-the-loop system. This includes:

1. The alert triage architecture — how model confidence scores are translated into routing decisions
2. The escalation state machine — states, transitions, time SLAs, and actors at each level
3. The consent-aware routing logic for 42 CFR Part 2 patients
4. The alternative detection pathway for patients whose notes are fully protected
5. The feedback loop from counselor dismissals to model retraining
6. The before/after cost model — quantify the ROI of this system design

### Deliverables

- [ ] **Mermaid architecture diagram**: Full crisis detection pipeline from signal ingestion → LSTM inference → triage routing → counselor alert UI → escalation → audit log
- [ ] **Escalation state machine diagram** (Mermaid): States — Detected, Routed, Delivered, Acknowledged, Dismissed, Escalated, Resolved, Missed; transitions with time SLAs
- [ ] **Alert routing decision table**: Input: (confidence score, patient risk tier, consent flags, counselor availability) → Output: routing path, urgency level, escalation chain
- [ ] **42 CFR Part 2 routing logic**: Pseudocode or decision tree for consent-aware alert routing
- [ ] **Database schema**: Alert table, escalation log, dismissal feedback, consent routing permissions (with column types and indexes)
- [ ] **Scaling estimation**: Show math for alert volume per day, state machine event throughput, audit log storage per year
- [ ] **Tradeoff analysis**: Minimum 3 explicit tradeoffs (threshold tuning vs. routing tiers, model explainability vs. latency, alert richness vs. counselor cognitive load)
- [ ] **ROI cost model**: Before/after cost comparison at target 90% precision

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

---

#### Scaling Estimation (Starter Math — Complete This)

**Alert generation:**
- 2M patients × LSTM inference every 15 minutes = 2M inference events / 15 min = 2,222 patient evaluations/second
- But inference is batched: 2M patients processed in batches of 1,000 → 2,000 batch jobs per inference run
- Each batch: 400ms → 2,000 batches × 400ms = 800 seconds = ~13 minutes for full population sweep
- With 15-minute inference cycle, this is tight but feasible on 4× A10G GPU
- Add 30% growth buffer: at 2.6M patients, inference cycle takes ~17 minutes → need to either add GPU or reduce inference frequency for low-risk patients

**Alert routing throughput:**
- 4,000 alerts/day = 167 alerts/hour = 2.8 alerts/minute = trivial for any queue system
- State machine events (detection + routing + delivery + ack + escalation + resolution): 6 events per alert × 4,000 alerts/day = 24,000 state machine events/day = 0.28 events/second
- Kafka topic for alert events: single partition, trivial throughput

**Audit log storage:**
- Each audit event: ~600 bytes (alert_id, patient_id_hash, counselor_id, action, timestamp, signal_snapshot_ref)
- 24,000 events/day × 600 bytes = 14.4 MB/day
- 6-year HIPAA retention: 14.4 MB × 365 × 6 = 31.5 GB — fits in PostgreSQL with partitioning by month
- Note: patient_id must be stored as a one-way hash in logs where the log viewer doesn't need to look up the patient — minimize PII in logs

**Counselor alert load target:**
- Goal: ≤ 200 immediate alerts/day across 1,200 counselors
- Average: 0.17 high-confidence alerts/counselor/day
- But counselors are assigned to specific patients: each counselor handles ~40 patients
- At target model precision of 90%: 200 alerts/day → 180 true crises caught, 20 false positives
- Current: 4,000 alerts/day → 3,120 true cases + 880 false positives (but 35% of crises missed)
- Transition: we are NOT reducing crisis detection — we're tiering. The 200 high-confidence alerts + 1,000 medium-confidence daily reviews still cover the same crisis population; we're changing urgency level, not total volume

---

#### Threshold Band Calibration (Design Prompt)

Priscilla has given you the model's operating curve data:

| Decision Threshold | Precision | Recall | Alerts/Day | False Positives/Day |
|---|---|---|---|---|
| 0.85 | 94% | 41% | 1,060 | 64 |
| 0.75 | 88% | 55% | 2,220 | 267 |
| 0.65 | 82% | 62% | 3,120 | 562 |
| 0.62 (current) | 78% | 65% | 4,000 | 880 |
| 0.50 | 68% | 74% | 5,600 | 1,792 |
| 0.45 | 54% | 81% | 7,440 | 3,422 |

Design a tiered alert system using this data. Your goal: maximize recall (catch as many real crises as possible) while keeping high-urgency counselor alerts ≤ 200/day/1,200 counselors (system-wide immediate-action ≤ 240,000 — wait, re-do this math).

Actually: ≤ 200 immediate alerts SYSTEM-WIDE per hour during business hours, with medium-confidence cases entering a 60-minute review queue rather than a real-time page.

What threshold bands do you choose for Tier 1 (immediate), Tier 2 (60-min review), Tier 3 (daily review)? What is the total system-wide recall across all tiers?
