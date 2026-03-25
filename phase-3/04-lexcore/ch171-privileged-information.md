---
**Title**: LexCore — Chapter 171: Privileged Information
**Level**: Staff
**Difficulty**: 8
**Tags**: #access-control #multi-tenancy #compliance #legal #conflict-of-interest #data-isolation
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 172, Ch. 173, Ch. 174
**Exercise Type**: System Design
---

### Story Context

On your third day at LexCore, you read the incident report twice. Cameron Ruiz, a paralegal at Hartwell & Morse, was performing research on a real estate acquisition for a client called Aldgate Properties. The LexCore search returned a result — a due diligence memo — that belonged to a different matter: an M&A transaction where the counterparty to the deal was Aldgate Properties. Same name. Different matter. Different firm's client.

The system had no concept of matter-level isolation. It searched across all indexed documents for all clients of all firms and returned results ranked by relevance. The relevance algorithm did not know — and was not designed to care — which firm's attorney had uploaded which document.

You schedule a meeting with the product team. The room: you, VP Eng Sasha Petrenko, Head of Product Mara Obi, Senior Engineer Dmitri Zhao, and a guest — Priscilla Hayward, a partner from Hartwell & Morse who agreed to come in and explain how law firms actually work. Priscilla is in her fifties, meticulous, and has the manner of someone accustomed to billing at $1,200 per hour.

**Priscilla Hayward:** "The fundamental unit of data organization in a law firm is the 'matter.' A matter is a specific client engagement. I represent Aldgate Properties in a real estate transaction. That's one matter. I also represent Aldgate Properties in a securities dispute. That's a different matter. The documents in those two matters must never commingle — not just for confidentiality, but because the strategy in one matter cannot be visible to attorneys working the other, even at the same firm."

**Mara Obi:** "So it's firm → client → matter as the hierarchy?"

**Hayward:** "Yes. But there's a more complex problem: conflict of interest. If Firm A represents Company X in a transaction, and later Firm B — which has joined LexCore — represents the counterparty to that same transaction, Company Y — those two firms' workspaces must be completely isolated. They are adversarial. If an LexCore engineer can see both sides, or if the platform can surface information from one side to the other, we have an ethical violation and potentially a disqualification motion in court."

**Dmitri Zhao:** "How would we even know when two firms are on opposite sides of the same matter?"

**Hayward:** "You check the parties. Every matter has a client and, in adversarial matters, opposing parties. If Firm A's client appears as an opposing party in Firm B's matter, that's a conflict flag. It's called a conflicts check. Every firm does it before accepting a new client. You should be doing it before letting two firms share a platform."

**Sasha Petrenko:** "We don't currently have opposing party data. We have matter names and client names."

**Hayward:** "Then you have a problem."

After the meeting, you sit with Dmitri and map the data model on a whiteboard. Currently: Firm → Matter → Document. No client entities. No party relationships. No conflict-of-interest metadata. The entire multi-tenant isolation story is a single `firm_id` column in the document table with a `WHERE firm_id = ?` clause in every query. That's it.

You think about RLS (Row-Level Security) from your CloudStack days. You think about the multi-tenant architecture from the NeuroLearn platform. You've done tenant isolation before. But those systems didn't have adversarial tenants — tenants who were specifically prohibited from seeing each other's data not just by access policy but by legal and ethical duty.

You write on the whiteboard: *Matter-level isolation is not a feature. It is the product.*

---

### Problem Statement

LexCore's current data model treats all documents as belonging to a firm, with isolation enforced by a single `firm_id` filter. This is insufficient: legal practice requires matter-level isolation (a paralegal on Matter A cannot see documents from Matter B, even within the same firm), party-level isolation (opposing parties in adversarial matters cannot share a platform without complete data isolation), and conflict-of-interest detection (before a new client is onboarded, the platform must detect if that client is an opposing party in any existing matter on the platform).

You must redesign the data model and access control architecture for LexCore's document platform to enforce all three levels of isolation at query time, with conflict detection as a first-class workflow.

---

### Explicit Requirements

