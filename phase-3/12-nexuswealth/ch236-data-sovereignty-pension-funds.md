---
**Title**: NexusWealth — Chapter 236: One Platform, Three Regulators
**Level**: Staff
**Difficulty**: 9
**Tags**: #data-sovereignty #compliance #multi-region #distributed-systems #financial-systems
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 231, Ch. 232, Ch. 237
**Exercise Type**: System Design
---

### Story Context

**Email chain — Subject: NexusWealth UK Expansion — Technical Architecture Sign-Off Required**

---

**From**: Claudia Hartmann, General Counsel
**To**: Priya Nair, CTO; you
**CC**: Oliver Pemberton (UK Country Director); Solicitors at Clifford Chance LLP (redacted)
**Date**: March 4, 2025 — 2:33 PM

Priya,

Following last week's regulatory pre-application meeting with the UK Pensions Regulator (TPR), I need to formalize a technical architecture requirement that we cannot move forward without resolving.

TPR has made clear that any pension data for UK scheme members must remain within the United Kingdom for storage, processing, and calculation. This is consistent with the UK GDPR (post-Brexit retained EU law), the Pensions Act 2004, and TPR's own data residency guidance published in September 2024. Specifically: a UK pension scheme member's compensation history, benefit entitlements, contribution records, and any derived calculations may not be processed on infrastructure outside the United Kingdom.

This creates a specific problem. Our risk calculation engine, actuarial calculation engine, and the benefit statement pipeline all run in US-East-1. I have been advised by Clifford Chance that routing UK member data through a US data center — even temporarily for calculation — constitutes a transfer of personal data outside the UK under UK GDPR Article 46. There is no adequacy decision between the UK and US that covers pension data of this nature.

The EU IORP II Directive adds a further requirement: if we extend to any EU member state in the future (likely Germany and Netherlands are the next targets), EU participant data cannot leave the EU. Separate from UK data.

We have also been made aware that the SEC, under its existing MOU with us for our US operations, requires that US participant data be accessible from within the United States for regulatory examination. Transferring US participant computation to UK infrastructure to serve UK members would potentially violate this MOU.

We have three conflicting data residency requirements that apply simultaneously to our platform. I need your team to design an architecture that satisfies all three before we submit our TPR application.

The TPR submission deadline is May 1st. We have 8 weeks.

Claudia

---

**1:1 transcript — you + Oliver Pemberton (UK Country Director) — March 7, 2025 — via video call**

**Oliver**: I want to give you practical context for what this means commercially. We've identified 180,000 UK scheme members in the first two years. By year five, our target is 2 million UK members across 400 UK pension schemes. These are defined benefit occupational pension schemes — FTSE 250 companies, large NHS trusts, some local government pension schemes. The schemes themselves are deeply regulated and their trustees are personally liable under UK law.

**You**: When you say trustees are personally liable — what does that mean in practice?

**Oliver**: It means if NexusWealth has a data residency violation and UK member pension data is found to have been processed in the US, the trustees of those schemes face potential regulatory action from TPR. Trustees have to certify annually that their scheme's data is handled compliantly. If they can't make that certification because of our architecture, they won't use us.

**You**: Help me understand the risk calculation problem. Our risk engine calculates portfolio-level VaR. A UK pension scheme's portfolio might hold US equities, UK gilts, European bonds. The market data for those securities comes from Bloomberg in the US. Can we use US-sourced market data if we process it on UK infrastructure?

**Oliver**: I've asked the same question. Clifford Chance says: market data is not personal data. It's fine to pull Bloomberg feeds from anywhere. The restriction is on the member-level personal data — compensation, service records, benefit entitlements. Market data is public. Member data is private. You can do the calculation in the UK using US-sourced market data; the member data never leaves the UK.

**You**: That's a useful distinction. But the actuarial calculation engine for benefit statements — that calculation depends on 15-25 variables per member, all of which are personal data. That calculation cannot run in the US. It must run on UK infrastructure.

**Oliver**: Correct.

**You**: And we need the same actuarial engine to work for US members, UK members, and eventually EU members. The engine itself is the same code. The data it operates on must stay in its jurisdiction.

**Oliver**: That's exactly the problem. Can you solve it?

---

**Slack DM — @marcus.webb → @you — March 8, 2025 — 11:14 AM**

**Marcus**: Just read Claudia's email (Priya forwarded it to me as outside advisor). The CLOUD Act problem is hiding in this. US-headquartered company running infrastructure in UK — US government can serve a CLOUD Act subpoena for data held in UK AWS infrastructure. UK authorities may have conflicting legal obligations to block that disclosure. Have you checked whether your UK AWS deployment satisfies UK GDPR's transfer safeguard requirements given that AWS is US-owned?

**Marcus**: This is not theoretical. There's a case from 2023 where a US fintech operating in Germany got caught between GDPR and a DOJ subpoena. Legal said "we're compliant with local law." They were — right up until they weren't. You need a legal opinion specifically on the CLOUD Act exposure for UK pension data.

**Marcus**: Architecture question: if the UK risk engine needs market data from Bloomberg (US), and market data is not personal data, how does the system authenticate the UK calculation worker to Bloomberg's API without the authentication credential (which is scoped to your US entity) passing through US infrastructure? That's your first practical problem.

---

You spend the weekend drafting the architecture. Three jurisdictions, three separate regulatory frameworks, one calculation engine, shared market data, isolated member data. The CLOUD Act risk Marcus raised is not a hypothetical — you need a legal opinion, but you also need to design the architecture so that even if a subpoena arrives, the data is structured in a way that minimizes exposure.

Eight weeks. May 1st.

### Problem Statement

