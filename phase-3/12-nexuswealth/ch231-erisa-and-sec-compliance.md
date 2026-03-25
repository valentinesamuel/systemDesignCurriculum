---
**Title**: NexusWealth — Chapter 231: ERISA and SEC Compliance
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #compliance #erisa #sec #audit #fiduciary #data-pipeline
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 230 (Fifty Years of Data), Ch. 6 (MeridianHealth HIPAA), Ch. 40 (CivicOS FedRAMP)
**Exercise Type**: System Design
---

### Story Context

**Day 9. NexusWealth HQ. Compliance Wing, 3rd Floor. 8:02 AM.**

The letter arrives via registered mail and email simultaneously. That's never a good sign.

> **United States Department of Labor**
> **Employee Benefits Security Administration**
> **Office of Enforcement**
>
> **Via Certified Mail and Electronic Transmission**
>
> NexusWealth Investment Management, LLC
> Attn: Chief Compliance Officer
>
> **RE: Voluntary Compliance Examination — Pension Plan Fee Disclosure and Investment Decision Documentation**
>
> Dear Sir or Madam,
>
> The Employee Benefits Security Administration (EBSA) has selected NexusWealth Investment Management, LLC for a voluntary compliance examination under ERISA Sections 404 and 408. This examination covers the period January 1, 2020 through December 31, 2024.
>
> Within **30 calendar days** of this notice, please provide:
>
> 1. All investment committee meeting minutes and supporting analysis for any investment decisions affecting plan assets, 2020–2024
> 2. All fee disclosure notices provided to participants under ERISA Section 404a-5 (annual fee disclosure), 2020–2024
> 3. Documentation of the fee calculation methodology in effect for each plan year, including any changes to methodology and the date such changes took effect
> 4. For any plan where fees exceeded 1.5% of plan assets in any year: a written fiduciary justification
> 5. All participant complaints related to fees received during the examination period, and documentation of how each complaint was resolved
>
> Failure to respond fully and timely may result in escalation to a formal investigation under ERISA Section 504.
>
> Sincerely,
> Margaret Chen, Regional Director
> EBSA, Northeast Region

---

**#compliance-emergency — Slack channel — 8:17 AM**

**@thomas.reyes** *(Chief Compliance Officer)*: Everyone who needs to be in the war room at 9 AM, be there. This is a DOL audit request. 30 days.

**@diane.yoshida**: What's our exposure?

**@thomas.reyes**: Unknown until we pull the data. That's the problem — we don't have a single system that can answer all five of these requests. The investment decisions are in one system. The fee disclosures are in another. The fee calculation methodology changes are... I'm not sure they're documented anywhere.

**@priya.nair**: The fee methodology changes. That's the one that worries me.

**@thomas.reyes**: Why specifically?

**@priya.nair**: Because in March 2022, we changed how we calculate management fees for defined-benefit plans. We moved from a flat basis-point model to a tiered AUM model. That change should have triggered participant disclosure notices under 404a-5. Did we send them?

*[silence in the channel for 47 seconds]*

**@thomas.reyes**: I'm going to need someone to pull every fee disclosure sent in Q2 2022 and verify coverage.

**@robert.osei**: There's another issue. The DOL's item 3 asks for "documentation of the fee calculation methodology in effect for each plan year, including any changes." We don't have a versioned record of our fee calculation logic. When we updated the algorithm, we just... updated it. We don't have the old version.

**@priya.nair**: We have git history.

**@robert.osei**: Git history for the code, yes. But is that the same as "documentation of the methodology"? I don't think a DOL examiner wants to read TypeScript.

**@priya.nair**: @consultant — join the war room at 9. Bring your whiteboard markers.

---

**9:00 AM — War Room ("Independence" conference room)**

Around the table: Thomas Reyes (CCO), Priya Nair (CTO), Robert Osei (Chief Actuary), Keiko Tanaka (Senior Counsel from Hartwell & Crane, dialed in), and you.

Keiko speaks first. "I want to be direct. The DOL selected us, which means they have a reason. Voluntary compliance examinations don't happen at random — they're triggered by anomalies in Form 5500 filings, participant complaints, or referrals. The March 2022 fee methodology change is the most likely trigger. If we changed our fee calculation and didn't properly disclose it to participants, we have a fiduciary breach."

Thomas: "How bad?"

Keiko: "ERISA violations carry personal liability for plan fiduciaries. We're talking about potential restoration of losses to participants, excise taxes, and in egregious cases, DOL can seek civil penalties. The number on the table for an undisclosed fee methodology change affecting 847 plans for 2+ years is... significant."

