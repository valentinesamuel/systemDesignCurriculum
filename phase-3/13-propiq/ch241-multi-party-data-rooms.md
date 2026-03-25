---
**Title**: PropIQ — Chapter 241: The $2M Data Room
**Level**: Staff
**Difficulty**: 8
**Tags**: #data-rooms #access-control #abac #audit-trails #multi-party #compliance
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 238 (PropIQ property data mesh), Ch. 159 (IronWatch compartmentalization)
**Exercise Type**: System Design
---

### Story Context

**#deals-channel — PropIQ Slack**
_Tuesday, 9:47 AM_

**Leila Ahmadi [Product]:** URGENT. Meridian Capital wants to use PropIQ as their due diligence platform for a $400M portfolio acquisition. 62 commercial properties across Austin, Dallas, and Phoenix. They need a secure data room — like a VDR but built into our platform. Deal closes in 6 weeks. They want our access controls live in 2 weeks.

**Leila Ahmadi [Product]:** This is a $2M ARR deal. I already said yes to the timeline. Divya — can we do this?

**Divya Nair [VP Eng]:** Leila, you said yes before asking engineering?

**Leila Ahmadi [Product]:** They were about to go to CoStar. We had 10 minutes.

**Divya Nair [VP Eng]:** @you come find me right now.

---

**#eng-staff — PropIQ Slack**
_Tuesday, 10:03 AM_

**Divya Nair [VP Eng]:** Here's what Meridian Capital needs. I just got off the phone with their GC.

**Divya Nair [VP Eng]:**
```
Parties requiring access:
- Meridian Capital (PE firm) — 8 analysts, 2 partners
- Kirkland & Ellis LLP — 4 attorneys (due diligence counsel)
- Latham & Watkins LLP — 2 attorneys (seller's counsel)
- Sullivan & Cromwell LLP — 3 attorneys (lender's counsel)
- Goldman Sachs — 2 bankers (acquisition financing)
- Deutsche Bank — 2 bankers (mezzanine lender)

Total: 23 external parties

Data they need access to:
- 62 property records (financial models, rent rolls, inspection reports)
- Municipal zoning docs (some restricted to Meridian only)
- Environmental reports (restricted: Meridian + lender banks only)
- Tenant estoppels (restricted: Meridian + Kirkland only)
- Title commitments (all parties but redacted versions for banks)

Requirements:
- Access expires automatically at deal close OR in 30 days, whichever comes first
- Full audit trail: who viewed what, when, for how long
- Revocation: any party's access can be cut instantly
- Watermarking: every document download must embed recipient identity
- No cross-contamination: Kirkland cannot see Deutsche Bank's access logs
- MFA required for all external access
```

**Divya Nair [VP Eng]:** We have document storage (S3-backed). We have auth (Cognito + JWTs). We do NOT have policy-enforced access at the document level. We do NOT have watermarking. We do NOT have multi-party audit isolation.

**Divya Nair [VP Eng]:** What do we build and what do we buy?

---

**DM: you → Marcus Tran**
_Tuesday, 10:31 AM_

**you:** Marcus — quick question before I design anything. What's our current document access model? How does authorization work today?

**Marcus Tran [Senior Data Engineer]:** Ha. "Authorization model." That's generous. We have a `documents` table. Each document has a `tenant_id`. If your JWT has the right `tenant_id` claim, you can see it. That's... it.

**you:** No resource-level permissions at all?

**Marcus Tran:** We have a `is_public` flag. Very sophisticated.

**you:** And the document storage — presigned URLs?

**Marcus Tran:** 1-hour presigned S3 URLs. No revocation. Once it's issued, it's issued. I've been asking to fix this for two years.

**you:** What about audit logs?

**Marcus Tran:** We log to CloudWatch. But it's just HTTP request logs. We don't log document views specifically. No download tracking.

**you:** Okay. One more thing — do we have a watermarking service anywhere?

