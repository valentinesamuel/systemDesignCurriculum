---
**Title**: OrbitCore — Chapter 151: The Cell Problem
**Level**: Staff
**Difficulty**: 8
**Tags**: #cell-based-architecture #blast-radius #failover #distributed #resilience #ground-station
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 150 (OrbitCore telemetry ingestion), Ch. 69 (VertexCloud blast radius), Ch. 144 (Pattern Summary 10)
**Exercise Type**: System Design
---

### Story Context

It is one week into the job. You've presented the high-level ingestion redesign to Ingrid and Tomás. They approved the direction. Now comes the harder conversation.

**[Staff Design Review — Recorded Meeting Transcript, March 16, 2:00pm]**

**Attendees:**
- You (Staff Engineer)
- Tomás Reyes (Staff Engineer, Infra)
- Ingrid Solberg (VP Engineering)
- Dr. Yusuf Adeyemi (Principal Engineer — joined OrbitCore 6 months ago from ESA)
- Preet Kapila (SRE Lead)

---

> **Ingrid Solberg:** Okay. You've had a week. Walk us through how you want to structure the ground station network.

> **You:** Right. So the core proposal is to treat each ground station as an independent cell. Each cell owns its satellite cluster, its ingestion pipeline, its local buffer — the whole stack. Cross-cell communication only happens for coordination, not for data flow.

> **Dr. Yusuf Adeyemi:** Define "satellite cluster" precisely. Orbital mechanics means a satellite's ground station changes every 90 seconds. How is a satellite "owned" by a cell?

> **You:** The cell owns the satellite during its visibility window. The satellite's authoritative current state lives with whatever cell last had contact. We use a handoff protocol during —

> **Dr. Yusuf Adeyemi:** And when the handoff fails?

> [Pause]

> **You:** The receiving cell requests a state sync from the outgoing cell before assuming authority.

> **Dr. Yusuf Adeyemi:** And if the outgoing cell has already gone offline? Singapore, for example. What happened last Tuesday at 14:23 UTC?

> **Preet Kapila:** [checking laptop] Singapore went silent for 11 minutes. Power fluctuation. Three Kepler satellites were mid-handoff to Sydney.

> **Dr. Yusuf Adeyemi:** During those 11 minutes, who had authoritative state for those three satellites?

> [Silence]

> **Tomás Reyes:** Nobody. We lost the handoff state. We replayed from the last confirmed telemetry frame.

> **Dr. Yusuf Adeyemi:** We lost 11 minutes of telemetry for three satellites because our "cell" design doesn't define who owns state during a handoff failure. This is not a scaling problem. This is a correctness problem.

> **Ingrid Solberg:** [to you] This is why you're here. The current architecture is a mesh with no failure semantics. I need a cell design with explicit blast radius limits. What does Johannesburg going down look like? What does it affect? What does it not affect?

---

**[Slack DM — Dr. Yusuf Adeyemi → You, 4:15pm, same day]**

> **Dr. Yusuf Adeyemi**
> I was not trying to embarrass you in there. I've watched three engineers present similar designs and all three missed the handoff problem.
> The question I didn't ask in the meeting: what's your recovery model when TWO adjacent cells fail simultaneously? It hasn't happened yet. But solar weather during equinox season hits East Asia and Australia at the same time. Singapore and Sydney could both go dark.

> **You**
> How often does that happen?

> **Dr. Yusuf Adeyemi**
> Every March and September. We're in March now.

---

