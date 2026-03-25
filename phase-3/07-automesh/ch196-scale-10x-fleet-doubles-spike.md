---
**Title**: AutoMesh — Chapter 196: Scale 10x — Fleet Doubles
**Level**: Staff
**Difficulty**: 9
**Tags**: #scaling #capacity-planning #cost-engineering #spike #consumer-vehicles
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 193, Ch. 195, Ch. 198
**Exercise Type**: System Design (Scale 10x Spike)
---

### Story Context

**#general — AutoMesh Engineering Slack**
*Thursday, 9:03 AM*

**Dr. Priscilla Nakamura**: @channel — big news dropping today at 11AM EST. Press embargo until then. Stand by.

**Tomás Reyes**: ...that's ominous.

**Darius Okonkwo**: Priscilla when you say "big news" do you mean good-big or oh-no-big?

**Dr. Priscilla Nakamura**: Both.

---

*11:02 AM — AutoMesh Twitter/X account*

> AutoMesh and Meridian Motors announce strategic partnership to bring connected vehicle safety features to Meridian's 2026 and 2027 model year passenger vehicles. AutoMesh's V2X platform will be integrated across Meridian's entire North American consumer fleet — approximately 900,000 vehicles currently on road, with 120,000 new vehicles sold annually. Onboarding begins in 90 days.

---

**#infrastructure — AutoMesh Engineering Slack**
*11:07 AM*

**Tomás Reyes**: so

**Tomás Reyes**: nine hundred thousand

**Ifeoma Adeyemi**: I saw. I'm reading it again to make sure I'm reading it correctly.

**Tomás Reyes**: we currently run 100K vehicles. this is 10x.

**Tomás Reyes**: also: these are CONSUMER vehicles. personal passenger cars. they are not commercial fleet. they behave completely differently.

**Darius Okonkwo**: Different how?

**Tomás Reyes**: Commercial fleet is 24/7. Trucks run 18-22 hours/day, relatively flat load. Our infrastructure was sized for that. Consumer vehicles are parked 95% of the time. But during the 5% they're driving? It's ALL AT THE SAME TIME. 8AM to 9AM. 5PM to 6:30PM. Rush hour. We have ZERO experience with rush hour load profiles.

---

**DM — Dr. Priscilla Nakamura → You (consultant)**
*11:14 AM*

**Dr. Priscilla Nakamura**: You saw the announcement. I need you in my office at 1PM with a preliminary read on what this means for infrastructure. The board has a call Friday at 4PM and they're going to ask me directly whether we can handle this.

**You**: I'll be there.

---

**DM — Dr. Priscilla Nakamura → You (consultant)**
*Thursday, 5:42 PM*

**Dr. Priscilla Nakamura**: How bad is it?

**You**: The architecture can handle 1M vehicles. With significant work. The consumer traffic pattern is the harder problem — your current system was sized for flat commercial load. Rush hour will spike to 4-5x your average message rate for about 90 minutes, twice a day. That's the capacity cliff.

**Dr. Priscilla Nakamura**: What's the cliff look like in dollars?

**You**: I'll have numbers by tomorrow morning. But preliminary: you'll need to rethink your Kafka partition strategy, your ingestion autoscaling configuration, and you need to have a conversation with your CDN provider about burst pricing. The OTA system we just redesigned is also undersized for 1M vehicles.

**Dr. Priscilla Nakamura**: The board wants a 12-month scaling plan.

**You**: They're going to get one.

---

**DM — Dr. Priscilla Nakamura → You (consultant)**
*Friday, 7:15 AM*

**Dr. Priscilla Nakamura**: I need the plan in 4 hours. Board call at 11. Can you do it?

**You**: Already working on it. I'll have a draft by 10.

**Dr. Priscilla Nakamura**: One more thing. Legal just sent me the partnership term sheet. The onboarding isn't 90 days — it's 90 days to start onboarding. The full 900K vehicles needs to be onboarded within 6 months. Meridian's contract has a penalty clause: $500K/week if we miss the onboarding SLA.

*You look at your notes. The numbers you've been running assume a gradual 90-day onboarding. 6 months for 900K vehicles is 150K/month. That's 5,000 vehicles per day at steady state. Your current vehicle onboarding pipeline can handle 200 per day.*

**You**: How long has legal known about the 6-month clause?

**Dr. Priscilla Nakamura**: Since Tuesday apparently.

**You**: ...I'll have the plan by 10.

---

