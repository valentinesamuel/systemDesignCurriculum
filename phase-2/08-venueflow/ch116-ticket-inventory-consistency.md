---
**Title**: VenueFlow — Chapter 116: Ticket Inventory Consistency — The Taylor Swift Problem
**Level**: Staff
**Difficulty**: 10
**Tags**: #distributed #inventory #event-sourcing #crdt #consistency #saga #cap-theorem
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 35 (PulseCommerce inventory reservation), Ch. 50 (OmniLogix inventory event sourcing), Ch. 53 (OmniLogix saga pattern), Ch. 113 (VenueFlow real-time)
**Exercise Type**: System Design
---

### Story Context

**#incidents — Slack**
**Saturday 10:02 AM — Two weeks before the Taylor Swift onsale**

**zara.ahmed [CTO]:** Load test results are in. Conference room in 15 minutes. Everyone on inventory team.

---

**Conference Room B2 — 10:18 AM**

Zara doesn't sit down. She stands at the whiteboard with a red marker and writes two numbers:

**2,000,000** (simulated concurrent users)
**80,000** (available tickets)

"The load test ran for six minutes," she says. "Tell them what happened."

**petra.holst [Staff Eng, Inventory]:** "At peak concurrency — around 1.4 million simulated users — the Postgres row lock acquisition time for seat reservation went above 800ms. At 1.8 million users, row locks started queuing behind each other. The lock wait queue grew faster than it drained. We had two failure modes: some requests timed out and returned a false 'seat unavailable' — a valid fan rejected. Others — this is the one that matters — two requests acquired what they both thought was an exclusive lock on the same row within the same millisecond window due to a race in our lock-upgrade path. We got 14 double-sells in the load test."

Silence.

**[you]:** "Fourteen double-sells in a load test. What's the extrapolation for a real onsale?"

**petra.holst:** "We estimated 200 to 600 actual double-sells. Depending on peak concurrency shape."

**zara.ahmed:** "The contractual penalty for a double-sell is a full refund plus a 40% inconvenience fee. At face value of $180 per ticket — 600 double-sells is about $150,000 in penalties. But that's not the problem. The problem is that a double-sell on a Taylor Swift onsale will be on Twitter within thirty seconds. Every news outlet in the US will cover it. We will lose the account."

She uncaps the red marker and writes a third line under the two numbers:

**Zero double-sells. Non-negotiable.**

**Slack DM — petra.holst → you — 10:44 AM**
**petra.holst:** The row-lock approach isn't salvageable at this scale. I've known it for six months. I didn't have political cover to rebuild it until now.
**[you]:** What did you want to build?
**petra.holst:** I wanted to model inventory as events, not state. Hold events, reserve events, release events. Append-only. The current row count is derived from the log. No lock contention on writes because you're never updating — only appending.
**[you]:** What stopped you?
**petra.holst:** The previous VP of Engineering said "if it ain't broke don't fix it." He left four months ago.
**[you]:** It was always broke. He just never load-tested it.
**petra.holst:** ...yes.

**10:51 AM**
**[you]:** What's the hold/reserve/confirm state machine look like right now?
**petra.holst:** Hold: 8 minutes. Reserve: payment window, up to 10 minutes. Confirm: on payment success. Release: on hold expiry or payment failure.
**[you]:** And if the payment service is slow? What happens to holds that are waiting on payment confirmation?
**petra.holst:** They stay held. We had an incident in March where the payment service degraded for 22 minutes and 60,000 seats were stuck in held state. Nobody could buy them. They weren't sold. They were just... frozen.
**[you]:** "Too late to compensate" territory. We've seen this pattern before.
**petra.holst:** What do you mean?
**[you]:** There's a version of this problem at scale where the saga compensation — releasing a hold — is itself a write operation that creates lock contention. You can't release your way out of a deadlock if releasing requires the same lock.

---

**Email**
**To:** zara.ahmed@venueflow.io
**From:** [you]
**Subject:** Inventory Rebuild — Approach Recommendation
**Date:** Saturday, 2:15 PM

Zara —

Here's my recommendation: rebuild inventory as an append-only event log. Every seat action — hold, reserve, confirm, release — is an immutable event appended to a per-seat (or per-section) event stream. The current state of any seat is the reduce of its event history. There are no row-level locks because there are no rows to lock — only appends.

For the 2M-user concurrency problem: we add a virtual waiting room ahead of inventory. Admission control limits how many users reach the reservation service at any time. We let in N users per second based on our confirmed throughput — not 2M simultaneously.

The compare-and-swap at append time (optimistic concurrency on the event log) replaces the pessimistic row lock. Conflicts are rare and cheap to retry. Double-sells become logically impossible: the event log is the source of truth and it is ordered.

Two weeks isn't enough time to rebuild the full system. But it's enough time to put the virtual waiting room in front of the existing system and remove the worst of the concurrency spike. The full event-log rebuild is a 6-week project post-Taylor Swift.

— [you]

---

This chapter sits at the intersection of every distributed systems concept you have touched in this curriculum: event sourcing from OmniLogix, saga compensation from the supply chain arc, CAP theorem tradeoffs, idempotency under retry, and the specific failure mode of "too late to compensate." The Taylor Swift onsale is two weeks away. Zero double-sells is not a goal. It is a constraint.

