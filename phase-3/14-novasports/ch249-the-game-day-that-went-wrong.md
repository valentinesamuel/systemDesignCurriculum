---
**Title**: NovaSports — Chapter 249: The Game Day That Went Wrong
**Level**: Staff
**Difficulty**: 10
**Tags**: #chaos-engineering #incident-response #blast-radius #circuit-breakers #postmortem #spike-production-down
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 245, Ch. 248, Ch. 250
**Exercise Type**: Incident Response (SPIKE — Production DOWN)
---

### Story Context

> **NOTE:** This chapter begins as a routine chaos engineering exercise. It does not stay that way.

---

**#chaos-game-day — Thursday, 10:00 AM**

**You:** Good morning. Starting the game day in 5 minutes. Today's scenario: inject 500ms artificial latency into the `fan-feed-service`. Expected blast radius: fan feeds show stale scores and commentary for up to 30 seconds. No downstream cascade expected. The bet placement service has its own data path.

**Chioma Osei [Senior Eng]:** Ready. I've got the fault injection script standing by.

**Kenji Watanabe [SRE]:** Observability stack is live. PagerDuty test alerts are muted — we don't want false pages. Real incident alerts are still armed.

**Dev Chatterjee [CTO]:** I'll be watching from the Grafana dashboard. Go ahead when ready.

**You [10:05 AM]:** Injecting 500ms latency to `fan-feed-service`. Fault injection active.

**Chioma Osei [10:05 AM]:** Confirmed. Seeing the latency spike on fan-feed-service pods.

**You [10:06 AM]:** Fan feed latency: 520ms p50, 590ms p95. As expected. Fan engagement metrics dropping — users are seeing stale data. No alerts firing from downstream.

**Kenji Watanabe [10:07 AM]:** Wait. I'm seeing something odd.

**You [10:07 AM]:** What?

**Kenji Watanabe [10:07 AM]:** `bet-placement-service` error rate is climbing. It's at 0.8%. Was 0.05% before the injection.

**You [10:07 AM]:** That shouldn't be happening. Bet placement doesn't call fan-feed-service. Those are completely separate paths.

**Chioma Osei [10:08 AM]:** I'm tracing the calls. Oh no. Oh no no no.

**You [10:08 AM]:** What did you find?

**Chioma Osei [10:08 AM]:** The odds calculation service. It calls fan-feed-service to get current match state for in-play odds. It's an undocumented dependency. Nobody knew it existed.

**You [10:08 AM]:** Abort the game day. NOW.

---

**#chaos-game-day — 10:09 AM**

**You:** ABORTING. Removing fault injection.

**Kenji Watanabe [10:09 AM]:** Fault injection removed. Fan-feed latency returning to normal. But... bet-placement error rate is still climbing. It's at 4.2%.

**You [10:10 AM]:** That's not right. We removed the fault.

**Kenji Watanabe [10:10 AM]:** The odds service was retrying during the latency window. Its retry queue backed up. Now it's hammering fan-feed-service trying to catch up. The connection pool is saturated.

---

**#incidents — Thursday, 10:11 AM**

**[AUTOMATED ALERT]** `bet-placement-service` error rate CRITICAL: 8.7% and climbing. SLO breach in progress.

**Kenji Watanabe:** Paging on-call. This is no longer a game day.

**You:** I'm on the bridge.

---

**INCIDENT BRIDGE — P0 — Thursday, 10:11 AM**

**Incident Commander:** [You]
**Bridge participants:** Kenji Watanabe (SRE), Chioma Osei (Eng), Dev Chatterjee (CTO), Rosa Delgado (Staff Eng), Tomás Reyes (Sportsbook Integration Lead)

---

**10:12 AM — You:** Kenji, what's the current state?

**10:12 AM — Kenji Watanabe:** Bet-placement error rate: 11.4%. Odds service is in a retry storm against fan-feed-service. Fan-feed connection pool: 98% utilized. New connections being rejected. Each rejection causes the odds service to log an error and schedule an immediate retry with 0ms backoff.

