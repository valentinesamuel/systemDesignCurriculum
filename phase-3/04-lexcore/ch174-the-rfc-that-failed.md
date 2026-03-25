---
**Title**: LexCore — Chapter 174: The RFC That Failed
**Level**: Staff
**Difficulty**: 9
**Tags**: #RFC #multi-party #data-rooms #encryption #legal #stakeholder-management
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 171, Ch. 173, Ch. 175
**Exercise Type**: System Design / RFC Process (MANDATORY STORY BEAT PART 1)
---

### Story Context

Eight weeks in. The access control architecture from Chapter 171 is implemented. The chunking pipeline from Chapter 172 is in testing. The UK residency design from Chapter 173 is in legal review. Sasha Petrenko tells you there's a new product requirement that needs an RFC before engineering begins.

The requirement: multi-party data rooms. In large M&A transactions, two law firms — representing the buyer and the seller — need to share specific documents during due diligence. Today, firms do this via third-party virtual data rooms (VDRs) like Intralinks or Donnelley. LexCore wants to build this capability natively: a deal room where attorneys from both sides can collaborate on a specific, curated set of documents, with granular permission controls and activity logs. The pitch from sales: if LexCore can replace VDR subscriptions, it's a $3,000-$6,000/deal upsell. AmLaw 100 firms do hundreds of deals per year.

You write the RFC over a weekend. You are thorough. The design: a Deal Room entity linking two or more firm workspaces, with a granular permission matrix (who can view/download/comment on which specific documents), activity logging, watermarking for downloaded documents, and an expiry mechanism when the deal closes. Platform infrastructure mediates all access. Documents are stored encrypted in LexCore's object storage; the platform holds and manages the encryption keys. Access is proxied through the LexCore API.

The RFC review is scheduled for a Thursday afternoon. Present: Sasha Petrenko (VP Eng), Mara Obi (VP Product), Dmitri Zhao (Sr. Engineer), Yasmine Korhonen (Staff Engineer), Elliot Chamberlain (CTO), and — unexpectedly — Rosalind Achebe, General Counsel, who joins by video call five minutes after the meeting starts.

You present for twenty-five minutes. The questions are good. The engineering team pushes on key management and watermarking implementation. Mara Obi asks about the UX for granular permission management. Elliot Chamberlain seems satisfied.

Then Rosalind Achebe unmutes.

**Rosalind Achebe:** "I've been listening carefully. I have a legal concern that I believe is fatal to this design."

The room becomes very still.

**Achebe:** "Under the ABA Model Rules of Professional Conduct — Rule 1.6, confidentiality of client information — an attorney has an obligation to take reasonable measures to prevent unauthorized disclosure of client information. The question I have for this design is: LexCore holds and manages the encryption keys for all deal room documents. That means LexCore, as a platform, can access privileged communications between attorneys and clients."

**Sasha Petrenko:** "We have strict access controls internally—"

**Achebe:** "That's not the point. The point is capability, not intent. If LexCore is technically capable of reading those documents — because LexCore holds the keys — then attorneys using LexCore for deal rooms are entrusting privileged client communications to a third party that can access them. Under Rule 1.6, that's a disclosure. The fact that LexCore *doesn't* access them is a policy question, not an architectural one. And policy can change. Companies get acquired. Employees go rogue. Subpoenas happen."

Silence.

**Achebe:** "There's also a litigation hold concern. If LexCore gets subpoenaed in a case involving a deal room, and LexCore holds the keys, we are potentially custodians of privileged communications that we then have to produce. That's a nightmare for every law firm using the platform and for us."

**Elliot Chamberlain:** "What would satisfy you architecturally?"

**Achebe:** "A design where LexCore is technically incapable of reading the content. Not just contractually prohibited. Technically incapable."

She pauses.

**Achebe:** "I'm rejecting this RFC. I want it redesigned. You have two weeks."

The meeting ends. The room empties. Dmitri Zhao leans over to you in the hallway and says, almost in a whisper: "I was reading about this last year. Check how Zoom handled legal holds during COVID — there was a similar architecture debate. Client-side encryption was the answer."

You take the train home and open a browser. Client-side encryption. Zero-knowledge. The platform holds no keys. You start reading.

---

### Problem Statement

LexCore's proposed multi-party deal room architecture uses platform-managed encryption keys stored in LexCore infrastructure. General Counsel rejected this RFC on two grounds: (1) platform key access constitutes a potential disclosure of privileged communications under ABA Rule 1.6, and (2) LexCore's key custody makes it a potential custodian of privileged communications in litigation, creating subpoena exposure.

This chapter is RFC Version 1 — the rejected design. Your task is to fully specify the rejected architecture, clearly document the legal objection, and articulate why this design fails the "technically incapable of access" requirement. Understanding precisely why a design fails is as important as knowing the replacement.