**[#incidents — Slack, March 17, 07:44am]**

> **PagerDuty [bot]**
> ALERT: GS-Johannesburg — telemetry ingest rate dropped 40% over 15 minutes. Auto-recovery did not trigger. Satellite SV-08 (TerraWatch constellation) has not reported in 18 minutes.

> **Niamh Okafor**
> Checking. GS-Joburg is up. The issue is SV-08 specifically — it's in a degraded power mode due to a battery anomaly. It's sending at reduced rate on a backup frequency that our ingest parser doesn't recognize.
> @you — this is actually relevant to your cell design. The cell needs to handle out-of-band signals from degraded satellites, not just nominal telemetry.

> **You**
> Adding to the design requirements.

> **Niamh Okafor**
> Story of my life at this company.

---

**[Email — Ingrid Solberg → You, March 17, 9:02am]**

**From:** Ingrid Solberg
**To:** [You]
**Subject:** Cell architecture — board expectations

> For the Series C due diligence call on March 23rd, I need to be able to say with confidence:
>
> "Each ground station failure is fully contained. No single ground station failure affects more than X% of our total telemetry capacity, and recovery is automatic within 30 seconds."
>
> I need that X. I need to know it's architecturally enforced, not just operationally hoped-for.
>
> Also: Orion Ventures will specifically ask about our RTO. 30 seconds is the number I've committed to. Make sure the design can actually deliver that.

### Problem Statement

OrbitCore's 12 ground stations operate as a loosely-coupled mesh with no explicit failure semantics, no formal blast radius limits, and no defined ownership model for satellite state during handoffs. The result is a system where any single ground station failure produces unpredictable, cascading effects across the network.

Design a cell-based architecture for OrbitCore's ground station network. Each cell encompasses one ground station, its satellite cluster (the satellites currently in its visibility window), and its local processing stack. Define: cell isolation guarantees, cross-cell communication protocols, satellite handoff semantics during both nominal and failure conditions, and the recovery model when one or two adjacent cells fail simultaneously.

### Explicit Requirements

1. Each cell (ground station) must be independently deployable and independently operable
2. A single cell failure must affect no more than the satellites in that cell's current visibility window — no cascade to other cells
3. Satellite state handoff between cells must be atomic or have a defined fallback if the handoff fails mid-transfer
4. RTO for cell recovery (auto-failover to adjacent cell): 30 seconds
5. RPO: 0 — no telemetry loss during cell failure or handoff
6. Degraded satellite signals (backup frequencies, reduced-rate modes) must be handled within the cell layer, not pushed to consumers as errors
7. The cell coordination layer must not become a single point of failure itself

### Hidden Requirements

1. **Hint: re-read Dr. Adeyemi's Slack DM.** He asks about simultaneous failure of TWO adjacent cells during equinox season. What does your architecture look like when Singapore AND Sydney are both offline? How many satellites are left unserviced, and is there a third cell that could cover them?

2. **Hint: re-read the #incidents thread about SV-08.** Out-of-band signals from degraded satellites are not a theoretical edge case — they happen. The cell must have a signal normalization layer before telemetry reaches the ingestion pipeline. Where does that normalization live, and does it affect your blast radius analysis?

3. **Hint: re-read Ingrid's email carefully.** She says "no single ground station failure affects more than X% of total telemetry capacity." What is X, and is it configurable per constellation SLA? Hint: the Kepler constellation contract likely has a different SLA from TerraWatch.

### Constraints

- **Ground stations (cells)**: 12 total — US West (2), US East (1), UK (2), Germany (1), Kenya (1), Singapore (1), Australia-Sydney (1), Australia-Perth (1), Johannesburg (1), spare capacity TBD
- **Satellites per cell (avg)**: 4-5 simultaneously visible during peak LEO window
- **Handoff frequency**: Every 90 seconds per satellite (LEO orbital period ~90 min, ground station visibility ~10 min)
- **RTO**: 30 seconds (auto-failover, no human intervention)
- **RPO**: 0 (no telemetry loss)
- **Cell coordination latency budget**: < 50ms for state sync between adjacent cells
- **Blast radius limit (required)**: Single cell failure must not cascade beyond its own satellite cluster
- **Team operating cells**: 2 SREs on-call globally
- **Series C due diligence**: March 23rd (6 days away from the story's "now")

### Your Task

Design the cell-based architecture for OrbitCore's ground station network. Produce: a cell topology diagram, a formal definition of the blast radius per cell, the satellite handoff protocol (including failure modes), the cell coordination layer design (without making it a SPOF), and the recovery automation model for both single-cell and dual-cell failure scenarios.

### Deliverables

- [ ] Mermaid architecture diagram showing:
  - Cell structure (what's inside a cell vs. what's shared)
  - Cross-cell coordination plane
  - Satellite handoff state machine (nominal and failure paths)
- [ ] Formal blast radius analysis:
  - Satellites affected per cell failure (show math: % of total fleet)
  - Simultaneous dual-cell failure scenario (Singapore + Sydney): what is the impact, and does your design have a mitigation?
- [ ] Satellite handoff protocol specification:
  - State transferred during handoff (what data? what format? what size?)
  - Handoff atomicity mechanism (two-phase? optimistic? something else?)
  - Fallback if receiving cell does not acknowledge within X ms
- [ ] Cell coordination layer design:
  - How do cells discover each other?
  - How is the coordination plane made fault-tolerant without becoming a SPOF?
  - What is the blast radius of the coordination plane itself failing?
- [ ] Scaling estimation (show math):
  - State sync data volume per handoff event
  - Handoff frequency at 47 satellites vs. 200 satellites
  - Coordination plane message volume per second
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Shared coordination service vs. peer-to-peer cell gossip
  - Optimistic handoff (send state, assume receipt) vs. two-phase handoff (ack required)
  - Cell-local storage for satellite state vs. centralized state store
- [ ] Cost modeling ($X/month):
  - Per-cell infrastructure cost (compute, local storage, local Kafka/Pulsar)
  - Coordination plane cost
  - Total at 12 cells vs. projected cost at 30 cells (200 satellites may require more ground stations)
- [ ] Capacity planning (18-month horizon):
  - How many cells are needed for 200 satellites?
  - What triggers a new ground station addition?
  - How does cell architecture change when you add a cell mid-constellation?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

Key diagram to produce: a state machine for the satellite handoff lifecycle.

```
stateDiagram-v2
  [*] --> Nominal
  Nominal --> HandoffInitiated
  HandoffInitiated --> HandoffConfirmed
  HandoffInitiated --> HandoffFailed
  HandoffFailed --> FallbackRecovery
  ...
```
