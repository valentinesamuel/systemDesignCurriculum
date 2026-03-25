---
**Title**: NexusWealth — Chapter 234: VIX 80 — The ERISA Clock Is Ticking
**Level**: Staff
**Difficulty**: 9
**Tags**: #spike-scale-10x #distributed-systems #compliance #financial-systems #incident-response
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 231, Ch. 232, Ch. 233
**Exercise Type**: Incident Response / System Evolution
---

### Story Context

**#incidents — NexusWealth Engineering**

```
[07:02:11] @pagerduty — CRITICAL: Election Processing Queue Depth > 50,000
[07:02:18] @pagerduty — CRITICAL: Election Processing Queue Depth > 100,000
[07:02:31] @pagerduty — CRITICAL: Election Processing Queue Depth > 200,000
[07:03:00] @you — Joining bridge. What happened?
[07:03:14] @marcus.delgado — VIX hit 80 at market open. Check Bloomberg.
                              Panic selling. Everyone is moving out of equities
                              to stable value funds simultaneously.
[07:03:42] @priya.nair — This happened at Fidelity in 2020 during COVID. Their
                          election processing backed up for 6 hours. Participants
                          called in and were told their elections "may not process."
                          We cannot have that story.
[07:04:01] @yusuf.okonkwo — I've been watching fund flow data. 2.5 million elections
                              submitted since 6:45 AM. That's 500,000 per hour.
                              Normal day is 50,000 total. This is 10x in the first
                              90 minutes of trading.
[07:04:33] @marcus.delgado — Our election processing service is horizontally scaled
                              to 12 workers. Each worker processes ~700 elections/min.
                              Total capacity: 8,400/min. We're receiving 8,300/min
                              right now and that number is climbing.
[07:05:00] @you — We're at 100% capacity with no margin. Any spike above current
                  rate and we start falling behind permanently.
[07:05:22] @renata.kowalski — I need to flag something. ERISA Section 404(c) requires
                              that investment elections are applied to the "next
                              available NAV" — which means today's end-of-day NAV
                              for elections received by 4:00 PM Eastern. Elections
                              that fail to process by 4:00 PM must roll to tomorrow's
                              NAV. If participants don't know this happened, they
                              may believe their election applied to today's lower
                              prices when it actually applied to tomorrow's
                              (potentially higher) prices. That is a disclosure
                              violation.
[07:06:01] @you — Renata — what's the regulatory clock exactly?
[07:06:18] @renata.kowalski — Elections confirmed received by 4:00 PM Eastern today
                              apply to today's NAV. 4:00 PM is the regulatory cutoff.
                              Any election in the queue at 4:01 PM that has not been
                              confirmed gets rolled to next business day NAV. The
                              participant must be notified. Automatically. Before end
                              of business today.
[07:07:00] @you — Current queue depth?
[07:07:04] @marcus.delgado — 847,000 and rising.
[07:07:19] @you — At current processing rate: 8,400/min. 847,000 ÷ 8,400 = 100 minutes
                  to clear. That's 8:47 AM. The queue is currently growing faster
                  than we're clearing it. We will not clear it by 4:00 PM at
                  current throughput. How fast can we add capacity?
[07:08:00] @marcus.delgado — ECS auto-scaling is configured for maximum 20 workers.
                              Hard limit I set last year after a runaway cost incident.
                              Even at 20 workers: 11,667/min. Against 8,300/min
                              incoming and rising — that's 40% headroom. But I don't
                              know where the incoming rate peaks.
[07:08:44] @yusuf.okonkwo — S&P is down 8% at open. This is not going to calm down
                              for hours.
[07:09:00] @priya.nair — Remove the 20-worker limit. Now. Scale to what we need.
                          We figure out the cost later.
[07:09:15] @you — On it. Also: I'm seeing something else. The account balance
                  displays for participants are showing incorrect values. Multiple
                  participants are calling the service center saying their balance
                  dropped 40% overnight. That's not possible given yesterday's
                  close prices.
[07:09:44] @marcus.delgado — Oh no. The balance display reads from the same database
                              that the election processing writes to. When the election
                              processing service is under load, it's holding long
                              transactions. Those long transactions are blocking reads.
                              Participants are seeing partially-applied state.
[07:10:02] @you — So we have two incidents. Election processing backlog AND
                  account balance display showing corrupted state. Both are
                  participant-facing. Both are live right now.
[07:10:30] @priya.nair — How many participants have seen wrong balances?
[07:10:45] @marcus.delgado — The service center has logged 4,300 calls in the last
                              30 minutes.
[07:11:00] @renata.kowalski — If participants made election decisions based on
                              incorrect balance information, that is a material
                              disclosure failure. I'm calling legal.
```

