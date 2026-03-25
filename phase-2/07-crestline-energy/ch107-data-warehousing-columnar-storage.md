---
**Title**: Crestline Energy — Chapter 107: Grid Analytics — Apache Iceberg and Columnar Storage
**Level**: Staff
**Difficulty**: 7
**Tags**: #data-warehousing #apache-iceberg #parquet #presto #olap #energy-analytics #columnar
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 106 (Time-series at scale), Ch. 103 (LightspeedRetail OLAP), Ch. 135 (Axiom Labs columnar)
**Exercise Type**: System Design
---

### Story Context

**Email Thread**
**From**: Lena Krause (Compliance Officer)
**To**: [you], Elena Vasquez
**Subject**: URGENT — NERC CIP-013 Quarterly Report — Analyst Unable to Run Analysis
**Date**: Monday, 7:12 AM

Elena, [you] —

NERC CIP-013 requires supply chain risk management reporting to the regulator by the 15th of each quarter. We are 9 days away from the deadline.

I spoke with Zhou Wei (Data Analyst) this morning. He ran the quarterly report last night. The Python script failed after 11 hours with a MemoryError. He is re-running it now on a larger EC2 instance. He estimates completion in 14 hours — which puts delivery at Tuesday evening at the earliest. The regulator's portal closes at 5 PM on the 15th.

Additionally: the script has failed 3 times this quarter with different errors. I have documented all three in the attached log. Each time, Zhou has made manual corrections. I am not confident we can submit data that has been manually corrected three times as a reliable regulatory filing.

Please advise.

— Lena

---

**Slack DM — Dr. Marcus Thompson → [you]**
**08:15**

**Dr. Marcus Thompson**: Did you see Lena's email?

**[you]**: Just read it. Walk me through what Zhou's script is actually doing.

**Dr. Marcus Thompson**: It's pulling raw readings from PostgreSQL — currently 47TB. Joining meter metadata from a separate table. Aggregating by grid region, supplier, and firmware version. Then running 12 different risk calculations. In Python. On a laptop. It was fine when we had 2M meters. We have 50M now.

**[you]**: Who built this?

**Dr. Marcus Thompson**: Sanjay, when he first joined. Before me, before you, before Lena. It was a one-off analysis that became "the report."

**[you]**: Does Zhou know it's not sustainable?

**Dr. Marcus Thompson**: He's been saying so for 18 months. Nobody listened because the report kept getting submitted — until this quarter.

**[you]**: I'm going to need a week to build the right solution. For this quarter, can Zhou run against a Postgres read replica with a 30-day snapshot to reduce the scan size?

**Dr. Marcus Thompson**: Probably. But Lena won't love "reduced scan size."

**[you]**: Lena needs a filed report by the 15th. This quarter, reduced scan with documented scope limitation beats a MemoryError. Next quarter, we'll have the real system.

**Dr. Marcus Thompson**: I'll tell her. She won't be happy.

**[you]**: She doesn't need to be happy. She needs to be compliant.

---

**Slack DM — Elena Vasquez → [you]**
**09:42**

**Elena Vasquez**: I've approved the temporary fix for this quarter. But I want the permanent system designed by end of month. And I want NERC CIP-013 reporting to become a scheduled pipeline, not a manual script.

**[you]**: Understood. I'm going to propose Apache Iceberg + Presto for the analytical layer. Separate entirely from the TimescaleDB operational store.

**Elena Vasquez**: How long to build?

**[you]**: 6 weeks for the core pipeline. Reporting runs automated from week 7.

**Elena Vasquez**: Do it.

---

Three weeks later, you're in the architecture review presenting to Elena, Dr. Thompson, and Lena Krause. You've designed an analytical data platform on top of the existing S3 data lake: Apache Iceberg tables for the energy data, Presto for ad-hoc queries, and dbt for transformation pipelines.

Lena is looking at the slide showing the NERC CIP-013 reporting pipeline. She asks: "If the regulator asks for a restatement of Q3 data six months from now — can you provide it?"

That question is the one that makes you realize Iceberg's time-travel capability isn't a nice-to-have. It's a compliance requirement.

---

### Problem Statement

Crestline Energy's analytical data platform is a 14-hour Python script running against a 47TB OLTP PostgreSQL database. NERC CIP-013 requires quarterly regulatory submissions with historical data restatement capability. The current approach is fragile, unscalable, and cannot support time-travel queries. Design a proper OLAP data platform.

---

### Explicit Requirements

1. Separate analytical layer from the OLTP operational database
2. NERC CIP-013 quarterly reports must run in under 30 minutes (not 14 hours)
3. Historical data restatement: ability to re-run any report against data as it existed at any prior point in time
4. Support ad-hoc queries by non-engineers (analysts like Zhou Wei using SQL)
5. Schema evolution: new meter types and new risk calculation fields must be addable without breaking existing queries
6. Cost: analytical workloads should not consume OLTP database resources

---

### Hidden Requirements

- **Hint**: Re-read Lena's question: "If the regulator asks for a restatement of Q3 data six months from now — can you provide it?" NERC CIP-013 specifically requires 90-day data retention for compliance records AND the ability to restate prior-period filings. What does "restatement" require from your storage layer?

- **Hint**: Dr. Thompson mentioned the script has failed with 3 different errors this quarter. One of those errors was related to a schema change — a new column added to the meter_metadata table that the script didn't handle. Your design must account for schema evolution without breaking downstream consumers.

---

### Constraints

- 50M smart meters × 96 readings/day = 4.8B rows/day ingested
- Historical data: 3 years = ~5.2T rows in analytical store
- Analysts: 8 non-engineer users running SQL reports
- NERC CIP-013 reporting: quarterly, must complete in < 30 min
- Budget: current PostgreSQL analytics cost = $42k/month (compute + storage); target < $20k/month
- Schema changes: ~2 per quarter (new meter types, new regulatory fields)

---

### Your Task

Design the OLAP analytical data platform for Crestline Energy's grid analytics and regulatory reporting.

---

### Deliverables

- [ ] Mermaid architecture diagram: OLTP source → CDC pipeline → Iceberg tables on S3 → Presto query engine → reporting layer
- [ ] Storage design: Iceberg table schema for energy data (partitioning strategy, Z-ordering for multi-dimensional queries)
- [ ] Schema evolution strategy: how to add new columns without breaking existing NERC reports
- [ ] Time-travel design: how Iceberg snapshots support regulatory restatement requests
- [ ] Query cost model: estimated Presto query cost for the NERC CIP-013 report vs current 14-hour script
- [ ] dbt transformation pipeline: data quality checks, incremental models, test framework
- [ ] Tradeoff analysis (minimum 3):
  - Apache Iceberg vs Delta Lake vs Apache Hudi — selection criteria
  - Presto vs Athena vs Redshift for this workload
  - Full refresh vs incremental CDC pipeline — cost and consistency tradeoffs
