---
**Title**: PrismHealth — Chapter 141: The Vacuum That Wasn't Running
**Level**: Staff
**Difficulty**: 7
**Tags**: #postgres #vacuum #autovacuum #connection-pooling #bloat #database-internals #healthcare
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 67 (Stratum WAL/MVCC), Ch. 71 (NexaCare B-tree vs LSM), Ch. 78 (TradeSpark deep dive)
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**Monday 08:03**

**#incidents — PrismHealth Slack**

**pagerduty-bot**: 🔴 ALERT: patient_vitals query p99 > 3000ms — baseline: 18ms

**on-call: Fatima Usman**: I'm on. Looking.

**08:05**
**fatima.usman**: The `patient_readings` table — all queries are doing sequential scans. The index isn't being used. I checked pg_stat_user_indexes — the index exists. Rebuilding didn't help.

**08:07**
**jake.huang**: I'm on bridge. Pull pg_stat_user_tables. What's the `n_dead_tup` value for patient_readings?

**08:09**
**fatima.usman**: 41,892,447 dead tuples. 67% of the table is dead.

**08:10**
**jake.huang**: Table bloat. The query planner chose a sequential scan because the statistics are stale. When did autovacuum last run on this table?

**08:11**
**fatima.usman**: pg_stat_user_tables shows `last_autovacuum` as 22 days ago.

**08:12**
**jake.huang**: And the write volume on this table?

**fatima.usman**: 14M writes/minute from device ingestion.

**jake.huang**: We're writing to `patient_readings` at 14M/minute, and autovacuum hasn't run in 22 days. Of course there are 42M dead tuples.

---

**08:17 — Bridge**
**[you]** joins the bridge.

**Jake Huang**: Situation: `patient_readings` table, 18M rows with 42M dead tuples (67% bloat). Autovacuum hasn't kept up with our write rate. Query planner picked a sequential scan. Queries that should take 18ms are taking 3.2 seconds.

**[you]**: What's the patient impact?

**Jake Huang**: Vital sign queries — what nurses see in the clinical dashboard — are all running against this table. The dashboard is effectively down for live patient monitoring.

**Priya Nair** *(joining)*: How many patients are actively monitored right now?

**Jake Huang**: 380,000 patients at this hour.

**[you]**: Is the alert evaluation path affected?

**Jake Huang**: Alert evaluation queries patient history. Yes, it's affected.

**Priya Nair**: How do we stabilize?

**[you]**: Manual VACUUM ANALYZE. No FULL (that takes a full lock, which is worse). A standard VACUUM will reclaim the dead tuples and refresh statistics. Estimated time for a table this size: 8-12 minutes.

**Jake Huang**: Running it.

**[you]**: While it runs, I want to look at the autovacuum configuration. 22 days without autovacuum on a table receiving 14M writes/minute is a configuration problem, not bad luck.

---

**Root cause analysis — that afternoon**

`autovacuum_vacuum_scale_factor` for the `patient_readings` table: 0.2 (default). This means autovacuum triggers when 20% of the rows are dead. At 18M rows, that's 3.6M dead tuples. The table has 42M dead tuples — it should have triggered 11 times.

But autovacuum also has `autovacuum_vacuum_cost_delay` = 20ms (default). At this delay, vacuum processes ~20MB/second. At a table adding 14M rows/minute of dead tuples under write load, vacuum is perpetually behind. It triggers, makes progress, falls behind again. It never catches up.

The Monday morning write spike is the precipitating factor. Every Monday at 08:00, 3.4M offline devices reconnect and send batched weekend readings. The write volume spikes 8x. Autovacuum can't keep up.

