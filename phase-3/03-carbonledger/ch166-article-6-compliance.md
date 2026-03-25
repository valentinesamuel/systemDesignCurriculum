---
**Title**: CarbonLedger — Chapter 166: Article 6 Compliance
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #compliance #distributed #api-design #data-residency #eventual-consistency
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 165 (Immutable Registry), Ch. 36 (CivicOS data sovereignty), Ch. 61 (SkyRoute multi-party rate limiting)
**Exercise Type**: System Design
---

### Story Context

**Email Chain**

---

**From**: Dr. Isabelle Fontaine, Swiss Federal Office for the Environment (BAFU)
**To**: Yara Osei; engineering@carbonledger.io
**CC**: Rafael Mendes, Brazilian Ministry of Environment (MMA); Lucas Park, UNFCCC Secretariat
**Subject**: RE: RE: RE: Technical Integration Specification — Article 6 Registry API
**Date**: Tuesday, 10:17 AM CET

Yara,

Thank you for the updated draft. We have reviewed it with our technical team and have the following concerns, which I will number for clarity:

1. **ISO 27001 certification requirement**: The Swiss Federal Data Protection Act (nFADP) requires that any system processing data on behalf of Swiss government entities must hold ISO 27001 certification or demonstrate equivalent controls. Your current SOC 2 Type II report is not accepted as equivalent under Swiss federal procurement law. This is a hard blocker.

2. **Corresponding Adjustment (CA) notification latency**: Your proposed 24-hour eventual consistency window for CA confirmations is technically acceptable under Article 6.2, but Switzerland has adopted a policy of requiring 4-hour confirmation for any transfer exceeding 10,000 tCO2e. This is not in the UNFCCC guidance — it is a Swiss national policy decision.

3. **Data residency**: Switzerland requires that any CA record involving Swiss NDC credits must have its authoritative copy stored in Switzerland or the EEA. AWS eu-west-1 (Ireland) is acceptable. AWS us-east-1 is not.

We are available for a technical call this week. Please coordinate with Rafael — Brazil has overlapping requirements that may conflict with points 1 and 3 above.

Best regards,
Dr. Isabelle Fontaine

---

**From**: Rafael Mendes, MMA — Diretoria de Políticas Ambientais
**To**: Yara Osei; engineering@carbonledger.io
**CC**: Dr. Isabelle Fontaine; Lucas Park
**Subject**: RE: RE: RE: Technical Integration Specification — Article 6 Registry API
**Date**: Tuesday, 11:43 AM BRT

Yara,

Thank you. I will add Brazil's requirements.

Brazil's REDD+ credits under Article 6.2 are managed through the SINARE system (Sistema Nacional de Registro). Any integration with CarbonLedger must go through the SINARE API (v2.1, documentation attached). SINARE uses a custom XML-over-HTTPS protocol — it does not support REST or JSON. Integration must be bidirectional: CarbonLedger notifies SINARE of transfers, and SINARE confirms corresponding adjustments.

Additionally, per Resolution CONAMA 500/2020, all credit data involving Brazilian forest projects must be stored on Brazilian territory or with a Brazilian data center provider. AWS sa-east-1 (São Paulo) is acceptable. The EEA is not acceptable for Brazilian data.

I note that Dr. Fontaine's requirement (EEA only) and Brazil's requirement (Brazil only) appear to conflict for cross-border transfers between Switzerland and Brazil. We have raised this with the UNFCCC Secretariat. Lucas, please advise.

Rafael Mendes

---

**From**: Lucas Park, UNFCCC Secretariat
**To**: All
**Subject**: RE: RE: RE: Technical Integration Specification — Article 6 Registry API
**Date**: Tuesday, 3:02 PM CET

All,

The UNFCCC Secretariat's position is that the Article 6 international registry (ITL successor) must maintain a *neutral* copy of all transactions that is not subject to the data residency requirements of either party to a transfer. CarbonLedger's architecture must therefore support three copies of each cross-border CA record:

