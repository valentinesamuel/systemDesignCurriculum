---
**Title**: VenueFlow — Chapter 113: Real-Time at Scale — 500k Concurrent Fans
**Level**: Staff
**Difficulty**: 9
**Tags**: #realtime #websockets #pubsub #redis #backpressure #distributed
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 20 (Beacon Media fan-out), Ch. 59 (SkyRoute irregular ops), Ch. 116 (ticket inventory consistency)
**Exercise Type**: System Design
---

### Story Context

**#eng-architecture — Slack**
**Tuesday, 09:14**

**zara.ahmed [CTO]:** Morning. I want to do a deep-dive on the seat map today. Scheduling a room for 11.
**zara.ahmed:** Bring whoever you need. This one's been on my list for eight months.

**09:16**
**tomasz.kowalski [Staff Eng, Platform]:** I'll be there. Fair warning: I've looked at this twice and I have more questions than answers.

**09:19**
**[you]:** I've been reading the incident reports from the Beyoncé pre-sale. Can you pull the WebSocket metrics from that window? Specifically connection count vs message lag.

**tomasz.kowalski:** Already pulled them. You're not going to like the numbers.

---

**VenueFlow Engineering — Conference Room B2 — 11:02 AM**

Tomasz connects his laptop to the projector. The graph he loads shows a familiar shape: a steep spike followed by a plateau that should be flat but isn't. He points at the y-axis without saying anything. Eight to twelve seconds. That's the seat-update latency during the Beyoncé pre-sale window.

"We had 280,000 concurrent WebSocket connections at peak," he says. "Our connection servers were at capacity. When a seat status changed — reserved, held, sold — the update had to fan out to every open connection watching that event. We have a single Redis pub/sub channel per event. At 280k subscribers and roughly 50,000 seat changes per minute during peak onsale, the fan-out queue backed up."

Zara crosses her arms. "Tell them about the errors."

Tomasz pulls up the next chart. **180,000 'Oops! That seat was just taken' errors in a four-minute window.** The error isn't from a double-sell — it's from a display inconsistency. A fan sees a seat as green, clicks it, and by the time the reservation request reaches the backend, the seat has been held for eight seconds and the UI didn't know. The seat map was lying.

"The root issue," Tomasz says, "is that we're running eight WebSocket gateway nodes, but they all subscribe to the same Redis pub/sub channel per event. Redis is doing the fan-out to eight nodes, and each node is doing its own fan-out to ~35,000 connections. The bottleneck isn't Redis — it's the Node.js event loop on each gateway doing 35,000 individual socket writes per seat update message."

You pull up your own notes. "What's the connection distribution strategy? Are fans pinned to a specific gateway?"

Tomasz pauses. "Round-robin at load balancer level. No affinity."

Zara looks at you. "Which means what?"

"Which means when a slow consumer on Gateway 3 backs up, it has no relationship to the other connections on that node. Backpressure is per-node but the fan-out is global. You get head-of-line blocking per gateway."

**Slack DM — tomasz.kowalski → you — 11:34**
**tomasz.kowalski:** Also worth knowing: we have a 500k connection target for the Taylor Swift onsale in six weeks. Current architecture tops out around 320k before we start dropping heartbeats.
**tomasz.kowalski:** Zara knows. That's why you're here.

**[you]:** What's the heartbeat interval right now?
**tomasz.kowalski:** 30 seconds. We had one incident where a proxy was silently dropping connections and we didn't know for four minutes.
**[you]:** 30 seconds is too long. We need to know in under 10.
**tomasz.kowalski:** Agreed. There's also no reconnection backoff. When a gateway restarts, all 35k connections reconnect simultaneously.
**[you]:** Thundering herd on reconnect. Of course.
**tomasz.kowalski:** Welcome to VenueFlow.

---

The seat map is the product. When it lies, fans lose trust in the whole ticketing experience. The Taylor Swift onsale is in six weeks. The engineering team needs a real-time architecture that can handle 500,000 concurrent connections, deliver seat status updates in under 200ms end-to-end, handle slow consumers without head-of-line blocking, and recover from gateway restarts without a reconnection storm.