Priya turns to you: "Here's what I need from the architecture side. We need to be able to answer these five DOL requests within 30 days. Our current systems cannot do that — at least not in a way I'd want to present to a federal examiner. I want to understand what a proper compliance data architecture looks like, so this never happens again."

Robert adds quietly: "And I want to know how we would have *caught* this. A compliance violation this serious should have triggered an alert when the methodology changed, not when the DOL knocked on the door."

---

**You stay late. Whiteboard notes from 6 PM.**

You map the current state. Five systems, no integration:

1. **InvestmentPro** (legacy Java monolith): investment committee decisions, meeting minutes
2. **FeeEngine** (Node.js service, deployed 2021): current fee calculations
3. **ParticipantComms** (third-party SaaS, Broadridge): disclosure notices sent
4. **ComplianceTracker** (Excel spreadsheets, maintained by Thomas's team): manual compliance calendar
5. **PlanAdmin** (core platform): plan rules, participant data

None of these systems talk to each other in a structured way. There is no event that fires when FeeEngine logic changes. There is no system that cross-references "fee methodology changed" with "participant disclosures sent."

The March 2022 change is reconstructable from git history. But git history doesn't tell you *which participants should have received disclosures* or *whether they did*.

---

**Slack DM — 11:23 PM**

**@priya.nair**: You still here?

**@you**: Yeah. Working through the compliance data model.

**@priya.nair**: What are you thinking?

**@you**: The core problem is that compliance is reactive. Events happen — fee methodology changes, investment decisions — and there's no system that turns those events into compliance obligations and tracks them to completion. We need compliance-as-events, not compliance-as-spreadsheets.

**@priya.nair**: Can we build that in 30 days?

**@you**: No. For the audit response, we patch and document. The architecture takes 6 months. What we can do in 30 days is pull the data manually, build a one-time extraction pipeline for the DOL response, and document what we find — including the gap on Q2 2022 disclosures.

**@priya.nair**: Thomas is going to want to know if we self-report the disclosure gap.

**@you**: That's a legal question. But architecturally — if we had the right system in place, we wouldn't have to decide whether to self-report because we'd have detected it and fixed it before the DOL did.

**@priya.nair**: "Build the system that removes the decision."

**@you**: Exactly.

---

### Problem Statement

NexusWealth faces a DOL compliance examination covering 5 years of investment decisions and fee disclosures. The examination has exposed a structural gap: compliance obligations (fee disclosures, fiduciary documentation, participant notices) are tracked manually in spreadsheets and are not automatically triggered by the events that create them. A fee methodology change in March 2022 may not have generated the required participant disclosure notices, representing a potential fiduciary breach.

You must design:
1. The compliance data architecture that enables DOL audit response within 30 days (current-state remediation)
2. The target compliance event system that automatically generates and tracks compliance obligations when regulated events occur (future-state architecture)

### Explicit Requirements

1. Compliance data store: federate data from InvestmentPro, FeeEngine, ParticipantComms, ComplianceTracker, PlanAdmin into a queryable compliance ledger
2. Audit response pipeline: given a date range and query type (e.g., "all fee disclosures sent Q1 2022"), produce a complete, auditable response within hours (not days)
3. Compliance event system: when a regulated event occurs (fee methodology change, investment decision, plan amendment), automatically generate a compliance obligation record with: what must be done, deadline, who is responsible, and status
4. Fee methodology versioning: every change to fee calculation logic must be captured with: previous version, new version, effective date, approver, and list of affected plans/participants
5. Participant disclosure tracking: for every required disclosure, track: was it generated, was it sent, was it received (to extent provable), and was it acknowledged
6. Fiduciary decision audit trail: investment committee decisions must be immutable once recorded, with supporting analysis linked
7. Alert system: proactive alerts when compliance obligations are approaching deadline or at risk of breach

### Hidden Requirements

- **Hint**: Re-read Keiko's statement: "Voluntary compliance examinations don't happen at random — they're triggered by anomalies in Form 5500 filings." Form 5500 is an annual IRS/DOL filing that NexusWealth submits on behalf of each pension plan. The DOL's automated systems analyze these filings for anomalies. Your architecture should include a Form 5500 pre-flight validation system — if your 5500 data contains anomalies, you should detect them *before* the DOL does.

- **Hint**: Re-read Robert's concern: "Git history for the code, yes. But is that the same as 'documentation of the methodology'?" The DOL does not accept source code as methodology documentation. Your fee methodology versioning system must produce human-readable methodology documents (not just git diffs) that are co-generated with code changes. Think: policy-as-code with auto-generated human-readable summaries.

- **Hint**: Thomas said the compliance calendar is in Excel spreadsheets. This means compliance obligations are being tracked outside any system of record. When an engineer changes FeeEngine, there is no mechanism to notify Thomas's team. Your architecture needs a compliance obligation trigger that fires on code deployment events for compliance-relevant services — a connection between your CI/CD pipeline and the compliance event system.

- **Hint**: The DOL request item 5 asks for "all participant complaints related to fees received during the examination period, and documentation of how each complaint was resolved." Participant complaints likely flow through multiple channels: call center, email, participant portal, written correspondence. This implies a complaint unification layer — all channels must write to a single complaint ledger with resolution tracking.

### Constraints

- **Scale**: 847 pension plans, 12M participants, 5-year lookback for this audit
- **Fee disclosures**: ~12M disclosure notices/year, sent via mail + email + portal
- **Investment decisions**: ~2,400 documented investment decisions over 5 years (across all plans)
- **Audit response SLA**: 30 days from DOL notice (hard deadline, legal consequences for missing)
- **Compliance events**: estimated 50-200 compliance-triggering events per month across all plans
- **Retention**: all compliance records must be retained for at least 6 years (ERISA statute of limitations) + ongoing plan life
- **Integration**: must federate from 5 existing systems without replacing them (budget and timeline constraints)
- **Team**: 4 engineers + Thomas's compliance team (2 people) for the 30-day sprint

### Your Task

Design the compliance data architecture for NexusWealth. Separate your design into two tracks:
- **Track 1 (30 days)**: DOL audit response — data extraction, federation, and response generation
- **Track 2 (6 months)**: Target state compliance event system — event-driven compliance obligation management

### Deliverables

- [ ] **Mermaid architecture diagram**: Full compliance architecture — data federation layer, compliance event bus, obligation tracker, audit response generator
- [ ] **Database schema**: `compliance_events` table (event_type, triggered_by, affected_plans, affected_participants, created_at, status); `compliance_obligations` table (obligation_type, deadline, owner, status, evidence_links); `fee_methodology_versions` table (version_id, effective_from, methodology_doc_hash, affected_plans, approver)
- [ ] **DOL audit response pipeline**: Step-by-step data extraction and response assembly for each of the 5 DOL request items; estimate time to produce each
- [ ] **Scaling estimation**: Volume math — 12M disclosures/year × 5 years = 60M records; compliance event throughput; audit query latency expectations
- [ ] **Tradeoff analysis**: Minimum 3 tradeoffs:
  - Federated compliance query vs centralized compliance data warehouse
  - Event-driven compliance triggers vs scheduled compliance sweeps
  - Automated disclosure generation vs human-in-the-loop approval workflow
- [ ] **Cost modeling**: $X/month — compliance data warehouse storage, query compute, audit trail retention, disclosure delivery infrastructure
- [ ] **Capacity planning**: 12-month build-out plan from current state to target compliance architecture; phased deployment with regulatory milestones

### Diagram Format

All architecture diagrams: Mermaid syntax.

```mermaid
graph TD
    subgraph Current Systems — Data Sources
        A[InvestmentPro<br/>Java monolith]
        B[FeeEngine<br/>Node.js]
        C[ParticipantComms<br/>Broadridge SaaS]
        D[ComplianceTracker<br/>Excel / SharePoint]
        E[PlanAdmin<br/>Core Platform]
    end

    subgraph Track 1 — Audit Response — 30 days
        A --> F[CDC / Export Jobs]
        B --> F
        C --> F
        D --> G[Manual Import]
        E --> F
        F --> H[Compliance Data Lake<br/>S3 + Athena]
        G --> H
        H --> I[Audit Query Engine<br/>Presto / Athena]
        I --> J[DOL Response Assembler<br/>5 query templates]
        J --> K[Response Package<br/>PDF + CSV + chain of custody]
    end

    subgraph Track 2 — Compliance Event System — 6 months
        B --> L[Compliance Event Bus<br/>Kafka topic: compliance-events]
        B --> M{Code Deploy<br/>FeeEngine?}
        M -->|yes| L
        L --> N[Obligation Engine<br/>event → obligation rules]
        N --> O[Compliance Obligation Store<br/>PostgreSQL]
        O --> P[Alert Service<br/>deadline monitoring]
        O --> Q[Dashboard<br/>Thomas's team]
        R[Form 5500 Pre-flight<br/>anomaly detection] --> L
    end
```
