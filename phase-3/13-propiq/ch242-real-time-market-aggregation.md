---
**Title**: PropIQ — Chapter 242: The Real-Time Lie
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #data-ingestion #real-time #streaming #hybrid-pipeline #api-integration #etl
**Estimated Time**: 2.5 hours
**Related Chapters**: Ch. 238 (PropIQ property data mesh), Ch. 239 (PropIQ search)
**Exercise Type**: System Design
---

### Story Context

**Email chain — PropIQ internal**

---

**From:** Marcus Tran <marcus.tran@propiq.com>
**To:** Divya Nair <divya.nair@propiq.com>
**CC:** you <you@propiq.com>
**Subject:** Re: Re: Re: Market Pulse SLA Review — ACTION REQUIRED
**Date:** Wednesday, 8:14 AM

Divya,

I've tried to flag this politely three times in sprint reviews. I'm going to be direct now because the Q3 enterprise sales deck has a screenshot of Market Pulse with the tagline "Real-Time Commercial Real Estate Intelligence."

It is not real-time. I need someone with authority to understand what "real-time" actually means in our system, so I'm going to explain it in full.

We have 40 data providers. Here is an honest breakdown of their ingestion latency:

| Provider | Method | Actual Latency |
|---|---|---|
| CoStar | REST API (webhook push) | 8 minutes avg |
| Reonomy | REST API (polling, 15-min intervals) | 15-30 minutes |
| MLS Bright | REST API (polling, 1-hr intervals) | 1-2 hours |
| PropertyShark | FTP batch (daily at 3am EST) | 2-24 hours |
| ATTOM Data | FTP batch (daily at 6am EST) | 2-26 hours |
| NYC DOF Records | Municipal API (polling, 4-hr intervals) | 4-8 hours |
| Austin APD Zoning | PDF email attachment (daily digest) | 4-28 hours |
| TxDOT Traffic Data | FTP weekly | 24-168 hours |
| Orbital Insights (satellite) | S3 batch (weekly) | 7-14 days |
| CoStar Comps | **MANUAL** | Variable |

I want to talk about that last one. "CoStar Comps" is a CoStar sub-product that our contract does not include in their API. Our data contract predates the API availability. What we actually do today: I wrote a cron job in 2021 that:
1. Opens headless Chromium
2. Logs into CoStar using a shared credentials file in a `.env` file on my laptop
3. Navigates to the Comps tab
4. Takes a screenshot
5. Runs Tesseract OCR on the screenshot
6. Parses the OCR output with a regex I wrote over 3 nights
7. Inserts the parsed records into our database

This runs every morning at 6:30 AM from my laptop. If my laptop is asleep, it doesn't run. This data feeds the "recent comparable sales" widget on every property page. We market this as "AI-powered comparable sales intelligence."

The cron job breaks roughly once a month when CoStar changes their UI. Last time it was down for 11 days before I noticed. The "recent comps" were silently serving 11-day-old data.

I've been here 4 years. Nobody has ever asked how this data gets updated. I'm telling you now because we just signed an enterprise deal requiring real-time SLAs and I want it in writing that I flagged this.

Marcus

---

**From:** Divya Nair <divya.nair@propiq.com>
**To:** Marcus Tran <marcus.tran@propiq.com>
**CC:** you <you@propiq.com>
**Subject:** Re: Re: Re: Re: Market Pulse SLA Review — ACTION REQUIRED
**Date:** Wednesday, 9:02 AM

Marcus,

Thank you for being direct. This is a serious problem and I'm not going to minimize it.

A few immediate questions:
1. Does Leila know about the CoStar Comps situation?
2. Have we ever missed an SLA due to any of the FTP providers?
3. Is the CoStar shared credentials technically a violation of their ToS?

I'm adding this to the engineering priorities for this sprint. Please block time with the new Staff Engineer today.

Divya

---

**From:** Marcus Tran <marcus.tran@propiq.com>
**To:** Divya Nair <divya.nair@propiq.com>
**CC:** you <you@propiq.com>
**Subject:** Re: Re: Re: Re: Re: Market Pulse SLA Review — ACTION REQUIRED
**Date:** Wednesday, 9:17 AM

1. No.
2. We haven't tracked it — but yes, almost certainly.
3. Yes. Probably. Almost definitely.

Marcus

