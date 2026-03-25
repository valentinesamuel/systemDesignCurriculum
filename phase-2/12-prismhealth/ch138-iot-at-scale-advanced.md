---
**Title**: PrismHealth — Chapter 138: 14 Million Device Readings Per Minute
**Level**: Staff
**Difficulty**: 8
**Tags**: #iot #mqtt #device-registry #fan-in-ingestion #deduplication #message-ordering #healthcare
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 106 (Crestline IoT time-series), Ch. 139 (PrismHealth mTLS)
**Exercise Type**: System Design
---

### Story Context

**Architecture Review — Week 2**
**Presenter**: Jake Huang (Staff Engineer)
**Audience**: Priya Nair (CTO), [you], Dr. Amara Chen (Clinical Safety Lead)

Jake is presenting the IoT ingestion pipeline. He's proud of the work — it was built from scratch over 18 months and it shows. The pipeline handles 14 million readings per minute with a p99 latency of 340ms from device to alert evaluation.

**Jake Huang**: Three problems have surfaced in the last 60 days. I want to present them honestly.

He advances to slide 4.

**Jake Huang**: Problem one: out-of-order readings. Our devices are mobile. A patient wearing a pulse oximeter in an elevator loses signal for 4 minutes, then reconnects. The device batches those readings and sends them all at once with their original timestamps. They arrive at the ingestion layer out of chronological order relative to readings from other sources. Our alert evaluation engine processes readings in arrival order, not timestamp order. This means a reading that shows SpO2 dropping to 88% — which happened 3 minutes ago while the patient was in the elevator — might be evaluated after a normal reading taken after the patient got off the elevator. The evaluation engine sees: normal, 88%, normal. The alert logic is looking for sustained low readings, not isolated spikes, so it doesn't fire. But the patient did have a sustained low reading — it's just out of order in the stream.

**Dr. Amara Chen**: How often does this happen?

**Jake Huang**: 0.3% of readings arrive out of order. At 14 million readings per minute, that's 42,000 out-of-order readings per minute.

**Dr. Amara Chen**: Across how many patients?

**Jake Huang**: We estimate 12,000-15,000 patients per day are affected by out-of-order evaluation.

Dr. Ngozi Obi is not in the room. Dr. Amara Chen speaks for her: "That is not acceptable."

**Jake Huang**: Problem two: duplicates. When a device loses network connectivity and retries, it sometimes sends the same reading multiple times. Currently 1.2% of readings are duplicates. They don't cause incorrect alerts — our deduplication is probabilistic and catches most of them — but the 0.1% that slip through inflate our vital sign trend graphs and confuse clinical staff.

**Jake Huang**: Problem three: corrupt readings. Two devices last month sent readings with physically impossible values — one reported a heart rate of 0 (device firmware bug, patient was fine), one reported a SpO2 of 131% (sensor malfunction). Both readings triggered alerts. Both triggered unnecessary nursing interventions. $2,400 each.

**Dr. Amara Chen**: I want to add something to problem three. The alert fatigue impact is not just cost. When nurses respond to false alerts twice in a row, their response time to the next alert — even a real one — increases by 23%. We have clinical studies on this. Every false dispatch degrades the system's reliability for real emergencies.

---

### Problem Statement

PrismHealth's IoT ingestion pipeline processes 14 million device readings per minute but has three quality problems: out-of-order readings affecting 15,000 patients daily cause incorrect alert evaluation; duplicate readings at 1.2% inflate clinical trend graphs; and corrupt readings from malfunctioning devices trigger false clinical alerts costing $2,400 each and increasing nurse response latency. Redesign the ingestion pipeline for clinical data quality.

---

### Explicit Requirements

1. Out-of-order readings must be reordered before alert evaluation (short reorder buffer: max 10 minutes)
2. Duplicate readings must be detected and discarded with ≥ 99.9% accuracy
3. Corrupt readings must be flagged before reaching alert evaluation (physiological plausibility checks)
4. Pipeline latency must not increase: p99 reading-to-alert-evaluation ≤ 400ms (currently 340ms)
5. Device registry must track device health state (last seen, firmware version, signal quality)
6. 10M persistent MQTT connections must be maintained without connection management becoming the bottleneck

---

### Hidden Requirements

- **Hint**: Jake mentioned "probabilistic deduplication catches most duplicates." Probabilistic deduplication (Bloom filters) has a false positive rate — it will occasionally mark a legitimate unique reading as a duplicate and discard it. For a non-healthcare system, this is acceptable. For a patient vital signs system, discarding a legitimate reading could mean missing a clinical event. What is the correct deduplication design for life-safety data?

- **Hint**: Re-read the corrupt reading examples: heart rate 0, SpO2 131%. Both are outside physiological ranges. But what about a reading that's within range but clinically inconsistent — e.g., a patient who was SpO2 99% one minute and SpO2 82% the next, with no previous history of respiratory issues? This could be a genuine deterioration event or a sensor glitch. Your plausibility checking must distinguish between "impossible" values and "suspicious" values with different handling for each.

---

### Constraints

- 10M devices, 14M readings/minute = 233k readings/second
- MQTT broker fleet: currently 40 broker nodes
- Out-of-order arrival window: typically < 4 minutes (elevator/dead zone scenarios)
- Duplicate detection window: must cover within same 5-minute window (device retry window)
- Physiological plausibility ranges: HR 30-300 BPM, SpO2 50-100%, BP systolic 60-250 mmHg
- Clinical alert SLA: reading to alert evaluation ≤ 400ms p99 (current 340ms — 60ms budget for quality controls)

---

### Your Task

Redesign PrismHealth's IoT ingestion pipeline for clinical data quality with out-of-order handling, deterministic deduplication, and physiological validation.

---

### Deliverables

- [ ] Architecture diagram: MQTT broker fleet → ingestion pipeline → quality controls → Kafka → alert evaluation
- [ ] Out-of-order buffer design: sliding window approach, 10-minute buffer, timestamp comparison logic
- [ ] Deduplication design: content hash + device_id + timestamp window; why probabilistic (Bloom filter) is insufficient for life-safety data; deterministic alternative
- [ ] Physiological validation tiers: impossible values (discard + device health flag) vs suspicious values (forward with clinical flag)
- [ ] Device registry schema: device ID, patient mapping, last seen, firmware version, signal quality metrics
- [ ] MQTT connection management: 10M persistent connections across 40 broker nodes — load distribution, reconnection handling
- [ ] Latency budget analysis: current 340ms pipeline, with 60ms budget for quality controls — where each stage fits
- [ ] Tradeoff analysis (minimum 3):
  - Reorder buffer (latency cost) vs accept out-of-order (clinical accuracy cost) — the 60ms budget tradeoff
  - Bloom filter deduplication (fast, probabilistic) vs exact deduplication (slower, deterministic) for life-safety data
  - Strict physiological plausibility (discard borderline readings) vs permissive (forward with flag) — false negative risk in clinical context
