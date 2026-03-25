---
**Title**: HarvestAI — Chapter 223: When the Earth Won't Fit in a B-Tree
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #geospatial #postgis #spatial-indexing #caching #performance
**Estimated Time**: 2.5 hours
**Related Chapters**: Ch. 209 (DeepOcean ocean data mesh), Ch. 33 (PulseCommerce product search)
**Exercise Type**: Performance Optimization
---

### Story Context

**#eng-platform — Slack thread, last Tuesday, 2:14 PM PST**

---

**Raj Patel** [2:14 PM]
okay something is very wrong
the drought alert query has been running for 45 seconds on prod
farmers in the Central Valley are getting timeouts on the dashboard
this is the query: `SELECT field_id, avg_moisture, geometry FROM soil_readings WHERE avg_moisture < 0.22 AND recorded_at > NOW() - INTERVAL '72 hours'`
with a ST_Within filter on the geometry column
PostGIS is just... sitting there

**Carlos Mendes** [2:16 PM]
@raj how many rows are we scanning?
I can log in and check

**Raj Patel** [2:17 PM]
EXPLAIN ANALYZE shows 9.8M rows sequential scan
it decided NOT to use the GIST index
I don't understand why

**Carlos Mendes** [2:18 PM]
ok so the planner gave up on the spatial index because the moisture filter is too selective
it thinks a seq scan is cheaper because 22% moisture is below average for half of California right now
the drought is so bad that the index selectivity estimate is wrong

**Dr. Aisha Kamara** [2:21 PM]
This is actually a known issue with PostGIS and highly correlated spatial+attribute data
when there's a drought event, moisture values cluster spatially — wet fields are spatially contiguous, dry fields are spatially contiguous
the query planner doesn't know that
it treats the spatial filter and the moisture filter as statistically independent
they are very much not independent right now

**Raj Patel** [2:23 PM]
so what do we do, tell farmers to wait 45 seconds?
we have 300 concurrent requests right now
the DB CPU is at 94%

**Priya Sundaram** [2:24 PM]
everyone calm down
Raj: put the drought alert endpoint behind a 5-minute cache immediately, it's a read-only dashboard view
Carlos: can you add a BRIN index on recorded_at in the next 10 minutes?
Aisha: start writing up why the GIST index isn't being used — I want to understand this properly
I'm going to find someone who can fix this properly

**Raj Patel** [2:25 PM]
cache is up, timeouts stopped
but the underlying query is still broken — next drought alert will blow through the cache

**Carlos Mendes** [2:27 PM]
BRIN on recorded_at is done
running ANALYZE now
...actually this made it slightly worse? the planner now tries a BRIN scan + spatial filter and that's even slower than the seq scan

**Dr. Aisha Kamara** [2:30 PM]
@priya the real problem is architectural
we're storing 10M geospatial records as a flat table
the right approach for this kind of query is tile-based decomposition
you partition the geography into spatial tiles — say, H3 hexagons at resolution 7 —
each tile is a precomputed aggregate
the drought alert query becomes: "which tiles have avg_moisture below threshold in the last 72h"
that's a lookup on a ~50K row materialized view instead of a 10M row scan

**Raj Patel** [2:32 PM]
how long would that take to build?

**Dr. Aisha Kamara** [2:33 PM]
the tile precomputation? a few days of engineering. maintaining it in real-time is the hard part.

**Priya Sundaram** [2:35 PM]
ok. I have a new Staff Engineer starting Monday.
this is their first project.
@raj keep the cache in place. we'll do a proper design next week.

---

**Direct Message — Priya Sundaram → You, Monday 9:03 AM**

**Priya Sundaram** [9:03 AM]
welcome to HarvestAI
your first task is in #eng-platform from last Tuesday
you'll find the Slack thread. read it all.
the TL;DR: our geospatial query architecture is broken at scale
we need a proper design before the next drought alert — which could be any day
we have 50M acres, 10M geospatial records in a flat PostGIS table, and a query planner that lies to us under load
I need an architecture, not a patch
come find me at 2pm

---

You spend the morning reading the thread, pulling the EXPLAIN ANALYZE output Raj saved, and looking at the data model. The `soil_readings` table has a GiST index on the `geometry` column and a standard B-tree on `recorded_at`. There is no partitioning. The table is 180GB on disk and growing at roughly 800MB per day during sensor-active months. The geometry column stores point geometries (individual sensor locations). The `fields` table stores polygon geometries for field boundaries — 400,000 fields across 8,000 farm operators. The drought alert query does a `ST_Within(soil_reading.geometry, field.geometry)` join, which means for every drought threshold check, the database is computing spatial containment between 10M points and 400K polygons.

By 11 AM you have a hypothesis. By 2 PM you have a design.

---

### Problem Statement

HarvestAI's geospatial query infrastructure is hitting a fundamental scaling wall. The current architecture stores 10 million sensor readings as point geometries in a flat PostgreSQL/PostGIS table, with 400,000 farm field boundaries as separate polygon geometries. Aggregation queries — particularly the drought alert query that asks "which fields have soil moisture below threshold in the last N hours" — require expensive spatial joins between points and polygons, and the PostgreSQL query planner cannot effectively leverage GiST spatial indexes when attribute filters (moisture values) are spatially correlated with the geometry distribution.