---

**Slack DM — @marcus.webb → @you — 9:45 AM**

**Marcus**: Saw you're on an incident. Recognize the pattern. This exact thing happened at CitiGroup in 2010 during the Flash Crash. Long-running write transactions blocking reads. They called it a "database coupling problem." They spent 18 months fixing it properly after they spent 6 hours fighting it.

**Marcus**: The ERISA clock is the thing nobody's thinking about clearly right now. 4:00 PM is not just a deadline. It's a compliance event. Elections that miss it need automatic participant notifications — that's a second system you probably haven't built. Have you?

**Marcus**: Scale first. Then think about what happens to the elections that can't make the NAV cutoff. That's the design problem.

---

By 10 AM you have scaled election processing to 50 workers and cleared the immediate backlog. But the architectural problems are now visible: write/read coupling on the same database, a 20-worker hard limit that was almost catastrophic, no separation between "elections received" and "elections processed," and — as Marcus correctly identified — no automated participant notification system for elections rolled to the next NAV day.

The VIX is still at 72. The elections are still coming in. You have until 4:00 PM.

### Problem Statement

Scale NexusWealth's election processing pipeline to handle 50x normal load (2.5M elections in a single day vs. 50K baseline) under extreme market volatility conditions. Simultaneously resolve the read/write database coupling that is causing incorrect account balance displays. Design the ERISA NAV cutoff processing: at 4:00 PM, automatically identify every election that will not make today's NAV, roll them to next business day, and send participant notifications — all before end of business.

The constraint that cannot be violated: under ERISA, a participant who submitted an election before 4:00 PM must have that election applied to either today's NAV or next business day's NAV, with explicit notification if the latter. Missing this is a fiduciary violation.

### Explicit Requirements

1. Scale election processing from 8,400/min to 83,000/min (10x) without data loss
2. Separate read path (account balance display) from write path (election processing) — eliminate long transaction blocking
3. Implement hard queue depth monitoring with auto-scale triggers before queue becomes unmanageable
4. At 4:00 PM Eastern: atomically snapshot the NAV cutoff state — which elections made today's NAV, which roll to tomorrow
5. Automatic participant notifications for all rolled elections before end of business (5:00 PM Eastern)
6. Idempotent election submission: duplicate elections during the crisis (participant hitting submit repeatedly) must not result in duplicate fund transfers
7. Audit log of all elections: received timestamp, processed timestamp, NAV date applied, participant notification sent
8. Real-time queue depth dashboard visible to operations team and Renata's compliance team
9. Zero-loss guarantee: no election that arrived before 4:00 PM may be silently dropped
10. Account balance display must show consistent state — no partial-apply visibility

### Hidden Requirements

