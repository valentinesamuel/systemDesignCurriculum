---
**Title**: TradeSpark — Chapter 78: Database Internals — LSM-Tree Compaction Storm
**Level**: Staff
**Difficulty**: 9
**Tags**: #database-internals #rocksdb #lsm-tree #compaction #write-stall #performance
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 71, Ch. 141
**Exercise Type**: Incident Response / Performance Optimization
---

### Story Context

**#incidents — TradeSpark**
`Tuesday 09:31:02 EST` **pagerduty-bot**: 🔴 P0 ALERT — trading_engine: order_submission_latency_p99 > 500ms (threshold: 10ms)
`09:31:04` **pagerduty-bot**: 🔴 P0 ALERT — trading_engine: rocksdb_write_stall ACTIVE
`09:31:07` **alex.hartmann**: TRADING IS DOWN. all hands. war room now.
`09:31:09` **you**: joining
`09:31:11` **marco.bellini** *(Senior Engineer)*: I'm looking at RocksDB metrics. L0 file count is at 36. That's the write stall threshold.
`09:31:15` **you**: L0 compaction is blocking writes. when did this start?
`09:31:18` **marco.bellini**: the stall started at 09:30:58. market opened at 09:30:00. we were processing normal open-bell volume for 58 seconds before this hit.
`09:31:22` **you**: 58 seconds after market open. that's when order flow spikes 10x.
`09:31:25` **diana.voss**: are we recording trades? if we're not recording trades during this window, I need to know immediately for regulatory purposes.
`09:31:28` **you**: the trading engine is paused. no orders are being submitted. no trades executing. we are NOT losing trade data — but we ARE losing market opportunity.
`09:31:31` **alex.hartmann**: P&L estimate: $2.3M in trades missed since 09:30:58. clock is running.
`09:32:04` **you**: I can manually trigger a compaction. that will reduce L0 file count. but it will spike CPU to 100% for 2-3 minutes during the compaction run.
`09:32:08` **alex.hartmann**: what happens during those 2-3 minutes?
`09:32:10` **you**: write latency goes from 500ms to hopefully 5ms. reads remain available.
`09:32:12` **alex.hartmann**: do it
`09:32:13` **you**: triggering manual compaction... now

**#incidents — Incident Timeline**
`09:32:13` Manual compaction triggered
`09:33:47` L0 file count drops from 36 to 4
`09:33:51` Write stall clears. Order submission latency: 3.2ms.
`09:33:52` **marco.bellini**: we're back
`09:33:55` **alex.hartmann**: damage estimate?
`09:33:58` **diana.voss**: 3 minutes 53 seconds of trading suspension. 40 trading strategies paused.
`09:34:02` **alex.hartmann**: @you — I need a root cause in 2 hours and a prevention plan by end of day. this cannot happen again.

**Post-Incident Investigation — 11:00 AM**

**You**: The root cause is a compaction policy mismatch. Let me explain what happened.

**Marco**: *(pulling up RocksDB metrics)* The L0 file count hit 36 at market open. Our write stall threshold is 36 L0 files. So the moment we hit peak write volume at bell open, we immediately hit the stall.

**You**: Right. Here's the sequence of events:

1. Pre-market (9:20-9:29): Order flow is moderate. RocksDB is receiving ~8,000 writes/second. Memtable flushes to L0 files at a manageable rate.
2. Bell open (9:30:00): Order flow spikes to 180,000 writes/second. Memtable fills faster. Flush rate increases dramatically.
3. The L0 compaction job — which merges L0 files into L1 — is running at its default rate. It *cannot keep up* with the incoming flush rate.
4. By 9:30:58 (58 seconds after open): L0 file count reaches 36 (the configured `level0_slowdown_writes_trigger`). RocksDB enters write stall.

**Alex**: Why is the default rate too slow?

