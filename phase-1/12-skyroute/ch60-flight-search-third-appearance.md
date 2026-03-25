---
**Title**: SkyRoute — Chapter 60: Flight Search — The Hardest Search Problem
**Level**: Strong Senior
**Difficulty**: 9
**Tags**: #search #elasticsearch #multi-criteria #real-time-inventory #third-appearance
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 33, Ch. 45, Ch. 59
**Exercise Type**: System Design
---

### Story Context

**Week 2 — Search team onboarding**

Elena Vasquez, the tech lead for search, walks you through SkyRoute's search
architecture. Her tone is careful — the kind of careful that means she's
showing you something she's embarrassed about.

**Elena Vasquez**: Flight search is not like product search. At PulseCommerce
or NeuroLearn, when a user searches, the inventory doesn't change while they're
looking at results. We have 45 seconds of stale results? That's fine.

**You**: But for flights, seat inventory changes while passengers are looking
at results.

**Elena Vasquez**: Exactly. There's a concept called "phantom availability."
A search returns 3 seats available on the 9:15 AM flight. The passenger
spends 2 minutes choosing. They click "select." The seats were taken 90 seconds
ago. The booking fails. We send them back to search. They're angry.

**You**: What's our current phantom availability rate?

**Elena Vasquez**: 7%. That means 7% of flight selections end in a booking
failure because the seat was taken between search and selection. In 2024, we
had 180 million search → selection flows. That's 12.6 million phantom failures.

**You**: 12.6 million frustrated passengers.

**Elena Vasquez**: And rebooking calls to airline partners. Each rebooking
call costs $4-12 in support overhead. At 12.6 million... it's not small.

**You**: How is the search index built?

**Elena Vasquez**: We use Elasticsearch. Flight index is rebuilt nightly.
Seat availability is updated via a sync job that runs every 15 minutes.
So when you search for flights, you're seeing seat availability that is
at most 15 minutes old.

**You**: For a popular 6 AM Monday flight from JFK to LAX, how fast does
seat inventory move in the hour before departure?

**Elena Vasquez**: [pulls up a chart] Typically: 15 seats move per minute
in the final hour. So 15 minutes × 15 seats/minute = 225 seats could be
sold between your search result and your selection. That's a lot of phantom.

**You**: And your search result says "3 seats available" based on a
15-minute-old snapshot.

**Elena Vasquez**: Yes.

---

**Slack thread — #search-redesign, Wednesday**

```
You: The phantom availability problem has two components. One is Elasticsearch
  index freshness. One is the 2-minute window between search and seat selection
  where seats can move. Both need separate solutions.

You: For index freshness: we need real-time seat inventory updates pushed to
  Elasticsearch when a seat is booked or released. Not 15-minute batch sync.
  Seats should update in < 10 seconds.

Elena Vasquez: The current Elasticsearch index has 24,000 flights × average
  seat count 185 = 4.4 million seat records. Each seat has: flight_id,
  seat_number, class, status (available/reserved/sold), last_updated.
  If we update in real-time, we're pushing updates to ES every time a seat
  is booked — at peak, 2,400 bookings/minute.

You: 40 updates per second to Elasticsearch. That's very manageable.

Elena Vasquez: But there's a second problem you're not seeing yet. We have
  multi-leg searches. "I want to fly from SFO to LHR on March 14." That
  might involve SFO-JFK leg (flight AA1234) and JFK-LHR leg (AA101).
  BOTH legs must have availability for the itinerary to be offered.
  An itinerary search is a multi-constraint query.

You: How many legs in a typical itinerary?

Elena Vasquez: 1-4. But for international: usually 2. For complex routing:
  up to 4 with different airline partners.

Tariq Nassar [joining the thread]: And the itinerary must be "jointly available."
  It's not enough that leg 1 has seats and leg 2 has seats. The legs must have
  seats that connect correctly: legal connection time (minimum 45-90 minutes),
  same terminal (or we offer the airport transit warning), baggage compatibility.

You: So we're not searching individual flights. We're searching graphs of flights
  with constraints.

Elena Vasquez: Exactly. And the seat inventory of each leg must be real-time.
  If a connecting flight at JFK just sold its last seat in your target fare class,
  the itinerary is no longer available.

You: This is not a standard Elasticsearch full-text search problem.
  It's a constrained graph search with real-time state.
```

