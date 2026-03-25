---
**Title**: QuantumEdge — Chapter 263: Error Mitigation at Scale
**Level**: Staff
**Difficulty**: 9
**Tags**: #performance #cost-engineering #distributed #ml-systems
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 262, Ch. 265, Ch. 108 (TeleNova QoS), Ch. 56 (LuminaryAI model serving)
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**[Email chain — Subject: Unacceptable Result Variance — Portfolio Optimization Circuits]**

```
From: Tobias Kramer <t.kramer@financialfirst.de>
To: yuki.tanaka@quantumedge.io, dr.hassan.ali@quantumedge.io
Date: Thursday, 7:52am
Subject: Unacceptable Result Variance — Portfolio Optimization Circuits

Yuki, Hassan —

We've completed our initial analysis of the portfolio optimization results
from last week's QPU runs.

The variance in our risk model outputs is 14.7%.
Our models require variance < 1% to be used in live trading decisions.

Our classical Monte Carlo simulations produce < 0.1% variance.
Our expectation was that QPU results would be comparable or superior,
given the theoretical advantages of quantum optimization.

Instead, we are seeing results that are statistically indistinguishable from random.

Our quant team's conclusion: QPU hardware error rates are corrupting
our circuit results. We are paying for quantum compute and receiving noise.

We are prepared to terminate our contract ($400K/yr) if this is not resolved
within 30 days.

Tobias Kramer
Head of Quantitative Technology, FinancialFirst
```

---

**[Slack DM — Dr. Yuki Tanaka → You, Thursday 8:15am]**

```
yuki.tanaka [8:15]
We need to talk about error mitigation.

The short version:
- IBM Eagle has ~1% 2-qubit gate error rate
- FinancialFirst's portfolio optimization uses 16 qubits, 800+ gates
- At 1% error rate per gate, 800 gates = ~99.9% probability of at least one error
- Every single circuit run is corrupted

Error mitigation is a classical post-processing technique.
You run the circuit many times at different noise levels,
extrapolate to the "zero noise" result.
Standard method: Zero-Noise Extrapolation (ZNE).

The catch: ZNE requires running each circuit 3-5x at different noise levels.
FinancialFirst runs 1,000-shot circuits.
ZNE would require 3,000-5,000 shots per circuit.
3x-5x QPU cost. 3x-5x time.

The question I need you to answer:
At what point does error mitigation cost more than upgrading to better hardware?

And: should error mitigation be a customer option or a platform default?
```

---