1. A copy in the seller country's jurisdiction
2. A copy in the buyer country's jurisdiction
3. A neutral copy in the ITL-successor system (hosted by UNFCCC in Bonn, Germany — EEA jurisdiction)

We recognize this creates complexity. That is why this contract is worth what it is worth.

Lucas

---

**Video Call Transcript — Wednesday 9:00 AM GMT**
*Participants: You, Yara Osei, Dr. Isabelle Fontaine (BAFU), Rafael Mendes (MMA), Lucas Park (UNFCCC)*

**Yara** [to you, pre-call Slack DM]: "You're on this call to listen and take notes. Don't commit to anything. If they ask a technical question, defer to 'we'll follow up in writing.'"

**Dr. Fontaine**: "Good morning. I want to be direct: Switzerland has a hard deadline. Our parliament has committed to reporting Article 6 transfers at COP31 in three weeks. We need a working integration — or we use the Gold Standard interim system. Your system is better. But only if it works."

**Rafael**: "Brazil agrees on the deadline pressure. I will add: SINARE has not changed its API in four years. We cannot change SINARE. CarbonLedger must adapt to us."

**Lucas**: "The UNFCCC position is simple: we are the single source of truth. Everyone syncs to us. Not the other way around."

**You** [on mute, typing to Yara in Slack]: "these three requirements are in direct conflict. Switzerland wants EEA. Brazil wants Brazil. Lucas wants Bonn as canonical. We can't satisfy all three with a single write path."

**Yara** [Slack]: "I know. Figure out how."

---

**Slack DM — Kwesi Ampah → You, Wednesday 2:15 PM**

**Kwesi**: so how was the call

**You**: switzerland wants EEA-only. brazil wants brazil-only. the UN wants to be the canonical source. for the same transfer record.

**Kwesi**: oh that's fun. you know what this reminds me of? we had a similar problem when we integrated with the South Korean K-ETS registry. they have a 6-hour blackout window every night for "system maintenance." during that window, any transfer involving Korean credits just... queues. we never designed for that. had to add a hold state mid-flight.

**You**: hold state — that's actually useful. thanks.

**Kwesi**: also heads up — i checked the SINARE docs. their XML schema has a field called `corresponding_adjustment_status` with three possible values: PENDING, CONFIRMED, DISPUTED. we've never modeled a DISPUTED state. that's going to be interesting.

---

### Problem Statement

CarbonLedger must integrate with 196 national registries under the Article 6 Paris Agreement framework. Each Corresponding Adjustment (CA) — the record that a credit transferred from Country A to Country B has been subtracted from A's national inventory and added to B's — must be simultaneously compliant with potentially conflicting data residency laws, recorded in the UNFCCC central registry as the authoritative source of truth, and eventually consistent within defined SLA windows.

The architecture must handle a distributed write problem across sovereign systems that cannot be modified, use incompatible protocols, have independent uptime windows, and operate under different legal frameworks — while maintaining the guarantee that no credit is double-counted across jurisdictions.

### Explicit Requirements

1. Each Article 6 transfer must generate Corresponding Adjustment records in: seller country's national registry, buyer country's national registry, and the UNFCCC ITL-successor system
2. Data residency: seller country's CA record must be stored in seller's required jurisdiction; buyer's in buyer's required jurisdiction; UNFCCC copy in Bonn (EEA)
3. Protocol adaptation: system must translate between CarbonLedger's internal format, REST/JSON (for modern registries), and legacy XML-over-HTTPS protocols (e.g., SINARE)
4. Consistency SLA: standard transfers must achieve tri-party confirmation within 24 hours; transfers >10,000 tCO2e involving Switzerland must achieve confirmation within 4 hours
5. The DISPUTED state from national registries must be handled — a disputed CA must freeze the associated credits and trigger a human review workflow
6. System must operate correctly when one national registry is in a maintenance window (e.g., Korean K-ETS 6-hour nightly blackout)
7. ISO 27001 controls must be demonstrable for the Swiss integration path