**10:12 AM — You:** Zero millisecond backoff. Who wrote that retry logic?

**10:13 AM — Chioma Osei:** ...I think that was before either of us joined.

**10:13 AM — You:** Okay. We can assign blame later. Kenji: can we manually circuit-break the odds service's calls to fan-feed?

**10:13 AM — Kenji Watanabe:** Yes but it means in-play odds will be frozen. Live betting will stop accepting new bets.

**10:13 AM — You:** How bad is that?

**10:13 AM — Tomás Reyes:** Very bad. Thursday afternoon, there are 4 live matches right now. We have 800,000 active bet tickets in progress that are not yet settled.

**10:14 AM — You:** What's the sportsbook partner's SLA for bet acceptance?

**10:14 AM — Tomás Reyes:** If bet acceptance goes down for more than 90 seconds, they have the right to pause all unsettled bets and declare them voided. They're already seeing errors on their end.

**10:14 AM — Dev Chatterjee:** How long have we been in this state?

**10:14 AM — You:** Three minutes since injection. About two minutes of degraded bet placement.

**10:15 AM — Rosa Delgado:** The retry storm is the active problem. If we don't drain it, even when fan-feed recovers, the retry queue will immediately re-saturate it.

**10:15 AM — You:** Two actions simultaneously: (1) Kenji, rate-limit the odds service's outbound calls to fan-feed to 10% of current rate — force the retry storm to slow down. (2) Chioma, scale fan-feed horizontally — add 20 replicas, drain the connection pool.

**10:15 AM — Kenji Watanabe:** Applying rate limit now.

**10:15 AM — Chioma Osei:** Scaling fan-feed. Will take 90 seconds for pods to come up.

---

**10:17 AM — Kenji Watanabe:** Bet-placement error rate: 14%. Still climbing. The backlog is large.

**10:17 AM — You:** How large is the retry queue?

**10:17 AM — Kenji Watanabe:** 2.3 million queued retry requests.

**10:17 AM — Dev Chatterjee:** That's going to take minutes to drain even at full fan-feed capacity.

**10:18 AM — Tomás Reyes:** I'm getting a call from our sportsbook partner. They're invoking the 90-second clause. They're pausing all unsettled bets.

**10:18 AM — Dev Chatterjee:** How many bets?

**10:18 AM — Tomás Reyes:** 800,000 tickets. Average value $17. They're calculating damages now.

---

**10:31 AM** — Error rate begins falling. Fan-feed replicas online.

**10:41 AM** — Error rate below 0.1%. System stable.

**10:41 AM — Kenji Watanabe:** We're back. Incident duration: 30 minutes.

---

**10:42 AM — Tomás Reyes:** The sportsbook partner has completed their calculation. 800,000 voided bets. Average net margin $2.87 per bet. Their stated loss: $2.296M. They are requesting a call with NovaSports leadership.

---

**#chaos-game-day — Thursday, 11:00 AM**

**Dev Chatterjee:** The game day is over. We will not be resuming chaos exercises until we have done a full architectural review. The undiscovered dependency between odds calculation and fan-feed-service represents a blast radius calculation failure of the first order. We ran a game day without knowing the full dependency graph of the system we were testing.

**Rosa Delgado:** This is the most important thing we've learned in two years. It cost us $2.3M to learn it.

**You:** I'll write the postmortem. I'll also write the architecture proposal for what should have been in place. Both documents by Monday.

---

### Problem Statement

A chaos engineering game day — designed with a 30-second stale-data blast radius estimate — triggered a cascading failure that took down bet placement for 30 minutes, voided 800K bets, and cost the sportsbook partner $2.3M. The root causes: (1) an undiscovered dependency between the odds calculation service and the fan-feed service; (2) a retry loop with zero-backoff behavior; (3) no circuit breaker between odds calculation and bet placement. Your task is to analyze where blast radius calculation failed and design the architectural safeguards that should have existed.

