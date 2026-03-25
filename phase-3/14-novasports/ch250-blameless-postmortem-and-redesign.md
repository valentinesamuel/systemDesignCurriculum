---
**Title**: NovaSports — Chapter 250: The Postmortem (And the Architecture We Should Have Built)
**Level**: Staff
**Difficulty**: 8
**Tags**: #postmortem #blameless #chaos-engineering #bulkhead #blast-radius #incident-management
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 249, Ch. 251
**Exercise Type**: System Evolution (Mandatory Story Beat — Part 2)
---

### Story Context

> The following is the NovaSports internal postmortem document for the October 12 Game Day Incident. Distribution: Engineering leadership, sportsbook partner (BetStream Technologies), and legal.
> Ground rules established at the start of the postmortem meeting: no blame, no finger-pointing, focus on systems.

---

## POSTMORTEM: NovaSports Game Day Cascade Incident
**Date of Incident:** Thursday, October 12
**Postmortem Meeting:** Saturday, October 14 — 2:00 PM
**Facilitator:** You (Incident Commander)
**Attendees:** Rosa Delgado, Chioma Osei, Kenji Watanabe, Tomás Reyes, Dev Chatterjee (NovaSports); Marcus Chen, Lisa Park (BetStream Technologies, via video)
**Document Classification:** Sensitive — Partner Distribution Authorized

---

### Meeting Opening

**Dev Chatterjee (2:04 PM):** Before we start, I want to say one thing. This room — everyone in this room, including our friends from BetStream — is here to understand what happened and make sure it never happens again. We are not here to find who to blame. Systems fail. Systems fail in predictable ways. Our job is to find the predictable ways and eliminate them.

**Marcus Chen (BetStream, via video):** I appreciate that framing. I'll be direct: this hurt us. $2.3M is real money. But I've been in enough incidents to know that finger-pointing produces nothing. We want the architectural analysis and we want the remediation plan. That's why we're here.

---

### Incident Timeline

| Time | Event |
|------|-------|
| 10:00 AM | Game day announced. Planned scenario: 500ms latency injection to `fan-feed-service`. |
| 10:05 AM | Fault injection activated. Fan-feed p95 latency: 590ms. Within expected parameters. |
| 10:07 AM | SRE (Kenji) observes `bet-placement-service` error rate rising to 0.8%. Not expected. |
| 10:08 AM | Undiscovered dependency identified: `odds-calculation-service` calls `fan-feed-service` for live match state. This dependency was not in any architectural documentation. |
| 10:09 AM | Fault injection removed. Fan-feed latency returns to normal. |
| 10:09 AM | `odds-calculation-service` retry storm begins: 2.3M queued requests with 0ms backoff. |
| 10:10 AM | Fan-feed connection pool 98% utilized from retry storm. New connections rejected. |
| 10:11 AM | `bet-placement-service` error rate: 8.7%. Automated SLO breach alert fires. Incident declared P0. |
| 10:14 AM | BetStream invokes 90-second contractual clause. 800K unsettled bets voided. |
| 10:15 AM | Mitigations applied: rate limit on odds-service outbound calls + fan-feed horizontal scale. |
| 10:31 AM | Error rate begins falling. Fan-feed replicas online. |
| 10:41 AM | Error rate below 0.1%. System stable. Incident duration: 30 minutes. |

---

### Contributing Factors

**Factor 1: No Pre-Game-Day Dependency Graph Validation**

The game day was approved based on a manually authored dependency diagram. The diagram was last updated 8 months ago. The `odds-calculation-service` dependency on `fan-feed-service` was added 6 months ago during a feature sprint, not via a formal architectural change process. It was never documented.

Root mechanism: we had no automated process to generate a current dependency graph from live trace data before approving a chaos experiment.

**Factor 2: Retry Logic Without Backoff**

The `odds-calculation-service` HTTP client for fan-feed calls had been configured with `retries: 3, retryDelay: 0`. This was a misconfiguration from an original copy-paste during initial service setup. It was never reviewed in a service configuration audit because there was no service configuration audit process.

Root mechanism: no standards enforcement for retry behavior; no automated check that would flag 0ms retry delay as a misconfiguration.

**Factor 3: No Circuit Breaker Between Odds Calculation and Fan Feed**

The `odds-calculation-service` calls `fan-feed-service` synchronously in the hot path of odds calculation. There was no circuit breaker, no fallback behavior (e.g., use last-known odds if fan-feed is unavailable), and no timeout shorter than the HTTP default (30 seconds).

Root mechanism: circuit breakers were implemented at the API gateway layer for external traffic, but not at the internal service-to-service layer. Internal calls were treated as reliable.

**Factor 4: Blast Radius Estimate Was Wrong**

The game day brief stated: "Expected blast radius: fan feeds show stale data for up to 30 seconds. No downstream cascade expected." This was wrong. The blast radius calculation did not account for the undiscovered dependency (Factor 1), did not model the retry amplification effect (Factor 2), and assumed no internal calls used fan-feed-service (Factor 3).

Root mechanism: blast radius calculation was a human exercise, not derived from production evidence.

**Factor 5: No Automatic Abort on Non-Target Service Degradation**

The game day was conducted with real-time monitoring, but there was no automated abort criterion. When `bet-placement-service` error rate began rising at 10:07 AM, the response was human observation and discussion. By 10:09 AM (when fault injection was removed), the cascade was already in progress and could not be stopped.

Root mechanism: abort criteria were not defined in advance, and there was no automated mechanism to terminate the fault injection if non-target services degraded.

---

### Action Items

