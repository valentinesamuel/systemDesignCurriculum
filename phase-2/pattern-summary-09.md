# Pattern Summary — Data Warehousing, Real-Time Scale & Advanced Search

**Type**: Pattern Summary (no deliverables)
**Covers**: Chapters 106–118 (Crestline + VenueFlow arcs)
**Tags**: #pattern-summary #data-warehousing #realtime #vector-search #ab-testing #rate-limiting

---

## From Marcus Webb — Voice Note Transcript

*[Background noise: airport terminal, gate announcements in the distance. Marcus sounds relaxed, unhurried. Different from his usual Slack terseness.]*

"Hey. Marcus. I'm at an airport — don't ask which one, it's not interesting. I was going to write this up but I figured a voice note was more honest.

You've covered a lot of ground in the last fifteen chapters. Let me pull the thread.

The Crestline arc — the data warehouse rebuild, the Iceberg tables, the SLO framework — that was about learning to manage *time*. Not time as in latency. Time as in: data that exists at 8 AM is not the same as data that exists at 8 PM, and your system needs a story for both. Columnar storage, schema evolution without downtime, error budgets that tell you when to act rather than just when something broke. Those are all answers to the same question: how do you build infrastructure that ages gracefully?

The VenueFlow arc was about learning to manage *scale with adversarial conditions*. That's a different problem. You can engineer around load. You can engineer around failure. Adversarial conditions are different because the attacker learns from your defenses. Your rate limiter becomes their test harness. Every threshold you set teaches them something.

I remember when 'real-time' meant 'under 10 seconds.' We were proud of that. Now your ticketing system needs to serialize 2 million seat claims in under 2 seconds. The physics haven't changed — only your ambitions. A Redis pub/sub channel still has the same fan-out characteristics it had in 2015. The difference is that you have 500,000 subscribers now instead of 5,000, and you've learned to shard the channel rather than scale the node.

The vector search chapter is worth sitting with. Every search system you've built until now was based on the assumption that the user knows what they want. Inverted indexes, BM25, relevance scoring — all of it assumes a query. Vector search is for when the user doesn't know what they want. That is a fundamentally different product premise, and it requires a fundamentally different retrieval model. Know which one you're building.

The A/B testing chapter is the one I'm most glad you got, because it's the one that will save you from the most invisible mistakes. Data that looks right and isn't right is more dangerous than data that looks wrong. The four confounded experiments at VenueFlow — that happens everywhere. Most companies never find out. They just make expensive decisions on bad data.

Anyway. That's the thread. Seven patterns below. Read them slowly.

— Marcus"

---

## Patterns Covered in Chapters 106–118

### Pattern 57: Columnar Storage Selection Criteria

Columnar storage (Parquet, ORC, Iceberg) is the right choice when:
- Query patterns are analytic (aggregate across many rows, few columns)
- Data is written in large batches, not row-by-row
- Schema evolution is expected (add columns without rewriting existing data)
- Storage cost is a constraint (columnar compression ratios are 5–10x better than row storage for structured numeric data)

It is the wrong choice when:
- You need to update or delete individual rows frequently
- Your primary access pattern is by primary key (row lookup)
- Write latency matters more than query performance

The Crestline arc introduced Iceberg specifically for schema evolution without full rewrite. The key insight: Iceberg's schema evolution is metadata-only — you add a column to the schema without touching the existing Parquet files. Existing partitions simply don't have that column; the reader returns NULL for it. This is what "evolution without downtime" means in practice.

### Pattern 58: Iceberg Table Evolution Without Downtime

The safe sequence for evolving an Iceberg table under live production queries:
1. Add the new column with a DEFAULT value (metadata operation, instant)
2. Update the writer to populate the new column in new partitions
3. Run a backfill job on old partitions during off-peak hours
4. Only after backfill: add NOT NULL constraint or application-level validation
5. Deprecate the old column after confirming no readers depend on it

The dangerous sequence (what not to do):
- Rename a column (breaks all existing readers that use the old name)
- Change a column type from INT to STRING without a migration plan
- Delete a column that still exists in old Parquet files (readers will error on old partitions)

### Pattern 59: SLO/SLI Design Process

A well-designed SLO answers: "What does success feel like to the user, and how bad does it have to get before we act?"

The design process:
1. Start with the user journey, not the system — identify the critical path a user takes
2. For each step in the critical path, define the SLI (the measurement): availability, latency, error rate, throughput
3. Set the SLO threshold based on user pain, not system capability ("99.9% of seat holds confirmed in <2s" not "our system can do 5ms")
4. Define the error budget: (1 - SLO) × time window. This is how much you're allowed to fail.
5. Define the error budget policy: what happens when the budget is 50% burned? 100%? (Freeze deploys? Page on-call? Escalate?)

The Crestline arc showed that SLOs without error budget policies are decorative. The metric tells you when something is wrong; the policy tells you what to do about it.