- **Hint**: Re-read Renata's 7:06 AM message carefully. She says elections received "by 4:00 PM Eastern" apply to today's NAV. The system receives elections from participants in all US time zones. What does "received" mean? Is it the timestamp when the participant submitted, or when the system acknowledged receipt? A participant on the West Coast who submits at 12:59 PM Pacific (3:59 PM Eastern) may have their election in the system's input queue but not yet processed. The ERISA clock uses your acknowledgment timestamp, not the processing timestamp. If the acknowledgment is delayed by queue depth, you have a compliance gap.
- **Hint**: Re-read Marcus Delgado's 7:08 AM message. He says the 20-worker ECS limit was "set last year after a runaway cost incident." When you scale to 50 workers during the crisis, you're overriding a safeguard that was put there for a reason. What cost controls must exist on the unlimited scaling? And what is the process for returning to normal capacity after the crisis — if you forget to scale back down, you're burning $X/hour indefinitely.
- **Hint**: Re-read Marcus Webb's DM: "Elections that miss it need automatic participant notifications — that's a second system you probably haven't built." This is confirmed by the explicit requirements above. But there is a second hidden requirement in his message: if participant notifications fail (email/SMS bounces), you have a compliance obligation to document the failure and the retry attempt. The ERISA disclosure requirement is not satisfied by "we tried to send an email." It requires proof of delivery or documented failed delivery with retry.
- **Hint**: Re-read the incident log at 7:09 AM. Priya says "We figure out the cost later." At 50 workers running for 9 hours during a high-compute day, what does the cost spike actually look like? And who approves that cost? This is a governance question — what is the runbook for declaring a financial emergency that allows cost controls to be bypassed, and who is in the approval chain?

### Constraints

- **Normal election volume**: 50K/day (35/min peak)
- **Crisis volume**: 2.5M in 6 hours = 416K/hour = 6,900/min (14 hours total for the day)
- **Peak crisis rate**: 500K/hour = 8,300/min (during VIX 80 first 2 hours)
- **Required throughput**: 83,000/min (10x normal max with margin)
- **ERISA NAV cutoff**: 4:00 PM Eastern — hard regulatory deadline
- **Notification deadline**: 5:00 PM Eastern — all rolled-election notifications sent
- **Current architecture**: single ECS service, reads/writes share one RDS PostgreSQL instance
- **Hard constraints**: zero election data loss, ERISA cutoff accuracy to the second
- **Team**: 4 engineers, incident conditions, 9-hour window
- **Budget**: cost controls suspended for duration of incident (with documented approval)

### Your Task

Redesign the election processing pipeline for 10x scale under crisis conditions. Address: read/write separation, queue depth-based auto-scaling with cost controls, ERISA NAV cutoff processing at 4:00 PM, participant notification system with delivery confirmation, and the runbook for scaling back down after the crisis.

### Deliverables

- [ ] Mermaid architecture diagram — election submission through NAV cutoff processing through participant notification
- [ ] Database schema — election record (received timestamp vs. processed timestamp), NAV batch assignment, notification delivery tracking
- [ ] Scaling estimation:
  - Workers needed for 83,000 elections/min: how many ECS tasks, how much memory/CPU per task?
  - Queue sizing: at 83,000/min processing and 100,000/min ingest during peak, how deep does the queue grow? How long to drain?
  - NAV cutoff processing: at 4:00 PM, how many elections might be in-flight? Snapshot window calculation.
  - Notification volume: assume 10% of 2.5M elections roll to next NAV day = 250K notifications in 60 minutes. Can your notification system handle this?
- [ ] Tradeoff analysis (minimum 3):
  - Read/write separation: read replica lag vs. CQRS projections vs. materialized views for balance display
  - Election acknowledgment: acknowledge on receipt (before processing) vs. acknowledge on processing — ERISA clock implications of each
  - NAV cutoff atomicity: distributed snapshot at 4:00 PM across a scaled cluster — how do you guarantee every in-flight election is captured?
- [ ] Cost modeling: 50 ECS workers × 9 hours vs. normal 12 workers × 9 hours — cost delta; notification infrastructure for 250K messages; auto-scale trigger settings that minimize unnecessary spend
- [ ] Capacity planning: redesign the permanent architecture so that VIX 80 events require no manual intervention — what does auto-scaling look like with proper cost guardrails?

### Diagram Format

Mermaid syntax.
