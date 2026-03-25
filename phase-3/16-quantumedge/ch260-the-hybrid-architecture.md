---
**Title**: QuantumEdge — Chapter 260: The Hybrid Architecture
**Level**: Staff
**Difficulty**: 8
**Tags**: #distributed #scheduling #compute #api-design #capacity-planning
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 147 (VertexCloud platform engineering), Ch. 56 (LuminaryAI GPU scheduling), Ch. 265
**Exercise Type**: System Design
---

### Story Context

**[Day 1, 9:14am — QuantumEdge Office, Amsterdam]**

Dr. Yuki Tanaka has the whiteboard markers out before you've taken your coat off.

"Coffee first?" she asks, already uncapping a red marker.

"Sure," you say, and she ignores this, drawing three rectangles labeled QPU-1, QPU-2, QPU-3.

The office is small — six desks in a converted canal house, two of them occupied by PhD students who double as customer support engineers. The windows face the water. The QPUs are not here; they are in a rack in Helsinki, accessed over a leased fiber connection.

"Here's the situation," Dr. Tanaka says. She has the clipped, dense speaking style of someone who doesn't waste words because she genuinely doesn't have time. "Three QPUs. Each QPU is a 16-qubit superconducting chip. Coherence time is 100 microseconds. Clock cycles run at 50 nanoseconds. Each circuit has to complete within the coherence window or the quantum state decoheres and the result is garbage."

She draws a box labeled GPU Cluster — 10 nodes, 8×A100.

"GPU nodes are for classical simulation — circuits too small for QPU overhead, hybrid algorithms where classical and quantum steps interleave, postprocessing. Right now we run Qiskit on the GPU nodes, can simulate up to 30 qubits with full state vector. Anything bigger, or anything that genuinely needs quantum advantage, goes to QPU."

She steps back and looks at the whiteboard.

"Fifty customers. Mix of pharma, finance, materials science. Each customer has a different usage pattern. Pharma wants to batch 10,000 variational quantum eigensolver circuits for molecular simulation — short circuits, embarrassingly parallel. Finance wants to run portfolio optimization — long-running circuits, high qubit count, monopolizes a QPU for hours. Materials science wants end-to-end hybrid pipelines: classical preprocessing, then QPU, then classical postprocessing."

She taps the laptop on her desk. It's open to a Python file called `scheduler.py`. You glance at it. It is 340 lines. It uses `time.sleep`.

"That's the job scheduler," she says. "It processes a JSON file of submitted jobs. I run it manually when I see an email from a customer. It has been running on my laptop since March. We have a pharma customer — Helix Pharma, their CTO is a friend from grad school — who is about to submit 10,000 circuits. On Friday. Three days from now."

She looks at you directly.

"I need a real system. I need you to tell me what that looks like. What's the architecture, how do we route jobs to the right compute, how do we handle the QPU constraints — coherence time, error rates, qubit count — how do we give customers fair access. And I need you to have something running by Thursday so Helix doesn't miss their molecular dynamics deadline."

---

**[Slack DM — Dr. Yuki Tanaka → You, 10:02am]**

```
yuki.tanaka [10:02]
A few things I didn't mention:

QPU-1 and QPU-2 are the same hardware generation (IBM Eagle, 16 qubits, ~1% gate error rate).
QPU-3 is a different vendor (IonQ, trapped ion, 23 qubits, 0.5% gate error, 10x slower gate time).
They have different strengths. Some circuits run better on one than the other.

FinancialFirst (our largest customer, $400K/yr) has been monopolizing QPU-1 for the last two weeks.
Helix Pharma has been waiting 4 days for QPU access.
I've been manually swapping the queue when I notice.

Also: our QPU vendor contract is metered. We pay per QPU-shot (each execution of a circuit).
Current rate: $0.0003 per shot.
Helix's 10,000 circuits × 1,000 shots each = 10 million shots = $3,000.
That cost comes out of our margin, not the customer's.

So: the scheduler is also a cost control problem.
```

---

**[Slack DM — You → Dr. Yuki Tanaka, 10:47am]**

```
you [10:47]
Quick questions before I design anything:

1. Can customers specify which compute type they want, or should routing be automatic?
2. Is there a QPU type affinity — some circuits work better on IonQ vs IBM Eagle?
3. Fair-use: first-in-first-out, or weighted by subscription tier?
4. What's the SLA on job completion? Do customers expect time estimates?
```

**[Slack DM — Dr. Yuki Tanaka → You, 11:03am]**

```
yuki.tanaka [11:03]
1. Automatic routing preferred. Customers don't want to think about hardware.
   But enterprise customers want override capability.
2. Yes. Variational algorithms → Eagle (faster gate time). Trapped-ion → IonQ (lower error, needed for deep circuits).
   We should route automatically based on circuit characteristics.
3. Fair-use by subscription tier. But right now all 50 customers are on the same tier.
   This will change when we launch tiered pricing in Q2.
4. No current SLA. But Helix's CTO asked me yesterday what the estimated completion time was.
   I said "soon". We need actual estimates.

Also: I should have told you this earlier.
QPU vendor gives us a 4-hour maintenance window every Sunday at 2am CET.
All three QPUs go offline simultaneously.
We have to drain the queue before that window or jobs get interrupted mid-circuit.
```

