# Pattern Summary 13: Staff Patterns III — Edge Computing, Safety-Critical Systems, Fleet-Scale Operations

**Type**: Pattern Summary (no deliverables)
**Covers**: Chapters from ShieldMutual and AutoMesh arcs
**Format**: Personal synthesis doc written after the ForgeSense onboarding week

---

> *Note written to self, Sunday night, after two weeks at ForgeSense.*
>
> During the all-hands last Thursday, Priya Nair from the platform team mentioned a Marcus Webb keynote — the one from StratechConf 2023. Someone pulled it up on a laptop. Webb was talking about systems that seem clever until they meet the physical world. "The edge is where your abstractions go to die," he said. "The sensor doesn't care about your beautiful microservices." He didn't name the company in the talk, but I recognized the AgroSense story immediately. Same cascading failure pattern. Same lesson.
>
> These are the patterns I'm carrying forward. Writing them down before they blur together.

---

## Pattern 1: Real-Time Risk Scoring Architecture (Multi-Source Data Fusion)

**From**: ShieldMutual arc

Risk scoring at real-time latency requires a different architecture than batch actuarial work. The core pattern:

- **Synchronous path**: sub-100ms scoring for decisions that block user action (underwriting approval, fraud check). Feed from: pre-computed features in Redis, lightweight model (logistic regression or gradient boost — not a deep network). Never call an external service on this path.
- **Asynchronous enrichment path**: enrich the decision with fuller context (credit bureau, claims history, third-party data) within 200-500ms. Update the risk score post-decision if needed. Use a soft-hold pattern similar to SkyRoute's seat reservation.
- **Batch recalibration path**: nightly or weekly. Pull all decisions from the day, compare against outcomes, recalibrate model weights. Feeds back into the feature store.

The three paths must share the same feature definitions but can have different feature freshness guarantees. Document which path accepts which freshness tolerance. If you don't, every engineer will make a different assumption and you'll have three incompatible risk models in six months.

**Key anti-pattern**: Trying to serve all three latency requirements from one scoring engine. The result is a system that's too slow for synchronous decisions and too shallow for batch recalibration.

---

## Pattern 2: Actuarial Pipeline Design (Correctness vs. Latency Tradeoffs)

**From**: ShieldMutual arc

Actuarial pipelines have a property that most software engineers don't encounter: **the output has legal standing**. A premium quote is a contract offer. A claims settlement calculation may be subject to regulatory review. This changes the architecture in non-obvious ways:

- **Immutability over efficiency**: actuarial calculation inputs must be versioned and stored alongside outputs. You cannot update a rate table in place and lose the version that produced a customer's premium quote three years ago. Bi-temporal modeling (valid time + transaction time) is required, not optional.
- **Reproducibility**: given the same inputs and the same model version, the system must produce the same output deterministically. This eliminates any randomized approximation methods on the critical path.
- **Audit trail depth**: every intermediate calculation should be capturable, not just the final output. Regulators can ask "show me how you arrived at this number" for any policy, any quote, any claim — years later.
- **Correctness gate before latency optimization**: run the correct calculation first. Then profile. Then optimize. The order matters. An incorrect actuarial calculation that runs fast is a liability, not an asset.

The implication for system design: the actuarial calculation engine should be a separate service with its own storage, its own versioning model, and its own SLO — separate from the customer-facing API that calls it.

---

## Pattern 3: V2X Communication Infrastructure Patterns

**From**: AutoMesh arc

Vehicle-to-everything (V2X) communication has latency requirements that cloud architectures cannot meet directly. The pattern stack:

- **V2V (vehicle-to-vehicle)**: direct DSRC (Dedicated Short-Range Communication) or C-V2X sidelink. No infrastructure in the path. Latency: 1-5ms. Use for: collision avoidance, cooperative cruise control.
- **V2I (vehicle-to-infrastructure)**: roadside units (RSUs) acting as local edge nodes. Latency: 5-20ms. Use for: signal phase and timing (SPaT), local hazard broadcast.
- **V2C (vehicle-to-cloud)**: cloud platform via cellular. Latency: 50-200ms. Use for: map updates, fleet telemetry, OTA staging, non-safety analytics.
- **Cloud to RSU to vehicle**: for content that is too large to broadcast V2V but needs to reach vehicles before a road segment. Use for: road work alerts, weather conditions ahead.

**Design rule**: never put a safety-critical decision on a path with cloud latency. If the path includes a cloud round-trip, it is not a safety path. Document this boundary explicitly in your architecture. Every engineer who touches the system must know which tier they are operating in.

