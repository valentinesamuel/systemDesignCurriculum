# Arc 12: NexusWealth — Pension & Retirement / Wealth Management Platform

## Company Overview

NexusWealth is a Philadelphia-based pension administration platform managing $2.3 trillion in assets on behalf of 12 million retirees and active participants across 847 corporate and public pension plans. Founded in 1994 as a back-office software vendor, it evolved into a full-stack fiduciary platform — handling benefit calculations, investment management, participant communications, and regulatory reporting.

The company sits at the intersection of financial services, insurance, and regulatory compliance. It operates under ERISA (Employee Retirement Income Security Act), SEC registered investment advisor rules, and, increasingly, international pension regulations as it expands globally. Its clients include Fortune 500 defined-benefit pension plans, public employee retirement systems (PERS), and union pension trusts.

NexusWealth's technology stack is a palimpsest of five decades: COBOL flat files from the 1980s, Oracle databases from the 1990s, a Java monolith from the 2000s, and a partially-migrated Node.js/TypeScript microservices layer started in 2019. The engineering team is 180 people. The platform processes $4.2 billion in benefit payments every month.

## Your Role

You are not a new hire here. You arrive as an **external consultant** — a Staff Engineer brought in by NexusWealth's CTO, Diane Yoshida, after a beneficiary dispute exposed a fundamental gap in the platform's data retention architecture. Your engagement is scoped to 90 days, but the problems you find will expand it.

Diane's briefing is short: "We need someone who has seen systems at scale and isn't afraid to tell us what we're doing wrong. Our engineers are brilliant, but they've been inside this thing for too long. They can't see it anymore."

You have no badge, no laptop yet, no access to production. Just a visitor badge and a conference room with whiteboards.

## First Day

The conference room is called "Vanguard." Someone has written on the whiteboard in red marker: **"12M people are depending on this."** Nobody has erased it.

The first meeting is with Robert Osei, the Chief Actuary, and Priya Nair — yes, *that* Priya Nair, now CTO of NexusWealth after you worked with her at Stratum Systems and PrismHealth. She's the one who called you. She's wearing the same focused expression she had the night of the Stratum freight cluster incident, except now she has a Director of Engineering badge and considerably more grey in her hair.

"The COBOL files," she says, sliding a folder across the table. "Start there."

## Arc Themes

- Long-term data persistence across format generations
- Regulatory compliance as first-class architectural constraint (ERISA, DOL, SEC)
- Correctness-first design for fiduciary systems
- Cost modeling for a regulated financial platform
- Multi-jurisdiction data sovereignty
- Architecture designed to outlast the technology it runs on

## Chapters in This Arc

| Chapter | Title | Type | Difficulty |
|---------|-------|------|------------|
| Ch. 230 | Fifty Years of Data | System Design | 8 |
| Ch. 231 | ERISA and SEC Compliance | System Design | 8 |
| Ch. 232 | Actuarial Calculation at Scale | System Design | 7 |
| Ch. 233 | Real-Time Portfolio Risk | System Design | 9 |
| Ch. 234 | SPIKE: Scale 10x — Market Volatility Surge | Incident Response | 9 |
| Ch. 235 | The Board Presentation | Cost Engineering | 9 |
| Ch. 236 | Data Sovereignty for Pension Funds | System Design | 8 |
| Ch. 237 | The 50-Year Architecture | Staff Design | 10 |

**Milestone**: NexusWealth Arc — Chapters 230–237
