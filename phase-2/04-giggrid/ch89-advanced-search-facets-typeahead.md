---
**Title**: GigGrid — Chapter 89: Advanced Search — Facets and Typeahead
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #search #elasticsearch #facets #typeahead #geosearch #relevance
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 115, Ch. 127
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**#product — GigGrid**
`Tuesday 11:30` **kenji.mori**: I need to share something from user research. I've been collecting feedback for 6 months and I'm finally putting it in front of engineering.
`11:31` **kenji.mori**: worker quote #1: "I searched for 'security guard London' and got results for 'security camera installation Brighton.'"
`11:32` **kenji.mori**: worker quote #2: "I have a forklift license and I keep seeing warehouse jobs that say 'forklift experience required' but I can't filter by license type."
`11:33` **kenji.mori**: worker quote #3: "Why does the search show jobs 80 miles away when I'm searching in Manchester? Is it broken?"
`11:34` **kenji.mori**: worker quote #4 (my personal favorite): "I've given up on search. I just scroll the list."
`11:35` **amara.chen**: @you — how is this not already fixed?
`11:36` **you**: because "search is someone else's problem" is a disease that infects every engineering team
`11:37` **kai.moreau**: I built the current search in 2021. it's a LIKE query on the job title and description. with an ORDER BY created_at. there's a JIRA ticket to "improve search" that's been in the backlog since 2022.
`11:38` **you**: LIKE queries. For job search. For 2 million workers.
`11:39` **kai.moreau**: it worked when we had 50,000 workers
`11:40` **kenji.mori**: I've had 6 months of this data and zero movement. I'm putting it in front of the CTO today.
`11:41` **amara.chen**: @you — this is yours. I want a design by end of week and a working prototype in 3 weeks.

**1:1 — You and Kenji Mori**
*GigGrid Offices, 2:00 PM*

**Kenji**: What can you actually build in 3 weeks?

**You**: The current LIKE search is fast to replace. Elasticsearch with proper job listing indexing can be running in a week. The harder parts are: faceted navigation (filter by skill, location, pay rate, license type), search-as-you-type (show suggestions as workers type), and geo-distance scoring (prefer nearby jobs).

**Kenji**: Walk me through faceted navigation. Workers filter by skill. How does that work technically?

**You**: Facets are pre-computed counts by category. "Security guard: 847 jobs in London this week." "Forklift operator: 312 jobs." These counts need to update in near-real-time as jobs are posted and filled. Elasticsearch aggregations handle this natively.

**Kenji**: The geo part — can you show jobs within 15 miles and rank by distance?

**You**: Yes. Elasticsearch has geo_point field types and geo-distance scoring. You score jobs by (relevance × distance_decay_factor). A perfect match 50 miles away scores lower than a good match 3 miles away.

**Kenji**: One thing I care about: a worker who's certified as a "senior security officer" shouldn't miss a job posted as "senior security guard" because the employer used different terminology. Can Elasticsearch handle synonyms?

**You**: Yes, with a synonym token filter. You define equivalences: "security guard" = "security officer" = "protection officer." The index expands all of these at query time.

**Kenji**: How long until workers see something better?

**You**: Two weeks to first version. Search-as-you-type and full facets in three.

---

### Problem Statement

GigGrid's current job search uses a SQL LIKE query with ORDER BY created_at. It returns irrelevant results, has no filtering capability, ignores geographic proximity, and doesn't support role terminology synonyms. With 2M active workers and 50k jobs per day, the search is a primary navigation surface — and it's failing. You need to build an Elasticsearch-powered search system with faceted navigation, typeahead, geo-distance scoring, and synonym handling.

---

### Explicit Requirements

1. Search-as-you-type: suggestions appear within 100ms as workers type
2. Faceted navigation: filter by skill/certification, location (radius), pay rate range, shift type
3. Geo-distance scoring: prefer jobs closer to the worker's registered location
4. Synonym handling: "security guard" = "security officer" = "security operative"
5. Facet counts must update within 5 minutes of job status changes (posted → filled → closed)
6. Search must handle multilingual job titles (GigGrid operates in 30 countries)

---

### Hidden Requirements

- **Hint**: Re-read worker quote #2: "I have a forklift license." Licenses and certifications are structured data, not free text. A "Forklift Operator (RTITB certified)" job requires an RTITB-issued license, not just the words "forklift" appearing in the job description. How does your search design distinguish between a keyword match on "forklift" in the description and a structured license requirement that must be matched to a worker's credential?

- **Hint**: Kenji mentioned "jobs 80 miles away." The current search returns results in creation order. If you switch to geo-distance scoring, what happens to a newly posted job 2 miles from the worker vs a well-matched job posted 6 hours ago 5 miles away? How do you balance recency and relevance in your scoring function?

---

### Constraints

- 50k active jobs at peak, ~500k total in index (historical)
- 2M workers, up to 100k search queries per minute during peak dispatch hours
- 30 countries with language-specific job titles
- Elasticsearch cluster budget: up to 3 nodes, 8GB RAM each
- CDC available from job_postings Postgres table (Debezium already installed)
- Typeahead SLA: < 100ms P99

---

### Your Task

Design the Elasticsearch-based job search system for GigGrid. Include the index schema, analyzer configuration (typeahead + synonyms), geo-distance scoring, and faceted navigation.

---

### Deliverables

- [ ] Elasticsearch index mapping: All fields with their types, analyzer assignments, and index configuration
- [ ] Typeahead configuration: Edge n-gram tokenizer or completion suggester — choose one and justify
- [ ] Synonym token filter: Example synonym list for security roles and warehouse roles; how it integrates with the analyzer chain
- [ ] Geo-distance scoring: Function score query that combines BM25 relevance with geo-distance decay
- [ ] Faceted navigation: Aggregation query design for skill facets, pay rate ranges, geo-bucket facets
- [ ] CDC sync: How job status changes (posted → filled) propagate to Elasticsearch within 5 minutes
- [ ] Scaling estimation: At 100k queries/minute, what's the Elasticsearch cluster size required? Show the math.
- [ ] Tradeoff analysis (minimum 3):
  - Completion suggester vs edge n-gram for typeahead (query latency vs index size)
  - Synonym expansion at index time vs query time
  - Elasticsearch vs Postgres full-text search (tsvector) for this use case at scale
