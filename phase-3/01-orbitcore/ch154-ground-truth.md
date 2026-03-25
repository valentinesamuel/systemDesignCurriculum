---
**Title**: OrbitCore — Chapter 154: Ground Truth
**Level**: Staff
**Difficulty**: 7
**Tags**: #rfc #protocol-migration #api-design #distributed #staff-work #typescript #strangler-fig
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 151 (cell architecture), Ch. 152 (sovereignty), Ch. 153 (latency routing)
**Exercise Type**: System Design
---

### Story Context

It is March 30th. Three weeks in.

The Series C call went well. Orion Ventures is satisfied with the roadmap. The Vantage crisis was communicated as "identified and remediated with architectural improvements" — technically true if you squint. You've been heads-down on the cell architecture since.

Then an email arrives.

**[Email chain]**

**From:** Dr. Yusuf Adeyemi <yusuf@orbitcore.io>
**To:** Engineering Leadership <eng-leads@orbitcore.io>
**Subject:** Proposal: CCSDS Protocol Adoption — Request for RFC Author
**Date:** March 30, 9:15am

> Team,
>
> I've been advocating for this for 18 months. The time has finally come to make it formal.
>
> OrbitCore's ground-station-to-backend protocol is a proprietary binary format we designed in 2019. At the time, it was pragmatic — we needed something fast and our only customer was Kepler Dynamics who didn't care. That pragmatism has become a liability.
>
> The Consultative Committee for Space Data Systems (CCSDS) has defined open standards for space data communication used by NASA, ESA, JAXA, and virtually every serious space agency on Earth. Our prospective customers — particularly government and defense-adjacent operators — expect CCSDS compatibility. Our inability to speak CCSDS has already cost us two RFPs.
>
> I'm proposing we adopt CCSDS CFDP (CCSDS File Delivery Protocol) for file-based telemetry and CCSDS TM/TC Space Data Link Protocol for the real-time telemetry stream.
>
> I am asking for a volunteer to write the formal RFC. This needs to be engineering-led, not an executive mandate. The RFC author will own the technical proposal and shepherd it through review.
>
> — Yusuf

---

**[Slack DM — Ingrid Solberg → You, 9:44am]**

> **Ingrid Solberg**
> Yusuf's CCSDS thing. You should own the RFC.
> You're 3 weeks in, you understand the current protocol better than anyone who just joined has a right to, and frankly — you need to establish yourself as the person who does this kind of work here.

> **You**
> Happy to. How much political resistance should I expect?

> **Ingrid Solberg**
> Tomás built the current protocol in 2019. He's proud of it. He also knows it needs to change. He'll push back hard on the RFC and then vote yes.
> The real resistance will be from the ground station ops team. They've written custom tooling against the proprietary format. A protocol migration means their tooling breaks.

---

**[RFC Draft Review Meeting — April 7, 10:00am]**
**Attendees:** You, Dr. Yusuf Adeyemi, Tomás Reyes, Ingrid Solberg, Niamh Okafor (Ground Ops), Preet Kapila

**[Transcript excerpt]**

> **You:** The proposal is a strangler-fig migration. We run both protocols in parallel at the ground station agent layer. New satellite connections use CCSDS. Existing connections continue on the proprietary binary format until we've validated the adapter. We flip the switch per-satellite, not per-ground-station.

> **Tomás Reyes:** Why per-satellite?

> **You:** Because the blast radius of a failed migration is one satellite, not one ground station. A ground station handles 4-5 satellites. If the migration breaks a ground station, we lose all of them simultaneously. If it breaks a satellite connection, we lose one.

> **Tomás Reyes:** [pause] That's actually the right call. I was going to argue but I can't.

> **Niamh Okafor:** I want to talk about the ground station tooling. I have scripts. Not documented scripts. Scripts that parse byte offsets in the proprietary binary format. If you change the protocol, those scripts stop working.

> **You:** How many scripts?

> **Niamh Okafor:** [long pause] Forty-seven. Ish.