The system must serve a dashboard that 8,000 farm operators can query simultaneously during a drought event, with sub-5-second response times. The current P99 latency under concurrent drought-alert load is 45+ seconds. The system must also support more complex spatial queries: convex hull aggregations for zone-based irrigation planning, nearest-neighbor lookups for cross-field comparison, and time-series rollups per field. A purely index-tuning approach will not solve this — the data model itself must evolve.

---

### Explicit Requirements

1. Drought alert queries ("show all fields with moisture below X in the last 72 hours") must return in under 5 seconds at P99 under 300 concurrent users
2. The system must support spatial aggregation at multiple granularities: per-sensor, per-field, per-farm, per-county, per-state
3. Field boundary polygons (400,000 total) must be queryable with spatial predicates (contains, overlaps, intersects)
4. Time-series queries on soil readings must support 1-hour, 24-hour, 72-hour, 7-day, and 30-day windows
5. The system must handle new sensor readings at 150,000 inserts/hour during growing season
6. Spatial queries must degrade gracefully (not fail) under peak load; a stale result is acceptable for non-urgent queries if clearly flagged
7. The system must support multi-tenant data isolation: each farm operator's data must be queryable only by authorized users

---

### Hidden Requirements

1. **Hint: re-read Dr. Aisha's message about correlated spatial data.** She is not just describing a query planner bug — she is describing the core assumption violation in the entire data model. When physical phenomena (drought) cause attribute values (moisture) to be spatially autocorrelated, what does this imply about how query plans should be built? What does it imply about index design?

2. **Hint: re-read Raj's line "BRIN on recorded_at is done... this made it slightly worse."** Why would a BRIN index on a time column hurt a spatially-filtered query? What does this tell you about the order in which data is physically stored on disk? What indexing strategy addresses both the spatial and temporal dimensions simultaneously?

3. **Hint: re-read Priya's mitigation: "put the drought alert endpoint behind a 5-minute cache."** A 5-minute stale result is acceptable for a dashboard. What does this imply about the consistency model you need for the precomputed tile layer? Does the tile computation need to be synchronous with sensor ingestion, or can it be eventually consistent with a bounded lag?

4. **Hint: re-read the data volumes.** 150,000 inserts/hour = 2,500 inserts/minute = ~42 inserts/second in growing season, but this is not uniform — sensor flush cycles batch readings at the top of every hour. What does a burst of 150,000 inserts in 2 minutes do to a live materialized view refresh?

---

### Constraints

- **Data volume**: 10M soil reading records today; growing at ~800MB/day (sensor-active months); projected 50M records within 18 months
- **Farm operators**: 8,000 tenants; 400,000 field polygons; 50M acres under management
- **Sensor density**: ~25 sensors per farm on average; some large farms have 500+ sensors
- **Query concurrency**: up to 300 simultaneous dashboard queries during drought alert events
- **Ingestion rate**: 150,000 sensor reading inserts/hour during growing season; bursty at hour boundaries (top-of-hour flush: ~150K inserts in ~2 minutes)
- **Latency SLA**: drought alert dashboard: P99 < 5 seconds; time-series chart: P99 < 3 seconds; field overview: P99 < 2 seconds
- **Staleness tolerance**: drought alert: 5-minute stale data acceptable (dashboard, not actioning system); per-sensor real-time feed: < 30 seconds staleness
- **Infrastructure budget**: $8,000/month for database infrastructure during peak season; $2,500/month off-peak
- **Team size**: 1 infrastructure engineer (Raj), you as Staff Engineer for design
- **Existing stack**: PostgreSQL 15 with PostGIS 3.3, Redis 7, Node.js/TypeScript API layer

---

### Your Task

Design a geospatial query architecture that eliminates the spatial join performance problem, supports multi-granularity aggregation, handles the burst-ingestion pattern, and remains cost-effective across HarvestAI's dramatic seasonal load variation.

Your design must address: the spatial data model (how readings, fields, and aggregates are stored), the indexing strategy (beyond a single GiST index), the tile-based precomputation layer (what tiles, what granularity, how maintained), the caching strategy (what is cached, at what layer, with what invalidation), and the query routing layer (how the API decides whether to hit cache, materialized view, or live table).

---

### Deliverables

- [ ] **Mermaid architecture diagram**: full data flow from sensor ingestion → PostGIS → tile computation → cache → API → dashboard
- [ ] **Database schema**: `soil_readings`, `fields`, `spatial_tiles`, `tile_aggregates` tables with full column types, indexes (GiST, BRIN, composite), and partitioning strategy
- [ ] **Scaling estimation** (show math step by step):
  - Storage: 10M records × average row size → project to 50M over 18 months
  - Tile count: H3 resolution 7 over 50M acres → how many tiles? average readings per tile?
  - Materialized view refresh: 150K inserts in 2 minutes → how often can you safely refresh?
  - Query performance: tile lookup vs full scan — show the row count reduction
- [ ] **Tradeoff analysis** (minimum 3):
  - H3 hexagonal tiles vs quadtree/geohash vs native PostGIS partitioning
  - Synchronous vs async tile aggregation refresh (consistency vs write amplification)
  - Caching at the API layer vs caching at the DB layer (materialized views) vs both
- [ ] **Index design document**: explain why GiST alone fails under correlated data, and design a multi-index strategy (GiST + BRIN + partial indexes) with rationale
- [ ] **Query routing pseudocode**: TypeScript interface for the query router that selects between cache / materialized view / live table based on query age, staleness tolerance, and load signal

### Diagram Format
All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