### Hidden Requirements

- **Hint**: Re-read Kwesi's Slack message about the Korean K-ETS "hold state." What happens to credits that are "in flight" during a maintenance window? The system needs a credit state machine that includes a HELD-IN-TRANSIT state — and that state must be durable, not in-memory.
- **Hint**: Re-read Lucas Park's email carefully. "The UNFCCC Secretariat's position is that the international registry must maintain a *neutral* copy." What does "neutral" mean legally? The UNFCCC copy cannot be subject to any single country's data law — it must be architecturally isolated from national copies, not just logically separated.
- **Hint**: Rafael mentioned SINARE's XML schema has a `corresponding_adjustment_status` of PENDING, CONFIRMED, or DISPUTED. What happens if Switzerland confirms a transfer but Brazil disputes it? The system needs a conflict resolution protocol for split-decision CAs.
- **Hint**: Dr. Fontaine mentioned Switzerland's 4-hour SLA for large transfers only. But she also said "this is a Swiss national policy decision — not in UNFCCC guidance." What does this mean for your consistency model? Different buyers have different SLA tiers. The architecture must support per-country SLA configuration, not a single global consistency window.

### Constraints

- **Country registries**: 196 total; ~40 with REST APIs; ~80 with legacy XML/SOAP; ~76 with no digital API (manual notification required initially)
- **Transfer volume**: 10K/day today; up to 10M/day post-UN adoption
- **Cross-border transfers**: estimated 15% of all transfers are cross-border (today: 1,500/day; target: 1.5M/day)
- **Latency SLAs**: Standard: 24h confirmation; Large transfers (>10K tCO2e, Swiss buyer): 4h; Emergency corrections: 1h
- **Data residency**: Must support per-country storage jurisdiction configuration; at minimum: EEA, Brazil (sa-east-1), South Korea (ap-northeast-2), and UNFCCC Bonn
- **Protocol support**: REST/JSON, XML-over-HTTPS (SINARE-style), and a future gRPC path for modern registries
- **Availability**: CarbonLedger must remain operational when any single national registry is offline
- **Team**: 4 backend engineers, 3-week deadline for Swiss/Brazilian integration; full 196-country rollout over 12 months

### Your Task

Design the cross-border corresponding adjustment architecture. Include: the CA state machine (all states including HELD-IN-TRANSIT and DISPUTED), the data residency routing layer, the protocol adapter pattern for heterogeneous registry APIs, and the conflict resolution flow for split-decision CAs.

### Deliverables

- [ ] Mermaid architecture diagram: show the CA write path from CarbonLedger → seller registry → buyer registry → UNFCCC, with the protocol adapter layer and regional storage routing
- [ ] CA state machine diagram: all states (INITIATED, PENDING_SELLER, PENDING_BUYER, PENDING_UNFCCC, CONFIRMED, DISPUTED, HELD_IN_TRANSIT, CANCELLED), transitions, and timeout rules
- [ ] Database schema: `corresponding_adjustments` table, `registry_integrations` config table (per-country SLA, protocol type, jurisdiction), `ca_events` audit log
- [ ] Scaling estimation: at 1.5M cross-border transfers/day, what is the write fan-out? Calculate total writes across all three copies. What is the queue depth during a 6-hour national registry maintenance window?
- [ ] Tradeoff analysis (minimum 3): single authoritative source (UNFCCC canonical) vs. multi-master replication; synchronous vs. asynchronous CA confirmation; protocol abstraction layer vs. per-registry custom integrations
- [ ] Cost modeling: estimate monthly cost of the data residency layer (data transfer costs between regions + multi-region storage replication) at 1.5M cross-border transfers/day
- [ ] Conflict resolution protocol: write a structured decision tree for split-decision CAs (one registry confirms, another disputes)

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
