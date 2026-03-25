---
**Title**: QuantumEdge — Chapter 261: The Quantum API Gateway and Auth (Auth System: 6th Evolution)
**Level**: Staff
**Difficulty**: 9
**Tags**: #auth #api-design #rate-limiting #compliance #security
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 6 (MeridianHealth auth), Ch. 30 (CloudStack auth), Ch. 61 (SkyRoute auth), Ch. 76 (NexaCare auth), Ch. 109 (TeleNova auth), Ch. 260
**Exercise Type**: System Design
---

### Story Context

**[Slack — #engineering, Tuesday 9:47am]**

```
dr.hassan.ali [9:47]
@everyone heads up — FinancialFirst just submitted their monthly batch.
1 circuit, 16 qubits, 2,847 gates, shots=500000.

Their monthly QPU quota is 100 QPU-hours.
This circuit alone will take approximately 47 QPU-hours at 1000µs/circuit × 500K shots.
Plus it has a depth of 2,847 gates × 50ns = 142µs.
That's already over coherence time. The circuit will decohere mid-run.
It will produce garbage results and they won't know why.

The submission went straight to the queue.
Nothing stopped it.
```

```
yuki.tanaka [9:51]
How long has it been in queue?
```

```
dr.hassan.ali [9:53]
18 minutes. QPU-1 is currently occupied by a Helix Pharma job.
If it gets scheduled next, it'll monopolize QPU-1 for 2 days
and produce results that are scientifically invalid.
```

```
yuki.tanaka [9:54]
@you can you look at this?
```

---

**[Slack DM — You → Dr. Yuki Tanaka, 10:02am]**

```
you [10:02]
Pulling the submission now. A few issues:

1. The circuit has 2,847 gates × 50ns = 142.35µs execution time.
   Coherence window is 100µs.
   This circuit is physically impossible to run correctly.
   We need pre-submission validation to reject circuits that exceed coherence time.

2. 500,000 shots × $0.0003 = $150 for a single job.
   Their monthly quota is 100 QPU-hours.
   At 142µs/circuit, 500K shots = ~20 QPU-hours — half their monthly allocation.
   No warning to the customer before submit.

3. I looked at their other submissions this month.
   They've already used 87 QPU-hours.
   This submission would put them at 107 QPU-hours — 7% over quota.
   We'd either have to cut them off mid-job or eat the overage cost.

4. Meanwhile: Helix Pharma has 9,847 circuits still in queue.
   Axiom Materials has been waiting 3 days.
   FinancialFirst's runaway job would push everything back 47 hours.
```

```
yuki.tanaka [10:11]
This is exactly why we need a real auth layer.
Not just API keys.
Resource-based authorization.
The system needs to know: what is this customer allowed to consume?
And it needs to enforce that *before* the circuit enters the queue.
Not after.
```

---

**[Email chain — Subject: QuantumEdge API Access — FinancialFirst Integration]**

```
From: Tobias Kramer <t.kramer@financialfirst.de>
To: yuki.tanaka@quantumedge.io
Date: Tuesday, 9:24am
Subject: QuantumEdge API Access — FinancialFirst Integration

Yuki —

Our quant team has been integrating directly against the QuantumEdge API.
We've noticed a few things:

1. There's no documentation on QPU time quotas or how they're calculated.
   We've been assuming "unlimited" since nothing in the contract specifies units.
2. Our circuit compiler sometimes generates circuits above your hardware gate limits.
   We'd prefer to catch this at submission time, not after waiting in queue.
3. Can we get a "dry-run" endpoint? Submit a circuit, get back resource estimate and
   quota check, without actually queueing the job.
4. Is there a way to see our current quota usage before submitting?

We're spending about €140K/quarter on QuantumEdge.
We expect API-grade reliability and tooling.

Best,
Tobias Kramer
Head of Quantitative Technology, FinancialFirst
```

```
From: yuki.tanaka@quantumedge.io
To: Tobias Kramer <t.kramer@financialfirst.de>
Date: Tuesday, 11:15am
Subject: RE: QuantumEdge API Access — FinancialFirst Integration

Tobias —

All valid points. We're building exactly this this week.
Dry-run endpoint and pre-submission validation are on the list.
Quota display endpoint will be available tomorrow.

On the quota question: your contract specifies "QPU access as available."
We're moving to metered QPU-hours in Q2.
We'll give all customers 30 days notice before the billing change.

Yuki
```

---

**[1:1 transcript — You and Dr. Tanaka, Tuesday 2pm]**

**Dr. Tanaka**: "Walk me through what you're thinking for auth."

**You**: "We've built auth systems before at companies I've worked at. Each time, the scope grew. At MeridianHealth it was HIPAA SSO and MFA. At CloudStack it was SAML federation for enterprise tenants. At SkyRoute it was adversarial — agent-on-behalf-of OAuth2 delegation, behavioral detection. But all of those were identity questions: who are you? Can you access this resource?"

**Dr. Tanaka**: "And here?"

**You**: "Here the interesting question isn't who you are. It's what you're allowed to consume. Resource-based authorization. You've authenticated fine — you're Tobias Kramer at FinancialFirst. The question is: is this specific circuit — with this gate count, this qubit count, this shot count — within your contracted resource envelope? And can we answer that question in under 100 milliseconds, before we've spent any QPU budget?"

**Dr. Tanaka**: "The resource envelope is multidimensional."

**You**: "Exactly. It's not one number. It's a vector: QPU-hours per month, circuit depth limit (physical constraint — we can't override coherence time), qubit count limit (you can't run 17 qubits on a 16-qubit chip), shot count per job (cost control), daily burst limit (fairness), concurrent job limit. The auth decision is: does this circuit vector fit within the customer's resource vector?"

**Dr. Tanaka**: "And if we add tiered pricing in Q2?"

**You**: "The resource vector is just parameterized by tier. Basic tier gets {100 QPU-hours/month, depth≤500, shots≤10K/job}. Enterprise tier gets {unlimited QPU-hours, depth≤2000, priority queue access}. The policy engine doesn't change — just the policy data."

**Dr. Tanaka**: "What about the dry-run endpoint Tobias asked about?"

**You**: "That's free to implement once the authorization layer exists. `POST /circuits/validate` — runs the full auth check, returns resource estimate, quota remaining, coherence time check, routing recommendation. No queue admission. No QPU cost. We should build this first, actually. It's the thing that would have caught today's problem."

---

### Problem Statement

QuantumEdge needs a quantum API gateway with resource-based authorization — the sixth evolution of the auth system pattern that has appeared across MeridianHealth, CloudStack, SkyRoute, NexaCare, and TeleNova. Unlike all previous auth evolutions (identity, federation, delegation, clinical trial access, IoT device auth), this evolution asks a fundamentally new question: not "who are you?" but "can your circuit fit within your contracted resource envelope — and can we verify that in under 100ms before admitting it to the QPU queue?"

The system must enforce circuit-level constraints (coherence time, qubit limits), quota constraints (QPU-hours, shots, burst), and provide customers with pre-admission validation and real-time quota visibility — while remaining extensible for Q2 tiered pricing.

### Explicit Requirements

1. **Pre-admission circuit validation**: reject circuits that exceed physical hardware limits (coherence time, max qubit count per QPU type) before queue admission
2. **Resource-based authorization**: enforce per-customer resource envelope (QPU-hours/month, circuit depth limit, max shots/job, concurrent job limit, daily burst limit)
3. **Quota check in < 100ms**: authorization must not add meaningful latency to the submission path
4. **Dry-run endpoint**: `POST /circuits/validate` — full auth + resource estimate, no queue admission
5. **Quota dashboard endpoint**: `GET /customers/{id}/quota` — real-time usage, remaining budget, reset date
6. **Graceful overage handling**: warn at 80% quota, soft-block at 100%, allow configurable overage allowance for enterprise contracts
7. **API key authentication**: per-customer API keys with scope (submit, read-only, admin)
8. **Audit log**: all submission attempts (success + rejection), quota check results, resource consumed per job

### Hidden Requirements

- **Hint: re-read Tobias Kramer's email carefully.** He says "Our circuit compiler sometimes generates circuits above your hardware gate limits." What does this imply about the relationship between QuantumEdge's API and customer tooling? Should the API expose hardware limits programmatically so compiler toolchains can enforce them client-side? What's the API design for that?

- **Hint: re-read the 1:1 transcript.** Dr. Tanaka asks about tiered pricing in Q2. You design the resource vector as parameterized by tier. But how do you handle a customer mid-month when their tier changes? The quota resets monthly — but if the tier changes on the 15th, do they get the new tier's quota for the remaining half-month, prorated, or from next month?

- **Hint: re-read Dr. Hassan Ali's Slack message.** He says the circuit "will produce garbage results and they won't know why." This is the deeper problem: silent failure. The circuit runs, produces a result, but the result is scientifically invalid because decoherence occurred mid-execution. The authorization layer must reject coherence-violating circuits, but what about circuits that are borderline? What's the policy?

- **Hint: the email chain mentions "QPU access as available" in the original contract.** This creates a legal ambiguity: if the new metered billing applies to existing customers, does it constitute a contract modification requiring consent? The auth system needs to support both billing models simultaneously during a transition period.

### Constraints

- **Physical hardware limits**:
  - IBM Eagle (QPU-1, QPU-2): 16 qubits max, ~2,000 gates max (at 50ns/gate = 100µs), ~1% 2-qubit gate error
  - IonQ trapped ion (QPU-3): 23 qubits max, ~500 gates max (at 200µs/gate = 100ms — much deeper coherence), ~0.5% gate error
- **Current quota model**: 100 QPU-hours/month per customer (all tiers identical)
- **Q2 tiered pricing**: Basic (100h/month), Professional (500h/month, priority queue), Enterprise (custom SLA)
- **API key model**: per-customer, per-environment (prod/dev/sandbox)
- **Quota reset**: 1st of each calendar month, UTC
- **Shot cost**: $0.0003/shot — quota check must estimate cost before admission
- **Auth latency SLA**: < 100ms for quota check (customers call dry-run endpoint in CI pipelines)
- **Concurrent customers**: 50 today, planning for 500 in 18 months
- **Audit retention**: 7 years (enterprise financial customers have audit requirements)
- **Team**: 2 engineers, 72-hour initial build

### Your Task

Design the quantum API gateway and resource-based authorization system. Your design must cover: the API surface (submission, validation/dry-run, quota, hardware limits discovery), the authorization policy engine (circuit vector vs resource envelope), quota storage and atomic decrement, overage handling, audit logging, and the migration path to Q2 tiered pricing.

This is the sixth evolution of the auth pattern. Include a brief section at the end: what has changed about auth from MeridianHealth to QuantumEdge? What's the through-line across all 6 evolutions?

### Deliverables

- [ ] Mermaid architecture diagram: API gateway → auth service → quota store → circuit validator → queue admission
- [ ] Database schema: customers, api_keys, resource_policies, quota_ledger, circuit_submissions, audit_log (with column types, indexes, and quota atomic decrement strategy)
- [ ] Policy engine design: how the resource vector is structured and evaluated; how it extends to tiered pricing
- [ ] Dry-run endpoint contract: request/response schema, what it validates, what it returns
- [ ] Scaling estimation (show math): 50 customers × avg 100 submissions/day = quota check RPS; latency budget breakdown for < 100ms
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Pre-admission validation vs post-admission cancellation
  - Atomic quota decrement (strong consistency) vs optimistic quota (eventual consistency)
  - Per-customer API keys vs OAuth2 client credentials (enterprise asks)
- [ ] Cost modeling: auth service infrastructure cost at 50 customers vs 500 customers
- [ ] The 6-evolution auth pattern summary: 2-3 paragraphs on what changed from identity-first to resource-first auth, and what drove each evolution

### Diagram Format

Mermaid syntax.

```
Example node structure:
graph TD
    A[POST /circuits/submit] --> B[API Gateway]
    B --> C[Auth: API Key Validation]
    C --> D[Resource Authorization Engine]
    D --> E{Circuit within envelope?}
    E -->|Yes| F[Queue Admission]
    E -->|No| G[Reject with reason]
```

### Code Task (TypeScript Sketch)

Write the TypeScript interface for the resource authorization policy and the circuit submission request:

```typescript
interface ResourcePolicy {
  // Define the multidimensional resource envelope here
  // Must support: QPU-hours, circuit depth, qubit count, shots/job, concurrent jobs, daily burst
}

interface CircuitSubmissionRequest {
  // Define the submission payload here
  // Must capture everything needed for routing + auth + cost estimation
}

interface AuthorizationDecision {
  // Define the decision payload: approved/rejected, reason, estimated cost, quota remaining
}
```