---

**Slack DM — Marcus Webb → You, Thursday**

**Marcus Webb**
Third search system. Let me give you the arc.

First time (PulseCommerce, Ch. 33): product catalog search.
Inverted index for text matching. Relevance scoring. Merchant scoping.
Sync from Postgres via CDC → Kafka → Elasticsearch.

Second time (NeuroLearn, Ch. 45): academic document search.
Institution scoping, language filter, academic level boost.
Text relevance + behavioral signals.

Both times: the data didn't change while the user was looking at it.
You could afford stale index. Consistency requirement: eventual.

This time: flight seat inventory.
The data changes every second. Consistency requirement: near-real-time.
The search result is a contract — if we show "3 seats available," we're
implying the user can book those seats.

Two distinct problems:

1. Index freshness: how does seat status flow from the booking system to
   Elasticsearch in < 10 seconds? CDC → Kafka → Elasticsearch works,
   same pattern as before. The new constraint is that each seat update
   must also invalidate cached search results for affected itineraries.

2. The phantom availability problem: even with a 10-second index, there's
   still a window between search and selection where seats can be sold.
   The fix is not a faster index — it's a "soft hold" mechanism.
   When a user selects a flight, place a 10-minute soft hold on the seats.
   The hold reserves the seats for that user's session without requiring
   full payment. On booking attempt: convert the hold. On session expiry:
   release the hold. This reduces phantom rate from 7% to near 0.

The soft hold is a distributed lock with TTL. You've done this before.
Think about what happens when the user abandons their session without
explicitly cancelling — the hold must auto-release.

---

### Problem Statement

SkyRoute's flight search has a 7% phantom availability rate — search results
show seats that are no longer available by the time a passenger completes
booking — costing an estimated $50-150M annually in failed bookings and
support overhead. The root causes are: 15-minute Elasticsearch index sync
lag, no real-time seat inventory updates, and no session-level seat hold
mechanism. Additionally, multi-leg itinerary search requires constrained
graph traversal across partner airline data, which the current architecture
does not efficiently support.

### Explicit Requirements

1. Seat inventory in Elasticsearch must update within 10 seconds of a booking
2. Multi-leg itinerary search must return only jointly-available itineraries
   (both legs available, legal connection time, same fare class)
3. When a user selects a flight itinerary, place a 10-minute soft hold on
   the seats to prevent phantom bookings
4. Soft holds must auto-release on session expiry (no manual cleanup required)
5. The search index must support filtering by: origin, destination, date,
   airline, fare class, stops, and price range
6. Search P99 latency for a standard one-leg query: < 200ms

### Hidden Requirements

- **Hint**: Elena said "24,000 flights × 185 seats = 4.4 million seat records."
  Soft holds add a new state: "held (session_id, expires_at)." When a user
  searches and a seat is "held" (not "sold"), should search results show
  that seat as available or unavailable? If "held" seats are shown as unavailable,
  a seat that is held and then released won't appear in search results during
  the hold period — even though it's technically obtainable. If "held" seats
  are shown as available, multiple users might attempt to book the same seat.
  Which is the right behavior, and what does it mean for the search index?

- **Hint**: Multi-leg searches are "constrained graph searches." For a
  SFO→LHR query, Elasticsearch doesn't natively support graph traversal.
  The search system must either: (a) issue multiple Elasticsearch queries
  and join them in application code, or (b) denormalize itineraries into
  the index (pre-compute all valid SFO→LHR connections and index them).
  Both approaches have tradeoffs. What is the index size for pre-computed
  itineraries? How many valid SFO→LHR combinations exist?

