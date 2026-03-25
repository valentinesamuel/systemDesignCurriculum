---
**Title**: ShieldMutual — Chapter 187: The Actuarial Pipeline
**Level**: Staff
**Difficulty**: 8
**Tags**: #data-pipeline #batch-processing #observability #insurance #compliance #debugging #mentorship
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 186 (Risk Engine), Ch. 55 (LuminaryAI ETL), Ch. 105 (Pattern Summary 8)
**Exercise Type**: System Design + Debugging
---

### Story Context

The Slack message arrives at 11:47 PM on a Wednesday.

---

**Direct Message — @DrAdaezeObi → @You**
**Wednesday 11:47 PM**

**Dr. Adaeze Obi**: I hear you're at ShieldMutual now. Interesting domain.

**Dr. Adaeze Obi**: Tell me something — what's the most important constraint in an actuarial pipeline that doesn't exist in a regular data pipeline?

**Dr. Adaeze Obi**: Don't answer yet. Think about it first.

---

You put down your phone. You were already up because of a different Slack message, this one from the on-call rotation:

---

**#incidents** | Wednesday 11:31 PM

**[PagerDuty]**: ALERT: actuarial-pipeline — runtime exceeded 6h threshold (currently 14h 22min). SLA breach risk.

**Adwoa Sarfo** [on-call]: I'm on it. Checking now.

**Adwoa Sarfo**: okay this is bad. The pipeline started at 9:30am. It's been running for 14 hours. It should finish in 4. We're at 47% complete.

**Adwoa Sarfo**: I'm looking at the logs. Something is wrong with the claims aggregation step. It's processing... very slowly.

**Tomás Rivera** [joined at 11:41pm]: did you check the DB connections?

**Adwoa Sarfo**: yes. Connection pool looks fine on the surface. 20/20 connections in use.

**Tomás Rivera**: all 20 in use is not "fine." that's saturated.

**Adwoa Sarfo**: ...oh.

**You**: I'm on. What does the pipeline do in the claims aggregation step?

**Adwoa Sarfo**: It joins 50 billion historical claims events against 200 million policy records to calculate loss ratios by risk class, state, and year. It's a massive cross-join with aggregation.

**You**: Where does it run?

**Adwoa Sarfo**: RDS PostgreSQL. r5.4xlarge. 16 vCPU, 128GB RAM.

**You**: How long has it run on that instance?

**Adwoa Sarfo**: ...since 2019.

**Adwoa Sarfo**: The data has grown about 40% per year since then.

**You**: So the compute hasn't changed but the data has 3.4x'd.

**Adwoa Sarfo**: yes.

**Tomás Rivera**: and the query plans haven't been reviewed since the migration from MySQL in 2021

**You**: Okay. Walk me through the pipeline stages and their expected vs actual runtimes.

---

**Adwoa's pipeline stage breakdown** (shared as a spreadsheet link at 12:03 AM):

| Stage | Expected Duration | Actual Duration | Status |
|-------|------------------|-----------------|--------|
| 1. Policy extract | 20 min | 22 min | DONE |
| 2. Claims extract | 40 min | 43 min | DONE |
| 3. External enrichment (weather, ISO) | 60 min | 58 min | DONE |
| 4. Claims aggregation | 90 min | 12h 18min | RUNNING |
| 5. Reserve calculation | 45 min | NOT STARTED | PENDING |
| 6. Pricing table update | 30 min | NOT STARTED | PENDING |
| 7. Regulatory report generation | 15 min | NOT STARTED | PENDING |

**Total expected**: 4h 20min
**Current elapsed**: 14h 41min

---

**#incidents** | Thursday 12:09 AM

**Adwoa Sarfo**: I'm looking at the query for the claims aggregation. It's a 400-line SQL query. I'm going to paste a snippet.

