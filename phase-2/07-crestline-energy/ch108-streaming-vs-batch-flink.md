---
**Title**: Crestline Energy — Chapter 108: The 18-Hour Batch Job
**Level**: Staff
**Difficulty**: 9
**Tags**: #flink #streaming #batch-processing #watermarks #late-arriving-events #spike-inherited-disaster #apache-flink
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 106, Ch. 107, Ch. 65 (Lamport timestamps / event ordering)
**Exercise Type**: Spike — Inherited Disaster
---

### Story Context

**#incidents — Crestline Engineering Slack**
**Wednesday 06:43**

**pagerduty-bot**: 🔴 ALERT: `nightly-demand-forecast-job` — status: FAILED after 12h 7m — OOMKilled — node: forecast-worker-07

**06:44**
**[you]**: I'm on. What failed?

**06:45**
**on-call-bot**: `forecast-worker-07` killed by OOM at 06:43. Job had been running since 18:30 yesterday. No partial output written. All work lost.

**06:46**
**elena.vasquez**: Morning. What's the blast radius?

**06:47**
**[you]**: Checking now. What does the demand forecast job output?

**06:48**
**elena.vasquez**: It calculates peak demand forecasts for all 50M meters across 12 countries. Grid operators use it to manage power procurement contracts. Contracts activate at 09:00. If there are no forecasts, grid ops can't bid in the morning markets.

**06:48**
**[you]**: We have 73 minutes.

**06:49**
**elena.vasquez**: Yes.

---

**DM from Dr. Marcus Thompson — 06:51**

**Dr. Marcus Thompson**: I woke up to the page. Sanjay is on paternity leave. I have his Confluence page for the job. I'm sending it.

**[you]**: How up to date is it?

**Dr. Marcus Thompson**: It was written in 2021. The current job uses a different Spark version and different S3 paths. Some of it is wrong.

**[you]**: Send it anyway.

The Confluence page arrives. The job runs on Apache Spark. It reads 4.8B rows from S3 (the previous 30 days of meter readings), joins against a 50M-row meter metadata table, and runs a demand estimation model per meter per 15-minute interval for the next 48 hours. The OOM is in the join phase — the broadcast join of the meter metadata table is exhausting heap space.

**06:58**
**[you]**: I can't fix the OOM in 60 minutes. But I can run a scoped version — last 7 days of data instead of 30, and only the top 200 grid zones by volume. That's 80% of procurement contract coverage. Grid ops can trade on that.

**elena.vasquez**: Do it. I'll call grid ops and explain the scope limitation.

---

**09:15 — Post-triage**

The scoped job ran in 47 minutes. Grid ops accepted the limited forecast. The morning market bids went out on time with a noted caveat. No procurement breach.

Now Elena Vasquez is standing at your desk.

**Elena Vasquez**: I want to understand how we got here. An 18-hour job that OOMs and loses all work. No partial output. No incremental checkpointing. Single point of failure.

**[you]**: It was designed for 5M meters in 2019. It's running against 50M in 2024. Nobody migrated the design when the data grew 10x.

**Elena Vasquez**: When can you have a replacement ready?

**[you]**: A streaming replacement using Apache Flink — where forecasts are computed continuously as readings arrive, not in a nightly batch — would give us real-time forecasts available throughout the day. 8 weeks to migrate.

**Elena Vasquez**: What happens if a meter sends a reading 4 hours late? Our meters in rural Norway sometimes buffer for hours.

**[you]**: That's why we need watermarks. It's the same problem as event ordering — we need to define how long to wait for late events before computing a window.

She nods. "Design it. And make it so we never have a 73-minute crisis window again."

---

### Problem Statement

Crestline Energy's nightly demand forecast pipeline is a brittle 18-hour Spark batch job that OOMs on the current data volume, loses all work on failure, and produces forecasts only once per day. Grid operators need demand forecasts continuously updated throughout the day, with late-arriving meter readings (up to 4 hours delayed from rural meters) handled correctly.

---

### Explicit Requirements

1. Replace nightly batch job with continuous streaming forecast updates
2. Forecasts for any grid zone must be available within 15 minutes of new meter readings arriving
3. Handle late-arriving events: meters may send readings up to 4 hours delayed
4. No data loss on failure: checkpointing every 5 minutes maximum
5. Run-time cost ≤ current batch job cost ($2,800/run)
6. Grid operators must see forecast confidence intervals alongside point estimates (late data affects confidence)

---

### Hidden Requirements

- **Hint**: Re-read Elena's question about rural Norway meters. If a meter in rural Norway is offline for 4 hours and sends 16 readings at once — all with their original timestamps — your streaming system must decide: do you recompute the window that those readings belong to? This is the "late data" watermark problem, and the answer determines your SLA.

- **Hint**: The Confluence page mentions the job uses a "broadcast join" for meter metadata — that's why it OOMs at 50M rows. The streaming replacement has the same join requirement. What's the streaming equivalent of a broadcast join, and how does Flink handle it?

---

### Constraints

- 50M meters × 1 reading per 15 minutes = 3.3M readings/minute = 55k readings/second
- Late arrivals: up to 4 hours, primarily from rural meters (~5% of fleet)
- Forecast window: 48-hour forward horizon, 15-minute intervals per meter = 192 forecasts per meter per run
- Processing SLA: grid zone forecast available within 15 minutes of real-time readings
- Checkpoint interval: ≤ 5 minutes
- Infrastructure: existing Kafka topics for meter readings already exist (from ch106)

---

### Your Task

Design the Apache Flink streaming replacement for the nightly batch demand forecast job.

---

### Deliverables

- [ ] Mermaid architecture diagram: Kafka → Flink streaming job → forecast output → grid ops dashboard
- [ ] Flink job design: DataStream API pipeline (source → keyed state → window functions → sink)
- [ ] Watermark strategy: maximum lateness = 4 hours; what happens to readings that arrive after the watermark?
- [ ] Late data handling design: allowed lateness window, side output for very-late events, confidence interval degradation
- [ ] State design: per-meter keyed state for rolling 30-day history (memory vs RocksDB state backend)
- [ ] Checkpointing design: checkpoint interval, storage backend, recovery time objective
- [ ] Migration plan: running batch and streaming in parallel during transition, cutover criteria
- [ ] Tradeoff analysis (minimum 3):
  - Flink vs Spark Structured Streaming for this workload
  - Event time vs processing time for demand forecasting
  - 4-hour watermark (high completeness) vs 15-minute watermark (low latency) — the SLA tradeoff