- **Hint**: "Legal connection time (minimum 45-90 minutes)." This depends
  on the airports involved. A connection at a hub like JFK (single terminal)
  might be 45 minutes. A connection that requires changing terminals (JFK
  Terminal 4 to Terminal 8) might require 90 minutes. SkyRoute's partners
  have different connection time policies. Where is this data stored?
  How does the search system use it at query time?

### Constraints

- **Flight inventory**: 24,000 flights/day × 185 avg seats = 4.4M seat records
- **Booking rate**: ~2,400 bookings/minute peak = 40 seat status changes/second
- **Search volume**: 5M search queries/day = ~58/second; peak 400/second
- **One-leg P99 latency SLA**: < 200ms
- **Multi-leg P99 latency SLA**: < 500ms (additional join cost accepted)
- **Soft hold TTL**: 10 minutes
- **Phantom availability target**: < 0.5% (down from 7%)
- **Elasticsearch index refresh interval**: currently 1 minute; target 10 seconds

### Your Task

Design the real-time flight search architecture: index freshness pipeline,
multi-leg itinerary query strategy, and soft-hold seat reservation mechanism.

### Deliverables

- [ ] **Real-time index update pipeline** (Mermaid) — how seat status changes
  flow from the booking service via CDC → Kafka → Elasticsearch. What is
  the Kafka topic? What does the Elasticsearch consumer update? How is the
  index refresh triggered?

- [ ] **Seat document schema** — the Elasticsearch document for a single
  seat. Include: seat_id, flight_id, seat_number, class, status
  (available/held/sold), hold_session_id, hold_expires_at, price, last_updated.
  What is the index mapping for status filtering?

- [ ] **Multi-leg query design** — for a SFO→LHR search, define the query
  strategy. Application-side join or denormalized itinerary index?
  Show the query plan (pseudo-Elasticsearch DSL) for a two-leg international
  search with connection time constraint.

- [ ] **Soft hold design** — when a user selects a flight, how is the hold
  placed? Show the Redis data structure for hold tracking (or Elasticsearch
  update?). What is the TTL mechanism? How does the hold auto-release?
  What happens if two users simultaneously try to hold the same seat?

- [ ] **Cache invalidation for search results** — if a seat's status changes,
  cached search results that include that flight are now stale. What is the
  cache invalidation strategy? (Tag-based invalidation, TTL, or no caching?)

- [ ] **Phantom rate estimation** — with 10-second index freshness + 10-minute
  soft hold, what is the residual phantom rate? Show the math.

- [ ] **Tradeoff analysis** — minimum 3 tradeoffs:
  1. Pre-computed itinerary index (faster multi-leg query, stale combinatorial data)
     vs application-side join (fresh, slower)
  2. Soft hold as Elasticsearch update vs Redis distributed lock
  3. Real-time index refresh (10s) vs event-driven update on booking (< 1s)

### Diagram Format

```mermaid
graph TB
  BOOKING[Seat booked\nor released] --> CDC[Debezium CDC\nseats table]
  CDC --> KAFKA[Kafka\nseat-status-updates]
  KAFKA --> ES_CONSUMER[Elasticsearch\nindex updater]
  ES_CONSUMER --> ES[(Elasticsearch\nFlight index)]

  USER[Passenger\nsearches] --> SEARCH_API[Search API]
  SEARCH_API --> ES
  ES --> RESULTS[Search results\n"3 seats available"]

  USER --> SELECT[Passenger\nselects flight]
  SELECT --> HOLD_SVC[Soft hold\nservice]
  HOLD_SVC --> REDIS[(Redis\nhold:seat_id → session)]
  HOLD_SVC --> ES_UPDATE[Update seat status\nheld in ES]

  REDIS -->|TTL expires| RELEASE[Auto-release\nheld seat]
  RELEASE --> ES_RELEASE[Restore status\navailable in ES]
```
