---
**Title**: CHECKPOINT 5 — Strong Senior → Staff Interview @ Apex Engineering
**Level**: Staff
**Difficulty**: 10
**Tags**: #checkpoint #staff-level #system-design #ambiguity #clarifying-questions #distributed-systems #safety-critical
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 149 (VertexCloud), Ch. 176 (Checkpoint 4), Ch. 186 (ShieldMutual)
**Exercise Type**: Promotion Checkpoint
---

### Story Context

The calendar invite arrives on a Thursday afternoon, sandwiched between a VertexCloud sprint retro and a 1:1 with your manager. The subject line: **"System Design Interview — Final Round | Apex Engineering"**. You remember applying three weeks ago during a low-energy Sunday, one of those evenings where you scroll job boards half-heartedly. You'd submitted your CV mostly because the role description used the phrase "deeply ambiguous problems in safety-critical domains" and it sounded interesting.

You'd forgotten about it.

The invite lists three panelists. Two are names you don't recognize: **Sanjay Patel, Staff Engineer** and **Lena Fischer, Senior Engineer**. The third stops you mid-scroll.

**Dr. Adaeze Obi, Principal Researcher.**

You do what any engineer does: you Google her immediately. The results are sparse in the way that genuinely important people's results sometimes are. A few ACM papers. A brief bio on a conference website: *"Dr. Adaeze Obi led the distributed consensus layer for the European Space Agency's Galileo ground control modernization program. She currently advises early-stage infrastructure companies on correctness guarantees in physical-world systems."*

You mention it to your colleague **Priya Nair** over Slack — she's been your informal sounding board since the Apex checkpoint.

---

**#general-eng** | Thursday 3:47 PM

**You**: hey random question — you know anyone named Adaeze Obi? principal researcher, ESA background?

**Priya Nair**: ...wait

**Priya Nair**: THE Adaeze Obi?

**Priya Nair**: she built the consensus protocol for Galileo ground systems. I read her paper in grad school

**Priya Nair**: she's interviewing you?

**You**: apparently. final round at some company called Apex Engineering

**Priya Nair**: okay. different advice than I'd normally give

**Priya Nair**: don't prepare answers. prepare questions.

**Priya Nair**: she's famous for rejecting candidates who start designing before they understand the problem

**You**: how famous

**Priya Nair**: there's a blog post. guy called it "the most humbling 90 minutes of my career"

**You**: great

**Priya Nair**: you'll be fine. you've been doing this for a while now

**Priya Nair**: just remember: she's not testing what you know. she's testing how you think when you don't know.

---

The video call opens at 10:00 AM sharp. Sanjay and Lena are warm but professional. Dr. Adaeze Obi has a calm, unhurried quality — the kind of stillness that belongs to people who have made genuinely consequential decisions under pressure and no longer feel the need to perform urgency.

She speaks first.

**Dr. Obi**: "Good morning. Before we give you the problem, I want to ask you something. When you receive a system design brief that's unfamiliar, what's the first thing you do?"

You answer. She nods once and makes a small note.

**Dr. Obi**: "Here's the brief. I'll read it exactly once."

*"Design a system that ingests signals from distributed physical infrastructure and makes real-time decisions that affect physical-world outcomes."*

A beat of silence.

**Dr. Obi**: "We're not going to say anything else yet. What clarifying questions do you have?"

**Lena Fischer** has a laptop open in front of her, fingers hovering. She'll be taking notes on every question you ask and the order you ask them in. You realize — the questions themselves are the first deliverable.

**Dr. Obi** leans slightly forward. "Take your time. There's no penalty for thinking before you speak. There is a penalty for designing before you understand."

You have the distinct sense that the last sentence is not a warning. It's a test to see if you already knew it.

---

*Later, in the debrief section of the interview, Dr. Obi says something that will stick with you:*

**Dr. Obi**: "The best Staff engineers I've worked with share one trait. They can describe the shape of a problem before they know its contents. They know which questions are load-bearing — the ones where the answer changes the entire architecture — versus which questions are decorating."

**Sanjay Patel** leans in: "Walk us through which of your questions were load-bearing and why."

---

### Problem Statement

The brief is deliberately, professionally vague: *"Design a system that ingests signals from distributed physical infrastructure and makes real-time decisions that affect physical-world outcomes."*

This is not a trick. It is a Staff-level brief. In real Staff engineering work, problems arrive this way: business need without technical specification, urgency without context, constraints buried in organizational history you don't yet have. The ability to identify the right clarifying questions — and to know which answers are architecturally load-bearing — is itself a Staff-level skill.

The panel will reveal requirements only in response to your questions. Your questions determine what system you design. A candidate who skips clarifying questions and starts designing has already failed, regardless of technical quality. A candidate who asks 20 low-signal questions has also failed. The task is to ask the minimum necessary set of high-signal questions that uniquely determine the architecture.

---

### Explicit Requirements

*(Revealed by the panel only when asked the right questions)*

1. The physical infrastructure is a network of 2,000 IoT environmental sensors deployed across a regional water treatment network (8 treatment plants, 200 pumping stations, 1,200 remote sensors)
2. Signals include: water pressure, flow rate, chemical levels (pH, chlorine, turbidity), pump operational status — each sensor emits readings every 5 seconds
3. "Real-time decisions" include: automated valve control commands (< 500ms from anomaly detection to actuator command), regulatory threshold alerts (< 5 seconds), human operator notifications (< 30 seconds)
4. Decisions that trigger actuator commands must be logged with: sensor reading, decision logic version, timestamp, operator override history — retained for 7 years (regulatory)
5. The system must continue operating for critical safety functions during full cloud connectivity loss (island mode)
6. The system operates under EPA Safe Drinking Water Act reporting requirements and state-level DHS infrastructure protection regulations
7. 99.999% availability SLA for safety-critical decision paths (< 5.26 minutes downtime per year)
8. Current state: a SCADA system from 2009, no cloud integration, operators make manual decisions from a control room dashboard

