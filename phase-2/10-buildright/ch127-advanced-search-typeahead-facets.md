---
**Title**: BuildRight — Chapter 127: Advanced Search — Material Catalog and Vendor Discovery
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #search #elasticsearch #faceted-search #geospatial #typeahead #multilingual
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 33 (PulseCommerce Elasticsearch), Ch. 45 (NeuroLearn academic search), Ch. 60 (SkyRoute flight search)
**Exercise Type**: System Design
---

### Story Context

The thread has 47 replies. Kenji Mori pinned it.

---

**#platform-feedback** *(Slack)*

**Tomás Herrera** *(GC, Mexico City office)*: Searched "varilla corrugada" and got zero results. Had to type "rebar" and got 800 results. I know two languages. The search does not.

**Björn Lindqvist** *(Site Engineer, Stockholm)*: I searched "HEA 200" and got results for "HEA 20" and "HEA 2000" but not "HEA 200". These are different beams. Very different. In a building.

**Rachel Okonkwo** *(Procurement, Lagos)*: Can someone explain why searching "fire-rated insulation" returns "fire extinguisher" and "rated R movies"? I'm serious. Screenshots available.

**Pradeep Nair** *(PM, Bengaluru)*: I searched "steel I-beam" and got "steel wool" and "I-beam theorem". I assumed the theorem result was a joke. It was not.

**Kenji Mori**: Okay. I've seen enough. Adding this to the Q2 roadmap.

**Tomás Herrera**: Q2? We're purchasing materials now. For buildings. Today.

**Kenji Mori**: ...Q1.

---

Kenji drops into your office the next morning with a printed roadmap and the expression of a man who has been apologizing for a search engine for two years.

**Kenji**: "I have a six-month roadmap. The business problem is simple: contractors can't find materials or vendors reliably, so they go off-platform and we lose the transaction. We lose the margin. We lose the data. Every off-platform purchase is a hole in our analytics."

**You**: "What's the current architecture?"

**Kenji**: "PostgreSQL full-text search. `tsvector`. Added by an intern in 2021. Never updated."

**You**: "And the catalog size?"

**Kenji**: "2.4 million SKUs. Growing by about 40,000 per month as vendors onboard. 18,000 vendor profiles across 60 countries."

**You**: "The 'I-beam theorem' result — how does that happen?"

**Kenji**: "The `tsvector` tokenizer splits 'I-beam' into 'I' and 'beam.' Then it matches anything with 'I' in it, ranked by TF-IDF. 'I-beam theorem' has both tokens. High frequency of 'I.' It wins."

**You**: "And 'varilla corrugada'?"

**Kenji**: "Catalog is English-only. Spanish terms return nothing. We have 8,000 contractors in Latin America who purchase materials using Spanish-language standards. They've adapted by memorizing English SKU codes, which is absurd."

**You**: "What standards do I need to know?"

**Kenji**: "DIN is German. ASTM is American. EN is European. A 'HEA 200' is a DIN structural steel section — the EN equivalent is 'HE 200 A.' They're the same beam. We list it as both in some records, neither in others. It's a mess."

He slides a spreadsheet across the desk. Synonym pairs: 832 rows. Vendor-submitted, inconsistent, partially duplicated.

**Kenji**: "The other problem is vendor discovery. A contractor in Nairobi needs a certified concrete supplier within 80 kilometers of the project site. There's no way to do that search today. They go to Google. They leave us."

---

**#search-rebuild** *(Slack — three days later, after you've done your research)*

**You**: "Liam — does the material catalog have normalized category taxonomy or is it flat?"

**Liam Kowalski**: "Flat-ish. There's a `category` string column. Values include: 'Steel', 'steel', 'STEEL', 'Steel Products', 'Structural Steel', 'Structural Steel Products'. No normalization. Vendor-entered."

**You**: "Material certifications?"

**Liam Kowalski**: "Free-text field. Some vendors write 'ISO 9001', some write 'ISO9001', some write 'certified ISO 9001:2015 (pending renewal)'. Parsing that reliably is... a project."

**You**: "Vendor coordinates?"

**Liam Kowalski**: "Address string. Not geocoded. About 30% have a `lat`/`lng` column populated — added last year, never backfilled."

You write a three-line summary to Kenji:

**You [DM to Kenji]**: "The search problem is actually three problems: (1) query understanding — synonyms, multilingual, standard codes; (2) catalog quality — taxonomy normalization, certification parsing; (3) vendor discovery — geospatial, filter-by-certification. I can solve 1 and 3 in Q1. We need a data quality sprint for 2 or the search results will be correct but incomplete."

**Kenji**: "What does 'correct but incomplete' mean for a contractor trying to source rebar in Lagos?"

**You**: "It means they find the vendors who filled in their data correctly. Maybe 40% of them."

**Kenji**: "That's better than zero. Do it."

---

### Problem Statement

BuildRight's material catalog (2.4M SKUs) and vendor directory (18,000 vendors across 60 countries) are powered by PostgreSQL full-text search added in 2021 and never updated. The system fails to handle multilingual queries, synonym resolution (trade names vs standards names), engineering code lookups (DIN/ASTM/EN), and geospatial vendor discovery. The result is contractors leaving the platform to source materials elsewhere, costing BuildRight transaction margin and supply chain visibility.

