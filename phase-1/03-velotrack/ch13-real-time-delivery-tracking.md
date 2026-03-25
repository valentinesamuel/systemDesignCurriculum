---
**Title**: VeloTrack — Chapter 13: Design the Real-Time Driver Tracking System
**Level**: Senior
**Difficulty**: 6
**Tags**: #real-time #kafka #redis #geospatial #websockets #distributed
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 11, Ch. 12, Ch. 15
**Exercise Type**: System Design
---

### Story Context

**Product spec — "Live Driver Tracking" (Notion doc, shared by Emeka)**

```
Feature: Consumer-Facing Live Driver Tracking
Owner: Emeka Eze (CTO)
Status: Design Needed

Background:
Uber Eats and DoorDash show a live map of your driver moving toward you.
Our merchant partners are asking for the same thing for B2B deliveries.
A buyer at a logistics company wants to see their freight driver on a map,
updated in real-time as they approach.

Current state:
- Drivers send location pings every 30 seconds via REST POST /location
- We store latest location in PostgreSQL
- Merchant dashboard polls GET /delivery/{id}/location every 10 seconds
- Results in 6-30 second ETA staleness. Merchants hate it.

Target state:
- Drivers send location pings every 5 seconds
- Merchant dashboard shows location with < 2-second latency (end-to-end)
- Map updates smoothly (no jumps)
- System handles 500,000 active drivers simultaneously at peak

Scale (current → target):
  Active drivers: 50,000 → 500,000
  Location updates/second: 50k/30s ≈ 1,667/s → 500k/5s = 100,000/s
  Merchant dashboard connections: 20,000 → 200,000 concurrent WebSocket connections
```

---

**Architecture discussion — #platform-team, Thursday 2:00 PM**

**Emeka**: Okay, two things jumped out at me immediately. A hundred thousand location
updates per second. Two hundred thousand WebSocket connections. Our current stack
handles neither of those. How do we approach this?

**You**: The location ingestion and the WebSocket fan-out are two separate problems.
For ingestion: 100k RPS is manageable with a Kafka producer. Location pings are
fire-and-forget — we acknowledge receipt and process asynchronously. The challenge
is the fan-out: for each location update, we need to push it to every WebSocket
connection watching that driver. A busy merchant might have 10 staff watching
the same delivery.

**Lindiwe**: Won't WebSockets overwhelm our Node.js servers? Each connection holds
a socket open. Two hundred thousand sockets times the memory per connection...

**You**: Right. A naive implementation tries to hold 200k connections on one server.
That doesn't work. We need a scaled WebSocket architecture — probably a dedicated
WebSocket tier, with location events fanned out via Redis Pub/Sub or a similar
pub/sub mechanism.

**Emeka**: What about geospatial stuff? Can we just store lat/lng in Postgres?

**You**: For current position, Redis is better — geospatial data that changes
every 5 seconds doesn't belong in Postgres. Redis has native GEOADD/GEORADIUS
commands. It's much faster for "what's the current position of driver X" and
"show all drivers within 5km of this address."

**Emeka**: What does the ETA calculation look like?

**You**: ETA is a computation: current position + route + speed model. That's a
separate service. The tracking system's job is just to ingest and distribute location.
ETA sits on top.

---

**Slack DM — Marcus Webb → You, Friday morning**

**Marcus Webb**
Real-time tracking at scale. Two things nobody thinks about until they're on fire:
First: driver location data at 100,000 updates/second. Most of that data is
useless. A driver on a highway for 10 minutes sends 120 updates — but the path
is almost a straight line. You don't need to store every ping. What's your
storage strategy for location history vs current location?
Second: WebSocket connections drop. Mobile networks are terrible. A driver goes
through a tunnel, disconnects, reconnects 3 seconds later. The merchant's map jumps.
How do you make the driver's path on the map smooth, even with connection gaps?

---

### Problem Statement

VeloTrack needs to build a real-time driver tracking system that ingests 100,000
location updates per second from 500,000 active drivers and fans out those updates
to 200,000 concurrent WebSocket connections (merchant dashboards) with < 2-second
end-to-end latency. The current polling-based REST architecture cannot scale to
these requirements.

