---
**Title**: VenueFlow — Chapter 114: Concert Cancellation Notification Cascade (4th Appearance)
**Level**: Staff
**Difficulty**: 9
**Tags**: #notifications #spike-production-down #messaging #rate-limiting #dlq #multi-channel
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 20 (Beacon Media push notifications), Ch. 43 (NeuroLearn pre-staged fan-out), Ch. 59 (SkyRoute priority tiers), Ch. 113 (VenueFlow real-time)
**Exercise Type**: Incident Response
---

### Story Context

**#incidents — Slack**
**Thursday 11:47 PM**

**pagerduty-bot:** 🔴 ALERT: notification-service queue depth > 500k — threshold: 50k
**pagerduty-bot:** 🔴 ALERT: SendGrid 429 rate limit — notification-service-prod

**11:48 PM**
**[you]:** I'm on. What triggered?
**dev.on-call [Ravi Nair]:** Artist cancellation came through at 11:44. We have a confirmed cancel for tomorrow's stadium show. Two million ticket holders.
**[you]:** Two million. What's our legal SLA for notification?
**ravi.nair:** Per EU Consumer Rights Directive and our ticketing partner contracts: 2 hours from confirmed cancellation for all ticket holders to receive notification. We have until 1:47 AM.
**[you]:** It is 11:48. We have 119 minutes.
**ravi.nair:** Yes.
**[you]:** Current queue depth?
**ravi.nair:** It's climbing. 1.2M now. We triggered the fan-out before we knew SendGrid was going to 429 us.

**11:51 PM**
**zara.ahmed [CTO]:** I'm on the bridge. Talk to me.
**[you]:** We have 2M notifications to deliver in 118 minutes. SendGrid is rate-limiting us. We need parallel multi-channel delivery right now — email is not going to make it alone.
**zara.ahmed:** Twilio?
**ravi.nair:** Already checked. Twilio SMS is also rate-limiting on this burst. Their burst allowance is 100/sec for our tier. At that rate: 333 minutes. We don't have 333 minutes.
**zara.ahmed:** What do we have?
**[you]:** Tell me we have FCM and APNs configured.
**ravi.nair:** Yes. Push is configured. Roughly 60% of ticket holders have the app installed.
**[you]:** Then we split the load. Push for app users right now. Email and SMS for the rest, rotated across providers. Do we have a Mailgun account?
**ravi.nair:** We have credentials. We've never sent production traffic through them.
**zara.ahmed:** Tonight's a great night to start.

**11:54 PM**
**ravi.nair:** New problem. A fan with four tickets is in the queue four times. One per ticket. We're going to send four identical "your show is cancelled" messages to the same email address.
**[you]:** Deduplication layer. Does the notification service have one?
**ravi.nair:** ...no.
**[you]:** Of course not.

**11:56 PM**
**pagerduty-bot:** 🔴 ALERT: notification-service queue depth > 4M — current: 4,200,000

**ravi.nair:** Queue depth is accelerating. We have fans with 6, 8, sometimes 20 tickets in group bookings. Each ticket triggers a separate notification event.
**[you]:** Fan-out is running at the ticket level, not the fan level. We need to deduplicate upstream before we hit any provider. How fast can you write a Redis SETNX dedup filter?
**ravi.nair:** I can do that in 20 minutes.
**[you]:** You have 10.

**12:03 AM**
**ravi.nair:** Dedup filter is live. Queue depth is stabilizing at 2.1M — which is roughly our unique fan count.

**12:07 AM**
**[you]:** Route status: FCM running at 8k/sec. Mailgun running at 2k/sec. SendGrid recovered from 429, running at 1k/sec. Twilio at 100/sec. Total: ~11k/sec.
**[you]:** At 11k/sec: 2.1M notifications in approximately 191 seconds. That's just over 3 minutes.
**zara.ahmed:** That's the first good number I've heard tonight.

**12:09 AM**
**ravi.nair:** DLQ is filling up. About 80k notifications bounced — bad email addresses, invalid device tokens.
**[you]:** Those bounce to Twilio SMS as fallback. What's their status?
**ravi.nair:** Still at 100/sec. 80k messages at 100/sec = 800 seconds. That's 13 minutes.
**zara.ahmed:** That's fine. Legal SLA is 2 hours. We'll make it.

**12:14 AM**
**notification-service:** ✅ 1,987,422 notifications delivered. 89,341 in DLQ fallback queue (SMS). Estimated completion: 12:29 AM.

**zara.ahmed:** We made it. But we shouldn't have been in this position.

---

**The Next Morning — Postmortem Prep**

Zara opens the postmortem doc and types the first line: "We delivered 2M notifications in 41 minutes under hostile conditions. That is not acceptable as a heroic outcome — it should be the baseline."