### Explicit Requirements

1. Post-incident: identify all missing architectural safeguards
2. Design explicit circuit breakers between fan-feed, odds calculation, and bet placement
3. Define a blast radius calculation process that accounts for undiscovered dependencies
4. Design retry behavior standards with exponential backoff and jitter
5. Define abort criteria for future chaos game days (what observable signal triggers abort)
6. Design a dependency graph discovery mechanism that prevents unknown coupling

### Hidden Requirements

1. **Hint: re-read Chioma's discovery of the undiscovered dependency.** "It's an undocumented dependency. Nobody knew it existed." This is not just a process failure — it's an architectural failure. Service-to-service calls should be discoverable via service mesh telemetry. The game day was run without generating a live dependency graph from production traffic first.

2. **Hint: re-read the retry queue size: 2.3 million queued requests with 0ms backoff.** The retry storm amplified the incident from a 30-second fan-feed slowdown to a 30-minute bet placement outage. The amplification factor was approximately 60x. Any circuit breaker design must account for retry amplification in the blast radius model.

3. **Hint: re-read Tomás Reyes's 90-second clause.** The sportsbook partner contract has a 90-second SLA clause for bet acceptance. This is a compliance and contractual obligation, not just a quality target. Any future architecture must treat bet-placement uptime as a contractual SLA with a defined financial penalty per violation.

4. **Hint: re-read Dev Chatterjee's final message.** "We ran a game day without knowing the full dependency graph of the system we were testing." The fix is not "be more careful" — it's a tooling and process requirement. Dependency graphs must be generated from live traffic tracing before any chaos experiment is approved.

### Constraints

- **Affected bets**: 800K voided bets, $2.3M partner loss
- **Cascade path**: fan-feed latency → odds calculation retry storm → connection pool saturation → fan-feed full failure → bet placement error rate
- **Retry behavior**: 0ms backoff, 2.3M queued retries at peak
- **Incident duration**: 30 minutes from fault injection to full recovery
- **Circuit breaker requirement**: bet placement must be isolated from fan-feed failures within 5 seconds of detection
- **Abort criteria**: game day must auto-abort if any non-target service error rate exceeds 1%
- **Dependency graph**: must be generated from OpenTelemetry trace data before every game day
- **Partner SLA**: bet acceptance must not degrade below 99.9% (currently violated)

### Your Task

Write the architectural analysis of what failed and design the corrective architecture. This is a two-part exercise: (1) failure mode analysis — trace the exact cascade and identify every missing safeguard; (2) redesign — propose the specific architectural changes that prevent this class of incident.

### Deliverables

- [ ] Failure mode cascade diagram: exact path from fault injection to $2.3M loss (Mermaid)
- [ ] Missing safeguard analysis: for each failure mode, what architectural pattern would have prevented it
- [ ] Circuit breaker design: between fan-feed → odds calculation → bet placement (with timeout values, half-open thresholds, and fallback behavior)
- [ ] Retry behavior standard: exponential backoff with jitter specification (algorithm + parameters)
- [ ] Blast radius calculation process: step-by-step methodology using live dependency graph from trace data
- [ ] Game day abort criteria specification: observable signals, thresholds, and automatic abort mechanism
- [ ] Dependency graph discovery design: how OpenTelemetry trace data is used to generate a dependency graph before game day approval
- [ ] Database schema for game day experiment registry (with dependency graph snapshot, abort criteria, blast radius estimate)
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs)
- [ ] Cost modeling: cost of missing circuit breakers ($2.3M incident) vs cost of implementing them ($X engineering time + infrastructure)

### Diagram Format

Mermaid syntax. Two diagrams required:
1. **Incident cascade diagram**: what actually happened (fault injection → retry storm → cascade → partner loss)
2. **Target architecture diagram**: what should exist (circuit breakers, rate limiters, dependency isolation)