### Explicit Requirements

1. Ingest 100,000 location updates/second (500k drivers × 1 ping/5 seconds)
2. Fan out each location update to all WebSocket connections watching that driver
   with < 2-second end-to-end latency
3. Support 200,000 concurrent WebSocket connections
4. Store current driver location with geospatial query support (nearest drivers,
   drivers within radius)
5. Maintain at least 24 hours of location history per driver for audit/dispute purposes
6. Handle driver reconnects gracefully — map should not "jump" on reconnect

### Hidden Requirements

- **Hint**: Marcus Webb raised the question of storage strategy. At 100,000
  updates/second × 86,400 seconds/day, how many location records is that per day?
  At an average record size of 100 bytes, what is daily storage? Can you store
  every ping, or do you need a sampling/compression strategy?
- **Hint**: The product spec says "map updates smoothly (no jumps)." Smooth
  interpolation on the client side requires the client to know the driver's
  speed and heading, not just lat/lng. What additional fields does each location
  update need to carry to enable client-side path smoothing?
- **Hint**: WebSocket connections are stateful — a connection to server A can't
  be transferred to server B. If you have 200k connections across N servers and
  a location update arrives, how does the right server know which of its connections
  are watching that driver? (Hint: Redis Pub/Sub channel per driver_id)

### Constraints

- **Driver pings**: 100,000 updates/second at peak
- **WebSocket connections**: 200,000 concurrent at peak
- **End-to-end latency SLA**: < 2 seconds (driver sends update → merchant sees it)
- **Location history retention**: 24 hours minimum; 7 days preferred for disputes
- **Current position storage**: Must support GEORADIUS queries within 50ms
- **Infrastructure budget**: Within current AWS spend (scaled horizontally is fine,
  no new managed services without approval)
- **Message size**: Location ping ~200 bytes (driver_id, lat, lng, speed, heading, timestamp)

### Your Task

Design the real-time driver tracking architecture — from driver location ping to
merchant WebSocket update. Address ingestion, storage, pub/sub fan-out, WebSocket
scaling, and location history.

### Deliverables

- [ ] **Architecture diagram** (Mermaid) — full pipeline: driver app → ingestion API
  → Kafka → location processor → Redis (current position + pub/sub) → WebSocket
  servers → merchant dashboard
- [ ] **Location data model** — what is stored where: current position (Redis GEOADD
  schema), location history (what store, what schema), what gets pruned and when
- [ ] **WebSocket fan-out design** — how does a location update reach the right
  WebSocket connections across N servers? Show the Redis Pub/Sub channel design.
- [ ] **Scaling estimation** — at 100,000 updates/second:
  - How many Kafka partitions?
  - How many WebSocket servers at 10,000 connections/server?
  - What is Redis memory footprint for 500,000 current driver positions (GEOADD)?
  - What is daily location history storage at full ping rate?
- [ ] **Location history storage decision** — do you store every ping or sample?
  Show the storage math that motivates your decision.
- [ ] **Tradeoff analysis** — minimum 3 tradeoffs:
  1. WebSocket vs Server-Sent Events (SSE) for merchant dashboard
  2. Redis Pub/Sub vs Kafka for the WebSocket fan-out layer
  3. Storing every ping vs sampling (e.g., every 30s for history, every 5s for real-time only)

### Diagram Format

```mermaid
graph LR
  DA[Driver App] -->|POST /location 100k/s| ING[Ingestion API]
  ING -->|produce| KT[Kafka: location-events]
  KT -->|consume| LP[Location Processor]
  LP -->|GEOADD| RC[Redis: Current Position]
  LP -->|PUBLISH driver:{id}| RP[Redis Pub/Sub]
  LP -->|write| TS[(Location History DB)]
  RP -->|SUBSCRIBE| WS1[WebSocket Server 1]
  RP -->|SUBSCRIBE| WS2[WebSocket Server N]
  WS1 -->|push update| MD[Merchant Dashboard]
```
