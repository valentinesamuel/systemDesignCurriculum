---
**Title**: PrismHealth — Chapter 140: Life-Safety Alerting — When False Positives Kill Trust
**Level**: Staff
**Difficulty**: 9
**Tags**: #observability #alerting #slo #false-positive #life-safety #escalation #healthcare
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 74 (NexaCare SLOs), Ch. 109 (Crestline SLOs), Ch. 142 (PrismHealth CQRS)
**Exercise Type**: System Design
---

### Story Context

**Clinical Review Meeting — Monday 10:00**
**Attendees**: Dr. Ngozi Obi (CEO), Dr. Amara Chen (Clinical Safety Lead), Priya Nair (CTO), [you]

Dr. Amara Chen presents two numbers.

**Dr. Amara Chen**: In the last 30 days: 340 clinical alerts fired that resulted in unnecessary nursing interventions. And 3 alerts that — based on retrospective chart review — should have fired and did not fire in time.

**Dr. Ngozi Obi**: I want to focus on the three.

**Dr. Amara Chen**: Patient A: continuous glucose monitor showed values trending toward hypoglycemia over 47 minutes. Our alert engine requires a threshold breach, not a trend. The threshold was never breached — the patient corrected before reaching it. But the trend warranted a check-in call. We have no trend-based alert for glucose.

Patient B: fall detection device showed 3 minor fall events over 4 hours that were below the alert threshold individually. Our system evaluates each event independently. A pattern of 3 falls in 4 hours is clinically significant — it suggests an acute neurological event. We have no cumulative pattern alert.

Patient C: heart rate data gap. The monitoring device went silent for 22 minutes. The alert for "device offline" fired — but it was routed to the technical team, not the clinical team. The clinical team didn't know. The device came back online. The patient was fine. But 22 minutes of a cardiac patient's device being silent — without clinical review — is a safety gap.

**Dr. Ngozi Obi**: And the 340 unnecessary interventions?

**Dr. Amara Chen**: I have a letter from Massachusetts General Hospital's VP of Nursing. The false positive rate for PrismHealth alerts at MGH is 23% this month. She has instructed her nursing staff to apply a "clinical judgment filter" — which is a polite way of saying they're starting to ignore our alerts.

**Dr. Ngozi Obi**: *(to you and Priya)* A nursing team that ignores our alerts is more dangerous than no alerts at all. You have 90 days to get the false positive rate below 5% at MGH. Without compromising our ability to catch the three cases Dr. Amara just described.

**[you]**: Those two requirements are in direct tension. Lower false positive rate typically means raising thresholds. Raising thresholds typically means missing more real events.

**Dr. Ngozi Obi**: I know. That's why I'm asking you, not the data team.

---

**Analysis — two days later**

You spend two days with Dr. Amara Chen reviewing 30 days of alert data. Findings:

1. **Threshold-only alerting** misses trend-based and pattern-based deterioration (patients A and B from the review)
2. **Device health alerts** route to engineering, not clinical teams — creating a blind spot (patient C)
3. **Alert suppression** is occurring informally: nurses are manually overriding alerts without logging, making it invisible to the analytics system
4. **No confidence scoring**: every alert fires with equal urgency regardless of the supporting evidence quality
5. **No escalation path**: if a nurse acknowledges an alert but doesn't act, nothing happens. The alert is considered "handled."

The alert model needs to be rebuilt from first principles. Not tweaked — rebuilt.

---

**DM — Priya Nair → [you]**

**Priya Nair**: I've been thinking about the SLO framework question. We have a 99.9% platform uptime SLO. But what does "platform uptime" mean for a monitoring system? If the platform is up but the alert quality is poor, the SLO is misleading.

**[you]**: The right SLO is clinical: "percentage of clinically significant events that trigger appropriate clinical review within the target response time." Everything else is a proxy metric.

**Priya Nair**: And the false positive rate should be a counter-SLO — an upper bound. "No more than X% of fired alerts should result in unnecessary interventions."

**[you]**: Yes. And those two SLOs will be in tension at the margins. That tension is intentional — it forces us to get the algorithm right, not just tune thresholds.

---

### Problem Statement

PrismHealth's clinical alerting system has a 23% false positive rate at MGH (threatening nursing team trust) while missing 3 clinically significant events in the last month (trend-based deterioration, cumulative fall patterns, device-offline gaps). The system needs to be redesigned with multi-layer alert logic, confidence scoring, proper escalation, and clinical SLOs.

---

### Explicit Requirements

1. False positive rate: ≤ 5% at any hospital system within 90 days
2. True negative rate (missed events): ≤ 0.1% of clinically significant events
3. Alert confidence scoring: each alert must carry an evidence score (threshold, trend, pattern, or combination)
4. Device offline events must route to clinical team (not just engineering) when the patient is actively monitored
5. Escalation tree: unacknowledged alert → nurse → charge nurse → physician → emergency response (with timing)
6. Alert suppression visibility: all informal overrides must be logged and reviewable

---

### Hidden Requirements

- **Hint**: Dr. Amara's case patient B (3 falls in 4 hours, each below threshold) is a **temporal correlation** problem — no single event triggers an alert, but the pattern does. Your alert engine must support multi-event temporal patterns, not just single-threshold evaluation. This requires stateful evaluation across time, not stateless threshold checking.

- **Hint**: Dr. Ngozi Obi's core concern is that nursing teams ignoring alerts is more dangerous than no alerts. This is the **alarm fatigue** problem studied extensively in clinical literature. The clinical literature quantifies it: after 3 consecutive false alarms from the same device, nurse response time to the 4th alarm (even if real) increases by 40%. Your SLO must track and surface per-device false alarm sequences.

---

### Constraints

- 8M patients, 10M devices
- Alert volume: ~450 alerts/day currently (340 unnecessary + 110 legitimate)
- Target: ≤ 60 alerts/day with false positive rate ≤ 5%
- Clinical review SLA: life-threatening alerts must reach clinical staff within 90 seconds
- Escalation design: nurse acknowledgment expected within 5 minutes; escalation to physician within 10 minutes if unacknowledged
- HIPAA: all alert events, acknowledgments, and escalations are PHI and must be retained 6 years

---

### Your Task

Redesign PrismHealth's clinical alerting architecture for low false positive rate and high sensitivity simultaneously.

---

### Deliverables

- [ ] Multi-layer alert model: threshold-based (immediate) → trend-based (sliding window) → pattern-based (temporal correlation)
- [ ] Alert confidence scoring: how evidence type and strength determine the confidence score
- [ ] Escalation tree design: timing and routing for each tier (device alert → nurse → charge nurse → physician)
- [ ] Device offline routing: clinical team notification when a monitored patient's device goes silent
- [ ] Alert suppression logging: how informal overrides are captured and surfaced to clinical leadership
- [ ] Clinical SLO definition: "meaningful alert rate" SLO + false positive rate counter-SLO
- [ ] Per-device alarm fatigue tracking: metric for consecutive false alarms per device, alert when fatigue threshold exceeded
- [ ] Tradeoff analysis (minimum 3):
  - Threshold-based vs trend-based alert sensitivity — specificity tradeoff
  - Strict false positive SLO (≤ 5%) vs sensitivity SLO (≤ 0.1% missed events) — what happens when they conflict?
  - Automated clinical alert routing vs physician review of all high-confidence alerts — false positive cost vs oversight burden