### Pattern 60: Pub/Sub Connection Sharding Strategy

When WebSocket fan-out reaches tens of thousands of connections per node, the Node.js event loop becomes the bottleneck — not the pub/sub layer. The fan-out from Redis pub/sub to individual socket writes is sequential in a single-threaded event loop.

The sharding strategy:
- Use consistent hashing to assign connections to gateway nodes by event_id (or venue_id)
- All connections watching Event X land on the same 2–3 gateway nodes
- Those nodes subscribe to the Redis channel for Event X
- Redis fan-out goes to 2–3 nodes instead of all nodes; each node does fewer socket writes

The tradeoff: consistent hashing creates hot nodes when a single event is unusually popular (Taylor Swift onsale). Mitigation: virtual nodes in the hash ring, with overflow routing to secondary nodes.

Heartbeat intervals below 10 seconds are necessary but not sufficient for detecting silent proxy drops. Add: connection-level sequence numbers. If a client misses 3 consecutive sequence numbers, the server treats the connection as dead regardless of heartbeat status.

### Pattern 61: Vector Search Architecture (When kNN vs. Text Search)

Use inverted index (BM25, TF-IDF) when:
- The user has explicit intent: they know what they're looking for ("Taylor Swift tickets")
- The query vocabulary matches the document vocabulary
- Relevance means frequency and position, not semantic meaning

Use vector search (kNN, cosine similarity) when:
- The user has implicit intent: they don't know what they're looking for
- You want to match *meaning* across different vocabulary ("indie folk" → similar artist graph)
- You have a representation model that can encode domain knowledge into embedding space

Hybrid search (BM25 + kNN re-ranked) is almost always better than either alone for user-facing search. BM25 for recall; kNN for personalization and semantic lift.

Cold-start in vector search has two forms:
- New user cold-start: use content-based signals (location, demographic, explicit preferences) to synthesize an initial embedding
- New item cold-start: use item metadata (genre, artist relations, category) to generate an embedding before any user interaction signal exists

### Pattern 62: A/B Testing Statistical Rigor

The four requirements for a valid A/B experiment:
1. **Consistent assignment**: same user → same variant, every session (use consistent hashing, not random)
2. **Mutual exclusion**: two experiments measuring the same metric cannot share users (use layers or orthogonal splits)
3. **Pre-determined sample size**: calculate required sample size before launching; do not stop based on observed significance
4. **Single primary metric**: declare one metric as the decision criterion at launch. Post-hoc metric selection inflates false positive rate.

The most common failure mode in practice: peeking. Running an experiment, checking results every day, and stopping when p < 0.05. This inflates the false positive rate from 5% to 20–30% depending on how often you peek. Sequential testing (SPRT, always-valid inference) is the correct solution when you need to check continuously.

Global holdout (5% of users excluded from all experiments) is the most underused tool in experimentation. It detects secular trends — metric changes that have nothing to do with any experiment.

### Pattern 63: Adaptive Rate Limiting (Traffic-Aware vs. Fixed-Rate)

Fixed rate limiting answers: "How many requests per second is this IP allowed to make?"
Adaptive rate limiting answers: "Is this traffic pattern consistent with human behavior?"

The key architectural difference:
- Fixed limits: compare request count against threshold. Simple. Evadable by distribution.
- Adaptive limits: compare behavioral signals against a baseline model. Complex. Requires signal collection infrastructure. Harder to evade because the baseline evolves.

The five-generation evolution of rate limiting in this curriculum:
1. CloudStack (Ch. 29): token bucket, Redis Lua. Defense against accidental overload.
2. CivicOS (Ch. 40): ASN-level limits, oracle removal. Defense against distributed attack.
3. SkyRoute (Ch. 62): multi-class, contractual overrides. Defense against insider gaming.
4. VenueFlow (Ch. 118): behavioral fingerprinting, adaptive thresholds. Defense against adversarial bots.
5. (Coming): network-level QoS. Defense at the carrier layer.

The pattern: each generation moves the defense closer to the client and further from the resource. The closer to the client, the more signal you have; the closer to the resource, the more certain you are of intent. The ideal system has both.

---

## Reflection Questions

*(No answers. Sit with these.)*

1. Pattern 63 traces five generations of rate limiting from CloudStack through VenueFlow. In each generation, the attacker adapted. What does the sixth generation look like — and is there a generation at which adaptive rate limiting becomes economically unviable for the defender?

2. Pattern 62 describes the global holdout as "the most underused tool in experimentation." Why do you think most companies skip it? What organizational dynamics make it politically difficult to exclude 5% of users from all experiments permanently?

3. Patterns 60 and 61 are both about the same underlying tension: the right data structure for the right retrieval problem. When you are designing a new system and the product team asks for "search," what questions do you ask before you decide whether you need inverted index, vector search, or both?

---
