# Arc 10: PharmaSync — Pharmaceutical / Drug Discovery Data Platform

## Company Overview

PharmaSync is a B2B SaaS platform serving the pharmaceutical industry. Founded in 2017 by a former FDA regulatory affairs officer and a computational biologist, it sits at the intersection of compliance engineering and life sciences data infrastructure. The platform serves 8 of the top 20 pharmaceutical companies globally, managing clinical trial data, laboratory information, and drug discovery workflows for some of the highest-stakes software deployments on earth.

This is not fintech. This is not e-commerce. When data integrity fails here, clinical trials halt. When compliance architecture is inadequate, the FDA issues warning letters — or worse, clinical holds that freeze life-saving research mid-trial. The consequences of a bug are not a failed transaction or a missed notification. They are measured in months of delayed drug approvals and, occasionally, patient safety.

The stack is a careful mix of modern and legacy: PostgreSQL for clinical data, a custom audit logging layer built in-house, Python-heavy ML pipelines for computational biology, and a large volume of on-premise infrastructure at customer lab sites that must synchronize with the cloud. Node.js and TypeScript on the API layer, Kubernetes for orchestration, and a sprawling integration surface with every major LIMS vendor in the market.

## Your Role

You join as a Senior Engineer on the Platform Team. Your job title says "Senior," but the problems you'll be handed are Staff-level from day one. The team is small — seven engineers, two data scientists, and a regulatory affairs advisor named Preethi Nair who spent 14 years at the FDA before crossing to industry. Preethi is not an engineer, but she will review your architectural decisions with a precision that will make you reconsider everything you thought you knew about data integrity.

Your hiring manager, Tariq Osei, was one of the engineers who survived the 2022 FDA warning letter incident. He has the look of someone who has been through a real crisis and is building the team he wishes he'd had then.

## First Day

You arrive at PharmaSync's San Francisco office on a Tuesday morning. The lobby has a whiteboard with a quote from the FDA's 21 CFR Part 11 regulation. The all-hands standup lasts exactly eleven minutes. Tariq hands you a laptop and says: "Read the FDA warning letter we got last year. Then read our response. Then let's talk."

You read both. By 10am, you understand why this company exists and exactly how much pressure is riding on everything you are about to build.

## Arc Chapters

| Chapter | Title | Exercise Type | Difficulty |
|---------|-------|---------------|------------|
| Ch. 216 | 21 CFR Part 11 | System Design | 8 |
| Ch. 217 | Lab Data Integration | System Design | 7 |
| Ch. 218 | Feature Stores for Drug Discovery | System Design | 8 |
| Ch. 219 | Clinical Data Management | System Design | 9 |
| Ch. 220 | The LIMS Deprecation | System Evolution | 9 |
| Ch. 221 | SPIKE: Production DOWN — Clinical Trial Data Discrepancy | Incident Response | 10 |
| Ch. 222 | Validation Architecture | System Design | 9 |

## Industry Context

Pharmaceutical data infrastructure operates under regulatory frameworks that most engineers never encounter:

- **21 CFR Part 11**: FDA regulation governing electronic records and electronic signatures. Every record that replaces a paper record must be attributable, legible, contemporaneous, original, and accurate (ALCOA principles).
- **CDISC SDTM**: Clinical Data Interchange Standards Consortium Study Data Tabulation Model — the required format for FDA drug approval submissions.
- **GxP**: Good Practice regulations covering manufacturing, laboratory, and clinical practice. Software used in GxP environments requires validation.
- **CSV/CSA**: Computer System Validation (legacy) and Computer Software Assurance (modern FDA guidance) — the process of proving that software does what it's designed to do, consistently.

If you have never worked in a regulated industry before, these are not bureaucratic formalities. They are the mechanisms by which the FDA can trust that a drug approved for 300 million people was tested with data that was never tampered with.

## Milestone

GitHub Milestone: `PharmaSync Arc — Chapters 216–222`
