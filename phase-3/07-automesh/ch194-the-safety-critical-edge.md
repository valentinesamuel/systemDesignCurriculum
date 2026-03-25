---
**Title**: AutoMesh — Chapter 194: The Safety-Critical Edge
**Level**: Staff
**Difficulty**: 9
**Tags**: #edge-computing #safety-critical #fmea #nhtsa #iso26262 #distributed
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 193, Ch. 199
**Exercise Type**: System Design
---

### Story Context

**AUTOMESH INTERNAL — FMEA Workshop Transcript**
*Wednesday, Day 3 of Engagement*
*Attendees: You (consultant), Dr. Priscilla Nakamura (CTO), Ifeoma Adeyemi (Safety Engineer), Darius Okonkwo (Principal Engineer), Marcus Osei (Safety Lead), Tomás Reyes (Infrastructure Lead)*
*Location: War Room, AutoMesh HQ*

---

**Marcus Osei**: Okay. Let's go through the FMEA table. I'll read each failure mode. You tell me what happens. Simple.

**Marcus Osei**: Failure Mode 1: RSU loses power — intersection node goes dark. Failure effect: vehicles no longer receive V2I signal phase data. Mitigation?

**Ifeoma Adeyemi**: Vehicles fall back to default intersection behavior — treat it as an uncontrolled intersection, reduce speed to 15mph. That's in the OBU firmware.

**Marcus Osei**: Good. That's documented. Failure Mode 2: GPS spoofing — attacker broadcasts false vehicle position to surrounding vehicles. Failure effect: collision avoidance decisions based on phantom vehicle positions.

**Darius Okonkwo**: SCMS certificate validation. A spoofed vehicle can't produce a valid signed BSM. We reject unsigned or invalid-cert messages at the OBU.

**Marcus Osei**: Good. Mitigation documented. Failure Mode 3: Kafka ingestion saturation — cloud platform falls behind on processing telemetry. Failure effect: safety alerts computed from telemetry are delayed or not computed.

*You lean forward.*

**You**: What happens to vehicles in the corridor when the alert pipeline is backed up?

**Tomás Reyes**: They continue operating normally. They don't know there's a delay.

**You**: So if there's a debris field forming in lane 2, we detect it from aggregated telemetry... but if we're saturated, the alert to downstream vehicles might come 30 seconds late instead of 2 seconds.

**Tomás Reyes**: ...yes.

**Marcus Osei**: That's TBD in the mitigation column. Failure Mode 4: OBU hardware fault — onboard unit crashes mid-drive. Failure effect: vehicle can no longer send or receive V2V or V2I messages.

**Ifeoma Adeyemi**: Vehicle reverts to standard ADAS sensors only. No V2X assistance. That's acceptable.

**Marcus Osei**: Failure Mode 5: Cloud connectivity loss — vehicle cannot reach AutoMesh platform.

*The room goes quiet.*

**Marcus Osei** *(quietly)*: What does the vehicle do?

*Nobody answers for four seconds.*

**Darius Okonkwo**: The vehicle can still receive DSRC broadcasts directly from other vehicles and RSUs. Local radio is unaffected.

**You**: That's V2V and V2I. What about any safety logic that runs on the cloud platform and pushes decisions back to the vehicle?

**Darius Okonkwo**: We have... some of that.

**You**: How much?

**Darius Okonkwo**: Route safety scoring. Hazard zone flagging. Some of the intersection priority logic for emergency vehicles.

**You**: So if the vehicle loses cloud connectivity, those features go dark.

**Darius Okonkwo**: Yes.

**Marcus Osei**: That's TBD in the mitigation column.

---

*The workshop continues for another ninety minutes. By the end, the FMEA table reads as follows (abbreviated):*

| # | Failure Mode | ASIL | Mitigation |
|---|---|---|---|
| 1 | RSU power loss | B | Documented: fallback to uncontrolled intersection |
| 2 | GPS spoofing | D | Documented: SCMS cert validation |
| 3 | Ingestion saturation | C | **TBD** |
| 4 | OBU hardware fault | B | Documented: ADAS fallback |
| 5 | Cloud connectivity loss | C | **TBD** |
| 6 | V2V message storm (broadcast flood) | B | **TBD** |
| 7 | Stale map tile in OBU | C | Documented: tile expiry timestamp, OBU rejects stale tiles |
| 8 | Cellular handoff during safety event | B | Documented: DSRC maintained during handoff |
| 9 | RSU firmware compromise | D | Documented: signed firmware, TPM verification |
| 10 | OBU clock drift beyond 10ms | D | **TBD** |
| 11 | Message replay attack | C | Documented: timestamp window + sequence number |
| 12 | Concurrent emergency vehicle preemption conflict | D | **TBD** |

