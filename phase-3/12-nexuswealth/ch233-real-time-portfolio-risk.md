---
**Title**: NexusWealth — Chapter 233: Ten Minutes to Exposure
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #real-time #financial-systems #distributed-computing #risk #streaming #compliance
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 231, Ch. 232, Ch. 234
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**Incident Bridge Log — #incidents — NexusWealth Engineering**

```
[09:14:07] @pagerduty — ALERT: Risk Dashboard P1 — "Portfolio Risk Service: stale data"
[09:14:22] @you — Joining bridge. What's the trigger?
[09:14:31] @marcus.delgado — Investment Committee is convened. SVB contagion fears.
                              Regional bank bond exposure across 50 portfolios. They need
                              current VaR right now.
[09:15:02] @yusuf.okonkwo (Investment Risk, VP) — I have 12 Investment Committee members
                              on a call asking me what our exposure is. I'm looking at the
                              dashboard and it says "Last updated: 47 minutes ago." Is this
                              a system problem or a data problem?
[09:15:19] @you — Investigating. How fast is the market moving?
[09:15:44] @yusuf.okonkwo — 10-year Treasury yield moved 40bps in the last 20 minutes.
                              Regional bank spread widening 150bps. This is not normal
                              volatility. We need live numbers.
[09:16:01] @marcus.delgado — Our risk calculation runs on a 30-minute job schedule.
                              Yusuf, current answer: we cannot give you real-time numbers.
                              Best we can do is trigger a manual recalculation right now,
                              which takes 8 minutes.
[09:16:23] @yusuf.okonkwo — 8 minutes is too long. The market is moving in real-time.
                              By the time we have numbers, they're already stale.
[09:16:40] @you — Triggering manual recalc now. We'll have numbers in ~8 min.
[09:16:55] @yusuf.okonkwo — I'm going to tell the Committee we have a data freshness
                              problem and we're working on it. This is not acceptable for
                              a production risk system.
[09:24:30] @you — Recalc complete. Numbers in dashboard. Updating bridge.
[09:24:45] @yusuf.okonkwo — Total VaR is $340M at 95% confidence. Regional bank exposure
                              is $87M across 8 portfolios. ERISA 404(c) allocation check
                              shows 3 portfolios are within 2% of their fixed income
                              allocation ceiling. I need to know: if yields move another
                              20bps, do we breach?
[09:25:01] @you — I don't have a "what-if" function built. That's a new request.
[09:25:19] @yusuf.okonkwo — It's not new. It's been on the requirements document since
                              March. It was deprioritized.
[09:25:44] @you — Understood. I'll build it. Let's get through today first.
```

---

**1:1 transcript — Yusuf Okonkwo (VP Investment Risk) + you — 3:30 PM same day**

**Yusuf**: I want to be direct with you. We had $87 million of regional bank bond exposure moving in real-time, and our risk system was showing 47-minute-old data. The Investment Committee was making verbal decisions on position limits — they were going off gut feel because our system couldn't tell them anything current. That is an ERISA fiduciary problem, not just a technology problem.

**You**: Walk me through what you actually need. Not what was in the requirements doc — what you actually need when the market is moving.

**Yusuf**: Three things. First, portfolio-level VaR and CVaR updated continuously. Not every 30 minutes. When things are calm, every 5 minutes is fine. When things are stressed — VIX above 25, credit spreads widening — I need sub-60-second recalculation. Second, factor exposures: duration, convexity, credit spread beta, equity beta for each portfolio. These need to update every time a price moves materially. Third — and this is the one that got deprioritized — scenario analysis. "If rates move 50bps up, what happens to each portfolio's allocation?" I need that answered in under 30 seconds.

**You**: How many portfolios?

**Yusuf**: 50 active investment portfolios. But here's the thing — these aren't 50 independent portfolios. They share securities. Portfolio A and Portfolio C might both hold the same corporate bond. When that bond's price updates, both portfolios' risk numbers change simultaneously. Any architecture that treats portfolios as independent silos will miss correlated updates.

**You**: How many unique securities across all 50 portfolios?

**Yusuf**: About 10,000. Of those, maybe 500 are the "hot" ones — they move frequently during stress and they appear in multiple portfolios. The rest are illiquid credit positions that price once a day.

**You**: And during stress, how fast are prices coming in?

**Yusuf**: Bloomberg feed gives us up to 50,000 price updates per minute for liquid securities. For regional bank bonds specifically today, we were getting a new quote every 15 seconds. For Treasuries, every 3 seconds.

**You**: ERISA allocation limits — you mentioned 3 portfolios were within 2% of ceiling. Do you need an alert when a portfolio approaches a limit?

**Yusuf**: I need an alert when a portfolio is within 5% of any regulatory limit. Before it breaches, not after. After is a fiduciary violation. Before I can act.

**You**: How long do I have to build this?

**Yusuf**: The Investment Committee meets again in three weeks. They've asked Priya directly whether we can have a real-time risk dashboard by then. Priya said yes.

---

After the meeting, you open a blank architecture doc. 10,000 securities. 50 portfolios. Up to 1M price updates per hour during stress. Full portfolio recalculation needed in under 5 minutes. Scenario analysis in under 30 seconds. ERISA allocation limits as hard guardrails. And Priya already committed to three weeks.

This is not a reporting problem. This is a real-time computation graph problem.

### Problem Statement

