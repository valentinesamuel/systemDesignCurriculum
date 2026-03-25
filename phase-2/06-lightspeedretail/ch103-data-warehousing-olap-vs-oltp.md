---
**Title**: LightspeedRetail — Chapter 103: Data Warehousing — OLAP vs OLTP
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #data-warehousing #olap #oltp #redshift #parquet #star-schema #etl #analytics
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 107 (Crestline Parquet/Iceberg), Ch. 53 (OmniLogix ETL), Ch. 104
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**Email Chain**

**From**: Sofia Andersson <s.andersson@lightspeedretail.com> (VP Analytics)
**To**: Beatriz Santos <b.santos@lightspeedretail.com>
**CC**: [you] <you@lightspeedretail.com>
**Subject**: Analytics Query Performance — Urgent
**Date**: Monday October 21, 08:47

Beatriz,

I have to be direct: our analytics infrastructure is broken for our current data volume.

Last night's daily sales report took 47 minutes to generate. Our retail clients expect this report in their inbox by 07:00. It arrived at 08:32. I received three support escalations before 09:00 from enterprise clients (Henley & Partners, Meridian Fashion Group, Costello Retail).

The report runs five queries against the `transactions` table in our main Postgres database. The heaviest query is a GROUP BY aggregation across 18 months of transaction history — 4.2 billion rows. It executed for 43 minutes and blocked two background index maintenance jobs while it ran.

I have been raising this for three quarters. The answer has always been "we'll optimize the queries." We have optimized the queries. We have added indexes. The data volume has outgrown the OLTP database.

I need a proper analytics data warehouse. I have pricing from Redshift. I need architectural sign-off from engineering.

Sofia

---

**From**: Beatriz Santos <b.santos@lightspeedretail.com>
**To**: [you] <you@lightspeedretail.com>
**Subject**: FWD: Analytics Query Performance — Urgent
**Date**: Monday October 21, 09:03

Can you take this? Sofia is right. We've been patching this for too long.

One constraint: I don't want our OLTP DB to be involved in analytics at all after this. Complete separation.

—B

---

**#data-platform** — Slack, Monday October 21, 11:30

**[11:30] Sofia Andersson**: For context on our query patterns. We have three types:
1. Daily aggregation reports: whole-day rollups, 18-month lookback, GROUP BY store/region/category
2. Ad-hoc merchant queries: individual merchants querying their own transaction history with filters (date range, product, cashier)
3. Trend analysis: week-over-week and year-over-year comparisons, seasonality modeling

**[11:33] Sofia Andersson**: Type 1 is killing Postgres. Type 2 and 3 are mixed in there too.

**[11:37] [you]**: How often does the source data change? What's the write pattern on `transactions`?

**[11:39] Sofia Andersson**: Transactions are immutable once settled — about T+2 hours after authorization. So the write path is effectively append-only for anything older than 2 hours.

**[11:42] [you]**: Perfect. That simplifies the ETL significantly.

**[11:44] Sofia Andersson**: What's the architecture? I showed the team Redshift pricing last week. $250/TB/year for compressed storage. We have 14TB in Postgres right now.

**[11:47] [you]**: Redshift is a good option but let me walk you through the decision. The key question isn't just storage — it's query pattern. What you're describing is classic OLAP: aggregations over large fact tables with filter pushdown on dimension columns. That's exactly what columnar storage is designed for.

**[11:51] Sofia Andersson**: "Columnar storage"?

**[11:53] [you]**: Postgres stores data row by row. To answer "total sales by region for Q3" it has to read every row and every column in those rows, even the ones you don't need. A columnar store — Redshift, Parquet on S3, BigQuery — stores each column separately. To answer the same question, it only reads the `region` and `amount` columns. For a table with 50 columns and 4.2 billion rows, that's the difference between reading 210 billion values and reading 8.4 billion.

**[11:57] Sofia Andersson**: That's a 25x reduction in data read.

**[11:58] [you]**: Plus compression is dramatically better on columnar data — same column values compress together much more tightly than interleaved row data.

