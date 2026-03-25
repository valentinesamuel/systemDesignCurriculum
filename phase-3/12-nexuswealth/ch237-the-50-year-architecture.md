---
**Title**: NexusWealth — Chapter 237: A Letter to 2074
**Level**: Staff
**Difficulty**: 9
**Tags**: #architecture #long-term-thinking #compliance #data-retention #staff-engineering
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 230, Ch. 231, Ch. 232, Ch. 236
**Exercise Type**: System Design
---

### Story Context

**Slack DM — @priya.nair → @you — April 22, 2025 — 9:05 AM**

**Priya**: I have an unusual request. We've been engaged by our outside counsel regarding the COBOL flat files — the ones you discovered in Ch. 230 from the predecessor company's 1987 system. They found something. One of those flat files is not just legacy data. It's an active benefit calculation input for a participant whose original hire date predates our acquisition. His name is Raymond Osei-Mensah. He's 71. He's been receiving pension checks for 6 years. The COBOL record is the authoritative source for his defined benefit formula.

**Priya**: Raymond's attorney has asked for a full accounting of how his benefit is calculated. The flat file format was documented in a 1991 technical memo that no one at the predecessor company can locate. We spent three weeks reverse-engineering the format to answer his question.

**Priya**: This should never happen again. Not in 10 years. Not in 50 years. I want you to write an architecture document — a real one, not a slide deck — that articulates how we build a system that a new engineer in 2074 can still understand, operate, and audit. What do we commit to? What do we plan to change? What do we absolutely refuse to let become another COBOL flat file?

---

**Email chain — Subject: Long-Term Architecture Document Request**

---

**From**: Priya Nair, CTO
**To**: you; Franklin Drummond, Board Chairman
**CC**: Renata Kowalski, VP Actuarial; Claudia Hartmann, General Counsel
**Date**: April 23, 2025 — 11:00 AM

Franklin,

As discussed in the February Trustees meeting, we committed to a long-term technology governance document. Our Staff Engineer is drafting it. I want to give you context for why this document matters beyond its face value.

The Raymond Osei-Mensah case is not an isolated incident. Our predecessor company made architecture decisions in 1987 that are creating real legal exposure today. The 2-inch-thick printout in our storage room is a liability, not a record.

Pension funds have a unique characteristic that almost no other industry shares: the beneficiary may live and receive payments for 30-50 years after the benefit is first calculated. That means our data — not our software, our data — must outlive every technology generation we will ever work in.

The document our engineer is writing needs to answer one question clearly: If someone tears up the office in 2074 looking for why a participant's benefit was reduced, will they find a clear, readable, independently verifiable audit trail — or will they find a 1.2TB parquet file that requires a tool no one has installed in 20 years?

We answer that question now, in architecture decisions we make today.

Priya

---

**1:1 transcript — you + Renata Kowalski (VP Actuarial) — April 24, 2025 — 3:00 PM**

**Renata**: I want to share my perspective on what "50 years" actually means for actuarial data. In 1974, when ERISA was signed, the standard benefit calculation was relatively simple — years of service times a multiplier. By 2024, the calculations we run involve 25+ variables, three regulatory frameworks, stochastic projections, and computational methods that didn't exist in 1974.

**You**: So the methods themselves change.

**Renata**: Dramatically. Every 10-15 years. The formula that calculated Raymond Osei-Mensah's benefit in 1988 is not the formula we'd use to calculate it today. But for Raymond's benefit, the 1988 formula is the right answer. His benefit was locked in under that formula's terms. We need to run that formula, not the 2024 formula.

**You**: So you need formula versioning. Not just data versioning.

**Renata**: Formula versioning, actuarial parameter versioning, and — here's the thing people miss — regulatory interpretation versioning. The DOL issued guidance in 2012 that changed how certain multi-employer plan calculations are done. Before that guidance, our calculations used one interpretation. After, a different one. If a 2012 participant asks for a benefit statement today, which interpretation applies? The one in effect when they accrued the benefit, or the current one?

**You**: That's a temporal query problem across three axes: the data as it existed, the formula as it was written, and the regulatory interpretation that was in effect.

**Renata**: Exactly. And in 2074, you need to be able to answer that question for a participant who accrued benefits in 2025. The engineer doing that work will not have been born yet when you make the architecture decisions today.

**You**: What's the biggest risk you see in our current architecture?

