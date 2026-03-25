---
**Title**: CHECKPOINT 2 — Senior → Strong Senior Interview at Stratum Systems
**Level**: Strong Senior (gate)
**Difficulty**: 10
**Tags**: #promotion-checkpoint #distributed #consistency #notifications #search #synthesis
**Estimated Time**: 3 hours (treat as real interview)
**Related Chapters**: Ch. 1–63 (synthesis of all Phase 1 content)
**Exercise Type**: Promotion Checkpoint
---

### Story Context

**The application — three weeks before the checkpoint**

After SkyRoute, you take a week off. You're approached by a recruiter from
Stratum Systems — a company you know by reputation. Stratum builds the backend
infrastructure for global freight forwarding: containerized shipping, cross-border
customs, multi-carrier logistics. 240 countries. $4.2 trillion in freight
value managed annually.

The job description is intentionally sparse:

```
Stratum Systems — Staff Engineer, Distributed Infrastructure

Stratum manages the world's freight. Our backend processes 18 million
shipment events per day across 240 countries. We are looking for engineers
who have solved real consistency, scale, and reliability problems — not
engineers who have read about them.

Process: Technical phone screen → On-site system design (3 hours) → Decision.

We do not provide interview preparation materials. The design problem will
be ambiguous. You will be expected to ask clarifying questions.
```

You apply. You get the phone screen. You pass. The on-site is scheduled.

---

**The interview — Monday, 9 AM, Stratum Systems New York office**

The conference room has a whiteboard and three people you've never met:

**Priya Gangadharan** [Principal Engineer at Stratum]: I'll lead the session.
**Devraj Iyer** [Senior Staff Engineer at Stratum]: I'll ask follow-ups.
**Unknown third person** [who doesn't introduce themselves and sits in the back]

**Priya Gangadharan**: Thanks for coming. The format: I'll give you a brief.
It's intentionally incomplete. You should ask clarifying questions. We're
evaluating your questions as much as your answers. You have 3 hours.

She writes on the whiteboard:

```
Stratum Systems processes 18 million shipment events per day.
A "shipment event" is: a container scanned, a customs declaration filed,
a carrier hand-off logged, a delivery confirmed.

Design a system that:
1. Accepts 18M events/day from 240 countries
2. Provides real-time tracking (a shipper can query: where is my container?)
3. Notifies stakeholders when key events occur (container arrives at port,
   customs cleared, delivery window confirmed)
4. Supports document generation (customs forms, bill of lading)

That's it. Go.
```

**You**: [Asks clarifying questions first — this is part of the exercise]

---

**Clarifying questions you should ask** (the interviewer will answer these):

- What is the expected read:write ratio for real-time tracking queries?
  *(Answer: 100:1 — tracking queries dominate)*
- What is the definition of "real-time" for tracking? Sub-second? < 5 minutes?
  *(Answer: < 30 seconds for most events; < 5 seconds for customs clearance
  events because freight is time-sensitive)*
- Who are the "stakeholders" for notifications? Shippers only? Customs agents?
  Third-party logistics partners?
  *(Answer: all three. Plus carrier partners (FedEx, DHL, Maersk) who each
  have their own notification requirements)*
- What is the consistency requirement for event ingestion? Can two events for
  the same container arrive out of order?
  *(Answer: events arrive from 240 countries with unpredictable latency.
  Out-of-order events are common. The system must handle event ordering.)*
- What is the volume spike pattern? Is 18M/day uniformly distributed?
  *(Answer: no — massive spikes at "port arrival" events when container ships
  dock. A single container ship can carry 20,000 containers. All scan at once.)*

---

**30 minutes in — Devraj Iyer asks his follow-ups**

After you've sketched the high-level architecture, Devraj pushes on each
component:

**Devraj Iyer**: You've described a Kafka ingestion layer for the 18M events.
What is your partition key for the event topic, and why?

**Devraj Iyer**: You said "real-time tracking via Redis." A container's location
is a point-in-time snapshot — where it is NOW. But a shipper might also want
to see the full event history — everywhere the container has been. Is that
one query or two? How does your data model support both?

**Devraj Iyer**: You mentioned notifications. How many notification types exist,
and do they all have the same delivery SLA? A "delivery confirmed" notification
to a retail shipper is different from a "customs hold" notification to a
customs agent. Walk me through the priority model.

**Devraj Iyer**: Document generation — customs forms must be generated within
minutes of a customs event, in the local regulatory format for the destination
country. There are 240 countries, each with different form requirements.
How does your document service know which format to generate?

---

**The unknown person speaks — 90 minutes in**

The person in the back has been silent for 90 minutes. They speak once:

"Your event ordering design assumes events can be sorted by timestamp.
But timestamps from 240 countries are produced by 240 different systems
with 240 different clock synchronizations. A container scanned in Lagos at
14:00 UTC and a container scanned in Shanghai at 14:00 UTC — those clocks
may disagree by seconds or minutes. Your sort key is unreliable.
How do you reconstruct the correct event sequence for a container
that touched 8 countries?"

And then they return to silence.

*(Later you find out: the unknown person is a former distributed systems
architect from Amazon who now advises Stratum's board.)*

---

### What You Must Design

This is an open-ended synthesis exercise. There are no prescribed deliverables —
you must choose the appropriate level of depth for each component. The
interviewers are looking for:

- **Clarifying questions first**: never design before you understand the scope
- **Prioritization**: you have 3 hours. Not everything can be designed in depth.
  Explicitly say what you are and are not designing.
- **Known patterns applied correctly**: you have seen all the patterns
  needed for this problem in Phase 1. Apply them.
- **Tradeoffs, not answers**: for every design decision, articulate the
  alternative and why you chose your approach.
- **Clock skew is not a trick question**: it's a real distributed systems
  problem with real solutions. Think about vector clocks, Lamport timestamps,
  or logical ordering approaches.

### The Required Deliverables

Unlike regular chapters, you choose the depth for each. But you must produce:

- [ ] **Clarifying questions list** — before designing anything: list the
  questions you would ask and the answers you require before designing

- [ ] **System architecture diagram** (Mermaid) — the high-level components.
  At minimum: ingestion layer, event store, tracking query layer, notification
  system, document generation

- [ ] **Event data model** — what fields does a shipment event have?
  What is the partition key for Kafka? What is the primary key in the event store?

- [ ] **Event ordering design** — how do you reconstruct the correct sequence
  of events for a container that touched 8 countries with unsynchronized clocks?
  This is the "clock skew problem." Show your approach.

- [ ] **Real-time tracking query design** — how does a shipper query "where is
  my container right now"? What data store? What is the data model?
  Distinguish from the full event history query.

- [ ] **Notification system design** — priority tiers, delivery channels,
  at-least-once delivery. Which patterns from Phase 1 apply directly here?

- [ ] **Scaling bottleneck analysis** — the port arrival spike: 20,000 containers
  scan simultaneously. Which component is the bottleneck? How do you handle it?

- [ ] **One explicit tradeoff** — identify the single most important architectural
  tradeoff in this system and argue both sides.

---

### Scoring Rubric (for self-evaluation)

After you complete the design, evaluate yourself against these criteria:

| Criterion | Met? | Notes |
|-----------|------|-------|
| Asked clarifying questions before designing | | |
| Chose a Kafka partition key with reasoning | | |
| Distinguished tracking query from history query | | |
| Addressed clock skew (Lamport timestamps or equivalent) | | |
| Priority tiers in notification system | | |
| Identified port arrival as the scaling bottleneck | | |
| Named at least one pattern from Phase 1 explicitly | | |
| Stated what you did NOT design and why | | |

**Strong Senior gate**: you must address clock skew. An answer that ignores
it scores below the gate. Clock skew in distributed systems is not a trick —
it's the event ordering problem you've seen at every company that processes
time-ordered events.

---

### Post-Interview Note

Whether or not you get the offer, the unknown advisor's question is the
question you'll carry into Phase 2. Clock skew — events from distributed
sources with unsynchronized clocks — is the beginning of a much deeper
distributed systems problem.

Phase 2 starts there.

---

### Diagram Format (starter — extend significantly)

```mermaid
graph TB
  subgraph "Event Ingestion"
    SCANNERS[240 countries\nContainer scanners] --> KAFKA[Kafka\nshipment-events]
  end
  subgraph "Event Processing"
    KAFKA --> ORDER[Event ordering\nservice]
    ORDER --> STORE[(Event store\nTimeseries DB)]
    ORDER --> TRACK_CACHE[(Redis\nCurrent location)]
  end
  subgraph "Query Layer"
    TRACK_CACHE --> TRACK_API[Tracking API\n/shipment/{id}/location]
    STORE --> HIST_API[History API\n/shipment/{id}/events]
  end
  subgraph "Downstream"
    ORDER --> NOTIF[Notification\nsystem]
    ORDER --> DOC[Document\ngeneration]
  end
```