**Adwoa Sarfo**:
```sql
SELECT
  p.risk_class,
  p.state,
  EXTRACT(YEAR FROM c.incident_date) as loss_year,
  COUNT(c.claim_id) as claim_count,
  SUM(c.paid_amount) as total_losses,
  SUM(p.written_premium) as total_premium,
  SUM(c.paid_amount) / NULLIF(SUM(p.written_premium), 0) as loss_ratio
FROM claims c
JOIN policies p ON c.policy_id = p.id
  AND c.incident_date BETWEEN p.effective_date AND p.expiry_date
WHERE c.incident_date >= '2000-01-01'
GROUP BY p.risk_class, p.state, EXTRACT(YEAR FROM c.incident_date)
ORDER BY p.state, p.risk_class, loss_year;
```

**Tomás Rivera**: the JOIN condition

**Tomás Rivera**: `c.incident_date BETWEEN p.effective_date AND p.expiry_date`

**Tomás Rivera**: that is a range join. PostgreSQL cannot use a standard B-tree index on a range join condition. it's doing a sequential scan on 50 billion rows.

**You**: Can it finish?

**Adwoa Sarfo**: At current rate, another 6 hours.

**You**: Which means it completes at 6am. Reserves calculation starts at 7am at earliest. Report generation at 8am. Regulatory submission deadline?

**Adwoa Sarfo**: 9am. State insurance department. It's automated.

**Adwoa Sarfo**: If it misses, we get a notice of violation.

---

**Direct Message — @DrAdaezeObi → @You**
**Thursday 12:17 AM**