Design a multi-jurisdiction pension administration platform that simultaneously satisfies: US ERISA + SEC requirements (US member data accessible in the US), UK Pensions Act + UK GDPR requirements (UK member data never leaves the UK), and EU IORP II requirements (EU member data never leaves the EU). The platform must support the same actuarial calculation engine, risk calculation engine, and benefit statement pipeline across all jurisdictions — with complete data residency enforcement at every layer. Market data (non-personal) may flow between jurisdictions. Member data (personal) may not.

### Explicit Requirements

1. US participant data: processed and stored in US-East-1; SEC-accessible for regulatory examination
2. UK participant data: processed and stored in EU-West-2 (London); no transfer outside UK under any condition
3. EU participant data (future): processed and stored in EU-Central-1 (Frankfurt); no transfer outside EU
4. Shared actuarial calculation engine codebase: deployed independently in each jurisdiction, operating on local data only
5. Bloomberg market data: can be ingested in US and replicated to UK/EU (it's non-personal); replication must not carry personal data
6. Cross-border risk calculation: portfolio risk for a UK scheme holding US equities — market data for US equities is non-personal and can flow; the portfolio composition (member ownership) is personal and must not
7. Portfolio-level VaR: calculated on UK infrastructure using locally-available market data; results visible to UK Investment Committee only
8. Audit trails: each jurisdiction maintains its own audit logs, non-exportable; DOL can examine US logs, TPR can examine UK logs
9. Identity: a NexusWealth employee in London managing UK accounts must not be able to inadvertently access US participant data and vice versa — data-aware RBAC
10. CLOUD Act mitigation: legal architecture opinion required + technical controls to minimize personal data exposure in any single jurisdiction subpoena

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's Slack message. He asks: "how does the system authenticate the UK calculation worker to Bloomberg's API without the authentication credential passing through US infrastructure?" Bloomberg API credentials are registered to NexusWealth's US legal entity. If the UK worker authenticates using those credentials, the authentication call may be logged in the US. What is the credential architecture? Does Bloomberg offer per-jurisdiction credential scoping? If not, what is the alternative?
- **Hint**: Re-read Oliver's description of UK trustee liability: "trustees have to certify annually that their scheme's data is handled compliantly." This means NexusWealth needs to produce a data residency compliance certificate for each UK scheme, each year. Your architecture must not just enforce data residency — it must produce auditable evidence that data residency was enforced for every calculation, every data access, every day of the year. This is a reporting requirement, not just an architecture constraint.
- **Hint**: Re-read Claudia's email about the IORP II extension. She says Germany and Netherlands are "likely next targets." Germany has additional data sovereignty requirements under the BDSG (Bundesdatenschutzgesetz) that go beyond GDPR — specifically, some categories of financial data require German data center residency, not just EU residency. If your architecture treats "EU" as a single zone, the German expansion may require a separate deployment. How does your multi-region architecture accommodate n+1 jurisdictions without becoming n separate codebases?
- **Hint**: Re-read your own question in the Oliver call: "A UK pension scheme's portfolio might hold US equities, UK gilts, European bonds." A UK member who is a dual US-UK citizen is both a UK scheme member (data must stay in UK) and potentially a US person under US tax law (IRS may have reporting obligations). What is the data model for a member who has personal data that is simultaneously subject to multiple jurisdictions? This is the hardest case, and it's not in the explicit requirements.

### Constraints

- **UK launch**: 180K members in year 1, 2M by year 5, 400 pension schemes
- **Jurisdictions at launch**: US + UK (EU in 12 months)
- **Data residency enforcement**: technical + legal + auditable
- **Shared components**: actuarial calculation engine codebase, risk calculation engine codebase, Bloomberg market data feed
- **Non-shared components**: member data, calculation results, audit logs
- **CLOUD Act**: US-headquartered company, UK/EU-deployed infrastructure
- **TPR submission deadline**: May 1st (8 weeks)
- **Team**: you + 3 engineers + Clifford Chance for legal review
- **Infrastructure**: AWS (US-East-1, EU-West-2, EU-Central-1)

### Your Task

Design the multi-jurisdiction data architecture. Address: data classification (personal vs. non-personal), per-jurisdiction deployment model for the calculation engines, cross-border market data replication without personal data leakage, data-aware RBAC for NexusWealth employees, annual compliance certification generation, and CLOUD Act risk mitigation architecture.

### Deliverables

- [ ] Mermaid architecture diagram — three-jurisdiction deployment with data flow labels explicitly showing personal vs. non-personal data
- [ ] Database schema — participant data model with jurisdiction tag, data residency enforcement layer, calculation result ownership, cross-border market data ledger
- [ ] Data classification taxonomy: define the boundary between personal data (residency-restricted) and operational data (non-restricted) for pension systems — at least 10 data types classified with rationale
- [ ] Scaling estimation:
  - UK deployment sizing for 180K members at launch and 2M at year 5
  - Market data replication volume: Bloomberg feed to 2 additional jurisdictions — bandwidth and storage
  - Compliance certificate generation: 400 UK schemes × annual certificates — what does the audit log query look like and how long does it take?
- [ ] Tradeoff analysis (minimum 3):
  - Shared codebase: single deployment with data-aware routing vs. per-jurisdiction independent deployments
  - RBAC model: attribute-based (ABAC) with jurisdiction as attribute vs. separate IAM for each jurisdiction
  - CLOUD Act mitigation: technical controls (encryption with local key management) vs. contractual controls (data processing agreements) vs. legal entity structuring
- [ ] Cost modeling: UK AWS deployment (EU-West-2) alongside US infrastructure — incremental cost; at 2M UK members vs. 12M US members, what is the cost-per-member differential?
- [ ] Capacity planning: design for Germany/Netherlands expansion as jurisdiction 3 and 4 with minimal incremental architecture change

### Diagram Format

Mermaid syntax.
