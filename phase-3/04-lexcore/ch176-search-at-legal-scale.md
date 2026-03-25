---
**Title**: LexCore — Chapter 176: Find the Precedent
**Level**: Staff
**Difficulty**: 8
**Tags**: #search #legal #Elasticsearch #RAG #multi-tenant #citation-graph #compliance
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 171, Ch. 172, Ch. 175
**Exercise Type**: System Design
---

### Story Context

The deal room RFC is approved. The chunking pipeline is in testing. The UK residency design is in legal review. For the first time in weeks, you have a morning with no active incidents.

You use it to sit in on a work session with Marcus Delgado, a fourth-year associate at Thornhill & Osei LLP, one of LexCore's oldest pilot customers. Marcus has agreed to let you observe how he actually uses the platform. He's researching a securities class action and needs to find precedent cases where courts dismissed similar claims at the pleading stage.

You watch for forty-five minutes.

**Marcus Delgado [working]:** *"Okay, I'm going to search for cases where Rule 10b-5 claims were dismissed for failure to plead scienter. I know there are at least twenty cases like this from the Second Circuit in the last ten years. Let me try LexCore."*

He types the query. The search returns 847 results. The top result is a 2019 securities decision from the Ninth Circuit. The second result is a law review article. The third result is a deposition transcript from an unrelated case.

**Delgado:** "This is terrible. The Ninth Circuit decision doesn't help me — my case is in the Second Circuit. The law review article is not precedent. The deposition is completely irrelevant."

He switches to Westlaw. He types the same query. Within three seconds, the top results are Second Circuit decisions, ranked by a combination of recency, citation count, and binding authority level. The sixth result is exactly the case he was looking for.

**Delgado:** "LexCore has the same documents. I know because I helped upload them. But Westlaw knows that a Second Circuit decision *binds* my case and a Ninth Circuit decision doesn't. LexCore doesn't know that."

You take notes.

After the session, you write three observations on a whiteboard:

1. **Jurisdiction matters**: a case from the Second Circuit is worth more to a Second Circuit practitioner than a Ninth Circuit case, regardless of semantic similarity.
2. **Citation count matters**: a case cited 400 times is more authoritative than a case cited twice, regardless of the text.
3. **Authority level matters**: a Supreme Court decision > Circuit Court of Appeals > District Court > state court, within applicable jurisdictions. This is a structured ranking signal that is not in the document text.

These are not signals that embedding similarity captures. They are structured metadata signals that require a citation graph and a jurisdiction taxonomy layered on top of the vector search.

You also note a fourth observation — one that Delgado didn't say but that you inferred from watching him: he filtered out law review articles immediately. LexCore returns articles, treatises, deposition transcripts, and case law in the same result set with no type-aware ranking. Legal professionals have strong opinions about source type relevance depending on what they're trying to do.

Sasha Petrenko reviews your notes. "How do we build Westlaw?"

You say: "We don't build Westlaw. We build something that works better for the specific workflows our customers are doing in LexCore. Which means knowing the workflows."

---

### Problem Statement

LexCore's search experience ranks results by embedding vector similarity, with no awareness of jurisdiction binding authority, citation count, court level, source type, or recency. For legal research — the core use case — these structured signals are more important than semantic similarity for determining document relevance and utility. A practitioner searching for Second Circuit precedent should not receive Ninth Circuit decisions as top results.

You must design a hybrid search architecture that combines: dense retrieval (vector similarity), structured ranking signals (jurisdiction, citation count, court hierarchy, source type, recency), and per-firm tenant isolation (from Ch. 171). The system must handle 500M documents across 50,000 attorneys with < 500ms p95 latency.

---

### Explicit Requirements

