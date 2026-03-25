---
**Title**: GigGrid — Chapter 86: Real-Time at Scale — WebSockets, SSE, and Pub/Sub
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #realtime #websockets #sse #pubsub #redis #long-polling #backpressure
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 113
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**#engineering — GigGrid**
`Thursday 09:45` **kai.moreau** *(Platform Engineer)*: the load balancers are at 94% CPU. i'm seeing 400k requests per second to the /poll endpoint.
`09:47` **you**: /poll? what's that?
`09:48` **kai.moreau**: workers poll every 5 seconds to check if there's a new job for them
`09:49` **you**: 2 million active workers × 1 poll every 5 seconds = 400,000 requests per second
`09:50` **kai.moreau**: yes
`09:51` **you**: to one endpoint
`09:52` **kai.moreau**: yes
`09:53` **you**: and each poll response is...?
`09:54` **kai.moreau**: 99.3% of the time: "no new jobs." 0.7% of the time: "here's a job offer"
`09:55` **you**: so 397,200 requests per second are returning an empty response
`09:56` **kai.moreau**: yes
`09:57` **you**: and the load balancers are dying under that load
`09:58` **kai.moreau**: the load balancers plus the API servers plus the database (because each poll queries the job queue table)
`09:59` **amara.chen**: this was already broken when I took over. i didn't have time to fix it. now we do.
`10:00` **you**: how many workers use the mobile app vs the web dashboard?
`10:01` **kai.moreau**: 85% mobile (iOS/Android), 15% web
`10:02` **you**: mobile apps behind corporate proxies?
`10:03` **kai.moreau**: unknown. workers are from 30 countries. some are in enterprise hospital networks, some are in warehouses with restricted wifi.
`10:04` **you**: that changes the protocol choice. WebSockets don't work well behind some corporate proxies.

**Whiteboard Session — Platform Architecture**

**You**: The current polling architecture is simple but catastrophically inefficient. At 2M workers, even 1 poll per 5 seconds is 400k RPS. The fix is push-based real-time: the server pushes job offers to workers when they're available, instead of workers asking every 5 seconds.

**Kai**: We considered WebSockets in 2022. The concern was connection state — 2M long-lived TCP connections is a lot to manage.

**You**: It is a lot. But 2M persistent connections is far better than 400k requests per second. Each persistent WebSocket connection sits idle 99.3% of the time. The resource cost per connection is much lower than the cost per request under polling.

**Kai**: What about SSE?

**You**: Server-Sent Events is worth considering. It's simpler than WebSockets — unidirectional, works over standard HTTP/2, doesn't need special proxy support. For job dispatch — which is unidirectional (server tells worker about a job) — SSE is architecturally appropriate.

**Amara**: What are the failure modes?

**You**: The main challenge is connection sharding. At 2M concurrent connections, you can't maintain all connections on one server. You need to shard workers to connection servers by consistent hash. When a job needs to be dispatched to worker ID #GG-884271, the dispatch system needs to know which connection server that worker is connected to.

**Kai**: And if the worker's connection server goes down?

**You**: The worker's app reconnects. The reconnection needs to re-register the worker's availability with the new connection server. There's a brief window of potential missed dispatches during reconnection — the design needs to handle that.

---

### Problem Statement

GigGrid's 2 million active workers poll a single API endpoint every 5 seconds to check for job offers. This generates 400,000 RPS of mostly-empty responses, overloading load balancers, API servers, and the database. The architecture needs to shift from client-initiated polling to server-initiated push using WebSockets or SSE, with a pub/sub backplane that routes job offers to the correct worker across multiple connection servers.

---

### Explicit Requirements

1. Job dispatch latency must be < 500ms from offer creation to worker notification
2. System must support 2M concurrent connected workers
3. Workers must not miss job offers during brief reconnection windows (< 5 seconds)
4. The solution must work for workers behind corporate proxies that may block WebSocket upgrades
5. Connection servers must be horizontally scalable (add servers without reconfiguring workers)
6. Backpressure mechanism required: a slow consumer (worker with poor network) must not block dispatches to other workers

---

### Hidden Requirements

- **Hint**: Re-read Kai's message: "15% web, 85% mobile." Mobile workers frequently lose connectivity (entering buildings, switching cell towers). When a worker reconnects after 60 seconds of disconnection, they may have missed job offers that were dispatched during the outage. How does your architecture handle "catch up on missed messages" without storing all dispatches indefinitely?

- **Hint**: The dispatch system needs to know which connection server a worker is connected to. If 2M workers are distributed across 50 connection servers, that's a routing table of 2M entries. How does the dispatch system efficiently look up "which server is worker #GG-884271 connected to" at 2,000 job dispatches/minute?

---

### Constraints

- 2M concurrent active workers
- 50k job dispatches/day = ~35/minute average, 2,000/minute peak
- 85% mobile (iOS/Android), 15% web
- Workers behind enterprise proxies in some cases
- Current infrastructure: Redis Cluster (available for pub/sub), Kubernetes
- Connection server budget: can add up to 50 dedicated connection servers

---

### Your Task

Design the real-time job dispatch architecture for GigGrid using persistent connections (WebSocket or SSE) with Redis pub/sub backplane and consistent-hash connection sharding.

---

### Deliverables

- [ ] Protocol selection: WebSocket vs SSE comparison for GigGrid's specific constraints (proxy support, proxy detection strategy)
- [ ] Mermaid diagram: Architecture showing workers → connection servers → Redis pub/sub → dispatch service
- [ ] Connection sharding: Consistent hash ring for routing workers to connection servers, and how the dispatch service finds the correct server for a given worker ID
- [ ] Reconnection and missed message handling: How workers catch up on offers missed during a < 60 second disconnection window
- [ ] Backpressure design: How the connection server handles a slow consumer without blocking fast consumers
- [ ] Load calculation: At 2M connections across 50 servers, what's the per-server connection count? What's the memory footprint per connection? Show the math.
- [ ] Tradeoff analysis (minimum 3):
  - WebSocket vs SSE vs long polling for this specific use case
  - Redis pub/sub vs Kafka for the connection server backplane
  - Per-worker channels vs broadcast with client-side filtering
