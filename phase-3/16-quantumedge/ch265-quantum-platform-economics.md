---
**Title**: QuantumEdge — Chapter 265: Quantum Platform Economics
**Level**: Staff
**Difficulty**: 9
**Tags**: #cost-engineering #capacity-planning #finops #distributed
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 144 (VertexCloud FinOps), Ch. 131 (Crestline capacity planning), Ch. 263
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**[Email chain — Subject: Series A Data Room — Financial Model Request]**

```
From: Preet Malhotra <p.malhotra@sequoia.com>
To: yuki.tanaka@quantumedge.io
Date: Friday, 2:08pm
Subject: Series A Data Room — Financial Model Request

Dr. Tanaka —

Sequoia is proceeding to the next stage of our evaluation.
For the data room, we need a 3-year financial model covering:

1. Revenue projections by customer tier
2. Unit economics: cost to serve per customer, per QPU-hour
3. Infrastructure cost projections: QPU, GPU, networking, personnel
4. Capacity planning: QPU count needed at Year 1, Year 2, Year 3 scale

One area we'd like to discuss specifically:
The quantum hardware improvement curve.
QPU prices are dropping. Qubit counts are increasing. Error rates are falling.
Today's 16-qubit noisy QPU and tomorrow's 100-qubit error-corrected QPU
are not the same product for your customers.

Your model needs to account for:
a) Hardware obsolescence — today's QPU-1 may be worthless in 18 months
b) Customer re-pricing — if QPU cost drops 40%/yr, do customers expect lower prices?
c) Demand expansion — as QPUs improve, customers will want more QPU time
   (better hardware = more problems become economically viable to run on QPU)

We need to see that you've modeled this, not just assumed a flat cost curve.

Target: 3-year model, quarterly granularity.
Deadline: next Wednesday.

Preet Malhotra
Partner, Sequoia Capital
```

---

**[1:1 transcript — You and Dr. Yuki Tanaka, Friday 3pm]**

**Dr. Tanaka**: "I've been staring at this request for two hours. I know the physics better than anyone in this building. I do not know how to model exponential improvement in a financial forecast."

**You**: "Walk me through what you know about the hardware trajectory."

**Dr. Tanaka**: "IBM's public roadmap: 16 qubits today, 127 by end of year, 400 next year, error-corrected systems by year three. IonQ: similar trajectory on error rates, slightly different on qubit count. The key number is the 'quantum volume' metric — it combines qubit count AND gate fidelity AND connectivity into one number. IBM Eagle today: quantum volume ~64. Their forecast: 1,000 by year two, 100,000 by year three."

**You**: "What does that mean for QuantumEdge's business?"

**Dr. Tanaka**: "Today's use cases — molecular simulation, portfolio optimization, optimization problems — they're all in the 'noisy intermediate-scale' regime. You get approximate answers with significant noise. You pay for error mitigation. In two years, error-corrected QPUs change the math completely. You get exact answers without mitigation overhead. The problems you can solve expand dramatically."

**You**: "So today's revenue is from NISQ-era use cases. Year two-three revenue is from fault-tolerant use cases that don't exist yet."

**Dr. Tanaka**: "Exactly. And the customers doing NISQ simulations today are paying for something that gets better every 18 months. If we don't model the price-performance trajectory, Sequoia will model it for us — and they'll be more pessimistic."

**You**: "What's the QPU acquisition model? You lease from IBM and IonQ?"

**Dr. Tanaka**: "Right now, yes. Per-shot pricing via cloud API. As we grow, we'll want reserved capacity contracts — lock in a fixed QPU allocation for 12 months at a discount. Similar to AWS Reserved Instances. The hardware vendors are starting to offer this."

**You**: "And depreciation? If QPU-1 has a quantum volume of 64 today and next year's QPU has a quantum volume of 1,000 — QPU-1 is still physically functional, but it's commercially worthless for any serious workload."

**Dr. Tanaka**: "The depreciation curve for quantum hardware is unlike anything in classical computing. It's not wear — it's obsolescence. The hardware works fine. But no serious customer will choose it over the next generation. We need to model economic depreciation, not physical depreciation."