---

*After the workshop. Conference room empties. You and Marcus Osei are alone.*

**Marcus Osei**: Five TBDs out of twelve. NHTSA won't sign off on this. ISO 26262 requires every failure mode to have a documented mitigation with a defined safe state. "TBD" is not a safe state.

**You**: What's the certification timeline?

**Marcus Osei**: We're supposed to submit the safety case in six weeks. If we go in with five TBDs, they'll reject it. Reject means no corridor deployment. No corridor deployment means the city clawbacks the $20M upfront payment.

*He looks at you.*

**Marcus Osei**: I've been saying this in internal reviews for four months. Nobody prioritized it because the architecture team said it was "platform-level" not "safety-level." I need a second opinion.

**You**: It's both.

**Marcus Osei**: I know. Tell that to the org chart.

---

*You spend the next two hours reading ISO 26262 Part 6 and writing in your notebook. The core problem: five failure modes have no defined safe state. Defining them isn't just a documentation exercise — it requires architectural decisions about what the vehicle must be able to do without the cloud.*

### Problem Statement

Five of twelve safety-critical failure modes in AutoMesh's FMEA have no defined mitigation. ISO 26262 ASIL compliance and NHTSA certification both require complete coverage. The root cause is architectural: safety-critical decisions that should run at the edge (on the vehicle's onboard unit) are instead dependent on cloud connectivity. When cloud is unavailable, the vehicle has no defined behavior for several failure modes.

You must design the edge computing architecture for the AutoMesh OBU (Onboard Unit) that enables vehicles to make safety-critical decisions autonomously — with defined safe states for all five open failure modes — without requiring cloud connectivity for any ASIL-C or ASIL-D function.

### Explicit Requirements

1. All 12 FMEA failure modes must have documented mitigations — no TBDs
2. ASIL-D functions (GPS spoofing, RSU firmware compromise, OBU clock drift, emergency vehicle conflict) must have mitigations that do not depend on cloud connectivity
3. ASIL-C functions (ingestion saturation, cloud connectivity loss) must have defined safe states that the vehicle enters autonomously
4. Edge safety decisions (collision avoidance, hazard zone entry/exit) must execute in < 10ms on the OBU hardware
5. Cloud connectivity state must be continuously monitored; graceful degradation on connectivity loss within 500ms of detection
6. Emergency vehicle preemption conflict: if two emergency vehicles simultaneously request intersection priority from opposing directions, the OBU must have a conflict resolution protocol that does not require cloud arbitration
7. OBU clock drift: if GPS synchronization is lost and the OBU clock drifts beyond a defined threshold, the DSRC message validity window must adjust accordingly — V2V messages must not be silently invalidated due to clock skew
8. V2V broadcast storm (message flood): OBU must implement backpressure to prevent buffer overflow under dense vehicle conditions (e.g., accident scene with 50+ vehicles within 400m)

### Hidden Requirements

1. **Hint: re-read the FMEA table row for OBU clock drift (Failure Mode 10, ASIL-D).** DSRC BSM messages use timestamps with a validity window. If an OBU clock drifts > 10ms from GPS time, it starts rejecting valid messages from other vehicles and its own messages get rejected by others. What happens to V2V collision avoidance when this occurs? How does the edge architecture handle a vehicle that is effectively deaf to surrounding V2V traffic?

2. **Hint: re-read Marcus Osei's description of the org chart problem.** He says the architecture team classified these as "platform-level" not "safety-level." This means the current architecture has safety-critical logic in services that are not ISO 26262 certified. Any component in the safety critical path must itself be certified to the appropriate ASIL level. The cloud platform was not built to ISO 26262 standards. This has implications for what can legally sit in the safety path.

3. **Hint: re-read Darius's list of cloud-dependent features.** "Route safety scoring. Hazard zone flagging. Some of the intersection priority logic for emergency vehicles." These must either be classified as non-safety (ASIL QM — quality management, no safety requirement) or moved to the edge. Which of these could cause a fatality if unavailable? That classification determines ASIL level.