The new search system must support typeahead autocomplete, faceted navigation, multilingual queries, synonym handling, geospatial radius search for vendors, and certification-based filtering — at a catalog scale of 2.4M SKUs with sub-200ms typeahead response time.

---

### Explicit Requirements

1. Typeahead autocomplete for material and vendor names: P99 < 200ms, triggered at 2+ characters.
2. Synonym handling: "rebar" = "reinforcing bar" = "varilla corrugada" (ES) = "armature" (FR). Bidirectional synonym expansion.
3. Engineering standard code resolution: "HEA 200" (DIN) must match "HE 200 A" (EN) and related ASTM equivalents. Store a standards cross-reference table.
4. Faceted navigation: material type, certification (ISO, ASTM, CE Mark), price range, lead time, country of origin.
5. Geospatial vendor search: "find certified concrete suppliers within X km of project site [lat, lng]."
6. Multi-language support: query in Spanish, French, German must return results. Display names in query language where available.
7. Relevance ranking: exact code match > standard-equivalent match > synonym match > partial match. Boosting by vendor rating and order volume.
8. CDC sync: material catalog in PostgreSQL must sync to search index within 60 seconds of any change.

---

### Hidden Requirements

- **Hint**: Re-read Liam's message about certification data: "ISO9001", "ISO 9001", "certified ISO 9001:2015 (pending renewal)". Contractors filter by certification. If certification data is inconsistent, a facet filter for "ISO 9001" returns only the records that happen to use that exact string. What does your schema design for certifications look like, and how do you handle the normalization problem at index time?
- **Hint**: Re-read the conversation about the 30% geocoding coverage. Kenji says "correct but incomplete" means 40% of vendors. A geospatial search that only searches geocoded vendors silently returns incomplete results — the contractor doesn't know what's missing. How does your search result metadata communicate coverage confidence?
- **Hint**: Re-read Tomás Herrera's message and Kenji's note about 8,000 Latin American contractors. The synonym file is 832 rows, vendor-submitted, inconsistent. Synonyms are a living dataset — vendors want to add their trade names. Who owns the synonym dictionary? How is it updated without requiring a full re-index?
- **Hint**: Re-read Björn Lindqvist's message carefully: "HEA 200", "HEA 20", "HEA 2000" — different products that look similar in text search. A typo-tolerant fuzzy search that helps with "reinforcng bar" will hurt with structural codes where one digit difference is a different product. How do you apply fuzzy matching selectively?

---

### Constraints

- **Catalog size**: 2.4M material SKUs, 18,000 vendor profiles. Growth: 40,000 SKUs/month.
- **Query volume**: 85,000 search queries/day. Peak: 12,000/hour during morning procurement windows.
- **Typeahead SLA**: P99 < 200ms for autocomplete.
- **Full-text search SLA**: P99 < 800ms for full search with facets.
- **Geospatial SLA**: P99 < 1 second for radius search with filters.
- **CDC sync lag**: < 60 seconds from PostgreSQL update to search index.
- **Languages**: English (primary), Spanish, French, German, Portuguese, Arabic.
- **Standards databases**: DIN (German), ASTM (American), EN (European) — cross-reference table ~50,000 entries.
- **Infrastructure**: Elasticsearch (managed, AWS OpenSearch). Budget: up to $4,000/month for search infrastructure.
- **Team**: 2 engineers + you for the search rebuild.

---

### Your Task

Design the search platform for BuildRight's material catalog and vendor discovery. Address the query understanding pipeline (synonyms, multilingual, code resolution), the Elasticsearch index design (mappings, analyzers, boosting), geospatial vendor search, the CDC sync pipeline from PostgreSQL, and the typeahead autocomplete architecture.

---

### Deliverables

- [ ] **Mermaid architecture diagram** — show query path (client → typeahead service → Elasticsearch), CDC sync pipeline (PostgreSQL → Debezium → Kafka → indexer), and standards resolution service
- [ ] **Elasticsearch index mapping** — define the `materials` index with field types (`keyword` vs `text` vs `dense_vector`), custom analyzers (synonym filter, multilingual), and the `vendors` index with `geo_point` field; show the `certifications` normalized sub-document structure
- [ ] **Scaling estimation** — calculate: index size for 2.4M SKUs at 5 KB/doc; Elasticsearch shard strategy (target shard size 30–50 GB); peak QPS vs instance sizing; CDC lag math at 40,000 new SKUs/month
- [ ] **Tradeoff analysis** — minimum 3: (1) fuzzy matching globally vs selectively by field type, (2) synonym expansion at query time vs index time, (3) geospatial accuracy (bounding box vs radius) vs query performance
- [ ] **Synonym management design** — how synonyms are stored, updated, versioned, and applied without full re-index; who can submit new synonyms and what the approval workflow looks like
- [ ] **Standards cross-reference schema** — `material_standards` table linking DIN/ASTM/EN codes with bidirectional lookup; describe how this integrates with Elasticsearch query expansion