---

**Slack DM — [you] → Marcus Webb** — Monday October 21, 14:15

**[you]**: Building a data warehouse for retail analytics. 4.2B rows in Postgres currently. Transactions are append-only after T+2h. Moving to Redshift or similar. Standard star schema for retail?

**Marcus Webb**: Yes. Standard retail star schema: `fact_transactions` in the center. Dimension tables: `dim_store`, `dim_product`, `dim_cashier`, `dim_time`. Denormalize aggressively — OLAP queries do not want JOINs across normalized tables at 4.2B rows.

**Marcus Webb**: One thing people always get wrong: `dim_time`. Pre-generate a calendar table with every day, week, month, quarter, year pre-computed. Never compute date parts in a query — always join to `dim_time`. Your query planner will thank you.

**[you]**: ETL pattern?

**Marcus Webb**: CDC from Postgres → staging area → warehouse. Transactions are append-only so it's simple: high-watermark on `settled_at` timestamp. Load everything newer than the last ETL run. No upserts needed for the settled window.

**Marcus Webb**: Watch out for the 2-hour unsettled window. You need a reconciliation pass for transactions that were authorized but not yet settled when the ETL ran.

**[you]**: What about the ad-hoc merchant queries? They want to filter by their own store IDs with arbitrary date ranges.

**Marcus Webb**: Distribution key and sort key. In Redshift: DISTKEY on `store_id` and SORTKEY on `transaction_date`. Queries filtering by store + date will be co-located on the same node and will skip irrelevant blocks via zone maps. Sub-second on 4.2B rows with the right keys.

---

### Problem Statement

LightspeedRetail's 4.2-billion-row `transactions` table in Postgres is being used for both OLTP (live POS writes) and OLAP (analytics reporting). Long-running aggregation queries hold table locks, block maintenance operations, and miss SLAs. The VP Analytics reports a 47-minute query time against a 7AM delivery SLA. Complete physical separation of OLAP workloads from the OLTP database is required.

You must design a retail analytics data warehouse using a star schema, define the ETL pipeline from Postgres to the warehouse, select and justify the columnar storage technology, and establish query performance guarantees for the three analytics query types.

### Explicit Requirements

1. Analytics queries must run against a separate data store — no analytics workloads on the OLTP Postgres instance after migration.
2. Daily aggregation reports must complete in under 5 minutes (down from 47 minutes).
3. Star schema design: `fact_transactions` as the central fact table with at least 4 dimension tables.
4. ETL pipeline must run daily at 04:00 UTC and complete before the 07:00 report delivery SLA.
5. The ETL must handle the 2-hour unsettled transaction window: include a reconciliation pass for transactions authorized but not yet settled when the prior ETL ran.
6. Ad-hoc merchant queries must filter by `store_id` + date range in under 10 seconds at 4.2B rows.
7. Warehouse must support 3-year historical retention (current 18 months must expand to 36 months).
8. No analytics queries may touch the OLTP Postgres `transactions` table after migration cutover.

### Hidden Requirements

- **Hint**: Re-read Sofia Andersson: "I received three support escalations... from enterprise clients." These are multi-tenant clients. There is a hidden requirement: row-level security on the warehouse — merchant A must never be able to query merchant B's transaction data, even via ad-hoc queries.

- **Hint**: Re-read Beatriz Santos: "Complete separation." This implies the OLTP DB should not be involved in the ETL pipeline's hot path. There is a hidden requirement: CDC (not direct query) for ETL — using logical replication or a change stream from Postgres rather than a polling query that competes with live writes.

- **Hint**: Re-read Marcus Webb's note about `dim_time`: "Never compute date parts in a query." The hidden requirement is pre-computed fiscal calendar alignment. Retail analytics uses fiscal quarters (which do not align with calendar quarters). `dim_time` must include fiscal week, fiscal quarter, and retail season columns.

### Constraints

