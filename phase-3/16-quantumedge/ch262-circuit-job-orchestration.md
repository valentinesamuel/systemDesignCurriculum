---
**Title**: QuantumEdge — Chapter 262: Circuit Job Orchestration
**Level**: Staff
**Difficulty**: 8
**Tags**: #distributed #messaging #pipeline #performance #scheduling
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 260, Ch. 261, Ch. 56 (LuminaryAI GPU scheduling), Ch. 142 (VertexCloud pipeline)
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**[Slack — #engineering, Wednesday 8:32am]**

```
dr.hassan.ali [8:32]
Good news: Helix Pharma's first batch of circuits is in queue. 10,000 circuits.
VQE algorithm, ~20 qubits each, ~150 gates each.

Bad news: I did the naive math.

Each circuit: 150 gates × 50ns = 7.5µs QPU execution time.
10,000 circuits × 7.5µs = 75 milliseconds of total quantum compute.

But each circuit submission to the QPU vendor API has overhead:
- Setup (load circuit, calibrate): 150ms
- Teardown (read results, reset): 50ms
- Transmission latency (Amsterdam → Helsinki): ~8ms round trip

So per circuit: 208ms overhead vs 7.5µs actual quantum compute.
Total naive execution time: 10,000 × 208ms = 2,080 seconds ≈ 34.6 minutes.
Only 0.036% of that time is actual quantum computation.
The rest is overhead.

That's... bad. Right?
```

```
yuki.tanaka [8:41]
That's the standard problem with NISQ-era QPUs.
Gate time is nanoseconds. Overhead is milliseconds.
99.9% of the time you're waiting for setup/teardown, not computing.
```

```
you [8:45]
What if we batch circuits?
The QPU vendor API supports batch submission — send N circuits in one API call.
Setup once, run N circuits, teardown once.
```

```
dr.hassan.ali [8:48]
I looked at this. The vendor supports batches up to 300 circuits per call.
At 300 circuits/batch:
- Setup: 150ms (once)
- 300 × 7.5µs = 2.25ms quantum compute
- Teardown: 50ms (once)
- Total: ~202ms for 300 circuits
vs 300 × 208ms = 62.4 seconds naive

That's a 308x speedup.
10,000 circuits ÷ 300 per batch × 34 batches × 202ms = ~6.9 seconds total.
Down from 34.6 minutes.
```

```
yuki.tanaka [8:52]
Wait. Are all 10,000 Helix circuits compatible with batching?
Batching only works if:
1. All circuits in a batch use the same qubit count
2. Same gate set (basis gates must match hardware calibration)
3. Same number of shots (vendor API limitation)
```

```
dr.hassan.ali [8:55]
Checking... no. Helix has 12 different VQE circuit "templates".
Circuits within the same template can be batched.
Across templates, they cannot.
So it's not one batch of 10,000. It's 12 batches of different sizes.
The largest template has ~3,800 circuits.
```

```
you [9:01]
Okay. So the orchestrator needs to:
1. Group circuits by template/compatibility profile
2. Create optimal batches within each group
3. Schedule batches across QPU-1, QPU-2 (IonQ isn't right for VQE)
4. Handle partial batch failure (if 1 circuit in a batch fails, what happens to the other 299?)
5. Aggregate results back to the original job

This is a fan-out/fan-in problem at the circuit level, not the job level.
```

---

**[1:1 transcript — You and Priya Nair (external advisor, video call), Wednesday 11am]**

*(Priya Nair, formerly Staff Engineer at Stratum Systems, now CTO at PrismHealth — she's been advising you informally on distributed systems architecture since your Stratum days.)*

**Priya**: "Walk me through the orchestration problem."

**You**: "We have a job — let's say Helix's molecular simulation job. It contains 10,000 circuits. Those 10,000 circuits need to be grouped, batched, scheduled across QPUs, executed, and the results aggregated back into a single result set that Helix downloads as one package. The job has a logical identity. The individual circuits are the unit of execution. The batch is the unit of QPU interaction."

**Priya**: "What's the failure model?"

**You**: "QPU hardware errors. Quantum decoherence. Network partition between Amsterdam and Helsinki. QPU vendor API rate limits. A batch might partially execute — 300 circuits submitted, QPU returns results for 247, the other 53 show error codes. Do we retry the 53? Re-batch them? Or do we return partial results to the customer and let them decide?"

**Priya**: "What does Helix need?"

**You**: "Scientific reproducibility. They need to know, for each circuit: was it executed? How many shots? What were the measurement outcomes? If we silently drop 53 circuits, they'll publish a paper with missing data. That's a career-ending problem for their researchers."

**Priya**: "So completeness matters more than speed."

**You**: "Completeness matters more than speed. But speed matters too — their simulation has a deadline. The orchestrator needs to: guarantee all circuits eventually complete (at most once per QPU attempt, at least once total), track which circuits have been executed vs pending vs failed, and give Helix a final complete result package when all 10,000 are done."

**Priya**: "You're describing idempotent, tracked, resumable execution at the circuit level."

**You**: "Exactly. The same pattern as outbox-based messaging. Each circuit is a unit of work with an execution state machine. The orchestrator is the coordinator. Retries are safe because circuits are stateless — same circuit, same QPU, same input → same output (modulo quantum randomness, but that's expected)."

**Priya**: [leans forward] "Quantum randomness. That's interesting. You said 'same circuit → same output.' But quantum measurement is probabilistic. Every run gives different shot counts. That's the point. So idempotency here doesn't mean same result — it means same execution attempt. You need to track the attempt, not the output."

**You**: "Right. The idempotency key is 'this circuit was submitted to this QPU at this time with these parameters.' Not 'this circuit produced this result.' The result is inherently non-deterministic."

**Priya**: "That's a subtle thing to get right. Make sure your schema reflects it."

---

### Problem Statement

QuantumEdge's circuit job orchestration system must manage the lifecycle of large quantum circuit jobs — from admission through batching, QPU execution, error handling, and result aggregation. The system must handle the unique execution model of quantum hardware: nanosecond gate times dwarfed by millisecond setup overhead, batch compatibility constraints, probabilistic results, and partial execution failures.

The Helix Pharma job — 10,000 circuits, 12 compatibility groups, two compatible QPUs — must be orchestrated to completion with guaranteed circuit-level execution tracking, efficient batching (target < 10 minutes for the full job), and a complete, auditable result package.

### Explicit Requirements

1. **Circuit grouping**: automatically group circuits by hardware compatibility profile (qubit count, gate set, shots per circuit)
2. **Batch optimization**: create optimal batches within each group, respecting vendor batch limit (300 circuits/batch)
3. **QPU assignment**: assign batches to available compatible QPUs based on utilization and circuit type affinity
4. **Execution tracking**: per-circuit execution state machine (pending → batched → submitted → executing → complete / failed → retrying)
5. **Partial batch failure handling**: handle QPU returning fewer results than circuits submitted — retry failed circuits
6. **Result aggregation**: aggregate all circuit results into a single job result package when all circuits complete
7. **Job-level status**: real-time job progress (N circuits complete, M pending, K failed, ETA to completion)
8. **Result storage**: store circuit-level results (shot count, measurement outcomes) in durable storage; job result package available for customer download
9. **Idempotent retry**: circuit retry must be safe — same circuit submitted again produces valid (probabilistic but valid) results

### Hidden Requirements

- **Hint: re-read Priya's point about quantum randomness.** The idempotency of quantum circuit execution is fundamentally different from deterministic computation. What does the result schema look like? If a circuit is retried (hardware error on first attempt), and produces results both times — which result is canonical? Should both be stored?

- **Hint: re-read Dr. Hassan's batch compatibility constraints.** "Same number of shots (vendor API limitation)" — this means a Helix job requesting variable shot counts per circuit is not batchable. Should the orchestrator normalize shot counts within a batch (round up to max), or split the job into shot-count-homogeneous groups? What's the cost impact of rounding up?

- **Hint: re-read Dr. Tanaka's earlier Slack about maintenance windows.** A Helix job running across a Sunday 2am maintenance window would have in-flight batches interrupted. The orchestrator must checkpoint job state so execution resumes after the 4-hour window without re-running completed circuits.

- **Hint: re-read the cost model.** Helix's 10,000 circuits at 1,000 shots each = 10M shots = $3,000. But if shot normalization rounds 800-shot circuits up to 1,000 shots, the actual cost increases. The orchestrator must report cost per job based on actual shots executed, not the submitted shot count.

### Constraints

- **QPU vendor batch limit**: 300 circuits per batch API call
- **QPU-1 and QPU-2 (IBM Eagle)**: VQE-compatible; 16 qubits max; setup/teardown: 200ms; transmission: 8ms RTT
- **QPU-3 (IonQ)**: not suitable for Helix VQE (gate time 200µs/gate makes short VQE circuits inefficient)
- **Helix job**: 10,000 circuits, 12 compatibility groups (largest: ~3,800 circuits), ~150 gates each, ~20 qubits
- **Shot cost**: $0.0003/shot — each circuit retry costs additional shots
- **QPU error rate**: ~1% per circuit — at 10,000 circuits, ~100 circuits expected to need retry
- **Result storage**: each circuit result is ~4KB (shot measurement bitstrings) × 10,000 = ~40MB per job
- **Job deadline**: Helix needs results by end of day Friday (48 hours)
- **Concurrent jobs**: up to 20 jobs from different customers at the same time
- **Maintenance window**: Sunday 2am CET — must checkpoint and resume

### Your Task

Design the circuit job orchestration system. Cover: the job lifecycle state machine, batching algorithm, QPU assignment, execution tracking database, partial failure handling, idempotent retry, result aggregation, and maintenance window checkpoint/resume.

### Deliverables

- [ ] Mermaid architecture diagram: job intake → batch optimizer → QPU dispatcher → result collector → aggregator → customer download
- [ ] Database schema: jobs, circuits, batches, qpu_executions, circuit_results, job_result_packages (with column types, indexes, and execution state machine fields)
- [ ] Batching algorithm design: how compatibility groups are identified, how batches are formed, how shot normalization works
- [ ] Execution state machine: diagram of circuit state transitions (pending → batched → submitted → complete/failed/retrying)
- [ ] Scaling estimation (show math): 10,000 circuits at 300/batch = 34 batches; 2 QPUs; parallel scheduling throughput; expected wall-clock time
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Shot normalization (round up for batchability) vs variable shots (complex batching, lower cost)
  - Aggressive retry (minimize missed circuits) vs conservative retry (control cost overrun)
  - Checkpoint frequency (resumability) vs checkpoint overhead (storage writes per circuit)
- [ ] Cost modeling: actual job cost for Helix (with retry overhead) vs quoted cost; monthly orchestration infrastructure cost
- [ ] Capacity planning: maximum concurrent jobs at current QPU count; threshold for QPU-4 addition
- [ ] Maintenance window checkpoint design: what state is persisted, how resume works after 4-hour window

### Diagram Format

Mermaid syntax.