Design a real-time portfolio risk calculation system for NexusWealth's 50 investment portfolios. The system must ingest continuous market price feeds (up to 1M updates/hour during market stress), maintain current risk metrics (VaR, CVaR, factor exposures) for each portfolio with sub-5-minute full recalculation, detect ERISA allocation limit breaches before they occur, and support on-demand scenario analysis ("if rates move X bps, what happens?") with under-30-second response time.

The fundamental challenge: 10,000 securities shared across 50 portfolios. A single security price update may require updating multiple portfolios' risk simultaneously. The system must handle correlated portfolio updates without redundant computation.

### Explicit Requirements

1. Ingest Bloomberg price feed: up to 1M updates/hour during market stress (baseline: 50K/hour)
2. Maintain VaR (95%, 99% confidence) and CVaR for each of 50 portfolios
3. Maintain factor exposures per portfolio: interest rate duration, convexity, credit spread beta, equity beta
4. Full portfolio risk recalculation in under 5 minutes during market stress
5. Alert when any portfolio approaches within 5% of an ERISA allocation limit (before breach)
6. Scenario analysis: "shift rates X bps / credit spreads Y bps" → portfolio impact in under 30 seconds
7. Security-to-portfolio mapping: update all affected portfolios when a shared security prices
8. Adaptive update frequency: 5-minute cycles at baseline, sub-60-second cycles when VIX > 25
9. Historical VaR series retention for regulatory reporting (ERISA requires investment performance records)
10. Audit log of all risk calculations for DOL examination

### Hidden Requirements

- **Hint**: Re-read Yusuf's comment: "these aren't 50 independent portfolios — they share securities." He says "any architecture that treats portfolios as independent silos will miss correlated updates." There is a dependency graph problem here: a change in one security should trigger recalculation for every portfolio holding that security. How do you maintain this mapping efficiently and update it when portfolio holdings change?
- **Hint**: Re-read Yusuf's final requirement about ERISA allocation limits. He says "Before it breaches, not after. After is a fiduciary violation." An ERISA allocation limit breach that goes undetected is not just a technology failure — it's a legal fiduciary violation that must be reported to participants. What does your monitoring system need to guarantee about the detection latency? And what happens to the alert if the monitoring service is down during a breach?
- **Hint**: Re-read the incident log. The market moved 40bps in 20 minutes during the incident. The current system showed 47-minute-old data. By the time the 8-minute manual recalculation completed, the data was already stale again. What does this tell you about the fundamental architecture flaw — and how does your streaming design ensure that recalculation always uses the latest prices at the moment the calculation runs, not at the moment it was scheduled?
- **Hint**: Yusuf mentions "illiquid credit positions that price once a day." These are likely Level 3 fair-value instruments under ASC 820 — they require documented valuation methodology and human sign-off. Your architecture must distinguish between market-priced securities (can be updated automatically) and internally-valued securities (require human-approved marks). This distinction is never stated as a requirement — it's buried in the portfolio composition detail.

### Constraints

- **Portfolios**: 50 active investment portfolios
- **Unique securities**: ~10,000 (500 "hot" liquid, 9,500 illiquid)
- **Price update rate**: 50K/hour baseline, up to 1M/hour during market stress
- **Correlated portfolios**: average security appears in 4-6 portfolios
- **VaR calculation**: requires 250 days of return history per security (historical simulation method)
- **Full recalculation SLA**: under 5 minutes during stress
- **Scenario analysis SLA**: under 30 seconds on-demand
- **ERISA alert SLA**: before allocation breach, not after
- **VaR history retention**: indefinite (ERISA investment records requirement)
- **Team**: 3 engineers
- **Timeline**: 3 weeks (Priya has already committed to Investment Committee)

### Your Task

Design the real-time portfolio risk calculation architecture. Address: price feed ingestion, security-to-portfolio dependency graph maintenance, incremental vs. full recalculation strategy, VaR/CVaR computation approach (historical simulation vs. parametric vs. Monte Carlo), scenario analysis engine, and ERISA allocation limit monitoring with pre-breach alerting.

### Deliverables

- [ ] Mermaid architecture diagram — price feed ingestion through risk dashboard, including scenario analysis path and ERISA alert path
- [ ] Database schema — portfolio holdings, security-to-portfolio mapping index, risk calculation results time-series, ERISA limit definitions, scenario definitions
- [ ] Scaling estimation:
  - Message throughput: 1M price updates/hour → events/second → ingestion partition count
  - Incremental recalculation: if a single security updates, how many portfolio recalcs are triggered? What is the total computation load per second during peak?
  - Historical simulation VaR: 250 days × 10,000 securities × 50 portfolios = how many data points? Storage estimate.
  - Scenario analysis: brute-force vs. precomputed sensitivity approach — latency and storage tradeoff
- [ ] Tradeoff analysis (minimum 3):
  - VaR method: historical simulation (accurate, slow) vs. parametric (fast, assumes normality) vs. Monte Carlo (flexible, expensive)
  - Recalculation granularity: recalculate on every price tick vs. batch updates every N seconds vs. smart invalidation
  - Dependency graph: in-memory graph per service instance vs. shared Redis graph vs. event-driven fan-out
- [ ] Cost modeling: streaming infrastructure (Kafka/Kinesis), compute for VaR calculations, time-series storage — $X/month estimate
- [ ] Capacity planning: what does this architecture support if NexusWealth grows to 200 portfolios and 50,000 securities in 18 months?

### Diagram Format

Mermaid syntax.