---

**From:** Divya Nair <divya.nair@propiq.com>
**To:** you <you@propiq.com>
**Subject:** Fwd: Market Pulse SLA Review — ACTION REQUIRED
**Date:** Wednesday, 9:19 AM

Please read the attached chain and then go talk to Marcus. I need a redesign proposal by end of next week. The Meridian deal has a "real-time market data" clause in the contract. Legal needs to know what we can actually commit to.

Please approach this as: what's the honest SLA we can offer per provider tier, and what does a hybrid ingestion architecture look like that maximizes real-time quality while being honest about its limits?

One constraint: we cannot renegotiate the CoStar contract this quarter. We have to work around it.

Divya

---

**DM: you → Marcus Tran**
_Wednesday, 9:45 AM_

**you:** Hey. I read the chain. Can we grab a room?

**Marcus Tran:** Yeah. I'm in the kitchen. Bring coffee.

---

**PropIQ Kitchen, Wednesday 9:51 AM**

Marcus has his laptop open, showing you the CoStar cron job. It's a 247-line shell script with a Node.js file attached. The Node.js file has a comment at the top: `// TODO: ask engineering to productionize this. Written 2021-03-12.`

**Marcus:** "This has been running for 3 years. Nobody touched it. I productionized it on my own laptop."

**you:** "I'm not here to make you feel bad about it. You were solving a real problem with the resources you had. How often does it break?"

**Marcus:** "Maybe once a month. CoStar redesigned their Comps tab in September — the OCR regex broke. I didn't catch it for 11 days because the cron job wasn't alerting on failure. It just... silently inserted nothing."

**you:** "Silent failure. Classic. Okay, let me think about the architecture problem here. We have a spectrum: at one end, CoStar webhooks — 8-minute latency, fully automated. At the other end, a cron job running on your MacBook that OCRs a screenshot. And we need to provide a coherent 'real-time market data' product across all of them."

**Marcus:** "The FTP providers are actually more reliable than they sound. ATTOM has been delivering that daily file at 6:02 AM EST for two years without missing once. PropertyShark missed one delivery in 18 months. The problem is we pretend they're real-time when they're not."

**you:** "So the solution isn't to make everything faster. The solution is to: one, make each provider tier reliable and monitored. Two, accurately represent data freshness to the UI. And three, for the CoStar Comps problem specifically — find a sustainable path that doesn't involve your laptop."

**Marcus:** "There's a CoStar API upgrade we've been eligible for since 2022. It includes Comps. But it costs $180K/year more. Nobody approved the budget."

**you:** "Okay. We need to put a cost on the current approach. I'll help you model what the screen-scrape approach actually costs us in engineering time, reliability, and business risk. And then we'll present Divya with: here's the tiered architecture, here's the honest SLA per tier, and here's the upgrade cost to eliminate the CoStar Comps risk."

**Marcus:** (closing his laptop) "This is the first time in four years someone has asked how any of this actually works."

---

### Problem Statement

PropIQ's "Market Pulse" feature claims to provide real-time commercial real estate market data. In reality, data ingestion latency ranges from 8 minutes (CoStar webhooks) to 7 days (satellite imagery) to fully manual/unreliable (CoStar Comps via screen-scrape). The current system treats all providers uniformly, displaying all data as "real-time" regardless of its actual freshness.

Design a hybrid ingestion architecture that handles the full spectrum of data provider capabilities — from webhook push to FTP batch to screen-scrape — without compromising data quality, reliability, or honesty about data freshness. The system must provide per-source freshness metadata, eliminate single points of failure (especially the laptop-dependent cron job), and maximize real-time quality for providers that can support it, while accurately representing staleness for those that cannot.

### Explicit Requirements