> **You:** We need an inventory before the RFC is finalized. I can't write a migration plan without knowing what downstream consumers exist.

> **Niamh Okafor:** That's going to take time.

> **You:** How much time?

> **Niamh Okafor:** Two weeks if I deprioritize everything else.

> **Dr. Yusuf Adeyemi:** The RFC shouldn't block on the tooling inventory. The tooling inventory is a consequence of the RFC. We identify the protocol adapter interface, and the tooling owner adapts their tools. That's the cost of 5 years of undocumented parsing.

> **Niamh Okafor:** [sharply] I didn't build it undocumented because I wanted to. I built it undocumented because we were moving too fast for documentation to be possible.

> **Ingrid Solberg:** Niamh is right that the ops team bears disproportionate migration cost. The RFC needs to include a tooling migration support plan.

---

**[Slack DM — Tomás Reyes → You, after the meeting, 11:30am]**

> **Tomás Reyes**
> Off the record: the proprietary format has one thing CCSDS doesn't.
> Custom compression. We use a satellite-specific compression profile that reduces the telemetry payload by 40% for Kepler constellation birds. CCSDS doesn't support custom codecs.
> I built it because Kepler's satellites have limited downlink bandwidth. Without the compression, some satellites would overflow their downlink queue during a LEO window.
> This isn't documented anywhere. It's in a comment in a Go file called `telemetry_codec_kepler.go`.
>
> I'm telling you now so it's in the RFC and not discovered after the migration breaks Kepler's telemetry.

> **You**
> Thank you. This goes in the RFC as a hard constraint.

> **Tomás Reyes**
> There might be more like that. I'd do a grep for `_codec_` in the codebase before you finalize.

### Problem Statement

OrbitCore's ground-to-backend communication protocol is a proprietary binary format built in 2019. The format is undocumented in places, has satellite-specific compression profiles, is used by 47+ downstream tooling scripts, and is incompatible with CCSDS — the open industry standard expected by government and defense customers. A migration to CCSDS is strategically mandatory.

Write the RFC for OrbitCore's ground network protocol migration from the proprietary binary format to CCSDS CFDP (file-based telemetry) and CCSDS TM/TC Space Data Link Protocol (real-time stream). The RFC must include: protocol adapter interface design (TypeScript), migration strategy, rollback plan, downstream tooling impact assessment, and resolution for the custom compression constraint discovered by Tomás.

### Explicit Requirements

1. RFC must define the TypeScript interface for a CCSDS protocol adapter that supports both the proprietary format and CCSDS, selectable per satellite connection
2. Migration strategy must be strangler-fig per-satellite (not per-ground-station), limiting blast radius to one satellite per migration step
3. RFC must include a formal rollback plan per satellite migration unit
4. RFC must enumerate all known categories of downstream protocol consumers and their migration burden
5. The custom Kepler compression profiles must be addressed — the RFC must propose a solution that preserves downlink bandwidth efficiency under CCSDS
6. RFC must include a timeline with explicit go/no-go criteria per migration phase
7. RFC must include a "what we are NOT doing and why" section (explicit non-goals)

### Hidden Requirements

1. **Hint: re-read Tomás's DM about the compression codec.** He says "there might be more like that — do a grep for `_codec_` in the codebase." The RFC explicitly acknowledges one custom codec. But the hidden requirement is: your migration plan must include a pre-migration discovery step that uncovers ALL protocol customizations before migration begins. What does that discovery process look like?

2. **Hint: re-read Niamh's comment about 47 undocumented scripts.** She said "ish." The RFC must handle the reality that downstream consumer inventory will be incomplete. Your adapter interface design should be compatible enough with the proprietary binary format's field layout that existing parsers do not need to be rewritten — only the wrapper format changes. Is that possible? Is it a design goal?

3. **Hint: re-read Dr. Adeyemi's opening email.** He says "inability to speak CCSDS has already cost us two RFPs." This implies the migration has a business deadline driven by RFP cycles, not just engineering readiness. The RFC timeline must be anchored to a concrete business milestone. What is that milestone, and does it affect your per-satellite migration pace?