**You**: "This is a 3-year model with at least two discontinuous phase transitions: NISQ to early fault-tolerant, and then to full fault-tolerant. You can't fit that on a straight-line curve."

**Dr. Tanaka**: [nods] "Which is why I need someone who understands both the technical trajectory and the financial modeling. That's why I hired a founding engineer, not just a PhD student."

---

**[Slack DM — Marcus Webb → You, Friday 5:44pm]**

```
marcus.webb [17:44]
Heard you're at a quantum computing startup now.

I've seen companies try to build on emerging hardware before.
InfiniBand, RDMA, FPGA cloud accelerators. Most of them
got the technical architecture right and the economics wrong.

The question isn't "how do we build a scheduler for QPUs."
The question is "what happens to our business model
when the hardware changes faster than our customer contracts?"

I've seen that kill companies that had technically superior products.
The unit economics shift underneath you.

Build the financial model the same way you'd build a distributed system:
assume failure modes. What's the scenario where this all breaks?
Model it explicitly. Then decide if you can survive it.

And: good work on the auth system. Hassan told me.
The resource-based policy engine is clean.
Don't screw it up.
```

---

### Problem Statement

QuantumEdge needs a 3-year capacity plan and financial model for its Series A data room. The model must account for the exponential improvement curve of quantum hardware: QPU prices dropping ~40%/year, quantum volume increasing by an order of magnitude every 18 months, customer demand expanding as more problems become economically viable on QPU. The model must cover: QPU cost curves, reserved vs spot capacity strategy, per-customer unit economics, and the phase transition from NISQ-era to fault-tolerant computing.

This is a FinOps problem for a technology that doesn't yet work at full scale. The modeling decisions made now will shape the company's pricing strategy for the next three years.

### Explicit Requirements

1. **3-year QPU capacity model**: QPU-hours needed at Year 0 (50 customers), Year 1 (200 customers), Year 2 (500 customers), Year 3 (1,000 customers)
2. **Hardware cost model**: per-QPU-shot cost trajectory under 40%/year price decline; model both spot (current) and reserved (future) pricing
3. **QPU economic depreciation model**: model when each generation of QPU becomes commercially non-competitive; replacement decision framework
4. **Per-customer unit economics**: cost to serve per customer per tier (Basic, Professional, Enterprise) including QPU shots, GPU compute, networking, error mitigation overhead, storage
5. **Margin modeling**: gross margin per customer tier at Year 0 vs Year 2 (how does margin evolve as costs drop and competition increases?)
6. **Reserved capacity strategy**: when does it become cost-effective to move from per-shot spot pricing to 12-month reserved QPU capacity? Model the crossover.
7. **Demand expansion model**: as QPU quantum volume increases, model the expected increase in QPU-hours consumed per customer (use-case expansion multiplier)
8. **Phase transition model**: model the business impact of the NISQ → early fault-tolerant transition (~Year 2): new customer types, new pricing, NISQ customers migrating to new hardware

### Hidden Requirements

- **Hint: re-read Marcus Webb's Slack.** "What happens to our business model when the hardware changes faster than our customer contracts?" Enterprise customers sign 12-month contracts. If QPU prices drop 40% during the contract period, the customer will expect a price reduction at renewal. But QuantumEdge has committed to serving them at the contracted price on older hardware. What's the contract architecture that manages this?