**Renata**: Two things. First: format lock-in. If we store everything in Parquet and Parquet is abandoned in 2040, we need a migration path. Second: key management. We encrypt participant data — as we should. But our encryption keys are managed by AWS KMS. AWS may not exist in 50 years in a form we recognize. The keys to decrypt 50-year-old pension data need to be under our custody in a way that doesn't depend on a specific cloud provider's continued existence.

---

**Slack DM — @marcus.webb → @you — April 25, 2025 — 6:45 PM**

**Marcus**: Priya told me what you're working on. I want to tell you something.

**Marcus**: In 1995 I worked on a system that was designed to last 20 years. We were proud of it. We used the best database available — it was called Sybase. We used the dominant interchange format — it was called EDI X12. We stored data on DLT tapes that required a specific tape drive to read.

**Marcus**: By 2005 Sybase was niche. By 2010 DLT tapes were unreadable without hardware that cost more to find than the data was worth. The "20-year system" was a migration nightmare by year 12.

**Marcus**: The engineers who built it weren't stupid. They were careful. They just didn't ask the right questions. They asked "will this technology still work in 20 years?" They should have asked "what does our migration strategy look like when this technology stops working — because it will?"

**Marcus**: Your 50-year architecture is not a document about which technologies to use. It's a document about how to migrate out of any technology before it becomes a prison.

**Marcus**: This is probably my last piece of advice I'll give you on an active project. I'm stepping back from client work. Write it well.

---

The weight of Marcus's message sits with you as you open the blank document. He's stepping back. This is his last active piece of advice — not a farewell yet, but the beginning of one.

You think about Raymond Osei-Mensah. The 1987 COBOL flat file. The 1991 memo no one can find. The 2-inch printout. And you think about the engineer in 2074 who will need to answer a retiree's question about a benefit you calculated in 2025.

You start writing.

### Problem Statement

Write a 50-year technical architecture document for NexusWealth's pension administration platform. The document must address: how participant data survives format obsolescence, how benefit calculation formulas are versioned and replayed across 50 years, how encryption keys outlive any specific cloud provider, how regulatory interpretation changes are tracked alongside data and formula changes, and what migration strategy must be executed every 10-15 years to prevent technology lock-in. The document must be honest about what cannot be predicted and provide governance mechanisms for navigating unknowns.

The Raymond Osei-Mensah case — a live participant whose benefit calculation depends on a 1987 COBOL record and a 1991 memo no one can find — is the concrete evidence for every architectural principle in this document.

### Explicit Requirements

1. Data format independence: participant data must be readable without proprietary tools in 50 years; define a format migration strategy triggered every 10-15 years
2. Calculation formula versioning: every formula used in a benefit calculation must be version-controlled, human-readable, and executable in isolation; a formula from 2025 must be runnable in 2074
3. Regulatory interpretation versioning: track which DOL/TPR/SEC interpretation was in effect at each point in time; point-in-time replay must include regulatory context, not just data
4. Encryption key governance: encryption keys for participant data must remain accessible for 50 years without dependency on any specific cloud provider or vendor
5. Audit trail permanence: every benefit calculation event must be independently auditable 50 years after it occurs; no part of the audit trail may be dependent on runtime infrastructure
6. Migration cadence: define a mandatory architecture review every 10 years with specific migration criteria and decision authority
7. Format sunset plan: define maximum allowed format age before mandatory migration (must address both storage format and query interface)
8. Technology independence layer: application logic must be separable from storage format — a new storage system in 2040 must not require rewriting the actuarial calculation engine
9. Catastrophic data loss prevention: offline, jurisdiction-controlled backup strategy that survives cloud provider bankruptcy or service termination
10. Human-readable record of last resort: for any participant, it must be possible to produce a human-readable text description of their benefit calculation that requires no software to interpret

### Hidden Requirements