*You stare at a blank spreadsheet. The problem has three parts: capacity (can the infrastructure handle 1M vehicles?), traffic patterns (can it handle consumer rush-hour spikes?), and onboarding velocity (can it enroll 5,000 vehicles/day?). None of these were true yesterday. All three need to be true in 6 months.*

*You open the Kafka configuration file and start reading partition counts.*

### Problem Statement

AutoMesh must scale its connected vehicle platform from 100K commercial fleet vehicles to 1M mixed fleet (100K commercial + 900K consumer) within 6 months. Consumer vehicles behave fundamentally differently from commercial fleet: instead of flat 24/7 load, they produce rush-hour spikes of 4-5× average load for 90 minutes twice daily. The current infrastructure was sized exclusively for commercial flat-load patterns and will fail its first rush hour after consumer onboarding begins.

Additionally, the vehicle onboarding pipeline must scale from 200 vehicles/day to 5,000 vehicles/day (25× increase) — a separate engineering problem with a contractual penalty if missed.

You have 72 hours to produce a 6-month scaling plan for the board.

### Explicit Requirements

1. Infrastructure must handle 1M vehicles: 1M × 100 messages/sec = 100M messages/sec peak during rush hour (at 5× average, average = 20M/sec)
2. Kafka ingestion layer must handle 100M messages/sec peak during rush-hour windows (90 minutes, twice daily)
3. Consumer vehicle traffic model: rush hour (8–9AM, 5–6:30PM) = 5× average; overnight (11PM–6AM) = 0.1× average; shoulder periods = 1× average
4. Vehicle onboarding pipeline: 5,000 vehicles/day sustained (vs current 200/day)
5. HD map distribution (Ch. 197 system) must scale to serve 1M vehicles with differential updates
6. OTA system (Ch. 195 system) must scale to deliver firmware updates to 1M vehicles
7. Consumer vehicles have different data profiles: shorter trips (avg 22 min vs commercial avg 4 hours), more frequent engine-on/off cycles, and V2X telemetry only while driving
8. Consumer vehicle SLA: the Meridian Motors contract specifies 99.5% uptime (V2X feature availability) for consumer vehicles — lower than the 99.9% commercial fleet SLA, but measured by the 90M-vehicle-minute/month metric rather than individual vehicle uptime
9. Cost per vehicle per month must remain below $4.00 for consumer vehicles (commercial fleet is currently $2.10/vehicle/month at 100K scale — cost pressures increase at consumer scale)
10. 6-month onboarding timeline with progressive milestones: 100K consumer vehicles onboarded by month 2, 400K by month 4, 900K by month 6

### Hidden Requirements

1. **Hint: re-read Tomás Reyes's Slack message about consumer vs. commercial vehicles.** He says "Consumer vehicles are parked 95% of the time." A parked vehicle still maintains a heartbeat connection for OTA readiness and safety recall capability. At 1M vehicles with 95% parked, that is 950,000 persistent idle connections. The current connection management system uses a stateful WebSocket pool sized for 100K active connections. What happens to that pool at 1M connections, and what fraction are idle (heartbeat-only)?

2. **Hint: re-read the legal detail from Dr. Nakamura about the 6-month clause.** She says legal has known since Tuesday. The announcement was Thursday. Somebody on the business side knew the timeline was 6 months but the engineering team planned for 90 days. This is an organizational trust problem, not just a technical one. Your architecture plan must include a milestone accountability structure that prevents this kind of surprise again — specifically, what architectural telemetry would have made the 6-month clause visible to engineering earlier?

3. **Hint: re-read the onboarding velocity gap.** 200 vehicles/day current → 5,000 vehicles/day required. This is 25× increase in the onboarding pipeline. Onboarding a vehicle involves: SCMS certificate provisioning, vehicle registration in the fleet registry, OBU firmware version check, initial HD map tile download, and enrollment in the OTA system. Which of these steps is the bottleneck at 5,000/day, and is it an infrastructure bottleneck or a process bottleneck (e.g., SCMS certificate provisioning requires a coordination with the federal V2X SCMS provider)?

4. **Hint: look at the message rate math carefully.** At 1M vehicles × 100 messages/sec = 100M messages/sec at 100% fleet activity. But consumer vehicles only drive 5% of the time. Average active vehicles = 50K at any given time on average. Rush hour peaks to ~25% active (250K vehicles) for 90-minute windows. 250K × 100 msg/sec = 25M messages/sec rush hour peak. This is NOT 100M. The 100M figure assumes all vehicles active simultaneously — which is physically impossible. Show the correct peak calculation and explain where the 100M figure is wrong and what the real peak is.

### Constraints