1. **Provider tier classification**: Classify all 40 providers into tiers based on their push/pull capabilities and minimum achievable latency
2. **Webhook/push ingestion**: Handle real-time webhook push from CoStar and similar providers with sub-10-minute latency
3. **Polling ingestion**: Handle REST API polling with configurable intervals (15 min to 4 hours) with jitter to avoid thundering herd on provider APIs
4. **FTP/SFTP batch ingestion**: Reliable FTP pickup with checksum verification, deduplication, and failure alerting
5. **Manual/unreliable source handling**: Replace the CoStar Comps screen-scrape with a sustainable architecture (no laptop dependencies)
6. **Data freshness metadata**: Every data point must carry a `last_refreshed_at` timestamp and `source_tier` label visible to the UI
7. **Freshness SLA per tier**: Define and enforce per-tier freshness SLAs; alert when a source misses its SLA
8. **Silent failure detection**: Every ingestion path must detect and alert on silent failures (no data = problem, not success)
9. **Deduplication**: Idempotent ingestion — re-delivering the same data must not create duplicate records
10. **Provider health dashboard**: Internal dashboard showing per-provider last delivery time, delivery success rate, current freshness age
11. **Graceful degradation**: If a provider is stale, display stale data with a freshness indicator rather than showing errors or hiding the field

### Hidden Requirements

1. **Hint**: Re-read Marcus's email carefully. The CoStar shared credentials are stored "in a `.env` file on my laptop." This is a credential security crisis independent of the architecture problem. Any solution must include secret management migration — credentials cannot live on a developer's laptop. What are the audit implications if CoStar's ToS prohibits automated screen access and they can trace the scraping to PropIQ?

2. **Hint**: Marcus mentions the FTP provider misses are untracked — "we haven't tracked it, but yes, almost certainly." This means PropIQ may have been serving stale data to enterprise clients who have SLA contracts. Before you build forward, you need a historical freshness audit: what data was actually delivered when? If the Meridian Capital contract has a "real-time market data" clause, this could be a breach of contract issue. The system design must include retroactive audit capability.

3. **Hint**: The CoStar Comps API upgrade costs $180K/year. Your job is to quantify the cost of NOT upgrading: engineering hours (Marcus's time per incident × incidents per year), business risk (what's the revenue loss if a major client catches the data quality issue?), and legal risk. Present this as a build-vs-buy decision with numbers.

### Constraints

- **Providers**: 40 total; ~8 webhook/push, ~15 REST polling, ~12 FTP/SFTP batch, ~4 manual/unreliable, ~1 email attachment (Austin APD)
- **Data volume**: ~500K property records in the system; ~50K updated daily across all providers
- **CoStar webhook volume**: ~2,000 events/hour during business hours; ~200/hour overnight
- **FTP batch size**: ATTOM delivers ~120K records/day as a single CSV (~800 MB compressed)
- **Data freshness SLA to customers**: Tier 1 (webhook) < 15 minutes; Tier 2 (fast polling) < 30 minutes; Tier 3 (slow polling/FTP) displayed with actual age; Tier 4 (manual) displayed as "updated daily, last updated: [timestamp]"
- **Database**: PostgreSQL (RDS) for property records; TimescaleDB for time-series market data
- **Budget for infrastructure**: $5,000/month incremental (current infra already running)
- **CoStar Comps API upgrade**: $180K/year to eliminate screen-scrape dependency
- **Team**: Marcus Tran (data engineer) + 1 backend engineer; 4-week implementation window
- **Existing stack**: AWS (SQS, Lambda, RDS, S3), BullMQ for job queues, Node.js/TypeScript

### Your Task

Design the hybrid ingestion architecture for PropIQ's Market Pulse system. This includes:

1. The provider tier classification system and per-tier ingestion pipeline
2. The replacement for the CoStar Comps screen-scrape (technical path + cost model)
3. The freshness metadata model and per-source SLA enforcement
4. Silent failure detection and alerting per ingestion path
5. The FTP/SFTP reliable batch ingestion pipeline with deduplication
6. The data freshness UI contract (how the frontend represents staleness to users)

### Deliverables

- [ ] **Mermaid architecture diagram**: Full hybrid ingestion pipeline showing all 4 provider tiers and how data flows to the property records database
- [ ] **Provider tier schema**: Classification model for 40 providers with tier assignment, expected delivery interval, last delivery timestamp, SLA definition
- [ ] **Freshness metadata model**: How freshness is stored per property field, not just per provider (a property can have rent roll from Tier 1, zoning from Tier 3)
- [ ] **Scaling estimation**: Show math for event throughput, storage growth, polling API call volume per day
- [ ] **Tradeoff analysis**: Minimum 3 explicit tradeoffs
- [ ] **CoStar Comps cost model**: Build-vs-buy analysis with numbers (current cost of screen-scrape vs. $180K/year API upgrade)
- [ ] **Silent failure detection design**: How does the system know that "no data received" is a failure vs. a quiet period?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

