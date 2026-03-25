---
**Title**: NexaCare — Chapter 74: Advanced Observability — SLOs, SLIs, and Error Budgets
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #observability #slo #sli #error-budget #sre #alerting #toil
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 69, Ch. 109, Ch. 140
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**#incidents — NexaCare**
`Monday 02:17` **pagerduty-bot**: 🔴 P1 ALERT — kafka_consumer_lag > 50000
`02:18` **on-call-engineer (nina.park)**: acknowledging. looking.
`02:19` **pagerduty-bot**: 🔴 P1 ALERT — postgres_connection_pool: idle_connections < 5
`02:20` **pagerduty-bot**: 🟡 P2 ALERT — api_response_time_p99 > 2000ms
`02:21` **pagerduty-bot**: 🔴 P1 ALERT — kafka_consumer_lag > 100000
`02:22` **nina.park**: can someone please tell me which of these is actually user-affecting?
`02:22` **nina.park**: I'm looking at three dashboards. everything is red.
`02:23` **nina.park**: but the status page says green. and no trial sites have reported issues.
`02:26` **nina.park**: resolved the connection pool thing. that was a false positive — idle connections drop at night because load drops.
`02:28` **nina.park**: Kafka lag is a backlog from the nightly batch job. not an incident.
`02:31` **nina.park**: p99 latency spike was one bad query from a trial coordinator in Australia. fixed itself.
`02:33` **nina.park**: false positive false positive false positive.
`02:33` **nina.park**: I've been up for 15 minutes for NOTHING. again.
`02:34` **nina.park**: this is the third time this week. I'm going to burn out.

**#incidents — same week, three alerts later**
`Thursday 03:44` **nina.park**: I'm not acknowledging this one. I don't trust our alerts anymore.
`03:52` **pagerduty-bot**: Escalating to secondary on-call...
`03:52` **you**: ...what's happening
`03:53` **nina.park**: I went back to sleep. every alert this week has been a false positive.
`03:54` **you**: this one is real. the trial data submission API is returning 503 for 3 trial sites in Germany. they've been trying to submit observations for 8 minutes.
`03:55` **nina.park**: oh no
`03:56` **you**: Nina — this isn't your fault. the alert system is broken. I'll fix it. go sleep.

**1:1 Session — Marcus Webb (External Consultant) and You**
*Following week, video call*

**Marcus Webb**: Fatima called me. Told me you have an on-call problem. She was right — she's been right about everything she's called me about for six years. What happened?

**You**: We have 847 alerts configured. Most of them are cause-based — "Kafka lag is high," "connection pool is low." They fire frequently and most of them don't indicate user-affecting problems. The engineers have learned to tune them out. Which means when a real incident fires, nobody trusts it.

**Marcus Webb**: The boy who cried wolf. Classic. What's your error budget policy?

**You**: *(pause)* We don't have one.

**Marcus Webb**: *(silence)*

**You**: We don't have formal SLOs either.

**Marcus Webb**: *(more silence)*

**You**: ...we have uptime monitoring.

**Marcus Webb**: Uptime monitoring tells you if your service is up. SLOs tell you if your *users* are experiencing the service you promised them. Those are different things. Do you know the difference between symptom-based and cause-based alerting?

**You**: Symptom-based: alerts on what users experience (error rate, latency). Cause-based: alerts on what's happening internally (Kafka lag, connection pool size).

**Marcus Webb**: Which type should be a P1 at 3 AM?

**You**: ...symptom-based only.

**Marcus Webb**: Now you understand the problem. Here's your assignment: redesign the alerting system from scratch. Start by defining your SLIs and SLOs. Don't come back to me until you've written down what "good service" means in measurable terms for NexaCare's users.

---

### Problem Statement

NexaCare's alerting system generates ~200 alerts per week, of which 85% are false positives (cause-based alerts that don't indicate user-affecting problems). The on-call team has lost confidence in the alert system, creating alert fatigue that caused a real incident to go unacknowledged for 8 minutes. The system has no formal SLOs, no error budgets, and no error budget policy.

You need to design a complete SLO/SLI framework and rebuild the alerting philosophy from symptom-based principles.

---

### Explicit Requirements

1. Define SLIs and SLOs for all user-facing services (trial data submission, query API, audit trail access)
2. Implement multi-window burn rate alerts (fast burn for P1s, slow burn for P2s)
3. P1 alerts must only fire for symptom-based indicators (user-experiencing degradation)
4. Error budget policies: define what happens when error budget is 50% consumed, 100% consumed
5. Toil reduction: reduce P1 alert volume to < 5/week while maintaining incident detection within 5 minutes
6. Error budgets must be visible to both engineering and the Compliance team (they need to plan FDA audits around error budget windows)

---

### Hidden Requirements

- **Hint**: Re-read Nina's message: "I've been up for 15 minutes for NOTHING. again. Third time this week." Burnout from on-call isn't just a morale problem — HIPAA requires that PHI systems be monitored by qualified staff. If the on-call engineer is tuning out alerts, what's the compliance risk?

- **Hint**: NexaCare serves trial sites in 180 countries across all time zones. "99.9% uptime" as a monthly SLO means 43.8 minutes of downtime per month. But if all 43.8 minutes happen during US business hours, it has zero impact on a trial in Australia. How should your SLO account for geography-specific availability requirements?

---

### Constraints

- 847 configured alerts (currently)
- ~200 alerts/week, ~85% false positive rate
- 3-person on-call rotation (first responder, secondary, tertiary)
- Services: trial_submission_api, patient_data_api, audit_trail_api, event_ingestion_pipeline
- Current uptime: 99.7% (claimed); actual user-affecting availability: not measured
- Compliance requirement: incident response within 15 minutes for patient data access failures (HIPAA)

---

### Your Task

Design the SLO/SLI framework and alerting architecture for NexaCare. Include the SLO definitions, error budget calculations, burn rate alert thresholds, and error budget policy.

---

### Deliverables

- [ ] SLI definitions: For each service, define the SLI (what metric, how measured, what counts as "bad")
- [ ] SLO definitions: Targets for each SLI (availability, latency P99, error rate)
- [ ] Error budget calculation: Given a 99.9% availability SLO, what's the monthly error budget in minutes? In request failures?
- [ ] Burn rate alert design: Multi-window burn rate thresholds for P1 (5% budget in 1 hour) and P2 (10% budget in 6 hours)
- [ ] Alert audit: Classify each of the current 847 alerts as symptom-based or cause-based. Define which cause-based alerts should become dashboards vs alerts.
- [ ] Error budget policy: What happens at 50% budget consumed? 100%? Who decides to halt feature work?
- [ ] Tradeoff analysis (minimum 3):
  - Symptom-based vs cause-based alerting (when are cause-based alerts still useful?)
  - Monthly vs rolling 28-day error budget windows
  - Tight SLOs (99.99%) vs realistic SLOs (99.9%) for NexaCare's infrastructure maturity
