---
**Title**: BuildRight — Chapter 126: CQRS and Advanced Event Sourcing for Project State
**Level**: Staff
**Difficulty**: 9
**Tags**: #event-sourcing #cqrs #bi-temporal #distributed #compliance #legal
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 51 (OmniLogix event sourcing intro), Ch. 70 (NexaCare GDPR), Ch. 131 (BuildRight GDPR erasure)
**Exercise Type**: System Design
---

### Story Context

The email arrives on a Tuesday morning. Fatima Al-Rashid, BuildRight's Legal Counsel, forwards it to you with a single line of her own: *"We need to talk. Now."*

---

**From**: Hassan Al-Farsi, Partner, Al-Farsi & Noor Advocates, Dubai
**To**: legal@buildright.io
**Subject**: Formal Request for System Records — Al-Barsha Residential Tower, Project ID BR-48291
**Date**: March 3, 2026

Dear BuildRight Legal Team,

Our client, Majestic Development LLC, is pursuing arbitration proceedings against Khalid Construction & Engineering LLC regarding alleged deviations from approved structural drawings on the Al-Barsha Residential Tower project.

Specifically, our client claims the contractor deviated from approved rebar spacing specifications on floors 17–23 sometime during Q3 2025. The contractor denies any deviation occurred.

We are formally requesting complete system records from BuildRight's platform for Project BR-48291, including all submittal approvals, RFI responses, and design revision histories, with timestamps showing the **state of approved drawings at any given point in time**.

Please advise on your data preservation obligations and timeline for response.

Regards,
Hassan Al-Farsi

---

By the time you reach the conference room, Fatima is already there with a printed copy and a highlighter. Liam Kowalski, the principal engineer, is staring at his laptop with an expression that suggests he already knows what you're about to find.

**Fatima**: "Tell me you can show me what 'approved rebar spacing' meant on March 1st, 2025. Not what it means now. What it meant then."

**Liam**: "...I mean. The current value is in the database. The audit log has timestamps. But reconstructing the exact state on a specific date — that's not something we've done before."

**Fatima**: "Can we do it or not?"

**Liam**: "Not quickly. The audit log records changes, but it doesn't record the full document state. We'd have to replay it manually. For a complex project with 600 submittals, that's..."

**Fatima**: "How long?"

**Liam**: "Weeks. If we don't have data gaps."

**Fatima**: "We have 30 days to respond to the subpoena. If we can't produce records, that is a default judgment. In construction arbitration in Dubai, the exposure is eight figures."

Silence.

**Fatima** (to you): "You've been here three weeks. Tell me this is something you can fix. Not for this case — for the next one. Because there will be a next one."

You pull up the current data model on the conference room TV. The `project_documents` table has a `current_state` JSONB column. There's an `audit_log` table with `changed_by`, `changed_at`, and a `delta` JSON column that stores only the fields that changed.

The audit log was added eighteen months ago. Before that, there is nothing.

You spend the next hour in Liam's office reconstructing what BR-48291's submittal record actually looked like on March 1, 2025. You find three data gaps where the audit log has jumps of 48+ hours with no entries — periods where a maintenance script was truncating what it thought were duplicate rows.

That afternoon, you write one sentence in a Slack DM to Thomas Adeyemi:

**You**: "Our system only stores current state. We cannot answer temporal legal questions reliably. I need six weeks and full platform access to redesign this."

His response comes back in four minutes.

**Thomas**: "You have four. Don't let the lawyers find out it's only four."

---

**#arch-platform** *(Slack — the next morning)*

**You**: "Liam — I need to understand the full event flow for a project document. Where does a submittal approval currently get persisted?"

**Liam Kowalski**: "API handler → writes to `project_documents.current_state` → fires `document.updated` event → audit_log trigger captures delta. That's it."

**You**: "So we have the delta, but we don't have snapshots. If I wanted to reconstruct state as of a date 18 months ago, I'd have to replay every delta since creation."

**Liam Kowalski**: "Correct. And some deltas reference external document versions in Autodesk BIM360 that we didn't snapshot either."

**You**: "How many events does a typical complex project generate over its lifetime?"

**Liam Kowalski**: "BR-48291 has about 14,000 audit entries. Large airport project we have has 140,000."

**You**: "Replaying 140,000 deltas to answer a legal question is not a viable strategy."

**Liam Kowalski**: "No. It is not."

Marcus Webb sends you a DM that evening. No greeting.

**Marcus Webb [DM]**: "Heard you're redesigning state management for 40,000 live projects. A few questions. What's the difference between the state that was VALID at a point in time vs the state that was RECORDED at a point in time? Are those the same thing in construction? Think carefully."

You stare at that for a long time. Then you start reading about bi-temporal modeling.

---

### Problem Statement

BuildRight's project state management uses a mutable current-state model with a delta-based audit log. This is legally insufficient: the platform cannot answer "what was the approved state of document X on date Y?" with confidence or performance. The system needs to be redesigned to support temporal queries for legal discovery, compliance audits, and arbitration proceedings, without destroying the existing operational write path or degrading the performance of the daily project management experience.

