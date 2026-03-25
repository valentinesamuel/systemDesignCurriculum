---
**Title**: NexaCare — Chapter 71: Database Internals — B-Tree vs LSM-Tree
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #database-internals #btree #lsm-tree #rocksdb #write-amplification #compaction
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 67, Ch. 78, Ch. 141
**Exercise Type**: Performance Optimization / System Design
---

### Story Context

**#engineering — NexaCare**
`Wednesday 14:22` **kofi.mensah** *(Data Engineering)*: the clinical trial write pipeline is at 68% throughput compared to last month. same data volume. something's wrong.
`14:24` **sasha.petrov** *(DBA)*: I've been saying the schema is wrong for months. too many nullable columns on trial_observations. the indexes are bloated.
`14:25` **kofi.mensah**: the schema hasn't changed. this is a storage engine issue.
`14:26` **sasha.petrov**: it's definitely a schema issue. have you seen the index sizes?
`14:27` **you**: before we argue about this — what's the actual write pattern for trial_observations?
`14:28` **kofi.mensah**: mostly inserts. new observations coming in from 47 active trials. maybe 2% updates (data corrections from site coordinators).
`14:29` **sasha.petrov**: 98% inserts and you're using Postgres? that's fine. Postgres is excellent for inserts.
`14:30` **you**: show me the `pg_stat_user_tables` for trial_observations
`14:31` **sasha.petrov**: *(pastes output)* n_live_tup: 847M, n_dead_tup: 312M, last_autovacuum: 3 days ago
`14:32` **you**: 312 million dead tuples. Postgres isn't keeping up with vacuum. this IS a storage pattern mismatch — but not in the way you're thinking, Sasha.

**DM — Marcus Webb → You**
`Wednesday 16:03`
**marcus.webb**: heard you're looking at a write throughput problem at NexaCare
**you**: how do you keep hearing about my projects before I've even diagnosed them?
**marcus.webb**: Fatima and I went to the same distributed systems conference in 2019. she texts me when she's worried.
**marcus.webb**: what's the write pattern?
**you**: 98% inserts, 2% updates. 4 million writes per day. one very large table.
**marcus.webb**: and you're using Postgres?
**you**: yes
**marcus.webb**: Postgres is a B-tree database. B-trees are optimized for mixed read/write workloads. for 98% insert workloads at 4M writes/day, you're fighting the storage engine.
**you**: what should we use?
**marcus.webb**: what do you know about LSM-trees?
**you**: log-structured merge trees. writes go to a memtable first, then flush to SSTables, then get compacted. optimized for write-heavy workloads.
**marcus.webb**: good. now tell me what write amplification is and why it matters for your 4M/day workload.
**marcus.webb**: do your own research first. come back with numbers.

---

**1:1 Session — You and Sasha Petrov, Two Days Later**

**Sasha**: You've been reading about LSM-trees.

**You**: RocksDB uses LSM-trees. Cassandra uses LSM-trees. ScyllaDB uses LSM-trees. They're all optimized for write-heavy append workloads — which is exactly what trial_observations is.

**Sasha**: Postgres is fast enough.

**You**: Postgres is great. But for this specific workload — 4M inserts/day, 98% write path, large table that never shrinks — the B-tree structure means every insert needs to maintain the index. On a table with 847 million rows, that index maintenance cost is significant. And because updates create dead tuples in Postgres's MVCC model, vacuum can't keep up.

**Sasha**: You want to move trial_observations to RocksDB? That's a significant architectural change.

**You**: I want to understand if it's worth it. That's what this meeting is for. Walk me through why you think the B-tree design is still the right call.

**Sasha**: *(pauses)* ...because we know how to operate Postgres. RocksDB compaction is complex. I've seen teams spend 6 months tuning compaction policies.

**You**: That's a valid operations concern. Let me model both options with real numbers.

---

### Problem Statement

NexaCare's `trial_observations` table has 847 million rows, receives 4 million inserts per day (98% inserts, 2% updates), and has accumulated 312 million dead tuples that autovacuum cannot keep pace with. The B-tree-based PostgreSQL storage engine is a mismatch for this heavily write-skewed workload, causing write throughput degradation over time as the B-tree indexes grow and vacuum falls behind.

You need to evaluate whether migrating the write path to an LSM-tree-based storage engine (RocksDB/Cassandra) is justified, design the migration if so, or design a PostgreSQL-optimized alternative that addresses the mismatch without changing storage engines.

---

### Explicit Requirements

1. Write throughput must return to baseline (and maintain it as the table grows to 2B+ rows)
2. Dead tuple accumulation must be bounded (autovacuum must reliably keep up)
3. Read performance for trial data queries must remain sub-500ms for single-trial range scans
4. The solution must be operable by a DBA team familiar with SQL databases (no exotic operational requirements)
5. Data integrity guarantees must match or exceed current PostgreSQL guarantees
6. Migration (if required) must be possible without downtime to active trials

---

### Hidden Requirements

- **Hint**: Re-read Kofi's message: "4 million writes/day." But also — NexaCare has 47 active trials. If a trial sponsor runs a data correction sweep (bulk updates to 500,000 observations), the update ratio temporarily spikes to 40%. How does your storage engine recommendation hold up under this burst write pattern?

- **Hint**: Sasha mentioned "we know how to operate Postgres." This is a real constraint, not just resistance. What's the total cost of ownership difference between operating RocksDB-backed storage at NexaCare's scale vs optimizing PostgreSQL? Consider that NexaCare has a 3-person DBA team.

---

### Constraints

- 847M rows currently, growing to ~2B over 2 years
- 4M inserts/day baseline; burst updates up to 500k/hour during correction sweeps
- PostgreSQL 15 on RDS (current setup, 3-DBA team, 6 years of institutional knowledge)
- Read pattern: 200 range scans/day per trial (trial coordinators querying their trial data)
- FDA audit queries: full trial history scans, run monthly, 47 trials × ~10M rows/trial
- Storage budget: current RDS instance is $8,400/month; cannot increase by more than 30%

---

### Your Task

Evaluate B-tree (PostgreSQL) vs LSM-tree (RocksDB/Cassandra) for NexaCare's write-heavy trial observations workload. Make a recommendation with supporting analysis, and design the implementation path.

---

### Deliverables

- [ ] B-tree internals explanation: How a B-tree index handles a 4M/day insert workload — where write amplification occurs and how dead tuples accumulate
- [ ] LSM-tree internals explanation: How a memtable → SSTable → compaction cycle handles the same workload, and what the write amplification factor is
- [ ] Comparative analysis: Write amplification factor comparison for this specific workload (B-tree vs LSM-tree, with math)
- [ ] Recommendation: B-tree optimization path (PostgreSQL tuning) vs LSM-tree migration (RocksDB). Pick one and defend it.
- [ ] PostgreSQL optimization path (if chosen): Specific settings — `autovacuum_vacuum_scale_factor`, `fillfactor`, partitioning strategy, UNLOGGED tables for staging
- [ ] RocksDB/Cassandra migration path (if chosen): Data model changes, migration procedure, operational requirements
- [ ] Scaling estimation: At 2B rows in 2 years, what's the index size and vacuum cost under each approach?
- [ ] Tradeoff analysis (minimum 3):
  - Write amplification: B-tree vs LSM-tree at 4M writes/day
  - Read performance: B-tree (index lookups) vs LSM-tree (bloom filters + SSTable scan) for range queries
  - Operational complexity: DBA team expertise gap for LSM-tree-based systems