**[you]** *(posting in #incidents)*: Root cause identified. Autovacuum configuration appropriate for a small table, not a 14M-writes/minute table. Fix: reduce `autovacuum_vacuum_cost_delay` to 2ms for this table, reduce `autovacuum_vacuum_scale_factor` to 0.01 (trigger at 1% dead tuples instead of 20%). Estimated effect: autovacuum triggers every ~3 hours instead of every 3 weeks.

---

### Problem Statement

PrismHealth's `patient_readings` table — the primary table for 380,000 actively monitored patients — accumulated 42 million dead tuples over 22 days because autovacuum configuration is set for a 2021 table size, not the current 14M-writes/minute write rate. Monday morning device reconnection spikes accelerate the problem. The result is a p99 query time of 3.2 seconds on a table that should respond in 18ms.

---

### Explicit Requirements

1. Restore `patient_readings` query performance to ≤ 25ms p99 (current baseline 18ms, +7ms tolerance)
2. Autovacuum must keep pace with 14M-writes/minute during normal load AND Monday morning spikes
3. Table bloat monitoring: alert before dead tuple count reaches 10% of live row count
4. Connection pooling: 10M devices × periodic queries → Postgres connection count must not exceed 200 connections to the primary
5. ANALYZE must run automatically after high-write windows (Monday morning spike) to keep query planner statistics current
6. Configuration changes must not require table downtime (no `VACUUM FULL`)

---

### Hidden Requirements

- **Hint**: The immediate fix is autovacuum tuning. But re-read the write pattern: "14M writes/minute." At this rate, the `patient_readings` table is growing indefinitely. Every patient reading is an INSERT (not an UPDATE), so there are no dead tuples from updates — but dead tuples from deleted expired records accumulate. Check: does PrismHealth have a data retention policy that deletes old readings? If readings are never deleted, there are no dead tuples from deletes either — which means the bloat is from MVCC update side-effects on rows that were touched by other operations. Clarify the actual source of dead tuples.

- **Hint**: 10M devices connecting to query the API → connection pooling. PgBouncer in transaction mode allows many clients to share few Postgres connections. But PrismHealth runs HIPAA-sensitive queries. Some Postgres features (advisory locks, `pg_notify`, `SET LOCAL`) break in PgBouncer transaction mode. Which features does the alert evaluation service rely on?

---

### Constraints

- `patient_readings` table: 18M rows, 42M dead tuples, 67% bloat
- Write rate: 14M writes/minute (233k writes/second)
- Monday morning spike: 8x write volume for 90 minutes starting at 08:00
- Current autovacuum: `scale_factor=0.2`, `cost_delay=20ms` (defaults)
- Target: autovacuum keeps up with write rate, triggers every 2-4 hours at normal load
- Connection limit: 200 Postgres connections (RDS instance limit)
- External clients: 10M device reads/minute + 50k clinical dashboard queries/minute

---

### Your Task

Tune PostgreSQL autovacuum and connection pooling for PrismHealth's high-write healthcare database.

---

### Deliverables

- [ ] Immediate recovery procedure: VACUUM ANALYZE command, expected runtime, monitoring during execution
- [ ] Autovacuum parameter calculations: `scale_factor`, `cost_delay`, `cost_limit`, `vacuum_freeze_min_age` — derived from write rate math
- [ ] Monday morning spike mitigation: pre-scheduled manual ANALYZE before 08:00 Monday, or autovacuum configuration that handles the spike
- [ ] Table bloat monitoring: Prometheus query for dead tuple percentage, alert threshold, PagerDuty escalation
- [ ] PgBouncer configuration: transaction mode vs session mode for PrismHealth's query patterns, pool size math
- [ ] Connection count model: 10M device clients + 50k dashboard queries → PgBouncer pool → Postgres (show the math)
- [ ] Tradeoff analysis (minimum 3):
  - `VACUUM FULL` (reclaims space, requires lock) vs standard `VACUUM` (no lock, slower reclamation) — when each is appropriate
  - Aggressive autovacuum tuning (more CPU) vs reactive manual vacuum (less overhead, more risk) — for life-safety systems
  - PgBouncer transaction mode vs session mode — feature compatibility vs connection multiplexing for HIPAA queries
