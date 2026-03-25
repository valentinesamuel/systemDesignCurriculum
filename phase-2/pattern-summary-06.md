# Pattern Summary — Distributed Consensus, Event Sourcing & Database Internals

**Type**: Pattern Summary Chapter
**Covers**: Chapters 64–75 (Stratum Systems, NexaCare)
**Chapter**: 76

---

**Slack Voice Note Transcript — Marcus Webb**
*Sent to: you@[current-company].io*
*Attached: annotated_architecture_diagram_ps6.pdf*

"This is Marcus. I sent you an annotated diagram — PDF attachment. Look at it while you listen.

The theme across Stratum and NexaCare is: **state is time**. Every hard problem you solved in those two companies — event ordering, audit trails, CQRS projections, vacuum storms — comes down to the fact that distributed systems need to agree on what the state of the world was at a specific point in time. Raft is about that. Lamport timestamps are about that. Event sourcing is a direct implementation of it. MVCC is Postgres's answer to the same question.

Here's the annotated diagram. I've drawn arrows between patterns that are the same idea wearing different clothes.

Let me walk you through what I flagged."

---

## Patterns 36–42

### Pattern 36: Raft Leader Election and Fencing Tokens

**What it is**: Raft is a consensus algorithm that ensures exactly one node acts as leader at any time. Fencing tokens (monotonically increasing numbers tied to leadership terms) prevent a stale leader from acting after it's been replaced.

**Why it keeps appearing**: Any system that needs a single authoritative decision-maker — primary databases, event sequencing, distributed locks — needs this pattern. The pattern isn't Raft specifically; it's the idea that leadership authority must be monotonically versioned and enforceable.

**The key insight**: A leader that *thinks* it's a leader is more dangerous than a node that's offline. Fencing tokens address this.

---

### Pattern 37: Lamport Timestamps and Vector Clocks for Causal Ordering

**What it is**: Lamport timestamps assign a logical counter to events such that if A happens-before B, then L(A) < L(B). Vector clocks extend this to track causality per-process, enabling "happened-before" reasoning across a distributed system.

**Why it keeps appearing**: Any system where physical clocks can't be trusted (which is every distributed system) needs a logical clock. The choice between Lamport and vector clocks depends on whether you need to detect concurrent events (vector clocks) or just establish a total order (Lamport is sufficient).

**The key insight**: "Timestamp" doesn't mean wall clock. In distributed systems, it should mean "logical position in the causal ordering of events."

---

### Pattern 38: CQRS Projection Design

**What it is**: Command Query Responsibility Segregation separates the write path (commands → event log) from the read path (projections derived from the event log). Each projection is optimized for a specific read pattern.

**Why it keeps appearing**: Any system with divergent read/write patterns needs this. The anti-pattern is a single table that serves both. CQRS appears in freight (current location + compliance history), pharma (trial state + audit trail), trading (order book + settlement history), and will appear again.

**The key insight**: The event log is the source of truth. Projections are caches. If a projection is wrong, you rebuild it. If the event log is wrong, you have a much worse problem.

---

### Pattern 39: WAL Mechanics and MVCC Isolation Levels

**What it is**: PostgreSQL's Write-Ahead Log ensures durability (writes are safe once the WAL is flushed). MVCC (Multi-Version Concurrency Control) provides each transaction with a consistent snapshot of the database without locking readers.

**Why it keeps appearing**: Every PostgreSQL application eventually runs into MVCC behavior — dead tuple accumulation, isolation level mismatches, vacuum storms. Understanding the mechanics is prerequisite to diagnosing these problems.

**The key insight**: Every UPDATE in Postgres is a write (new version) + a soft delete (mark old version as dead). High UPDATE workloads create dead tuples faster than vacuum can clean them. Design schemas to minimize updates.

---

### Pattern 40: LSM-Tree vs B-Tree Selection Criteria

**What it is**: B-trees are optimized for mixed read/write workloads with random access patterns. LSM-trees (Log-Structured Merge Trees) are optimized for write-heavy sequential workloads. The choice is driven by write-amplification math, not preference.

**The selection criteria**:
- > 80% writes, mostly inserts → LSM-tree (RocksDB, Cassandra, ScyllaDB)
- Mixed reads and writes, complex queries → B-tree (Postgres, MySQL)
- Time-series data → specialized (TimescaleDB, InfluxDB — usually LSM-based)

---

### Pattern 41: Event-Sourced Log as Compliance Audit Trail

**What it is**: When the event log is the source of truth, it naturally serves as an audit trail — every state change is recorded as an immutable event with a timestamp and author. Compliance requirements (FDA Part 11, financial audit trails, HIPAA) align with event sourcing semantics.

**The additional requirement**: The event log must be *verifiably* immutable. A regular table that happens to only be appended to isn't sufficient — you need cryptographic proof that records weren't modified. Hash chains achieve this.

---

### Pattern 42: Cryptographic Erasure for GDPR in Event-Sourced Systems

**What it is**: If patient data in an event log must be "erased" (GDPR Article 17), but the log is immutable (compliance requirement), you encrypt each patient's data with a patient-specific encryption key. "Erasure" becomes key destruction — the data remains in the log but is permanently unreadable.

**The conflict it resolves**: GDPR right to erasure vs append-only audit trail retention requirements. Cryptographic erasure satisfies both — the data is "erased" from the patient's perspective (unreadable), while the audit trail structure (record count, timestamps, non-PHI metadata) remains intact.

**Warning**: This requires per-record encryption at write time. It cannot be retrofitted to an existing unencrypted log without a full rewrite.

---

## Cross-Industry Pattern Observation

Both Stratum and NexaCare deal with *regulated state*: state that isn't just for application functionality, but that has legal and compliance meaning. The Stratum event log is evidence for customs authorities. The NexaCare audit trail is evidence for the FDA. In both cases:

1. The data must be immutable (append-only)
2. The data must be verifiably immutable (cryptographic proof)
3. The data must be queryable across time ("what was the state at T?")
4. The data may eventually need to be "erased" even though it's immutable

These four requirements together almost always lead to event sourcing + cryptographic chaining + cryptographic erasure. This pattern will appear again in: healthcare (PrismHealth), energy (Crestline), construction (BuildRight), and financial systems.

Marcus Webb's annotation on the diagram: *"I've built 6 of these systems in 20 years. They all look different on the surface. They're all the same problem."*

---

## Reflection Questions

1. In Pattern 38 (CQRS Projections), the event log is the source of truth. If a bug in the projection consumer creates an incorrect projection, you rebuild it from the event log. But what if the bug existed for 6 months and downstream systems (trial sponsors, customs authorities) made decisions based on the incorrect projection? The event log is correct — but the real-world state was wrong. How do you handle the gap between "what the system said" and "what the system should have said"?

2. Pattern 42 (Cryptographic Erasure) solves the GDPR vs immutability conflict by destroying the encryption key. But key destruction is irreversible — once destroyed, the encrypted data is permanently inaccessible. What's your verification process before destroying a key? What happens if you destroy the wrong key?

3. Pattern 37 (Lamport Timestamps) establishes causal ordering but not real-time ordering. Two events might be causally concurrent (neither happened-before the other) but physically one happened an hour before the other. When does this distinction matter to your application? Design a scenario where Lamport ordering produces a "correct" but unexpected result.