The redesign must handle projects with up to 140,000 events without making temporal reconstruction prohibitively slow, and must distinguish between when something was recorded in the system versus when it was legally valid — a distinction that matters when field approvals are logged hours after they occur.

---

### Explicit Requirements

1. Support temporal queries: "What was the approved state of document D on date T?" answerable in under 2 seconds for any project.
2. Maintain an immutable, append-only event log for all project state changes (no updates, no deletes to the log).
3. Implement snapshot optimization so temporal reconstruction does not require replaying the full event history for large projects.
4. Implement bi-temporal modeling: capture both `valid_time` (when the fact was true in the real world) and `transaction_time` (when it was recorded in the system).
5. Provide two separate read models via CQRS: one optimized for daily project management (current state, fast writes), one optimized for legal/compliance queries (temporal, historical).
6. Ensure the event log is cryptographically tamper-evident (hash chaining or equivalent).
7. Support legal export: given a project ID and date range, produce a complete ordered event timeline suitable for legal discovery.
8. The write path must not increase P99 latency beyond 50ms over current baseline.

---

### Hidden Requirements

- **Hint**: Re-read the Slack thread with Liam. What did he say about the Autodesk BIM360 document references? The event log records that a document was approved, but the document itself lives in an external system. How do you guarantee that the external document version referenced in an event is also preserved and retrievable?
- **Hint**: Re-read Marcus Webb's DM. He asks about the difference between valid time and transaction time in construction. Consider: a field inspector approves rebar spacing at 6 AM on-site (valid time), but the approval is logged in BuildRight at 3 PM when they regain cell coverage (transaction time). Legal proceedings care about the former. What happens to your temporal query results if you only store transaction time?
- **Hint**: Re-read Fatima's statement about the next case. She says there will be a next one. The system must not just reconstruct state — it must produce output that is admissible as evidence. That means chain of custody, non-repudiation. What cryptographic guarantees does an append-only log need to be legally defensible?
- **Hint**: Liam mentions a maintenance script that was truncating what it thought were duplicate rows. Eighteen months of audit history has gaps. Your redesign must either reconstruct or formally document those gaps — a partial history presented as complete is legally worse than no history at all.

---

### Constraints

- **Scale**: 40,000 active projects. Average 8,000 events/project lifetime. Largest project: 140,000 events.
- **Write throughput**: ~15,000 project events/hour across the platform.
- **Temporal query SLA**: P99 < 2 seconds for any project, any date.
- **Snapshot strategy**: Snapshots must be generated without blocking the write path.
- **Retention**: Event log must be retained for 10 years (NSW construction regulations). Some jurisdictions: 15 years.
- **Storage budget**: Current DB is 4 TB. Event log redesign budget: up to 3x current storage (12 TB total).
- **Team**: 2 platform engineers + you. Four weeks.
- **Legal deadline**: First legal export (BR-48291) required in 30 days.
- **External systems**: Autodesk Revit, Bentley, Trimble BIM integrations — document versions must be snapshotted locally, not by reference only.

---

### Your Task

Design the event-sourced project state system for BuildRight. Your design must cover the write path (event ingestion, snapshot generation), the read models (operational vs legal), the bi-temporal schema, and the legal export pipeline. You must explicitly address the data gap problem for existing historical data and the external BIM document version preservation problem.

---

### Deliverables

- [ ] **Mermaid architecture diagram** — show write path, snapshot service, two CQRS read models (operational vs legal), and legal export pipeline
- [ ] **Database schema** — `project_events` table (with `valid_time`, `transaction_time`, `event_hash`, `prev_hash`), `project_snapshots` table (with snapshot trigger policy), `document_version_archive` table for BIM external references; include all column types and indexes
- [ ] **Scaling estimation** — calculate: total event storage for 40,000 projects × 8,000 events average at 2 KB/event; snapshot frequency needed to keep temporal reconstruction under 2 seconds; storage cost at $0.023/GB-month
- [ ] **Tradeoff analysis** — minimum 3: (1) snapshot frequency vs storage cost vs reconstruction latency, (2) bi-temporal complexity vs operational simplicity, (3) hash-chaining tamper evidence vs write performance
- [ ] **Snapshot trigger policy** — define when snapshots are generated (time-based? event-count-based? both?), how they are invalidated if earlier events are amended (are they?), and how reconstruction falls back to replay when no snapshot exists
- [ ] **Data gap remediation plan** — describe how you handle the 18-month period with audit gaps: formal gap annotation in the event log, legal disclosure language, reconstruction confidence scoring
- [ ] **Legal export format** — describe the output schema of a legal discovery export: what fields, what ordering, what signature/hash verification a court system would need

---

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

```
Example node style:
graph TD
    A[API Layer] --> B[(Event Store)]
    B --> C[Snapshot Service]
```
