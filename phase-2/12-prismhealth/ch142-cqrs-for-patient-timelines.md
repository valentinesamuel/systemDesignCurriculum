---
**Title**: PrismHealth — Chapter 142: A Patient's Complete Vital History
**Level**: Staff
**Difficulty**: 8
**Tags**: #cqrs #event-sourcing #patient-timelines #temporal-queries #clinical-review #hipaa #retention
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 66 (Stratum CQRS), Ch. 112 (Crestline temporal), Ch. 126 (BuildRight CQRS)
**Exercise Type**: System Design
---

### Story Context

**Support Ticket #P-2847 — Priority: High**
**Submitted by**: Dr. Marcus Wei, MD — UCSF Medical Center
**Category**: Clinical Data Request

> I am requesting access to my patient's complete SpO2 timeline for the night of March 3rd — not just the alert events, but the complete raw readings at 30-second intervals from 22:00 to 06:00.
>
> My patient (de-identified as MRN-77329 in your system) was monitoring at home with a continuous pulse oximeter. Your system generated one alert at 02:14 AM for SpO2 84%, which resolved without intervention by 02:19 AM. The episode was treated as a single isolated event.
>
> I am requesting the complete raw data because my review of the limited data available suggests there may have been multiple sub-threshold readings (SpO2 88-90%) in the preceding 3 hours that didn't individually trigger alerts but may represent a pattern requiring clinical review. I need the raw readings to submit to the insurance board for my patient's home oxygen therapy authorization.
>
> PrismHealth support has informed me that raw readings are retained for only 72 hours before deletion. The incident was 11 days ago.

---

**Meeting — Dr. Ngozi Obi, Dr. Amara Chen, Priya Nair, [you] — Monday afternoon**

**Dr. Amara Chen**: Dr. Wei's request is medically appropriate. The pattern he's describing — repeated sub-threshold SpO2 readings in the hours before a threshold breach — is a known precursor to obstructive sleep apnea and is clinically significant for oxygen therapy authorization.

**Dr. Ngozi Obi**: Can we provide the data?

**Priya Nair**: Not for this specific patient. The 72-hour deletion window passed 9 days ago. We have the alert event (SpO2 84% at 02:14) because alert events are retained in the clinical record. We have hourly summary statistics. We don't have the raw 30-second interval readings.

**Dr. Ngozi Obi**: Why do we delete raw readings after 72 hours?

**Priya Nair**: Storage cost. At 14M readings per minute with 30-second interval retention, raw readings for 8M patients would be 800TB/day. At $0.023/GB, that's $17M/year for storage alone.

**Dr. Ngozi Obi**: How much would it cost to keep 7 days?

**Priya Nair**: $120M/year. Not viable.

**Dr. Ngozi Obi**: How much would it cost to keep 30 days of raw data only for patients who've had a clinical event?

**[you]**: That's a different design. Event-triggered retention. If a patient generates an alert event, we extend raw reading retention for that patient for 30 days from the alert. For patients with no alert events, 72-hour deletion stands.

**Dr. Amara Chen**: That would cover the Dr. Wei scenario. The alert on March 3rd would have triggered extended retention, keeping the 3 hours of pre-alert raw readings.

**[you]**: Yes. But there's still a gap for the scenario Dr. Wei actually identified — sub-threshold readings that don't trigger an alert but are clinically meaningful in aggregate. Those would still be deleted because no alert was triggered.

**Dr. Ngozi Obi**: *(quietly)* That's the harder problem.

---

### Problem Statement

PrismHealth deletes raw device readings after 72 hours to control storage costs, but clinical review of complex cases (like insurance authorization, sleep disorder diagnosis) requires access to raw readings from events weeks ago. A CQRS model with tiered retention — raw readings for short windows, event-triggered extended retention, and clinician-requested retention holds — can support clinical needs without requiring indefinite raw data retention.

---

### Explicit Requirements

1. Raw reading retention of ≥ 30 days for patients with any clinical alert event in the preceding 30 days
2. Clinician-requested retention hold: a physician can request a 90-day retention hold for a specific patient
3. Hourly summarized data: retained for 12 months (for trend analysis and insurance review)
4. Alert events: retained indefinitely as part of the clinical record (per CMS requirements for remote patient monitoring)
5. CQRS projections: separate read models for real-time monitoring vs clinical review vs insurance reporting
6. HIPAA minimum necessary: retention of raw data must be justifiable under HIPAA's minimum necessary principle

---

### Hidden Requirements

- **Hint**: CMS (Centers for Medicare & Medicaid Services) requires remote monitoring records to be retained for at least 7 years. PrismHealth's indefinite alert event retention satisfies this. But CMS has also proposed rules that would require raw device data to be available for at least 90 days for Medicare billing audit purposes. If this proposed rule is finalized, the 72-hour raw retention would be non-compliant for Medicare patients. Your design must be extensible to support regulatory retention changes without architecture redesign.

- **Hint**: Re-read Dr. Wei's request: "readings that didn't individually trigger alerts but may represent a pattern requiring clinical review." This is the hardest sub-problem. A reading that's SpO2 88% is not an alert. Three readings at SpO2 88% over 2 hours is clinically significant. Your alert system (ch140) may catch this via trend analysis going forward — but the data was deleted before the trend could be identified retrospectively. The gap is in retrospective analysis, not real-time alerting. What is the right balance between retention cost and retrospective clinical value?

---

### Constraints

- 14M device readings/minute = 800TB/day of raw data at 30-second intervals
- Current retention: 72 hours raw (= ~2.4PB raw data at any time)
- Target: event-triggered 30-day retention for ~3% of patients with alerts = ~$2M/year incremental
- Clinician holds: estimated 0.01% of patients, 90-day hold = manageable volume
- Alert event retention: indefinite (already implemented, negligible storage)
- HIPAA minimum necessary: must justify each retention tier to auditors

---

### Your Task

Design PrismHealth's tiered retention and CQRS architecture for patient vital timelines.

---

### Deliverables

- [ ] Retention policy tiers: raw (72h default) → alert-triggered (30 days) → clinician hold (90 days) → hourly summary (12 months) → alert events (indefinite)
- [ ] CQRS projections: real-time monitoring read model vs retrospective clinical review read model vs insurance/audit read model
- [ ] Event-triggered retention pipeline: when an alert fires, how raw readings for that patient are flagged for extended retention
- [ ] Clinician hold API: how a physician requests a retention hold for a specific patient
- [ ] Storage cost model: each retention tier × volume × cost ($0.023/GB S3 vs Glacier for cold tiers)
- [ ] Regulatory extensibility: how to change the 72-hour default to 90 days (CMS proposed rule) without architectural change
- [ ] HIPAA minimum necessary justification: written justification for each retention tier that satisfies HIPAA auditors
- [ ] Tradeoff analysis (minimum 3):
  - Event-triggered retention vs uniform extended retention — cost vs coverage for edge cases
  - Hourly summaries as permanent record vs raw data as permanent record — query flexibility vs storage cost
  - Retrospective clinical value vs cost of "just keep everything forever" — where the line is for life-safety data