---

### Problem Statement

QuantumEdge needs to replace Dr. Tanaka's Python laptop-scheduler with a production-grade hybrid quantum-classical job scheduling system. The system must route circuit submissions to the appropriate compute resource (QPU or GPU), enforce fair-use scheduling across 50 enterprise customers, respect the hard physical constraints of quantum hardware (coherence time, qubit count, gate error rates), and provide customers with job status and completion time estimates.

The system must be operational enough to handle Helix Pharma's 10,000-circuit submission arriving in 72 hours — without blocking FinancialFirst's ongoing work, monopolizing QPU resources unfairly, or creating a $3,000 cost overrun that wipes the month's margin.

### Explicit Requirements

1. Accept circuit submissions via an API (REST initially; gRPC extension planned)
2. Route circuits automatically to QPU or GPU based on circuit characteristics (qubit count, gate depth, circuit type)
3. Support QPU-type affinity routing (IBM Eagle vs IonQ trapped ion)
4. Enforce per-customer fair-use scheduling (prevent monopolization)
5. Provide job status, estimated completion time, and position-in-queue
6. Handle QPU vendor maintenance windows — drain queue gracefully before 2am CET Sundays
7. Track cost per job (shots × $0.0003/shot) for margin monitoring
8. Handle QPU coherence time constraint — circuits must complete within 100µs window

### Hidden Requirements

- **Hint: re-read Dr. Tanaka's Slack at 11:03am.** She mentions "tiered pricing in Q2." What does that mean for the scheduler's fair-use algorithm? If you build FIFO now, how painful is migrating to weighted-fair-queuing in 3 months?

- **Hint: re-read the QPU vendor contract detail.** The cost is per-shot, not per-circuit. A customer could submit a circuit with `shots=10000` vs `shots=100`. Should shot count be a scheduling input? What happens to cost estimation if it isn't?

- **Hint: Dr. Tanaka mentioned "circuit type → compute type routing" casually.** But who classifies the circuit? The customer, or the system? What happens if a customer mislabels a 30-qubit circuit as a GPU job? The GPU simulator will either fail or produce wrong results silently.

- **Hint: re-read the maintenance window detail.** "All three QPUs go offline simultaneously." What does the scheduler do during the 4-hour window? Jobs queued for QPU but not yet admitted — what happens to them?

### Constraints

- **Compute**: 3 QPUs (QPU-1: IBM Eagle 16q, QPU-2: IBM Eagle 16q, QPU-3: IonQ 23q), 10 GPU nodes (8×A100 each)
- **QPU coherence time**: 100µs — circuit must complete within this window
- **QPU gate time**: IBM Eagle ~50ns/gate; IonQ ~200µs/gate (10x slower but lower error)
- **QPU error rate**: IBM Eagle ~1% 2-qubit gate error; IonQ ~0.5%
- **Shot cost**: $0.0003/shot via vendor API
- **Customers**: 50 enterprise, all same tier currently
- **Helix Pharma submission**: 10,000 circuits, VQE algorithm, ~20 qubits each, 1,000 shots each = 10M shots = $3,000
- **FinancialFirst**: 1 long-running circuit, 16 qubits, portfolio optimization, ~4 QPU-hours
- **Maintenance window**: Every Sunday 2am CET, 4 hours, all QPUs offline
- **Team**: 2 engineers (you + Dr. Tanaka), 2 PhD students part-time
- **Timeline**: Something running by Thursday (72 hours)

### Your Task

Design the hybrid quantum-classical job scheduling and routing system for QuantumEdge. Your design must cover: the job submission API, circuit classification and routing logic, the scheduling algorithm (including fair-use enforcement), QPU assignment, maintenance window handling, cost tracking, and customer-facing status/ETA endpoints.

### Deliverables

- [ ] Mermaid architecture diagram showing job submission → classification → routing → QPU/GPU execution → result storage
- [ ] Database schema for jobs, circuits, customers, cost ledger (with column types and indexes)
- [ ] Scheduling algorithm design: explain the fair-use approach and how it evolves to weighted-fair-queuing for Q2 tiered pricing
- [ ] Scaling estimation (show math): 50 customers × average job size → QPU utilization %, GPU utilization %, queue depth at peak
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Automatic routing vs customer-controlled routing
  - FIFO vs weighted-fair-queue (migration path)
  - Job pre-validation vs admit-then-validate
- [ ] Cost modeling: $X/month at current 50-customer load; projected cost at 200 customers
- [ ] Capacity planning: QPU utilization at 50 customers vs 200 customers — when do you need QPU-4?
- [ ] Maintenance window drain strategy: pseudocode for graceful queue drain before 2am CET

### Diagram Format

Mermaid syntax (renders in GitHub Issues).

```
Example node structure:
graph TD
    A[Circuit Submission API] --> B[Circuit Classifier]
    B --> C{Route Decision}
    C -->|QPU required| D[QPU Queue]
    C -->|GPU sufficient| E[GPU Queue]
```