**Marcus Tran:** We have a PDF generation library (pdf-lib) for report exports. Zero watermarking.

**you:** Right. Let me come find you in 20 minutes. We're going to design this from scratch.

---

**Meridian Capital Requirements Call — Design Session Notes**
_PropIQ Conference Room B, Tuesday 11:15 AM_

You and Marcus Tran sit at the whiteboard. You've pulled up the Meridian requirements and you have a legal pad.

**you:** "Okay. Let's start with what we actually need to build. The core problem is that we have a single tenant model — Meridian Capital is our customer. But inside this deal, there are six distinct parties who need different views of the same document set. And those parties cannot see each other's access patterns. This is not multi-tenancy. This is multi-party access control."

**Marcus Tran:** "It's basically compartmentalization. Like security clearances but for a real estate deal."

**you:** "Exactly. And here's the wrinkle — the documents themselves don't change. What changes is who can see which documents, under what conditions, for how long. That's policy, not data."

**Marcus Tran:** "ABAC?"

**you:** "Has to be. A tenant-based model can't express 'Goldman Sachs can see the environmental report but only the unredacted version, and only until March 15th.' We need subject attributes, resource attributes, and environmental attributes — time, action, party type."

**Marcus Tran:** "And the audit log isolation is actually harder than the access control. Because Kirkland can't see that Goldman downloaded something. But Meridian needs to see ALL access logs for all parties."

**you:** "Right. So audit logs need a visibility model of their own. They're not a single stream — they're a stream with access control applied."

**Marcus Tran:** (writing on whiteboard) "Data room → parties → document sets → policies → audit logs (with their own access control). And watermarking is a delivery concern, not a storage concern. We watermark at serve time, not at upload time."

**you:** "Correct. S3 stores the clean document. Our delivery layer embeds the watermark per recipient before streaming bytes. Never store the watermarked version — it's ephemeral."

**Marcus Tran:** (pause) "We're building a VDR. Firms charge $50K/month for this."

**you:** "We're building the minimum that satisfies Meridian's GC. In 2 weeks. Let's scope carefully."

---

### Problem Statement

Meridian Capital, a PE firm, is conducting due diligence on a $400M commercial property portfolio. They need PropIQ to provide a secure, policy-enforced data room for 23 external parties (attorneys, bankers, analysts) across 6 organizations — each requiring different document access permissions, time-limited credentials, and full audit trails with cross-party isolation.

PropIQ currently has no resource-level access control, no download tracking, no watermarking, and no document-level policy enforcement. The deal requires this to be live in 14 days. The system must satisfy the legal and compliance requirements of a high-stakes M&A transaction, including audit-admissible logs and instant access revocation.

### Explicit Requirements

1. **Multi-party access control**: 6 distinct parties (Meridian Capital, 3 law firms, 2 banks) each with different document-level permissions
2. **Attribute-Based Access Control (ABAC)**: policies based on party type, document classification, time, and action (view vs. download)
3. **Time-limited access**: all access expires automatically at deal close date or 30-day TTL, whichever comes first
4. **Instant revocation**: any party's access can be terminated immediately; previously issued URLs must become invalid
5. **Full audit trail**: every document view, download, and access attempt logged with timestamp, party, duration
6. **Cross-party audit isolation**: Kirkland cannot see Goldman's access logs; Meridian can see all parties' logs
7. **Digital watermarking**: every document download embeds the recipient's name, organization, and timestamp invisibly in the PDF
8. **MFA enforcement**: all external party members must complete MFA before accessing any document
9. **Redacted variants**: some documents have organization-specific redacted versions (e.g., banks get redacted title commitments)
10. **Data room lifecycle**: data room state machine — Draft → Active → Closed; documents become inaccessible at Closed state

### Hidden Requirements

1. **Hint**: Re-read the Meridian GC requirements — "No cross-contamination: Kirkland cannot see Deutsche Bank's access logs." This implies not just audit log filtering but that the audit log query API itself must enforce party-scoped visibility. If you build a single audit stream and filter at query time, a security bug could leak logs. Consider append-only, party-partitioned audit storage.

