---
**Title**: TeleNova — Chapter 124: 5G QoS — Fair Use Without Unfair Enforcement
**Level**: Staff
**Difficulty**: 8
**Tags**: #rate-limiting #5g #qos #fair-use #network-policy #per-subscriber #audit-trail #sixth-appearance
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 29 (CloudStack rate limiting), Ch. 40 (CivicOS behavioral), Ch. 62 (SkyRoute multi-class), Ch. 87 (GigGrid fairness), Ch. 118 (VenueFlow DDoS)
**Exercise Type**: System Design
---

### Story Context

**This is the 6th rate limiting appearance in the curriculum — each one has escalated: token bucket (CloudStack) → behavioral ASN (CivicOS) → contractual partner overrides (SkyRoute) → weighted fair queuing (GigGrid) → adaptive DDoS mitigation (VenueFlow) → telecom QoS enforcement (TeleNova)**

---

**Email**
**From**: Yuki Tanaka (Head of Security)
**To**: [you], Fatima Al-Rashid
**Subject**: Three Fair-Use Policy Problems — Action Required
**Date**: Wednesday

Colleagues,

I'm flagging three separate issues with our current fair-use policy enforcement system, all of which require architectural resolution.

**Issue 1: The Reconnect Bypass**
A power user on plan NL-Personal-100 (100GB/month at full speed, then throttled) discovered that disconnecting and reconnecting their mobile device resets the throttling counter. They've shared this on a Dutch consumer forum. We have 340 subscribers replicating this behavior. Estimated bandwidth impact: 12TB/month above entitled usage.

**Issue 2: The Business SIM Reseller**
A small ISP named DataReach B.V. is running 4,200 IoT devices behind a single enterprise SIM. Their contract: TeleNova Business Unlimited plan. Monthly usage: 847GB. The contract doesn't prohibit device sharing, but the network impact is equivalent to running a small ISP on one contract. We're absorbing the cost of serving 4,200 devices while billing for one.

**Issue 3: Regulatory Audit Request**
The French telecom regulator (ARCEP) has formally requested that TeleNova demonstrate fair-use throttling is applied "consistently and non-discriminatorily" across all subscriber classes. This requires an audit trail: for any subscriber, we must be able to show when throttling was applied, what threshold triggered it, and whether the same threshold was applied to comparable subscribers.

All three issues require changes to the rate limiting architecture.

— Yuki

---

**Meeting — Thursday 09:00**
**Attendees**: [you], Yuki Tanaka, Alejandro Reyes, Kenji Watanabe (Head of Network Engineering)

**Alejandro Reyes**: Walk us through the current architecture.

**Kenji Watanabe**: Rate limiting runs at the PDN gateway. Per-subscriber counters tracked in Redis. When a subscriber hits their monthly limit, we flip a policy flag that routes their traffic through a throttle tier (1Mbps cap instead of full 5G speed). The counter is keyed to `session_id`.

**[you]**: And a new session means a new counter?

**Kenji Watanabe**: *(pause)* Yes. When a device disconnects and reconnects, it gets a new session. New session, new counter key. Old counter is abandoned.

**[you]**: That's Issue 1. The key is wrong. The counter should be keyed to `subscriber_id`, not `session_id`.

**Kenji Watanabe**: That's a one-line fix.

**Yuki Tanaka**: Yes, but now that fix needs to be auditable. If we change the throttling behavior, ARCEP wants to see that the change was applied consistently and didn't accidentally throttle some subscribers incorrectly.

**Alejandro Reyes**: Which brings us to Issue 3. The audit trail.

**[you]**: The audit trail and the counter key fix are the same architecture change. If we move from session-keyed to subscriber-keyed counters with event logging on every threshold transition — we solve Issue 1 and Issue 3 simultaneously.

**Kenji Watanabe**: What about Issue 2? The IoT reseller?

**[you]**: Issue 2 is a different problem. The current policy says "unlimited" per subscriber. We need a policy layer that detects device sharing patterns — not to throttle them punitively, but to identify commercial reselling that should be on a different contract tier.

**Alejandro Reyes**: We can't throttle them without contract basis. But we can flag them for account review?

**[you]**: Correct. The system design is: detect anomalous connection patterns (4,200 distinct device MACs behind one subscriber), alert the account team, they renegotiate the contract. The technical system identifies; the commercial team enforces.

---

### Problem Statement

TeleNova's fair-use enforcement system has three failures: a session-keyed counter allows throttle bypass via reconnect; a single enterprise SIM hosting 4,200 devices consumes infrastructure intended for one user; and the French regulator requires an audit trail proving consistent enforcement across subscribers. Redesign the QoS enforcement architecture to fix all three.

---

### Explicit Requirements

1. Throttle counters must be keyed to subscriber identity, not session (fixes reconnect bypass)
2. All throttle threshold transitions must be event-logged with timestamp, subscriber ID, threshold triggered, and prior usage
3. Device sharing detection: identify subscribers with > 50 distinct device MACs per 30-day window
4. ARCEP audit API: given a subscriber ID and date range, return complete throttle event history
5. Policy engine: throttle thresholds and plan definitions must be configurable without code deployment
6. Enforcement must work at 120M subscriber scale with ≤ 10ms latency impact on normal requests

---

### Hidden Requirements

- **Hint**: Yuki mentioned ARCEP wants proof of "consistent" enforcement. "Consistent" means: two subscribers on the same plan who hit the same threshold in the same month must receive the same throttle treatment. But what if one subscriber was throttled due to a bug (and wasn't actually at the threshold) — and that's discoverable in the audit trail? Your audit design must support finding and correcting erroneous throttle applications retroactively.

- **Hint**: The DataReach B.V. problem is 4,200 devices behind one SIM. But in legitimate IoT deployments (like Deutsche Bahn from ch123), thousands of devices behind enterprise accounts is normal and expected. What is the distinguishing characteristic between a legitimate enterprise IoT deployment and an unauthorized reseller? Your detection logic must not flag Deutsche Bahn.

---

### Constraints

- 120M subscribers, sliding window counters in Redis Cluster
- Counter update latency: ≤ 5ms (called on every network event)
- Audit log write throughput: ~50k throttle events/day → append-only event store
- ARCEP audit response SLA: 30 calendar days to provide audit data for a specific subscriber
- Policy changes: must propagate to all enforcement points within 60 seconds
- DataReach B.V. detection threshold: > 50 distinct device MACs in 30 days (not applicable to enterprise IoT accounts with registered device pools)

---

### Your Task

Design TeleNova's QoS enforcement architecture that fixes the three identified issues and satisfies regulatory audit requirements.

---

### Deliverables

- [ ] Counter key redesign: subscriber-keyed counter schema with session continuity logic
- [ ] Throttle event log schema: fields, retention policy, query patterns for ARCEP audit
- [ ] ARCEP audit API design: request/response format, authentication, rate limiting for regulators
- [ ] Device sharing detection algorithm: how to distinguish DataReach B.V. (unauthorized) from Deutsche Bahn (legitimate enterprise)
- [ ] Policy engine design: configurable thresholds, plan definitions, real-time propagation to 120M subscriber enforcement points
- [ ] Redis Cluster design for subscriber-keyed counters at 120M subscribers: key distribution, memory model, expiry strategy
- [ ] Retroactive correction process: if a subscriber was erroneously throttled, how to identify and remediate
- [ ] Tradeoff analysis (minimum 3):
  - Sliding window log vs fixed window counter for monthly usage tracking — accuracy vs storage
  - Centralized policy engine vs embedded policy rules at enforcement points — consistency vs latency
  - Retroactive correction capability vs audit trail immutability — regulatory requirement tension