This is not a nice-to-have. The CEO has told the artist's management team that VenueFlow has "resolved the technical issues from the 2024 pre-sale." The engineering team found out about that statement from a press release.

---

### Problem Statement

VenueFlow's real-time seat map relies on WebSocket connections fanned out through Redis pub/sub. At current scale (280k concurrent connections), seat update latency reaches 8–12 seconds during peak onsale windows, causing hundreds of thousands of stale-state errors. The system must scale to 500,000 concurrent connections with a seat update latency target of under 200ms P99, graceful backpressure handling for slow consumers, and resilient reconnection behavior.

### Explicit Requirements

1. Support 500,000 concurrent WebSocket connections across a horizontally-scalable gateway tier
2. Deliver seat status updates (held, reserved, sold, released) to all watching connections in under 200ms P99
3. Handle slow consumers without blocking faster consumers on the same gateway node
4. Heartbeat interval: detect dead connections within 10 seconds
5. Reconnection after gateway restart must use jitter-based exponential backoff to prevent thundering herd
6. Fan-out architecture must scale to 50,000 seat update events per second at peak onsale
7. Seat map state must be reconstructable from scratch if a client reconnects mid-sale
8. System must degrade gracefully (drop slow consumers) rather than back-pressuring the entire fan-out path

### Hidden Requirements

- **Hint: re-read the conversation about round-robin load balancing.** What problem does Tomasz identify with no connection affinity? How does consistent hashing change the fan-out topology, and what does it mean for seat map state locality?

- **Hint: re-read the Slack DM about the press release.** The CEO has made a public commitment. What does that mean for the operational runbook — specifically, what monitoring and alerting must be in place before the Taylor Swift onsale, not just the architecture?

- **Hint: re-read the line about seat map state being reconstructable.** If a fan reconnects mid-sale, they need current seat state, not just future updates. Where is the authoritative "current snapshot" of seat status stored, and how does it interact with the pub/sub stream?

- **Hint: re-read the 30-second heartbeat interval discussion.** A proxy silently dropping connections went undetected for four minutes. What class of failure does a 10-second heartbeat still miss, and what additional mechanism is required?

### Constraints

- **Concurrent connections**: 500,000 peak (Taylor Swift onsale target)
- **Seat update rate**: 50,000 events/second at onsale open
- **Latency SLA**: <200ms P99 seat update delivery (fan open to fan browser)
- **Heartbeat**: dead connection detection within 10 seconds
- **Reconnection storm budget**: gateway restart must not produce >5% of connections reconnecting in the same 1-second window
- **Gateway nodes**: target 10 nodes at peak, horizontally scalable
- **Redis Cluster**: existing deployment, 6 shards
- **Team size**: 4 platform engineers + you
- **Timeline**: 6 weeks to Taylor Swift onsale
- **Budget constraint**: no new infrastructure categories — must use existing Redis Cluster and Node.js gateway tier

### Your Task

Design the real-time seat map architecture that supports 500,000 concurrent WebSocket connections with sub-200ms update latency. Define the connection sharding strategy, fan-out topology, backpressure handling, heartbeat/reconnection protocol, and state snapshot mechanism.

### Deliverables

- [ ] Mermaid architecture diagram showing WebSocket gateway tier, Redis Cluster pub/sub topology, seat update fan-out path, and client reconnection flow
- [ ] Database schema for seat state snapshot store (column types and indexes) — used for state reconstruction on reconnect
- [ ] Scaling estimation (show math):
  - Connections per gateway node at 500k
  - Fan-out messages per second per gateway
  - Redis pub/sub throughput at 50k seat events/sec across 6 shards
  - Memory footprint per connection (WebSocket overhead + seat subscription metadata)
- [ ] Tradeoff analysis (minimum 3):
  - Consistent hashing for connection affinity vs. stateless round-robin
  - Per-event Redis channel vs. per-venue vs. per-section sharding
  - Dropping slow consumers (availability) vs. applying backpressure (consistency)
- [ ] Reconnection backoff algorithm with jitter — pseudocode showing the client-side reconnect schedule
- [ ] Operational runbook checklist: pre-onsale monitoring setup, alerting thresholds, kill switch for slow-consumer drain

---