### Constraints

- **Satellites to migrate**: 47 (Kepler: 22, Vantage: 11, ArcticLens: 8, TerraWatch: 14 — total: 55, but 47 active)
- **Custom compression**: Kepler satellites only — 40% payload reduction required for downlink bandwidth budget
- **CCSDS compatibility**: CFDP for file-based, TM/TC Space Data Link for real-time stream
- **Downstream consumers**: 47+ tooling scripts (inventory TBD), all backend ingestion consumers, SLA monitoring systems
- **Migration pace**: strangler-fig per-satellite; suggest max 3-5 satellites per migration batch
- **Rollback requirement**: per-satellite, < 60 seconds to rollback a failed migration
- **Team capacity**: Tomás and one additional engineer own the adapter implementation; Niamh owns tooling migration
- **Business deadline**: next major RFP window in approximately 6 months

### Your Task

Write the RFC for OrbitCore's ground network protocol migration. This is a staff-level deliverable: it must be precise enough for engineers to implement from, persuasive enough to win a vote from skeptics, and honest enough to enumerate risks without hiding them. Also produce the TypeScript interface for the CCSDS protocol adapter — the core technical artifact of the RFC.

### Deliverables

- [ ] RFC document structure:
  - Problem statement
  - Proposal (strangler-fig per-satellite migration)
  - Technical design: protocol adapter architecture
  - TypeScript interface for the CCSDS protocol adapter (`IProtocolAdapter`, `ITelemetryFrame`, `ICompressionCodec`)
  - Migration phases with go/no-go criteria
  - Rollback plan
  - Non-goals (explicit)
  - Risks and mitigations
  - Timeline
- [ ] TypeScript code task — the protocol adapter interface:
  ```typescript
  // Write the TypeScript interface for a dual-protocol adapter
  // that supports both the proprietary binary format and CCSDS TM/TC.
  // The interface must:
  // - Be selectable per-satellite connection
  // - Support pluggable compression codecs (for Kepler custom compression)
  // - Expose a normalize() method that produces a canonical TelemetryFrame
  //   regardless of wire protocol
  // - Support a dry-run mode for migration validation
  interface IProtocolAdapter { ... }
  interface ITelemetryFrame { ... }
  interface ICompressionCodec { ... }
  ```
- [ ] Mermaid architecture diagram showing:
  - Current protocol flow (proprietary binary)
  - Target protocol flow (CCSDS with adapter layer)
  - Strangler-fig migration state per satellite
  - Rollback path
- [ ] Migration phases:
  - Phase 0: Pre-migration discovery (codec audit, downstream consumer inventory)
  - Phase 1: Adapter implementation + shadow mode validation
  - Phase 2: Per-satellite strangler-fig migration (batch of 3-5 per sprint)
  - Phase 3: Legacy protocol deprecation
  - Go/no-go criteria for each phase transition
- [ ] Kepler compression resolution:
  - Option A: CCSDS with custom codec extension (non-standard, but possible)
  - Option B: CCSDS with out-of-band compression layer (standard CCSDS + compression header)
  - Option C: Negotiate with Kepler Dynamics to upgrade downlink bandwidth (business option)
  - Recommend one with rationale
- [ ] Tradeoff analysis (minimum 3):
  - Per-satellite migration vs. per-ground-station migration (blast radius analysis)
  - Shadow mode validation duration (how long to run both protocols simultaneously per satellite before flipping?)
  - Custom codec extension vs. out-of-band compression layer
- [ ] Cost modeling:
  - Engineering cost (person-weeks) for each migration phase
  - Infrastructure cost during parallel protocol operation (both protocols active simultaneously)
  - Revenue opportunity cost: what does each delayed RFP represent?
- [ ] Capacity planning:
  - Migration timeline at max 5 satellites/sprint, 2-week sprints
  - Total migration duration
  - Does this timeline meet the 6-month RFP window?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
