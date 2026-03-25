---
**Title**: Crestline Energy — Chapter 109: Grid Reliability SLOs — When 99.9% Is Not Good Enough
**Level**: Staff
**Difficulty**: 8
**Tags**: #slo #sli #error-budget #reliability #sre #alerting #toil #energy
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 74 (NexaCare SLOs), Ch. 108 (Flink streaming), Ch. 140 (PrismHealth life-safety alerting)
**Exercise Type**: System Design
---

### Story Context

**Monthly Infrastructure Review — Conference Room B**
**Attendees**: Elena Vasquez (CTO), Dr. Marcus Thompson (Head of Data Engineering), [you], Sanjay Mehta (back from paternity leave), Amara Diallo (Platform SRE)

**Elena Vasquez**: *(opening slide)* Our platform SLA, as stated in our enterprise contracts: meter readings processed within 60 seconds of receipt, 99.9% of the time. We are currently hitting 99.7%.

**Sanjay Mehta**: 99.7% is good for a software platform.

**Amara Diallo**: *(quietly)* It's not good for a grid.

**Elena Vasquez**: Correct. Amara, explain.

**Amara Diallo**: 99.7% means 0.3% of meter readings miss the 60-second SLA. At 50M meters, that's 150,000 meters per day with delayed readings. During peak demand periods — typically 6-8 AM and 5-8 PM in European markets — those delayed readings happen in clusters. In a 2-hour window, we could have 80,000 meters unread during a demand spike. Grid operators are making procurement decisions based on incomplete data.

**Sanjay Mehta**: But it's only 0.3%.

**Amara Diallo**: *(showing a slide)* Last Tuesday. 6:47 AM. A cold snap in Germany. Demand spike. 6% of German meters were delayed by more than 60 seconds for 11 minutes. Grid operators over-procured by €340,000 in emergency capacity because they couldn't trust the demand signal.

Silence.

**Elena Vasquez**: *(to you)* The SLO framework we inherited was designed for a 5M meter platform. We've 10x'd the meters and nobody revisited the SLO. I want you to redesign it. Not just the number — the entire framework: what we measure, how we alert, and what we do when we breach.

---

**Slack DM — Amara Diallo → [you] — 3 days later**

**Amara Diallo**: I pulled the alerting stats you asked for. Last 30 days: 847 alerts fired. I went through them with the on-call team.

**[you]**: How many were actionable?

**Amara Diallo**: 57. The other 790 were either transient blips (auto-recovered before anyone could look), duplicates, or alerts for symptoms that have a single common root cause that we don't alert on directly.

**[you]**: Which symptoms have a common root cause?

**Amara Diallo**: Three. Memory pressure alerts fire whenever a database vacuum runs. CPU alerts fire during Kafka rebalancing. Disk I/O alerts fire during Iceberg compaction. We get 6-12 alerts each time any of those three things happen. And they happen every day.

**[you]**: So the on-call team is getting 790 false-start alerts per month?

**Amara Diallo**: We've been silencing things. Which means we're going to miss a real alert soon. If we haven't already.

**[you]**: I need to see the full alert history for the last Tuesday incident. The one with the 11-minute German delay.

**Amara Diallo**: *(sends file)* There were 23 alerts during the incident. 17 of them were causes (CPU on the ingestion tier, memory on the processing tier, Kafka consumer lag). Only 6 were symptoms (elevated p99 latency for meter reading acknowledgment). But the on-call team was looking at cause alerts, not symptom alerts.

**[you]**: Which means they spent 11 minutes debugging causes while the SLA was breaching. If they'd been watching only the symptom alert — latency — they'd have known something was wrong in 90 seconds.

**Amara Diallo**: Yes.

---

### Problem Statement

Crestline Energy's current SLO framework was designed for a 5M-meter platform and fails to provide meaningful reliability signals at 50M meters. The on-call team receives 847 alerts per month, of which only 57 are actionable. During real incidents, cause-based alerts overwhelm symptom-based signals, increasing time-to-detection. Redesign the SLO framework for a grid-critical energy platform.

---

### Explicit Requirements

1. Define SLIs that directly measure user-visible reliability (not infrastructure health)
2. SLOs must be tiered: platform SLO for enterprise contracts vs internal component SLOs
3. Error budget policy: what happens when error budget is 50% consumed? 75%? 100%?
4. Alert volume must reduce from 847/month to ≤ 100/month without reducing coverage
5. Symptom-based alerting: on-call team should see customer-impact signals first
6. Toil quantification: measure and track alert noise as an engineering metric

---

### Hidden Requirements

- **Hint**: Re-read the German demand-spike incident. The 0.3% SLA miss happened during a cluster period — 6% of meters delayed for 11 minutes in one country. Does your SLO catch this? A platform-level 99.7% can hide a severe regional incident. What's the right granularity for SLO measurement — platform, country, or grid zone?

- **Hint**: Amara mentioned "we've been silencing things." This is a critical system safety signal. What does your error budget policy require when on-call teams start silencing alerts?

---

### Constraints

- 50M meters across 12 countries
- Current alert volume: 847/month, 57 actionable (6.7% actionable rate)
- Enterprise SLA: 60-second processing, 99.9% → current actual 99.7%
- On-call team: 4 engineers rotating, ~7 pages/week per engineer currently
- Budget: no new monitoring infrastructure cost (use existing Prometheus + Grafana)
- Regulatory: NERC CIP requires documented incident response procedures for SLA breaches

---

### Your Task

Redesign the SLO framework for Crestline Energy's smart meter platform.

---

### Deliverables

- [ ] SLI definitions: 3-5 user-visible SLIs (what to measure, not how to measure it)
- [ ] SLO hierarchy: platform SLO → country SLO → grid zone SLO (with justification for each tier)
- [ ] Error budget calculation: monthly error budget for 99.9% SLO at 50M meters × 60-second window
- [ ] Error budget policy: burn rate thresholds at 50%, 75%, 100% burn — what actions are triggered?
- [ ] Alert redesign: map current 847 alerts to symptom-based replacements; target ≤ 100/month
- [ ] Toil dashboard: what metrics track alert noise over time?
- [ ] Tradeoff analysis (minimum 3):
  - Symptom-based vs cause-based alerting — when do you need both?
  - Tight SLO (99.99%) vs achievable SLO (99.9%) — the business impact of each choice
  - Error budget as a feature gate vs error budget as a freeze trigger