- **Hint: re-read Dr. Tanaka's roadmap description.** "100,000 quantum volume by year three" — this means error mitigation (which drives 2-3x cost today) becomes unnecessary. QuantumEdge's error mitigation pipeline (Ch. 263) is a revenue opportunity today but a liability in 2 years (customers won't pay for mitigation on fault-tolerant hardware). The 3-year model must include mitigation revenue decay.

- **Hint: re-read Preet Malhotra's email about demand expansion.** "better hardware = more problems become economically viable." But this is a new customer type problem, not just more of the same. Fault-tolerant QPU customers will be completely different industries (cryptography, drug discovery, logistics optimization at scale). The Year 3 revenue model may look nothing like Year 0. What are the assumptions?

- **Hint: re-read the QPU acquisition model conversation.** Dr. Tanaka mentions IBM and IonQ are starting to offer reserved capacity. But reserved QPU capacity has a utilization problem: if QuantumEdge reserves 1,000 QPU-hours/month and only sells 600 QPU-hours/month, the 400 idle hours are a pure cost. The model needs a minimum utilization assumption and a downside scenario.

### Constraints

- **Current state**: 3 QPUs (2× IBM Eagle 16q, 1× IonQ 23q), 10 GPU nodes, 50 customers, $5M ARR
- **QPU cost today**: $0.0003/shot (spot), ~$0.18/QPU-hour (at 1,000 shots/circuit, 2µs circuits)
- **QPU price trajectory**: ~40% YoY decline (industry consensus from IBM/IonQ public statements)
- **QPU hardware improvement**: quantum volume doubling approximately every 18 months (IBM public roadmap)
- **Current gross margin**: ~35% (before error mitigation overhead)
- **Error mitigation overhead**: adds 2-3x QPU cost for ~30% of current workloads
- **FinancialFirst contract**: $400K/yr (largest customer; 20% of current ARR)
- **Reserved capacity discount**: estimated 30% discount vs spot for 12-month commitment
- **Reserved capacity minimum utilization**: 80% utilization required to justify reserved over spot
- **Team size at Year 3 target**: ~50 engineers (current: 4)
- **Series A target**: $15M; implies $3M ARR minimum, path to $30M ARR by Year 3

### Your Task

Build the 3-year financial model framework and capacity plan. You are not building a spreadsheet — you are designing the system of equations and data models that would drive a financial model. Cover: the QPU capacity growth model, hardware cost curve assumptions, per-customer unit economics evolution, reserved capacity strategy, and the phase transition impact.

Write the model as a structured analysis with clearly stated assumptions, the math at each step, and explicit scenario analysis (base case, upside, downside).

### Deliverables

- [ ] Mermaid diagram: QPU capacity model over 3 years — customers × QPU-hours/customer × cost/QPU-hour = monthly cost; show the curves
- [ ] QPU capacity model (show math step by step):
  - Year 0: 50 customers × average QPU-hours/month × current cost/shot
  - Year 1: 200 customers × expanded usage × declining cost/shot
  - Year 2: 500 customers × use-case expansion factor × further cost decline
  - Year 3: 1,000 customers × fault-tolerant era assumptions
- [ ] Per-customer unit economics table: cost to serve per tier (Basic/Professional/Enterprise) at Year 0 and Year 2
- [ ] Reserved vs spot capacity crossover analysis: at what customer count does reserved QPU capacity become cost-optimal? Show the math.
- [ ] QPU depreciation model: economic depreciation curve; replacement decision framework (when does QPU-1 become commercially non-competitive?)
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Aggressive reserved capacity (lower cost, utilization risk) vs stay spot (higher cost, no utilization risk)
  - Price QPU-hours to customer at today's cost (protect margin now, lose customers at renewal) vs price at Year 2 cost (compress margin now, build customer loyalty)
  - Expand NISQ-era customer base now vs focus on fault-tolerant readiness at cost of near-term revenue
- [ ] Scenario analysis: base case (40% YoY QPU price decline), upside (60% decline — faster than expected), downside (20% decline — slower), disruption (fault-tolerant arrives 12 months early)
- [ ] Phase transition impact: what happens to current NISQ customers and revenue model at the NISQ → fault-tolerant transition? Is it an upgrade opportunity or a churn risk?
- [ ] Cost modeling summary: QuantumEdge platform infrastructure cost at Year 0 ($X/month), Year 1, Year 2, Year 3

### Diagram Format

Mermaid syntax. The capacity model diagram should show growth curves as a flowchart of decision nodes and calculation boxes, not a time-series chart (which Mermaid doesn't support well).