2. **Hint**: Marcus Tran notes presigned URLs are "1-hour, no revocation — once issued, it's issued." This means URL-based access control is architecturally insufficient for revocation requirements. Revisit how documents are actually delivered — is a short-lived signed URL still valid if the party's access is revoked 5 minutes after issuance? You need a server-side access check at delivery time, not just at URL generation time.

3. **Hint**: The watermarking requirement says "embed recipient identity." Watermarking happens at serve time, not storage time. But what if the same user downloads the same document twice? The watermark must be consistent (same timestamp? or per-download timestamp?). And what if a document is printed and photographed? Consider visible vs. invisible watermarking strategy — the GC mentioned both.

4. **Hint**: Leila said "deal closes in 6 weeks, they want access controls live in 2 weeks." The data room has a hard-coded expiry at deal close. But deal close dates slip. What happens if Meridian's GC calls and says "we need 2 more weeks"? The system needs a deal admin extension flow — and that extension must itself be audited.

### Constraints

- **Parties**: 6 organizations, 23 individual users
- **Documents**: ~200 documents per data room (62 properties × ~3 docs each + deal-level docs)
- **Document size**: average 8 MB PDF, max 150 MB (engineering survey reports)
- **Total storage per data room**: ~1.6 GB
- **Concurrent sessions**: peak 15 simultaneous users (cross-timezone deal teams)
- **Audit log retention**: 7 years (M&A legal hold requirement, not HIPAA — but same principle)
- **Revocation SLA**: access must become invalid within 60 seconds of revocation action
- **Watermarking latency**: must not add more than 2 seconds to document serve time
- **MFA**: TOTP or SMS OTP acceptable; hardware key optional
- **Budget for external services**: $2,000/month max (watermarking vendor, MFA provider)
- **Team size**: you + Marcus Tran for implementation; 2 weeks timeline
- **Existing infrastructure**: AWS (S3, Cognito, RDS PostgreSQL, Lambda, API Gateway)

### Your Task

Design the PropIQ Multi-Party Data Room system. This includes:

1. The ABAC policy model — how policies are represented, evaluated, and stored
2. The document delivery architecture — how access is enforced at serve time (not just URL generation)
3. The watermarking pipeline — how recipient identity is embedded at serve time without storing watermarked copies
4. The audit trail architecture — how logs are stored with party-scoped visibility
5. The revocation mechanism — how previously issued access is invalidated within 60 seconds
6. The data room lifecycle state machine — Draft → Active → Closed with extension flow

### Deliverables

- [ ] **Mermaid architecture diagram**: End-to-end data room system (document upload, policy assignment, access request, watermarked delivery, audit logging)
- [ ] **ABAC policy schema**: PostgreSQL schema for subjects, resources, policies, conditions (with column types and indexes)
- [ ] **Audit log schema**: Party-partitioned audit storage with visibility model
- [ ] **Scaling estimation**: Show math for storage, audit log volume, watermarking throughput over 30-day deal lifecycle
- [ ] **Tradeoff analysis**: Minimum 3 explicit tradeoffs
- [ ] **Revocation design**: How 60-second revocation SLA is achieved given S3 presigned URLs
- [ ] **Watermarking architecture**: Serve-time vs. store-time tradeoffs; visible vs. steganographic watermark decision

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

---

#### Scaling Estimation (Starter Math — Complete This)

**Storage:**
- 200 documents × 8 MB average = 1.6 GB per data room
- Clean originals only in S3 (watermarked versions ephemeral, not stored)
- PropIQ may run 50 concurrent data rooms at peak → 80 GB total S3 storage
- At $0.023/GB/month: ~$1.84/month per data room (negligible)