- **Current data**: 4.2 billion rows in Postgres `transactions` table, 14TB uncompressed
- **Daily growth**: ~40M new rows/day (Black Friday: ~200M)
- **Target retention**: 36 months (from current 18 months)
- **Projection at 36 months**: ~25TB uncompressed, ~6TB compressed columnar (estimated 4:1 compression)
- **Query SLAs**: Daily report < 5 min; ad-hoc merchant query < 10 sec; trend analysis < 60 sec
- **ETL window**: 04:00 UTC start, must complete by 05:30 UTC to allow report generation by 07:00
- **Technology options**: Redshift (provisioned or Serverless), Snowflake, BigQuery, Parquet on S3 with Athena
- **Budget**: $250k/year maximum for analytics infrastructure
- **Concurrent analytics users**: up to 200 merchants running ad-hoc queries simultaneously
- **Team**: 2 data engineers, 1 DBA

### Your Task

Design the complete retail analytics data warehouse architecture, star schema, ETL pipeline, and query optimization strategy.

### Deliverables

- [ ] **Mermaid architecture diagram** showing: Postgres OLTP → CDC (logical replication) → staging layer → ETL transformation → columnar warehouse → analytics access layer (BI tool, SQL API). Include merchant row-level security layer.

- [ ] **Star schema design**:
  ```
  fact_transactions (
    transaction_id   BIGINT        SORTKEY,
    store_key        INT           NOT NULL REFERENCES dim_store(store_key),
    product_key      INT           NOT NULL REFERENCES dim_product(product_key),
    cashier_key      INT           REFERENCES dim_cashier(cashier_key),
    time_key         INT           NOT NULL REFERENCES dim_time(time_key),
    amount_cents     BIGINT        NOT NULL,
    quantity         INT           NOT NULL,
    payment_method   VARCHAR(20)   NOT NULL,
    DISTKEY(store_key)
  )
  ```
  Define all four dimension tables. Show distribution and sort key choices with justification.

- [ ] **`dim_time` design**: Include calendar columns (year, month, day, day_of_week) AND retail-specific columns (fiscal_week, fiscal_quarter, retail_season, is_holiday, is_black_friday). Show how to pre-generate this table for 36 months.

- [ ] **ETL pipeline design**:
  - Source: Postgres logical replication slot → Debezium or pglogical
  - Staging: S3 landing zone (raw Parquet files, partitioned by `settled_at::date`)
  - Transform: settle unsettled transactions (join against authorization records), dimension key lookup
  - Load: Redshift COPY from S3 (append-only for settled window, UPSERT for reconciliation window)
  - Show: the 2-hour reconciliation pass design

- [ ] **Technology selection**: Choose between Redshift, Snowflake, and Parquet/S3+Athena. Justify on: cost at 6TB compressed, concurrent query performance at 200 users, management overhead, and row-level security support.

- [ ] **Scaling estimation**:
  - Daily ETL load: 40M rows/day × estimated 200 bytes/row = 8GB/day raw. After 4:1 compression = 2GB/day.
  - 36-month total: show storage projection and cost at your chosen technology's pricing.
  - Query performance: how does columnar storage + DISTKEY on store_key reduce a "total sales by store for Q3" query from 43 minutes to under 5 minutes? Show the I/O reduction math.

- [ ] **Row-level security design for multi-tenant merchants**: Each merchant may only query rows where `store_id IN (merchant's stores)`. Show the implementation in your chosen technology (Redshift: late binding views + RLS policies; Snowflake: row access policies).

- [ ] **Tradeoff analysis** (minimum 3):
  1. Redshift provisioned vs Redshift Serverless: Serverless scales to zero (saves cost overnight) but has cold-start latency. For a 07:00 daily report SLA, does serverless work?
  2. Star schema (denormalized) vs 3NF in warehouse: OLAP benefits from denormalization but dimension updates require full reprocessing. How do you handle slowly-changing dimensions (a store that changes region)?
  3. CDC-based ETL vs scheduled query dump: CDC adds complexity (Debezium, replication slots) but removes contention on OLTP. When is a scheduled nightly dump sufficient and when is CDC required?
