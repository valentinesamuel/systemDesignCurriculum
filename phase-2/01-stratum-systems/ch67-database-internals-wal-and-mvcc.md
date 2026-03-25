---
**Title**: Stratum Systems — Chapter 67: Database Internals — WAL and MVCC
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #database-internals #wal #mvcc #postgres #transactions #isolation-levels
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 66, Ch. 71, Ch. 141
**Exercise Type**: Performance Optimization / Debugging
---

### Story Context

**Slack DM — Amara Chen (DBA, Stratum Systems) → You**
`Monday 10:44`
**amara.chen**: your CQRS event log is going to kill the DB if you don't fix two things
**you**: which two things?
**amara.chen**: 1. your transaction isolation is wrong. 2. your vacuum is eating your write throughput.
**you**: can you be more specific?
**amara.chen**: come to my desk at 2pm. bring coffee.

---

**1:1 Session Transcript — Amara Chen and You**
*Stratum Systems, Hamburg Office, Amara's Desk*

**Amara**: Look at this. *(pulls up a Grafana dashboard)* This is your write throughput on the shipment_events table. Tuesday morning: 7,400 writes/minute. By Thursday afternoon: 4,100 writes/minute. Same load. Half the throughput.

**You**: What changed?

**Amara**: Nothing changed. That's the problem. What you're seeing is vacuum falling behind. Your dead tuple count on shipment_events crossed 20 million rows on Wednesday night. When autovacuum finally kicked in Thursday morning, it held a share lock long enough to slow down your writes by 44%.

**You**: I didn't know dead tuples accumulated that fast.

**Amara**: Every UPDATE and DELETE in Postgres leaves a dead tuple. Your CQRS event log is append-only, so you don't have UPDATEs there — good. But your shipment_status projection table? Every time a shipment changes state, you're updating a row. That's dead tuples. You have 200,000 active shipments that each change state 3–5 times per day. That's up to a million dead tuples per day on one table.

**You**: And the isolation issue?

**Amara**: *(opens a query)* Your projection consumer is reading from the projection table with READ COMMITTED. So is the API. The problem is your batch projection rebuilds run at REPEATABLE READ. When both run simultaneously, you get write skew. The API returns stale data *after* the rebuild completes because it was already in the middle of a READ COMMITTED transaction when the rebuild ran.

**You**: That explains the stale data complaints from the customs API team.

**Amara**: Yes. They see "delivered" status. Then they re-query 30 seconds later during the same API flow and get "in transit." Their system throws a consistency validation error.

**You**: What should the isolation level be?

**Amara**: *(leans back)* I'm not going to tell you. What do you know about MVCC?

---

**#incidents — Wednesday, 3 Weeks Later**
`14:37` **amara.chen**: @you — the projection rebuild we ran at 14:00 took 47 minutes instead of the expected 12. write throughput dropped 78% during the rebuild. three partner APIs timed out.
`14:38` **you**: I see it. Looking at the EXPLAIN plan now.
`14:39` **amara.chen**: what does the autovacuum log say?
`14:40` **you**: autovacuum was in the middle of a full table scan on shipment_events when the rebuild started. they're fighting over the same pages.
`14:41` **amara.chen**: Design Review tomorrow. Bring a fix. This is now a design problem, not just a tuning problem.

**Design Review — Stratum Systems Architecture Team**
*Thursday 10:00 AM — Present: You, Amara Chen, Priya Nair, Lars Eriksson*

The learner presents the schema and vacuum configuration. Two minutes in, Amara interrupts:

**Amara**: Before you go further — explain MVCC to Lars. He's going to be maintaining this.

**Lars**: *(uncomfortable)* I know what MVCC is.

**Amara**: Then tell me what version a READ COMMITTED transaction sees if a row is updated by a concurrent REPEATABLE READ transaction that started before it.

*(Silence.)*

**Amara**: *(to you)* That's the gap in your design. The isolation level mismatch between your rebuild job and your API readers creates this exact scenario. What's your fix?

---

### Problem Statement

The CQRS projection table for shipment current-state is experiencing degraded write performance due to autovacuum contention and a transaction isolation mismatch between the projection rebuild process and the read API. You need to understand the root cause through the lens of PostgreSQL's WAL and MVCC mechanics, and redesign the schema and transaction model to eliminate both problems.

---

### Explicit Requirements

1. Projection table writes must maintain 5,000+ writes/minute throughput under concurrent read load
2. Projection rebuild jobs must not reduce write throughput below 80% of baseline
3. The read API must never return data that is older than the last completed projection update
4. Dead tuple accumulation on the projection table must be bounded (autovacuum must be able to keep up)
5. Transaction isolation must prevent write skew between the rebuild job and the read API
6. The solution must not require application-level locking that serializes all projection writes

---

### Hidden Requirements

- **Hint**: Re-read Amara's question about "what version a READ COMMITTED transaction sees if a row is updated by a concurrent REPEATABLE READ transaction." The isolation level interaction is the key. What transaction isolation level *should* the projection reads use, and why does it matter that the rebuild uses REPEATABLE READ?

- **Hint**: Amara said "bring coffee" — an unusually warm invitation. Re-read the incident where autovacuum and the projection rebuild were "fighting over the same pages." This hints at a schema-level solution (table design) that can reduce the fight, not just a configuration solution.

---

### Constraints

- PostgreSQL 15 on RDS (cannot change to a different database)
- 200,000 active shipments × avg 4 state changes/day = 800,000 projection updates/day
- Projection rebuild job runs daily at 02:00 UTC, takes 10–47 minutes depending on backlog
- API SLA: 500ms P99 for current-state queries
- Autovacuum cannot be disabled — compliance requirement (dead tuple history needed for audit)
- Team size: 2 engineers (you + Amara)

---

### Your Task

Design the schema and transaction isolation model for the shipment_status projection table that eliminates autovacuum contention and write skew. Include the WAL and MVCC mechanics that explain why the current design fails.

---

### Deliverables

- [ ] Explanation: WAL mechanics — how a write to the projection table becomes a WAL entry, gets flushed, and how dead tuples accumulate from UPDATEs
- [ ] Explanation: MVCC snapshot isolation — what each transaction isolation level sees when concurrent transactions modify the same row
- [ ] Schema redesign: New projection table design that reduces UPDATE frequency (hint: consider append-only with periodic compaction vs mutable row)
- [ ] Autovacuum configuration: Per-table autovacuum settings for the projection table (`autovacuum_vacuum_scale_factor`, `autovacuum_vacuum_cost_delay`, `fillfactor`)
- [ ] Transaction model: Correct isolation level assignment for (a) projection write consumer, (b) projection rebuild job, (c) read API
- [ ] Scaling estimation: At 800k projection updates/day, what's the dead tuple accumulation rate before and after your fix? Show the math.
- [ ] Tradeoff analysis (minimum 3):
  - REPEATABLE READ vs SERIALIZABLE for the rebuild job
  - Mutable row projection vs append-only + compaction
  - Autovacuum tuning vs connection pooling interaction (PgBouncer in transaction mode and MVCC)