**Audit log volume:**
- 23 users × estimated 20 document views/day × 30 days = 13,800 audit events per data room
- Each audit event: ~500 bytes (JSON with party_id, document_id, action, timestamp, IP, user agent, duration_ms)
- Per data room: 13,800 × 500 bytes = 6.9 MB of audit logs
- At 50 concurrent data rooms: 345 MB of audit logs
- 7-year retention: 345 MB × 12 months × 7 years = ~29 GB (small — use RDS, not object storage)

**Watermarking throughput:**
- Peak: 15 concurrent users, each downloading 1 doc simultaneously
- 15 parallel watermarking operations × 8 MB average = 120 MB being processed concurrently
- pdf-lib in Lambda: ~800ms for a 8MB PDF watermark embed (CPU-bound)
- 15 concurrent = 15 Lambda instances (parallelism trivially handled by Lambda)
- Cold start concern: Lambda warm-up adds 2-3s; use provisioned concurrency for the watermarking function

**Revocation:**
- Current S3 presigned URLs: 1-hour validity
- Problem: revoked party's URL valid for up to 60 minutes post-revocation
- Solution options: (1) reduce URL TTL to 30 seconds + server-side token check, (2) proxy-based delivery (no direct S3 URL exposure), (3) CloudFront signed cookies with short TTL
- Cost of option 2 (proxy): Lambda Data Transfer: 200 docs × 8MB × 23 users × 2 downloads avg = ~73 GB/month × $0.09/GB = ~$6.57/month per data room — acceptable

---

#### Tradeoff Analysis (Starter — Complete This)

**Tradeoff 1: Store-time vs. serve-time watermarking**
- Store-time: pre-generate watermarked copies per recipient → instant delivery but 23× storage per document, no revocation granularity, stale watermarks if recipient identity changes
- Serve-time: watermark on every download → storage efficient, always-fresh watermark, but adds latency (~800ms) and compute cost
- **Recommendation**: serve-time for documents <50MB; store cached watermarked versions with 30-min TTL for large documents (>50MB engineering reports)

**Tradeoff 2: ABAC policy engine — homebrew vs. OPA vs. vendor**
- Homebrew (SQL WHERE clauses): fast, no external dependency, but complex policies become unmaintainable SQL; no policy testing framework
- Open Policy Agent (OPA): battle-tested, rego language for complex policies, sidecar deployment, ~5ms evaluation latency; adds operational complexity
- AWS Cedar / AuthZed: managed, scales, but vendor lock-in and adds per-evaluation cost
- **Recommendation**: OPA sidecar for this use case — policies are well-bounded (deal lifecycle), rego is expressive enough, team can learn it in 2 weeks

**Tradeoff 3: Audit log isolation — physical vs. logical separation**
- Logical (single table, party_id column, row-level security): simpler to build, harder to guarantee isolation under bugs; RLS policies can be bypassed by superuser or admin roles
- Physical (separate audit log table per party, or separate schema per party): stronger isolation, harder to build cross-party views for Meridian, more tables to manage
- **Recommendation**: Separate audit log partitions by party_id with application-layer query construction — never expose a "query all parties" endpoint; Meridian's admin view calls N separate queries and merges client-side, enforcing that no single query can return cross-party data

---

#### ABAC Policy Model (Design Prompt)

Your policy model must express:
```
Subject:    { party_id: "kirkland", user_id: "alice@kirkland.com", role: "attorney" }
Resource:   { document_id: "doc_123", classification: "tenant_estoppel", deal_id: "deal_456" }
Environment:{ timestamp: now(), action: "download", data_room_state: "active" }
Policy:     { effect: "allow", conditions: [party_role IN ["buyer_counsel"], classification IN ["tenant_estoppel"], data_room_state = "active", timestamp < deal_close_date] }
```

Design the PostgreSQL schema that stores these policies. Consider:
- How do you store `conditions` — JSON? JSONB? Separate condition rows?
- How do you evaluate policies at serve time in < 10ms?
- How do you handle policy conflicts (two matching policies, one allow, one deny)?
- How do you version policies (what if Meridian changes permissions mid-deal)?
