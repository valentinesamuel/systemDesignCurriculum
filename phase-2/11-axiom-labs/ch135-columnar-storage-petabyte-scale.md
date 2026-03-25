---
**Title**: Axiom Labs — Chapter 135: Twelve Petabytes and Counting
**Level**: Staff
**Difficulty**: 8
**Tags**: #columnar-storage #apache-parquet #apache-iceberg #petabyte-scale #query-optimization #z-ordering #cost-engineering
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 107 (Crestline Iceberg), Ch. 71 (NexaCare LSM), Ch. 103 (LightspeedRetail OLAP)
**Exercise Type**: System Design
---

### Story Context

**Slack — #platform-infrastructure — Monday 09:15**

**Kofi Mensah**: Monthly cost review done. I need someone to look at this with me.

He posts a screenshot: AWS S3 bill — $280,000/month. Growing at 20% per month.

**[you]**: That's $3.36M/year annualized. At 20% monthly growth, it doubles in under 4 months.

**Kofi Mensah**: I know.

**[you]**: What's the data?

**Kofi Mensah**: Raw FASTQ files from sequencing machines. FASTQ is the standard format — plain text, each read is 4 lines: sequence ID, nucleotide string, plus sign, quality scores. 250KB per file average. 500TB of new files per month.

**[you]**: Is any of it compressed?

**Kofi Mensah**: No. The sequencing machines output uncompressed FASTQ directly.

**[you]**: Uncompressed FASTQ to S3 is the worst possible design. FASTQ compresses at 4:1 with gzip and 6:1 with zstd. We're paying to store 3-6x more data than necessary.

**Kofi Mensah**: That's the smaller problem. The bigger problem is query performance.

---

**Researcher feedback (compiled by Kofi)**

Dr. Mei Lin (Stanford, Computational Oncology): "I submitted a cross-cohort variant analysis last Thursday. I'm querying variants across 50 research groups — all groups that have consented to sharing for oncology studies. The query has been running for 14 hours. I haven't gotten results yet."

Dr. Aiko Tanaka (Tokyo University, Rare Diseases): "Every time I run a query that filters by gene region (chromosome 7, positions 117,000,000 to 117,500,000), the system scans every file in the database. My filters are never applied before reading. I've started submitting queries before I leave for the night and checking in the morning."

Kofi Mensah's assessment: "The data is stored as raw FASTQ files in a flat S3 directory tree: `s3://axiom-data/[university]/[year]/[study_id]/[sample_id].fastq`. There is no columnar storage. There is no partitioning. There are no statistics. Every query is a full scan of everything."

---

**Design meeting — Tuesday 14:00**
**Attendees**: Dr. Priya Sharma, Kofi Mensah, [you]

**Dr. Priya Sharma**: How bad is it?

**[you]**: Let me give you the math. 12 petabytes of raw FASTQ. If we convert to Parquet with zstd compression and extract the key analytical columns (chromosome, position, allele, quality score, sample ID, study ID), we get: 6:1 compression → 2 petabytes. Query performance: instead of scanning 12PB, a query filtering on chromosome 7 scans only the chromosome 7 partition: ~500GB. That's a 24,000x reduction in data scanned per query.

**Dr. Priya Sharma**: Why hasn't anyone done this?

**Kofi Mensah**: Because converting 12 petabytes of existing FASTQ to Parquet takes approximately 3 weeks of continuous compute at current processing rates. And nobody wanted to change the storage format while the platform was live.

**Dr. Priya Sharma**: What does it cost to not change?

**[you]**: At 20% monthly growth and current $280k/month: $2.1M more per year in storage costs. Plus researcher time: if Dr. Lin's query took 14 hours instead of 4 minutes, we're destroying thousands of researcher-hours per month. At a university cost of $100/researcher-hour, that's a significant but hard-to-quantify cost.

**Dr. Priya Sharma**: Design the migration. What do we need?

---

### Problem Statement

Axiom Labs stores 12 petabytes of genomic sequence data as uncompressed raw FASTQ files in flat S3 directories with no columnar optimization, no partitioning, and no query statistics. Storage costs are $280k/month and growing 20% monthly. Cross-cohort queries take 14 hours due to full-database scans. Migrate to Apache Parquet + Apache Iceberg with an optimized partition strategy for genomic access patterns.

---

### Explicit Requirements

1. Reduce storage cost from $280k/month to ≤ $80k/month through compression and format optimization
2. Cross-cohort variant queries filtering by chromosome/position must complete in ≤ 10 minutes
3. Schema evolution: new genomic annotation fields must be addable without rewriting historical data
4. Time travel: researchers must be able to query the database state at a specific historical date (for reproducibility)
5. Iceberg table evolution must support the 500TB/month write rate without write amplification
6. Zero data loss during migration (3-week migration window, platform must remain live)

---

### Hidden Requirements

- **Hint**: Dr. Tanaka's query filters by chromosome and position range. The optimal partition strategy is (chromosome, position_range) — but chromosomes 1-22 have very different sizes (chr1 is 8x larger than chr22). A naive partition-by-chromosome creates wildly uneven partition sizes. What is the correct sub-partitioning strategy for the genome's uneven distribution?

- **Hint**: 12 petabytes of FASTQ → Parquet conversion takes 3 weeks. During those 3 weeks, new data is arriving at 500TB/month (~16TB/day). Your migration must handle live writes to the new Parquet format while the conversion of historical data is still in progress. What is the dual-write strategy?

---

### Constraints

- 12 petabytes existing data, 500TB/month incoming
- FASTQ → Parquet conversion rate: ~200TB/day with 40 compute nodes (60 days for full migration)
- Human genome: 23 chromosome pairs, highly uneven length distribution
- Query patterns: filter by (chromosome, position range) + (study ID) + (date range)
- S3 storage pricing: $0.023/GB → current 12PB = $280k/month (including replication); target ≤ $80k/month
- Iceberg table format: must support ACID writes, schema evolution, time travel

---

### Your Task

Design the columnar storage migration for Axiom Labs' 12-petabyte genomic data lake.

---

### Deliverables

- [ ] Parquet schema design: column layout for genomic data (chromosome, position, allele, quality, sample_id, study_id, ingestion_date)
- [ ] Partition strategy: genome coordinates × study dimensions, with sub-partitioning for uneven chromosome sizes
- [ ] Z-ordering columns: multi-dimensional access pattern optimization for (chromosome, position) range scans
- [ ] Compression codec selection: gzip vs zstd vs snappy for genomic data — storage vs CPU tradeoff
- [ ] Migration strategy: historical data conversion + live write dual-path during 60-day migration window
- [ ] Iceberg table management: manifest file tuning for 500TB/month write rate, compaction scheduling
- [ ] Cost model: before (12PB uncompressed) vs after (projected compressed size) with storage cost math
- [ ] Tradeoff analysis (minimum 3):
  - Apache Iceberg vs Delta Lake vs Apache Hudi for this genomics workload
  - Partition pruning vs Z-ordering for multi-dimensional genomic queries — when each helps
  - In-place conversion vs shadow copy — risk vs downtime for 12PB migration
