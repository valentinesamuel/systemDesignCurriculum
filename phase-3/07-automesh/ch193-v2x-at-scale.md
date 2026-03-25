---
**Title**: AutoMesh — Chapter 193: V2X at Scale
**Level**: Staff
**Difficulty**: 8
**Tags**: #v2x #safety-critical #messaging #real-time #distributed #api-design
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 194, Ch. 196, Ch. 199
**Exercise Type**: System Design
---

### Story Context

**#general — AutoMesh Engineering Slack**
*Monday, 8:47 AM*

**Dr. Priscilla Nakamura** [CTO]: Good morning everyone. Our new external consultant joins us today for the corridor architecture review. Please give them full access to the internal architecture docs, the FMEA drafts, and the San Francisco deployment contract exhibits. Let's make this week productive.

**Darius Okonkwo** [Principal Engineer, V2X]: Welcome. Fair warning: the architecture docs are... aspirational in places. I've been meaning to add a "this is what we built vs. what we planned" section for about six months.

**You**: Good to know. I'll read the aspirational docs first and then ask you to tell me where they diverge.

**Darius Okonkwo**: I like you already.

---

*10:30 AM — Conference Room B, AutoMesh HQ*

Dr. Nakamura sets up a slide. It shows a corridor map: 50 miles of I-280, 200 intersection nodes marked in blue, a density heat map of expected vehicle population.

**Dr. Nakamura**: Let me give you the full picture. The corridor contract covers three things: vehicle-to-infrastructure messaging — that's traffic signal data, road conditions, incident alerts going to vehicles; vehicle-to-vehicle collision avoidance — vehicles broadcasting their position, speed, and intent to nearby vehicles directly; and our cloud platform, which aggregates telemetry, feeds the city's traffic management system, and provides analytics.

**You**: What are the latency requirements for V2V collision avoidance?

**Dr. Nakamura**: NHTSA guidance says emergency braking messages need to reach nearby vehicles in under ten milliseconds. The DSRC standard assumes direct radio, peer-to-peer. You don't go through a server for that.

**You**: And you're routing it through your cloud platform.

*A pause.*

**Darius Okonkwo**: We're not routing V2V through cloud. The onboard units broadcast over DSRC radio. That part works. The problem is we also need to support C-V2X — cellular vehicle-to-everything — for vehicles that don't have DSRC hardware. The newer OEM vehicles, especially the passenger cars in the partnership discussions, use C-V2X LTE-PC5. The latency on that is higher, and it does go through network infrastructure, though not our cloud.

---

**#v2x-protocol-debate — AutoMesh Engineering Slack**
*Monday, 2:15 PM*

**Darius Okonkwo**: Okay so I want to surface the DSRC vs C-V2X debate again now that we have fresh eyes on it. DSRC (IEEE 802.11p) gives us sub-5ms direct V2V. C-V2X (3GPP PC5 mode) gives us 10-20ms but works on standard cellular chipsets. The new OEM vehicles are shipping C-V2X only. If we want the consumer fleet integration to work, we need C-V2X.

**Ifeoma Adeyemi** [Safety Engineer]: The problem is we certified our collision avoidance system against DSRC timings. If we introduce C-V2X vehicles into the corridor, they're operating on a different latency profile. The FMEA doesn't cover mixed-protocol environments.

**Darius Okonkwo**: Which is exactly why this needs to be in the architecture review.

**Tomás Reyes** [Infrastructure Lead]: The other thing nobody's said yet: our cloud platform receives telemetry from all vehicles — both DSRC and C-V2X — at 100 messages per vehicle per second. At 10,000 vehicles that's a million messages per second. I stress-tested the current ingestion layer at 800K/sec and it started dropping messages at around 650K.

**You**: How long has that number been known?

**Tomás Reyes**: Three months. I filed an issue. It's in the backlog.

**Ifeoma Adeyemi**: ...what?

**Tomás Reyes**: I assumed someone would prioritize it.

**Darius Okonkwo**: This is exactly the kind of thing I was talking about when I said "aspirational."

---

*Tuesday, 9:00 AM — Your desk. You open the architecture doc and start marking divergences.*

The current platform uses a custom UDP-based protocol for V2I messaging (intersection nodes to cloud), a Kafka cluster for telemetry ingestion, and a websocket layer for pushing safety alerts back to vehicles. The V2V layer is DSRC only — the C-V2X integration is listed as "Phase 2" with no implementation date. The ingestion cluster was sized for 100,000 messages per second, not 1,000,000.

You write in your notes: "Three problems before lunch. This is going to be a long engagement."

### Problem Statement

AutoMesh's V2X platform must handle communication between 10,000 connected vehicles and 200 intersection infrastructure nodes across a 50-mile corridor. The architecture must support two coexisting radio protocols (DSRC and C-V2X), maintain sub-10ms latency for safety-critical V2V collision avoidance messages, and ingest 1M telemetry messages per second into the cloud platform — a 10x increase over the current tested capacity.

You must design the full V2X communication infrastructure: the onboard protocol stack, the roadside unit (RSU) architecture, the cloud ingestion layer, and the control plane that ties it together. Every latency decision must be traceable to a safety requirement.

### Explicit Requirements

