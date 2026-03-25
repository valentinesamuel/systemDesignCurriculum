---
**Title**: LexCore — Chapter 172: The Chunking Pipeline
**Level**: Staff
**Difficulty**: 7
**Tags**: #RAG #document-processing #chunking #legal-AI #pipeline #embeddings
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 171, Ch. 173, Ch. 175, Ch. 176
**Exercise Type**: System Design
---

### Story Context

The incident report from the first week was about access control. The crisis that arrives in week four is quieter — no angry phone call from a managing partner, just an email. But it's worse.

---

**From:** Theodora Wren \<t.wren@kessler-aldrich.com\>
**To:** Mara Obi \<m.obi@lexcore.ai\>
**CC:** Sasha Petrenko \<s.petrenko@lexcore.ai\>
**Subject:** RE: LexCore pilot — critical issue with contract review feature
**Date:** Tuesday, 09:18

Mara —

Following up on our call yesterday. I want to document this in writing.

Our associate, James Okeke, was using LexCore's contract review feature to summarize a vendor services agreement we're drafting for a client. The AI summary included the following clause in the "key obligations" section:

*"Vendor shall provide a 99.9% uptime guarantee for all services, with penalties of $50,000 per hour of downtime exceeding the threshold."*

James reviewed the summary, found it accurate against his recollection of the document, and the partner signed off on the term sheet based partly on this summary. The actual contract language reads:

*"Vendor shall provide commercially reasonable efforts toward service availability, with any remedies subject to the liability cap in Section 14.4."*

Those are not the same clause. The $50,000 penalty language came from a different contract in our document management system — a prior agreement with a different vendor that was chunked alongside this document in LexCore's processing pipeline, apparently because they were uploaded in the same batch.

The partner is now in a difficult conversation with the client. We have paused using LexCore effective immediately.

Please advise.

Theodora Wren
Senior Associate, Kessler, Aldrich & Partners LLP

---

