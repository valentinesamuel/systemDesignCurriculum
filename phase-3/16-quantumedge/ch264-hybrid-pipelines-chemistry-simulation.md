---
**Title**: QuantumEdge — Chapter 264: Hybrid Pipelines — Chemistry Simulation
**Level**: Staff
**Difficulty**: 8
**Tags**: #pipeline #distributed #observability #reliability #data-pipeline
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 262, Ch. 263, Ch. 53 (OmniLogix saga pattern), Ch. 97 (NexaCare clinical pipeline)
**Exercise Type**: System Design
---

### Story Context

**[Email chain — Subject: LithiumX-7 Battery Material Simulation — Research Agreement]**

```
From: Dr. Ingrid Solberg <i.solberg@nordmaterials.no>
To: yuki.tanaka@quantumedge.io
Date: Monday, 10:14am
Subject: LithiumX-7 Battery Material Simulation — Research Agreement

Dr. Tanaka —

NordMaterials is ready to proceed with the LithiumX-7 solid-state electrolyte
simulation. As discussed, this is part of our EU Horizon grant for
next-generation battery technology.

Our pipeline specification:
1. Classical preprocessing: 1 hour (DFT geometry optimization, input preparation)
   Compute: 8× CPU nodes, 256GB RAM
2. Quantum simulation: VQE for electronic structure of Li₁₄Al₀.₄Ge₁.₆(PO₄)₃
   Estimated: 2 hours on QPU (23-qubit IonQ preferred — lower error rate critical)
3. Classical postprocessing: 4 hours (molecular dynamics analysis, visualization)
   Compute: 4× GPU nodes
Total estimated pipeline: 7 hours

Two requirements I must emphasize:

First: Scientific reproducibility. This simulation underpins a Nature paper
submission. Every step of the pipeline — inputs, parameters, QPU execution records,
measurement outcomes, postprocessing version — must be logged in a format suitable
for supplementary materials in a peer-reviewed publication. Our reviewers will ask
to reproduce our results.

Second: If the QPU step fails mid-execution (we've had hardware interrupts on
other platforms), we cannot re-run the 1-hour preprocessing from scratch.
The pipeline must checkpoint and resume from the QPU step.

Budget: €85,000 for 3 simulation runs (in case of parameter adjustment).

Dr. Ingrid Solberg
Principal Investigator, NordMaterials AS
```

---

**[Slack DM — Dr. Yuki Tanaka → You, Monday 11:00am]**

```
yuki.tanaka [11:00]
This is our first hybrid pipeline customer.
Not "send a circuit, get a result." A full 7-hour pipeline
where classical and quantum steps are tightly coupled.

The IonQ issue: QPU-3 (IonQ) has a 2% hardware failure rate per session.
At 2 hours of QPU time, that's... concerning.
We've never had a 2-hour IonQ session before.

If QPU-3 fails at hour 1:45, what happens?
The classical preprocessing took an hour.
The postprocessing hasn't started.
Do they pay for a re-run? Does that come out of their €85K budget?

Also: "suitable for scientific publication" is not a standard we've built for.
What does an audit log look like when the auditor is Nature's peer reviewers?
```

```
you [11:14]
The pipeline is a saga.

Step 1 (classical preprocessing) → Step 2 (QPU simulation) → Step 3 (classical postprocessing)

If Step 2 fails, we can't compensate for Step 1 (it's already done, can't un-run it).
But we CAN checkpoint Step 2's progress — how many circuits ran before failure.
And we can resume from that checkpoint, not from Step 1.

This is the "too late to compensate" problem from the saga pattern.
The recovery action isn't undo — it's resume.

For scientific audit: think of it like a lab notebook.
Every reagent (input parameter), every instrument setting (QPU calibration state),
every measurement (shot outcomes), every version of every tool (software versions).
The audit log is a reproducibility record, not a security record.
Different audience, same principle: tamper-evident, append-only, complete.
```

---