This is the fourth time in your career you've designed a notification system under pressure: Beacon Media's Champions League push blast, NeuroLearn's exam reminder cascade, SkyRoute's irregular operations broadcast. Each time, the system was different. Each time, a variation of the same problems appeared. Deduplication. Rate limiting. Multi-channel fallback. Provider capacity. This time you had all of them at once, at 2AM, with a legal deadline.

The question now is not how to survive the next one. It's how to design a system that makes the next one boring.

---

### Problem Statement

VenueFlow must deliver 2 million fan notifications within a 2-hour legal window when an event is cancelled. The current notification service fans out at the ticket level (not fan level), has no deduplication, and has no multi-channel orchestration — it relies on a single SendGrid integration that rate-limits under burst load. Design a notification system that handles cancellation events at this scale reliably, with multi-channel delivery, deduplication, priority queue bypass, and DLQ fallback.

### Explicit Requirements

1. Fan-level deduplication: a fan with N tickets receives exactly 1 notification per cancellation event
2. Multi-channel delivery: FCM/APNs (push) > Email (primary) > SMS (fallback), in priority order
3. Per-provider rate limiting with dynamic throughput allocation across active providers
4. Emergency queue bypass: cancellation notifications skip standard priority queues and enter a dedicated high-priority lane
5. DLQ for failed deliveries with automatic fallback channel escalation
6. Delivery audit log: every notification attempt (success or failure) logged with timestamp, channel, provider, and fan identifier
7. Legal SLA monitoring: real-time dashboard showing projected delivery completion time vs. contractual deadline

### Hidden Requirements

- **Hint: re-read Ravi's message about the queue depth hitting 4.2M.** The fan-out is happening at the ticket level. Where in the notification pipeline should fan-level aggregation happen — before the queue, at the queue, or at the consumer? What are the tradeoffs of each placement?

- **Hint: re-read the line about Mailgun credentials never being used in production.** What does it mean to onboard a backup provider for the first time during an incident? What pre-incident validation is required to make a backup provider actually usable under pressure?

- **Hint: re-read the exchange about 80k DLQ messages routing to Twilio SMS.** SMS fallback at 100/sec for 80k messages takes 13 minutes. Is that acceptable? What would change your answer, and what SLA does the legal contract actually define — per-fan or per-channel?

- **Hint: re-read Zara's final comment.** "We delivered 2M notifications in 41 minutes under hostile conditions. That is not acceptable as a heroic outcome." What does a notification system designed for boring look like — specifically, what pre-event capacity contracts, synthetic probing, and provider health checks would have made this incident a non-event?

### Constraints

- **Fan base**: 2,000,000 unique fans per major cancellation event
- **Legal SLA**: 2 hours from confirmed cancellation to all fans notified
- **Push coverage**: ~60% of fans have app installed (FCM/APNs)
- **Provider limits**: SendGrid 1k/sec burst recovery, Mailgun 2k/sec, FCM 8k/sec, Twilio SMS 100/sec (standard tier)
- **Ticket-to-fan ratio**: average 1.8 tickets per fan; tail at 20 tickets (group bookings)
- **DLQ volume**: expect 3–5% bounce rate on email (bad addresses, hard bounces)
- **Audit retention**: notification delivery logs must be retained 7 years (EU consumer protection)
- **On-call team**: 2 engineers available at 11PM on a weeknight
- **Infrastructure**: existing BullMQ + Redis + Node.js notification service

### Your Task

Redesign the VenueFlow notification system to handle mass-cancellation events reliably. Define the fan aggregation strategy, multi-channel orchestration, per-provider rate limiting, emergency queue bypass, DLQ escalation chain, and delivery audit schema. Explicitly show how this design evolves from the three previous notification system designs in the curriculum.

### Deliverables

- [ ] Mermaid architecture diagram: cancellation event → fan aggregation → priority queue → multi-channel orchestrator → per-provider workers → DLQ → fallback chain
- [ ] Database schema for notification audit log (column types, indexes, partition strategy for 7-year retention)
- [ ] Scaling estimation (show math):
  - Required throughput (notifications/sec) to meet 2-hour SLA with 20% buffer
  - Provider capacity allocation across FCM, email (SendGrid + Mailgun), SMS
  - Queue depth and worker count required
  - Redis deduplication key memory footprint (2M keys, TTL strategy)
- [ ] Tradeoff analysis (minimum 3):
  - Fan aggregation before queue vs. deduplication at consumer level
  - Single high-priority queue vs. per-channel queues with independent workers
  - Fail-open (deliver duplicate) vs. fail-closed (deduplicate strictly) under Redis failure
- [ ] Evolution summary: one paragraph comparing this design against Beacon Media (Ch. 20), NeuroLearn (Ch. 43), and SkyRoute (Ch. 59) — what pattern evolved, what stayed the same
- [ ] Pre-incident validation checklist: what must be verified for backup providers before an incident window

---