---

### Explicit Requirements (for the REJECTED design)

1. Deal Room entity: links two or more firm workspaces for a specific matter/transaction
2. Granular permission matrix: per-document, per-user permissions (view/download/comment)
3. Activity logging: all document access events, download events, and permission changes
4. Watermarking: downloaded documents carry watermarks identifying the accessing user and timestamp
5. Document encryption: all deal room documents encrypted at rest
6. Key management: encryption keys managed by LexCore platform infrastructure (this is the fatal flaw)
7. Expiry mechanism: deal room expires when transaction closes; documents no longer accessible
8. API-proxied access: all document access goes through LexCore API (not direct storage access)

---

### Hidden Requirements

- **Hint**: Re-read Achebe's subpoena scenario. If LexCore is subpoenaed and holds the keys, LexCore must either produce the documents or assert privilege — but LexCore is not the attorney. What party can assert attorney-client privilege? What does this mean for the key custody model? Is there a mechanism where the *law firm* holds the keys and LexCore cannot comply with a subpoena even if it wanted to?
- **Hint**: Re-read Dmitri's hint about Zoom and client-side encryption. Zoom's end-to-end encryption design stores keys on client devices, not servers. What does "client-side" mean in a browser-based SaaS product? Where does the key live if the client is a web application?
- **Hint**: Re-read Achebe's framing: "policy can change — companies get acquired, employees go rogue, subpoenas happen." The concern is not just current policy but future scenarios. What architectural property must the replacement design have to be *structurally* immune to these scenarios, not just *contractually* protected against them?
- **Hint**: The watermarking requirement and client-side encryption are in tension. If LexCore cannot read document content, how does LexCore apply a watermark to a document before download? This is a hidden technical constraint that the rejected RFC glosses over and the accepted RFC (Ch. 175) must solve.

---

### Constraints

- **Deal volume**: AmLaw 100 firms do 200-500 deals/year each; 500 firms × 350 avg = 175,000 deals/year (only fraction use deal rooms initially)
- **Document size**: M&A due diligence rooms: 5,000-50,000 documents per deal, average 1MB each
- **Participants per deal room**: 2-6 firms, 5-20 attorneys per firm = 10-120 participants per room
- **Activity log volume**: estimated 50 access events/document/day × 10,000 documents = 500K events/day/large deal
- **Latency SLA**: document access < 2 seconds p95 (large PDFs)
- **Legal constraints**: ABA Model Rule 1.6, state bar confidentiality rules, attorney-client privilege doctrine
- **Team**: Dmitri + Yasmine + 1 backend engineer; 2 weeks to produce revised RFC (Ch. 175)

---

### Your Task

Produce the full RFC document for Version 1 (the rejected design). Specify the architecture completely, then document the rejection: what legal objection was raised, what architectural property the design lacks, and what the redesign must achieve. This is a documentation exercise as much as a design exercise — understanding a rejected RFC deeply is Staff-level work.

---

### Deliverables

- [ ] Mermaid architecture diagram: Version 1 deal room architecture (platform-managed keys, API-proxied access, activity logging)
- [ ] RFC document structure (full):
  - **Title**: Multi-Party Deal Room Architecture v1
  - **Status**: REJECTED
  - **Author(s)**: [You]
  - **Reviewers**: Sasha Petrenko, Mara Obi, Elliot Chamberlain, Rosalind Achebe
  - **Problem Statement**: 1 paragraph
  - **Proposed Solution**: Architecture description (reference diagram)
  - **Key Management**: how keys are stored and accessed
  - **Activity Logging**: what is logged and where
  - **Alternatives Considered**: 2 alternatives and why they were not chosen
  - **Legal Review Result**: verbatim capture of Achebe's objection
  - **Rejection Rationale**: architectural property that is missing
  - **Required for v2**: what the redesign must achieve
- [ ] Database schema for rejected design: `deal_room`, `deal_room_participant`, `deal_room_document`, `document_permission`, `activity_log` (with column types and indexes)
- [ ] Scaling estimation:
  - Deal room volume: 175,000 deals/year → peak concurrent rooms
  - Activity log write rate at peak: X events/sec
  - Encrypted storage: 10,000 docs × 1MB × AES overhead = X GB per large deal room
  - Show math step by step
- [ ] Tradeoff analysis for the REJECTED design (3 tradeoffs that were considered valid before the legal review):
  - Platform-managed keys vs. customer-managed keys (operational simplicity vs. customer control)
  - API proxy vs. presigned URL access (audit completeness vs. latency)
  - Per-document vs. per-room encryption (granularity vs. key management overhead)
- [ ] Rejection analysis: clearly articulate the two legal objections and map them to specific architectural components that must change in v2

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