**[Slack — #incidents — LexCore Internal]**

**Dmitri Zhao [09:31]:** I pulled the ingestion logs. The two contracts were uploaded in the same batch job. The chunking pipeline splits documents into 512-token chunks. The Kessler vendor agreement ends mid-sentence at chunk boundary 47. The next chunk starts with the penalty clause from the prior vendor agreement.

**Dmitri Zhao [09:32]:** The RAG retrieval pulled chunk 47 (which ends with "penalties of") and then chunk 48 from the wrong document — they were adjacent in the vector store because they were processed in the same batch and had similar embedding vectors.

**Sasha Petrenko [09:35]:** How many documents have we processed with this chunking logic?

**Dmitri Zhao [09:36]:** All of them. 2.3 million documents across all pilot firms.

**Sasha Petrenko [09:37]:** [You] — you're the architect on document pipeline. What do we do?

You look at the Slack thread. The problem is not subtle. The current chunking strategy is naive token-count splitting: 512 tokens, slide a window, write chunks to vector store. No semantic awareness. No document boundary enforcement. No preservation of section hierarchy, clause numbering, cross-references, or defined terms. A contract might define "Force Majeure" in Section 1.3 and reference it fourteen times throughout — but the definition and the references are in separate chunks with no semantic link between them.

You walk to the whiteboard. You draw a contract as a hierarchical tree: Document → Sections → Subsections → Clauses → Defined Terms. You draw what the current pipeline does: a horizontal sliding window that cuts across the tree at arbitrary token boundaries.

Then you draw what it should do.

The problem is that legal documents have structure — explicit, predictable structure. Every contract has a table of contents. Section numbers are parseable. Defined terms appear in quotation marks or bold. Cross-references use section numbers ("as defined in Section 4.2(b)"). Exhibit references are structured ("attached hereto as Exhibit A"). A legal-aware chunking strategy must preserve all of this.

This is not just a chunking problem. It's a document understanding problem. And the correction will require reprocessing 2.3 million documents.

---

### Problem Statement

LexCore's RAG pipeline uses naive 512-token sliding-window chunking for all legal documents. This strategy does not respect document structure, causing chunks to split mid-clause and allowing content from adjacent documents (processed in the same batch) to cross-contaminate retrieval results. A production incident resulted in AI-generated contract summaries that hallucinated penalty clauses from unrelated contracts.

You must design a legal-aware document chunking pipeline that preserves: section hierarchy, clause integrity (no mid-clause splits), defined term linkage (definitions and their references must be semantically linked), cross-reference resolution (section references should link chunks to their targets), exhibit references, and document boundary enforcement (no cross-document chunk contamination). The pipeline must process 10 million documents (with reprocessing of the existing 2.3 million).

---

### Explicit Requirements

1. Document boundary enforcement: chunks from different documents must never be adjacent in the vector store; each chunk must carry an immutable document_id and matter_id
2. Section-aware splitting: chunk boundaries must align with legal document structure (section breaks, clause breaks) not arbitrary token counts
3. Defined term preservation: when a term is defined ("'Force Majeure' shall mean..."), that definition chunk must be linked to all chunks containing references to the term
4. Cross-reference resolution: when a chunk references Section 4.2(b), the chunk metadata must include a link to the target section's chunk(s)
5. Exhibit handling: exhibit references must link to the exhibit's own chunks
6. Table of contents parsing: when present, use ToC to initialize the section hierarchy before chunking
7. Pipeline must support: PDF (primary), DOCX, and plain text formats
8. Reprocessing: 2.3 million existing documents must be reprocessed without downtime; new pipeline runs parallel, not replacing existing until validated

---

### Hidden Requirements

- **Hint**: Re-read the Kessler incident: two contracts uploaded in the same batch were cross-contaminated. Batch processing is a root cause. What does this imply about the pipeline's unit of isolation — should the pipeline process documents individually (one message per document in the queue) or in batches? What is the throughput implication?
- **Hint**: Re-read Dmitri's analysis: "adjacent in the vector store because they were processed in the same batch and had similar embedding vectors." Embedding similarity is the retrieval mechanism. Even with correct chunking, two different contracts' clauses about the same topic will have similar embedding vectors. How does matter-level isolation (from Ch. 171) interact with the vector store? Can you solve cross-document contamination at the retrieval layer instead of (or in addition to) the chunking layer?
- **Hint**: Re-read the defined term problem: the RAG system needs to link a defined term's definition to its occurrences across a 200-page contract. What does this imply about the chunk metadata schema? Is a flat chunk record sufficient, or does the chunk need a relationship graph?
- **Hint**: The reprocessing requirement covers 2.3 million documents "without downtime." The existing chunks are in production, serving live queries. How do you run a parallel pipeline and transition atomically without a window where old and new chunks coexist for the same document in the vector store?

---

### Constraints

- **Document volume**: 10 million total (2.3M existing, growing at ~100K/month)
- **Average document size**: 50 pages ≈ 25,000 tokens (legal documents are verbose)
- **Formats**: PDF (70%), DOCX (25%), plain text (5%)
- **Pipeline throughput required**: process 100K new documents/month; reprocess 2.3M documents in < 30 days
- **Chunk size target**: 800-1,200 tokens (section-aligned), with overlap for context window
- **Vector dimensions**: 1,536 (OpenAI text-embedding-3-large equivalent)
- **Retrieval latency**: < 500ms p95 (unchanged from Ch. 171 requirement)
- **Team**: Dmitri Zhao + 1 ML engineer + 1 backend engineer
- **Infrastructure**: existing BullMQ job queue (inherited from prior tech); Pinecone or equivalent vector store

---

### Your Task

Design the legal-aware document chunking pipeline. Produce the parsing architecture, chunk data model with relationship graph, pipeline queue design, vector store isolation strategy, and the reprocessing plan with zero-downtime transition.

---

### Deliverables

- [ ] Mermaid architecture diagram: document ingestion → parser → section tree builder → chunk generator → embedding service → vector store, with reprocessing parallel lane
- [ ] Chunk data model (with column types and indexes):
  - `document`, `section` (hierarchical: parent_section_id), `chunk` (section_id, token_start, token_end, chunk_type: clause/definition/exhibit), `term_definition`, `term_reference`, `cross_reference`
- [ ] Scaling estimation:
  - Reprocessing throughput: 2.3M documents ÷ 30 days = X documents/day = Y documents/hour
  - Chunks per document: 50 pages ÷ 1,000 tokens/chunk average = Z chunks/document
  - Total chunks: 10M documents × Z = total vector store entries
  - Embedding API cost: Z chunks × $X/1K tokens
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3):
  - Section-aligned chunking vs. semantic chunking (rule-based vs. ML-based; accuracy vs. cost)
  - Chunk relationship graph in relational DB vs. embedded metadata in vector store (queryability vs. retrieval performance)
  - Zero-downtime parallel reprocessing vs. maintenance window replacement (operational safety vs. complexity)
- [ ] Cost modeling: embedding API cost for 2.3M reprocessing + ongoing, vector store storage ($X/month)
- [ ] Capacity planning: 12-month horizon (document volume growth, new firm onboardings, vector dimension changes)
- [ ] Zero-downtime reprocessing plan: how do you atomically cut over from old chunks to new chunks for a given document without serving mixed results?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
