---
**Title**: VenueFlow — Chapter 115: Vector Search — Personalized Event Recommendations
**Level**: Staff
**Difficulty**: 8
**Tags**: #search #vector-search #elasticsearch #ml-systems #recommendations #cold-start
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 33 (PulseCommerce Elasticsearch), Ch. 45 (NeuroLearn academic search), Ch. 60 (SkyRoute flight search), Ch. 113 (VenueFlow real-time)
**Exercise Type**: System Design
---

### Story Context

**Email**
**To:** engineering-leads@venueflow.io
**From:** marcus.thompson@venueflow.io (Head of Product)
**Subject:** Fan Discovery — Next Quarter Priority
**Date:** Tuesday, 9:03 AM

Team,

Sharing the latest fan behavior data from our Q3 analysis. The headline number: **71% of fans who buy a ticket discover the event by browsing the venue's upcoming shows page, not by searching for a specific artist or event.** They open the app, scroll, and buy something they didn't know they wanted.

This tells me we have an enormous opportunity we are not capitalizing on. Right now, browsing means "show me everything at MSG sorted by date." That is not discovery. That is a calendar.

I want to show fans events they'd love even if they don't know they love them yet. If someone has bought tickets to three indie folk shows and a jazz festival, we should probably be surfacing the Bon Iver tour before they find it themselves. We should be surfacing it before they even search.

I know this is a non-trivial engineering ask. I'm scheduling time with the arch team next week. Come with opinions.

— Marcus

---

**#eng-architecture — Slack**
**Tuesday, 2:18 PM**

**tomasz.kowalski:** Marcus's email landed. I assume we're talking about recommendation infrastructure.
**[you]:** Yes. You've looked at this before?
**tomasz.kowalski:** Briefly, last year. We did a proof of concept with collaborative filtering — "fans who bought X also bought Y." It worked okay for top-tier events. Completely fell apart for anything niche.
**[you]:** Cold-start problem?
**tomasz.kowalski:** Cold-start, sparse data, and the fact that our event catalog turns over every few weeks. An event that happened last month is dead data. A new tour goes on sale and we have zero purchase signal for it.
**[you]:** That's where vector search changes the picture. You're not matching on purchase overlap. You're matching on semantic similarity — artist genre, event category, venue vibe, previous audience demographic.
**tomasz.kowalski:** We'd need embeddings for events. Where do those come from?
**[you]:** Artist metadata, genre tags, similar-artist graphs, historical audience demographics. You can generate event embeddings at catalog-ingest time. You can generate fan embeddings from purchase + browsing history.

**2:31 PM**
**tomasz.kowalski:** What's the latency profile? Marcus is going to ask.
**[you]:** kNN search in Elasticsearch with dense_vector is fast — sub-100ms for most queries if the index is sized right. The fan embedding update is the interesting part. You don't want to recompute it on every page load.
**tomasz.kowalski:** Batch overnight?
**[you]:** Too stale. A fan who just bought two tickets to a bluegrass festival should see updated recommendations within a few minutes, not the next morning.
**tomasz.kowalski:** Near-real-time embedding updates, then.
**[you]:** Yes. Event-driven off the purchase stream.

---

**1:1 — Zara Ahmed → you — Wednesday, 10:00 AM**

"I want to be direct about something," Zara says. "Marcus thinks this is a product feature. I think it's an infrastructure bet. If we build this right, recommendation quality becomes a competitive moat — ticketing partners can't replicate it without our purchase history. If we build it wrong, we've got a slow search endpoint that occasionally surfaces irrelevant concerts and we've burned three months."

She leans back. "What's the failure mode you're most worried about?"

"Cold start," you say. "New fans with no purchase history get the same calendar-sorted browse experience they have today. And new events with no purchase signal get zero recommendations surface area until they've sold some tickets — which means the first tickets are the hardest to sell."

"How do you solve it?"

"For new fans: content-based recommendations from profile data — location, age cohort, genre preferences from onboarding. For new events: event embeddings from artist metadata, genre graph, and similar-event clustering. You have enough signal from the event itself to make reasonable recommendations before anyone buys a ticket."

Zara nods slowly. "And the hybrid search — you're mixing text search with vector search?"

"BM25 for explicit queries, cosine similarity for discovery. The fan who types 'Taylor Swift' should get Taylor Swift. The fan who opens the app and scrolls should get a ranked list built from their embedding."

---