*(You had forgotten about Dr. Obi's earlier message. You open it now, during a quiet moment while waiting for EXPLAIN ANALYZE to run.)*

**You**: I think I know the answer to your question. In a regular data pipeline, late is annoying. In an actuarial pipeline, late is a regulatory violation. The constraint is that the output has a legal deadline and a legal format — it's not just "data arrives eventually."

**Dr. Adaeze Obi**: That's part of it.

**Dr. Adaeze Obi**: What's the other part?

**Dr. Adaeze Obi**: Think about what the output is used for. Reserves. Pricing tables. These aren't analytics. What happens if the pipeline produces a subtly wrong answer that passes all validation checks?

*(You stare at that message for a long moment.)*

*(She never replies after that. She just asked.)*

---

By morning, you've documented the immediate fix (partition the claims table by year, pre-aggregate into a materialized view updated incrementally via CDC, rewrite the range join using an explicit date dimension table). But the deeper problem is architectural: this pipeline was built for 2019 data volumes and has never been formally reviewed. You need to redesign it.

---

### Problem Statement

ShieldMutual's nightly actuarial pipeline processes 50 billion historical claims events against 200 million policy records to produce: loss ratio calculations by risk class and state, reserve calculations (legally required daily), pricing table updates, and regulatory reports submitted to state insurance departments.

The pipeline runs on a single RDS PostgreSQL instance and has degraded from a 4-hour runtime to 14+ hours as data volume has grown 3.4x over five years. A pipeline failure or miss of the 9am regulatory submission deadline constitutes a regulatory violation. You must redesign the pipeline to meet the 6-hour window requirement at current data volumes and remain performant as data grows to 150B events over the next 3 years.

Additionally, Dr. Adaeze Obi's question must be answered architecturally: what happens when the pipeline produces a subtly wrong answer that passes all validation checks? The redesign must include a correctness verification layer.

---

### Explicit Requirements

1. Pipeline must complete within a 6-hour nightly window (hard deadline: output by 9am for 9am regulatory submission)
2. Process 50B historical claims events × 200M policy records; grow to 150B × 400M over 3 years
3. Regulatory reports must be: generated in state-mandated format, timestamped with processing completion time, and cryptographically signed
4. If the pipeline fails mid-run, it must be resumable from the last completed stage (not restart from scratch)
5. All aggregate outputs (loss ratios, reserve calculations) must be traceable to the source records that produced them — a regulator must be able to verify any individual output value
6. Reserve calculations must be version-controlled: each nightly run produces a versioned snapshot, and any run can be replayed to produce the same output given the same input snapshot
7. Alerting: if any pipeline stage exceeds 120% of its expected duration, on-call is paged immediately (not after the full SLA breach)

---

### Hidden Requirements

1. **Hint**: Dr. Adaeze Obi asks "what happens if the pipeline produces a subtly wrong answer that passes all validation checks?" Re-read this. In actuarial calculations, reserve amounts are used to determine whether ShieldMutual is solvent. An underestimated reserve is not just incorrect — it is a financial solvency risk. What does "correctness verification" mean for a calculation that has no external ground truth? The answer involves cross-validation against prior periods and independent recalculation of a sample subset.

2. **Hint**: Adwoa says the pipeline has been running on the same r5.4xlarge since 2019. The claim data has grown 40% per year. What does that trajectory look like at 3-year horizon? 50B × (1.4)^3 ≈ 137B. The current architecture hits a hard wall before that. The redesign must account for this — a purely vertical scaling answer fails within 36 months.

3. **Hint**: Tomás mentions the query plans "haven't been reviewed since the MySQL migration in 2021." A migration from MySQL to PostgreSQL does not automatically produce optimal PostgreSQL query plans. Implicit in this is that there may be data type choices (MySQL uses unsigned INT, PostgreSQL uses BIGINT differently), index types (MySQL InnoDB vs. PostgreSQL B-tree/BRIN), and statistics staleness issues that compound the range-join problem. The redesign must address not just the architecture but the query layer.

4. **Hint**: The stage table shows that Stage 3 ("External enrichment — weather, ISO") takes 58 minutes. This stage calls external third-party APIs. What happens to the pipeline if an external enrichment API is down during the nightly run? The current pipeline presumably fails. Should external enrichment be decoupled from the nightly pipeline run? This implies a pre-fetching or staging step for external data.

---

### Constraints

- **Data volume now**: 50B claims events, 200M policy records
- **Data volume 3 years**: 150B claims events, 400M policy records
- **Nightly window**: 9:30 PM start, must complete by 3:30 AM (6-hour budget)
- **Regulatory deadline**: 9:00 AM state submission (all 50 states, different formats)
- **Pipeline stages**: 7 stages, sequential dependencies between some, parallelizable between others
- **Team**: 3 data engineers, 1 actuary (for verification logic), 1 DBA
- **Current infrastructure**: Single RDS r5.4xlarge PostgreSQL; $8K/month. New design budget: up to $25K/month
- **Existing tools**: PySpark available (not yet used); Snowflake migration 60% complete
- **Compliance**: State insurance department reporting requirements vary by state (50 different schemas); NAIC annual statement requirements; reserve calculation must be auditable

---

### Your Task

Redesign the ShieldMutual nightly actuarial pipeline. Your design must: (1) solve the immediate 14-hour runtime problem; (2) scale to 3x data volume without architectural rework; (3) include a correctness verification layer that addresses Dr. Obi's question; (4) maintain a complete audit trail for regulatory traceability.

---

### Deliverables

- [ ] Mermaid architecture diagram (pipeline stages, data flow, compute layers)
- [ ] Root cause analysis of the current 14-hour runtime (3-4 specific technical issues identified in the story)
- [ ] Stage dependency graph: which stages can run in parallel, which are sequential
- [ ] Database schema changes: how to restructure the claims/policy tables to support the range-join efficiently (BRIN indexes, date dimension table, or pre-aggregation strategy)
- [ ] Incremental processing design: how to avoid full-table scans for historical data that hasn't changed
- [ ] Correctness verification layer design: how do you verify that aggregate outputs are correct without an external ground truth?
- [ ] Scaling estimation:
  - 50B rows at 200 bytes avg = 10 TB raw claims data
  - At 150B rows: 30 TB — what partitioning strategy handles this?
  - Nightly delta: estimate new claims per day, estimate processing time for delta-only run
- [ ] Cost modeling: current $8K/month → new design estimate. Must stay under $25K/month while handling 3x growth
- [ ] Tradeoff analysis (minimum 4):
  - Single-DB vs. distributed compute (Spark/Snowflake) for the aggregation step
  - Full reprocess vs. incremental/CDC-driven updates
  - Tight coupling between pipeline stages vs. staged materialized views
  - Monolithic pipeline vs. independent stage services with checkpointing

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