---

#### Scaling Estimation (Starter Math — Complete This)

**Webhook ingestion (Tier 1 — CoStar):**
- 2,000 events/hour peak × 16 business hours = 32,000 events/day
- Plus 200/hour × 8 overnight hours = 1,600 overnight events
- Total: ~33,600 CoStar events/day
- Each event: ~4 KB (property update JSON) → 134 MB/day from CoStar alone
- SQS → Lambda processing: Lambda invocation cost at 33,600/day × $0.0000002 = ~$0.0067/day (negligible)

**REST API polling (Tier 2 — 15 providers, 15-min intervals):**
- 15 providers × (96 polls/day per 15-min interval) = 1,440 API calls/day
- Providers with 1-hour polling: 15 providers × 24 calls/day = 360 calls/day
- Total polling calls: ~4,000/day across all polling providers
- Risk: if all 15 polling providers respond simultaneously (no jitter): thundering herd on provider APIs — add ±120s random jitter per provider

**FTP batch processing (Tier 3):**
- ATTOM: 800 MB compressed CSV daily → ~2.4 GB uncompressed
- PropertyShark: ~400 MB compressed
- Total FTP ingestion: ~1.2 GB compressed / ~4 GB uncompressed per day
- Processing: Lambda with 10 GB memory, 15-min timeout; or ECS Fargate task for large files
- ATTOM 120K records: if each record takes 5ms to validate + upsert → 600 seconds = too slow for Lambda
- Use ECS Fargate task with batch COPY instead: `COPY` 120K rows → ~12 seconds

**Storage growth:**
- 50K property record updates/day × 4 KB average = 200 MB/day in PostgreSQL
- TimescaleDB market data time-series: 50K × 200 bytes = 10 MB/day
- Annual growth: ~73 GB property records + ~3.65 GB time-series
- RDS db.r6g.2xlarge (8 vCPU, 64 GB RAM, 2 TB storage): $800/month — sufficient for 3 years of growth

**CoStar Comps screen-scrape cost model:**
- Marcus time: 1 incident/month × 4 hours to debug/fix × $200/hr effective cost = $800/month
- Missed data: 11 days missed in September = data quality incident; if Meridian Capital catches this and invokes SLA clause → potential contract credit of $50K
- Annual risk-adjusted cost: ($800 × 12) + ($50K × 0.3 probability of client SLA claim) = $9,600 + $15,000 = $24,600/year
- CoStar API upgrade: $180,000/year
- Net cost to upgrade: $180K - $24.6K = $155,400/year
- But: CoStar Comps data quality would improve, enabling upsell; frame as product investment, not pure cost

---

#### Ingestion Tier Design (Starter — Complete This)

```
Tier 1 — Real-Time Push (< 15 min SLA):
  Mechanism: Webhook push → SQS → Lambda → Upsert
  Providers: CoStar, Reonomy (if webhook available)
  Failure mode: Webhook delivery failure → SQS retry with exponential backoff
  Silent failure detection: If no events received in 60 min during business hours → alert

Tier 2 — Scheduled Pull (< 60 min SLA):
  Mechanism: Cron (EventBridge) → Lambda poller → SQS → Lambda processor → Upsert
  Providers: MLS Bright, NYC DOF, and similar
  Failure mode: HTTP 4xx/5xx → DLQ → alert
  Silent failure detection: Expected record count per poll; if count drops > 50% below baseline → alert

Tier 3 — Batch File (displayed with actual age):
  Mechanism: S3 bucket (SFTP gateway or direct upload) → S3 event notification → ECS Fargate task → bulk upsert
  Providers: ATTOM, PropertyShark, TxDOT
  Failure mode: Checksum mismatch → quarantine → alert (do not ingest corrupt data)
  Silent failure detection: If no file received by T+2 hours past expected delivery window → alert

Tier 4 — Manual/Unreliable (displayed as "updated daily"):
  Mechanism: [Design this — what replaces the CoStar Comps cron job on Marcus's laptop?]
  Consider: RPA tool (UI.Vision, Playwright Cloud), CoStar API upgrade, manual data entry workflow with SLA, third-party data broker
```

What is your recommended replacement for the CoStar Comps screen-scrape? Design the decision tree and implementation.