| # | Action | Owner | Due Date |
|---|--------|-------|----------|
| 1 | Implement OpenTelemetry-derived dependency graph generation; require sign-off before any chaos experiment | Kenji Watanabe | 3 weeks |
| 2 | Enforce retry configuration standard: exponential backoff with jitter, minimum 100ms initial delay, maximum 3 retries. Add automated linting to CI/CD pipeline. | Chioma Osei | 2 weeks |
| 3 | Implement circuit breakers for all internal service-to-service calls that touch the bet placement critical path. | You | 4 weeks |
| 4 | Define blast radius calculation template: must include dependency graph section, retry amplification modeling, and contractual SLA impact analysis. | You | 2 weeks |
| 5 | Implement automated game day abort: if any non-target service's error rate exceeds 2x baseline during fault injection, auto-abort and alert. | Kenji Watanabe | 2 weeks |
| 6 | Audit all internal HTTP clients for retry configuration; report findings to engineering leadership. | Chioma Osei | 1 week |

---

**Marcus Chen (BetStream, via video, 3:47 PM):** These action items are thorough. I want to say one more thing before we close. The speed of your incident response — you had mitigation actions applied within 10 minutes of the fault injection — was genuinely good. The problem wasn't the response. It was the architecture that was missing before the incident started. We're satisfied with this postmortem. Send us the remediation timeline.

**Rosa Delgado (3:48 PM):** One more thing from me. We should have run a game day on the game day process itself before we ran a game day on a production system. We learned the hard way. We won't forget it.

**Dev Chatterjee (3:50 PM):** Meeting closed. Thank you, everyone. Monday: architecture redesign proposal from [You]. Friday: action item status update from all owners.

---

### Slack thread after the meeting

**Rosa Delgado [#engineering-leadership, Saturday, 4:02 PM]:**
> "That was one of the best postmortems I've been in. The BetStream team came in ready to be angry and left with a signed remediation plan. The blameless culture we've been trying to build actually showed up today."

**Dev Chatterjee [4:04 PM]:**
> "Credit to the facilitator."

**You [4:05 PM]:**
> "I learned the format from someone who once told me: 'The postmortem is not the punishment. The redesign is the apology.'"

---

### Problem Statement

Following the game day cascade incident, you must redesign the NovaSports architecture to prevent this class of failure. The redesign must implement: (1) bulkhead isolation between fan engagement systems and bet placement critical path; (2) circuit breakers at the internal service-to-service layer; (3) explicit blast radius documentation and automated enforcement for future chaos experiments; and (4) retry behavior standards with automated compliance checking.

### Explicit Requirements

1. Bulkhead pattern: fan-feed-service failure cannot cascade to bet-placement-service
2. Circuit breakers on all service-to-service calls in the bet placement critical path
3. Fallback behavior: odds calculation must degrade gracefully (use cached odds) if fan-feed is unavailable
4. Retry standard: exponential backoff, jitter, max 3 retries, minimum 100ms initial delay
5. Automated CI/CD check for retry misconfiguration
6. Blast radius calculation template: dependency graph + retry amplification model + contractual impact
7. Game day abort automation: auto-abort if non-target service degrades beyond threshold
8. Dependency graph generation from OpenTelemetry trace data before every game day

### Hidden Requirements

1. **Hint: re-read Factor 3.** The circuit breaker must include a fallback behavior, not just a break. "Use last-known odds if fan-feed is unavailable" means the circuit breaker needs a local cache of the last successful response. This is a stateful circuit breaker — harder to implement but necessary for graceful degradation in a betting system.

2. **Hint: re-read Marcus Chen's closing statement.** "We're satisfied with this postmortem." BetStream will continue the partnership — but the contractual SLA has now been formally documented. Future incidents that trigger the 90-second clause will automatically incur financial penalties. The architecture must treat the 90-second SLA as a hard design constraint, not a best-effort target.

### Constraints

- **Bulkhead requirement**: bet-placement error rate must stay below 0.1% even if fan-feed-service goes fully down
- **Circuit breaker**: must open within 5 seconds of fan-feed degradation detection
- **Fallback TTL**: cached odds valid for up to 60 seconds before bet acceptance must pause
- **Retry standard**: exponential backoff (base 100ms, multiplier 2x, jitter ±50%, max 3 retries)
- **CI/CD check**: must flag 0ms retry delay as a build failure
- **Dependency graph**: generated from 24 hours of trace data, refreshed before each game day
- **Abort threshold**: auto-abort if non-target service error rate exceeds 2× baseline for > 30 seconds
- **Partner SLA**: 99.9% bet acceptance uptime (up from informal target; now contractual)

### Your Task

Produce the complete architectural redesign document. This should serve as the "architecture we should have built" — both the specific changes to NovaSports systems and the process changes to prevent blast radius miscalculation in future chaos experiments.

### Deliverables

- [ ] Mermaid architecture diagram: redesigned NovaSports system with bulkheads, circuit breakers, and fallback paths explicitly labeled
- [ ] Circuit breaker specification: per-service configuration (thresholds, timeouts, fallback behavior, half-open probe strategy)
- [ ] Bulkhead isolation design: how fan-feed failure is contained before reaching the bet placement path
- [ ] Graceful degradation design: what each service does when its dependency is unavailable (tiered fallback)
- [ ] Blast radius calculation template: the formal document template that must be completed before any future game day
- [ ] Retry behavior standard: specification document with algorithm, parameters, and CI/CD enforcement rule
- [ ] Game day governance process: dependency graph generation → blast radius calculation → abort criteria definition → approval → execution → auto-abort monitoring
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs)
- [ ] Cost modeling: estimated engineering cost of remediation vs cost of the incident ($2.3M)
- [ ] 6-month capacity plan for the redesigned architecture

### Diagram Format

Mermaid syntax. Two diagrams:
1. **Before (incident architecture)**: show the missing safeguards as gaps
2. **After (redesigned architecture)**: bulkheads, circuit breakers, fallback caches, all explicitly labeled