4. **Hint: re-read Failure Mode 12 — "concurrent emergency vehicle preemption conflict."** The ASIL-D rating means a failure here could cause a fatality. If two ambulances approach the same intersection from opposite directions simultaneously and both request green-light priority, the current system sends this conflict to the cloud for arbitration. Cloud round-trip is 80-120ms. Signal phase change requires 3 seconds of lead time minimum. Do the math.

### Constraints

- **OBU hardware**: ARM Cortex-A72 quad-core, 8GB RAM, 64GB eMMC storage (existing hardware, cannot change)
- **ASIL-D safety functions**: must execute in < 10ms, no cloud dependency allowed
- **ASIL-C safe state entry**: must complete within 500ms of failure detection
- **V2V broadcast storm**: assume max 200 vehicles within a 400m radius (accident scene)
- **OBU power budget**: safety subsystem must survive 30 seconds on capacitor backup if main power fails (relevant for vehicle crash scenario)
- **ISO 26262 Part 6**: software ASIL decomposition rules apply
- **Connectivity monitoring**: cellular + DSRC health check every 100ms
- **FMEA coverage requirement**: all 12 failure modes, ASIL-A through ASIL-D
- **Emergency vehicle preemption**: intersection signal change requires 3s lead time; cloud round-trip is 80-120ms; time budget for edge decision is 500ms total
- **Clock drift tolerance**: DSRC BSM timestamp validity window is ±20ms; GPS sync loss should be detectable within 500ms

### Your Task

Design the safety-critical edge computing architecture for the AutoMesh OBU. Your design must:

1. Define the edge safety compute layer — what runs on the OBU without cloud connectivity
2. Provide documented mitigations for all 5 open FMEA items (Failure Modes 3, 5, 6, 10, 12)
3. Show the ASIL decomposition — which functions are ASIL-D, ASIL-C, ASIL-B, and ASIL QM
4. Define the cloud connectivity state machine: Connected → Degraded → Disconnected and what changes at each state transition
5. Design the emergency vehicle preemption conflict resolution protocol for edge execution
6. Design the clock drift compensation mechanism

### Deliverables

- [ ] Mermaid architecture diagram: OBU internal component diagram showing ASIL partitioning (safety kernel vs. non-safety processes), connectivity to RSU and cloud, and the safe state machine
- [ ] Complete FMEA table with all 12 rows filled in — no TBDs. For each of the 5 open items, specify the mitigation and the architectural component that implements it
- [ ] Database schema for: OBU state log (edge-side), safety event log (edge-side, synced to cloud when connectivity restored), ASIL function registry (columns + indexes)
- [ ] Scaling estimation (show math step by step):
  - V2V message processing: 200 vehicles × 10 BSMs/sec = 2,000 messages/sec on the OBU receive buffer. At 256 bytes/message, what is the OBU memory requirement for a 1-second backpressure window?
  - Emergency preemption: 3s signal lead time − 500ms edge processing budget − 80ms RSU propagation = margin math
  - Clock drift detection: GPS sync check at 100ms intervals, maximum drift rate for a typical TCXO (temperature-compensated crystal oscillator): 0.5ppm = 0.5μs per second. How long until drift exceeds the 10ms threshold?
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Cloud-dependent safety logic vs. edge-only safety logic (latency vs. sophistication)
  - ASIL decomposition: one ASIL-D function vs. two parallel ASIL-B functions (ISO 26262 decomposition rule)
  - Message storm backpressure: drop oldest vs. drop by type priority
- [ ] ASIL decomposition table: list every safety function, its ASIL level, its location (edge/cloud/both), and its safe state on failure
- [ ] Cost modeling: OBU compute upgrade path if the current ARM Cortex-A72 is insufficient for ASIL-D timing requirements; estimated cost per vehicle for a hardware safety co-processor (lockstep ARM Cortex-R5 pair is common in ASIL-D automotive)
- [ ] Capacity planning: as fleet grows from 10K to 100K vehicles, what edge state synchronization load does cloud infrastructure receive when vehicles reconnect after extended disconnection?

### Diagram Format

All architecture diagrams must be in Mermaid syntax.

```
Example skeleton (expand significantly):
stateDiagram-v2
    [*] --> Connected
    Connected --> Degraded: connectivity_loss_detected (>500ms)
    Degraded --> Disconnected: connectivity_loss_confirmed (>5s)
    Disconnected --> Degraded: connectivity_restored
    Degraded --> Connected: full_sync_complete

    note right of Disconnected
        ASIL-D functions: edge only
        ASIL-C functions: safe state active
        Cloud features: suspended
    end note
```