1. Three-tier access hierarchy: Firm → Client → Matter → Document; a user can only see documents within their explicitly authorized matters
2. Matter-level isolation: an attorney authorized on Matter A cannot access documents from Matter B at the same firm (even if they are the same client)
3. Party isolation: opposing parties in adversarial matters must have complete data isolation — no shared search index, no cross-matter result leakage
4. Conflict-of-interest detection: when a new client is onboarded, the system must check whether the client entity (or its principals) appears as an opposing party in any existing matter across all firms on the platform
5. Party data model: matters must track client entity, opposing party entities (including principal names for conflict checks), and matter type (adversarial vs. non-adversarial)
6. Scale: 500 firms, 50,000 attorneys, 10 million documents
7. Performance: matter-authorized document search results in < 500ms p95

---

### Hidden Requirements

- **Hint**: Re-read Hayward's comment about "the strategy in one matter cannot be visible to attorneys working the other, even at the same firm." This is not just about document access — it implies that even the *existence* of Matter B should not be visible to an attorney on Matter A if the matters are strategically sensitive. What does this mean for the search index? Does a universal search index that returns "no results" for a query still leak information (the existence of documents matching the query)?
- **Hint**: Re-read Zhao's question about how to detect when firms are on opposite sides. The conflict check requires party entity resolution — "Aldgate Properties" in Firm A's matter and "Aldgate Properties LLC" in Firm B's matter may be the same entity. What does this imply about the conflict check system's data model? Is string matching sufficient, or is entity resolution required?
- **Hint**: Re-read Petrenko's line: "We don't currently have opposing party data." The conflict check system needs opposing party data that does not yet exist in the platform. Who provides this data, when, and how is it validated? What happens when a firm's intake process doesn't report opposing parties accurately?
- **Hint**: Re-read the access control for matters: "explicitly authorized matters." In a law firm, matter authorization is managed by the responsible partner — not by a system admin. What does this imply about the authorization workflow? Is it self-service (the partner authorizes via the platform UI) or admin-mediated?

---

### Constraints

- **Scale**: 500 firms, 50,000 attorneys, 10 million documents, 200 matters per firm average (100,000 matters total)
- **Read RPS**: estimated 5,000 document queries/second at peak (AmLaw 100 firms × concurrent research sessions)
- **Conflict checks**: estimated 50 new client onboardings/day × conflict check across 100,000 matters = X comparisons
- **Storage**: 10M documents × average 500KB = 5TB document storage; metadata separate from content
- **Latency SLA**: matter-authorized search < 500ms p95; conflict check < 5 seconds
- **Compliance**: ABA Model Rules of Professional Conduct (confidentiality, conflicts), state bar regulations
- **Team**: Dmitri Zhao + 2 backend engineers + you; 6-week timeline before next firm onboarding

---

### Your Task

Design the multi-tenant access control architecture for LexCore. Produce the data model, access enforcement mechanism (at query layer), conflict-of-interest detection system, and matter authorization workflow.

---

### Deliverables

- [ ] Mermaid architecture diagram: data model hierarchy, access enforcement at query layer, conflict detection service, matter authorization workflow
- [ ] Database schema (with column types and indexes):
  - `firm`, `client`, `matter`, `matter_party` (client + opposing parties), `matter_authorization` (user-matter grants), `document`
  - Include indexes for: matter authorization lookup, conflict check query, document search filter
- [ ] Scaling estimation:
  - Conflict check: 50 onboardings/day × entity comparison across 100,000 matters = X operations
  - Authorization lookup per query: 5,000 RPS × authorization cache hit/miss rate
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3):
  - Row-Level Security (database layer) vs. application-layer authorization (performance vs. defense in depth)
  - Per-matter search index vs. filtered universal index (isolation guarantee vs. operational complexity)
  - Self-service matter authorization vs. admin-mediated (usability vs. oversight control)
- [ ] Cost modeling: search infrastructure per-matter isolation, conflict detection service, authorization cache ($X/month)
- [ ] Capacity planning: 12-month horizon (firm growth from 500 to X, document volume growth, conflict check rate)
- [ ] Conflict detection algorithm design: given two entity name strings, propose a resolution strategy that handles common variations ("Aldgate Properties" vs. "Aldgate Properties LLC" vs. "Aldgate Prop.")

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