**[Slack — #engineering, Thursday 9:30am]**

```
dr.hassan.ali [9:30]
I've been running numbers on error mitigation methods. Quick summary:

1. Zero-Noise Extrapolation (ZNE)
   Run circuit at noise levels [1x, 1.5x, 2x], extrapolate to 0x.
   Requires 3x shot count. Variance reduction: ~70%.
   Cost multiplier: 3x QPU cost.
   FinancialFirst: 1% variance down to ~0.3% variance.
   Still above their 1% threshold? No, this gets them there.

2. Probabilistic Error Cancellation (PEC)
   Insert inverse noise operations. Requires circuit overhead.
   Requires 10-100x shot count depending on circuit depth.
   Near-exact noise removal. Variance: < 0.1%.
   Cost multiplier: 10-100x QPU cost.
   FinancialFirst: would cost $150K/month at their usage level. Unsustainable.

3. Symmetry Verification (SV)
   Post-select shots that satisfy known symmetries of the problem.
   Discards ~50-80% of shots as "noisy."
   Net: same accuracy at 2x cost (to compensate for discarded shots).
   Works only for problems with known symmetries (portfolio optimization qualifies).
   Cost multiplier: 2x.

FinancialFirst's specific problem:
- ZNE (3x cost): gets variance to 0.3%. Meets their threshold. Cost increase: 3x → $1.2M/yr equivalent.
- SV (2x cost): gets variance to ~0.5%. Meets their threshold. Cost: 2x → $800K/yr equivalent.

They're paying $400K/yr. Neither option is within their current contract.
```

```
yuki.tanaka [9:44]
So we have three options:
A. Offer error mitigation as a paid add-on, repriced contracts
B. Build error mitigation into the platform at cost, absorb the margin hit
C. Tell FinancialFirst their use case requires better hardware and
   the timeline for QPU improvement is 12-18 months

None of these are purely technical decisions.
What's the platform architecture that keeps all three options open?
```

```
you [9:51]
The platform architecture question is: error mitigation as a pluggable pipeline stage.
Customers opt in to a mitigation strategy. Platform calculates actual cost
before running with mitigation enabled. Customer approves or rejects.

The system needs to:
1. Estimate cost of running with vs without mitigation (before execution)
2. Execute mitigation strategy at the circuit batch level
3. Apply classical postprocessing (ZNE extrapolation, SV post-selection)
4. Return both the raw results AND the mitigated results
5. Track mitigation cost separately in the billing ledger

This also gives us data to answer Yuki's question:
at what error rate threshold does mitigation become more expensive than hardware?
We can build that cost-crossover model from real execution data.
```

---

### Problem Statement

QuantumEdge needs to build an error mitigation pipeline that can be applied to customer circuit jobs as a configurable layer. The system must support multiple mitigation strategies (ZNE, SV — and potentially PEC for future hardware), estimate the cost multiplier before execution, track mitigation overhead separately in the billing system, and provide the data infrastructure to answer a strategic question: at what QPU error rate does the cost of mitigation exceed the cost of a hardware upgrade?

FinancialFirst's contract renewal depends on variance < 1%. The architecture must support this without committing to an unsustainable cost model.

### Explicit Requirements

1. **Pluggable mitigation pipeline**: error mitigation is an opt-in pipeline stage configured per job, not per platform
2. **Pre-execution cost estimate**: before running a job with mitigation, calculate and return the cost multiplier (shots × mitigation factor × $0.0003)
3. **Customer approval flow**: present cost estimate; customer confirms or cancels before QPU budget is spent
4. **ZNE implementation**: run circuits at noise levels [1x, 1.5x, 2x], pass raw results to classical extrapolation postprocessor
5. **Symmetry Verification (SV) implementation**: flag shots that violate problem symmetries, post-select valid shots, compensate by scaling shot count
6. **Dual result storage**: store both raw results and mitigated results per job
7. **Mitigation cost tracking**: separate billing line item for mitigation overhead
8. **Cost-crossover dashboard**: aggregate data on mitigation cost vs error rate; project hardware upgrade crossover point
9. **Strategy selection recommendation**: given circuit profile and customer variance requirement, recommend the cheapest mitigation strategy that meets variance target

### Hidden Requirements

- **Hint: re-read Dr. Hassan's analysis carefully.** PEC requires "10-100x shot count depending on circuit depth." The range is an order of magnitude. Who estimates where in that range a given circuit falls? If the estimate is wrong and a customer's job costs 100x instead of 10x, whose liability is it?

- **Hint: re-read the email from Tobias Kramer.** He says "results statistically indistinguishable from random." This is a specific claim. The platform should be able to statistically validate circuit results against theoretical expectations for known problem classes (portfolio optimization has known mathematical structure). This is the "result quality monitoring" problem — detecting when QPU error rates have degraded.

- **Hint: re-read Yuki's question about hardware upgrade crossover.** To answer this, you need to know: the cost to improve hardware (QPU replacement/upgrade price, vendor negotiation), the current mitigation cost, and the error rate of the new hardware. This is a FinOps problem, not just a physics problem. What data does the platform need to collect to make this decision rigorously?

- **Hint: re-read the contract detail — FinancialFirst pays $400K/yr.** If ZNE triples their QPU cost, the implied cost to QuantumEdge is $1.2M/yr in QPU shots for a $400K/yr customer. This is an inverted unit economics problem. The mitigation system must surface this to the business before executing, not after.

### Constraints

- **IBM Eagle error rate**: ~1% per 2-qubit gate
- **FinancialFirst circuits**: 16 qubits, 800+ gates, 1,000 shots/circuit
- **ZNE cost multiplier**: 3x shot count
- **SV cost multiplier**: ~2x shot count (50-80% shot discard rate)
- **PEC cost multiplier**: 10-100x (circuit depth dependent)
- **FinancialFirst target variance**: < 1%
- **ZNE achievable variance**: ~0.3% for their circuit class
- **SV achievable variance**: ~0.5% for their circuit class
- **Current QPU cost**: $0.0003/shot
- **Mitigation classical compute**: ZNE postprocessing ~100ms/circuit on GPU (not QPU cost)
- **GPU cluster**: 10 nodes × 8 A100 — mitigation postprocessing runs here
- **FinancialFirst contract**: $400K/yr; renewal at risk

### Your Task

Design the error mitigation pipeline system. Cover: the pluggable mitigation architecture, pre-execution cost estimation, customer approval flow, ZNE and SV execution at the circuit-batch level, classical postprocessing pipeline, dual result storage, billing integration, and the cost-crossover analytics system.

Include a written analysis: given current numbers, what is the FinancialFirst unit economics decision? Should QuantumEdge offer ZNE at cost, at a premium, or refuse until better hardware is available?

### Deliverables

- [ ] Mermaid architecture diagram: job submission → mitigation strategy selection → pre-execution cost estimate → customer approval → QPU execution (with mitigation) → classical postprocessor → result storage → billing
- [ ] Database schema: mitigation_jobs, circuit_executions_with_mitigation, mitigation_results, mitigation_cost_ledger, hardware_error_rate_history (with column types and indexes)
- [ ] ZNE pipeline design: how noise levels are set, how raw results per noise level are stored, how extrapolation is computed classically
- [ ] Scaling estimation (show math): FinancialFirst 1,000-shot circuits × ZNE 3x = 3,000 shots; monthly volume; monthly QPU cost with mitigation
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - ZNE (3x cost, 0.3% variance) vs SV (2x cost, 0.5% variance) for FinancialFirst's use case
  - Platform absorbs mitigation cost vs customer pays per-mitigation-run
  - Build generic mitigation framework now vs hard-code ZNE and refactor later
- [ ] Unit economics analysis: FinancialFirst at current $400K/yr contract vs true cost with ZNE; decision recommendation
- [ ] Cost-crossover model: at what QPU error rate does mitigation cost = hardware upgrade cost? Show the math.
- [ ] Capacity planning: GPU cluster utilization for classical postprocessing at 50 customers with mitigation vs 200 customers

### Diagram Format

Mermaid syntax.
