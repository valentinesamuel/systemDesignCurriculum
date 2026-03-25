---
**Title**: OrbitCore — Chapter 156: SPIKE — Solar Flare
**Level**: Staff
**Difficulty**: 10
**Tags**: #incident-response #production-down #failover #resilience #itar #data-sovereignty #spike
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 151 (cell architecture), Ch. 152 (sovereignty in orbit), Ch. 155 (capacity planning)
**Exercise Type**: Incident Response
---

### Story Context

It is April 14th, 02:17 UTC. You are asleep.

**[PagerDuty — 02:17:44 UTC]**
```
CRITICAL — P1 INCIDENT
Service: GS-Singapore — Telemetry Ingest
Alert: Ingest rate: 0 events/sec (threshold: > 100,000 events/sec)
Satellites affected: KEP-07, KEP-12, KEP-18 (primary), SV-23 (partial)
Acknowledging engineer: NONE (auto-escalate in 5 minutes)
```

You acknowledge at 02:19 UTC.

---

**[#incidents — Slack, 02:19:47 UTC]**

> **[PagerDuty Bot]**
> P1 INCIDENT OPEN: GS-Singapore complete telemetry outage. Satellites KEP-07, KEP-12, KEP-18, SV-23 affected. Auto-escalation triggered. War room: https://orbitcore.zoom.us/j/9928374412

> **Preet Kapila** [02:20:12]
> On the bridge. Pulling logs now.

> **You** [02:21:04]
> On bridge. What do we have?

> **Preet Kapila** [02:21:33]
> GS-Singapore infrastructure appears UP — EC2 instances responding to health checks. But ingest workers are all in STALLED state. Kafka brokers at the Singapore cluster are healthy. The stall is at the ground station receiver hardware — the L-band antenna system is reporting hardware fault code 0xE7.

> **Niamh Okafor** [02:22:08]
> Pulling hardware manual for 0xE7. One sec.
> 0xE7: "Receiver overload — excessive RF input power. Automatic protective shutdown engaged."
> This is the antenna protecting itself from damage. High RF input = solar storm.

> **You** [02:22:45]
> Space weather event?

> **Dr. Yusuf Adeyemi** [02:23:11]
> Checking NOAA SWPC. YES — X2.1 class solar flare, peaked at 02:09 UTC. Southeast Asia is in the affected corridor. This is not a software problem. Singapore's antenna hardware is physically overloaded by the RF interference. It may be in protective shutdown for 45 minutes to 2 hours depending on flare duration.

> **Preet Kapila** [02:23:55]
> Checking satellite positions:
> - KEP-07: visible from Singapore AND Sydney. Can failover to Sydney.
> - KEP-12: visible from Singapore only for next 14 minutes, then enters Australian window.
> - KEP-18: mid-pass over Singapore. No other station has LOS for 22 minutes.
> - SV-23: visible from Singapore. NEXT US station visibility window: GS-US-West at 02:51 UTC. That's 32 minutes away.

> **Ingrid Solberg** [02:24:30]
> @you — I need someone to own this. What's our plan?

> **You** [02:24:58]
> I'm owning it. Starting incident timeline now.

---

**[INCIDENT TIMELINE — INC-2026-0414-001]**
**Incident Commander:** You
**Start:** 02:17:44 UTC
**Bridge:** https://orbitcore.zoom.us/j/9928374412

```
02:17:44  PagerDuty fires: GS-Singapore ingest rate = 0
02:19:00  [You] acknowledge
02:19:47  War room opened. Preet, Niamh, Yusuf, Ingrid on bridge.
02:21:33  Root cause: antenna hardware fault 0xE7 (RF overload, solar flare)
02:23:11  Confirmed: X2.1 solar flare, NOAA. Singapore in affected corridor.
02:23:55  Satellite visibility window analysis complete (Preet)
02:24:58  You: incident commander declared
```

---

**[Bridge — continued, 02:25 UTC]**

> **You:** Okay. Three immediate problems.
>
> Problem 1: KEP-07. Sydney can take it. Route it now. Preet, can you update the routing table?
>
> Problem 2: KEP-12 and KEP-18. No current alternative ground station visibility. We're going to lose up to 22 minutes of telemetry for KEP-18 and 14 minutes for KEP-12. Those satellites buffer onboard?

> **Niamh Okafor** [02:25:44]
> Yes. KEP-12 has 45-minute onboard buffer. KEP-18 has 60-minute onboard buffer. We won't lose the data — it'll be transmitted when a ground station acquires signal.

> **You** [02:26:01]
> Good. But we need a process to ingest the buffered replay when Sydney acquires KEP-12 and KEP-18. Is the replay ingestion path tested?

> **Niamh Okafor** [02:26:22]
> It exists. I don't know when it was last tested.

> **You** [02:26:38]
> We'll find out. Problem 3 — and this is the hard one. SV-23.

> [Silence on bridge for 3 seconds]

> **Dr. Yusuf Adeyemi** [02:27:01]
> The ITAR channel. SV-23 channel 4 cannot route to any non-US station.

> **You** [02:27:18]
> Correct. GS-US-West has LOS at 02:51. That's 24 minutes. What does SV-23 do for 24 minutes? Does it have onboard buffer?

> **Niamh Okafor** [02:27:44]
> SV-23 onboard buffer: I'm checking the satellite spec sheet.
> [pause]
> SV-23 has a 30-minute onboard buffer for the housekeeping channels (1-3). The payload channel (4) — the ITAR channel — the spec says "continuous downlink mode, no onboard storage."

> **You** [02:28:15]
> Say that again.

> **Niamh Okafor** [02:28:19]
> Channel 4 on SV-23 is designed for continuous real-time downlink. There is no onboard storage for channel 4. If we can't receive it, it's gone.

> **You** [02:28:40]
> So we have a 24-minute window where ITAR-classified telemetry from SV-23 channel 4 is being emitted but we cannot receive it from any US ground station. And we cannot route it to a non-US station.

> **Preet Kapila** [02:29:03]
> Correct.

> **You** [02:29:10]
> What does the Kepler Dynamics contract say about ITAR channel data loss?

> **Ingrid Solberg** [02:29:28]
> I'm pulling it up. One second.
> [45 second pause]
> Section 7.4: "Operator warrants best-effort continuity for payload channel data. Force majeure events including but not limited to space weather, ground station hardware failure, and RF interference are excluded from SLA obligations."
> We're covered on the SLA. But the data is still lost.

> **You** [02:30:05]
> Here's what I need to know: is there ANY US-sovereign ground station anywhere on Earth that has LOS to SV-23 right now? Not our stations. Anyone else's.

> **Dr. Yusuf Adeyemi** [02:30:44]
> That would require... AWS Ground Station service, or commercial providers like Atlas Space Operations or Leaf Space. We'd need an emergency account and SV-23's TLE data to task them. I don't know if we can do that in 24 minutes.

> **You** [02:31:02]
> Can we try?

> **Dr. Yusuf Adeyemi** [02:31:15]
> I can call Atlas. I have a contact there. No guarantees.

> **You** [02:31:22]
> Make the call. Log everything. Even if it doesn't work, I want the record that we tried.
>
> Meanwhile: Preet, route KEP-07 to Sydney NOW. Niamh, start testing the buffered replay ingest path against a test satellite. I want to know it works before we need it.
>
> Timeline stamp all of this. We're going to need to write this up for Kepler Dynamics.

---

**[Bridge — 02:51 UTC]**

> **Preet Kapila** [02:51:07]
> GS-US-West has acquired SV-23. Housekeeping telemetry (channels 1-3) is flowing. Channel 4 is silent — the satellite continued to transmit but there was nobody to receive it for 22 minutes.

> **Dr. Yusuf Adeyemi** [02:51:44]
> Atlas Space Operations couldn't task fast enough. Their next available contact pass was 03:15 UTC. By then SV-23 will be in US-West's window anyway.

> **Niamh Okafor** [02:52:01]
> KEP-12 and KEP-18 buffered replay is working. Sydney is ingesting the onboard buffers now. Data integrity checks passing.

> **You** [02:52:20]
> Singapore hardware status?

> **Preet Kapila** [02:52:38]
> Still in protective shutdown. Solar flare RF levels are still elevated. Estimated clear: 03:45-04:00 UTC.

> **You** [02:52:55]
> Set up automatic recovery trigger when Singapore hardware clears. Don't make someone manually restart it at 4am.
> I'm writing the incident timeline. Bridge stays open until Singapore is back.
> Outstanding questions for the postmortem:
> 1. Why does SV-23 have no onboard buffer for channel 4? Is that a satellite design choice or something Kepler Dynamics can change in firmware?
> 2. Why didn't our failover automation route KEP-07 to Sydney automatically? (We did it manually)
> 3. Is there a commercial ground station provider we should maintain a standing emergency agreement with for exactly this scenario?
> 4. Solar flare season is April through September. This WILL happen again.

### Problem Statement

A X2.1-class solar flare has caused GS-Singapore to enter hardware protective shutdown, taking three active Kepler satellites (KEP-07, KEP-12, KEP-18) and the ITAR-restricted SV-23 partially offline. KEP-07 can be rerouted to Sydney. KEP-12 and KEP-18 rely on onboard buffers. SV-23's ITAR channel (channel 4) has no onboard storage and cannot be received during the 22-minute gap before GS-US-West acquires it.

Design OrbitCore's ground station failover architecture that handles this scenario automatically and preventively: automatic cell failover with sovereignty enforcement, ITAR-channel buffer design (or alternative for the no-onboard-storage constraint), solar interference prediction and pre-positioning, and the post-incident architecture changes that prevent this specific failure mode from recurring.

### Explicit Requirements

1. Automatic failover from a failed cell to the best available cell must trigger within 30 seconds (current: manual)
2. Failover routing must enforce sovereignty constraints — routing SV-23 to Sydney must be architecturally impossible, not just policy-enforced
3. For ITAR-channel satellites with no onboard storage: design a "contact window pre-positioning" system — identify when the ITAR satellite will have no US ground station coverage and pre-task an emergency commercial ground station provider
4. Solar flare prediction integration: NOAA SWPC provides solar weather forecasts — integrate these into the capacity scheduler to pre-position satellite coverage before a predicted flare window
5. Buffered replay ingest must be a tested, automated process — not a manually triggered script
6. Incident runbook must be embedded in the automation: no manual steps for known failure scenarios
7. Post-incident: the 22-minute ITAR channel gap must be root-caused and mitigated

### Hidden Requirements

1. **Hint: re-read Niamh's message about SV-23's channel 4.** "Continuous downlink mode, no onboard storage" — this is a satellite design characteristic, not a software limitation. You cannot fix it in your ground infrastructure. But your architecture review should include a recommendation to Kepler Dynamics about whether a firmware OTA update to enable channel 4 buffering is possible. How do you frame that recommendation without overstepping into their satellite design decisions?

2. **Hint: re-read the incident timeline.** The automatic failover didn't trigger. You had to manually update the routing table for KEP-07. The cell architecture from Chapter 151 defined a 30-second RTO. Was that RTO met? What was the actual time between GS-Singapore going dark (02:17) and KEP-07 routing to Sydney? Calculate from the timeline. Was this a design failure, a configuration failure, or a process failure?

3. **Hint: re-read Dr. Adeyemi's message about the X2.1 flare.** NOAA SWPC provides 1-3 day solar weather forecasts for X-class flares. The April flare season is predictable — Adeyemi warned about it in March (Ch. 155). If you had integrated SWPC forecasts, you could have pre-positioned satellite coverage windows before the flare hit. Design this proactively.

4. **Hint: re-read the incident commander's outstanding questions at 02:52.** Question 3: "Is there a commercial ground station provider we should maintain a standing emergency agreement with?" This is a business and architecture question. Atlas Space Operations couldn't task fast enough for an emergency. A pre-negotiated emergency services agreement with a standing TLE database of your satellites would solve this. What is the cost/benefit of this agreement?

### Constraints

- **RTO requirement**: 30 seconds for automatic cell failover
- **SV-23 channel 4 gap**: 22 minutes of lost ITAR data during this incident
- **SV-23 constraint**: No onboard storage for channel 4 — architectural constraint, not configurable
- **Solar flare forecast**: NOAA SWPC provides 24-72 hour advance warning for X-class flares (not always reliable, but useful)
- **KEP-12, KEP-18 onboard buffer**: 45-60 minutes — sufficient for most single-station outages
- **Atlas Space Operations emergency tasking**: too slow for reactive response; needs pre-agreement
- **Solar flare season**: April–September (recurring, predictable seasonally)
- **Current failover**: manual (this incident revealed it); target: automatic
- **Team on-call**: 2 SREs globally (cannot assume human response faster than 5 minutes at 2am)

### Your Task

Design the post-incident failover architecture for OrbitCore's ground station network. Cover: automatic failover automation with sovereignty enforcement, ITAR pre-positioning system, solar weather integration, commercial ground station emergency agreement architecture, buffered replay automation, and the post-incident report structure (blameless, findings-driven).

### Deliverables

- [ ] Mermaid architecture diagram showing:
  - Automatic cell failover pipeline (health check → sovereignty check → failover trigger → routing update)
  - ITAR pre-positioning system (SWPC forecast → coverage gap analysis → commercial GS task)
  - Solar weather integration data flow
  - Buffered replay ingest automation
- [ ] Incident timeline reconstruction:
  - Annotate the incident timeline with what should have been automatic vs. what was manual
  - Calculate the actual RTO for KEP-07 from timeline (02:17 incident start to KEP-07 routing to Sydney — when did that happen?)
  - Identify every manual step and its equivalent automated action
- [ ] Automatic failover design:
  - Health check protocol for cell status detection
  - Failover decision logic (sovereignty-enforced routing candidate selection)
  - How is the 30-second RTO achieved with sovereignty constraints?
  - What does failover for SV-23 look like (constraint: only US cells, and US cells may all be unavailable)?
- [ ] ITAR pre-positioning system design:
  - Coverage gap prediction algorithm (satellite TLE + ground station locations + sovereignty rules)
  - Integration with NOAA SWPC solar forecast API
  - Commercial ground station emergency tasking API (e.g., AWS Ground Station, Atlas Space Ops)
  - Pre-negotiated agreement model: what SLA do you need from the commercial provider?
- [ ] SV-23 channel 4 gap mitigation options:
  - Option A: Commercial GS emergency agreement (cover the gap with third-party US station)
  - Option B: Request Kepler Dynamics firmware update to enable channel 4 buffering
  - Option C: Accept the gap as force majeure and improve early warning to reduce frequency
  - Recommend with rationale
- [ ] Blameless post-incident report structure:
  - Timeline
  - Impact
  - Root cause (hardware + process)
  - Contributing factors
  - Action items (with owners and deadlines)
  - What went well
- [ ] Tradeoff analysis (minimum 3):
  - Pre-positioned commercial GS agreement (proactive) vs. emergency reactive tasking
  - Strict sovereignty (drop ITAR data if no US station) vs. sovereign-buffer (hold ITAR data in US-bound encrypted queue until a US station becomes available)
  - Solar flare prediction integration (reduce gap frequency) vs. onboard buffer firmware request (eliminate gap for new satellites)
- [ ] Cost modeling:
  - Commercial GS emergency agreement cost (estimate: $5K-$50K/year depending on SLA)
  - Additional US ground station deployment cost (would a 4th US station eliminate this gap?)
  - Cost of 22 minutes of ITAR channel data loss per incident (contractual value? mission value?)
- [ ] Capacity planning update:
  - How does solar flare season affect the 18-month capacity plan from Chapter 155?
  - How many additional coverage incidents should you plan for between April and September?
  - What infrastructure changes from this incident feed into the Series C roadmap?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

Key diagram: the automated failover state machine with sovereignty enforcement gates.
