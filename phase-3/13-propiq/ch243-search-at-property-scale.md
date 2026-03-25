---
**Title**: PropIQ — Chapter 243: The Garbage Search
**Level**: Staff
**Difficulty**: 8
**Tags**: #search #elasticsearch #faceted-search #geospatial #relevance-tuning #multi-tenant #per-tenant-ranking
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 239 (PropIQ search baseline), Ch. 171 (LexCore RAG search), Ch. 33 (PulseCommerce Elasticsearch)
**Exercise Type**: System Design
---

### Story Context

**Email — PropIQ**

**From:** Leila Ahmadi <leila.ahmadi@propiq.com>
**To:** you <you@propiq.com>, Divya Nair <divya.nair@propiq.com>
**CC:** Marcus Tran <marcus.tran@propiq.com>
**Subject:** Urgent — Westfield Capital search feedback (they're threatening to cancel)
**Date:** Thursday, 8:49 AM

Team,

I just got off a 90-minute call with Westfield Capital's Head of Data, Derek Shen. They're one of our largest institutional clients — 50 analysts, $20M AUM in commercial real estate research. They've been on PropIQ for 8 months.

Derek says their analysts describe our search as "worse than Google Maps." He shared a recording of one of their analyst sessions (with consent). I watched it. It was painful.

Here's what the analyst was trying to find:

**Query:** "Office buildings greater than 50,000 square feet in Austin, TX, built after 2010, Class A, within 1 mile of a light rail station, occupancy rate above 85%, with at least 3 recent comparable sales in the last 18 months."

**What happened:**
1. The search bar returned 1,247 results for "Office buildings Austin TX" — no filters applied
2. The analyst then tried to use the filter panel to narrow down
3. Adding the "Class A" filter returned a different set of properties that didn't all match the free-text query
4. The geospatial filter ("within 1 mile of light rail") was missing entirely
5. The occupancy filter exists but doesn't filter — it only sorts
6. The "recent comparable sales" filter doesn't exist at all
7. The analyst gave up after 20 minutes and exported everything to Excel

Derek's exact words: "Your search returns results based on keyword matching. Our analysts think in investment criteria, not keywords. If you can't build an investment-grade search, we'll go back to CoStar."

There's one more thing Derek mentioned that I want to flag. He was careful about it — he said Westfield has "a proprietary cap rate analysis model" they use to evaluate properties. He's interested in whether we could offer "custom relevance tuning" that weights results by their model's output. But — and he was very explicit — he said "we would need absolute assurance that PropIQ cannot see our weighting formula. It's a competitive advantage we've spent 5 years building."

I told him I'd have a design proposal in 10 days.

Please tell me we can build this.

Leila

---

**DM: you → Marcus Tran**
_Thursday, 9:22 AM_

**you:** Marcus — before the search design session, can you walk me through what we're actually running under the hood? I want to understand the current Elasticsearch setup.

**Marcus Tran:** Sure. We have a single Elasticsearch 7.x cluster on AWS OpenSearch. One index: `properties`. About 4.2 million documents (all commercial properties we've ingested). Index was designed in 2020 and hasn't been redesigned since.

**you:** What does the mapping look like?

**Marcus Tran:** (shares screen) Here's the relevant part:
```json
{
  "mappings": {
    "properties": {
      "property_id": { "type": "keyword" },
      "name": { "type": "text" },
      "address": { "type": "text" },
      "city": { "type": "keyword" },
      "state": { "type": "keyword" },
      "sqft": { "type": "long" },
      "class": { "type": "keyword" },
      "year_built": { "type": "integer" },
      "occupancy_rate": { "type": "float" },
      "description": { "type": "text" },
      "tenant_id": { "type": "keyword" }
    }
  }
}
```

**you:** No geospatial fields?

**Marcus Tran:** Nope. No `geo_point`. No light rail proximity. We don't even store coordinates in Elasticsearch — they're in PostGIS.

**you:** Recent comparable sales?

**Marcus Tran:** Stored in a separate PostgreSQL table. Not indexed in Elasticsearch at all. The filter the analyst was trying to use — it doesn't work because it's a post-query filter that runs against PostgreSQL after Elasticsearch returns results. It's broken and nobody fixed it.

**you:** What about per-tenant relevance tuning?

**Marcus Tran:** We use the default BM25 scoring. All tenants get the same ranking. There's no concept of per-tenant relevance weights anywhere in the system.

**you:** And if Westfield wants custom weights that we can't see?

**Marcus Tran:** (long pause) I don't even know what that means architecturally. Can you even do that?

**you:** It means their weighting formula has to run somewhere we can't inspect. Either client-side re-ranking, or they provide an opaque scoring function we execute without seeing the coefficients. Let me think about this.

---

**PropIQ Conference Room A — Thursday 10:15 AM**

You're at the whiteboard with Marcus. You've sketched the current architecture:

```
Search Request
    → API Gateway
    → Search Service (Node.js)
    → OpenSearch query (BM25, single index)
    → Return top 10 results
    → [BROKEN] Post-query PostgreSQL filter for comps
    → Response
```

**you:** "The core problem is that Westfield's queries are investment criteria, not text queries. They're combining four different query types: structured filters (sqft, class, year_built, occupancy), geospatial proximity (distance from light rail), time-series conditions (comparable sales in last 18 months), and text search (maybe). Our current system can do exactly one of those well — text search."

**Marcus:** "So we need to handle all four in a single query."

**you:** "In a single Elasticsearch query, if possible. Elasticsearch supports geo queries, range queries, and term filters natively. The comparable sales count is the hard one — that requires a join with our comps data."

**Marcus:** "Could we denormalize the comps count into the property document? Like, `recent_comps_count_18mo` as a field?"

**you:** "Yes. Calculated at ingest time, updated when new comps arrive via CDC. That's the right move. It turns a join into a filter."

**Marcus:** "And light rail proximity?"

**you:** "We pre-compute a `near_light_rail_1mi` boolean at ingest time from PostGIS, OR we add a `location` geo_point field and let Elasticsearch do the proximity calculation. The geo_point approach is more flexible — users might want 0.5 miles tomorrow."

**Marcus:** "Okay. Now — the Westfield custom relevance problem. Derek doesn't want us to see their weights. How do you build that?"

**you:** "There are three approaches. One: full client-side re-ranking. We return the full result set in a neutral order, they apply their weights in their browser or analytics tool. Problem: we'd need to return thousands of results, not ten. Two: they provide a scoring function as code — a JS function we execute server-side. We can't read the function easily but we do execute it, so there's an argument we have access to it. Three: differential privacy approach — they give us a weight vector encrypted with their key, we apply it in a TEE (trusted execution environment). That's enterprise-grade but probably overkill for them."

**Marcus:** "What's the practical answer for a 10-day timeline?"

**you:** "Probably option one with a twist. We return a signed, encrypted result set that includes our proprietary scoring signals. They apply their model client-side. The key insight is: we never see their weights, and they never see our signals in plaintext. We define a result envelope format."

**Marcus:** (slowly) "You're describing a search API where the last mile of ranking happens outside PropIQ."

**you:** "Exactly. And for most clients, that last mile is just 'sort by price.' For Westfield, it's their $5M proprietary model. The architecture has to support both."

---

### Problem Statement

PropIQ's Elasticsearch-based property search cannot handle investment-grade queries that combine structured filters (building class, square footage, year built), geospatial proximity (distance from transit infrastructure), time-series conditions (recent comparable sales), and text search in a single query. The current system returns keyword matches with no faceted filtering, no geospatial support, and no relevance tuning.

Redesign PropIQ's search stack to support complex multi-criteria property queries at scale (4.2M documents, 50 concurrent analyst sessions, sub-2-second query SLA) while enabling per-tenant custom relevance tuning that protects the client's proprietary weighting formula from PropIQ inspection.

### Explicit Requirements

1. **Multi-criteria query**: Support combined structured filters + geospatial proximity + time-series conditions in a single query
2. **Geospatial search**: "Within X miles of [point of interest]" — support at least light rail stations, highways, airports, CBD center
3. **Faceted filtering**: All facets (class, city, state, year built range, sqft range, occupancy range, property type) must be true filters (affect result count), not sort-only
4. **Comparable sales filter**: "At least N comparable sales in last M months" — must work as a hard filter, not a post-query step
5. **Aggregations**: Return facet counts with results (e.g., "of 1,247 matching properties: 312 Class A, 580 Class B, 355 Class C")
6. **Relevance tuning**: Default relevance for all clients; custom relevance for enterprise clients who provide weight vectors
7. **Tenant-scoped search**: Each PropIQ client sees only their licensed properties (not all 4.2M — access is controlled by data licensing)
8. **Per-tenant custom ranking privacy**: Enterprise clients can provide a scoring function or weight vector that PropIQ executes but cannot inspect
9. **Query performance**: p95 < 2 seconds for complex multi-criteria queries against 4.2M documents
10. **Index freshness**: Property updates from ingestion pipeline reflected in search within 15 minutes (Tier 1 providers)
11. **Search-as-you-type**: Typeahead suggestions for address and property name fields (p99 < 200ms)

### Hidden Requirements

1. **Hint**: Leila's email quotes Derek Shen saying Westfield has "a proprietary cap rate analysis model they don't want us to know about" and requires "absolute assurance that PropIQ cannot see their weighting formula." This is not just a feature request — it's a contractual requirement. If PropIQ logs all search queries and parameters (which it should for observability), those logs could inadvertently capture Westfield's weights if weights are passed as query parameters. The architecture must ensure custom weights never appear in PropIQ's observability systems. Design the weight handling so that weights are encrypted in transit, not logged, and not accessible to PropIQ engineers.

2. **Hint**: Marcus mentions 50 concurrent analyst sessions at Westfield. But PropIQ has 2,000 client firms. If even 5% run concurrent searches at peak, that's 100 firms × multiple analysts each. The current single OpenSearch cluster has no read replica strategy. At what point does a single cluster become a bottleneck? Design for 500 concurrent search sessions.

3. **Hint**: The comparable sales denormalization (pre-computing `recent_comps_count_18mo`) sounds clean — but comparable sales are time-dependent. A property with 3 comps "in the last 18 months" on January 1st has 0 comps "in the last 18 months" by June 19th of the following year if no new comps arrived. A pre-computed static field will go stale. How do you keep a count-over-a-rolling-window field accurate without recomputing it every hour for 4.2M properties?

### Constraints

- **Document count**: 4.2M commercial properties in the index
- **Active tenant count**: 2,000 firms; Westfield Capital has 50 concurrent analysts (largest tenant)
- **Concurrent search sessions (peak)**: 500 simultaneous queries
- **Query p95 SLA**: < 2 seconds for multi-criteria queries
- **Typeahead p99 SLA**: < 200ms
- **Index update latency**: Tier 1 (webhook) data must appear in search within 15 minutes
- **Document size average**: ~8 KB per property document (after denormalization of comps count, geo_point, etc.)
- **OpenSearch cluster current**: 3-node cluster, r6g.2xlarge (64 GB RAM, 500 GB SSD each)
- **Points of interest dataset**: ~50,000 light rail stations, highway interchanges, airports, CBD centers in the US — needs to be loaded and geo-indexed
- **Comparable sales table**: ~2.8M comps records in PostgreSQL; updated ~8,000/day
- **Budget**: $3,000/month incremental for search infrastructure
- **Team**: you + Marcus Tran + 1 frontend engineer; 3-week implementation window

### Your Task

Design the PropIQ investment-grade search system. This includes:

1. The revised Elasticsearch index mapping with geo_point, denormalized fields, and proper type assignments
2. The query composition strategy for multi-criteria queries (how do you combine bool/filter/geo queries?)
3. The comparable sales rolling-window denormalization strategy (how do you keep it fresh without full recomputation?)
4. The per-tenant custom relevance architecture (how does Westfield's proprietary model participate without PropIQ seeing the weights?)
5. The typeahead/search-as-you-type implementation
6. The index update pipeline from ingestion to searchable

### Deliverables

- [ ] **Mermaid architecture diagram**: End-to-end search system (query path, index update path, custom relevance flow)
- [ ] **Elasticsearch index mapping**: Complete revised mapping with geo_point, nested fields for comps, tenant access control, and all required filter fields
- [ ] **Example query DSL**: Write the Elasticsearch query DSL for the Westfield analyst's query: "Office buildings >50K sqft in Austin TX, Class A, built after 2010, within 1 mile of light rail, occupancy >85%, at least 3 recent comps in 18 months"
- [ ] **Comparable sales freshness design**: How to keep `recent_comps_count_18mo` accurate for 4.2M properties without hourly full recomputation
- [ ] **Custom relevance privacy architecture**: How Westfield's weight vector is used without PropIQ being able to log or inspect it
- [ ] **Scaling estimation**: Show math for index size, query throughput, concurrent session headroom
- [ ] **Tradeoff analysis**: Minimum 3 explicit tradeoffs

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

---

#### Scaling Estimation (Starter Math — Complete This)

**Index size:**
- 4.2M property documents × 8 KB average = 33.6 GB of document data
- Plus inverted index overhead (typically 3-5× document size for text fields): ~150 GB per shard
- With 1 primary + 1 replica per shard: ~300 GB per shard group
- Recommendation: 6 primary shards = ~50 GB per shard (well within 50 GB/shard recommendation)
- Total index size on disk: ~300 GB (primary + replica)
- Current cluster: 3 nodes × 500 GB = 1.5 TB available → fits easily

**Query throughput:**
- 500 concurrent sessions × 1 query per 30 seconds (analyst browsing pattern) = ~16.7 QPS average
- Peak burst: 500 concurrent × 1 query per 5 seconds = 100 QPS
- OpenSearch r6g.2xlarge handles ~200 QPS for complex bool queries per node
- 3 nodes → ~600 QPS capacity vs. 100 QPS peak demand → comfortable headroom

**Typeahead throughput:**
- 50 concurrent typeahead users (subset of search sessions) × 5 keystrokes per second = 250 QPS
- Typeahead queries are much lighter (prefix queries on `address.keyword`, not multi-criteria)
- Separate typeahead index (address + property name only, 2 fields, no geo) → 10× faster than full index
- Alternative: Redis sorted sets for top-N autocomplete by city → sub-5ms, off-loads Elasticsearch

**Index update latency:**
- Tier 1 ingest: webhook → SQS → Lambda → Elasticsearch index → 15-minute SLA
- Elasticsearch index refresh: default 1 second (documents searchable within 1s of indexing)
- Bottleneck: Lambda → Elasticsearch bulk API. Bulk batch 1,000 documents per call → latency dominated by SQS polling interval (configurable: 20-second default, reduce to 1 second for Tier 1 queue)
- Comparable sales CDC: PostgreSQL comps insert → Debezium CDC → Kafka → Lambda consumer → Elasticsearch update of affected property's `recent_comps_count_18mo` field → 5-minute end-to-end

**Comparable sales rolling window freshness:**
- 4.2M properties × 1 update per rolling-window expiry = problem
- Rolling window: "last 18 months" — a comp expires from the count exactly 18 months after its `sale_date`
- Events to handle: (1) new comp inserted → increment count for affected property; (2) comp ages out of 18-month window → decrement count
- Solution A: daily batch job recalculates count for all properties with a comp that aged out today → max 1 day staleness on rolling window edge; ~8,000 updates/day (number of comps that age out daily)
- Solution B: comps aging is deterministic — schedule a Lambda to decrement exactly 18 months after each comp's sale_date via EventBridge delayed event
- Cost of Solution B: 8,000 delayed EventBridge events/day × $1/million events = $0.008/day → negligible

---

#### Custom Relevance Privacy Design (Starter — Complete This)

Westfield Capital's requirement: they provide a scoring function (or weight vector over PropIQ's scoring signals), PropIQ executes it, PropIQ cannot inspect the weights.

**Option A: Client-side re-ranking**
```
PropIQ returns: top 500 results with scoring signals in encrypted envelope
  { properties: [...], scoring_signals: ENCRYPT(propiq_key, { location_score, condition_score, comps_score, ... }) }
Westfield decrypts with their key: applies their model → re-sorts locally
PropIQ logs: query parameters only, no weights (weights never transmitted to PropIQ)
Problem: PropIQ must return 500+ results for client-side re-ranking to be meaningful. Large payload.
```

**Option B: Opaque weight vector, server-side**
```
Westfield registers a weight vector: ENCRYPT(propiq_public_key, { w1: 0.4, w2: 0.2, ... })
PropIQ stores encrypted vector per tenant — cannot decrypt without Westfield's private key
At query time: PropIQ uses a TEE (Nitro Enclave on AWS) to decrypt + apply weights
TEE output: sorted result list — scores never leave the enclave
Logs: TEE logs only final ranking order, not weights
Problem: TEE adds 50-200ms latency; complex to operate; overkill for most clients
```

**Option C: Weight vector as opaque scoring plugin**
```
Westfield provides weights as a signed, versioned payload (JWT-signed with their key)
PropIQ's search service passes the JWT to an Elasticsearch custom scoring script
Script receives the JWT as a parameter: scores it using a generic dot-product formula
PropIQ engineers can read the script code but not the weight values inside the JWT
Logs: log the JWT token, not the decoded weights
Problem: JWT can be decoded by anyone with base64 — not cryptographically private
True privacy requires encryption, not just signing
```

Which option do you recommend? What's the minimum viable privacy guarantee for Westfield's use case? What would you build in 10 days vs. in 3 months?
