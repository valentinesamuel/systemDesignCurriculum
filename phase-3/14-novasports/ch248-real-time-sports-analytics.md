---
**Title**: NovaSports — Chapter 248: Seven Seconds to National Television
**Level**: Staff
**Difficulty**: 9
**Tags**: #real-time #streaming #analytics #SLA #broadcast #rolling-windows #observability
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 245, Ch. 247, Ch. 105
**Exercise Type**: System Design
---

### Story Context

**From:** Marcus Obi <marcus.obi@broadcast-partners.com>
**To:** Dev Chatterjee <dev@novasports.com>
**CC:** Rosa Delgado <rosa@novasports.com>; You
**Subject:** Stats Feed Integration — Technical Requirements and SLA
**Date:** Wednesday, 11:15 AM

Dev,

Following our partnership announcement, I wanted to formalize the technical requirements for the NovaSports Stats Feed that will power our on-screen broadcast graphics.

We have 22 million peak viewers across our Premier League coverage. The on-screen graphics system — the overlay you see showing "Messi: 3 shots on target, xG 0.84" — reads from an API we call every 15 seconds per visible player. During key moments (goal, red card, penalty decision), it calls on demand, not on the polling interval.

**The hard constraint:** Our broadcast has a 7-second delay from real-world event to air. Our graphics system budget is 50ms from the time we call your API to when we can commit to rendering the graphic. If we miss this window, the stat is absent from the broadcast — not wrong, just absent. Our producers will call your team directly.

**Volume:**
- 20 simultaneous live matches during peak weekend rounds
- ~30 active players per match = 600 player stat objects in flight at any time
- 15-second polling: 600 / 15 = 40 API calls/sec baseline
- On-demand spikes: a goal triggers simultaneous calls for all 22 players on the pitch = 22 calls in < 1 second
- We estimate 6 goals per hour across all 20 matches = 132 on-demand burst calls/hour

**Stat types required (must be computed in real time, not looked up from a database):**
- Rolling window: shots on target (last 15 minutes), passes completed (last 15 minutes)
- Cumulative: total goals, assists, yellow cards in current match
- Percentile: "Messi's passing accuracy is in the top 5% of attackers this season" — must refresh hourly during the season
- Momentum score: composite score based on last 5 minutes of activity — proprietary formula we're providing

**SLA:** 99.999%. This is broadcast. A single missed major stat during a Champions League final is unacceptable.

**One more thing:** We are licensing exclusive rights to the NovaSports stats API for broadcast use in 14 countries. Your API must be able to prove, in an audit log, that no other broadcast partner received data in those territories during our exclusivity window. Our legal team will follow up.

Looking forward to building this together.

Marcus Obi
Head of Technology Partnerships, UEFA Broadcast Partners

---

You read the email twice. Then a third time.

**You [Slack DM to Rosa, 11:34 AM]:**
> "Have you read Marcus Obi's email?"

**Rosa Delgado [Slack DM, 11:35 AM]:**
> "Three times."

**You [Slack DM, 11:35 AM]:**
> "99.999% uptime. For 20 simultaneous matches. 50ms p100 response time for on-demand calls."

**Rosa Delgado [Slack DM, 11:36 AM]:**
> "p100. Not p99. p100."
> "Also: rolling window queries on live data. That's not a database read. That's stream processing."
> "Also: 'momentum score based on last 5 minutes' — that's stateful stream computation. They're sending us the formula?"

**You [Slack DM, 11:37 AM]:**
> "Just got the formula. It's a weighted event count over a 5-minute sliding window with exponential decay. I need Flink."

**Rosa Delgado [Slack DM, 11:38 AM]:**
> "You need Flink, a dedicated stat materialization layer, sub-10ms read path, and circuit breakers for every match feed independently so one bad match doesn't affect the other 19."
> "Also: the exclusivity audit log. That's a separate system entirely."
> "When do we start?"

**You [Slack DM, 11:39 AM]:**
> "Now."

---

**#broadcast-stats — Engineering channel, 2:15 PM**

**Chioma Osei [Senior Eng]:** I've been modeling the stat computation. The rolling window queries are the problem. Every 15 seconds, for 600 players, we need to answer: "How many shots has this player taken in the last 15 minutes?" That's a range query over an event stream. At query time.

**You:** We can't do range queries at query time against a raw event log at broadcast latency. We need pre-materialized windows.

**Chioma Osei:** Agreed. I'm thinking: Flink with 15-minute tumbling and sliding windows, pre-computing the stats, writing to a Redis hash per player per match. The API just does a Redis GET. Sub-5ms.

**You:** What happens when a goal is scored and all 22 players' stats get refreshed simultaneously?

**Chioma Osei:** ...I need to think about that.

**Dev Chatterjee:** While you're thinking: Legal just responded to Marcus Obi. They've agreed to the exclusivity audit log requirement. Our obligation: within 5 minutes of any API call to a non-exclusive territory, we must be able to prove which partner received which stats data in which territory and when. It has to be cryptographically verifiable. Marcus's legal team used the phrase "court-admissible."

Another silence in the channel.

**You:** I'll add it to the architecture doc.

---

### Problem Statement