- **Current fleet**: 100,000 commercial vehicles (flat load, 24/7)
- **New fleet**: 900,000 consumer vehicles (rush-hour spikes, 5% active at any time, 25% active during rush hour)
- **Peak message rate**: recalculate from consumer driving patterns (the "100M/sec" figure in the announcement math is wrong — show why)
- **Onboarding velocity required**: 5,000 vehicles/day
- **Onboarding velocity current**: 200 vehicles/day
- **Cost ceiling**: $4.00/vehicle/month for consumer tier; $2.10/vehicle/month for commercial tier
- **Onboarding timeline**: 100% consumer fleet in 6 months
- **Penalty clause**: $500K/week if onboarding SLA missed
- **Consumer SLA**: 99.5% V2X feature availability (lower than 99.9% commercial)
- **Idle connection model**: parked consumer vehicles maintain heartbeat connection (1 packet/30 seconds)
- **Rush hour definition**: 8–9AM, 5–6:30PM local time (multiple time zones as fleet is national)
- **72-hour window**: board presentation deadline
- **Infrastructure budget**: board wants 12-month cost projection with 3 scenarios

### Your Task

Produce a 6-month scaling plan covering:

1. Revised peak traffic calculation (correct the consumer vehicle math)
2. Kafka partition strategy redesign for bursty consumer load
3. Autoscaling configuration for rush-hour spike handling
4. Connection management strategy for 1M concurrent connections (95% idle)
5. Onboarding pipeline redesign: 200/day → 5,000/day
6. Cost model for three scenarios: base (900K consumer onboarded as planned), optimistic (1.2M consumer, faster growth), pessimistic (onboarding delays, penalty clause risk)
7. Milestone accountability structure for the board

### Deliverables

- [ ] Mermaid architecture diagram: show the scaling changes to each layer (ingestion, connection management, onboarding pipeline, autoscaling boundaries)
- [ ] Database schema changes: vehicle_registry table extension for consumer vs. commercial tier, connection_pool tracking, onboarding_queue table with priority/batching fields
- [ ] Scaling estimation (show math step by step):
  - **Correct peak calculation**: 900K consumer vehicles × 25% active during rush hour = 225K active + 100K commercial = 325K concurrent active vehicles × 100 msg/sec = **32.5M messages/sec peak** (not 100M — show why the difference matters for Kafka sizing)
  - **Kafka partition math**: at 32.5M messages/sec, with each partition handling 100K messages/sec → minimum partitions needed → add 50% headroom → final partition count
  - **Idle connection load**: 950K idle vehicles × 1 packet/30s = 31,666 packets/sec idle heartbeat. At 64 bytes/packet = 2MB/sec. Is this the bottleneck or is connection state the bottleneck? (Hint: each WebSocket connection requires ~10KB memory in the connection table.)
  - **Onboarding math**: 5,000 vehicles/day × 6 months = 900,000 vehicles. To complete in 6 months with headroom for delays, need to actually hit 5,500/day average. How many parallel SCMS certificate provisioning workers are needed if SCMS provisioning takes 500ms per vehicle?
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Static partition count vs. dynamic partition scaling (Kafka partitions cannot be reduced once created)
  - Separate Kafka clusters for commercial vs. consumer vs. unified cluster with topic partitioning
  - Rush-hour autoscaling with cold-start risk vs. pre-warmed capacity (cost vs. reliability)
- [ ] Consumer vs. commercial tier architecture: how does the system treat the two vehicle types differently at every layer (ingestion, storage, SLA monitoring, OTA priority)?
- [ ] 12-month cost model (three scenarios): itemized by compute, storage, network egress, CDN, SCMS fees, Kafka. Show cost per vehicle per month trend for commercial and consumer tiers separately.
- [ ] Capacity planning: what architectural decisions made today will become bottlenecks at 5M vehicles (3-year horizon)?

### Diagram Format

All architecture diagrams must be in Mermaid syntax.

```
Example skeleton (expand significantly):
graph LR
    subgraph "Consumer Tier"
        CV[Consumer Vehicles\n900K fleet\n25% rush-hour active]
    end
    subgraph "Commercial Tier"
        FV[Fleet Vehicles\n100K flat load\n90% active]
    end
    CV -->|32.5M msg/sec peak| KAFKA[Kafka Cluster\nPartition strategy TBD]
    FV -->|10M msg/sec| KAFKA
    KAFKA --> PROC[Stream Processors\nAutoscaling group]
    CV -.->|heartbeat 1/30s| CONN[Connection Manager\n1M concurrent connections]
```