### Problem Statement

VenueFlow's ticket inventory system uses Postgres row-level locking for seat reservation. Under 2M concurrent users with 80k available tickets, lock acquisition latency exceeds 800ms, producing both false rejections (valid fans turned away) and double-sells (two fans confirmed for the same seat) due to a race condition in the lock-upgrade path. The system must be redesigned to deliver zero double-sells, sub-2-second end-to-end reservation confirmation, and graceful degradation under extreme concurrency without requiring fans to compete directly against each other at the database lock layer.

### Explicit Requirements

1. Zero double-sells: the seat reservation system must guarantee that no seat is confirmed to more than one fan
2. Hold/reserve/confirm state machine with configurable timeouts (hold: 8 min, payment window: 10 min)
3. Automatic hold release: expired holds must release without manual intervention
4. Virtual waiting room: admission control to limit concurrent users reaching the reservation layer
5. Event-sourced inventory: seat state derived from append-only event log, not mutable rows
6. Idempotent reservation requests: retry-safe — a fan retrying a hold request must not create a duplicate hold
7. Inventory anti-entropy: periodic reconciliation to detect and resolve any state drift between event log and derived seat state
8. End-to-end reservation confirmation: <2 seconds P95 from "fan clicks seat" to "hold confirmed"

### Hidden Requirements

- **Hint: re-read Petra's description of the March payment service incident.** 60,000 seats stuck in held state for 22 minutes because payment service was slow. How does the event-sourced model change the hold-release path when an upstream dependency is degraded? What happens to holds that are waiting for a payment confirmation that never comes?

- **Hint: re-read the discussion about "too late to compensate."** In the saga pattern, compensation assumes the compensating action is always possible. When is a seat hold compensation impossible — and what is the guarantee the system must provide in that case?

- **Hint: re-read Zara's zero-double-sell mandate alongside the virtual waiting room proposal.** The waiting room limits concurrency into the reservation service. But what is the correct admission rate — how do you calculate N (users per second let through) from your known throughput, and what happens when your throughput estimate is wrong?

- **Hint: re-read the 14 double-sells in the load test.** They came from a "race in our lock-upgrade path." This is a Postgres-specific failure mode. Does event sourcing with compare-and-swap fully eliminate this class of race, or does it shift the race to the append layer? What is the residual failure mode?

### Constraints

- **Peak concurrency**: 2,000,000 simultaneous users at onsale open
- **Seat inventory**: 80,000 seats per event; stadium configuration
- **End-to-end hold confirmation**: <2 seconds P95
- **Hold timeout**: 8 minutes; payment confirmation timeout: 10 minutes
- **Double-sell guarantee**: zero tolerance (contractual + reputational)
- **False rejection rate**: <1% of valid buyers rejected due to system errors
- **Infrastructure**: Postgres (existing) + Redis (existing) + Node.js services
- **Timeline**: 2 weeks to Taylor Swift onsale (virtual waiting room must ship); full event-log rebuild in 6 weeks post-event
- **Team**: 3 inventory engineers + Petra (Staff) + you
- **Cost constraint**: no new database vendors; rebuild on existing Postgres + Redis stack

### Your Task

Design the ticket inventory consistency system. Define the event-sourced seat state model, the hold/reserve/confirm/release state machine, the compare-and-swap append mechanism, the virtual waiting room admission control, and the inventory anti-entropy reconciliation process. Separate the two-week sprint deliverable (waiting room + race fix) from the six-week full rebuild (event-sourced inventory).

### Deliverables

- [ ] Mermaid architecture diagram: fan → virtual waiting room → reservation service → event log → seat state projection; hold expiry worker; anti-entropy reconciler
- [ ] Database schema for:
  - Seat event log (event type, seat ID, fan ID, timestamp, version/sequence, idempotency key)
  - Derived seat state projection table (current status, held-by, held-until)
  - Waiting room queue (fan token, queue position, admitted-at)
- [ ] Scaling estimation (show math):
  - Append throughput required: 80k seats × average hold attempts per seat at 2M users
  - Virtual waiting room admission rate: calculate N based on reservation service confirmed throughput
  - Hold expiry worker: how many expirations per second at peak, and what is the release latency?
  - Anti-entropy scan cost: full reconciliation frequency and Postgres scan performance
- [ ] Tradeoff analysis (minimum 3):
  - Event-sourced inventory vs. optimistic locking on mutable rows (complexity vs. correctness)
  - Compare-and-swap on event log sequence vs. distributed lock (Redis SETNX) per seat
  - Waiting room (queue-based fairness) vs. random admission lottery (simplicity vs. fan experience)
- [ ] Two-phase delivery plan:
  - Phase 1 (2 weeks): virtual waiting room + race condition fix on existing system
  - Phase 2 (6 weeks): full event-sourced rebuild — migration strategy for in-flight holds
- [ ] State machine diagram (Mermaid): seat states (available → held → reserved → sold; held → available on expiry; reserved → available on payment failure)

---