1. Hybrid retrieval: combine dense vector retrieval (semantic similarity) with BM25/sparse retrieval (keyword matching) and re-rank with structured signals
2. Jurisdiction taxonomy: model the US court system hierarchy (SCOTUS > Circuit CoA > District Court) and state court equivalents; apply binding authority scoring based on user's case jurisdiction
3. Citation graph: track citation relationships between cases; use incoming citation count as an authority signal
4. Source type ranking: differentiate cases, statutes, regulations, law review articles, treatises, deposition transcripts — apply source type filters and default preferences per query type
5. Recency signal: apply configurable decay function; recent decisions may override older precedent in evolving areas
6. Per-firm tenant isolation (from Ch. 171): search must only return documents within the user's authorized matters
7. Per-firm private documents: firm-uploaded documents must be ranked alongside public legal corpus but kept isolated (firm A cannot see firm B's private documents)
8. Scale: 500M documents total (public legal corpus + firm-private documents), 50,000 attorneys, < 500ms p95

---

### Hidden Requirements

- **Hint**: Re-read Delgado's observation: "Westlaw knows that a Second Circuit decision *binds* my case." Binding authority is not just about the court level — it depends on the *user's case jurisdiction*. A user searching from a Second Circuit matter should see Second Circuit decisions boosted. This requires knowing the user's active matter's jurisdiction. What does this imply about the search query context — is jurisdiction a user preference, or is it derived from the active matter?
- **Hint**: Re-read your observation about private firm documents. A firm uploads a legal memo analyzing a specific issue — that memo is more relevant to that firm's work than a public law review article. But ranking private firm documents requires knowing the user's authorization (matter access from Ch. 171). What happens if the citation graph includes a private document? Could citation graph traversal leak the existence of a private document?
- **Hint**: Re-read the source type observation: Delgado "filtered out law review articles immediately" for this specific task, but law review articles are essential for other tasks (academic analysis, policy arguments). What does this imply about the search UX and the ranking model — should source type be a hard filter (user-controlled) or a soft signal (system-applied)?
- **Hint**: The citation graph requires knowing which documents cite which other documents. For existing documents, this can be computed from the text (cases cite other cases by name and reporter). For newly uploaded documents (firm-private PDFs), how does the citation graph stay current? What is the latency between upload and citation graph inclusion?

---

### Constraints

- **Corpus**: 500M documents total — 300M public (case law, statutes, regulations) + 200M firm-private
- **Firm-private**: 10M documents today (from Ch. 171), growing ~100K/month
- **Attorneys**: 50,000; peak concurrent search: ~5,000
- **Search RPS**: 5,000 concurrent × 2 queries/min = ~170 RPS
- **Latency SLA**: < 500ms p95
- **Citation graph size**: ~500M nodes, estimated ~2B citation edges (dense for major cases)
- **Jurisdiction taxonomy**: ~200 federal courts + 50 state supreme courts + ~1,000 state lower courts
- **Index freshness**: new documents added to search index within 30 minutes of ingestion
- **Budget signal**: Elasticsearch cluster at this scale costs $40K-$80K/month; vector search adds cost
- **Team**: Dmitri + Yasmine + 1 ML engineer

---

### Your Task

Design the hybrid legal search architecture. Produce the ranking model design, citation graph data structure, jurisdiction taxonomy model, per-tenant isolation at query time, and index architecture.

---

### Deliverables

- [ ] Mermaid architecture diagram: query flow → dense retrieval (vector) → sparse retrieval (BM25) → re-ranking service (citation, jurisdiction, source type, recency) → tenant filter → result presentation
- [ ] Database schema:
  - `document` (updated with: jurisdiction_code, court_level, citation_count, source_type, published_date)
  - `citation_edge` (citing_document_id, cited_document_id)
  - `jurisdiction` (jurisdiction_code, court_name, level, binding_for: array of jurisdiction codes)
  - `matter_jurisdiction` (matter_id → primary jurisdiction code)
- [ ] Scaling estimation:
  - Citation graph: 500M nodes × avg 4 outbound citations = 2B edges; storage at 24 bytes/edge = X GB
  - Search query: 170 RPS × 500ms budget = X concurrent queries; Elasticsearch node sizing
  - Re-ranking: 170 RPS × top-50 candidates × re-rank latency budget = X ms available per document
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3):
  - Dense vector retrieval vs. BM25 for legal search (semantic recall vs. keyword precision for citation names)
  - Real-time citation graph updates vs. batch updates (freshness vs. write amplification)
  - Per-jurisdiction index vs. unified filtered index (isolation quality vs. infrastructure complexity)
- [ ] Cost modeling: Elasticsearch cluster, vector store, citation graph database ($X/month at 500M documents)
- [ ] Capacity planning: 12-month horizon (corpus growth, firm-private document growth, citation graph growth)
- [ ] Re-ranking model: describe the scoring formula combining vector similarity, BM25, citation count, jurisdiction binding authority, source type weight, and recency decay

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