This is the fourth time in your career you've designed a search system: PulseCommerce's product catalog, NeuroLearn's academic document index, SkyRoute's flight availability search. Each time, the retrieval model was different. This is the first time vector search — not inverted indexes, not full-text BM25, but kNN embedding similarity — is the right answer. The event catalog is semantic. Taste is not a keyword.

### Problem Statement

VenueFlow wants to replace its date-sorted browse experience with a personalized event discovery system that can surface events fans would love before they know to search for them. The system must support hybrid search (explicit text queries + semantic similarity), generate and maintain fan and event embeddings, handle cold-start for new fans and new events, and serve recommendations at under 100ms P99 for 5 million daily active app opens.

### Explicit Requirements

1. Hybrid search: BM25 for explicit keyword queries, kNN cosine similarity for discovery/recommendation
2. Event embeddings generated at catalog-ingest time from artist metadata, genre tags, and historical audience demographics
3. Fan embeddings updated within 5 minutes of a new purchase event
4. Cold-start handling: new fans receive content-based recommendations from onboarding profile; new events receive similarity recommendations from event metadata before purchase signal accumulates
5. Recommendation endpoint: <100ms P99 response time for 5M DAU browse opens
6. Embedding model versioning: ability to roll out a new embedding model without full re-indexing downtime
7. Exclude already-purchased events from recommendations for a given fan

### Hidden Requirements

- **Hint: re-read Tomasz's comment about the event catalog turning over every few weeks.** An event that happened last month is dead data. How does this affect your index lifecycle strategy? What happens to embeddings for past events, and is there any value in retaining them?

- **Hint: re-read Zara's question about competitive moat.** She says recommendation quality becomes a moat "if we build this right." What does it mean to make the recommendation system defensible — specifically, what data assets and feedback loops would a competitor struggle to replicate?

- **Hint: re-read the discussion about near-real-time embedding updates.** A fan who buys two tickets should see updated recommendations within minutes. What does the event-driven embedding update pipeline look like — and what happens to recommendation quality if the embedding update pipeline falls behind?

- **Hint: re-read the cold-start conversation about new events.** "The first tickets are the hardest to sell." Is there a bootstrapping strategy that improves new-event discovery surface area before any tickets are sold — and what data does VenueFlow already have that could seed this?

### Constraints

- **DAU**: 5 million app opens per day (browse context)
- **Event catalog**: ~2 million active events at any time; ~50k new events per day
- **Fan profiles**: 80 million registered fans; ~5M active in any 30-day window
- **Recommendation latency**: <100ms P99
- **Embedding update lag SLA**: <5 minutes after a purchase event
- **Embedding dimensions**: 768-dimensional vectors (standard BERT-family output)
- **Infrastructure**: Elasticsearch cluster (existing); GPU compute for embedding generation (new)
- **Team**: 3 engineers (2 platform, 1 ML infra)
- **Timeline**: 10 weeks to MVP recommendation feature

### Your Task

Design the personalized event discovery system. Define the embedding generation pipeline for events and fans, the hybrid search architecture combining BM25 and kNN, the cold-start strategy, the near-real-time fan embedding update pipeline, and the index lifecycle management strategy for a high-churn event catalog.

### Deliverables

- [ ] Mermaid architecture diagram: event catalog ingest → embedding generation → ES index; fan purchase event → embedding update pipeline → fan embedding store; query path (text vs. discovery) → hybrid ranker → response
- [ ] Database schema for fan embedding store and event embedding store (column types, dimensions, version tracking, indexes)
- [ ] Scaling estimation (show math):
  - Elasticsearch index size: 2M events × 768 dimensions × 4 bytes per float
  - Fan embedding store size: 80M fans × 768 dimensions × 4 bytes
  - kNN query throughput: 5M DAU / (16 hours × 3600 seconds) = queries/sec
  - Embedding update pipeline throughput: purchases per day → events per second
- [ ] Tradeoff analysis (minimum 3):
  - HNSW approximate kNN vs. exact kNN (latency vs. recall)
  - Elasticsearch dense_vector vs. dedicated vector DB (Pinecone, Weaviate) for this use case
  - Batch embedding updates (overnight) vs. near-real-time event-driven updates
- [ ] Cold-start strategy document: separate strategies for (a) new fans, (b) new events, with specific data sources used for each
- [ ] Evolution summary: one paragraph comparing this search design against PulseCommerce (Ch. 33), NeuroLearn (Ch. 45), and SkyRoute (Ch. 60) — what is fundamentally different about vector search vs. inverted-index search

---