1. V2V collision avoidance messages: < 10ms latency, DSRC radio, peer-to-peer (not cloud-routed)
2. V2I safety messages (road closure, signal phase, incident alert): < 50ms end-to-end from infrastructure node to vehicle
3. Cloud telemetry ingestion: 1M messages/sec sustained, 1.5M/sec burst (for 60-second windows)
4. Support both DSRC (IEEE 802.11p) and C-V2X (3GPP LTE PC5) vehicle types in the same corridor simultaneously
5. Mixed-protocol safety: DSRC vehicles and C-V2X vehicles must receive collision avoidance data from each other (bridging required)
6. Infrastructure node heartbeat: each of 200 intersection nodes must report health status to cloud every 5 seconds
7. Cloud-to-vehicle push: safety alerts from cloud (incidents detected via aggregated telemetry) must reach all vehicles in a corridor zone within < 2 seconds
8. Message authentication: all V2I and cloud messages must be signed; DSRC uses certificate-based BSM (Basic Safety Message) format per SCMS (Security Credential Management System)

### Hidden Requirements

1. **Hint: re-read the Slack thread from Tomás Reyes.** He says the ingestion cluster "started dropping messages at around 650K" during a stress test at 800K target. What does this tell you about the failure mode — is it a hard limit or a graceful degradation? What happens to the dropped messages in a safety-critical context?

2. **Hint: re-read Ifeoma's message about FMEA.** She says the FMEA "doesn't cover mixed-protocol environments." If a DSRC vehicle emergency-brakes and broadcasts a collision avoidance message, a C-V2X-only vehicle in the same corridor segment will not receive that direct DSRC broadcast. What bridge must exist, and what is its latency budget?

3. **Hint: re-read Dr. Nakamura's contract description carefully.** She says the platform provides analytics to "the city's traffic management system." This is a third party with its own SLA requirements and API contracts. The city's system expects data in a specific format (NTCIP 1218 — the National Transportation Communications for ITS Protocol standard). This is not mentioned in the architecture doc.

4. **Hint: Darius says "Phase 2 with no implementation date" for C-V2X.** The consumer fleet partnership discussed in Ch. 196 uses C-V2X exclusively. If C-V2X integration is Phase 2, what happens when 900K consumer vehicles are onboarded? This is a strategic dependency, not just a technical one.

### Constraints

- **Fleet size**: 10,000 vehicles (commercial, corridor deployment); projected 100,000 within 12 months
- **Message rate**: 100 messages/vehicle/second for position + telemetry = 1M messages/sec baseline
- **V2V latency SLA**: < 10ms (DSRC, peer-to-peer, non-negotiable per NHTSA)
- **V2I latency SLA**: < 50ms (infrastructure to vehicle)
- **Cloud alert push SLA**: < 2,000ms (cloud detected incident to vehicle)
- **Infrastructure nodes**: 200 RSUs (Roadside Units), each covering ~400m radius
- **Protocol support**: DSRC (IEEE 802.11p) and C-V2X (LTE PC5 Mode 4)
- **Current ingestion capacity**: 650K messages/sec (stress test limit before drops)
- **Required ingestion capacity**: 1M sustained, 1.5M burst
- **Message size**: BSM (Basic Safety Message) = 256 bytes; telemetry message = 512 bytes
- **Security**: SCMS certificate provisioning for all vehicles; BSM signing required
- **Team size**: 12 engineers (V2X: 4, Infrastructure: 4, Platform: 4)
- **Timeline**: Corridor deployment in 90 days; architecture sign-off needed in 14 days
- **Budget**: $800K/month infrastructure ceiling (current: $210K/month)

### Your Task

Design the complete V2X communication infrastructure for the AutoMesh corridor deployment. Your design must cover:

1. The onboard unit (OBU) protocol stack — how a vehicle communicates over DSRC and/or C-V2X
2. The Roadside Unit (RSU) architecture — the 200 intersection nodes and their role
3. The DSRC-to-C-V2X bridge for mixed-protocol safety coverage
4. The cloud ingestion layer capable of 1M+ messages/sec
5. The cloud-to-vehicle alert push path (< 2,000ms SLA)
6. The security model (SCMS certificate flow, message signing, replay prevention)
7. The NTCIP 1218 integration for the city's traffic management system

### Deliverables

- [ ] Mermaid architecture diagram showing all communication paths: V2V, V2I, OBU-to-cloud, cloud-to-vehicle alert push, NTCIP integration
- [ ] Database schema for: vehicle registry, message audit log, RSU health table, SCMS certificate store (with column types and indexes)
- [ ] Scaling estimation (show math step by step):
  - Messages/sec per vehicle × fleet size = ingestion rate
  - Kafka partition sizing for 1M messages/sec
  - RSU broadcast radius coverage for 200 nodes over 50 miles
  - Cloud alert fan-out: 10K vehicles in < 2,000ms — how many parallel push workers needed?
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - DSRC vs C-V2X for safety-critical V2V
  - UDP vs TCP for telemetry ingestion (reliability vs throughput)
  - Centralized vs distributed RSU intelligence
- [ ] DSRC-to-C-V2X bridge design: latency analysis proving the bridge can meet the < 10ms mixed-protocol safety requirement (or documenting why it cannot and what the fallback behavior is)
- [ ] Cost modeling: Kafka cluster sizing, RSU hardware amortization, cellular data costs for C-V2X telemetry — estimate total infrastructure cost at 10K vehicles and 100K vehicles
- [ ] Capacity planning: what architectural changes are needed to scale from 10K to 100K vehicles?

### Diagram Format

All architecture diagrams must be in Mermaid syntax (renders in GitHub Issues).

```
Example skeleton (expand significantly):
graph TD
    V[Vehicle OBU] -->|DSRC BSM broadcast| RSU[Roadside Unit]
    V -->|C-V2X PC5| RSU
    RSU -->|V2I safety alert| V
    RSU -->|telemetry forwarding| INGEST[Cloud Ingestion Layer]
    INGEST -->|Kafka| PLATFORM[AutoMesh Platform]
    PLATFORM -->|NTCIP 1218| CITY[City Traffic Management]
    PLATFORM -->|alert push| V
```