**You**: Because RocksDB's default compaction settings were designed for roughly uniform write rates. Our write rate is 10x higher at market open than at any other time of day. The compaction system doesn't know that 9:30 AM EST is special.

**Marco**: Can we just raise the write stall threshold to something higher than 36?

**You**: That's a band-aid. A higher threshold means more L0 files accumulate before the stall. More L0 files means slower reads (RocksDB scans all L0 files for reads). And when the stall *does* eventually hit, it'll be worse. The fix is compaction scheduling, not threshold raising.

---

### Problem Statement

TradeSpark's RocksDB storage layer experienced a write stall at market open due to L0 file accumulation exceeding the write stall threshold. The root cause is a mismatch between RocksDB's default compaction scheduling and TradeSpark's highly non-uniform write pattern (10x spike at market open). You need to understand the RocksDB LSM-tree internals that caused this failure and design a compaction strategy that prevents write stalls under TradeSpark's write pattern.

---

### Explicit Requirements

1. Write stall must not occur during normal trading operations, including market open spike
2. Write latency P99 must remain < 10ms during all market hours (09:30–16:00 EST)
3. Compaction must not monopolize CPU to the point of affecting trading strategy execution
4. Read latency must not degrade as L0 file count grows (bounded L0 file count at all times)
5. The solution must work within RocksDB's configuration API (no custom storage engine patches)
6. Monitoring: write stall risk must be detectable 30+ seconds before it occurs

---

### Hidden Requirements

- **Hint**: Re-read Marco's suggestion: "raise the write stall threshold." The learner correctly identifies this as a band-aid. But there's a second order effect: if you raise the stall threshold from 36 to 100, what happens to the *read* performance of RocksDB when there are 80 L0 files? L0 files are not sorted by key — every read must check all L0 files. How does this interact with TradeSpark's 1ms read latency requirement for current order state?

- **Hint**: The incident occurred at 09:30 — market open. TradeSpark's write pattern is highly predictable: low pre-market, high spike at 09:30, high sustained volume until 16:00, drop at market close. How would you design a *scheduled* compaction policy that pre-emptively compacts L0 before market open, rather than reacting during the spike?

---

### Constraints

- RocksDB for order state storage (cannot replace)
- Write rate: ~8,000 writes/sec pre-market, 180,000 writes/sec at market open, ~50,000 writes/sec sustained during trading
- Read latency requirement: < 1ms P99 for current order state lookups
- Write latency requirement: < 10ms P99
- CPU headroom: trading strategies use 70% of available CPU during active hours
- Market hours: 09:30–16:00 EST Monday–Friday

---

### Your Task

Diagnose the RocksDB compaction storm at the internals level and design a compaction strategy that prevents write stalls under TradeSpark's write pattern.

---

### Deliverables

- [ ] LSM-tree internals explanation: L0 files (unsorted, all written directly from memtable flushes) vs L1+ files (sorted, result of compaction). Why L0 file count matters for both write stalls and read performance.
- [ ] Write stall mechanics: Exactly what happens at 36 L0 files — how RocksDB throttles writes, what the recovery path is
- [ ] Compaction configuration fix: Specific RocksDB settings to tune (`max_write_buffer_number`, `level0_slowdown_writes_trigger`, `level0_stop_writes_trigger`, `max_background_compactions`, `compaction_readahead_size`)
- [ ] Pre-market compaction schedule: Design a scheduled compaction job that runs at 09:15 AM EST daily, before market open
- [ ] Monitoring design: What metric(s) to alert on to detect write stall risk 30+ seconds before it occurs
- [ ] Scaling estimation: At 180k writes/sec, how long until L0 hits 36 files if compaction is disabled? Show the math.
- [ ] Tradeoff analysis (minimum 3):
  - L0 file count threshold vs compaction rate (raising threshold vs fixing compaction)
  - CPU allocation: compaction threads vs trading strategy threads
  - Leveled compaction vs Universal compaction for this workload