**[Incident bridge log — #pipeline-incidents, Tuesday 3:47pm]**

```
[15:47] ALERT: NordMaterials LithiumX-7 simulation — QPU-3 connection timeout
[15:47] Job ID: nm-lithiumx7-run001
[15:47] Pipeline stage: QPU_SIMULATION (step 2 of 3)
[15:47] Time elapsed in QPU step: 1h 43m 12s (estimated completion: 17m remaining)
[15:47] Circuits completed: 847 / 1,200
[15:47] Classical preprocessing: COMPLETE (checkpoint saved)
[15:47] Status: PAUSED — awaiting QPU recovery or manual intervention

[15:49] dr.hassan.ali: QPU-3 vendor status page shows "degraded performance"
         Estimated recovery: 30-60 minutes

[15:51] you: Pipeline is checkpointed at circuit 847.
         If QPU-3 recovers, we can resume from circuit 848.
         If QPU-3 doesn't recover within 2 hours, we have a decision:
         a) Re-queue remaining 353 circuits on QPU-1 (IBM Eagle, higher error rate)
         b) Wait for QPU-3 (could be hours)
         c) Notify NordMaterials and let them decide

[15:52] yuki.tanaka: NordMaterials chose IonQ specifically for lower error rate.
         They need < 0.5% variance for their publication.
         IBM Eagle would give ~2.5% variance on these circuits.
         Can we run the remaining 353 circuits on IBM Eagle and
         merge the results with the IonQ results for circuits 1-847?

[15:55] dr.hassan.ali: The problem is statistical compatibility.
         IonQ results and IBM Eagle results have different error profiles.
         Mixing them in a single simulation may introduce systematic bias.
         For a Nature paper, that would be a problem.

[16:02] you: Decision point.
         Option A: Wait for IonQ recovery (up to 2 hours). Delay run completion.
         Option B: Fail the job. NordMaterials re-runs from QPU step on IonQ.
                   They don't lose preprocessing. Budget impact: 1 of 3 runs consumed.
         Option C: Proceed on IBM Eagle for 353 circuits, flag mixing in the audit log.
                   Let NordMaterials decide if mixed hardware is acceptable for their paper.

[16:08] yuki.tanaka: I'm calling Dr. Solberg. Stand by.

[16:22] yuki.tanaka: NordMaterials chooses Option A. Wait for IonQ recovery.
         They have the time. They do NOT want mixed hardware results.
         Mark the job SUSPENDED pending QPU-3 recovery.

[16:24] PIPELINE STATUS: nm-lithiumx7-run001 SUSPENDED — QPU_SIMULATION paused at circuit 847/1200
        Checkpoint valid. Classical preprocessing result cached.
        Awaiting: QPU-3 recovery signal from vendor webhook.

[17:41] ALERT: QPU-3 status: OPERATIONAL
[17:43] PIPELINE RESUMED: nm-lithiumx7-run001 — QPU_SIMULATION resuming from circuit 848
[19:58] PIPELINE COMPLETE: nm-lithiumx7-run001 — all stages complete
        Total wall-clock time: 8h 11m (including 1h 54m suspension)
        Circuits: 1,200/1,200 complete
        Postprocessing: COMPLETE
        Scientific audit log: generated
```

---

### Problem Statement

QuantumEdge's hybrid pipeline system must orchestrate multi-step, long-running quantum-classical pipelines where classical preprocessing, QPU execution, and classical postprocessing are tightly coupled steps. The system must handle QPU hardware failures mid-execution with checkpoint/resume semantics, produce scientific audit logs suitable for peer-reviewed publication, and manage the cost accounting of multi-step pipelines where partial failure doesn't necessarily mean full restart.

### Explicit Requirements

1. **Multi-step pipeline definition**: customers define pipelines as a sequence of stages (classical CPU, QPU, classical GPU), with compute requirements and dependencies per stage
2. **Stage-level checkpointing**: each stage persists its completion state; QPU stage checkpoints at the circuit level (not just completion)
3. **Resume on failure**: if a QPU stage fails mid-execution, resume from the last checkpointed circuit, not from pipeline start
4. **QPU hardware suspension**: handle QPU vendor degraded/offline events — suspend pipeline, resume when QPU recovers (via vendor webhook)
5. **Scientific audit log**: per-pipeline audit record including: all input parameters, software versions, QPU calibration state at execution time, all circuit measurement outcomes, postprocessing method and version
6. **Audit log format**: structured, append-only, human-readable, exportable (JSON-LD or similar for academic use)
7. **Compute resource allocation**: pre-allocate classical compute (CPU/GPU nodes) for each stage; release after stage completion
8. **Cross-hardware compatibility flagging**: if QPU failover requires switching hardware vendor (IonQ → IBM Eagle), flag the result as mixed-hardware and alert the customer before proceeding
9. **Pipeline cost accounting**: per-stage cost, total pipeline cost, cost impact of suspension/resume vs restart

### Hidden Requirements

- **Hint: re-read the incident bridge log at 15:55.** Dr. Hassan raises "statistical compatibility" — mixing IonQ and IBM Eagle results may introduce systematic bias. But the platform can't make this scientific judgment. What's the right API design here? Should the platform hard-block mixed-hardware runs, or surface a warning and require explicit customer acknowledgment?

- **Hint: re-read Dr. Solberg's email about the EU Horizon grant.** EU-funded research has specific data residency requirements under the European Open Science Cloud (EOSC) framework. Scientific data produced under Horizon grants may need to be stored in EU infrastructure. Is QuantumEdge's Amsterdam co-location sufficient, or does the scientific audit log need a specific data residency guarantee?

- **Hint: re-read the 7-hour pipeline estimate.** "Three simulation runs (in case of parameter adjustment)" — NordMaterials plans to tune parameters between runs. Run 2's parameters depend on Run 1's results. This is an iterative simulation workflow, not three independent runs. The pipeline system should support parameterized re-runs that inherit the previous run's checkpointed preprocessing output if inputs haven't changed.

- **Hint: the incident log shows QPU-3 failure at 1h 43m, 17 minutes from completion.** What is the SLA for QPU vendor notification? If QuantumEdge doesn't proactively notify NordMaterials about the suspension, but the 7-hour pipeline turns into a 9-hour pipeline, that may violate an implicit SLA. Customer notification on pipeline state changes is a system requirement hidden in the professional services expectation.

### Constraints

- **Classical preprocessing**: 8 CPU nodes, 256GB RAM, ~1 hour runtime
- **QPU step**: IonQ QPU-3, 23 qubits, 1,200 circuits, ~2 hours estimated, ~200µs/gate
- **Classical postprocessing**: 4 GPU nodes, ~4 hours runtime
- **IonQ session failure rate**: ~2% per session
- **QPU-3 recovery SLA from vendor**: "best effort, typically 30-120 minutes"
- **Scientific audit log retention**: minimum 10 years (grant requirement)
- **Budget**: €85,000 for 3 runs; each run ~€28,333 budget
- **QPU cost per full run**: ~€12,000 at current rates (120 QPU-hours)
- **Data residency**: Amsterdam co-location; EU data sovereignty applies (EOSC guidance)
- **Audit log format**: must be machine-readable AND human-readable; referenced in Nature supplementary materials
- **Concurrent hybrid pipelines**: up to 5 simultaneous (limited by GPU node allocation)

### Your Task

Design the hybrid pipeline orchestration system. Cover: pipeline definition API, stage lifecycle management, circuit-level checkpointing for QPU stages, QPU vendor webhook integration (for recovery signals), scientific audit log schema and generation, mixed-hardware detection and flagging, and the compute resource allocation model.

### Deliverables

- [ ] Mermaid architecture diagram: pipeline intake → resource allocator → stage executor (classical-CPU / QPU / classical-GPU) → checkpoint manager → audit log writer → result packager
- [ ] Database schema: pipelines, pipeline_stages, stage_checkpoints, qpu_circuit_executions, pipeline_audit_events, resource_allocations (with column types, indexes)
- [ ] Checkpoint/resume design: what is checkpointed per QPU stage, how resume is triggered (webhook or manual), what happens to allocated resources during suspension
- [ ] Scientific audit log schema: what fields are required, how it's generated, export format
- [ ] Scaling estimation (show math): 5 concurrent 7-hour pipelines × resource allocation per pipeline = GPU/CPU node utilization; QPU-3 saturation point
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Circuit-level checkpointing (high overhead, fine resume granularity) vs batch-level checkpointing (lower overhead, coarser resume)
  - Hard-block mixed-hardware runs vs warn-and-proceed (with customer consent)
  - Eager resource allocation (hold nodes for duration) vs lazy allocation (release and re-acquire per stage)
- [ ] Cost modeling: per-pipeline cost for NordMaterials; cost impact of QPU suspension (extended node hold time)
- [ ] Capacity planning: max concurrent hybrid pipelines at current compute; threshold for additional GPU nodes

### Diagram Format

Mermaid syntax.