---

### Hidden Requirements

*(Discoverable only by asking the right clarifying questions)*

1. **Hint**: Dr. Obi says early in the interview, "What would this look like if you were wrong?" Re-read that line. What category of correctness guarantee does "physical-world outcomes" imply that a purely software system does not? Ask about rollback semantics for actuator commands.

2. **Hint**: Lena Fischer mentions "the operators in the control room" when clarifying requirement 8. What does it mean architecturally when humans are in the decision loop for some decisions but not others? Ask: which decisions are fully automated vs. human-confirmed?

3. **Hint**: The brief says "distributed physical infrastructure." Sanjay Patel uses the word "regional" when describing coverage. Ask about network topology between sensors and edge nodes — the answer reveals that 400 of the 1,200 remote sensors connect via 3G/LTE with intermittent coverage. This fundamentally changes the edge architecture.

4. **Hint**: The mention of "2009 SCADA system" is not just historical color. SCADA systems use industrial protocols (Modbus, DNP3, OPC-UA) — not REST APIs or Kafka producers. Ask about integration with existing systems. The answer reveals a protocol translation requirement that adds an entire architectural layer and a safety-critical data validation step (SCADA readings are unitless integers — context is required to interpret them).

---

### Constraints

*(To be elicited through clarifying questions — the panel reveals these only when asked)*

- **Scale**: 2,000 sensors × 1 reading/5 seconds = 400 readings/second (ingest baseline)
- **Decision latency**: Actuator commands < 500ms p99 from anomaly detection
- **Availability**: 99.999% for safety paths; 99.9% acceptable for analytics paths
- **Island mode**: Edge nodes must operate autonomously for up to 72 hours without cloud
- **Audit retention**: 7 years, tamper-evident, EPA-compliant format
- **Team**: 8 engineers (3 backend, 2 infra, 1 ML, 1 embedded/edge, 1 security)
- **Budget**: ~$180K/year infrastructure budget (currently $0 — greenfield cloud deployment)
- **Compliance**: EPA SDWA, state DHS critical infrastructure, NIST CSF for OT/IT convergence
- **Protocol**: Existing SCADA speaks DNP3 and Modbus; new system must bridge to modern stack
- **Data volume**: 400 readings/sec × 200 bytes = 80 KB/sec ingest; 7-year retention ≈ 17.6 TB raw

---

### Your Task

You must produce two things:

**Part 1 — The Clarifying Questions**: Before designing anything, document at least 10 clarifying questions you would ask the panel, in the order you would ask them. For each question, explain: (a) why you're asking it, and (b) how different answers would lead to fundamentally different architectures. Identify which 3-4 questions are "load-bearing" — the ones where the answer most dramatically changes the design.

**Part 2 — The System Design**: Using the requirements revealed through your questions, design the complete system. This is a safety-critical IoT system at the edge of IT/OT convergence. Your design must account for the constraints revealed, the compliance requirements, and the island-mode operation mandate.

---

### Deliverables

- [ ] **Clarifying questions** (minimum 10, each with rationale and "how would different answers change my design?")
- [ ] **Load-bearing question analysis**: identify your top 3-4 questions and explain the architectural fork each one creates
- [ ] **System architecture** — end-to-end: sensor → edge node → cloud → actuator command path
- [ ] **Mermaid architecture diagram** (Mermaid syntax, renders in GitHub Issues)
- [ ] **Scaling estimation** — show math step by step (ingest rate, storage, latency budget breakdown)
- [ ] **Cost modeling** — $X/month estimate against the $180K/year budget constraint
- [ ] **Island mode design** — how the edge layer operates autonomously during cloud connectivity loss
- [ ] **Audit trail design** — tamper-evident 7-year retention meeting EPA requirements
- [ ] **Tradeoff analysis** — minimum 4 explicit tradeoffs with justification
- [ ] **What you chose NOT to build** — Staff-level deliverable: explicitly name 3+ capabilities you scoped out, why, and what the cost of deferral is
- [ ] **Compliance checklist** — EPA SDWA, NIST CSF OT/IT, DHS critical infrastructure requirements mapped to architectural decisions

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

```
Example structure hint (not a solution):
graph TD
    subgraph Edge["Edge Layer (Per Treatment Plant)"]
        SCADA[SCADA / DNP3 / Modbus] --> Bridge[Protocol Bridge]
        Bridge --> EdgeProcessor[Edge Decision Engine]
        EdgeProcessor --> LocalStore[Local Time-Series Store]
        EdgeProcessor --> Actuator[Actuator Command Bus]
    end
    subgraph Cloud["Cloud Layer"]
        ...
    end
```

---

### Staff-Level Meta-Note

This checkpoint tests something that cannot be studied directly: the quality of your uncertainty. A Mid engineer doesn't know what they don't know. A Senior engineer knows what they don't know and looks it up. A Staff engineer knows which unknown unknowns are architecturally dangerous — and asks about those first.

Dr. Adaeze Obi's final question in every interview she runs: *"What would this design look like if your most important assumption turned out to be wrong?"*

Have an answer ready.