---

## Pattern 4: Safety-Critical Edge Computing (FMEA-Driven Design)

**From**: AutoMesh arc

FMEA (Failure Mode and Effects Analysis) is a process from aerospace and automotive engineering. For every component, you enumerate: what can fail, how likely is it, and what is the severity of the effect. For software systems in safety-critical contexts:

- **Fail-safe over fail-open**: when a safety system loses connectivity or crashes, it must move to a known safe state, not continue operating. A traffic light controller that loses its coordination signal defaults to flashing red — not to its last known state.
- **Defensive degradation levels**: design explicit degradation modes. Level 0: full function. Level 1: degraded (local only, no cloud sync). Level 2: minimal safe operation. Level 3: fail-safe halt. Each level must be formally defined and tested.
- **Redundancy for critical paths**: single points of failure are unacceptable on safety paths. Dual sensors, dual communication channels, watchdog timers.
- **Formal proof over coverage testing**: for safety-critical logic, 100% test coverage is not sufficient. Formal verification (model checking, theorem proving) is appropriate. At minimum, prove the correctness of the state machine governing safety transitions.

**For system design interviews**: when you encounter a safety-critical system, the first question to ask is "what is the safe failure mode?" If the interviewer hasn't thought about this, you have just demonstrated Staff-level thinking.

---

## Pattern 5: OTA Updates for Safety-Critical Systems

**From**: AutoMesh arc

Over-the-air updates for safety-critical systems (vehicles, medical devices, industrial controllers) require a different update architecture than typical software:

- **Staged rollout with safety gate**: never update the entire fleet simultaneously. Stages: shadow fleet (internal vehicles) → early adopters (1%) → broader rollout (10%) → full fleet. Between each stage: a safety validation period with defined pass/fail criteria.
- **A/B at the edge**: the vehicle must be able to run the new version in shadow mode alongside the old version before fully committing. Compare outputs. If they diverge beyond threshold, halt rollout.
- **Atomic updates with rollback**: the update either fully applies or fully rolls back. Partial updates are worse than no update. Use A/B partitions on the filesystem (like Android's A/B system updates).
- **Update authorization chain**: safety-critical updates require a cryptographic chain of authorization. The vehicle must verify: the update came from the authorized manufacturer, was signed by an authorized engineer, and has not been tampered with. No unsigned updates.
- **Regulatory notification**: in many jurisdictions, an OTA update to a safety-critical system requires regulatory notification or approval (NHTSA in the US, TÜV in Germany). Build this notification pipeline into the update process, not as an afterthought.

---

## Pattern 6: Fleet-Scale Data Platforms (100M msg/sec)

**From**: AutoMesh arc

At 100M messages/second, standard Kafka deployment patterns break. The patterns that scale:

- **Partition key design is everything**: partition by vehicle ID ensures all events for a vehicle land on one partition (order preserved). But 10M vehicles × 10 msg/sec = 100M/sec, and a single partition handles ~10K/sec. You need ~10,000 partitions minimum. This breaks Kafka's consumer group rebalancing (which is O(partitions) on rebalance). Use COOPERATIVE rebalancing.
- **Tiered storage**: active event log on SSDs (last 72 hours). Older data tiered to object storage (S3/GCS). Kafka's tiered storage feature or a custom archival consumer.
- **Hot path / cold path separation**: safety telemetry (braking, steering, collision signals) on a separate topic with high-priority consumers. Analytics telemetry on a different topic. Never mix them. A slow analytics consumer must not cause consumer lag on the safety topic.
- **Schema registry is non-negotiable**: at 100M msg/sec, a schema change without registry coordination corrupts millions of messages before anyone notices. Enforce schema compatibility policies (backward compatible for minor changes, new topic for breaking changes).
- **Consumer lag as a leading indicator**: at fleet scale, consumer lag is a leading indicator of system health. Alert when any consumer group falls more than 5 minutes behind. A consumer that is 10 minutes behind at 100M msg/sec has already missed 60 billion messages.

---

## Pattern 7: CFO-Level Cost Modeling and Capacity Planning

**From**: ShieldMutual and AutoMesh arcs

Staff engineers are sometimes asked to present infrastructure cost to executive audiences. The format changes:

- **Unit economics, not absolute costs**: "our Kafka cluster costs $120K/month" lands differently than "processing each vehicle event costs $0.000001, and we process 100M/sec." The second is a conversation about efficiency; the first is a line item to cut.
- **Cost per business unit**: break infrastructure cost into per-customer, per-transaction, per-vehicle, per-claim. This maps to pricing model conversations and helps the CFO understand where cost is created and where it can be recovered.
- **Growth cost curve**: show the cost curve at current scale, 3x scale, and 10x scale. If cost grows linearly with traffic, that is a different conversation than if there are step changes at specific thresholds (new database tier, new region, new compliance boundary).
- **Reserved vs. spot strategy**: at scale, reserved capacity (1-year or 3-year commitments) for baseline load, spot for burst, is almost always 40-60% cheaper than on-demand for the baseline. Present this as a financial decision, not just a technical one.
- **The CFO question to prepare for**: "If we cut this budget by 30%, what breaks first?" Have an answer. It demonstrates that you understand cost vs. capability tradeoffs, not just what things cost.

---

## Pattern 8: State Regulatory Compliance as Architecture (The 50-State Problem)

**From**: ShieldMutual arc

Insurance in the United States is regulated at the state level. 50 states, 50 sets of rules. This is a microcosm of a broader problem that appears in: multi-jurisdiction fintech, healthcare across state lines, data residency across countries.

The architecture pattern:

- **Rules engine with external rule definitions**: the compliance rules must not be hardcoded in application logic. They live in a versioned, auditable rules store. When Kansas changes its rate filing rules, you update the Kansas rule definition, not the application code.
- **Jurisdiction resolution on ingestion**: every data object that has compliance implications (a premium quote, a claim, a customer record) must have its jurisdiction(s) resolved and stored at creation time. Jurisdiction resolution later is unreliable (customer moves, policy moves, regulatory interpretation changes).
- **Per-jurisdiction audit logs**: some states require that audit logs be stored within state borders. Others require specific retention periods. Design the audit log storage to be jurisdiction-aware from the start.
- **Compliance as a data product**: expose the current compliance state of any entity as a queryable data product. "Is this policy compliant in Texas?" should have a deterministic, auditable answer.

---

## Pattern 9: Claims Cascade Failure Modes and Circuit Breaker Design

**From**: ShieldMutual arc

Claims processing is a fan-in system: a single catastrophic event (hurricane, earthquake, wildfire) generates millions of correlated claims simultaneously. This is structurally similar to the Champions League notification spike from Beacon Media — but with regulatory consequences if the system fails.

- **The cascade pattern**: catastrophe event detected → claim intake spike (10-100x normal volume) → adjuster assignment queue saturates → third-party data services (property records, weather data, contractor networks) rate-limited → claims stall → SLA violations → regulatory complaints.
- **Pre-positioned capacity**: unlike normal traffic spikes, catastrophe events often have warning (a hurricane track can be predicted 72 hours out). Use weather API integration to trigger pre-scaling before the claims spike arrives. This is the "pre-staged fan-out" pattern from NeuroLearn, applied to infrastructure.
- **Circuit breakers on third-party enrichment**: when property record lookup fails, a claim should not fail — it should proceed with the data it has and be flagged for manual enrichment. Build circuit breakers on every external dependency in the claims pipeline.
- **Degraded mode claims handling**: define what a "minimum viable claim" looks like. What data is mandatory vs. enrichment? In catastrophe mode, accept minimum viable claims, enqueue enrichment separately, process with whatever you have. Never let the perfect be the enemy of the filed.
- **Correlated failure detection**: if 10,000 claims arrive in 60 seconds all referencing the same ZIP code, that is a signal — not just load. Route correlated claims to a dedicated catastrophe processing pipeline, separate from routine claims handling.

---

## Reflection Questions

1. The Marcus Webb quote from StratechConf — "The edge is where your abstractions go to die" — is specifically about the failure mode when cloud-first assumptions meet physical-world constraints. Think about a system you know well that was designed cloud-first. What specific assumption would break first if you deployed it at the edge?

2. Pattern 2 establishes that actuarial pipelines require correctness gates before latency optimization. Most of your career, you have been taught to optimize early and continuously. What other domains or system types might require this inversion of priority? How would you recognize them before you've built the wrong thing?

3. The 50-state compliance problem (Pattern 8) is architecturally identical to the multi-jurisdiction data residency problem from Phase 2. Both require: rules external to code, jurisdiction resolution at ingestion, per-jurisdiction audit logs. If you were designing a new platform today that you knew would eventually face a 50-jurisdiction compliance problem, what would you build differently from day one — and what would you deliberately NOT build until you needed it?