NovaSports must build a real-time stats feed that powers live broadcast graphics. The system must compute complex statistical queries (rolling windows, percentiles, momentum scores) with sub-50ms response latency for broadcast partners. During live matches, 20 games run simultaneously with up to 22 on-demand stat requests per goal event. The SLA is 99.999% — failure is visible to 22 million viewers on national television. The system must also maintain a cryptographically verifiable audit log of which broadcast partners received which data in which territories, for legal exclusivity enforcement.

### Explicit Requirements

1. API response latency: < 50ms p100 for broadcast partner calls (not p99 — p100)
2. 20 simultaneous live matches, 30 active players per match = 600 player stat objects
3. Polling rate: 40 API calls/sec baseline; goal event burst: 22 calls in < 1 second per match
4. Rolling window stats: 15-minute sliding window, refreshed in real time
5. Cumulative match stats: updated within 1 second of event
6. Percentile stats: hourly refresh during season
7. Momentum score: 5-minute sliding window with exponential decay (stateful)
8. SLA: 99.999% (< 5.26 minutes downtime per year per match)
9. Exclusivity audit log: cryptographically verifiable, per-territory, per-partner, court-admissible
10. 20 match feeds must be independently circuit-broken (one bad match feed cannot affect others)

### Hidden Requirements

1. **Hint: re-read Dev's message about the exclusivity audit log.** "Within 5 minutes of any API call, prove who received what." This is not just logging — it's a cryptographic proof system. HMAC signatures on response payloads, stored in an append-only log. The requirement is real-time audit, not retroactive reconstruction.

2. **Hint: re-read Chioma's question about goal events.** When a goal is scored, all 22 players' stats are simultaneously invalidated in the rolling window because the event changes all momentum scores. This is a coordinated cache invalidation problem — 22 keys evicted simultaneously, all 22 being requested simultaneously by the broadcast API. The thundering herd is not random — it is perfectly synchronized to the goal event.

3. **Hint: re-read the "20 simultaneous matches" constraint alongside the 99.999% SLA.** 99.999% per match means each match can fail for 5.26 minutes per year. But if the SLA is contractually per-system (not per-match), all 20 matches must be up simultaneously for 99.999% of the time. This is a much harder requirement and changes the redundancy model.

4. **Hint: re-read the broadcast delay note.** "7-second broadcast delay." The stats system has 7 seconds from real-world event to on-screen. The broadcast partner calls the API at T+7s. The stat must be available at T+0 (real-world time) so it's ready at T+7s. This means the stat materialization must happen before the fan sees the stat — the pipeline latency budget is not 50ms, it is 50ms for the *read*, but the write pipeline must complete within seconds of the event.

### Constraints

- **Peak concurrent matches**: 20 (Premier League weekend rounds)
- **Players tracked**: 600 simultaneously (30 per match × 20 matches)
- **Baseline API RPS**: 40 calls/sec; goal-event burst: 22 calls/sec per match × 20 matches = 440 calls/sec simultaneous burst
- **Rolling window size**: 15 minutes, sliding (updated on every event)
- **Momentum window**: 5 minutes, exponential decay (λ = 0.7 per minute)
- **Event ingestion rate**: ~2,500 game events/sec across 20 matches (passes, tackles, shots, etc.)
- **SLA**: 99.999% for broadcast API
- **Audit log**: 100% write coverage, HMAC-signed responses, append-only
- **Circuit breaking**: per-match isolation, 19 matches must continue if 1 feed fails
- **Team**: 5 engineers, 10-week build window (season starts 10 weeks from now)
- **Budget**: $140K/month for broadcast stats stack

### Your Task

Design the real-time stats computation and serving architecture. Define the stream processing pipeline that materializes rolling window and momentum stats. Design the pre-materialized stat serving layer that achieves < 50ms p100. Address the synchronized cache invalidation problem on goal events. Design the cryptographically verifiable exclusivity audit log. Define the per-match circuit breaker isolation model.

### Deliverables

- [ ] Mermaid architecture diagram: event ingestion → stream processing (Flink) → stat materialization layer → broadcast API → audit log
- [ ] Stream processing design: how rolling windows, cumulative stats, and momentum scores are computed using Flink (window types, state management, watermarks)
- [ ] Stat materialization layer design: pre-computed Redis structure, TTL strategy, coordinated invalidation on goal events
- [ ] Goal-event thundering herd mitigation: how 22 simultaneous cache misses are handled without cascade
- [ ] Cryptographic audit log design: HMAC signature scheme, append-only storage, query path for legal review
- [ ] Per-match circuit breaker: how match feed isolation is implemented without shared state
- [ ] Database schema for stat materialization store (Redis hash structure + persistent backup)
- [ ] Scaling estimation (show math step by step):
  - Event ingestion throughput
  - Flink job sizing (cores, memory, parallelism)
  - Redis memory footprint for 600 players × all stat types
  - Burst handling capacity for simultaneous goal events
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs)
- [ ] Cost modeling ($X/month)
- [ ] SLA math: 99.999% error budget calculation and how the architecture defends it

### Diagram Format

Mermaid syntax. Show at minimum:
- Match event feed ingestion (per-match Kafka topics)
- Flink stream processing jobs (per-match or shared with isolation)
- Stat materialization layer (Redis, structured by match_id + player_id)
- Broadcast API tier (read-only, low-latency)
- Goal event invalidation path (coordinated, not thundering herd)
- Audit log pipeline (HMAC-signed, append-only)
- Circuit breaker isolation points between matches