- **Hint**: Re-read Renata's statement about "regulatory interpretation versioning." She describes three axes of temporality: data state, formula version, and regulatory interpretation. The Ch. 230 COBOL problem was a data format failure. But re-read the original Ch. 230 story — the flat file also didn't document which regulatory interpretation was used when it was created. A 50-year architecture must make the tri-temporal audit (data state × formula version × regulatory interpretation) possible as a first-class query, not a forensic reconstruction.
- **Hint**: Re-read Marcus Webb's Sybase/DLT tape story. His conclusion: "your 50-year architecture is not about which technologies to use — it's about how to migrate out of any technology before it becomes a prison." This points to a specific governance requirement: the architecture document must include explicit trigger criteria for migration — not "we'll evaluate when needed" but "when X happens, we must migrate within Y months." What are those triggers? Technology vendor acquisition? Format tool deprecation? Encryption algorithm vulnerability?
- **Hint**: Re-read the encryption key governance requirement in context of Renata's warning: "our encryption keys are managed by AWS KMS — AWS may not exist in 50 years." This points to a specific cryptographic architecture choice: HSM (Hardware Security Module) with physical key material held by NexusWealth, not delegated to a cloud provider's key management service. But HSMs also have a shelf life. What is the key rotation strategy for a 50-year encrypted archive? Specifically: how do you re-encrypt 50 years of data when an encryption algorithm becomes vulnerable (as AES-128 will eventually be to quantum computing)?
- **Hint**: Re-read the Raymond Osei-Mensah story from the opening. He's 71 years old. The document that governs his benefit was written when he was approximately 33 years old. He may live another 20 years, receiving monthly checks that depend on that 1987 calculation. The implicit 50-year architecture requirement that nobody states: the architecture must survive not just technology changes, but organizational changes — acquisitions, spin-offs, bankruptcy. If NexusWealth is acquired in 2040, the new entity must be able to assume the benefit obligations with a complete, portable, independently interpretable data archive.

### Constraints

- **Data retention minimum**: 50 years from participant's last benefit payment (a 25-year-old participant who retires at 65 may receive benefits until 95 — that's 70 years of retention from hire date)
- **Format migration frequency**: every 10-15 years, triggered by criteria you define
- **Encryption**: participant data is currently AES-256 encrypted; quantum-resistant algorithms needed by approximately 2035-2040
- **Regulatory changes**: DOL/SEC/TPR guidance changes materially every 7-10 years; ERISA itself has been amended multiple times since 1974
- **Team turnover**: assume 100% staff turnover every 10 years — no institutional memory can be relied upon
- **Cloud provider risk**: AWS, Azure, GCP may not exist in their current form in 50 years
- **Format risk**: Parquet, JSON, XML — all have finite life spans as dominant formats
- **Key management risk**: any cloud KMS is subject to vendor continuity risk

### Your Task

Design the 50-year architecture framework. This is not a system design in the traditional sense — it is an architectural governance document with technical substance. Address: data format lifecycle management, tri-temporal audit capability, encryption key governance over 50 years, mandatory migration criteria and governance, portable archive format, and organizational continuity (what does the data look like if NexusWealth is acquired in 2040?).

### Deliverables

- [ ] Mermaid architecture diagram — the technology independence layer: how application logic, storage format, and encryption are separated into independently replaceable components
- [ ] Tri-temporal data model schema: participant record with `(data_state_as_of, formula_version_id, regulatory_interpretation_id)` as queryable dimensions — schema and index design
- [ ] 50-year format lifecycle plan: table showing current format → migration trigger → replacement format → who decides → migration SLA (for both storage and encryption)
- [ ] Encryption key governance design: key hierarchy, custody requirements, quantum-resistant migration timeline, air-gapped backup strategy
- [ ] Scaling estimation:
  - Storage at 50 years: 12M participants × 50 years × average record size × calculation audit trail size — total storage estimate at 2025 and at 2075
  - Migration cost: estimating a format migration every 12 years — compute, engineering time, validation cost per migration event
  - Tri-temporal query performance: a query for "what was participant 77234891's benefit calculation, using the formula in effect on 2028-03-15, based on data as it existed on 2027-12-31, under the regulatory interpretation active in 2028?" — index design for sub-30-second response on a 50-year archive
- [ ] Tradeoff analysis (minimum 3):
  - Format choice: open formats (JSON, CSV — human-readable, format-stable) vs. efficient formats (Parquet — queryable, compact, tool-dependent)
  - Key custody: cloud KMS (convenient, vendor-dependent) vs. on-premises HSM (expensive, durable, requires physical security)
  - Organizational portability: self-contained archive format (interpretable without NexusWealth context) vs. integrated archive (requires NexusWealth metadata catalog to interpret)
- [ ] Cost modeling: 50-year infrastructure cost estimate — storage growth curve at $0.023/GB-month, with format migration overhead every 12 years; what is the total 50-year infrastructure cost for the data retention layer?
- [ ] Mandatory migration trigger criteria: a formal list of events that must trigger architecture review and potential migration, with decision authority and maximum response timeline

### Diagram Format

Mermaid syntax.
