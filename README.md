[![Stars](https://img.shields.io/github/stars/[YOUR-USERNAME]/the-staff-path?style=flat-square)](https://github.com/[YOUR-USERNAME]/the-staff-path/stargazers)  [![Forks](https://img.shields.io/github/forks/[YOUR-USERNAME]/the-staff-path?style=flat-square)](https://github.com/[YOUR-USERNAME]/the-staff-path/network/members)  [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](./LICENSE)  [![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](./CONTRIBUTING.md)

# The Staff Path
### Story-Driven System Design — Mid to Staff
*257 challenges built around real companies, real failures, real trade-offs*

The Staff Path is an open-source, self-directed system design curriculum that takes you from Mid-Level to Staff Engineer across 3 phases, 38 companies, and 20+ industries. Every chapter opens with a story — a Slack thread, an incident log, a 1:1 with a mentor — and drops you into a real constraint: a compliance audit, a production outage, a budget conversation with the CFO. You design your way out. There is no answer key.

---

## Why This Exists

I was looking ofr a good system design resource all over the internet lately that would not require me breaking the bank. The problem was that most system design resources were either too expensive or they teach you to perform in an interview, not think like a Staff engineer in production.

Staff work is messier than the canonical resources suggest. Requirements are buried in Slack threads. Compliance shows up after the architecture is half-built. The previous engineer left no documentation. Your design has to survive a budget conversation, a regulatory audit, and a 3x growth projection — simultaneously.

The gap between "I can draw the Twitter architecture" and "I can reason about what breaks at scale under real constraints" is the gap The Staff Path is designed to close.

What makes it different: you follow a single protagonist through 38 companies across 3 phases. The same systems — payments, notifications, auth, search, rate limiting — come back every 15–20 chapters, harder each time. By the third appearance, you understand the pattern deeply enough to design variations you have never seen before. This is the Repetition Engine, and it is the core pedagogy of the curriculum.

It is open-sourced so anyone can use it. Contributions are welcome.

---

## Curriculum at a Glance

| | |
|---|---|
| **Chapters** | 250+ across 3 phases |
| **Companies** | 38 real-world-inspired companies |
| **Industries** | 20+ (fintech, healthtech, defense, climate, quantum, maritime, agriculture, pension, legal AI, and more) |
| **Exercise types** | System Design, Debugging, Performance, Incident Response, System Evolution |
| **Compliance coverage** | GDPR, HIPAA, PCI-DSS, SOC2, FDA 510(k), 21 CFR Part 11, 42 CFR Part 2, ERISA, CAISO, ITAR, FedRAMP, NIST IAL2 |
| **Promotion Checkpoints** | 6 gates (Checkpoints 1–5 + Final Staff Interview) |
| **Pattern Summaries** | 18 synthesis chapters |

---

## Prerequisites

You are ready for The Staff Path if you:

- Are comfortable writing and deploying backend services (any language; Node.js/TypeScript is the primary lens)
- Understand what a database, cache, and message queue do — conceptually, not just syntactically
- Have read basic SQL and REST API design
- Can read a Kafka consumer or Redis cache-aside pattern without it being entirely new

You do **not** need distributed systems experience, cloud architecture experience, or prior interview prep.

You are **not** ready if you have never deployed a backend service. Build something first, then come back.

---

## How to Use This Curriculum

1. **Fork this repo.** Your progress lives in your fork.
2. **Clone your fork locally.**
3. **Open a chapter file.** Read the Story Context fully — all of it, including Slack threads and email chains.
4. **Close the file.** Design your solution on paper, a whiteboard, or Excalidraw. No peeking.
5. **Copy `SOLUTION_TEMPLATE.md`** into a `solutions/` folder in your fork, named `chXX-solution.md`.
6. **Fill it in:** architecture diagram (ASCII or link), database schema, scaling math, trade-offs, hidden requirements you found.
7. **Check off the chapter checkbox** in your fork's README.
8. Move to the next chapter.

> **There is no reference solution.** This is intentional. Real Staff engineers do not get an answer key. The value is in the struggle and the iteration. A chapter you completed imperfectly and reflected on honestly is worth more than one you copied.

---

## Recommended Study Plan

**Weekdays** (1 hour/day): 1 chapter every 2–3 days. Read the story fully, sketch deliverables.

**Weekends** (2–4 hours):
- Saturday: Complete deliverables for the week's hardest chapter. Draw the diagram. Do the scaling math. Write the trade-off analysis.
- Sunday: Review the most recent Pattern Summary chapter. Revisit any chapter marked "Revisit" on your GitHub board.

**Spike chapters** (marked in story but not telegraphed in title): budget 2–3 hours regardless of day.

**Promotion Checkpoints**: block a 3-hour session. Treat it like a real interview. No notes, no hints.

| Phase | Approximate Timeline |
|-------|---------------------|
| Phase 1 (60 chapters) | 3–4 months at recommended pace |
| Phase 2 (82 chapters) | 4–5 months |
| Phase 3 (115 chapters) | 6–7 months |

Do not skip Pattern Summary chapters. They are the synthesis layer.

---

## Promotion Levels

| Level | Phase | What You Can Do at the End |
|-------|-------|--------------------------|
| L1: Mid-Level | Phase 1 start | Structured problem-solving, basic trade-offs, back-of-envelope sizing |
| L2: Senior | Phase 1 end | Scale estimation, failure modes, distributed systems basics, compliance awareness |
| L3: Strong Senior | Phase 2 end | Multi-region design, compliance architecture, platform thinking, cost modeling |
| L4: Staff | Phase 3 end | Org-level design, blameless reviews, cost modeling, RFC writing, political navigation |

Every 15–20 chapters there is a **Promotion Checkpoint** — an ambiguous, end-to-end design challenge framed as a job interview. These are gates, not optional exercises.

---

## Full Table of Contents

## Phase 1 — Mid to Senior (Chapters 1–63)
*12 companies · 60 chapters · Foundations of scalable systems*

<details>
<summary>
  <strong>Company 01 — NovaPay</strong> · Fintech / Payment Processing · Chapters 1–5
  <br><em>A Nigerian payments startup onboarding its first enterprise client. You join as their first backend hire the week everything breaks.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 01 | [Design the Payment Processing Pipeline](./phase-1/01-novapay/ch01-payment-processing-pipeline.md) | Mid | 4/10 | 2h | [#payments](#tag-payments) [#api-design](#tag-api-design) [#database](#tag-database) [#idempotency](#tag-idempotency) [#distributed](#tag-distributed) | Async payment pipeline decoupling initiation from bank API calls, handling 8,000 RPM peak |
| [ ] | 02 | [Production is DOWN — Duplicate Charges at Scale](./phase-1/01-novapay/ch02-idempotency-and-duplicate-prevention.md) | Mid | 6/10 | 3h | [#payments](#tag-payments) [#idempotency](#tag-idempotency) [#incident-response](#tag-incident-response) [#distributed](#tag-distributed) [#database](#tag-database) | Incident response: preventing duplicate charges when connection pool exhausts under load |
| [ ] | 03 | [The Slow Query Nobody Wrote a Ticket For](./phase-1/01-novapay/ch03-database-indexing-for-financial-queries.md) | Mid | 5/10 | 2h | [#database](#tag-database) [#indexing](#tag-indexing) [#performance](#tag-performance) [#sql](#tag-sql) [#payments](#tag-payments) | Performance investigation: debugging slow merchant dashboard queries on a 14M-row table |
| [ ] | 04 | [Design the Merchant Balance Cache](./phase-1/01-novapay/ch04-write-through-cache-design.md) | Mid | 5/10 | 2h | [#caching](#tag-caching) [#redis](#tag-redis) [#database](#tag-database) [#consistency](#tag-consistency) [#payments](#tag-payments) | Write-through cache for merchant balances with two staleness tolerances: settled vs available |
| [ ] | 05 | [Design Review Ambush — PCI-DSS or the Audit Fails](./phase-1/01-novapay/ch05-pci-dss-compliance-architecture.md) | Senior | 7/10 | 3h | [#compliance](#tag-compliance) [#pci-dss](#tag-pci-dss) [#security](#tag-security) [#architecture](#tag-architecture) [#payments](#tag-payments) [#audit](#tag-audit) | PCI-DSS compliance architecture under design-review ambush before scheduled audit |

> **Special Files**: [Company Intro](./phase-1/01-novapay/intro.md) · [Transition Brief](./phase-1/01-novapay/transition.md)

</details>

<details>
<summary>
  <strong>Company 02 — MeridianHealth</strong> · Healthtech / EHR Integration · Chapters 6–10
  <br><em>A health information exchange integrating 23 hospital networks. You inherit a platform already under audit.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 06 | [Design the Authentication and Authorization System](./phase-1/02-meridian-health/ch06-jwt-oauth2-auth-system.md) | Mid | 5/10 | 2.5h | [#auth](#tag-auth) [#jwt](#tag-jwt) [#oauth2](#tag-oauth2) [#hipaa](#tag-hipaa) [#security](#tag-security) [#api-design](#tag-api-design) | JWT/OAuth2/OIDC auth system supporting three hospital networks with federated SSO |
| [ ] | 07 | [Build the HIPAA-Compliant Audit Trail](./phase-1/02-meridian-health/ch07-hipaa-audit-trails.md) | Senior | 6/10 | 2.5h | [#hipaa](#tag-hipaa) [#audit](#tag-audit) [#compliance](#tag-compliance) [#database](#tag-database) [#observability](#tag-observability) [#security](#tag-security) | HIPAA-compliant append-only audit trail for PHI access with tamper-evidence and 6-year retention |
| [ ] | 08 | [Data Residency — The State That Wants Its Data Back](./phase-1/02-meridian-health/ch08-data-residency-and-hipaa-architecture.md) | Senior | 7/10 | 3h | [#hipaa](#tag-hipaa) [#data-residency](#tag-data-residency) [#compliance](#tag-compliance) [#distributed](#tag-distributed) [#database](#tag-database) [#security](#tag-security) | Data residency architecture for HIPAA and Texas state law compliance across geographic regions |
| [ ] | 09 | [Scale the EHR Query Layer with Read Replicas](./phase-1/02-meridian-health/ch09-read-replicas-and-query-routing.md) | Senior | 6/10 | 2.5h | [#database](#tag-database) [#read-replicas](#tag-read-replicas) [#performance](#tag-performance) [#sql](#tag-sql) [#connection-pooling](#tag-connection-pooling) [#hipaa](#tag-hipaa) | Read replica routing and connection pooling for 500 concurrent clinical users |
| [ ] | 10 | [Scale 10x by Friday — The Northview Surge](./phase-1/02-meridian-health/ch10-ehr-integration-platform-design.md) | Senior | 8/10 | 3.5h | [#distributed](#tag-distributed) [#hipaa](#tag-hipaa) [#api-design](#tag-api-design) [#database](#tag-database) [#scaling](#tag-scaling) [#system-evolution](#tag-system-evolution) | Emergency 6-week platform scale for 2.1M-patient network with 400ms P99 SLA constraint |

> **Special Files**: [Company Intro](./phase-1/02-meridian-health/intro.md) · [Transition Brief](./phase-1/02-meridian-health/transition.md)

</details>

<details>
<summary>
  <strong>Company 03 — VeloTrack</strong> · Logistics / Delivery Tracking · Chapters 11–15
  <br><em>A gig-economy delivery platform handling 100K drivers in real-time. Your Kafka consumer just entered a crash loop.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 11 | [Untangle the Kafka Mess](./phase-1/03-velotrack/ch11-kafka-deep-dive-consumer-groups.md) | Senior | 6/10 | 3h | [#kafka](#tag-kafka) [#messaging](#tag-messaging) [#consumer-groups](#tag-consumer-groups) [#distributed](#tag-distributed) [#real-time](#tag-real-time) | Kafka consumer group redesign separating analytics from real-time tracking pipelines |
| [ ] | 12 | [Production is DOWN — The Notification Consumer Won't Stop Crashing](./phase-1/03-velotrack/ch12-dead-letter-queues-and-error-handling.md) | Senior | 7/10 | 3h | [#kafka](#tag-kafka) [#dlq](#tag-dlq) [#error-handling](#tag-error-handling) [#incident-response](#tag-incident-response) [#messaging](#tag-messaging) [#distributed](#tag-distributed) | Incident response: notification consumer crash-loop from malformed APNs payloads and missing DLQ |
| [ ] | 13 | [Design the Real-Time Driver Tracking System](./phase-1/03-velotrack/ch13-real-time-delivery-tracking.md) | Senior | 6/10 | 3h | [#real-time](#tag-real-time) [#kafka](#tag-kafka) [#redis](#tag-redis) [#geospatial](#tag-geospatial) [#websockets](#tag-websockets) [#distributed](#tag-distributed) | Real-time driver tracking ingesting 100K location updates/sec fanning out to 200K WebSockets |
| [ ] | 14 | [Design the Load Balancing Strategy for a Heterogeneous Fleet](./phase-1/03-velotrack/ch14-load-balancing-strategies.md) | Senior | 6/10 | 2.5h | [#load-balancing](#tag-load-balancing) [#distributed](#tag-distributed) [#api-design](#tag-api-design) [#performance](#tag-performance) [#websockets](#tag-websockets) | Load balancing strategy: round-robin for stateless ingestion, sticky sessions for WebSockets |
| [ ] | 15 | [The Delivery State Machine at Scale](./phase-1/03-velotrack/ch15-kafka-exactly-once-and-delivery-state-machine.md) | Strong Senior | 8/10 | 3.5h | [#kafka](#tag-kafka) [#state-machine](#tag-state-machine) [#idempotency](#tag-idempotency) [#distributed](#tag-distributed) [#exactly-once](#tag-exactly-once) [#database](#tag-database) | Delivery state machine with exactly-once Kafka semantics, preventing duplicate state transitions |

> **Special Files**: [Company Intro](./phase-1/03-velotrack/intro.md) · [Transition Brief](./phase-1/03-velotrack/transition.md)

</details>

---

**[Pattern Summary 1 — Data Layer & Caching Patterns](./phase-1/pattern-summary-01.md)** · Chapters 1–15
> Idempotency, write-through caching, read replicas, database indexing, DLQ lifecycle, Kafka consumer groups, partition key design, compliance architecture.

> ⚠️ **[CHECKPOINT 1 — Mid → Senior Interview @ Helios Systems](./phase-1/checkpoint-01.md)** — *Promotion Gate*
> Multi-currency payment ledger design. Marcus Webb is on the panel. This is not optional.

<details>
<summary>
  <strong>Company 04 — Beacon Media</strong> · Streaming / Media Platform · Chapters 18–22
  <br><em>A streaming video platform serving 28M users. You inherit a CDN that costs $2M/month more than it should.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 18 | [Fix the CDN — $2M/Month and Falling](./phase-1/04-beacon-media/ch18-blob-storage-and-cdn-architecture.md) | Senior | 7/10 | 3h | [#cdn](#tag-cdn) [#storage](#tag-storage) [#performance](#tag-performance) [#cost-optimization](#tag-cost-optimization) [#distributed](#tag-distributed) | CDN optimization: raising cache hit rate from 54% to 92%, saving $822K/month |
| [ ] | 19 | [Presigned URLs and the Pirates Problem](./phase-1/04-beacon-media/ch19-presigned-urls-and-content-security.md) | Senior | 6/10 | 2.5h | [#storage](#tag-storage) [#security](#tag-security) [#cdn](#tag-cdn) [#presigned-urls](#tag-presigned-urls) [#drm](#tag-drm) [#api-design](#tag-api-design) | Presigned URL architecture closing piracy vector while preserving CDN performance for 28M users |
| [ ] | 20 | [Design the Notification System (First Appearance)](./phase-1/04-beacon-media/ch20-notification-system-first-appearance.md) | Senior | 6/10 | 3h | [#notifications](#tag-notifications) [#fan-out](#tag-fan-out) [#push](#tag-push) [#email](#tag-email) [#system-design](#tag-system-design) [#kafka](#tag-kafka) | Notification system handling 8.5M simultaneous pushes during Champions League event |
| [ ] | 21 | [Scale 10x — The New Content Deal is Going to Break Transcoding](./phase-1/04-beacon-media/ch21-video-transcoding-pipeline.md) | Strong Senior | 8/10 | 3h | [#storage](#tag-storage) [#distributed](#tag-distributed) [#scaling](#tag-scaling) [#queues](#tag-queues) [#cost-optimization](#tag-cost-optimization) [#data-pipeline](#tag-data-pipeline) | Transcoding pipeline scale-up for 20K videos in 72 hours with BullMQ spot-instance management |
| [ ] | 22 | [The Recommendation API is Melting](./phase-1/04-beacon-media/ch22-content-recommendation-and-caching.md) | Strong Senior | 7/10 | 3h | [#caching](#tag-caching) [#api-design](#tag-api-design) [#performance](#tag-performance) [#cache-invalidation](#tag-cache-invalidation) [#scaling](#tag-scaling) | Recommendation cache stampede prevention for 6.8M concurrent post-live-event users |

> **Special Files**: [Company Intro](./phase-1/04-beacon-media/intro.md) · [Transition Brief](./phase-1/04-beacon-media/transition.md)

</details>

<details>
<summary>
  <strong>Company 05 — AgroSense</strong> · IoT / Agricultural Sensors · Chapters 23–26
  <br><em>A precision agriculture startup monitoring 2M sensors across 15,000 farms. The time-series database is collapsing.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 23 | [Design the Time-Series Sensor Data Platform](./phase-1/05-agrosense/ch23-time-series-data-design.md) | Senior | 7/10 | 3h | [#time-series](#tag-time-series) [#iot](#tag-iot) [#database](#tag-database) [#performance](#tag-performance) [#data-pipeline](#tag-data-pipeline) | TimescaleDB migration for 640M sensor readings/day from PostgreSQL with query performance fixes |
| [ ] | 24 | [Build the ETL Pipeline and Change Data Capture](./phase-1/05-agrosense/ch24-etl-pipeline-and-cdc.md) | Senior | 7/10 | 3h | [#etl](#tag-etl) [#cdc](#tag-cdc) [#data-pipeline](#tag-data-pipeline) [#kafka](#tag-kafka) [#observability](#tag-observability) [#database](#tag-database) | ETL to real-time streaming pipeline evolution for agricultural alerts with <3-minute latency |
| [ ] | 25 | [Build Observability Into the Sensor Platform](./phase-1/05-agrosense/ch25-observability-and-monitoring.md) | Senior | 6/10 | 2.5h | [#observability](#tag-observability) [#monitoring](#tag-monitoring) [#alerting](#tag-alerting) [#structured-logging](#tag-structured-logging) [#metrics](#tag-metrics) | Observability design: structured logging, Kafka consumer lag, and runbook-driven alerting |
| [ ] | 26 | [Design for Unreliable Sensors at the Edge](./phase-1/05-agrosense/ch26-edge-processing-and-sensor-reliability.md) | Strong Senior | 7/10 | 3h | [#iot](#tag-iot) [#edge-computing](#tag-edge-computing) [#reliability](#tag-reliability) [#distributed](#tag-distributed) [#data-quality](#tag-data-quality) [#gaps](#tag-gaps) | Edge sensor reliability for battery-powered devices with offline bursts and data corruption handling |

> **Special Files**: [Company Intro](./phase-1/05-agrosense/intro.md) · [Transition Brief](./phase-1/05-agrosense/transition.md)

</details>

---

**[Pattern Summary 2 — Async, Queues & Event-Driven Systems](./phase-1/pattern-summary-02.md)** · Chapters 11–26
> Consumer group isolation, DLQ lifecycle, fan-out patterns, event-time vs processing-time, ETL→streaming evolution, CDC, Kafka partition keys.

<details>
<summary>
  <strong>Company 06 — CloudStack</strong> · B2B SaaS / Developer Platform · Chapters 28–32
  <br><em>A developer platform serving 3,200 enterprise tenants. The noisy neighbor problem just hit your largest customer.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 28 | [Multi-Tenant Architecture and Tenant Isolation](./phase-1/06-cloudstack/ch28-multi-tenancy-architecture.md) | Senior | 7/10 | 3h | [#multi-tenancy](#tag-multi-tenancy) [#security](#tag-security) [#distributed](#tag-distributed) [#database](#tag-database) [#architecture](#tag-architecture) | Multi-tenancy isolation across data, performance, failure, and information leak layers |
| [ ] | 29 | [Design the API Gateway and Rate Limiting System (First Appearance)](./phase-1/06-cloudstack/ch29-api-gateway-and-rate-limiting.md) | Senior | 7/10 | 3h | [#rate-limiting](#tag-rate-limiting) [#api-gateway](#tag-api-gateway) [#distributed](#tag-distributed) [#redis](#tag-redis) [#multi-tenancy](#tag-multi-tenancy) | API gateway and token-bucket rate limiting for multi-tier SaaS with per-tenant configuration |
| [ ] | 30 | [Auth at Scale — SSO, Service Accounts, and API Keys (Second Appearance)](./phase-1/06-cloudstack/ch30-auth-system-second-appearance.md) | Strong Senior | 7/10 | 3h | [#auth](#tag-auth) [#oauth2](#tag-oauth2) [#sso](#tag-sso) [#api-keys](#tag-api-keys) [#multi-tenancy](#tag-multi-tenancy) [#security](#tag-security) | Auth at scale: SAML/OIDC federation with SCIM auto-provisioning for enterprise multi-tenancy |
| [ ] | 31 | [Design the Tenant Billing and Usage Metering System](./phase-1/06-cloudstack/ch31-tenant-billing-and-usage-metering.md) | Strong Senior | 8/10 | 3h | [#billing](#tag-billing) [#metering](#tag-metering) [#distributed](#tag-distributed) [#database](#tag-database) [#idempotency](#tag-idempotency) [#multi-tenancy](#tag-multi-tenancy) | Billing audit redesign: idempotent metering with append-only audit trail after $14K overcharge |
| [ ] | 32 | [Solve the Noisy Neighbor Problem for Good](./phase-1/06-cloudstack/ch32-noisy-neighbor-and-resource-quotas.md) | Strong Senior | 8/10 | 3h | [#multi-tenancy](#tag-multi-tenancy) [#distributed](#tag-distributed) [#kubernetes](#tag-kubernetes) [#performance](#tag-performance) [#isolation](#tag-isolation) [#system-evolution](#tag-system-evolution) | Noisy neighbor prevention across compute, database, and information layers with resource quotas |

> **Special Files**: [Company Intro](./phase-1/06-cloudstack/intro.md) · [Transition Brief](./phase-1/06-cloudstack/transition.md)

</details>

<details>
<summary>
  <strong>Company 07 — PulseCommerce</strong> · E-commerce / B2B Marketplace · Chapters 33–37
  <br><em>A B2B marketplace with 12M SKUs. The search is slow, inventory inconsistent, and it is flash sale day.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 33 | [Design the Product Search System (First Appearance)](./phase-1/07-pulsecommerce/ch33-search-inverted-index-elasticsearch.md) | Strong Senior | 8/10 | 3.5h | [#search](#tag-search) [#elasticsearch](#tag-elasticsearch) [#inverted-index](#tag-inverted-index) [#relevance](#tag-relevance) [#distributed](#tag-distributed) [#scaling](#tag-scaling) | Product search migration from PostgreSQL full-text to Elasticsearch with facets and synonyms |
| [ ] | 34 | [Payment Integration at Scale — Scaling Challenges (Second Appearance)](./phase-1/07-pulsecommerce/ch34-payment-processing-second-appearance.md) | Strong Senior | 8/10 | 3h | [#payments](#tag-payments) [#distributed](#tag-distributed) [#idempotency](#tag-idempotency) [#system-evolution](#tag-system-evolution) [#scaling](#tag-scaling) [#compliance](#tag-compliance) | Payment infrastructure optimization reducing 8.2% failure rate through smart retry and fraud scoring |
| [ ] | 35 | [The N+1 Query Disaster Eating Your Database](./phase-1/07-pulsecommerce/ch35-n-plus-one-queries-and-dataloader.md) | Senior | 6/10 | 2.5h | [#database](#tag-database) [#n-plus-one](#tag-n-plus-one) [#performance](#tag-performance) [#dataloader](#tag-dataloader) [#sql](#tag-sql) [#optimization](#tag-optimization) | N+1 query fix via DataLoader, reducing 1,800ms catalog response to 90ms baseline |
| [ ] | 36 | [Cache Invalidation Hell](./phase-1/07-pulsecommerce/ch36-cache-invalidation-patterns.md) | Strong Senior | 8/10 | 3h | [#caching](#tag-caching) [#cache-invalidation](#tag-cache-invalidation) [#redis](#tag-redis) [#consistency](#tag-consistency) [#distributed](#tag-distributed) | Event-driven cache invalidation across 4-layer cache stack (ES, Redis, CDN, in-memory) |
| [ ] | 37 | [Flash Sale — Inventory Overselling Under Pressure](./phase-1/07-pulsecommerce/ch37-inventory-management-and-consistency.md) | Strong Senior | 9/10 | 4h | [#database](#tag-database) [#consistency](#tag-consistency) [#distributed](#tag-distributed) [#race-conditions](#tag-race-conditions) [#inventory](#tag-inventory) [#scaling](#tag-scaling) | Flash sale inventory: race condition causing -1,847 unit stock at 50K concurrent checkouts |

> **Special Files**: [Company Intro](./phase-1/07-pulsecommerce/intro.md) · [Transition Brief](./phase-1/07-pulsecommerce/transition.md)

</details>

---

**[Pattern Summary 3 — Caching, Consistency & Cache Invalidation](./phase-1/pattern-summary-03.md)** · Chapters 28–37
> Staleness tolerance, write-through/behind/cache-aside, event-driven invalidation, thundering herd prevention, CDN cache strategies.

<details>
<summary>
  <strong>Company 08 — CivicOS</strong> · Government / Digital Public Services · Chapters 39–42
  <br><em>A GovTech platform serving 23 state governments. A federal audit found compliance gaps nobody knew existed.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 39 | [Design the Data Sovereignty Architecture for Government Clients](./phase-1/08-civicos/ch39-data-sovereignty-and-compliance-architecture.md) | Strong Senior | 8/10 | 3.5h | [#compliance](#tag-compliance) [#data-sovereignty](#tag-data-sovereignty) [#government](#tag-government) [#fedramp](#tag-fedramp) [#security](#tag-security) [#architecture](#tag-architecture) | Data sovereignty and FedRAMP architecture for 23 state governments with geo-restricted CDN |
| [ ] | 40 | [Rate Limiting for Government APIs — The Enumeration Attack](./phase-1/08-civicos/ch40-rate-limiting-second-appearance.md) | Strong Senior | 8/10 | 3h | [#rate-limiting](#tag-rate-limiting) [#security](#tag-security) [#api-design](#tag-api-design) [#government](#tag-government) [#distributed](#tag-distributed) [#redis](#tag-redis) | Distributed rate limiting against SSN enumeration attack via behavioral fingerprinting |
| [ ] | 41 | [SSO for 23 State Governments — Federated Identity at Scale](./phase-1/08-civicos/ch41-sso-and-identity-federation-for-government.md) | Strong Senior | 8/10 | 3h | [#auth](#tag-auth) [#sso](#tag-sso) [#saml](#tag-saml) [#government](#tag-government) [#fedramp](#tag-fedramp) [#multi-tenancy](#tag-multi-tenancy) [#identity](#tag-identity) | SSO for 23 state governments with SAML/OIDC and NIST IAL2 identity proofing |
| [ ] | 42 | [Design for Accessibility, Resilience, and the Citizen Who Can't Retry](./phase-1/08-civicos/ch42-accessibility-and-resilience-for-government-systems.md) | Strong Senior | 7/10 | 3h | [#resilience](#tag-resilience) [#accessibility](#tag-accessibility) [#government](#tag-government) [#distributed](#tag-distributed) [#api-design](#tag-api-design) [#system-design](#tag-system-design) | Accessibility and resilience for elderly citizens on slow connections, Section 508 compliance |

> **Special Files**: [Company Intro](./phase-1/08-civicos/intro.md) · [Transition Brief](./phase-1/08-civicos/transition.md)

</details>

<details>
<summary>
  <strong>Company 09 — NeuroLearn</strong> · Education / Academic Platform · Chapters 43–47
  <br><em>An adaptive learning platform serving 6M students. Finals week started and the database is at 98% CPU.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 43 | [Notification System at Scale — The Finals Week Problem (Second Appearance)](./phase-1/09-neurolearn/ch43-notification-system-second-appearance.md) | Strong Senior | 8/10 | 3.5h | [#notifications](#tag-notifications) [#fan-out](#tag-fan-out) [#scaling](#tag-scaling) [#push](#tag-push) [#email](#tag-email) [#distributed](#tag-distributed) | Notification scaling for 6M exam reminders in 90 minutes with pre-staging and provider coordination |
| [ ] | 44 | [Database Collapse During Finals Week — Read Replicas at Scale](./phase-1/09-neurolearn/ch44-read-replicas-and-connection-pooling-second-appearance.md) | Strong Senior | 7/10 | 2.5h | [#database](#tag-database) [#read-replicas](#tag-read-replicas) [#connection-pooling](#tag-connection-pooling) [#performance](#tag-performance) [#scaling](#tag-scaling) | Read replica routing during finals-week collapse: write-your-own-reads vs safe replica queries |
| [ ] | 45 | [Search Over Academic Content — Scaling Challenges (Second Appearance)](./phase-1/09-neurolearn/ch45-search-second-appearance.md) | Strong Senior | 7/10 | 3h | [#search](#tag-search) [#elasticsearch](#tag-elasticsearch) [#relevance](#tag-relevance) [#academic](#tag-academic) [#distributed](#tag-distributed) [#scaling](#tag-scaling) | Academic search for 48M documents with institution scoping, language, and level-based scoring |
| [ ] | 46 | [Design the Adaptive Quiz Engine at Scale](./phase-1/09-neurolearn/ch46-adaptive-quiz-engine-design.md) | Strong Senior | 8/10 | 3.5h | [#distributed](#tag-distributed) [#state-machine](#tag-state-machine) [#database](#tag-database) [#real-time](#tag-real-time) [#algorithm](#tag-algorithm) [#consistency](#tag-consistency) | Adaptive quiz engine: Redis session state and cached question bank for 40K concurrent sessions |
| [ ] | 47 | [Design Review Ambush — The Academic Integrity Architecture](./phase-1/09-neurolearn/ch47-academic-integrity-monitoring.md) | Strong Senior | 8/10 | 3.5h | [#system-design](#tag-system-design) [#compliance](#tag-compliance) [#real-time](#tag-real-time) [#database](#tag-database) [#privacy](#tag-privacy) [#education](#tag-education) | Academic integrity monitoring balancing surveillance, GDPR/FERPA, and algorithmic fairness |

> **Special Files**: [Company Intro](./phase-1/09-neurolearn/intro.md) · [Transition Brief](./phase-1/09-neurolearn/transition.md)

</details>

---

**[Pattern Summary 4 — API Design, Rate Limiting & Load Distribution](./phase-1/pattern-summary-04.md)** · Chapters 28–47
> Rate limiting threat models, three algorithms, multi-tenant APIs, auth vs authorization, response normalization, search as a system.

<details>
<summary>
  <strong>Company 10 — OmniLogix</strong> · Supply Chain / Global Logistics · Chapters 49–53
  <br><em>A global supply chain platform. A double-allocation incident just cost $240K across 14 data centers.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 49 | [CAP Theorem in Practice — When the Network Fails](./phase-1/10-omnilogix/ch49-cap-theorem-and-eventual-consistency.md) | Strong Senior | 9/10 | 4h | [#distributed](#tag-distributed) [#cap-theorem](#tag-cap-theorem) [#eventual-consistency](#tag-eventual-consistency) [#consistency](#tag-consistency) [#availability](#tag-availability) | CAP theorem: $240K double-allocation incident requiring per-operation consistency tiers |
| [ ] | 50 | [Design the Multi-Step Supply Chain Transaction with Saga](./phase-1/10-omnilogix/ch50-distributed-transactions-and-saga-pattern.md) | Strong Senior | 9/10 | 4h | [#distributed](#tag-distributed) [#saga](#tag-saga) [#transactions](#tag-transactions) [#consistency](#tag-consistency) [#idempotency](#tag-idempotency) [#supply-chain](#tag-supply-chain) | Saga pattern for 7-step supply chain transactions with compensating actions at scale |
| [ ] | 51 | [Idempotency at Scale — When Retries Become Dangerous](./phase-1/10-omnilogix/ch51-idempotency-at-scale.md) | Strong Senior | 8/10 | 3.5h | [#idempotency](#tag-idempotency) [#distributed](#tag-distributed) [#retries](#tag-retries) [#supply-chain](#tag-supply-chain) [#exactly-once](#tag-exactly-once) [#deduplication](#tag-deduplication) | Idempotency at scale: detection separate from execution, preventing $84K supplier triple-billing |
| [ ] | 52 | [Kafka Exactly-Once at the Application Level](./phase-1/10-omnilogix/ch52-kafka-exactly-once-application-level.md) | Strong Senior | 9/10 | 4h | [#kafka](#tag-kafka) [#exactly-once](#tag-exactly-once) [#idempotency](#tag-idempotency) [#outbox-pattern](#tag-outbox-pattern) [#distributed](#tag-distributed) [#supply-chain](#tag-supply-chain) | Kafka exactly-once at application level via outbox pattern for saga orchestration |
| [ ] | 53 | [Global Inventory Consistency — The Full Design](./phase-1/10-omnilogix/ch53-global-inventory-consistency-synthesis.md) | Strong Senior | 10/10 | 5h | [#distributed](#tag-distributed) [#cap-theorem](#tag-cap-theorem) [#eventual-consistency](#tag-eventual-consistency) [#saga](#tag-saga) [#idempotency](#tag-idempotency) [#supply-chain](#tag-supply-chain) [#synthesis](#tag-synthesis) | Synthesis: CAP theorem, saga, and idempotency combined for global inventory consistency |

> **Special Files**: [Company Intro](./phase-1/10-omnilogix/intro.md) · [Transition Brief](./phase-1/10-omnilogix/transition.md)

</details>

<details>
<summary>
  <strong>Company 11 — LuminaryAI</strong> · ML Infrastructure / Marketing Personalization · Chapters 54–58
  <br><em>An ML serving platform delivering 140M predictions/day. The feature store is drifting and nobody noticed.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 54 | [The Feature Store — Feeding Models the Right Data](./phase-1/11-luminaryai/ch54-feature-store-and-ml-pipeline.md) | Strong Senior | 8/10 | 4h | [#ml-systems](#tag-ml-systems) [#feature-store](#tag-feature-store) [#data-pipeline](#tag-data-pipeline) [#caching](#tag-caching) [#eventual-consistency](#tag-eventual-consistency) | Feature store with dual write paths (batch + real-time) for ML serving with <5ms latency |
| [ ] | 55 | [ETL to Streaming — Evolving the Data Pipeline](./phase-1/11-luminaryai/ch55-etl-to-streaming-pipeline-evolution.md) | Strong Senior | 8/10 | 3.5h | [#data-pipeline](#tag-data-pipeline) [#etl](#tag-etl) [#streaming](#tag-streaming) [#kafka](#tag-kafka) [#cdc](#tag-cdc) [#debezium](#tag-debezium) [#ml-systems](#tag-ml-systems) | ETL to streaming via Debezium CDC, handling 14K out-of-stock recommendations during outage |
| [ ] | 56 | [Payment Infrastructure — Billing the ML Platform](./phase-1/11-luminaryai/ch56-payment-processing-third-appearance.md) | Strong Senior | 7/10 | 3h | [#payments](#tag-payments) [#billing](#tag-billing) [#metering](#tag-metering) [#ml-systems](#tag-ml-systems) [#idempotency](#tag-idempotency) [#pci-dss](#tag-pci-dss) | Usage-based billing for 140M predictions/day with Stripe integration and immutable audit trail |
| [ ] | 57 | [Advanced Observability — When Models Lie Silently](./phase-1/11-luminaryai/ch57-advanced-observability-ml-systems.md) | Strong Senior | 8/10 | 3.5h | [#observability](#tag-observability) [#ml-systems](#tag-ml-systems) [#monitoring](#tag-monitoring) [#alerting](#tag-alerting) [#data-quality](#tag-data-quality) [#metrics](#tag-metrics) | ML observability detecting silent feature degradation via three-layer model monitoring |
| [ ] | 58 | [SPIKE — Scale 10x by Friday](./phase-1/11-luminaryai/ch58-recommendation-serving-scale.md) | Strong Senior | 9/10 | 4h | [#scaling](#tag-scaling) [#ml-systems](#tag-ml-systems) [#caching](#tag-caching) [#load-balancing](#tag-load-balancing) [#capacity-planning](#tag-capacity-planning) [#spike-scale10x](#tag-spike-scale10x) | Scale 10x in 9 days: 140M to 1.5B predictions/day with Redis expansion and batch pre-compute |

> **Special Files**: [Company Intro](./phase-1/11-luminaryai/intro.md) · [Transition Brief](./phase-1/11-luminaryai/transition.md)

</details>

---

**[Pattern Summary 5 — Distributed Systems Fundamentals](./phase-1/pattern-summary-05.md)** · Chapters 49–58
> CAP per-operation, saga pattern, dual-requirement idempotency, Kafka three-layer exactly-once, feature stores, ML observability layers, batch pipeline SPOF.

<details>
<summary>
  <strong>Company 12 — SkyRoute</strong> · Aviation / Airline Reservation & Operations · Chapters 59–63
  <br><em>An airline reservation platform. You are called in to redesign notifications, search, auth, and rate limiting simultaneously.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 59 | [Notification System — 8 Million in 40 Minutes](./phase-1/12-skyroute/ch59-notification-system-third-appearance.md) | Strong Senior | 10/10 | 5h | [#notifications](#tag-notifications) [#fan-out](#tag-fan-out) [#kafka](#tag-kafka) [#apns](#tag-apns) [#fcm](#tag-fcm) [#rate-limiting](#tag-rate-limiting) [#third-appearance](#tag-third-appearance) | Notification cascade failure: 8M notifications in 40 minutes with APNs rate limiting and priority tiers |
| [ ] | 60 | [Flight Search — The Hardest Search Problem](./phase-1/12-skyroute/ch60-flight-search-third-appearance.md) | Strong Senior | 9/10 | 4h | [#search](#tag-search) [#elasticsearch](#tag-elasticsearch) [#multi-criteria](#tag-multi-criteria) [#real-time-inventory](#tag-real-time-inventory) [#third-appearance](#tag-third-appearance) | Flight search phantom availability: 7% booking failures from stale seat inventory |
| [ ] | 61 | [Auth for the Most Adversarial User Base](./phase-1/12-skyroute/ch61-auth-third-appearance.md) | Strong Senior | 9/10 | 4h | [#auth](#tag-auth) [#oauth2](#tag-oauth2) [#travel-agent-bots](#tag-travel-agent-bots) [#identity](#tag-identity) [#fraud](#tag-fraud) [#third-appearance](#tag-third-appearance) | Auth for three distinct populations (passengers, agents, GDS) with behavioral fraud detection |
| [ ] | 62 | [Rate Limiting — Defending Against the GDS Giants](./phase-1/12-skyroute/ch62-rate-limiting-third-appearance.md) | Strong Senior | 8/10 | 3.5h | [#rate-limiting](#tag-rate-limiting) [#distributed](#tag-distributed) [#gds](#tag-gds) [#travel-agents](#tag-travel-agents) [#third-appearance](#tag-third-appearance) [#redis](#tag-redis) | Rate limiting with contractual partner SLA overrides: 48K RPS from GDS vs bot mitigation |
| [ ] | 63 | [Seat Inventory — The Overbooking Problem](./phase-1/12-skyroute/ch63-seat-inventory-and-overbooking.md) | Strong Senior | 9/10 | 4h | [#inventory](#tag-inventory) [#consistency](#tag-consistency) [#overbooking](#tag-overbooking) [#distributed](#tag-distributed) [#concurrency](#tag-concurrency) [#saga](#tag-saga) | Seat overbooking: triple-booked seat across three regional GDS endpoints via replica lag |

> **Special Files**: [Company Intro](./phase-1/12-skyroute/intro.md) · [Transition Brief](./phase-1/12-skyroute/transition.md)

</details>

---

> ⚠️ **[CHECKPOINT 2 — Senior → Strong Senior Interview @ Stratum Systems](./phase-1/checkpoint-02.md)** — *Promotion Gate — End of Phase 1*
> 18M shipment events/day, real-time tracking, clock skew problem. You are alone. The panel is tough.

---

## Phase 2 — Senior to Strong Senior (Chapters 64–149)
*13 companies · 82 chapters · Distributed systems depth and production mastery*

<details>
<summary>
  <strong>Company 01 — Stratum Systems</strong> · Logistics / Global Freight Intelligence · Chapters 64–69
  <br><em>A freight intelligence company tracking 18M shipment events/day. The distributed consensus model is broken and clock skew is corrupting order.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 64 | [Distributed Consensus and Raft](./phase-2/01-stratum-systems/ch64-distributed-consensus-and-raft.md) | Strong Senior | 7/10 | 3h | [#distributed-systems](#tag-distributed-systems) [#consensus](#tag-consensus) [#raft](#tag-raft) [#leader-election](#tag-leader-election) [#fencing-tokens](#tag-fencing-tokens) | Raft-based distributed consensus for freight event ordering across four data centers |
| [ ] | 65 | [Event Ordering and Lamport Timestamps](./phase-2/01-stratum-systems/ch65-event-ordering-and-lamport-timestamps.md) | Strong Senior | 8/10 | 3h | [#distributed-systems](#tag-distributed-systems) [#event-ordering](#tag-event-ordering) [#lamport-clocks](#tag-lamport-clocks) [#vector-clocks](#tag-vector-clocks) [#causal-consistency](#tag-causal-consistency) | Lamport timestamps and causal ordering for cross-DC event sequencing with clock skew |
| [ ] | 66 | [CQRS and Event Sourcing](./phase-2/01-stratum-systems/ch66-cqrs-and-event-sourcing.md) | Strong Senior | 7/10 | 3h | [#cqrs](#tag-cqrs) [#event-sourcing](#tag-event-sourcing) [#projections](#tag-projections) [#distributed-systems](#tag-distributed-systems) [#eventual-consistency](#tag-eventual-consistency) | CQRS and event sourcing for separated read/write paths in shipment state system |
| [ ] | 67 | [Database Internals — WAL and MVCC](./phase-2/01-stratum-systems/ch67-database-internals-wal-and-mvcc.md) | Strong Senior | 8/10 | 3h | [#database-internals](#tag-database-internals) [#wal](#tag-wal) [#mvcc](#tag-mvcc) [#postgres](#tag-postgres) [#transactions](#tag-transactions) [#isolation-levels](#tag-isolation-levels) | PostgreSQL WAL and MVCC internals: autovacuum contention on CQRS projection tables |
| [ ] | 68 | [Multi-Region Active-Active Architecture](./phase-2/01-stratum-systems/ch68-multi-region-active-active-intro.md) | Strong Senior | 8/10 | 4h | [#multi-region](#tag-multi-region) [#active-active](#tag-active-active) [#distributed-systems](#tag-distributed-systems) [#data-residency](#tag-data-residency) [#conflict-resolution](#tag-conflict-resolution) | Multi-region active-active architecture for 14 DCs with conflict resolution by data type |
| [ ] | 69 | [Distributed Tracing with OpenTelemetry](./phase-2/01-stratum-systems/ch69-distributed-tracing-intro.md) | Strong Senior | 7/10 | 3h | [#observability](#tag-observability) [#distributed-tracing](#tag-distributed-tracing) [#opentelemetry](#tag-opentelemetry) [#jaeger](#tag-jaeger) [#sampling](#tag-sampling) | OpenTelemetry distributed tracing across 8 services with non-W3C partner APIs |

> **Special Files**: [Company Intro](./phase-2/01-stratum-systems/intro.md) · [Transition Brief](./phase-2/01-stratum-systems/transition.md)

</details>

<details>
<summary>
  <strong>Company 02 — NexaCare</strong> · Healthtech / Clinical Trial Management · Chapters 70–75
  <br><em>A clinical trial management platform operating in 14 countries. GDPR deletion requests are 3 weeks backlogged.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 70 | [GDPR Deletion Pipelines](./phase-2/02-nexacare/ch70-gdpr-deletion-pipelines.md) | Strong Senior | 7/10 | 3h | [#gdpr](#tag-gdpr) [#compliance](#tag-compliance) [#event-sourcing](#tag-event-sourcing) [#data-deletion](#tag-data-deletion) [#privacy](#tag-privacy) [#cryptographic-erasure](#tag-cryptographic-erasure) | GDPR deletion pipelines with cryptographic erasure for 24-hour SLA in healthcare |
| [ ] | 71 | [Database Internals — B-Tree vs LSM-Tree](./phase-2/02-nexacare/ch71-database-internals-btree-vs-lsm.md) | Strong Senior | 8/10 | 3h | [#database-internals](#tag-database-internals) [#btree](#tag-btree) [#lsm-tree](#tag-lsm-tree) [#rocksdb](#tag-rocksdb) [#write-amplification](#tag-write-amplification) [#compaction](#tag-compaction) | B-tree vs LSM-tree internals for 4M writes/day clinical trial observation data |
| [ ] | 72 | [Advanced Audit Trails at Scale](./phase-2/02-nexacare/ch72-advanced-audit-trails-at-scale.md) | Strong Senior | 7/10 | 3h | [#audit-trails](#tag-audit-trails) [#compliance](#tag-compliance) [#fda](#tag-fda) [#append-only](#tag-append-only) [#cryptographic-chaining](#tag-cryptographic-chaining) [#21cfr11](#tag-21cfr11) | Cryptographically-chained audit trails with tampering detection for FDA Part 11 |
| [ ] | 73 | [Data Residency Across Multiple Jurisdictions](./phase-2/02-nexacare/ch73-data-residency-multi-jurisdiction.md) | Strong Senior | 8/10 | 4h | [#data-residency](#tag-data-residency) [#gdpr](#tag-gdpr) [#compliance](#tag-compliance) [#multi-region](#tag-multi-region) [#data-sovereignty](#tag-data-sovereignty) [#appi](#tag-appi) [#pipeda](#tag-pipeda) | Multi-jurisdiction data residency for 14 countries with federated analytics |
| [ ] | 74 | [Advanced Observability — SLOs, SLIs, and Error Budgets](./phase-2/02-nexacare/ch74-advanced-observability-slos.md) | Strong Senior | 7/10 | 3h | [#observability](#tag-observability) [#slo](#tag-slo) [#sli](#tag-sli) [#error-budget](#tag-error-budget) [#sre](#tag-sre) [#alerting](#tag-alerting) [#toil](#tag-toil) | SLO/SLI framework and error budgets for healthcare observability with alert fatigue |
| [ ] | 75 | [Secret Management with HashiCorp Vault](./phase-2/02-nexacare/ch75-secret-management-intro.md) | Strong Senior | 7/10 | 3h | [#security](#tag-security) [#secret-management](#tag-secret-management) [#vault](#tag-vault) [#hashicorp](#tag-hashicorp) [#secret-rotation](#tag-secret-rotation) [#audit](#tag-audit) | HashiCorp Vault secret management eliminating 340 static secrets across services |

> **Special Files**: [Company Intro](./phase-2/02-nexacare/intro.md) · [Transition Brief](./phase-2/02-nexacare/transition.md)

</details>

---

**[Pattern Summary 6 — Distributed Consensus & Consistency](./phase-2/pattern-summary-06.md)** · Chapters 64–75
> Raft leader election, fencing tokens, CRDT intro, bi-temporal queries, event sourcing replay, WAL/MVCC internals, multi-region conflict resolution.

<details>
<summary>
  <strong>Company 03 — TradeSpark</strong> · Fintech / Retail Algorithmic Trading · Chapters 77–83
  <br><em>A retail algorithmic trading platform. The LSM-tree compaction storm hits every market open.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 77 | [Low-Latency Event Sourcing for Trade Execution](./phase-2/03-tradespark/ch77-low-latency-event-sourcing.md) | Strong Senior | 8/10 | 3h | [#event-sourcing](#tag-event-sourcing) [#cqrs](#tag-cqrs) [#trading](#tag-trading) [#low-latency](#tag-low-latency) [#append-only](#tag-append-only) [#temporal-queries](#tag-temporal-queries) | Low-latency event-sourced order management for trading with sub-50µs write latency |
| [ ] | 78 | [Database Internals — LSM-Tree Compaction Storm](./phase-2/03-tradespark/ch78-database-internals-deep-dive.md) | Staff | 9/10 | 4h | [#database-internals](#tag-database-internals) [#rocksdb](#tag-rocksdb) [#lsm-tree](#tag-lsm-tree) [#compaction](#tag-compaction) [#write-stall](#tag-write-stall) [#performance](#tag-performance) | RocksDB compaction storm prevention at market open with 180K writes/sec burst |
| [ ] | 79 | [Advanced Kafka — Log Compaction for Order Book State](./phase-2/03-tradespark/ch79-advanced-kafka-log-compaction.md) | Strong Senior | 7/10 | 3h | [#kafka](#tag-kafka) [#log-compaction](#tag-log-compaction) [#consumer-lag](#tag-consumer-lag) [#partitioning](#tag-partitioning) [#trading](#tag-trading) [#order-book](#tag-order-book) | Kafka log compaction for order book state with dual-topic archival strategy |
| [ ] | 80 | [Distributed Transactions — 2PC vs Saga](./phase-2/03-tradespark/ch80-distributed-transactions-2pc-vs-saga.md) | Staff | 9/10 | 4h | [#distributed-transactions](#tag-distributed-transactions) [#2pc](#tag-2pc) [#saga](#tag-saga) [#settlement](#tag-settlement) [#trading](#tag-trading) [#compensating-transactions](#tag-compensating-transactions) | Distributed transactions: 2PC vs saga pattern for multi-leg trading settlement |
| [ ] | 81 | [DTCC Clearing Integration and Financial Settlement](./phase-2/03-tradespark/ch81-payments-fourth-appearance.md) | Staff | 9/10 | 4h | [#payments](#tag-payments) [#settlement](#tag-settlement) [#dtcc](#tag-dtcc) [#idempotency](#tag-idempotency) [#pci-dss](#tag-pci-dss) [#financial](#tag-financial) [#clearing](#tag-clearing) | DTCC clearing integration with deterministic idempotency keys and T+1 settlement |
| [ ] | 82 | [Advanced Caching — Probabilistic Early Expiration](./phase-2/03-tradespark/ch82-advanced-caching-probabilistic-expiration.md) | Strong Senior | 7/10 | 3h | [#caching](#tag-caching) [#cache-stampede](#tag-cache-stampede) [#probabilistic-expiration](#tag-probabilistic-expiration) [#redis](#tag-redis) [#market-data](#tag-market-data) [#thundering-herd](#tag-thundering-herd) | Probabilistic early cache expiration (XFetch) to prevent stampede at market open |
| [ ] | 83 | [Service Mesh and mTLS Introduction](./phase-2/03-tradespark/ch83-service-mesh-mtls-intro.md) | Strong Senior | 7/10 | 3h | [#service-mesh](#tag-service-mesh) [#mtls](#tag-mtls) [#istio](#tag-istio) [#security](#tag-security) [#circuit-breakers](#tag-circuit-breakers) [#linkerd](#tag-linkerd) | Service mesh and mTLS introduction for financial data security without hot-path latency |

> **Special Files**: [Company Intro](./phase-2/03-tradespark/intro.md) · [Transition Brief](./phase-2/03-tradespark/transition.md)

</details>

<details>
<summary>
  <strong>Company 04 — GigGrid</strong> · Creator Economy / Digital Goods Marketplace · Chapters 84–89
  <br><em>A digital goods marketplace serving 2M gig workers. Multi-region conflicts are causing double job assignments.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 84 | [Multi-Region Conflict Resolution](./phase-2/04-giggrid/ch84-multi-region-conflict-resolution.md) | Strong Senior | 8/10 | 3h | [#multi-region](#tag-multi-region) [#active-active](#tag-active-active) [#conflict-resolution](#tag-conflict-resolution) [#distributed-systems](#tag-distributed-systems) [#worker-assignment](#tag-worker-assignment) | Multi-region conflict resolution for job assignment preventing double-assignments |
| [ ] | 85 | [CRDTs — Conflict-Free Replicated Data Types](./phase-2/04-giggrid/ch85-crdts-introduction.md) | Staff | 9/10 | 4h | [#crdt](#tag-crdt) [#distributed-systems](#tag-distributed-systems) [#eventual-consistency](#tag-eventual-consistency) [#conflict-free](#tag-conflict-free) [#availability](#tag-availability) | CRDTs for worker availability with explicit intent overriding system inference |
| [ ] | 86 | [Real-Time at Scale — WebSockets, SSE, and Pub/Sub](./phase-2/04-giggrid/ch86-real-time-at-scale-websockets-sse.md) | Strong Senior | 8/10 | 3h | [#realtime](#tag-realtime) [#websockets](#tag-websockets) [#sse](#tag-sse) [#pubsub](#tag-pubsub) [#redis](#tag-redis) [#long-polling](#tag-long-polling) [#backpressure](#tag-backpressure) | Real-time job dispatch using WebSockets/SSE with 2M concurrent workers |
| [ ] | 87 | [Platform API Rate Limiting (4th Appearance)](./phase-2/04-giggrid/ch87-rate-limiting-fourth-appearance.md) | Strong Senior | 7/10 | 3h | [#rate-limiting](#tag-rate-limiting) [#fairness](#tag-fairness) [#sliding-window](#tag-sliding-window) [#distributed](#tag-distributed) [#redis](#tag-redis) [#api-gateway](#tag-api-gateway) | Weighted fair queuing for API rate limiting to prevent market-dominant bot hoarding |
| [ ] | 88 | [Canary Deployments and Feature Flags](./phase-2/04-giggrid/ch88-canary-deployments-and-feature-flags.md) | Strong Senior | 7/10 | 3h | [#deployment](#tag-deployment) [#canary](#tag-canary) [#feature-flags](#tag-feature-flags) [#blue-green](#tag-blue-green) [#rollback](#tag-rollback) [#automation](#tag-automation) | Canary deployments and feature flags with mobile API client version handling |
| [ ] | 89 | [Advanced Search — Facets and Typeahead](./phase-2/04-giggrid/ch89-advanced-search-facets-typeahead.md) | Strong Senior | 7/10 | 3h | [#search](#tag-search) [#elasticsearch](#tag-elasticsearch) [#facets](#tag-facets) [#typeahead](#tag-typeahead) [#geosearch](#tag-geosearch) [#relevance](#tag-relevance) | Elasticsearch job search with facets, typeahead, and geo-distance scoring |

> **Special Files**: [Company Intro](./phase-2/04-giggrid/intro.md) · [Transition Brief](./phase-2/04-giggrid/transition.md)

</details>

---

> ⚠️ **[CHECKPOINT 3 — Senior → Strong Senior Gate @ Apex Systems](./phase-2/checkpoint-03.md)** — *Promotion Gate*
> Real-time fraud signal aggregation: 8M transactions/hour, sub-50ms fraud score delivery. No Marcus Webb.

**[Pattern Summary 7 — Service Mesh, Zero-Trust & Security Patterns](./phase-2/pattern-summary-07.md)** · Chapters 77–91
> mTLS sidecar model, zero-trust BeyondCorp, secret rotation, OAuth2 at machine scale, audit log tamper-evidence, behavioral anomaly detection, circuit breakers at infra level.

<details>
<summary>
  <strong>Company 05 — SentinelOps</strong> · Cybersecurity / Threat Detection & Response · Chapters 92–98
  <br><em>A threat detection platform processing 40B security events/day. The FBI needs a forensic-grade evidence pipeline.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 92 | [Zero-Trust Architecture](./phase-2/05-sentinelops/ch92-zero-trust-architecture.md) | Staff | 9/10 | 4h | [#security](#tag-security) [#zero-trust](#tag-zero-trust) [#network-segmentation](#tag-network-segmentation) [#blast-radius](#tag-blast-radius) [#mtls](#tag-mtls) [#kubernetes](#tag-kubernetes) | Zero-trust architecture preventing lateral movement with Kubernetes NetworkPolicy |
| [ ] | 93 | [Service Mesh Deep Dive — Istio in Production](./phase-2/05-sentinelops/ch93-service-mesh-deep-dive.md) | Staff | 9/10 | 5h | [#service-mesh](#tag-service-mesh) [#istio](#tag-istio) [#mtls](#tag-mtls) [#zero-trust](#tag-zero-trust) [#distributed-systems](#tag-distributed-systems) [#observability](#tag-observability) [#security](#tag-security) | Istio production deployment with AuthorizationPolicy and mTLS migration plan |
| [ ] | 94 | [Advanced Authorization — RBAC vs ABAC and Policy-as-Code](./phase-2/05-sentinelops/ch94-auth-fourth-appearance.md) | Staff | 8/10 | 4h | [#auth](#tag-auth) [#rbac](#tag-rbac) [#abac](#tag-abac) [#opa](#tag-opa) [#policy-as-code](#tag-policy-as-code) [#multi-tenant](#tag-multi-tenant) [#security](#tag-security) [#compliance](#tag-compliance) | ABAC with OPA policy-as-code replacing broken RBAC in multi-tenant security platform |
| [ ] | 95 | [Secret Management at Scale](./phase-2/05-sentinelops/ch95-secret-management-at-scale.md) | Staff | 8/10 | 4h | [#vault](#tag-vault) [#secret-management](#tag-secret-management) [#security](#tag-security) [#distributed-systems](#tag-distributed-systems) [#database](#tag-database) [#compliance](#tag-compliance) [#zero-trust](#tag-zero-trust) | Vault at 99.999% availability with Raft clusters, dynamic secrets, and break-glass procedures |
| [ ] | 96 | [Advanced Observability — Distributed Tracing at 50M Spans/Day](./phase-2/05-sentinelops/ch96-advanced-observability-distributed-tracing.md) | Staff | 8/10 | 4h | [#observability](#tag-observability) [#distributed-tracing](#tag-distributed-tracing) [#opentelemetry](#tag-opentelemetry) [#jaeger](#tag-jaeger) [#grafana-tempo](#tag-grafana-tempo) [#cost-engineering](#tag-cost-engineering) [#sampling](#tag-sampling) | Tail-based sampling for OpenTelemetry to reduce $280K/month trace storage costs |
| [ ] | 97 | [Security Incident Forensics Pipeline](./phase-2/05-sentinelops/ch97-security-incident-forensics-pipeline.md) | Staff | 9/10 | 5h | [#incident-response](#tag-incident-response) [#forensics](#tag-forensics) [#immutable-storage](#tag-immutable-storage) [#worm](#tag-worm) [#evidence-preservation](#tag-evidence-preservation) [#security](#tag-security) [#compliance](#tag-compliance) [#spike](#tag-spike) | Forensic log pipeline with WORM storage and cryptographic hash chain for FBI evidence |
| [ ] | 98 | [Kafka at Scale — 40 Billion Security Events/Day](./phase-2/05-sentinelops/ch98-advanced-kafka-security-events.md) | Staff | 9/10 | 5h | [#kafka](#tag-kafka) [#distributed-systems](#tag-distributed-systems) [#event-streaming](#tag-event-streaming) [#consumer-lag](#tag-consumer-lag) [#partitioning](#tag-partitioning) [#capacity-planning](#tag-capacity-planning) [#performance](#tag-performance) | Advanced Kafka partitioning for 40B security events/day with consumer group isolation |

> **Special Files**: [Company Intro](./phase-2/05-sentinelops/intro.md) · [Transition Brief](./phase-2/05-sentinelops/transition.md)

</details>

<details>
<summary>
  <strong>Company 06 — LightspeedRetail</strong> · E-commerce / Omnichannel Retail · Chapters 99–104
  <br><em>An omnichannel retail platform serving 120K POS terminals. Black Friday is in 10 days.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 99 | [POS Terminal Payments and P2PE](./phase-2/06-lightspeedretail/ch99-payments-fifth-appearance.md) | Staff | 9/10 | 5h | [#payments](#tag-payments) [#pci-dss](#tag-pci-dss) [#idempotency](#tag-idempotency) [#offline](#tag-offline) [#p2pe](#tag-p2pe) [#reconciliation](#tag-reconciliation) | POS terminal offline dedup with deterministic idempotency keys for 120K terminals |
| [ ] | 100 | [Advanced Caching — Black Friday Thundering Herd](./phase-2/06-lightspeedretail/ch100-advanced-caching-thundering-herd.md) | Staff | 9/10 | 5h | [#caching](#tag-caching) [#thundering-herd](#tag-thundering-herd) [#redis](#tag-redis) [#black-friday](#tag-black-friday) [#distributed-lock](#tag-distributed-lock) [#cache-stampede](#tag-cache-stampede) | Thundering herd prevention via TTL jitter for POS catalog cache at 5x Black Friday scale |
| [ ] | 101 | [Immutable Infrastructure and Black Friday Scaling](./phase-2/06-lightspeedretail/ch101-immutable-infrastructure-and-scaling.md) | Staff | 8/10 | 5h | [#immutable-infra](#tag-immutable-infra) [#auto-scaling](#tag-auto-scaling) [#spot-instances](#tag-spot-instances) [#terraform](#tag-terraform) [#packer](#tag-packer) [#iac](#tag-iac) [#black-friday](#tag-black-friday) | Immutable infrastructure and auto-scaling for POS terminals 50K → 250K RPS in 10 days |
| [ ] | 102 | [Zero-Downtime Deploys for 120,000 POS Terminals](./phase-2/06-lightspeedretail/ch102-blue-green-deployments.md) | Staff | 8/10 | 4h | [#blue-green](#tag-blue-green) [#deployments](#tag-deployments) [#database-migrations](#tag-database-migrations) [#zero-downtime](#tag-zero-downtime) [#canary](#tag-canary) [#rollback](#tag-rollback) | Zero-downtime blue-green deploys with schema migration discipline for 120K POS terminals |
| [ ] | 103 | [Data Warehousing — OLAP vs OLTP](./phase-2/06-lightspeedretail/ch103-data-warehousing-olap-vs-oltp.md) | Strong Senior | 7/10 | 4h | [#data-warehousing](#tag-data-warehousing) [#olap](#tag-olap) [#oltp](#tag-oltp) [#redshift](#tag-redshift) [#parquet](#tag-parquet) [#star-schema](#tag-star-schema) [#etl](#tag-etl) [#analytics](#tag-analytics) | OLAP separation from OLTP: migrating 4.2B-row analytics out of production Postgres |
| [ ] | 104 | [CQRS Advanced Projections — Real-Time Inventory](./phase-2/06-lightspeedretail/ch104-cqrs-advanced-projections.md) | Staff | 8/10 | 5h | [#cqrs](#tag-cqrs) [#event-sourcing](#tag-event-sourcing) [#inventory](#tag-inventory) [#projections](#tag-projections) [#eventual-consistency](#tag-eventual-consistency) [#snapshot](#tag-snapshot) | CQRS advanced projections for real-time inventory under high write contention |

> **Special Files**: [Company Intro](./phase-2/06-lightspeedretail/intro.md) · [Transition Brief](./phase-2/06-lightspeedretail/transition.md)

</details>

---

**[Pattern Summary 8 — Data Warehousing, Streaming & Cost Patterns](./phase-2/pattern-summary-08.md)** · Chapters 92–104
> Columnar storage, Parquet/Iceberg, Flink watermarks, SLO design, capacity planning math, showback vs chargeback, FinOps idle resource detection.

<details>
<summary>
  <strong>Company 07 — Crestline Energy</strong> · Energy / European Smart Grid Operations · Chapters 106–112
  <br><em>A smart grid operations platform monitoring 50M smart meters across Europe. Ofgem wants millisecond precision on historical queries.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 106 | [Time-Series at Scale — 50M Meters, 200M Events Per Hour](./phase-2/07-crestline-energy/ch106-time-series-at-scale-advanced.md) | Strong Senior | 8/10 | 4h | [#time-series](#tag-time-series) [#influxdb](#tag-influxdb) [#timescaledb](#tag-timescaledb) [#data-tiering](#tag-data-tiering) [#downsampling](#tag-downsampling) [#iot](#tag-iot) [#database-internals](#tag-database-internals) | Time-series at scale: TimescaleDB, data tiering, hot/warm/cold for 50M smart meters |
| [ ] | 107 | [Grid Analytics — Apache Iceberg and Columnar Storage](./phase-2/07-crestline-energy/ch107-data-warehousing-columnar-storage.md) | Staff | 7/10 | 4h | [#data-warehousing](#tag-data-warehousing) [#apache-iceberg](#tag-apache-iceberg) [#parquet](#tag-parquet) [#presto](#tag-presto) [#olap](#tag-olap) [#energy-analytics](#tag-energy-analytics) [#columnar](#tag-columnar) | Columnar storage and Apache Iceberg data warehousing for 47TB energy meter platform |
| [ ] | 108 | [The 18-Hour Batch Job](./phase-2/07-crestline-energy/ch108-streaming-vs-batch-flink.md) | Staff | 9/10 | 5h | [#flink](#tag-flink) [#streaming](#tag-streaming) [#batch-processing](#tag-batch-processing) [#watermarks](#tag-watermarks) [#late-arriving-events](#tag-late-arriving-events) [#spike-inherited-disaster](#tag-spike-inherited-disaster) [#apache-flink](#tag-apache-flink) | Apache Flink streaming vs batch: windowing, watermarks, out-of-order energy events |
| [ ] | 109 | [Grid Reliability SLOs — When 99.9% Is Not Good Enough](./phase-2/07-crestline-energy/ch109-slo-sli-error-budget-design.md) | Staff | 8/10 | 4h | [#slo](#tag-slo) [#sli](#tag-sli) [#error-budget](#tag-error-budget) [#reliability](#tag-reliability) [#sre](#tag-sre) [#alerting](#tag-alerting) [#toil](#tag-toil) [#energy](#tag-energy) | Grid reliability SLOs where 99.9% is insufficient and symptoms matter over causes |
| [ ] | 110 | [Three Years of Growth, Two Years of Budget](./phase-2/07-crestline-energy/ch110-capacity-planning-and-forecasting.md) | Staff | 8/10 | 4h | [#capacity-planning](#tag-capacity-planning) [#cost-modeling](#tag-cost-modeling) [#finops](#tag-finops) [#forecasting](#tag-forecasting) [#reserved-instances](#tag-reserved-instances) [#spot-instances](#tag-spot-instances) [#infrastructure](#tag-infrastructure) | Three-year capacity planning: €3.4M cost trajectory with reserved instance optimization |
| [ ] | 111 | [Energy Data Sovereignty — When Regulators Disagree](./phase-2/07-crestline-energy/ch111-multi-region-data-residency-advanced.md) | Staff | 8/10 | 4h | [#data-residency](#tag-data-residency) [#gdpr](#tag-gdpr) [#eu-energy](#tag-eu-energy) [#data-sovereignty](#tag-data-sovereignty) [#cross-border](#tag-cross-border) [#anonymization](#tag-anonymization) [#federated-analytics](#tag-federated-analytics) | Energy data sovereignty for France with federated analytics across GDPR jurisdictions |
| [ ] | 112 | ["What Was the Grid State at 14:32:07 on March 3rd?"](./phase-2/07-crestline-energy/ch112-cqrs-temporal-queries.md) | Staff | 9/10 | 5h | [#cqrs](#tag-cqrs) [#temporal-queries](#tag-temporal-queries) [#bi-temporal](#tag-bi-temporal) [#event-sourcing](#tag-event-sourcing) [#regulatory-audit](#tag-regulatory-audit) [#compliance](#tag-compliance) [#time-travel](#tag-time-travel) | Bi-temporal queries for regulatory grid state reconstruction at Ofgem timestamp precision |

> **Special Files**: [Company Intro](./phase-2/07-crestline-energy/intro.md) · [Transition Brief](./phase-2/07-crestline-energy/transition.md)

</details>

<details>
<summary>
  <strong>Company 08 — VenueFlow</strong> · Events / Live Ticketing & Venue Operations · Chapters 113–118
  <br><em>A live ticketing platform. Taylor Swift just announced a stadium tour and 500K fans are trying to buy simultaneously.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 113 | [Real-Time at Scale — 500k Concurrent Fans](./phase-2/08-venueflow/ch113-real-time-at-scale-pub-sub.md) | Staff | 9/10 | 4h | [#realtime](#tag-realtime) [#websockets](#tag-websockets) [#pubsub](#tag-pubsub) [#redis](#tag-redis) [#backpressure](#tag-backpressure) [#distributed](#tag-distributed) | Real-time seat map for 500K concurrent connections with WebSocket fan-out under 200ms |
| [ ] | 114 | [Concert Cancellation Notification Cascade (4th Appearance)](./phase-2/08-venueflow/ch114-notifications-fourth-appearance.md) | Staff | 9/10 | 4h | [#notifications](#tag-notifications) [#spike-production-down](#tag-spike-production-down) [#messaging](#tag-messaging) [#rate-limiting](#tag-rate-limiting) [#dlq](#tag-dlq) [#multi-channel](#tag-multi-channel) | Concert cancellation notification cascade: 2M messages in 119 minutes during production down |
| [ ] | 115 | [Vector Search — Personalized Event Recommendations](./phase-2/08-venueflow/ch115-search-fourth-appearance.md) | Staff | 8/10 | 4h | [#search](#tag-search) [#vector-search](#tag-vector-search) [#elasticsearch](#tag-elasticsearch) [#ml-systems](#tag-ml-systems) [#recommendations](#tag-recommendations) [#cold-start](#tag-cold-start) | Venue and event search with vector recommendations, geo-proximity, and capacity filtering |
| [ ] | 116 | [Ticket Inventory Consistency — The Taylor Swift Problem](./phase-2/08-venueflow/ch116-ticket-inventory-consistency.md) | Staff | 10/10 | 5h | [#distributed](#tag-distributed) [#inventory](#tag-inventory) [#event-sourcing](#tag-event-sourcing) [#crdt](#tag-crdt) [#consistency](#tag-consistency) [#saga](#tag-saga) [#cap-theorem](#tag-cap-theorem) | Ticket inventory consistency preventing overselling with soft-hold and phantom availability |
| [ ] | 117 | [A/B Testing Infrastructure](./phase-2/08-venueflow/ch117-a-b-testing-infrastructure.md) | Strong Senior | 7/10 | 3h | [#ab-testing](#tag-ab-testing) [#experimentation](#tag-experimentation) [#feature-flags](#tag-feature-flags) [#statistics](#tag-statistics) [#data-infrastructure](#tag-data-infrastructure) | A/B testing infrastructure for pricing and UX experiments in live event ticketing |
| [ ] | 118 | [Adaptive Rate Limiting and Ticket Bot DDoS](./phase-2/08-venueflow/ch118-advanced-rate-limiting-ddos.md) | Staff | 9/10 | 4h | [#rate-limiting](#tag-rate-limiting) [#security](#tag-security) [#ddos](#tag-ddos) [#behavioral-fingerprinting](#tag-behavioral-fingerprinting) [#adaptive](#tag-adaptive) [#spike-scale-10x](#tag-spike-scale-10x) | Adaptive DDoS rate limiting distinguishing legitimate surge from attack traffic patterns |

> **Special Files**: [Company Intro](./phase-2/08-venueflow/intro.md) · [Transition Brief](./phase-2/08-venueflow/transition.md)

</details>

---

**[Pattern Summary 9 — Real-Time Systems, A/B Testing & Search](./phase-2/pattern-summary-09.md)** · Chapters 106–118
> WebSocket vs SSE at scale, ticket inventory soft-hold, A/B statistical significance, faceted search, typeahead debounce, DDoS rate limiting tiers, progressive delivery.

<details>
<summary>
  <strong>Company 09 — TeleNova</strong> · Telecommunications / Mobile Virtual Network Operator · Chapters 120–125
  <br><em>A mobile virtual network operator serving 120M subscribers. Five country launches in one quarter.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 120 | [Service Mesh at Production Scale — 60 Services, 18 Months of Istio](./phase-2/09-telanova/ch120-service-mesh-production.md) | Staff | 8/10 | 4h | [#service-mesh](#tag-service-mesh) [#istio](#tag-istio) [#mtls](#tag-mtls) [#traffic-management](#tag-traffic-management) [#sidecar](#tag-sidecar) [#envoy](#tag-envoy) [#latency](#tag-latency) [#production](#tag-production) | Service mesh in production: mTLS, circuit breakers, and telemetry at 180K RPS |
| [ ] | 121 | [200 Million Spans Per Day — Tail Sampling and Cost Control](./phase-2/09-telanova/ch121-distributed-tracing-opentelemetry.md) | Staff | 8/10 | 4h | [#opentelemetry](#tag-opentelemetry) [#distributed-tracing](#tag-distributed-tracing) [#tempo](#tag-tempo) [#tail-sampling](#tag-tail-sampling) [#cost-engineering](#tag-cost-engineering) [#observability](#tag-observability) [#sampling](#tag-sampling) | 200M spans/day: tail-based sampling and cost control with OpenTelemetry and Tempo |
| [ ] | 122 | [Six Hours of Consumer Lag](./phase-2/09-telanova/ch122-advanced-kafka-consumer-lag.md) | Staff | 9/10 | 5h | [#kafka](#tag-kafka) [#consumer-lag](#tag-consumer-lag) [#rebalancing-storm](#tag-rebalancing-storm) [#partition-reassignment](#tag-partition-reassignment) [#spike-inherited-disaster](#tag-spike-inherited-disaster) [#consumer-groups](#tag-consumer-groups) | Six hours of consumer lag: Kafka rebalancing storm prevention for large telecom fleet |
| [ ] | 123 | [Auth at 120 Million — Device Authentication and B2B2C Identity](./phase-2/09-telanova/ch123-auth-fifth-appearance.md) | Staff | 8/10 | 4h | [#auth](#tag-auth) [#oauth2](#tag-oauth2) [#device-authentication](#tag-device-authentication) [#sim-auth](#tag-sim-auth) [#b2b2c](#tag-b2b2c) [#scim](#tag-scim) [#device-identity](#tag-device-identity) [#fifth-appearance](#tag-fifth-appearance) | Auth for 120M subscribers: B2B2C IoT device provisioning with federated identity |
| [ ] | 124 | [5G QoS — Fair Use Without Unfair Enforcement](./phase-2/09-telanova/ch124-network-rate-limiting-qos.md) | Staff | 8/10 | 4h | [#rate-limiting](#tag-rate-limiting) [#5g](#tag-5g) [#qos](#tag-qos) [#fair-use](#tag-fair-use) [#network-policy](#tag-network-policy) [#per-subscriber](#tag-per-subscriber) [#audit-trail](#tag-audit-trail) [#sixth-appearance](#tag-sixth-appearance) | 5G QoS rate limiting for fair-use policies under ARCEP regulatory audit |
| [ ] | 125 | [Five Country Launches in One Quarter](./phase-2/09-telanova/ch125-multi-region-active-active-advanced.md) | Staff | 9/10 | 5h | [#multi-region](#tag-multi-region) [#active-active](#tag-active-active) [#crdt](#tag-crdt) [#anycast](#tag-anycast) [#traffic-steering](#tag-traffic-steering) [#spike-scale-10x](#tag-spike-scale-10x) [#global-expansion](#tag-global-expansion) | Five country launches in one quarter: CRDT-based active-active and anycast routing |

> **Special Files**: [Company Intro](./phase-2/09-telanova/intro.md) · [Transition Brief](./phase-2/09-telanova/transition.md)

</details>

<details>
<summary>
  <strong>Company 10 — BuildRight</strong> · Construction Tech / Project Management SaaS · Chapters 126–131
  <br><em>A construction project management platform. GDPR erasure requests just hit an event-sourced system with no deletion mechanism.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 126 | [CQRS and Advanced Event Sourcing for Project State](./phase-2/10-buildright/ch126-cqrs-event-sourcing-advanced.md) | Staff | 9/10 | 5h | [#event-sourcing](#tag-event-sourcing) [#cqrs](#tag-cqrs) [#bi-temporal](#tag-bi-temporal) [#distributed](#tag-distributed) [#compliance](#tag-compliance) [#legal](#tag-legal) | CQRS advanced event sourcing: replaying events for temporal consistency and legal audit |
| [ ] | 127 | [Advanced Search — Material Catalog and Vendor Discovery](./phase-2/10-buildright/ch127-advanced-search-typeahead-facets.md) | Strong Senior | 7/10 | 4h | [#search](#tag-search) [#elasticsearch](#tag-elasticsearch) [#faceted-search](#tag-faceted-search) [#geospatial](#tag-geospatial) [#typeahead](#tag-typeahead) [#multilingual](#tag-multilingual) | Advanced search with typeahead, facets, and structured credential matching for construction |
| [ ] | 128 | [Experimentation Platform — Contradictory A/B Results](./phase-2/10-buildright/ch128-a-b-testing-and-experimentation.md) | Staff | 8/10 | 5h | [#experimentation](#tag-experimentation) [#ab-testing](#tag-ab-testing) [#statistics](#tag-statistics) [#platform-design](#tag-platform-design) [#spike-design-review](#tag-spike-design-review) | Experimentation platform: resolving contradictory A/B results under design-review ambush |
| [ ] | 129 | [Progressive Delivery with Argo Rollouts](./phase-2/10-buildright/ch129-canary-deployments-advanced.md) | Staff | 8/10 | 5h | [#progressive-delivery](#tag-progressive-delivery) [#canary](#tag-canary) [#gitops](#tag-gitops) [#argocd](#tag-argocd) [#argo-rollouts](#tag-argo-rollouts) [#spike-production-down](#tag-spike-production-down) | Progressive delivery with Argo Rollouts: production-down spike during canary promotion |
| [ ] | 130 | [Data Pipeline Schema Evolution](./phase-2/10-buildright/ch130-data-pipeline-schema-evolution.md) | Staff | 8/10 | 4h | [#schema-evolution](#tag-schema-evolution) [#avro](#tag-avro) [#schema-registry](#tag-schema-registry) [#kafka](#tag-kafka) [#contract-testing](#tag-contract-testing) [#data-pipeline](#tag-data-pipeline) | Data pipeline schema evolution with expand/contract pattern across Kafka and Avro |
| [ ] | 131 | [GDPR Erasure in Event-Sourced Systems](./phase-2/10-buildright/ch131-compliance-gdpr-erasure-and-event-logs.md) | Staff | 9/10 | 6h | [#gdpr](#tag-gdpr) [#event-sourcing](#tag-event-sourcing) [#compliance](#tag-compliance) [#cryptographic-erasure](#tag-cryptographic-erasure) [#data-retention](#tag-data-retention) [#legal](#tag-legal) | GDPR erasure in event-sourced systems: cryptographic erasure from append-only logs |

> **Special Files**: [Company Intro](./phase-2/10-buildright/intro.md) · [Transition Brief](./phase-2/10-buildright/transition.md)

</details>

<details>
<summary>
  <strong>Company 11 — Axiom Labs</strong> · Biotech / Genomic Research Data Platform · Chapters 132–137
  <br><em>A genomic research data platform with 12 petabytes. HIPAA found genomic PII in your application logs.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 132 | [Finding Genetic Needles in Petabyte Haystacks](./phase-2/11-axiom-labs/ch132-vector-search-deep-dive.md) | Staff | 9/10 | 5h | [#vector-search](#tag-vector-search) [#hnsw](#tag-hnsw) [#pgvector](#tag-pgvector) [#pinecone](#tag-pinecone) [#genomics](#tag-genomics) [#similarity-search](#tag-similarity-search) [#embeddings](#tag-embeddings) | Vector search (HNSW) for genomic sequencing similarity at petabyte scale |
| [ ] | 133 | [Genomic Data Is Personal — Differential Privacy and Federated Computation](./phase-2/11-axiom-labs/ch133-privacy-preserving-systems.md) | Staff | 9/10 | 5h | [#differential-privacy](#tag-differential-privacy) [#privacy](#tag-privacy) [#genomics](#tag-genomics) [#federated-computation](#tag-federated-computation) [#data-minimization](#tag-data-minimization) [#gdpr](#tag-gdpr) [#hipaa](#tag-hipaa) | Differential privacy and federated computation for cross-institution genomic research |
| [ ] | 134 | [Drug Target Discovery — ML Pipelines at Research Scale](./phase-2/11-axiom-labs/ch134-ml-recommendation-pipeline.md) | Staff | 8/10 | 4h | [#ml-pipeline](#tag-ml-pipeline) [#model-versioning](#tag-model-versioning) [#shadow-mode](#tag-shadow-mode) [#feature-store](#tag-feature-store) [#drug-discovery](#tag-drug-discovery) [#mlops](#tag-mlops) [#model-monitoring](#tag-model-monitoring) | Drug target discovery: ML pipeline with shadow-mode deployment and feature versioning |
| [ ] | 135 | [Twelve Petabytes and Counting](./phase-2/11-axiom-labs/ch135-columnar-storage-petabyte-scale.md) | Staff | 8/10 | 4h | [#columnar-storage](#tag-columnar-storage) [#apache-parquet](#tag-apache-parquet) [#apache-iceberg](#tag-apache-iceberg) [#petabyte-scale](#tag-petabyte-scale) [#query-optimization](#tag-query-optimization) [#z-ordering](#tag-z-ordering) [#cost-engineering](#tag-cost-engineering) | 12 petabytes and counting: Parquet, Apache Iceberg, and time-travel queries at scale |
| [ ] | 136 | [200 Universities, 200 Data Silos](./phase-2/11-axiom-labs/ch136-data-mesh-intro.md) | Staff | 8/10 | 4h | [#data-mesh](#tag-data-mesh) [#data-products](#tag-data-products) [#federated-governance](#tag-federated-governance) [#data-catalog](#tag-data-catalog) [#domain-ownership](#tag-domain-ownership) [#discoverability](#tag-discoverability) | Data mesh: domain-oriented ownership and federated governance across 200 universities |
| [ ] | 137 | [Genomic PII in the Application Logs](./phase-2/11-axiom-labs/ch137-hipaa-compliance-remediation.md) | Staff | 9/10 | 5h | [#hipaa](#tag-hipaa) [#compliance](#tag-compliance) [#pii](#tag-pii) [#log-remediation](#tag-log-remediation) [#breach-notification](#tag-breach-notification) [#spike-inherited-disaster](#tag-spike-inherited-disaster) [#genomics](#tag-genomics) | HIPAA inherited disaster: genomic PII discovered in application logs and audit trail gaps |

> **Special Files**: [Company Intro](./phase-2/11-axiom-labs/intro.md) · [Transition Brief](./phase-2/11-axiom-labs/transition.md)

</details>

<details>
<summary>
  <strong>Company 12 — PrismHealth</strong> · Healthtech / Remote Patient Monitoring · Chapters 138–143
  <br><em>A remote patient monitoring platform with 10M IoT devices. A German patient's heartbeat cannot leave Germany.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 138 | [14 Million Device Readings Per Minute](./phase-2/12-prismhealth/ch138-iot-at-scale-advanced.md) | Staff | 8/10 | 4h | [#iot](#tag-iot) [#mqtt](#tag-mqtt) [#device-registry](#tag-device-registry) [#fan-in-ingestion](#tag-fan-in-ingestion) [#deduplication](#tag-deduplication) [#message-ordering](#tag-message-ordering) [#healthcare](#tag-healthcare) | 14M device readings/minute: MQTT ingestion, deduplication, and fan-in at IoT scale |
| [ ] | 139 | [mTLS for HIPAA — Every Service Call Is a PHI Flow](./phase-2/12-prismhealth/ch139-service-mesh-healthcare-compliance.md) | Staff | 8/10 | 4h | [#service-mesh](#tag-service-mesh) [#mtls](#tag-mtls) [#hipaa](#tag-hipaa) [#phi](#tag-phi) [#audit-trail](#tag-audit-trail) [#compliance](#tag-compliance) [#healthcare](#tag-healthcare) [#istio](#tag-istio) | mTLS for HIPAA: treating every service call as a PHI flow with istio audit logging |
| [ ] | 140 | [Life-Safety Alerting — When False Positives Kill Trust](./phase-2/12-prismhealth/ch140-advanced-observability-alerting.md) | Staff | 9/10 | 4h | [#observability](#tag-observability) [#alerting](#tag-alerting) [#slo](#tag-slo) [#false-positive](#tag-false-positive) [#life-safety](#tag-life-safety) [#escalation](#tag-escalation) [#healthcare](#tag-healthcare) | Life-safety alerting: false positives kill trust; designing symptom-based SLOs for healthcare |
| [ ] | 141 | [The Vacuum That Wasn't Running](./phase-2/12-prismhealth/ch141-database-internals-postgres-tuning.md) | Staff | 7/10 | 4h | [#postgres](#tag-postgres) [#vacuum](#tag-vacuum) [#autovacuum](#tag-autovacuum) [#connection-pooling](#tag-connection-pooling) [#bloat](#tag-bloat) [#database-internals](#tag-database-internals) [#healthcare](#tag-healthcare) | The vacuum that wasn't running: Postgres autovacuum, table bloat, and connection pool exhaustion |
| [ ] | 142 | [A Patient's Complete Vital History](./phase-2/12-prismhealth/ch142-cqrs-for-patient-timelines.md) | Staff | 8/10 | 4h | [#cqrs](#tag-cqrs) [#event-sourcing](#tag-event-sourcing) [#patient-timelines](#tag-patient-timelines) [#temporal-queries](#tag-temporal-queries) [#clinical-review](#tag-clinical-review) [#hipaa](#tag-hipaa) [#retention](#tag-retention) | Patient vital history: CQRS for time-series observations with append-only immutable timelines |
| [ ] | 143 | [A German Patient's Heartbeat Cannot Leave Germany](./phase-2/12-prismhealth/ch143-multi-region-data-sovereignty-healthcare.md) | Staff | 8/10 | 4h | [#data-sovereignty](#tag-data-sovereignty) [#gdpr](#tag-gdpr) [#hipaa](#tag-hipaa) [#multi-region](#tag-multi-region) [#healthcare](#tag-healthcare) [#cross-border](#tag-cross-border) [#federated-ml](#tag-federated-ml) | A German patient's heartbeat cannot leave Germany: HIPAA multi-region data sovereignty |

> **Special Files**: [Company Intro](./phase-2/12-prismhealth/intro.md) · [Transition Brief](./phase-2/12-prismhealth/transition.md)

</details>

---

**[Pattern Summary 10 — Privacy, Compliance & Platform Engineering](./phase-2/pattern-summary-10.md)** · Chapters 120–143
> Differential privacy tradeoffs, federated computation for cross-institution research, GDPR/HIPAA erasure from event logs, compliance architecture checklist, IDP golden path adoption, chaos engineering blast radius.

<details>
<summary>
  <strong>Company 13 — VertexCloud</strong> · Platform Engineering / Internal Developer Platform · Chapters 145–149
  <br><em>A developer platform serving 2,000 engineers. Nobody is using the paved road, and the $4.2M infrastructure bill belongs to no one.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 145 | [The Paved Road Nobody Walked On](./phase-2/13-vertexcloud/ch145-internal-developer-platform-design.md) | Staff | 8/10 | 4h | [#idp](#tag-idp) [#golden-paths](#tag-golden-paths) [#backstage](#tag-backstage) [#platform-engineering](#tag-platform-engineering) [#self-service-infra](#tag-self-service-infra) [#developer-experience](#tag-developer-experience) | The paved road nobody walked on: IDP golden paths, escape hatches, and compliance-by-design |
| [ ] | 146 | [The $4.2M Infrastructure Bill Nobody Owned](./phase-2/13-vertexcloud/ch146-platform-observability-and-cost.md) | Staff | 8/10 | 4h | [#finops](#tag-finops) [#showback](#tag-showback) [#chargeback](#tag-chargeback) [#cost-attribution](#tag-cost-attribution) [#rightsizing](#tag-rightsizing) [#platform-economics](#tag-platform-economics) [#tagging](#tag-tagging) | The $4.2M infrastructure bill nobody owned: FinOps, showback, chargeback, and tag enforcement |
| [ ] | 147 | [GitOps at 2,000 Engineers](./phase-2/13-vertexcloud/ch147-advanced-deployment-pipeline.md) | Staff | 8/10 | 4h | [#gitops](#tag-gitops) [#argocd](#tag-argocd) [#progressive-delivery](#tag-progressive-delivery) [#multi-env](#tag-multi-env) [#automated-rollback](#tag-automated-rollback) [#slo-breach](#tag-slo-breach) [#deployment](#tag-deployment) | GitOps at 2,000 engineers: ArgoCD, progressive delivery, and automated promotion gates |
| [ ] | 148 | [300 Internal APIs, Zero Deprecation Policy](./phase-2/13-vertexcloud/ch148-api-governance-at-org-scale.md) | Staff | 7/10 | 4h | [#api-governance](#tag-api-governance) [#versioning](#tag-versioning) [#deprecation](#tag-deprecation) [#breaking-changes](#tag-breaking-changes) [#api-gateway](#tag-api-gateway) [#contract-testing](#tag-contract-testing) | 300 internal APIs, zero deprecation policy: governance, breaking change detection, Pact testing |
| [ ] | 149 | [The Game Day That Became a Real Incident](./phase-2/13-vertexcloud/ch149-chaos-engineering-intro.md) | Staff | 9/10 | 5h | [#chaos-engineering](#tag-chaos-engineering) [#game-day](#tag-game-day) [#blast-radius](#tag-blast-radius) [#resilience](#tag-resilience) [#spike-design-review](#tag-spike-design-review) [#incident-response](#tag-incident-response) [#gremlin](#tag-gremlin) | The game day that became a real incident: chaos engineering, blast radius, and abort criteria |

> **Special Files**: [Company Intro](./phase-2/13-vertexcloud/intro.md) · [Transition Brief](./phase-2/13-vertexcloud/transition.md)

</details>

---

> ⚠️ **[CHECKPOINT 4 — Strong Senior Gate @ Meridian Exchange](./phase-2/checkpoint-04.md)** — *Promotion Gate — End of Phase 2*
> Market data distribution: 50M events/sec → 150M, 10K subscribers, sub-millisecond Tier 1 SLA, MiFID II/FINRA audit trail.

---

## Phase 3 — Strong Senior to Staff (Chapters 150–271)
*17 companies · 115 chapters · Staff-level work in frontier industries*

<details>
<summary>
  <strong>Company 01 — OrbitCore</strong> · Space Tech / Satellite Operations Platform · Chapters 150–156
  <br><em>A satellite data platform managing 47 active satellites. You are the third Staff Engineer, hired because the system cannot handle 200.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 150 | [Signal and Noise](./phase-3/01-orbitcore/ch150-signal-and-noise.md) | Staff | 7/10 | 4h | [#telemetry](#tag-telemetry) [#ingestion](#tag-ingestion) [#data-pipeline](#tag-data-pipeline) [#distributed](#tag-distributed) [#messaging](#tag-messaging) [#data-sovereignty](#tag-data-sovereignty) [#capacity-planning](#tag-capacity-planning) | Telemetry ingestion for 47 satellites at 2M events/sec with GDPR data sovereignty |
| [ ] | 151 | [The Cell Problem](./phase-3/01-orbitcore/ch151-the-cell-problem.md) | Staff | 8/10 | 4h | [#cell-based-architecture](#tag-cell-based-architecture) [#blast-radius](#tag-blast-radius) [#failover](#tag-failover) [#distributed](#tag-distributed) [#resilience](#tag-resilience) [#ground-station](#tag-ground-station) | Cell-based architecture for ground station network with explicit blast radius semantics |
| [ ] | 152 | [Sovereignty in Orbit](./phase-3/01-orbitcore/ch152-sovereignty-in-orbit.md) | Staff | 9/10 | 5h | [#data-sovereignty](#tag-data-sovereignty) [#compliance](#tag-compliance) [#gdpr](#tag-gdpr) [#itar](#tag-itar) [#data-residency](#tag-data-residency) [#security](#tag-security) [#distributed](#tag-distributed) | Multi-jurisdiction sovereignty enforcement: GDPR, ITAR, and Cloud Act for satellite data |
| [ ] | 153 | [The Latency Ladder](./phase-3/01-orbitcore/ch153-the-latency-ladder.md) | Staff | 8/10 | 4h | [#latency](#tag-latency) [#routing](#tag-routing) [#real-time](#tag-real-time) [#distributed](#tag-distributed) [#orbital-mechanics](#tag-orbital-mechanics) [#sla](#tag-sla) [#performance](#tag-performance) | Latency-aware routing under SLA constraints from orbital mechanics and ground station geometry |
| [ ] | 154 | [Ground Truth](./phase-3/01-orbitcore/ch154-ground-truth.md) | Staff | 7/10 | 5h | [#rfc](#tag-rfc) [#protocol-migration](#tag-protocol-migration) [#api-design](#tag-api-design) [#distributed](#tag-distributed) [#staff-work](#tag-staff-work) [#typescript](#tag-typescript) [#strangler-fig](#tag-strangler-fig) | CCSDS protocol migration via strangler-fig pattern with compression codec preservation |
| [ ] | 155 | [Capacity at Constellation Scale](./phase-3/01-orbitcore/ch155-capacity-at-constellation-scale.md) | Staff | 8/10 | 5h | [#capacity-planning](#tag-capacity-planning) [#scaling](#tag-scaling) [#cost-engineering](#tag-cost-engineering) [#finops](#tag-finops) [#distributed](#tag-distributed) [#infrastructure](#tag-infrastructure) | Capacity planning for 4x constellation expansion with cost-optimized scaling model |
| [ ] | 156 | [SPIKE — Solar Flare](./phase-3/01-orbitcore/ch156-solar-flare-spike.md) | Staff | 10/10 | 5h | [#incident-response](#tag-incident-response) [#production-down](#tag-production-down) [#failover](#tag-failover) [#resilience](#tag-resilience) [#itar](#tag-itar) [#data-sovereignty](#tag-data-sovereignty) [#spike](#tag-spike) | SPIKE — Solar flare: production down across ground station receivers during RF interference |

> **Special Files**: [Company Intro](./phase-3/01-orbitcore/intro.md) · [Transition Brief](./phase-3/01-orbitcore/transition.md)

</details>

<details>
<summary>
  <strong>Company 02 — IronWatch</strong> · Defense-Adjacent / Cybersecurity & Threat Intelligence · Chapters 157–163
  <br><em>A classified threat intelligence platform. A data breach just crossed a classification boundary.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 157 | [The Briefing Room](./phase-3/02-ironwatch/ch157-the-briefing-room.md) | Staff | 8/10 | 3h | [#security](#tag-security) [#clearance-tiers](#tag-clearance-tiers) [#architecture](#tag-architecture) [#compliance](#tag-compliance) [#air-gapped](#tag-air-gapped) [#data-flow](#tag-data-flow) | Multi-classification network enforcing UNCLAS/SECRET/TS/SCI tier isolation with data diodes |
| [ ] | 158 | [Zero Trust, Zero Margin](./phase-3/02-ironwatch/ch158-zero-trust-zero-margin.md) | Staff | 9/10 | 3h | [#zero-trust](#tag-zero-trust) [#auth](#tag-auth) [#FIDO2](#tag-fido2) [#access-control](#tag-access-control) [#audit](#tag-audit) [#air-gapped](#tag-air-gapped) [#compliance](#tag-compliance) | Zero-trust with FIDO2 hardware keys for classified intelligence platform with air gap |
| [ ] | 159 | [The Compartment](./phase-3/02-ironwatch/ch159-the-compartment.md) | Staff | 9/10 | 3h | [#ABAC](#tag-abac) [#access-control](#tag-access-control) [#compartmentalization](#tag-compartmentalization) [#policy-engine](#tag-policy-engine) [#compliance](#tag-compliance) [#audit](#tag-audit) | ABAC policy engine for TS/SCI compartment enforcement with bi-temporal authorization audit |
| [ ] | 160 | [Inherited Disaster — The Data That Crossed the Line](./phase-3/02-ironwatch/ch160-inherited-disaster-data-breach.md) | Staff | 10/10 | 4h | [#incident-response](#tag-incident-response) [#data-breach](#tag-data-breach) [#audit](#tag-audit) [#forensics](#tag-forensics) [#compliance](#tag-compliance) [#inherited-disaster](#tag-inherited-disaster) | Breach forensics: classified data crossed a tier boundary — reconstruct, notify, remediate |
| [ ] | 161 | [The Air Gap](./phase-3/02-ironwatch/ch161-the-air-gap.md) | Staff | 9/10 | 3h | [#air-gapped](#tag-air-gapped) [#cross-domain-solution](#tag-cross-domain-solution) [#data-diode](#tag-data-diode) [#network-security](#tag-network-security) [#threat-intel](#tag-threat-intel) [#hardware](#tag-hardware) | Hardware data diode architecture replacing USB courier for threat intel feed ingestion |
| [ ] | 162 | [Audit Everything](./phase-3/02-ironwatch/ch162-audit-everything.md) | Staff | 9/10 | 3h | [#audit](#tag-audit) [#compliance](#tag-compliance) [#cryptographic-chaining](#tag-cryptographic-chaining) [#WORM](#tag-worm) [#immutability](#tag-immutability) [#forensics](#tag-forensics) | Tamper-evident audit log with cryptographic chaining and WORM storage for federal compliance |
| [ ] | 163 | [The Exit Brief](./phase-3/02-ironwatch/ch163-the-exit-brief.md) | Staff | 10/10 | 4h | [#RFC](#tag-rfc) [#design-review](#tag-design-review) [#compliance](#tag-compliance) [#audit-escrow](#tag-audit-escrow) [#key-management](#tag-key-management) [#legal](#tag-legal) | Audit log escrow for contractor data custody under DFARS with key management regulation |

> **Special Files**: [Company Intro](./phase-3/02-ironwatch/intro.md) · [Transition Brief](./phase-3/02-ironwatch/transition.md)

</details>

---

**[Pattern Summary 11 — Staff-Level Architecture Principles](./phase-3/pattern-summary-11.md)** · Chapters 150–163
> Cell-based architecture, blast radius design, data sovereignty at org level, RFC writing and rejection, air-gapped systems, tamper-evident compliance chains.

<details>
<summary>
  <strong>Company 03 — CarbonLedger</strong> · Climate Tech / Carbon Credit Registry · Chapters 165–170
  <br><em>A carbon credit registry where every number must be immutable and auditable for 100 years.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 165 | [The Ledger That Cannot Lie](./phase-3/03-carbonledger/ch165-the-ledger-that-cannot-lie.md) | Strong Senior | 7/10 | 4h | [#database](#tag-database) [#compliance](#tag-compliance) [#distributed](#tag-distributed) [#immutability](#tag-immutability) [#event-sourcing](#tag-event-sourcing) | Immutable carbon credit registry preventing double-counting via event sourcing and Merkle proof |
| [ ] | 166 | [Article 6 Compliance](./phase-3/03-carbonledger/ch166-article-6-compliance.md) | Strong Senior | 8/10 | 4h | [#compliance](#tag-compliance) [#distributed](#tag-distributed) [#api-design](#tag-api-design) [#data-residency](#tag-data-residency) [#eventual-consistency](#tag-eventual-consistency) | Article 6 compliance architecture for multi-jurisdiction carbon credit transfers |
| [ ] | 167 | [SPIKE — Scale 10x by Friday](./phase-3/03-carbonledger/ch167-scale-10x-spike.md) | Strong Senior | 9/10 | 5h | [#distributed](#tag-distributed) [#database](#tag-database) [#caching](#tag-caching) [#messaging](#tag-messaging) [#spike-scale-10x](#tag-spike-scale-10x) [#cost-engineering](#tag-cost-engineering) | SPIKE — Scale 10x by Friday: UN announcement triggers 10x capacity surge with cost pressure |
| [ ] | 168 | [Event Sourcing for Regulators](./phase-3/03-carbonledger/ch168-event-sourcing-for-regulators.md) | Staff | 8/10 | 4h | [#event-sourcing](#tag-event-sourcing) [#database](#tag-database) [#compliance](#tag-compliance) [#distributed](#tag-distributed) [#audit](#tag-audit) | Event sourcing enabling UN audit trail reconstruction with Merkle data integrity verification |
| [ ] | 169 | [The Cost of Green](./phase-3/03-carbonledger/ch169-the-cost-of-green.md) | Staff | 7/10 | 3h | [#cost-engineering](#tag-cost-engineering) [#finops](#tag-finops) [#database](#tag-database) [#storage](#tag-storage) [#distributed](#tag-distributed) | Cost engineering: per-transaction break-even modeling for UN carbon registry at 10M/day |
| [ ] | 170 | [The Permanence Problem](./phase-3/03-carbonledger/ch170-the-permanence-problem.md) | Staff | 9/10 | 5h | [#storage](#tag-storage) [#compliance](#tag-compliance) [#database](#tag-database) [#distributed](#tag-distributed) [#data-pipeline](#tag-data-pipeline) | Permanence architecture: 100-year carbon credit durability with format migration and reversal events |

> **Special Files**: [Company Intro](./phase-3/03-carbonledger/intro.md) · [Transition Brief](./phase-3/03-carbonledger/transition.md)

</details>

<details>
<summary>
  <strong>Company 04 — LexCore</strong> · Legal AI / Privileged Information Management · Chapters 171–177
  <br><em>A legal AI platform managing attorney-client privileged documents. Your RFC just got rejected company-wide.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 171 | [Privileged Information](./phase-3/04-lexcore/ch171-privileged-information.md) | Staff | 8/10 | 3h | [#access-control](#tag-access-control) [#multi-tenancy](#tag-multi-tenancy) [#compliance](#tag-compliance) [#legal](#tag-legal) [#conflict-of-interest](#tag-conflict-of-interest) [#data-isolation](#tag-data-isolation) | Multi-tenant access control for legal documents enforcing matter isolation and conflict-of-interest |
| [ ] | 172 | [The Chunking Pipeline](./phase-3/04-lexcore/ch172-the-chunking-pipeline.md) | Staff | 7/10 | 3h | [#RAG](#tag-rag) [#document-processing](#tag-document-processing) [#chunking](#tag-chunking) [#legal-AI](#tag-legal-ai) [#pipeline](#tag-pipeline) [#embeddings](#tag-embeddings) | Legal-aware document chunking pipeline preserving clause integrity for RAG systems |
| [ ] | 173 | [Where Are the Embeddings?](./phase-3/04-lexcore/ch173-design-review-ambush-spike.md) | Staff | 9/10 | 3h | [#data-residency](#tag-data-residency) [#GDPR](#tag-gdpr) [#compliance](#tag-compliance) [#embeddings](#tag-embeddings) [#UK-GDPR](#tag-uk-gdpr) [#design-review](#tag-design-review) [#spike](#tag-spike) | SPIKE — Design review ambush: UK GDPR data residency requirements for embedding storage |
| [ ] | 174 | [The RFC That Failed](./phase-3/04-lexcore/ch174-the-rfc-that-failed.md) | Staff | 9/10 | 3h | [#RFC](#tag-rfc) [#multi-party](#tag-multi-party) [#data-rooms](#tag-data-rooms) [#encryption](#tag-encryption) [#legal](#tag-legal) [#stakeholder-management](#tag-stakeholder-management) | The RFC that failed: platform encryption in deal rooms violated attorney-client privilege |
| [ ] | 175 | [The RFC That Succeeded](./phase-3/04-lexcore/ch175-the-rfc-that-succeeded.md) | Staff | 10/10 | 4h | [#RFC](#tag-rfc) [#client-side-encryption](#tag-client-side-encryption) [#zero-knowledge](#tag-zero-knowledge) [#E2E](#tag-e2e) [#legal-hold](#tag-legal-hold) [#key-escrow](#tag-key-escrow) [#compliance](#tag-compliance) | The RFC that succeeded: client-side encryption with key escrow for legal holds |
| [ ] | 176 | [Find the Precedent](./phase-3/04-lexcore/ch176-search-at-legal-scale.md) | Staff | 8/10 | 3h | [#search](#tag-search) [#legal](#tag-legal) [#Elasticsearch](#tag-elasticsearch) [#RAG](#tag-rag) [#multi-tenant](#tag-multi-tenant) [#citation-graph](#tag-citation-graph) [#compliance](#tag-compliance) | Hybrid legal search: dense retrieval, citation graph, and jurisdiction-aware re-ranking |
| [ ] | 177 | [The Privilege Log](./phase-3/04-lexcore/ch177-the-privilege-log.md) | Staff | 8/10 | 2.5h | [#legal-ai](#tag-legal-ai) [#ml](#tag-ml) [#compliance](#tag-compliance) [#search](#tag-search) | Privilege log generation for litigation with ML confidence thresholds under time pressure |

> **Special Files**: [Company Intro](./phase-3/04-lexcore/intro.md) · [Transition Brief](./phase-3/04-lexcore/transition.md)

</details>

<details>
<summary>
  <strong>Company 05 — NeuralBridge</strong> · Neuroscience Tech / Brain-Computer Interface Data · Chapters 178–183
  <br><em>A BCI data platform with FDA 510(k) classification. Pipeline isolation failures put patients at risk.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 178 | [The Signal](./phase-3/05-neuralbridge/ch178-the-signal.md) | Strong Senior | 7/10 | 4h | [#real-time](#tag-real-time) [#streaming](#tag-streaming) [#edge-processing](#tag-edge-processing) [#hipaa](#tag-hipaa) [#distributed](#tag-distributed) [#kafka](#tag-kafka) | Real-time signal processing for 200 BCI devices with <10ms FDA-mandated latency under isolation |
| [ ] | 179 | [FDA 510(k) Architecture](./phase-3/05-neuralbridge/ch179-fda-510k-architecture.md) | Strong Senior | 8/10 | 5h | [#compliance](#tag-compliance) [#fda](#tag-fda) [#medical-device](#tag-medical-device) [#architecture](#tag-architecture) [#documentation](#tag-documentation) [#security](#tag-security) | FDA 510(k) SaMD classification requiring IEC 62304 Class C documentation |
| [ ] | 180 | [Brain Data Privacy](./phase-3/05-neuralbridge/ch180-brain-data-privacy.md) | Strong Senior | 8/10 | 4h | [#privacy](#tag-privacy) [#hipaa](#tag-hipaa) [#gdpr](#tag-gdpr) [#compliance](#tag-compliance) [#data-governance](#tag-data-governance) [#consent](#tag-consent) | Privacy architecture distinguishing employer wellness, research, and actuarial brain data uses |
| [ ] | 181 | [The Systemic Failure](./phase-3/05-neuralbridge/ch181-the-systemic-failure.md) | Staff | 9/10 | 5h | [#architecture-review](#tag-architecture-review) [#distributed](#tag-distributed) [#isolation](#tag-isolation) [#kafka](#tag-kafka) [#patient-safety](#tag-patient-safety) [#staff-skills](#tag-staff-skills) | SPIKE — Cross-team review exposes pipeline isolation failures with systemic patient safety risk |
| [ ] | 182 | [Edge Intelligence](./phase-3/05-neuralbridge/ch182-edge-intelligence.md) | Staff | 8/10 | 4h | [#edge-computing](#tag-edge-computing) [#iot](#tag-iot) [#graceful-degradation](#tag-graceful-degradation) [#ota-updates](#tag-ota-updates) [#medical-device](#tag-medical-device) | Edge processor graceful degradation for extended connectivity loss with buffer management |
| [ ] | 183 | [The Clinical Data Platform](./phase-3/05-neuralbridge/ch183-the-clinical-data-platform.md) | Staff | 9/10 | 5h | [#compliance](#tag-compliance) [#21cfr](#tag-21cfr) [#audit-trail](#tag-audit-trail) [#data-integrity](#tag-data-integrity) [#clinical-trials](#tag-clinical-trials) [#immutability](#tag-immutability) | FDA-compliant data lineage audit trail enabling full provenance reconstruction with hash proofs |

> **Special Files**: [Company Intro](./phase-3/05-neuralbridge/intro.md) · [Transition Brief](./phase-3/05-neuralbridge/transition.md)

</details>

---

> ⚠️ **[CHECKPOINT 5 — Strong Senior → Staff Gate @ Apex Engineering](./phase-3/checkpoint-05.md)** — *Promotion Gate*
> Safety-critical system design under ambiguity. Clarifying questions are required. Staff-level ambiguity tolerance test.

**[Pattern Summary 12 — Immutability, Permanence & Long-Horizon Systems](./phase-3/pattern-summary-12.md)** · Chapters 165–183
> Cryptographic immutability, 100-year data permanence, FDA-compliant data lineage, client-side encryption with key escrow, legal privilege boundaries, BCI safety-critical data isolation.

<details>
<summary>
  <strong>Company 06 — ShieldMutual</strong> · Insurance / Risk Engine & Claims · Chapters 186–192
  <br><em>An insurance risk and claims platform. A cascade failure in claims processing triggered regulatory notification.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 186 | [The Risk Engine](./phase-3/06-shieldmutual/ch186-the-risk-engine.md) | Staff | 8/10 | 3h | [#risk-scoring](#tag-risk-scoring) [#real-time](#tag-real-time) [#api-design](#tag-api-design) [#caching](#tag-caching) [#external-data](#tag-external-data) [#compliance](#tag-compliance) [#insurance](#tag-insurance) [#latency](#tag-latency) | Risk scoring pipeline: sub-second insurance underwriting with third-party data federation |
| [ ] | 187 | [The Actuarial Pipeline](./phase-3/06-shieldmutual/ch187-the-actuarial-pipeline.md) | Staff | 8/10 | 3h | [#data-pipeline](#tag-data-pipeline) [#batch-processing](#tag-batch-processing) [#observability](#tag-observability) [#insurance](#tag-insurance) [#compliance](#tag-compliance) [#debugging](#tag-debugging) [#mentorship](#tag-mentorship) | Actuarial data pipeline for insurance risk models with historical data and observability |
| [ ] | 188 | [Claims at Scale](./phase-3/06-shieldmutual/ch188-claims-at-scale.md) | Staff | 8/10 | 5h | [#spike-production-down](#tag-spike-production-down) [#scale-10x](#tag-scale-10x) [#kafka](#tag-kafka) [#ml-serving](#tag-ml-serving) [#distributed](#tag-distributed) [#incident-response](#tag-incident-response) | SPIKE — Claims cascade: catastrophic weather event causes production down across claims processing |
| [ ] | 189 | [SOC2 Under Pressure](./phase-3/06-shieldmutual/ch189-soc2-under-pressure.md) | Staff | 7/10 | 4h | [#compliance](#tag-compliance) [#soc2](#tag-soc2) [#security](#tag-security) [#audit](#tag-audit) [#architecture](#tag-architecture) [#multi-tenant](#tag-multi-tenant) | SOC2 compliance architecture under regulatory audit with comprehensive security controls |
| [ ] | 190 | [The Multi-Line Model](./phase-3/06-shieldmutual/ch190-the-multi-line-model.md) | Staff | 8/10 | 5h | [#database-design](#tag-database-design) [#data-modeling](#tag-data-modeling) [#mergers-acquisitions](#tag-mergers-acquisitions) [#schema-evolution](#tag-schema-evolution) [#multi-tenant](#tag-multi-tenant) | Multi-line insurance product model for post-acquisition system data consolidation |
| [ ] | 191 | [Production DOWN — Claims Cascade Failure](./phase-3/06-shieldmutual/ch191-production-down-claims-cascade.md) | Staff | 10/10 | 3h | [#incident-response](#tag-incident-response) [#cascade-failure](#tag-cascade-failure) [#circuit-breaker](#tag-circuit-breaker) [#ml-deployment](#tag-ml-deployment) | SPIKE — Production down: claims cascade failure with circuit breaker implementation |
| [ ] | 192 | [State Regulation as Architecture](./phase-3/06-shieldmutual/ch192-state-regulation-as-architecture.md) | Staff | 8/10 | 3h | [#compliance](#tag-compliance) [#regulatory](#tag-regulatory) [#data-architecture](#tag-data-architecture) [#multi-jurisdiction](#tag-multi-jurisdiction) | State-by-state regulatory compliance architecture for insurance platform operations |

> **Special Files**: [Company Intro](./phase-3/06-shieldmutual/intro.md) · [Transition Brief](./phase-3/06-shieldmutual/transition.md)

</details>

<details>
<summary>
  <strong>Company 07 — AutoMesh</strong> · Autonomous Vehicles / V2X & Fleet OTA · Chapters 193–200
  <br><em>A V2X coordination platform for autonomous vehicles. Five nines means someone could die.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 193 | [V2X at Scale](./phase-3/07-automesh/ch193-v2x-at-scale.md) | Staff | 8/10 | 3h | [#v2x](#tag-v2x) [#safety-critical](#tag-safety-critical) [#messaging](#tag-messaging) [#real-time](#tag-real-time) [#distributed](#tag-distributed) [#api-design](#tag-api-design) | V2X messaging supporting DSRC and C-V2X at million-messages-per-second ingestion |
| [ ] | 194 | [The Safety-Critical Edge](./phase-3/07-automesh/ch194-the-safety-critical-edge.md) | Staff | 9/10 | 3h | [#edge-computing](#tag-edge-computing) [#safety-critical](#tag-safety-critical) [#fmea](#tag-fmea) [#nhtsa](#tag-nhtsa) [#iso26262](#tag-iso26262) [#distributed](#tag-distributed) | Safety-critical edge design with FMEA-driven failure mode analysis for autonomous vehicles |
| [ ] | 195 | [OTA at Fleet Scale](./phase-3/07-automesh/ch195-ota-at-fleet-scale.md) | Staff | 8/10 | 2.5h | [#ota](#tag-ota) [#security](#tag-security) [#fleet](#tag-fleet) [#safety-critical](#tag-safety-critical) [#firmware](#tag-firmware) [#distributed](#tag-distributed) | OTA update pipeline for vehicle fleet with rollback safety and staged deployment guarantees |
| [ ] | 196 | [Scale 10x — Fleet Doubles](./phase-3/07-automesh/ch196-scale-10x-fleet-doubles-spike.md) | Staff | 9/10 | 3h | [#scaling](#tag-scaling) [#capacity-planning](#tag-capacity-planning) [#cost-engineering](#tag-cost-engineering) [#spike](#tag-spike) [#consumer-vehicles](#tag-consumer-vehicles) | SPIKE — Fleet doubles overnight: scale 10x with cost optimization under safety constraints |
| [ ] | 197 | [The Fractured Map](./phase-3/07-automesh/ch197-hd-map-distribution.md) | Staff | 8/10 | 3h | [#mapping](#tag-mapping) [#distribution](#tag-distribution) [#consistency](#tag-consistency) [#ota](#tag-ota) [#safety-critical](#tag-safety-critical) [#edge](#tag-edge) | HD map distribution with consistency guarantees and differential update optimization |
| [ ] | 198 | [The $2.3M Conversation](./phase-3/07-automesh/ch198-the-cfo-presentation.md) | Staff | 7/10 | 2.5h | [#cost-engineering](#tag-cost-engineering) [#finops](#tag-finops) [#capacity-planning](#tag-capacity-planning) [#infrastructure](#tag-infrastructure) [#staff-patterns](#tag-staff-patterns) | The $2.3M conversation: infrastructure cost analysis presentation to the CFO |
| [ ] | 199 | [Every Way This Can Kill Someone](./phase-3/07-automesh/ch199-fmea-driven-architecture.md) | Staff | 9/10 | 3.5h | [#fmea](#tag-fmea) [#safety-critical](#tag-safety-critical) [#reliability](#tag-reliability) [#iso26262](#tag-iso26262) [#v2x](#tag-v2x) [#risk-analysis](#tag-risk-analysis) | FMEA-driven architecture review identifying all failure modes with safety impact analysis |
| [ ] | 200 | [Five Nines for a Moving Vehicle](./phase-3/07-automesh/ch200-the-partnership-aftermath.md) | Staff | 9/10 | 3h | [#multi-region](#tag-multi-region) [#high-availability](#tag-high-availability) [#v2x](#tag-v2x) [#sla](#tag-sla) [#active-active](#tag-active-active) [#distributed](#tag-distributed) | Five nines for a moving vehicle: high-availability V2X coordination at 99.999% |

> **Special Files**: [Company Intro](./phase-3/07-automesh/intro.md) · [Transition Brief](./phase-3/07-automesh/transition.md)

</details>

---

**[Pattern Summary 13 — Safety-Critical & Industrial Systems](./phase-3/pattern-summary-13.md)** · Chapters 186–200
> FMEA-driven architecture, safety-critical edge design, OTA at fleet scale, V2X protocols, five-nines design for systems where failure means physical harm.

<details>
<summary>
  <strong>Company 08 — ForgeSense</strong> · Manufacturing / Industrial IoT & Digital Twins · Chapters 202–208
  <br><em>An industrial IoT platform bridging OT and IT. The SCADA system has unknown vulnerabilities and serves live production.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 202 | [The Digital Twin](./phase-3/08-forgesense/ch202-the-digital-twin.md) | Strong Senior | 7/10 | 4h | [#iot](#tag-iot) [#digital-twin](#tag-digital-twin) [#real-time](#tag-real-time) [#time-series](#tag-time-series) [#edge-computing](#tag-edge-computing) [#distributed](#tag-distributed) | Digital twin for 50K sensors balancing 10ms Group A latency and 1s Group B across a factory |
| [ ] | 203 | [OPC-UA to Cloud](./phase-3/08-forgesense/ch203-opc-ua-to-cloud.md) | Strong Senior | 7/10 | 3.5h | [#iot](#tag-iot) [#opc-ua](#tag-opc-ua) [#data-pipeline](#tag-data-pipeline) [#brownfield](#tag-brownfield) [#reliability](#tag-reliability) [#edge](#tag-edge) | OPC-UA industrial protocol integration with cloud data pipeline for SCADA modernization |
| [ ] | 204 | [The Webb Keynote](./phase-3/08-forgesense/ch204-the-webb-keynote.md) | Strong Senior | 7/10 | 4h | [#ml-systems](#tag-ml-systems) [#data-quality](#tag-data-quality) [#feature-stores](#tag-feature-stores) [#predictive-maintenance](#tag-predictive-maintenance) [#observability](#tag-observability) | Marcus Webb keynote at ForgeSense Engineering Summit: platform engineering as product |
| [ ] | 205 | [The Ghost in the Machine](./phase-3/08-forgesense/ch205-inherited-disaster-scada.md) | Staff | 9/10 | 4h | [#security](#tag-security) [#ot-it-convergence](#tag-ot-it-convergence) [#scada](#tag-scada) [#incident-response](#tag-incident-response) [#inherited-disaster](#tag-inherited-disaster) [#compliance](#tag-compliance) [#industrial](#tag-industrial) | SPIKE — Inherited disaster: brownfield SCADA integration with unknown security vulnerabilities |
| [ ] | 206 | [What Do You Mean, Read-Only?](./phase-3/08-forgesense/ch206-design-review-ambush-brownfield.md) | Staff | 9/10 | 3.5h | [#design-review](#tag-design-review) [#ot-it-convergence](#tag-ot-it-convergence) [#bidirectional-control](#tag-bidirectional-control) [#opc-ua](#tag-opc-ua) [#safety](#tag-safety) [#command-audit](#tag-command-audit) [#industrial](#tag-industrial) | SPIKE — Design review ambush: defending bidirectional control against manufacturing engineers |
| [ ] | 207 | [The 200ms Window](./phase-3/08-forgesense/ch207-real-time-control-layer.md) | Staff | 9/10 | 4h | [#real-time](#tag-real-time) [#edge-computing](#tag-edge-computing) [#latency](#tag-latency) [#safety-critical](#tag-safety-critical) [#hierarchical-control](#tag-hierarchical-control) [#industrial](#tag-industrial) [#ot-it-convergence](#tag-ot-it-convergence) | Real-time control layer: predictive maintenance and anomaly detection at 200ms window |
| [ ] | 208 | [The Twelve Factories](./phase-3/08-forgesense/ch208-factory-data-mesh.md) | Staff | 8/10 | 3.5h | [#data-mesh](#tag-data-mesh) [#federated-governance](#tag-federated-governance) [#multi-tenancy](#tag-multi-tenancy) [#analytics](#tag-analytics) [#manufacturing](#tag-manufacturing) [#competitive-sensitivity](#tag-competitive-sensitivity) [#data-products](#tag-data-products) | Factory data mesh: domain-oriented product ownership and federated governance across 12 sites |

> **Special Files**: [Company Intro](./phase-3/08-forgesense/intro.md) · [Transition Brief](./phase-3/08-forgesense/transition.md)

</details>

<details>
<summary>
  <strong>Company 09 — DeepOcean</strong> · Oceanography / Maritime Data & Vessel Tracking · Chapters 209–214
  <br><em>A maritime intelligence platform. Ships go offline for 3 weeks at a time and regulatory jurisdiction changes by flag state.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 209 | [The Ocean's Heartbeat](./phase-3/09-deepocean/ch209-ais-the-oceans-heartbeat.md) | Staff | 7/10 | 3h | [#data-ingestion](#tag-data-ingestion) [#kafka](#tag-kafka) [#geospatial](#tag-geospatial) [#ais](#tag-ais) [#streaming](#tag-streaming) [#partitioning](#tag-partitioning) [#backpressure](#tag-backpressure) | AIS maritime pipeline handling 98K messages/sec with geographic hotspot partitioning |
| [ ] | 210 | [Three Weeks at Sea](./phase-3/09-deepocean/ch210-offline-first-for-ships.md) | Staff | 8/10 | 3.5h | [#offline-first](#tag-offline-first) [#sync](#tag-sync) [#conflict-resolution](#tag-conflict-resolution) [#maritime](#tag-maritime) [#edge](#tag-edge) [#connectivity](#tag-connectivity) [#mobile](#tag-mobile) | Offline-first architecture for ships with edge caching and eventual sync for 3-week voyages |
| [ ] | 211 | [The Napkin Diagram](./phase-3/09-deepocean/ch211-the-global-ocean-data-mesh.md) | Staff | 9/10 | 4h | [#data-mesh](#tag-data-mesh) [#conflict-resolution](#tag-conflict-resolution) [#federated-data](#tag-federated-data) [#multi-source](#tag-multi-source) [#oceanography](#tag-oceanography) [#data-quality](#tag-data-quality) [#authority-levels](#tag-authority-levels) | Global ocean data mesh federating AIS, oceanographic, and environmental data sources |
| [ ] | 212 | [2,000 Miles from Anywhere](./phase-3/09-deepocean/ch212-edge-processing-at-sea.md) | Staff | 8/10 | 3.5h | [#edge-ml](#tag-edge-ml) [#model-updates](#tag-model-updates) [#bandwidth-constrained](#tag-bandwidth-constrained) [#differential-updates](#tag-differential-updates) [#maritime](#tag-maritime) [#offline](#tag-offline) [#optimization](#tag-optimization) | Edge processing at sea: local anomaly detection reducing backhaul bandwidth for remote vessels |
| [ ] | 213 | [Designing for the Speed of Light](./phase-3/09-deepocean/ch213-extreme-latency-architecture.md) | Staff | 8/10 | 3h | [#latency](#tag-latency) [#offline-first](#tag-offline-first) [#optimistic-updates](#tag-optimistic-updates) [#ux](#tag-ux) [#connectivity-tiers](#tag-connectivity-tiers) [#sync](#tag-sync) [#edge](#tag-edge) | Extreme latency architecture: designing for 24-hour round-trip communication windows |
| [ ] | 214 | [Flag State to Port State](./phase-3/09-deepocean/ch214-maritime-compliance.md) | Staff | 8/10 | 3h | [#compliance](#tag-compliance) [#maritime](#tag-maritime) [#multi-jurisdiction](#tag-multi-jurisdiction) [#regulatory](#tag-regulatory) | Maritime compliance for SOLAS, IMO, and regional shipping regulations with audit trail |

> **Special Files**: [Company Intro](./phase-3/09-deepocean/intro.md) · [Transition Brief](./phase-3/09-deepocean/transition.md)

</details>

---

**[Pattern Summary 14 — Edge Intelligence & Offline-First Architecture](./phase-3/pattern-summary-14.md)** · Chapters 202–214
> Digital twin design, OT/IT convergence, OPC-UA integration, maritime offline-first, extreme latency architecture, edge ML model management.

<details>
<summary>
  <strong>Company 10 — PharmaSync</strong> · Biotech & Pharma / Clinical Data Compliance · Chapters 216–222
  <br><em>A pharmaceutical data platform under FDA oversight. Two phantom patients appeared in a clinical trial dataset.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 216 | [21 CFR Part 11](./phase-3/10-pharmasync/ch216-21-cfr-part-11.md) | Strong Senior | 8/10 | 4h | [#compliance](#tag-compliance) [#database](#tag-database) [#audit](#tag-audit) [#security](#tag-security) [#distributed](#tag-distributed) | 21 CFR Part 11 audit trail with ALCOA principles and independent modification log storage |
| [ ] | 217 | [Lab Data Integration](./phase-3/10-pharmasync/ch217-lab-data-integration.md) | Strong Senior | 7/10 | 3h | [#data-pipeline](#tag-data-pipeline) [#integration](#tag-integration) [#etl](#tag-etl) [#database](#tag-database) [#api-design](#tag-api-design) | Lab data integration for pharma with LIMS compatibility and analytical instrument federation |
| [ ] | 218 | [Feature Stores for Drug Discovery](./phase-3/10-pharmasync/ch218-feature-stores-drug-discovery.md) | Staff | 8/10 | 4h | [#ml-systems](#tag-ml-systems) [#data-pipeline](#tag-data-pipeline) [#storage](#tag-storage) [#database](#tag-database) [#distributed](#tag-distributed) | Feature stores for drug discovery combining batch training with real-time assay results |
| [ ] | 219 | [The Six-Week Submission](./phase-3/10-pharmasync/ch219-clinical-data-management.md) | Strong Senior | 7/10 | 4h | [#compliance](#tag-compliance) [#data-pipeline](#tag-data-pipeline) [#distributed](#tag-distributed) [#database](#tag-database) [#api-design](#tag-api-design) | Clinical data platform with 21 CFR Part 11 compliance and patient safety event detection |
| [ ] | 220 | [Gerald Must Die](./phase-3/10-pharmasync/ch220-the-lims-deprecation.md) | Staff | 8/10 | 5h | [#system-evolution](#tag-system-evolution) [#compliance](#tag-compliance) [#database](#tag-database) [#distributed](#tag-distributed) [#api-design](#tag-api-design) | LIMS deprecation strategy for legacy laboratory system with zero-downtime migration |
| [ ] | 221 | [The Two Phantom Patients](./phase-3/10-pharmasync/ch221-production-down-clinical-trial.md) | Staff | 9/10 | 4h | [#incident-response](#tag-incident-response) [#data-pipeline](#tag-data-pipeline) [#compliance](#tag-compliance) [#distributed](#tag-distributed) [#database](#tag-database) | SPIKE — Two phantom patients: clinical trial data integrity breach with FDA notification |
| [ ] | 222 | [The Retroactive Proof](./phase-3/10-pharmasync/ch222-the-validation-protocol.md) | Staff | 9/10 | 3h | [#21cfr](#tag-21cfr) [#fda-validation](#tag-fda-validation) [#compliance](#tag-compliance) [#pharma](#tag-pharma) [#data-integrity](#tag-data-integrity) [#audit](#tag-audit) | Validation protocol proving system correctness for regulated pharma software under FDA |

> **Special Files**: [Company Intro](./phase-3/10-pharmasync/intro.md) · [Transition Brief](./phase-3/10-pharmasync/transition.md)

</details>

<details>
<summary>
  <strong>Company 11 — HarvestAI</strong> · Precision Agriculture / Geospatial & Edge ML · Chapters 223–228
  <br><em>A precision agriculture platform. The harvest season spike just exposed architectural bottlenecks nobody planned for.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 223 | [When the Earth Won't Fit in a B-Tree](./phase-3/11-harvestai/ch223-geospatial-at-scale.md) | Strong Senior | 7/10 | 2.5h | [#geospatial](#tag-geospatial) [#postgis](#tag-postgis) [#spatial-indexing](#tag-spatial-indexing) [#caching](#tag-caching) [#performance](#tag-performance) | Geospatial query optimization for agriculture: PostGIS index selectivity under drought loads |
| [ ] | 224 | [Design for 100x, Spend 2x](./phase-3/11-harvestai/ch224-the-seasonal-spike.md) | Staff | 8/10 | 3h | [#seasonal-scaling](#tag-seasonal-scaling) [#auto-scaling](#tag-auto-scaling) [#finops](#tag-finops) [#cost-engineering](#tag-cost-engineering) [#capacity-planning](#tag-capacity-planning) | Design for 100x, spend 2x: seasonal harvest spike handling with predictable auto-scaling |
| [ ] | 225 | [The Model in the Field](./phase-3/11-harvestai/ch225-edge-ml-inference.md) | Staff | 8/10 | 3h | [#edge-ml](#tag-edge-ml) [#iot](#tag-iot) [#model-serving](#tag-model-serving) [#ota](#tag-ota) [#bandwidth-constrained](#tag-bandwidth-constrained) | Edge ML inference on farm devices for soil moisture prediction with OTA model updates |
| [ ] | 226 | [The Three-Meter Problem](./phase-3/11-harvestai/ch226-carbon-credits-integration.md) | Staff | 8/10 | 3h | [#carbon-credits](#tag-carbon-credits) [#geospatial](#tag-geospatial) [#compliance](#tag-compliance) [#integration](#tag-integration) [#article6](#tag-article6) | Carbon credit integration bridging agricultural soil sequestration to voluntary carbon market |
| [ ] | 227 | [One Account, Eight Hundred Farms](./phase-3/11-harvestai/ch227-multi-tenant-farm-management.md) | Staff | 8/10 | 3h | [#multi-tenancy](#tag-multi-tenancy) [#rbac](#tag-rbac) [#hierarchical-permissions](#tag-hierarchical-permissions) [#saas](#tag-saas) [#data-isolation](#tag-data-isolation) | Multi-tenant farm management with tenant isolation and per-farmer data governance model |
| [ ] | 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | Staff | 7/10 | 2.5h | [#postmortem](#tag-postmortem) [#incident-management](#tag-incident-management) [#blameless](#tag-blameless) [#reliability](#tag-reliability) [#observability](#tag-observability) [#alerting](#tag-alerting) [#geospatial](#tag-geospatial) [#edge-computing](#tag-edge-computing) [#job-monitoring](#tag-job-monitoring) | Harvest postmortem: incident analysis revealing architectural bottlenecks during peak season |

> **Special Files**: [Company Intro](./phase-3/11-harvestai/intro.md) · [Transition Brief](./phase-3/11-harvestai/transition.md)

</details>

---

**[Pattern Summary 15 — Regulated Industry Deep Dive](./phase-3/pattern-summary-15.md)** · Chapters 216–228
> 21 CFR Part 11, FDA validation protocols, ALCOA principles, geospatial at scale, seasonal spike design, edge ML for agriculture, blameless postmortem culture.

<details>
<summary>
  <strong>Company 12 — NexusWealth</strong> · Pension & Retirement Systems / Long-Term Wealth Management · Chapters 230–237
  <br><em>A pension fund administration platform managing 50 years of data. ERISA and SEC require 7-year audit trails with millisecond precision.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 230 | [Fifty Years of Data](./phase-3/12-nexuswealth/ch230-fifty-years-of-data.md) | Strong Senior | 8/10 | 4h | [#data-retention](#tag-data-retention) [#compliance](#tag-compliance) [#database](#tag-database) [#storage](#tag-storage) [#erisa](#tag-erisa) [#migration](#tag-migration) | Fifty years of data: COBOL flat file and TIFF scan reconstruction with chain-of-custody proof |
| [ ] | 231 | [ERISA and SEC Compliance](./phase-3/12-nexuswealth/ch231-erisa-and-sec-compliance.md) | Strong Senior | 8/10 | 4h | [#compliance](#tag-compliance) [#erisa](#tag-erisa) [#sec](#tag-sec) [#audit](#tag-audit) [#fiduciary](#tag-fiduciary) [#data-pipeline](#tag-data-pipeline) | ERISA and SEC compliance architecture for pension fund administration with audit trails |
| [ ] | 232 | [The March 31st Problem](./phase-3/12-nexuswealth/ch232-actuarial-calculation-at-scale.md) | Strong Senior | 7/10 | 4h | [#batch-processing](#tag-batch-processing) [#distributed-computing](#tag-distributed-computing) [#compliance](#tag-compliance) [#financial-systems](#tag-financial-systems) [#actuarial](#tag-actuarial) | Actuarial calculation engine for pension benefit computation with 50-year historical validation |
| [ ] | 233 | [Ten Minutes to Exposure](./phase-3/12-nexuswealth/ch233-real-time-portfolio-risk.md) | Strong Senior | 8/10 | 5h | [#real-time](#tag-real-time) [#financial-systems](#tag-financial-systems) [#distributed-computing](#tag-distributed-computing) [#risk](#tag-risk) [#streaming](#tag-streaming) [#compliance](#tag-compliance) | Real-time portfolio risk for pension funds with intra-day market volatility and rebalancing |
| [ ] | 234 | [VIX 80 — The ERISA Clock Is Ticking](./phase-3/12-nexuswealth/ch234-scale-10x-market-volatility-spike.md) | Staff | 9/10 | 5h | [#spike-scale-10x](#tag-spike-scale-10x) [#distributed-systems](#tag-distributed-systems) [#compliance](#tag-compliance) [#financial-systems](#tag-financial-systems) [#incident-response](#tag-incident-response) | SPIKE — VIX 80: scale 10x market volatility spike with ERISA clock ticking |
| [ ] | 235 | [The Trustees Want Answers](./phase-3/12-nexuswealth/ch235-the-board-presentation.md) | Staff | 7/10 | 4h | [#cost-engineering](#tag-cost-engineering) [#finops](#tag-finops) [#capacity-planning](#tag-capacity-planning) [#compliance](#tag-compliance) [#staff-engineering](#tag-staff-engineering) | Board presentation: justifying infrastructure spend and architectural choices to pension trustees |
| [ ] | 236 | [One Platform, Three Regulators](./phase-3/12-nexuswealth/ch236-data-sovereignty-pension-funds.md) | Staff | 9/10 | 5h | [#data-sovereignty](#tag-data-sovereignty) [#compliance](#tag-compliance) [#multi-region](#tag-multi-region) [#distributed-systems](#tag-distributed-systems) [#financial-systems](#tag-financial-systems) | One platform, three regulators: data sovereignty for international pension fund operations |
| [ ] | 237 | [A Letter to 2074](./phase-3/12-nexuswealth/ch237-the-50-year-architecture.md) | Staff | 9/10 | 5h | [#architecture](#tag-architecture) [#long-term-thinking](#tag-long-term-thinking) [#compliance](#tag-compliance) [#data-retention](#tag-data-retention) [#staff-engineering](#tag-staff-engineering) | A letter to 2074: 50-year architecture ensuring pension system longevity through format migrations |

> **Special Files**: [Company Intro](./phase-3/12-nexuswealth/intro.md) · [Transition Brief](./phase-3/12-nexuswealth/transition.md)

</details>

<details>
<summary>
  <strong>Company 13 — PropIQ</strong> · Real Estate Tech / Property Intelligence & Market Data · Chapters 238–243
  <br><em>A property intelligence platform federating 12 conflicting data sources. The data quality crisis is live in production.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 238 | [Twelve Sources, One Truth](./phase-3/13-propiq/ch238-the-property-data-mesh.md) | Staff | 7/10 | 4h | [#data-mesh](#tag-data-mesh) [#data-quality](#tag-data-quality) [#etl](#tag-etl) [#distributed-systems](#tag-distributed-systems) [#real-estate](#tag-real-estate) | Twelve sources, one truth: property data mesh federating county, MLS, permit, and satellite data |
| [ ] | 239 | [Search for Property Intelligence](./phase-3/13-propiq/ch239-search-for-property-intelligence.md) | Strong Senior | 8/10 | 4h | [#search](#tag-search) [#vector-search](#tag-vector-search) [#elasticsearch](#tag-elasticsearch) [#spatial](#tag-spatial) [#multi-tenant](#tag-multi-tenant) [#real-estate](#tag-real-estate) | Search for property intelligence with source credibility ranking and data quality disclosure |
| [ ] | 240 | [Inherited Disaster — The Data Quality Crisis](./phase-3/13-propiq/ch240-inherited-disaster-data-quality-crisis.md) | Staff | 9/10 | 5h | [#data-quality](#tag-data-quality) [#incident-response](#tag-incident-response) [#inherited-disaster](#tag-inherited-disaster) [#spike](#tag-spike) [#compliance](#tag-compliance) [#legal](#tag-legal) [#monitoring](#tag-monitoring) | SPIKE — Inherited disaster: data quality crisis from conflicting source truth in property platform |
| [ ] | 241 | [The $2M Data Room](./phase-3/13-propiq/ch241-multi-party-data-rooms.md) | Staff | 8/10 | 3h | [#data-rooms](#tag-data-rooms) [#access-control](#tag-access-control) [#abac](#tag-abac) [#audit-trails](#tag-audit-trails) [#multi-party](#tag-multi-party) [#compliance](#tag-compliance) | The $2M data room: multi-party secure collaboration for real estate transactions |
| [ ] | 242 | [The Real-Time Lie](./phase-3/13-propiq/ch242-real-time-market-aggregation.md) | Strong Senior | 7/10 | 2.5h | [#data-ingestion](#tag-data-ingestion) [#real-time](#tag-real-time) [#streaming](#tag-streaming) [#hybrid-pipeline](#tag-hybrid-pipeline) [#api-integration](#tag-api-integration) [#etl](#tag-etl) | The real-time lie: market aggregation pipeline resolving conflicting property data sources |
| [ ] | 243 | [The Garbage Search](./phase-3/13-propiq/ch243-search-at-property-scale.md) | Staff | 8/10 | 3h | [#search](#tag-search) [#elasticsearch](#tag-elasticsearch) [#faceted-search](#tag-faceted-search) [#geospatial](#tag-geospatial) [#relevance-tuning](#tag-relevance-tuning) [#multi-tenant](#tag-multi-tenant) [#per-tenant-ranking](#tag-per-tenant-ranking) | Search at property scale: location queries with commercial attributes and risk factor ranking |

> **Special Files**: [Company Intro](./phase-3/13-propiq/intro.md) · [Transition Brief](./phase-3/13-propiq/transition.md)

</details>

---

**[Pattern Summary 16 — Long-Term Data & Cross-Border Compliance](./phase-3/pattern-summary-16.md)** · Chapters 230–243
> ERISA and SEC compliance, 50-year data retention, multi-jurisdiction sovereignty, property data mesh, conflicting source truth resolution, multi-party data rooms.

<details>
<summary>
  <strong>Company 14 — NovaSports</strong> · Sports Analytics / Real-Time Event Processing · Chapters 245–251
  <br><em>A sports analytics platform handling 50M events/second. The chaos engineering game day became a real incident during a championship broadcast.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 245 | [50 Million Events Per Second](./phase-3/14-novasports/ch245-50-million-events-per-second.md) | Staff | 8/10 | 4h | [#realtime](#tag-realtime) [#messaging](#tag-messaging) [#distributed](#tag-distributed) [#kafka](#tag-kafka) [#streaming](#tag-streaming) | Event streaming at 50M events/sec for live sports with fan feed, analytics, and sportsbook isolation |
| [ ] | 246 | [The World is Watching (And They're All in London)](./phase-3/14-novasports/ch246-multi-region-active-active.md) | Staff | 8/10 | 4h | [#multi-region](#tag-multi-region) [#active-active](#tag-active-active) [#data-residency](#tag-data-residency) [#GDPR](#tag-gdpr) [#conflict-resolution](#tag-conflict-resolution) [#distributed](#tag-distributed) | Multi-region active-active for global sports with latency-aware routing and GDPR consistency |
| [ ] | 247 | [35 Million Opinions, Updated in Real Time](./phase-3/14-novasports/ch247-personalization-at-scale.md) | Staff | 9/10 | 4h | [#ml-systems](#tag-ml-systems) [#feature-stores](#tag-feature-stores) [#personalization](#tag-personalization) [#real-time](#tag-real-time) [#recommendation](#tag-recommendation) [#cache](#tag-cache) | Personalization at scale: 35M real-time sports content recommendations per match |
| [ ] | 248 | [Seven Seconds to National Television](./phase-3/14-novasports/ch248-real-time-sports-analytics.md) | Staff | 9/10 | 4h | [#real-time](#tag-real-time) [#streaming](#tag-streaming) [#analytics](#tag-analytics) [#SLA](#tag-sla) [#broadcast](#tag-broadcast) [#rolling-windows](#tag-rolling-windows) [#observability](#tag-observability) | Seven seconds to national television: sports analytics pipeline with sub-SLA computation |
| [ ] | 249 | [The Game Day That Went Wrong](./phase-3/14-novasports/ch249-the-game-day-that-went-wrong.md) | Staff | 10/10 | 5h | [#chaos-engineering](#tag-chaos-engineering) [#incident-response](#tag-incident-response) [#blast-radius](#tag-blast-radius) [#circuit-breakers](#tag-circuit-breakers) [#postmortem](#tag-postmortem) [#spike-production-down](#tag-spike-production-down) | SPIKE — The game day that went wrong: chaos exercise triggers real production failure |
| [ ] | 250 | [The Postmortem (And the Architecture We Should Have Built)](./phase-3/14-novasports/ch250-blameless-postmortem-and-redesign.md) | Staff | 8/10 | 3h | [#postmortem](#tag-postmortem) [#blameless](#tag-blameless) [#chaos-engineering](#tag-chaos-engineering) [#bulkhead](#tag-bulkhead) [#blast-radius](#tag-blast-radius) [#incident-management](#tag-incident-management) | Blameless postmortem: architectural redesign following sports platform outage |
| [ ] | 251 | [Before Next Season](./phase-3/14-novasports/ch251-streaming-architecture-review.md) | Staff | 9/10 | 4h | [#architecture-review](#tag-architecture-review) [#streaming](#tag-streaming) [#cost-engineering](#tag-cost-engineering) [#capacity-planning](#tag-capacity-planning) [#kafka](#tag-kafka) [#performance-optimization](#tag-performance-optimization) | Before next season: streaming architecture review for championship-scale reliability |

> **Special Files**: [Company Intro](./phase-3/14-novasports/intro.md) · [Transition Brief](./phase-3/14-novasports/transition.md)

</details>

<details>
<summary>
  <strong>Company 15 — MindScale</strong> · Mental Health & Behavioral Health Tech / Crisis Detection · Chapters 252–257
  <br><em>A mental health platform where data isolation is a matter of patient safety and federal law under 42 CFR Part 2.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 252 | [The Most Sensitive Data in the Room](./phase-3/15-mindscale/ch252-the-most-sensitive-data.md) | Staff | 9/10 | 4h | [#privacy](#tag-privacy) [#HIPAA](#tag-hipaa) [#42CFR-Part2](#tag-42cfr-part2) [#compliance](#tag-compliance) [#data-governance](#tag-data-governance) [#mental-health](#tag-mental-health) [#tenant-isolation](#tag-tenant-isolation) | The most sensitive data in the room: mental health data isolation under 42 CFR Part 2 |
| [ ] | 253 | [The Precision-Recall Dilemma](./phase-3/15-mindscale/ch253-crisis-detection-ml.md) | Staff | 9/10 | 3h | [#ml-systems](#tag-ml-systems) [#crisis-detection](#tag-crisis-detection) [#mental-health](#tag-mental-health) [#precision-recall](#tag-precision-recall) [#human-in-the-loop](#tag-human-in-the-loop) [#system-design](#tag-system-design) | The precision-recall dilemma: ML crisis detection model for mental health interventions |
| [ ] | 254 | [The Consent That Can't Be a Toggle](./phase-3/15-mindscale/ch254-42-cfr-part-2-compliance.md) | Staff | 9/10 | 3h | [#42cfr](#tag-42cfr) [#hipaa](#tag-hipaa) [#compliance](#tag-compliance) [#consent-management](#tag-consent-management) [#data-access](#tag-data-access) [#healthcare](#tag-healthcare) [#substance-use](#tag-substance-use) | The consent that can't be a toggle: 42 CFR Part 2 compliance for substance use disorder |
| [ ] | 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | Staff | 10/10 | 4h | [#notifications](#tag-notifications) [#crisis-detection](#tag-crisis-detection) [#hipaa](#tag-hipaa) [#human-in-the-loop](#tag-human-in-the-loop) [#escalation](#tag-escalation) [#mental-health](#tag-mental-health) [#42cfr](#tag-42cfr) [#notification-fatigue](#tag-notification-fatigue) [#audit-trail](#tag-audit-trail) | The scarcest resource: final evolution of notification system with crisis-aware routing |
| [ ] | 256 | [The 23 Percent](./phase-3/15-mindscale/ch256-re-identification-risk-mitigation.md) | Staff | 9/10 | 3h | [#privacy](#tag-privacy) [#de-identification](#tag-de-identification) [#k-anonymity](#tag-k-anonymity) [#differential-privacy](#tag-differential-privacy) [#data-minimization](#tag-data-minimization) [#research](#tag-research) [#hipaa](#tag-hipaa) [#re-identification](#tag-re-identification) | The 23 percent: re-identification risk mitigation for aggregated mental health analytics |
| [ ] | 257 | [Privacy by Forgetting](./phase-3/15-mindscale/ch257-the-data-minimization-architecture.md) | Staff | 8/10 | 3h | [#data-minimization](#tag-data-minimization) [#privacy-by-design](#tag-privacy-by-design) [#gdpr](#tag-gdpr) [#hipaa](#tag-hipaa) [#retention-policy](#tag-retention-policy) [#right-to-erasure](#tag-right-to-erasure) [#purpose-bound-storage](#tag-purpose-bound-storage) [#event-log-deletion](#tag-event-log-deletion) | Privacy by forgetting: data minimization architecture preserving intervention efficacy |

> **Special Files**: [Company Intro](./phase-3/15-mindscale/intro.md) · [Transition Brief](./phase-3/15-mindscale/transition.md)

</details>

---

**[Pattern Summary 17 — Staff Engineering & Organizational Patterns](./phase-3/pattern-summary-17.md)** · Chapters 245–257
> Blameless postmortem culture, chaos engineering at org level, 42 CFR Part 2 mental health compliance, privacy by design, data minimization architecture, re-identification risk.

> ⚠️ **[FINAL CHECKPOINT — The Staff Interview — Final Checkpoint](./phase-3/checkpoint-final.md)** — *Staff-Level Promotion Gate*
> Most ambiguous brief in the curriculum. Design at scale, model costs, address compliance, present trade-offs, AND explain what you would NOT build and why. Marcus Webb sends a Slack message the night before.

<details>
<summary>
  <strong>Company 16 — QuantumEdge</strong> · Quantum Computing Platform / Hybrid Circuits · Chapters 260–265
  <br><em>A quantum-classical hybrid computing platform. Nobody has built this at production scale before.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 260 | [The Hybrid Architecture](./phase-3/16-quantumedge/ch260-the-hybrid-architecture.md) | Staff | 8/10 | 4h | [#distributed](#tag-distributed) [#scheduling](#tag-scheduling) [#compute](#tag-compute) [#api-design](#tag-api-design) [#capacity-planning](#tag-capacity-planning) | Hybrid quantum-classical scheduler for QPU + GPU resources with cost optimization |
| [ ] | 261 | [The Quantum API Gateway and Auth (Auth System: 6th Evolution)](./phase-3/16-quantumedge/ch261-quantum-api-gateway-and-auth.md) | Staff | 9/10 | 4h | [#auth](#tag-auth) [#api-design](#tag-api-design) [#rate-limiting](#tag-rate-limiting) [#compliance](#tag-compliance) [#security](#tag-security) | Quantum API gateway and auth for multi-tenant QPU service with job isolation |
| [ ] | 262 | [Circuit Job Orchestration](./phase-3/16-quantumedge/ch262-circuit-job-orchestration.md) | Staff | 8/10 | 4h | [#distributed](#tag-distributed) [#messaging](#tag-messaging) [#pipeline](#tag-pipeline) [#performance](#tag-performance) [#scheduling](#tag-scheduling) | Circuit job orchestration across heterogeneous QPU types with coherence profiles |
| [ ] | 263 | [Error Mitigation at Scale](./phase-3/16-quantumedge/ch263-error-mitigation-at-scale.md) | Staff | 9/10 | 4h | [#performance](#tag-performance) [#cost-engineering](#tag-cost-engineering) [#distributed](#tag-distributed) [#ml-systems](#tag-ml-systems) | Error mitigation at scale: noise-aware compilation and results correction for QPU |
| [ ] | 264 | [Hybrid Pipelines — Chemistry Simulation](./phase-3/16-quantumedge/ch264-hybrid-pipelines-chemistry-simulation.md) | Staff | 8/10 | 4h | [#pipeline](#tag-pipeline) [#distributed](#tag-distributed) [#observability](#tag-observability) [#reliability](#tag-reliability) [#data-pipeline](#tag-data-pipeline) | Hybrid classical-quantum pipeline for chemistry simulation with QPU advantage modeling |
| [ ] | 265 | [Quantum Platform Economics](./phase-3/16-quantumedge/ch265-quantum-platform-economics.md) | Staff | 9/10 | 5h | [#cost-engineering](#tag-cost-engineering) [#capacity-planning](#tag-capacity-planning) [#finops](#tag-finops) [#distributed](#tag-distributed) | Quantum platform economics: per-shot cost modeling and infrastructure efficiency |

> **Special Files**: [Company Intro](./phase-3/16-quantumedge/intro.md) · [Transition Brief](./phase-3/16-quantumedge/transition.md)

</details>

---

**[Pattern Summary 18 — Frontier Systems & Platform Economics](./phase-3/pattern-summary-18.md)** · Chapters 260–270
> Quantum-classical hybrid scheduling, per-shot cost modeling, virtual power plant dispatch, grid stability algorithms, heterogeneous protocol aggregation, energy sovereignty architecture.

<details>
<summary>
  <strong>Company 17 — GlacierGrid</strong> · Energy / Distributed Solar & Wind Grid · Chapters 266–271
  <br><em>A virtual power plant dispatching 350K distributed energy resources. The dispatch algorithm just caused a grid cascade.</em>
</summary>

| | Ch | Title | Level | Diff | Time | Tags | What You'll Design |
|---|---|-------|-------|------|------|------|---------------------|
| [ ] | 266 | [The Virtual Power Plant](./phase-3/17-glaciergrid/ch266-the-virtual-power-plant.md) | Staff | 9/10 | 3h | [#vpp](#tag-vpp) [#grid](#tag-grid) [#real-time](#tag-real-time) [#iot-at-scale](#tag-iot-at-scale) [#incident-response](#tag-incident-response) [#demand-response](#tag-demand-response) | Virtual power plant dispatching 350K distributed energy resources for grid demand response |
| [ ] | 267 | [The 500ms Problem](./phase-3/17-glaciergrid/ch267-grid-edge-intelligence.md) | Staff | 9/10 | 3h | [#grid-edge](#tag-grid-edge) [#dispatch](#tag-dispatch) [#latency](#tag-latency) [#real-time](#tag-real-time) [#iot](#tag-iot) [#distributed](#tag-distributed) | The 500ms problem: grid edge intelligence enabling local frequency support without central control |
| [ ] | 268 | [The Numbers Are Wrong](./phase-3/17-glaciergrid/ch268-distributed-solar-wind-aggregation.md) | Staff | 8/10 | 3h | [#data-ingestion](#tag-data-ingestion) [#heterogeneous-protocols](#tag-heterogeneous-protocols) [#temporal-alignment](#tag-temporal-alignment) [#iot](#tag-iot) [#aggregation](#tag-aggregation) | The numbers are wrong: heterogeneous renewable source aggregation with temporal alignment |
| [ ] | 269 | [Three ISOs, One Morning](./phase-3/17-glaciergrid/ch269-production-down-grid-cascade.md) | Staff | 10/10 | 4h | [#incident-response](#tag-incident-response) [#cascade-failure](#tag-cascade-failure) [#bulkheads](#tag-bulkheads) [#circuit-breaker](#tag-circuit-breaker) [#grid](#tag-grid) [#production-down](#tag-production-down) | SPIKE — Three ISOs, one morning: cascade failure from dispatch algorithm in grid operations |
| [ ] | 270 | [The Demand Cliff](./phase-3/17-glaciergrid/ch270-real-time-grid-stability.md) | Staff | 10/10 | 4h | [#grid-stability](#tag-grid-stability) [#optimization](#tag-optimization) [#real-time](#tag-real-time) [#frequency-regulation](#tag-frequency-regulation) [#algorithm-design](#tag-algorithm-design) [#dispatch](#tag-dispatch) [#constraint-programming](#tag-constraint-programming) [#vpp](#tag-vpp) | The demand cliff: real-time grid stability monitoring with frequency regulation and dispatch |
| [ ] | 271 | [The Call](./phase-3/17-glaciergrid/ch271-the-final-webb-conversation.md) | Staff | N/A (narrative)/10 | 30 minutes (reading only) | [#mentor](#tag-mentor) [#narrative](#tag-narrative) [#career-arc](#tag-career-arc) [#marcus-webb](#tag-marcus-webb) [#reflection](#tag-reflection) [#staff-engineer](#tag-staff-engineer) | The Call — Marcus Webb's final retirement conversation (narrative, no deliverables) |

> **Special Files**: [Company Intro](./phase-3/17-glaciergrid/intro.md) · [Transition Brief](./phase-3/17-glaciergrid/transition.md)

</details>

---

**[Grand Finale — The First DM](./phase-3/grand-finale.md)** · *Narrative chapter — no deliverables*
> You are now a Staff Engineer. A new hire messages you on Slack with a question about the payment pipeline you designed in Chapter 1.
> You respond the way Marcus Webb used to respond to you.
> Your final message: *"Let me tell you a story."*

---

## Tag Index

<details>
<summary>Expand full tag index (click to open)</summary>

<a name="tag-21cfr"></a>

<details>
<summary><strong>#21cfr</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 183 | [The Clinical Data Platform](./phase-3/05-neuralbridge/ch183-the-clinical-data-platform.md) | NeuralBridge | 3 |
| 222 | [The Retroactive Proof](./phase-3/10-pharmasync/ch222-the-validation-protocol.md) | PharmaSync | 3 |

</details>

<a name="tag-42cfr"></a>

<details>
<summary><strong>#42cfr</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 254 | [The Consent That Can't Be a Toggle](./phase-3/15-mindscale/ch254-42-cfr-part-2-compliance.md) | MindScale | 3 |
| 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | MindScale | 3 |

</details>

<a name="tag-gdpr"></a>

<details>
<summary><strong>#GDPR</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 173 | [Where Are the Embeddings?](./phase-3/04-lexcore/ch173-design-review-ambush-spike.md) | LexCore | 3 |
| 246 | [The World is Watching (And They're All in London)](./phase-3/14-novasports/ch246-multi-region-active-active.md) | NovaSports | 3 |

</details>

<a name="tag-rag"></a>

<details>
<summary><strong>#RAG</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 172 | [The Chunking Pipeline](./phase-3/04-lexcore/ch172-the-chunking-pipeline.md) | LexCore | 3 |
| 176 | [Find the Precedent](./phase-3/04-lexcore/ch176-search-at-legal-scale.md) | LexCore | 3 |

</details>

<a name="tag-rfc"></a>

<details>
<summary><strong>#RFC</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 163 | [The Exit Brief](./phase-3/02-ironwatch/ch163-the-exit-brief.md) | IronWatch | 3 |
| 174 | [The RFC That Failed](./phase-3/04-lexcore/ch174-the-rfc-that-failed.md) | LexCore | 3 |
| 175 | [The RFC That Succeeded](./phase-3/04-lexcore/ch175-the-rfc-that-succeeded.md) | LexCore | 3 |

</details>

<a name="tag-ab-testing"></a>

<details>
<summary><strong>#ab-testing</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 117 | [A/B Testing Infrastructure](./phase-2/08-venueflow/ch117-a-b-testing-infrastructure.md) | VenueFlow | 2 |
| 128 | [Experimentation Platform — Contradictory A/B Results](./phase-2/10-buildright/ch128-a-b-testing-and-experimentation.md) | BuildRight | 2 |

</details>

<a name="tag-abac"></a>

<details>
<summary><strong>#abac</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 94 | [Advanced Authorization — RBAC vs ABAC and Policy-as-Code](./phase-2/05-sentinelops/ch94-auth-fourth-appearance.md) | SentinelOps | 2 |
| 241 | [The $2M Data Room](./phase-3/13-propiq/ch241-multi-party-data-rooms.md) | PropIQ | 3 |

</details>

<a name="tag-access-control"></a>

<details>
<summary><strong>#access-control</strong> — 4 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 158 | [Zero Trust, Zero Margin](./phase-3/02-ironwatch/ch158-zero-trust-zero-margin.md) | IronWatch | 3 |
| 159 | [The Compartment](./phase-3/02-ironwatch/ch159-the-compartment.md) | IronWatch | 3 |
| 171 | [Privileged Information](./phase-3/04-lexcore/ch171-privileged-information.md) | LexCore | 3 |
| 241 | [The $2M Data Room](./phase-3/13-propiq/ch241-multi-party-data-rooms.md) | PropIQ | 3 |

</details>

<a name="tag-active-active"></a>

<details>
<summary><strong>#active-active</strong> — 5 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 68 | [Multi-Region Active-Active Architecture](./phase-2/01-stratum-systems/ch68-multi-region-active-active-intro.md) | Stratum Systems | 2 |
| 84 | [Multi-Region Conflict Resolution](./phase-2/04-giggrid/ch84-multi-region-conflict-resolution.md) | GigGrid | 2 |
| 125 | [Five Country Launches in One Quarter](./phase-2/09-telanova/ch125-multi-region-active-active-advanced.md) | TeleNova | 2 |
| 200 | [Five Nines for a Moving Vehicle](./phase-3/07-automesh/ch200-the-partnership-aftermath.md) | AutoMesh | 3 |
| 246 | [The World is Watching (And They're All in London)](./phase-3/14-novasports/ch246-multi-region-active-active.md) | NovaSports | 3 |

</details>

<a name="tag-air-gapped"></a>

<details>
<summary><strong>#air-gapped</strong> — 3 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 157 | [The Briefing Room](./phase-3/02-ironwatch/ch157-the-briefing-room.md) | IronWatch | 3 |
| 158 | [Zero Trust, Zero Margin](./phase-3/02-ironwatch/ch158-zero-trust-zero-margin.md) | IronWatch | 3 |
| 161 | [The Air Gap](./phase-3/02-ironwatch/ch161-the-air-gap.md) | IronWatch | 3 |

</details>

<a name="tag-alerting"></a>

<details>
<summary><strong>#alerting</strong> — 6 chapters across 6 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 25 | [Build Observability Into the Sensor Platform](./phase-1/05-agrosense/ch25-observability-and-monitoring.md) | AgroSense | 1 |
| 57 | [Advanced Observability — When Models Lie Silently](./phase-1/11-luminaryai/ch57-advanced-observability-ml-systems.md) | LuminaryAI | 1 |
| 74 | [Advanced Observability — SLOs, SLIs, and Error Budgets](./phase-2/02-nexacare/ch74-advanced-observability-slos.md) | NexaCare | 2 |
| 109 | [Grid Reliability SLOs — When 99.9% Is Not Good Enough](./phase-2/07-crestline-energy/ch109-slo-sli-error-budget-design.md) | Crestline Energy | 2 |
| 140 | [Life-Safety Alerting — When False Positives Kill Trust](./phase-2/12-prismhealth/ch140-advanced-observability-alerting.md) | PrismHealth | 2 |
| 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | HarvestAI | 3 |

</details>

<a name="tag-analytics"></a>

<details>
<summary><strong>#analytics</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 103 | [Data Warehousing — OLAP vs OLTP](./phase-2/06-lightspeedretail/ch103-data-warehousing-olap-vs-oltp.md) | LightspeedRetail | 2 |
| 208 | [The Twelve Factories](./phase-3/08-forgesense/ch208-factory-data-mesh.md) | ForgeSense | 3 |
| 248 | [Seven Seconds to National Television](./phase-3/14-novasports/ch248-real-time-sports-analytics.md) | NovaSports | 3 |

</details>

<a name="tag-apache-iceberg"></a>

<details>
<summary><strong>#apache-iceberg</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 107 | [Grid Analytics — Apache Iceberg and Columnar Storage](./phase-2/07-crestline-energy/ch107-data-warehousing-columnar-storage.md) | Crestline Energy | 2 |
| 135 | [Twelve Petabytes and Counting](./phase-2/11-axiom-labs/ch135-columnar-storage-petabyte-scale.md) | Axiom Labs | 2 |

</details>

<a name="tag-api-design"></a>

<details>
<summary><strong>#api-design</strong> — 17 chapters across 11 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 01 | [Design the Payment Processing Pipeline](./phase-1/01-novapay/ch01-payment-processing-pipeline.md) | NovaPay | 1 |
| 06 | [Design the Authentication and Authorization System](./phase-1/02-meridian-health/ch06-jwt-oauth2-auth-system.md) | MeridianHealth | 1 |
| 10 | [Scale 10x by Friday — The Northview Surge](./phase-1/02-meridian-health/ch10-ehr-integration-platform-design.md) | MeridianHealth | 1 |
| 14 | [Design the Load Balancing Strategy for a Heterogeneous Fleet](./phase-1/03-velotrack/ch14-load-balancing-strategies.md) | VeloTrack | 1 |
| 19 | [Presigned URLs and the Pirates Problem](./phase-1/04-beacon-media/ch19-presigned-urls-and-content-security.md) | Beacon Media | 1 |
| 22 | [The Recommendation API is Melting](./phase-1/04-beacon-media/ch22-content-recommendation-and-caching.md) | Beacon Media | 1 |
| 40 | [Rate Limiting for Government APIs — The Enumeration Attack](./phase-1/08-civicos/ch40-rate-limiting-second-appearance.md) | CivicOS | 1 |
| 42 | [Design for Accessibility, Resilience, and the Citizen Who Can't Retry](./phase-1/08-civicos/ch42-accessibility-and-resilience-for-government-systems.md) | CivicOS | 1 |
| 154 | [Ground Truth](./phase-3/01-orbitcore/ch154-ground-truth.md) | OrbitCore | 3 |
| 166 | [Article 6 Compliance](./phase-3/03-carbonledger/ch166-article-6-compliance.md) | CarbonLedger | 3 |
| 186 | [The Risk Engine](./phase-3/06-shieldmutual/ch186-the-risk-engine.md) | ShieldMutual | 3 |
| 193 | [V2X at Scale](./phase-3/07-automesh/ch193-v2x-at-scale.md) | AutoMesh | 3 |
| 217 | [Lab Data Integration](./phase-3/10-pharmasync/ch217-lab-data-integration.md) | PharmaSync | 3 |
| 219 | [The Six-Week Submission](./phase-3/10-pharmasync/ch219-clinical-data-management.md) | PharmaSync | 3 |
| 220 | [Gerald Must Die](./phase-3/10-pharmasync/ch220-the-lims-deprecation.md) | PharmaSync | 3 |
| 260 | [The Hybrid Architecture](./phase-3/16-quantumedge/ch260-the-hybrid-architecture.md) | QuantumEdge | 3 |
| 261 | [The Quantum API Gateway and Auth (Auth System: 6th Evolution)](./phase-3/16-quantumedge/ch261-quantum-api-gateway-and-auth.md) | QuantumEdge | 3 |

</details>

<a name="tag-api-gateway"></a>

<details>
<summary><strong>#api-gateway</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 29 | [Design the API Gateway and Rate Limiting System (First Appearance)](./phase-1/06-cloudstack/ch29-api-gateway-and-rate-limiting.md) | CloudStack | 1 |
| 87 | [Platform API Rate Limiting (4th Appearance)](./phase-2/04-giggrid/ch87-rate-limiting-fourth-appearance.md) | GigGrid | 2 |
| 148 | [300 Internal APIs, Zero Deprecation Policy](./phase-2/13-vertexcloud/ch148-api-governance-at-org-scale.md) | VertexCloud | 2 |

</details>

<a name="tag-append-only"></a>

<details>
<summary><strong>#append-only</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 72 | [Advanced Audit Trails at Scale](./phase-2/02-nexacare/ch72-advanced-audit-trails-at-scale.md) | NexaCare | 2 |
| 77 | [Low-Latency Event Sourcing for Trade Execution](./phase-2/03-tradespark/ch77-low-latency-event-sourcing.md) | TradeSpark | 2 |

</details>

<a name="tag-architecture"></a>

<details>
<summary><strong>#architecture</strong> — 7 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 05 | [Design Review Ambush — PCI-DSS or the Audit Fails](./phase-1/01-novapay/ch05-pci-dss-compliance-architecture.md) | NovaPay | 1 |
| 28 | [Multi-Tenant Architecture and Tenant Isolation](./phase-1/06-cloudstack/ch28-multi-tenancy-architecture.md) | CloudStack | 1 |
| 39 | [Design the Data Sovereignty Architecture for Government Clients](./phase-1/08-civicos/ch39-data-sovereignty-and-compliance-architecture.md) | CivicOS | 1 |
| 157 | [The Briefing Room](./phase-3/02-ironwatch/ch157-the-briefing-room.md) | IronWatch | 3 |
| 179 | [FDA 510(k) Architecture](./phase-3/05-neuralbridge/ch179-fda-510k-architecture.md) | NeuralBridge | 3 |
| 189 | [SOC2 Under Pressure](./phase-3/06-shieldmutual/ch189-soc2-under-pressure.md) | ShieldMutual | 3 |
| 237 | [A Letter to 2074](./phase-3/12-nexuswealth/ch237-the-50-year-architecture.md) | NexusWealth | 3 |

</details>

<a name="tag-architecture-review"></a>

<details>
<summary><strong>#architecture-review</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 181 | [The Systemic Failure](./phase-3/05-neuralbridge/ch181-the-systemic-failure.md) | NeuralBridge | 3 |
| 251 | [Before Next Season](./phase-3/14-novasports/ch251-streaming-architecture-review.md) | NovaSports | 3 |

</details>

<a name="tag-argocd"></a>

<details>
<summary><strong>#argocd</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 129 | [Progressive Delivery with Argo Rollouts](./phase-2/10-buildright/ch129-canary-deployments-advanced.md) | BuildRight | 2 |
| 147 | [GitOps at 2,000 Engineers](./phase-2/13-vertexcloud/ch147-advanced-deployment-pipeline.md) | VertexCloud | 2 |

</details>

<a name="tag-audit"></a>

<details>
<summary><strong>#audit</strong> — 12 chapters across 8 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 05 | [Design Review Ambush — PCI-DSS or the Audit Fails](./phase-1/01-novapay/ch05-pci-dss-compliance-architecture.md) | NovaPay | 1 |
| 07 | [Build the HIPAA-Compliant Audit Trail](./phase-1/02-meridian-health/ch07-hipaa-audit-trails.md) | MeridianHealth | 1 |
| 75 | [Secret Management with HashiCorp Vault](./phase-2/02-nexacare/ch75-secret-management-intro.md) | NexaCare | 2 |
| 158 | [Zero Trust, Zero Margin](./phase-3/02-ironwatch/ch158-zero-trust-zero-margin.md) | IronWatch | 3 |
| 159 | [The Compartment](./phase-3/02-ironwatch/ch159-the-compartment.md) | IronWatch | 3 |
| 160 | [Inherited Disaster — The Data That Crossed the Line](./phase-3/02-ironwatch/ch160-inherited-disaster-data-breach.md) | IronWatch | 3 |
| 162 | [Audit Everything](./phase-3/02-ironwatch/ch162-audit-everything.md) | IronWatch | 3 |
| 168 | [Event Sourcing for Regulators](./phase-3/03-carbonledger/ch168-event-sourcing-for-regulators.md) | CarbonLedger | 3 |
| 189 | [SOC2 Under Pressure](./phase-3/06-shieldmutual/ch189-soc2-under-pressure.md) | ShieldMutual | 3 |
| 216 | [21 CFR Part 11](./phase-3/10-pharmasync/ch216-21-cfr-part-11.md) | PharmaSync | 3 |
| 222 | [The Retroactive Proof](./phase-3/10-pharmasync/ch222-the-validation-protocol.md) | PharmaSync | 3 |
| 231 | [ERISA and SEC Compliance](./phase-3/12-nexuswealth/ch231-erisa-and-sec-compliance.md) | NexusWealth | 3 |

</details>

<a name="tag-audit-trail"></a>

<details>
<summary><strong>#audit-trail</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 124 | [5G QoS — Fair Use Without Unfair Enforcement](./phase-2/09-telanova/ch124-network-rate-limiting-qos.md) | TeleNova | 2 |
| 139 | [mTLS for HIPAA — Every Service Call Is a PHI Flow](./phase-2/12-prismhealth/ch139-service-mesh-healthcare-compliance.md) | PrismHealth | 2 |
| 183 | [The Clinical Data Platform](./phase-3/05-neuralbridge/ch183-the-clinical-data-platform.md) | NeuralBridge | 3 |
| 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | MindScale | 3 |

</details>

<a name="tag-audit-trails"></a>

<details>
<summary><strong>#audit-trails</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 72 | [Advanced Audit Trails at Scale](./phase-2/02-nexacare/ch72-advanced-audit-trails-at-scale.md) | NexaCare | 2 |
| 241 | [The $2M Data Room](./phase-3/13-propiq/ch241-multi-party-data-rooms.md) | PropIQ | 3 |

</details>

<a name="tag-auth"></a>

<details>
<summary><strong>#auth</strong> — 8 chapters across 8 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 06 | [Design the Authentication and Authorization System](./phase-1/02-meridian-health/ch06-jwt-oauth2-auth-system.md) | MeridianHealth | 1 |
| 30 | [Auth at Scale — SSO, Service Accounts, and API Keys (Second Appearance)](./phase-1/06-cloudstack/ch30-auth-system-second-appearance.md) | CloudStack | 1 |
| 41 | [SSO for 23 State Governments — Federated Identity at Scale](./phase-1/08-civicos/ch41-sso-and-identity-federation-for-government.md) | CivicOS | 1 |
| 61 | [Auth for the Most Adversarial User Base](./phase-1/12-skyroute/ch61-auth-third-appearance.md) | SkyRoute | 1 |
| 94 | [Advanced Authorization — RBAC vs ABAC and Policy-as-Code](./phase-2/05-sentinelops/ch94-auth-fourth-appearance.md) | SentinelOps | 2 |
| 123 | [Auth at 120 Million — Device Authentication and B2B2C Identity](./phase-2/09-telanova/ch123-auth-fifth-appearance.md) | TeleNova | 2 |
| 158 | [Zero Trust, Zero Margin](./phase-3/02-ironwatch/ch158-zero-trust-zero-margin.md) | IronWatch | 3 |
| 261 | [The Quantum API Gateway and Auth (Auth System: 6th Evolution)](./phase-3/16-quantumedge/ch261-quantum-api-gateway-and-auth.md) | QuantumEdge | 3 |

</details>

<a name="tag-auto-scaling"></a>

<details>
<summary><strong>#auto-scaling</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 101 | [Immutable Infrastructure and Black Friday Scaling](./phase-2/06-lightspeedretail/ch101-immutable-infrastructure-and-scaling.md) | LightspeedRetail | 2 |
| 224 | [Design for 100x, Spend 2x](./phase-3/11-harvestai/ch224-the-seasonal-spike.md) | HarvestAI | 3 |

</details>

<a name="tag-availability"></a>

<details>
<summary><strong>#availability</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 49 | [CAP Theorem in Practice — When the Network Fails](./phase-1/10-omnilogix/ch49-cap-theorem-and-eventual-consistency.md) | OmniLogix | 1 |
| 85 | [CRDTs — Conflict-Free Replicated Data Types](./phase-2/04-giggrid/ch85-crdts-introduction.md) | GigGrid | 2 |

</details>

<a name="tag-backpressure"></a>

<details>
<summary><strong>#backpressure</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 86 | [Real-Time at Scale — WebSockets, SSE, and Pub/Sub](./phase-2/04-giggrid/ch86-real-time-at-scale-websockets-sse.md) | GigGrid | 2 |
| 113 | [Real-Time at Scale — 500k Concurrent Fans](./phase-2/08-venueflow/ch113-real-time-at-scale-pub-sub.md) | VenueFlow | 2 |
| 209 | [The Ocean's Heartbeat](./phase-3/09-deepocean/ch209-ais-the-oceans-heartbeat.md) | DeepOcean | 3 |

</details>

<a name="tag-bandwidth-constrained"></a>

<details>
<summary><strong>#bandwidth-constrained</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 212 | [2,000 Miles from Anywhere](./phase-3/09-deepocean/ch212-edge-processing-at-sea.md) | DeepOcean | 3 |
| 225 | [The Model in the Field](./phase-3/11-harvestai/ch225-edge-ml-inference.md) | HarvestAI | 3 |

</details>

<a name="tag-batch-processing"></a>

<details>
<summary><strong>#batch-processing</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 108 | [The 18-Hour Batch Job](./phase-2/07-crestline-energy/ch108-streaming-vs-batch-flink.md) | Crestline Energy | 2 |
| 187 | [The Actuarial Pipeline](./phase-3/06-shieldmutual/ch187-the-actuarial-pipeline.md) | ShieldMutual | 3 |
| 232 | [The March 31st Problem](./phase-3/12-nexuswealth/ch232-actuarial-calculation-at-scale.md) | NexusWealth | 3 |

</details>

<a name="tag-bi-temporal"></a>

<details>
<summary><strong>#bi-temporal</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 112 | ["What Was the Grid State at 14:32:07 on March 3rd?"](./phase-2/07-crestline-energy/ch112-cqrs-temporal-queries.md) | Crestline Energy | 2 |
| 126 | [CQRS and Advanced Event Sourcing for Project State](./phase-2/10-buildright/ch126-cqrs-event-sourcing-advanced.md) | BuildRight | 2 |

</details>

<a name="tag-billing"></a>

<details>
<summary><strong>#billing</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 31 | [Design the Tenant Billing and Usage Metering System](./phase-1/06-cloudstack/ch31-tenant-billing-and-usage-metering.md) | CloudStack | 1 |
| 56 | [Payment Infrastructure — Billing the ML Platform](./phase-1/11-luminaryai/ch56-payment-processing-third-appearance.md) | LuminaryAI | 1 |

</details>

<a name="tag-black-friday"></a>

<details>
<summary><strong>#black-friday</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 100 | [Advanced Caching — Black Friday Thundering Herd](./phase-2/06-lightspeedretail/ch100-advanced-caching-thundering-herd.md) | LightspeedRetail | 2 |
| 101 | [Immutable Infrastructure and Black Friday Scaling](./phase-2/06-lightspeedretail/ch101-immutable-infrastructure-and-scaling.md) | LightspeedRetail | 2 |

</details>

<a name="tag-blameless"></a>

<details>
<summary><strong>#blameless</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | HarvestAI | 3 |
| 250 | [The Postmortem (And the Architecture We Should Have Built)](./phase-3/14-novasports/ch250-blameless-postmortem-and-redesign.md) | NovaSports | 3 |

</details>

<a name="tag-blast-radius"></a>

<details>
<summary><strong>#blast-radius</strong> — 5 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 92 | [Zero-Trust Architecture](./phase-2/05-sentinelops/ch92-zero-trust-architecture.md) | SentinelOps | 2 |
| 149 | [The Game Day That Became a Real Incident](./phase-2/13-vertexcloud/ch149-chaos-engineering-intro.md) | VertexCloud | 2 |
| 151 | [The Cell Problem](./phase-3/01-orbitcore/ch151-the-cell-problem.md) | OrbitCore | 3 |
| 249 | [The Game Day That Went Wrong](./phase-3/14-novasports/ch249-the-game-day-that-went-wrong.md) | NovaSports | 3 |
| 250 | [The Postmortem (And the Architecture We Should Have Built)](./phase-3/14-novasports/ch250-blameless-postmortem-and-redesign.md) | NovaSports | 3 |

</details>

<a name="tag-blue-green"></a>

<details>
<summary><strong>#blue-green</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 88 | [Canary Deployments and Feature Flags](./phase-2/04-giggrid/ch88-canary-deployments-and-feature-flags.md) | GigGrid | 2 |
| 102 | [Zero-Downtime Deploys for 120,000 POS Terminals](./phase-2/06-lightspeedretail/ch102-blue-green-deployments.md) | LightspeedRetail | 2 |

</details>

<a name="tag-cache-invalidation"></a>

<details>
<summary><strong>#cache-invalidation</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 22 | [The Recommendation API is Melting](./phase-1/04-beacon-media/ch22-content-recommendation-and-caching.md) | Beacon Media | 1 |
| 36 | [Cache Invalidation Hell](./phase-1/07-pulsecommerce/ch36-cache-invalidation-patterns.md) | PulseCommerce | 1 |

</details>

<a name="tag-cache-stampede"></a>

<details>
<summary><strong>#cache-stampede</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 82 | [Advanced Caching — Probabilistic Early Expiration](./phase-2/03-tradespark/ch82-advanced-caching-probabilistic-expiration.md) | TradeSpark | 2 |
| 100 | [Advanced Caching — Black Friday Thundering Herd](./phase-2/06-lightspeedretail/ch100-advanced-caching-thundering-herd.md) | LightspeedRetail | 2 |

</details>

<a name="tag-caching"></a>

<details>
<summary><strong>#caching</strong> — 10 chapters across 9 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 04 | [Design the Merchant Balance Cache](./phase-1/01-novapay/ch04-write-through-cache-design.md) | NovaPay | 1 |
| 22 | [The Recommendation API is Melting](./phase-1/04-beacon-media/ch22-content-recommendation-and-caching.md) | Beacon Media | 1 |
| 36 | [Cache Invalidation Hell](./phase-1/07-pulsecommerce/ch36-cache-invalidation-patterns.md) | PulseCommerce | 1 |
| 54 | [The Feature Store — Feeding Models the Right Data](./phase-1/11-luminaryai/ch54-feature-store-and-ml-pipeline.md) | LuminaryAI | 1 |
| 58 | [SPIKE — Scale 10x by Friday](./phase-1/11-luminaryai/ch58-recommendation-serving-scale.md) | LuminaryAI | 1 |
| 82 | [Advanced Caching — Probabilistic Early Expiration](./phase-2/03-tradespark/ch82-advanced-caching-probabilistic-expiration.md) | TradeSpark | 2 |
| 100 | [Advanced Caching — Black Friday Thundering Herd](./phase-2/06-lightspeedretail/ch100-advanced-caching-thundering-herd.md) | LightspeedRetail | 2 |
| 167 | [SPIKE — Scale 10x by Friday](./phase-3/03-carbonledger/ch167-scale-10x-spike.md) | CarbonLedger | 3 |
| 186 | [The Risk Engine](./phase-3/06-shieldmutual/ch186-the-risk-engine.md) | ShieldMutual | 3 |
| 223 | [When the Earth Won't Fit in a B-Tree](./phase-3/11-harvestai/ch223-geospatial-at-scale.md) | HarvestAI | 3 |

</details>

<a name="tag-canary"></a>

<details>
<summary><strong>#canary</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 88 | [Canary Deployments and Feature Flags](./phase-2/04-giggrid/ch88-canary-deployments-and-feature-flags.md) | GigGrid | 2 |
| 102 | [Zero-Downtime Deploys for 120,000 POS Terminals](./phase-2/06-lightspeedretail/ch102-blue-green-deployments.md) | LightspeedRetail | 2 |
| 129 | [Progressive Delivery with Argo Rollouts](./phase-2/10-buildright/ch129-canary-deployments-advanced.md) | BuildRight | 2 |

</details>

<a name="tag-cap-theorem"></a>

<details>
<summary><strong>#cap-theorem</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 49 | [CAP Theorem in Practice — When the Network Fails](./phase-1/10-omnilogix/ch49-cap-theorem-and-eventual-consistency.md) | OmniLogix | 1 |
| 53 | [Global Inventory Consistency — The Full Design](./phase-1/10-omnilogix/ch53-global-inventory-consistency-synthesis.md) | OmniLogix | 1 |
| 116 | [Ticket Inventory Consistency — The Taylor Swift Problem](./phase-2/08-venueflow/ch116-ticket-inventory-consistency.md) | VenueFlow | 2 |

</details>

<a name="tag-capacity-planning"></a>

<details>
<summary><strong>#capacity-planning</strong> — 12 chapters across 9 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 58 | [SPIKE — Scale 10x by Friday](./phase-1/11-luminaryai/ch58-recommendation-serving-scale.md) | LuminaryAI | 1 |
| 98 | [Kafka at Scale — 40 Billion Security Events/Day](./phase-2/05-sentinelops/ch98-advanced-kafka-security-events.md) | SentinelOps | 2 |
| 110 | [Three Years of Growth, Two Years of Budget](./phase-2/07-crestline-energy/ch110-capacity-planning-and-forecasting.md) | Crestline Energy | 2 |
| 150 | [Signal and Noise](./phase-3/01-orbitcore/ch150-signal-and-noise.md) | OrbitCore | 3 |
| 155 | [Capacity at Constellation Scale](./phase-3/01-orbitcore/ch155-capacity-at-constellation-scale.md) | OrbitCore | 3 |
| 196 | [Scale 10x — Fleet Doubles](./phase-3/07-automesh/ch196-scale-10x-fleet-doubles-spike.md) | AutoMesh | 3 |
| 198 | [The $2.3M Conversation](./phase-3/07-automesh/ch198-the-cfo-presentation.md) | AutoMesh | 3 |
| 224 | [Design for 100x, Spend 2x](./phase-3/11-harvestai/ch224-the-seasonal-spike.md) | HarvestAI | 3 |
| 235 | [The Trustees Want Answers](./phase-3/12-nexuswealth/ch235-the-board-presentation.md) | NexusWealth | 3 |
| 251 | [Before Next Season](./phase-3/14-novasports/ch251-streaming-architecture-review.md) | NovaSports | 3 |
| 260 | [The Hybrid Architecture](./phase-3/16-quantumedge/ch260-the-hybrid-architecture.md) | QuantumEdge | 3 |
| 265 | [Quantum Platform Economics](./phase-3/16-quantumedge/ch265-quantum-platform-economics.md) | QuantumEdge | 3 |

</details>

<a name="tag-cascade-failure"></a>

<details>
<summary><strong>#cascade-failure</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 191 | [Production DOWN — Claims Cascade Failure](./phase-3/06-shieldmutual/ch191-production-down-claims-cascade.md) | ShieldMutual | 3 |
| 269 | [Three ISOs, One Morning](./phase-3/17-glaciergrid/ch269-production-down-grid-cascade.md) | GlacierGrid | 3 |

</details>

<a name="tag-cdc"></a>

<details>
<summary><strong>#cdc</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 24 | [Build the ETL Pipeline and Change Data Capture](./phase-1/05-agrosense/ch24-etl-pipeline-and-cdc.md) | AgroSense | 1 |
| 55 | [ETL to Streaming — Evolving the Data Pipeline](./phase-1/11-luminaryai/ch55-etl-to-streaming-pipeline-evolution.md) | LuminaryAI | 1 |

</details>

<a name="tag-cdn"></a>

<details>
<summary><strong>#cdn</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 18 | [Fix the CDN — $2M/Month and Falling](./phase-1/04-beacon-media/ch18-blob-storage-and-cdn-architecture.md) | Beacon Media | 1 |
| 19 | [Presigned URLs and the Pirates Problem](./phase-1/04-beacon-media/ch19-presigned-urls-and-content-security.md) | Beacon Media | 1 |

</details>

<a name="tag-chaos-engineering"></a>

<details>
<summary><strong>#chaos-engineering</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 149 | [The Game Day That Became a Real Incident](./phase-2/13-vertexcloud/ch149-chaos-engineering-intro.md) | VertexCloud | 2 |
| 249 | [The Game Day That Went Wrong](./phase-3/14-novasports/ch249-the-game-day-that-went-wrong.md) | NovaSports | 3 |
| 250 | [The Postmortem (And the Architecture We Should Have Built)](./phase-3/14-novasports/ch250-blameless-postmortem-and-redesign.md) | NovaSports | 3 |

</details>

<a name="tag-circuit-breaker"></a>

<details>
<summary><strong>#circuit-breaker</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 191 | [Production DOWN — Claims Cascade Failure](./phase-3/06-shieldmutual/ch191-production-down-claims-cascade.md) | ShieldMutual | 3 |
| 269 | [Three ISOs, One Morning](./phase-3/17-glaciergrid/ch269-production-down-grid-cascade.md) | GlacierGrid | 3 |

</details>

<a name="tag-circuit-breakers"></a>

<details>
<summary><strong>#circuit-breakers</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 83 | [Service Mesh and mTLS Introduction](./phase-2/03-tradespark/ch83-service-mesh-mtls-intro.md) | TradeSpark | 2 |
| 249 | [The Game Day That Went Wrong](./phase-3/14-novasports/ch249-the-game-day-that-went-wrong.md) | NovaSports | 3 |

</details>

<a name="tag-compaction"></a>

<details>
<summary><strong>#compaction</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 71 | [Database Internals — B-Tree vs LSM-Tree](./phase-2/02-nexacare/ch71-database-internals-btree-vs-lsm.md) | NexaCare | 2 |
| 78 | [Database Internals — LSM-Tree Compaction Storm](./phase-2/03-tradespark/ch78-database-internals-deep-dive.md) | TradeSpark | 2 |

</details>

<a name="tag-compliance"></a>

<details>
<summary><strong>#compliance</strong> — 61 chapters across 25 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 05 | [Design Review Ambush — PCI-DSS or the Audit Fails](./phase-1/01-novapay/ch05-pci-dss-compliance-architecture.md) | NovaPay | 1 |
| 07 | [Build the HIPAA-Compliant Audit Trail](./phase-1/02-meridian-health/ch07-hipaa-audit-trails.md) | MeridianHealth | 1 |
| 08 | [Data Residency — The State That Wants Its Data Back](./phase-1/02-meridian-health/ch08-data-residency-and-hipaa-architecture.md) | MeridianHealth | 1 |
| 34 | [Payment Integration at Scale — Scaling Challenges (Second Appearance)](./phase-1/07-pulsecommerce/ch34-payment-processing-second-appearance.md) | PulseCommerce | 1 |
| 39 | [Design the Data Sovereignty Architecture for Government Clients](./phase-1/08-civicos/ch39-data-sovereignty-and-compliance-architecture.md) | CivicOS | 1 |
| 47 | [Design Review Ambush — The Academic Integrity Architecture](./phase-1/09-neurolearn/ch47-academic-integrity-monitoring.md) | NeuroLearn | 1 |
| 70 | [GDPR Deletion Pipelines](./phase-2/02-nexacare/ch70-gdpr-deletion-pipelines.md) | NexaCare | 2 |
| 72 | [Advanced Audit Trails at Scale](./phase-2/02-nexacare/ch72-advanced-audit-trails-at-scale.md) | NexaCare | 2 |
| 73 | [Data Residency Across Multiple Jurisdictions](./phase-2/02-nexacare/ch73-data-residency-multi-jurisdiction.md) | NexaCare | 2 |
| 94 | [Advanced Authorization — RBAC vs ABAC and Policy-as-Code](./phase-2/05-sentinelops/ch94-auth-fourth-appearance.md) | SentinelOps | 2 |
| 95 | [Secret Management at Scale](./phase-2/05-sentinelops/ch95-secret-management-at-scale.md) | SentinelOps | 2 |
| 97 | [Security Incident Forensics Pipeline](./phase-2/05-sentinelops/ch97-security-incident-forensics-pipeline.md) | SentinelOps | 2 |
| 112 | ["What Was the Grid State at 14:32:07 on March 3rd?"](./phase-2/07-crestline-energy/ch112-cqrs-temporal-queries.md) | Crestline Energy | 2 |
| 126 | [CQRS and Advanced Event Sourcing for Project State](./phase-2/10-buildright/ch126-cqrs-event-sourcing-advanced.md) | BuildRight | 2 |
| 131 | [GDPR Erasure in Event-Sourced Systems](./phase-2/10-buildright/ch131-compliance-gdpr-erasure-and-event-logs.md) | BuildRight | 2 |
| 137 | [Genomic PII in the Application Logs](./phase-2/11-axiom-labs/ch137-hipaa-compliance-remediation.md) | Axiom Labs | 2 |
| 139 | [mTLS for HIPAA — Every Service Call Is a PHI Flow](./phase-2/12-prismhealth/ch139-service-mesh-healthcare-compliance.md) | PrismHealth | 2 |
| 152 | [Sovereignty in Orbit](./phase-3/01-orbitcore/ch152-sovereignty-in-orbit.md) | OrbitCore | 3 |
| 157 | [The Briefing Room](./phase-3/02-ironwatch/ch157-the-briefing-room.md) | IronWatch | 3 |
| 158 | [Zero Trust, Zero Margin](./phase-3/02-ironwatch/ch158-zero-trust-zero-margin.md) | IronWatch | 3 |
| 159 | [The Compartment](./phase-3/02-ironwatch/ch159-the-compartment.md) | IronWatch | 3 |
| 160 | [Inherited Disaster — The Data That Crossed the Line](./phase-3/02-ironwatch/ch160-inherited-disaster-data-breach.md) | IronWatch | 3 |
| 162 | [Audit Everything](./phase-3/02-ironwatch/ch162-audit-everything.md) | IronWatch | 3 |
| 163 | [The Exit Brief](./phase-3/02-ironwatch/ch163-the-exit-brief.md) | IronWatch | 3 |
| 165 | [The Ledger That Cannot Lie](./phase-3/03-carbonledger/ch165-the-ledger-that-cannot-lie.md) | CarbonLedger | 3 |
| 166 | [Article 6 Compliance](./phase-3/03-carbonledger/ch166-article-6-compliance.md) | CarbonLedger | 3 |
| 168 | [Event Sourcing for Regulators](./phase-3/03-carbonledger/ch168-event-sourcing-for-regulators.md) | CarbonLedger | 3 |
| 170 | [The Permanence Problem](./phase-3/03-carbonledger/ch170-the-permanence-problem.md) | CarbonLedger | 3 |
| 171 | [Privileged Information](./phase-3/04-lexcore/ch171-privileged-information.md) | LexCore | 3 |
| 173 | [Where Are the Embeddings?](./phase-3/04-lexcore/ch173-design-review-ambush-spike.md) | LexCore | 3 |
| 175 | [The RFC That Succeeded](./phase-3/04-lexcore/ch175-the-rfc-that-succeeded.md) | LexCore | 3 |
| 176 | [Find the Precedent](./phase-3/04-lexcore/ch176-search-at-legal-scale.md) | LexCore | 3 |
| 177 | [The Privilege Log](./phase-3/04-lexcore/ch177-the-privilege-log.md) | LexCore | 3 |
| 179 | [FDA 510(k) Architecture](./phase-3/05-neuralbridge/ch179-fda-510k-architecture.md) | NeuralBridge | 3 |
| 180 | [Brain Data Privacy](./phase-3/05-neuralbridge/ch180-brain-data-privacy.md) | NeuralBridge | 3 |
| 183 | [The Clinical Data Platform](./phase-3/05-neuralbridge/ch183-the-clinical-data-platform.md) | NeuralBridge | 3 |
| 186 | [The Risk Engine](./phase-3/06-shieldmutual/ch186-the-risk-engine.md) | ShieldMutual | 3 |
| 187 | [The Actuarial Pipeline](./phase-3/06-shieldmutual/ch187-the-actuarial-pipeline.md) | ShieldMutual | 3 |
| 189 | [SOC2 Under Pressure](./phase-3/06-shieldmutual/ch189-soc2-under-pressure.md) | ShieldMutual | 3 |
| 192 | [State Regulation as Architecture](./phase-3/06-shieldmutual/ch192-state-regulation-as-architecture.md) | ShieldMutual | 3 |
| 205 | [The Ghost in the Machine](./phase-3/08-forgesense/ch205-inherited-disaster-scada.md) | ForgeSense | 3 |
| 214 | [Flag State to Port State](./phase-3/09-deepocean/ch214-maritime-compliance.md) | DeepOcean | 3 |
| 216 | [21 CFR Part 11](./phase-3/10-pharmasync/ch216-21-cfr-part-11.md) | PharmaSync | 3 |
| 219 | [The Six-Week Submission](./phase-3/10-pharmasync/ch219-clinical-data-management.md) | PharmaSync | 3 |
| 220 | [Gerald Must Die](./phase-3/10-pharmasync/ch220-the-lims-deprecation.md) | PharmaSync | 3 |
| 221 | [The Two Phantom Patients](./phase-3/10-pharmasync/ch221-production-down-clinical-trial.md) | PharmaSync | 3 |
| 222 | [The Retroactive Proof](./phase-3/10-pharmasync/ch222-the-validation-protocol.md) | PharmaSync | 3 |
| 226 | [The Three-Meter Problem](./phase-3/11-harvestai/ch226-carbon-credits-integration.md) | HarvestAI | 3 |
| 230 | [Fifty Years of Data](./phase-3/12-nexuswealth/ch230-fifty-years-of-data.md) | NexusWealth | 3 |
| 231 | [ERISA and SEC Compliance](./phase-3/12-nexuswealth/ch231-erisa-and-sec-compliance.md) | NexusWealth | 3 |
| 232 | [The March 31st Problem](./phase-3/12-nexuswealth/ch232-actuarial-calculation-at-scale.md) | NexusWealth | 3 |
| 233 | [Ten Minutes to Exposure](./phase-3/12-nexuswealth/ch233-real-time-portfolio-risk.md) | NexusWealth | 3 |
| 234 | [VIX 80 — The ERISA Clock Is Ticking](./phase-3/12-nexuswealth/ch234-scale-10x-market-volatility-spike.md) | NexusWealth | 3 |
| 235 | [The Trustees Want Answers](./phase-3/12-nexuswealth/ch235-the-board-presentation.md) | NexusWealth | 3 |
| 236 | [One Platform, Three Regulators](./phase-3/12-nexuswealth/ch236-data-sovereignty-pension-funds.md) | NexusWealth | 3 |
| 237 | [A Letter to 2074](./phase-3/12-nexuswealth/ch237-the-50-year-architecture.md) | NexusWealth | 3 |
| 240 | [Inherited Disaster — The Data Quality Crisis](./phase-3/13-propiq/ch240-inherited-disaster-data-quality-crisis.md) | PropIQ | 3 |
| 241 | [The $2M Data Room](./phase-3/13-propiq/ch241-multi-party-data-rooms.md) | PropIQ | 3 |
| 252 | [The Most Sensitive Data in the Room](./phase-3/15-mindscale/ch252-the-most-sensitive-data.md) | MindScale | 3 |
| 254 | [The Consent That Can't Be a Toggle](./phase-3/15-mindscale/ch254-42-cfr-part-2-compliance.md) | MindScale | 3 |
| 261 | [The Quantum API Gateway and Auth (Auth System: 6th Evolution)](./phase-3/16-quantumedge/ch261-quantum-api-gateway-and-auth.md) | QuantumEdge | 3 |

</details>

<a name="tag-conflict-resolution"></a>

<details>
<summary><strong>#conflict-resolution</strong> — 5 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 68 | [Multi-Region Active-Active Architecture](./phase-2/01-stratum-systems/ch68-multi-region-active-active-intro.md) | Stratum Systems | 2 |
| 84 | [Multi-Region Conflict Resolution](./phase-2/04-giggrid/ch84-multi-region-conflict-resolution.md) | GigGrid | 2 |
| 210 | [Three Weeks at Sea](./phase-3/09-deepocean/ch210-offline-first-for-ships.md) | DeepOcean | 3 |
| 211 | [The Napkin Diagram](./phase-3/09-deepocean/ch211-the-global-ocean-data-mesh.md) | DeepOcean | 3 |
| 246 | [The World is Watching (And They're All in London)](./phase-3/14-novasports/ch246-multi-region-active-active.md) | NovaSports | 3 |

</details>

<a name="tag-connection-pooling"></a>

<details>
<summary><strong>#connection-pooling</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 09 | [Scale the EHR Query Layer with Read Replicas](./phase-1/02-meridian-health/ch09-read-replicas-and-query-routing.md) | MeridianHealth | 1 |
| 44 | [Database Collapse During Finals Week — Read Replicas at Scale](./phase-1/09-neurolearn/ch44-read-replicas-and-connection-pooling-second-appearance.md) | NeuroLearn | 1 |
| 141 | [The Vacuum That Wasn't Running](./phase-2/12-prismhealth/ch141-database-internals-postgres-tuning.md) | PrismHealth | 2 |

</details>

<a name="tag-consistency"></a>

<details>
<summary><strong>#consistency</strong> — 9 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 04 | [Design the Merchant Balance Cache](./phase-1/01-novapay/ch04-write-through-cache-design.md) | NovaPay | 1 |
| 36 | [Cache Invalidation Hell](./phase-1/07-pulsecommerce/ch36-cache-invalidation-patterns.md) | PulseCommerce | 1 |
| 37 | [Flash Sale — Inventory Overselling Under Pressure](./phase-1/07-pulsecommerce/ch37-inventory-management-and-consistency.md) | PulseCommerce | 1 |
| 46 | [Design the Adaptive Quiz Engine at Scale](./phase-1/09-neurolearn/ch46-adaptive-quiz-engine-design.md) | NeuroLearn | 1 |
| 49 | [CAP Theorem in Practice — When the Network Fails](./phase-1/10-omnilogix/ch49-cap-theorem-and-eventual-consistency.md) | OmniLogix | 1 |
| 50 | [Design the Multi-Step Supply Chain Transaction with Saga](./phase-1/10-omnilogix/ch50-distributed-transactions-and-saga-pattern.md) | OmniLogix | 1 |
| 63 | [Seat Inventory — The Overbooking Problem](./phase-1/12-skyroute/ch63-seat-inventory-and-overbooking.md) | SkyRoute | 1 |
| 116 | [Ticket Inventory Consistency — The Taylor Swift Problem](./phase-2/08-venueflow/ch116-ticket-inventory-consistency.md) | VenueFlow | 2 |
| 197 | [The Fractured Map](./phase-3/07-automesh/ch197-hd-map-distribution.md) | AutoMesh | 3 |

</details>

<a name="tag-consumer-groups"></a>

<details>
<summary><strong>#consumer-groups</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 11 | [Untangle the Kafka Mess](./phase-1/03-velotrack/ch11-kafka-deep-dive-consumer-groups.md) | VeloTrack | 1 |
| 122 | [Six Hours of Consumer Lag](./phase-2/09-telanova/ch122-advanced-kafka-consumer-lag.md) | TeleNova | 2 |

</details>

<a name="tag-consumer-lag"></a>

<details>
<summary><strong>#consumer-lag</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 79 | [Advanced Kafka — Log Compaction for Order Book State](./phase-2/03-tradespark/ch79-advanced-kafka-log-compaction.md) | TradeSpark | 2 |
| 98 | [Kafka at Scale — 40 Billion Security Events/Day](./phase-2/05-sentinelops/ch98-advanced-kafka-security-events.md) | SentinelOps | 2 |
| 122 | [Six Hours of Consumer Lag](./phase-2/09-telanova/ch122-advanced-kafka-consumer-lag.md) | TeleNova | 2 |

</details>

<a name="tag-contract-testing"></a>

<details>
<summary><strong>#contract-testing</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 130 | [Data Pipeline Schema Evolution](./phase-2/10-buildright/ch130-data-pipeline-schema-evolution.md) | BuildRight | 2 |
| 148 | [300 Internal APIs, Zero Deprecation Policy](./phase-2/13-vertexcloud/ch148-api-governance-at-org-scale.md) | VertexCloud | 2 |

</details>

<a name="tag-cost-engineering"></a>

<details>
<summary><strong>#cost-engineering</strong> — 13 chapters across 10 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 96 | [Advanced Observability — Distributed Tracing at 50M Spans/Day](./phase-2/05-sentinelops/ch96-advanced-observability-distributed-tracing.md) | SentinelOps | 2 |
| 121 | [200 Million Spans Per Day — Tail Sampling and Cost Control](./phase-2/09-telanova/ch121-distributed-tracing-opentelemetry.md) | TeleNova | 2 |
| 135 | [Twelve Petabytes and Counting](./phase-2/11-axiom-labs/ch135-columnar-storage-petabyte-scale.md) | Axiom Labs | 2 |
| 155 | [Capacity at Constellation Scale](./phase-3/01-orbitcore/ch155-capacity-at-constellation-scale.md) | OrbitCore | 3 |
| 167 | [SPIKE — Scale 10x by Friday](./phase-3/03-carbonledger/ch167-scale-10x-spike.md) | CarbonLedger | 3 |
| 169 | [The Cost of Green](./phase-3/03-carbonledger/ch169-the-cost-of-green.md) | CarbonLedger | 3 |
| 196 | [Scale 10x — Fleet Doubles](./phase-3/07-automesh/ch196-scale-10x-fleet-doubles-spike.md) | AutoMesh | 3 |
| 198 | [The $2.3M Conversation](./phase-3/07-automesh/ch198-the-cfo-presentation.md) | AutoMesh | 3 |
| 224 | [Design for 100x, Spend 2x](./phase-3/11-harvestai/ch224-the-seasonal-spike.md) | HarvestAI | 3 |
| 235 | [The Trustees Want Answers](./phase-3/12-nexuswealth/ch235-the-board-presentation.md) | NexusWealth | 3 |
| 251 | [Before Next Season](./phase-3/14-novasports/ch251-streaming-architecture-review.md) | NovaSports | 3 |
| 263 | [Error Mitigation at Scale](./phase-3/16-quantumedge/ch263-error-mitigation-at-scale.md) | QuantumEdge | 3 |
| 265 | [Quantum Platform Economics](./phase-3/16-quantumedge/ch265-quantum-platform-economics.md) | QuantumEdge | 3 |

</details>

<a name="tag-cost-optimization"></a>

<details>
<summary><strong>#cost-optimization</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 18 | [Fix the CDN — $2M/Month and Falling](./phase-1/04-beacon-media/ch18-blob-storage-and-cdn-architecture.md) | Beacon Media | 1 |
| 21 | [Scale 10x — The New Content Deal is Going to Break Transcoding](./phase-1/04-beacon-media/ch21-video-transcoding-pipeline.md) | Beacon Media | 1 |

</details>

<a name="tag-cqrs"></a>

<details>
<summary><strong>#cqrs</strong> — 6 chapters across 6 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 66 | [CQRS and Event Sourcing](./phase-2/01-stratum-systems/ch66-cqrs-and-event-sourcing.md) | Stratum Systems | 2 |
| 77 | [Low-Latency Event Sourcing for Trade Execution](./phase-2/03-tradespark/ch77-low-latency-event-sourcing.md) | TradeSpark | 2 |
| 104 | [CQRS Advanced Projections — Real-Time Inventory](./phase-2/06-lightspeedretail/ch104-cqrs-advanced-projections.md) | LightspeedRetail | 2 |
| 112 | ["What Was the Grid State at 14:32:07 on March 3rd?"](./phase-2/07-crestline-energy/ch112-cqrs-temporal-queries.md) | Crestline Energy | 2 |
| 126 | [CQRS and Advanced Event Sourcing for Project State](./phase-2/10-buildright/ch126-cqrs-event-sourcing-advanced.md) | BuildRight | 2 |
| 142 | [A Patient's Complete Vital History](./phase-2/12-prismhealth/ch142-cqrs-for-patient-timelines.md) | PrismHealth | 2 |

</details>

<a name="tag-crdt"></a>

<details>
<summary><strong>#crdt</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 85 | [CRDTs — Conflict-Free Replicated Data Types](./phase-2/04-giggrid/ch85-crdts-introduction.md) | GigGrid | 2 |
| 116 | [Ticket Inventory Consistency — The Taylor Swift Problem](./phase-2/08-venueflow/ch116-ticket-inventory-consistency.md) | VenueFlow | 2 |
| 125 | [Five Country Launches in One Quarter](./phase-2/09-telanova/ch125-multi-region-active-active-advanced.md) | TeleNova | 2 |

</details>

<a name="tag-crisis-detection"></a>

<details>
<summary><strong>#crisis-detection</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 253 | [The Precision-Recall Dilemma](./phase-3/15-mindscale/ch253-crisis-detection-ml.md) | MindScale | 3 |
| 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | MindScale | 3 |

</details>

<a name="tag-cross-border"></a>

<details>
<summary><strong>#cross-border</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 111 | [Energy Data Sovereignty — When Regulators Disagree](./phase-2/07-crestline-energy/ch111-multi-region-data-residency-advanced.md) | Crestline Energy | 2 |
| 143 | [A German Patient's Heartbeat Cannot Leave Germany](./phase-2/12-prismhealth/ch143-multi-region-data-sovereignty-healthcare.md) | PrismHealth | 2 |

</details>

<a name="tag-cryptographic-chaining"></a>

<details>
<summary><strong>#cryptographic-chaining</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 72 | [Advanced Audit Trails at Scale](./phase-2/02-nexacare/ch72-advanced-audit-trails-at-scale.md) | NexaCare | 2 |
| 162 | [Audit Everything](./phase-3/02-ironwatch/ch162-audit-everything.md) | IronWatch | 3 |

</details>

<a name="tag-cryptographic-erasure"></a>

<details>
<summary><strong>#cryptographic-erasure</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 70 | [GDPR Deletion Pipelines](./phase-2/02-nexacare/ch70-gdpr-deletion-pipelines.md) | NexaCare | 2 |
| 131 | [GDPR Erasure in Event-Sourced Systems](./phase-2/10-buildright/ch131-compliance-gdpr-erasure-and-event-logs.md) | BuildRight | 2 |

</details>

<a name="tag-data-governance"></a>

<details>
<summary><strong>#data-governance</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 180 | [Brain Data Privacy](./phase-3/05-neuralbridge/ch180-brain-data-privacy.md) | NeuralBridge | 3 |
| 252 | [The Most Sensitive Data in the Room](./phase-3/15-mindscale/ch252-the-most-sensitive-data.md) | MindScale | 3 |

</details>

<a name="tag-data-ingestion"></a>

<details>
<summary><strong>#data-ingestion</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 209 | [The Ocean's Heartbeat](./phase-3/09-deepocean/ch209-ais-the-oceans-heartbeat.md) | DeepOcean | 3 |
| 242 | [The Real-Time Lie](./phase-3/13-propiq/ch242-real-time-market-aggregation.md) | PropIQ | 3 |
| 268 | [The Numbers Are Wrong](./phase-3/17-glaciergrid/ch268-distributed-solar-wind-aggregation.md) | GlacierGrid | 3 |

</details>

<a name="tag-data-integrity"></a>

<details>
<summary><strong>#data-integrity</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 183 | [The Clinical Data Platform](./phase-3/05-neuralbridge/ch183-the-clinical-data-platform.md) | NeuralBridge | 3 |
| 222 | [The Retroactive Proof](./phase-3/10-pharmasync/ch222-the-validation-protocol.md) | PharmaSync | 3 |

</details>

<a name="tag-data-isolation"></a>

<details>
<summary><strong>#data-isolation</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 171 | [Privileged Information](./phase-3/04-lexcore/ch171-privileged-information.md) | LexCore | 3 |
| 227 | [One Account, Eight Hundred Farms](./phase-3/11-harvestai/ch227-multi-tenant-farm-management.md) | HarvestAI | 3 |

</details>

<a name="tag-data-mesh"></a>

<details>
<summary><strong>#data-mesh</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 136 | [200 Universities, 200 Data Silos](./phase-2/11-axiom-labs/ch136-data-mesh-intro.md) | Axiom Labs | 2 |
| 208 | [The Twelve Factories](./phase-3/08-forgesense/ch208-factory-data-mesh.md) | ForgeSense | 3 |
| 211 | [The Napkin Diagram](./phase-3/09-deepocean/ch211-the-global-ocean-data-mesh.md) | DeepOcean | 3 |
| 238 | [Twelve Sources, One Truth](./phase-3/13-propiq/ch238-the-property-data-mesh.md) | PropIQ | 3 |

</details>

<a name="tag-data-minimization"></a>

<details>
<summary><strong>#data-minimization</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 133 | [Genomic Data Is Personal — Differential Privacy and Federated Computation](./phase-2/11-axiom-labs/ch133-privacy-preserving-systems.md) | Axiom Labs | 2 |
| 256 | [The 23 Percent](./phase-3/15-mindscale/ch256-re-identification-risk-mitigation.md) | MindScale | 3 |
| 257 | [Privacy by Forgetting](./phase-3/15-mindscale/ch257-the-data-minimization-architecture.md) | MindScale | 3 |

</details>

<a name="tag-data-pipeline"></a>

<details>
<summary><strong>#data-pipeline</strong> — 16 chapters across 11 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 21 | [Scale 10x — The New Content Deal is Going to Break Transcoding](./phase-1/04-beacon-media/ch21-video-transcoding-pipeline.md) | Beacon Media | 1 |
| 23 | [Design the Time-Series Sensor Data Platform](./phase-1/05-agrosense/ch23-time-series-data-design.md) | AgroSense | 1 |
| 24 | [Build the ETL Pipeline and Change Data Capture](./phase-1/05-agrosense/ch24-etl-pipeline-and-cdc.md) | AgroSense | 1 |
| 54 | [The Feature Store — Feeding Models the Right Data](./phase-1/11-luminaryai/ch54-feature-store-and-ml-pipeline.md) | LuminaryAI | 1 |
| 55 | [ETL to Streaming — Evolving the Data Pipeline](./phase-1/11-luminaryai/ch55-etl-to-streaming-pipeline-evolution.md) | LuminaryAI | 1 |
| 130 | [Data Pipeline Schema Evolution](./phase-2/10-buildright/ch130-data-pipeline-schema-evolution.md) | BuildRight | 2 |
| 150 | [Signal and Noise](./phase-3/01-orbitcore/ch150-signal-and-noise.md) | OrbitCore | 3 |
| 170 | [The Permanence Problem](./phase-3/03-carbonledger/ch170-the-permanence-problem.md) | CarbonLedger | 3 |
| 187 | [The Actuarial Pipeline](./phase-3/06-shieldmutual/ch187-the-actuarial-pipeline.md) | ShieldMutual | 3 |
| 203 | [OPC-UA to Cloud](./phase-3/08-forgesense/ch203-opc-ua-to-cloud.md) | ForgeSense | 3 |
| 217 | [Lab Data Integration](./phase-3/10-pharmasync/ch217-lab-data-integration.md) | PharmaSync | 3 |
| 218 | [Feature Stores for Drug Discovery](./phase-3/10-pharmasync/ch218-feature-stores-drug-discovery.md) | PharmaSync | 3 |
| 219 | [The Six-Week Submission](./phase-3/10-pharmasync/ch219-clinical-data-management.md) | PharmaSync | 3 |
| 221 | [The Two Phantom Patients](./phase-3/10-pharmasync/ch221-production-down-clinical-trial.md) | PharmaSync | 3 |
| 231 | [ERISA and SEC Compliance](./phase-3/12-nexuswealth/ch231-erisa-and-sec-compliance.md) | NexusWealth | 3 |
| 264 | [Hybrid Pipelines — Chemistry Simulation](./phase-3/16-quantumedge/ch264-hybrid-pipelines-chemistry-simulation.md) | QuantumEdge | 3 |

</details>

<a name="tag-data-products"></a>

<details>
<summary><strong>#data-products</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 136 | [200 Universities, 200 Data Silos](./phase-2/11-axiom-labs/ch136-data-mesh-intro.md) | Axiom Labs | 2 |
| 208 | [The Twelve Factories](./phase-3/08-forgesense/ch208-factory-data-mesh.md) | ForgeSense | 3 |

</details>

<a name="tag-data-quality"></a>

<details>
<summary><strong>#data-quality</strong> — 6 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 26 | [Design for Unreliable Sensors at the Edge](./phase-1/05-agrosense/ch26-edge-processing-and-sensor-reliability.md) | AgroSense | 1 |
| 57 | [Advanced Observability — When Models Lie Silently](./phase-1/11-luminaryai/ch57-advanced-observability-ml-systems.md) | LuminaryAI | 1 |
| 204 | [The Webb Keynote](./phase-3/08-forgesense/ch204-the-webb-keynote.md) | ForgeSense | 3 |
| 211 | [The Napkin Diagram](./phase-3/09-deepocean/ch211-the-global-ocean-data-mesh.md) | DeepOcean | 3 |
| 238 | [Twelve Sources, One Truth](./phase-3/13-propiq/ch238-the-property-data-mesh.md) | PropIQ | 3 |
| 240 | [Inherited Disaster — The Data Quality Crisis](./phase-3/13-propiq/ch240-inherited-disaster-data-quality-crisis.md) | PropIQ | 3 |

</details>

<a name="tag-data-residency"></a>

<details>
<summary><strong>#data-residency</strong> — 8 chapters across 8 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 08 | [Data Residency — The State That Wants Its Data Back](./phase-1/02-meridian-health/ch08-data-residency-and-hipaa-architecture.md) | MeridianHealth | 1 |
| 68 | [Multi-Region Active-Active Architecture](./phase-2/01-stratum-systems/ch68-multi-region-active-active-intro.md) | Stratum Systems | 2 |
| 73 | [Data Residency Across Multiple Jurisdictions](./phase-2/02-nexacare/ch73-data-residency-multi-jurisdiction.md) | NexaCare | 2 |
| 111 | [Energy Data Sovereignty — When Regulators Disagree](./phase-2/07-crestline-energy/ch111-multi-region-data-residency-advanced.md) | Crestline Energy | 2 |
| 152 | [Sovereignty in Orbit](./phase-3/01-orbitcore/ch152-sovereignty-in-orbit.md) | OrbitCore | 3 |
| 166 | [Article 6 Compliance](./phase-3/03-carbonledger/ch166-article-6-compliance.md) | CarbonLedger | 3 |
| 173 | [Where Are the Embeddings?](./phase-3/04-lexcore/ch173-design-review-ambush-spike.md) | LexCore | 3 |
| 246 | [The World is Watching (And They're All in London)](./phase-3/14-novasports/ch246-multi-region-active-active.md) | NovaSports | 3 |

</details>

<a name="tag-data-retention"></a>

<details>
<summary><strong>#data-retention</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 131 | [GDPR Erasure in Event-Sourced Systems](./phase-2/10-buildright/ch131-compliance-gdpr-erasure-and-event-logs.md) | BuildRight | 2 |
| 230 | [Fifty Years of Data](./phase-3/12-nexuswealth/ch230-fifty-years-of-data.md) | NexusWealth | 3 |
| 237 | [A Letter to 2074](./phase-3/12-nexuswealth/ch237-the-50-year-architecture.md) | NexusWealth | 3 |

</details>

<a name="tag-data-rooms"></a>

<details>
<summary><strong>#data-rooms</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 174 | [The RFC That Failed](./phase-3/04-lexcore/ch174-the-rfc-that-failed.md) | LexCore | 3 |
| 241 | [The $2M Data Room](./phase-3/13-propiq/ch241-multi-party-data-rooms.md) | PropIQ | 3 |

</details>

<a name="tag-data-sovereignty"></a>

<details>
<summary><strong>#data-sovereignty</strong> — 8 chapters across 6 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 39 | [Design the Data Sovereignty Architecture for Government Clients](./phase-1/08-civicos/ch39-data-sovereignty-and-compliance-architecture.md) | CivicOS | 1 |
| 73 | [Data Residency Across Multiple Jurisdictions](./phase-2/02-nexacare/ch73-data-residency-multi-jurisdiction.md) | NexaCare | 2 |
| 111 | [Energy Data Sovereignty — When Regulators Disagree](./phase-2/07-crestline-energy/ch111-multi-region-data-residency-advanced.md) | Crestline Energy | 2 |
| 143 | [A German Patient's Heartbeat Cannot Leave Germany](./phase-2/12-prismhealth/ch143-multi-region-data-sovereignty-healthcare.md) | PrismHealth | 2 |
| 150 | [Signal and Noise](./phase-3/01-orbitcore/ch150-signal-and-noise.md) | OrbitCore | 3 |
| 152 | [Sovereignty in Orbit](./phase-3/01-orbitcore/ch152-sovereignty-in-orbit.md) | OrbitCore | 3 |
| 156 | [SPIKE — Solar Flare](./phase-3/01-orbitcore/ch156-solar-flare-spike.md) | OrbitCore | 3 |
| 236 | [One Platform, Three Regulators](./phase-3/12-nexuswealth/ch236-data-sovereignty-pension-funds.md) | NexusWealth | 3 |

</details>

<a name="tag-data-warehousing"></a>

<details>
<summary><strong>#data-warehousing</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 103 | [Data Warehousing — OLAP vs OLTP](./phase-2/06-lightspeedretail/ch103-data-warehousing-olap-vs-oltp.md) | LightspeedRetail | 2 |
| 107 | [Grid Analytics — Apache Iceberg and Columnar Storage](./phase-2/07-crestline-energy/ch107-data-warehousing-columnar-storage.md) | Crestline Energy | 2 |

</details>

<a name="tag-database"></a>

<details>
<summary><strong>#database</strong> — 31 chapters across 11 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 01 | [Design the Payment Processing Pipeline](./phase-1/01-novapay/ch01-payment-processing-pipeline.md) | NovaPay | 1 |
| 02 | [Production is DOWN — Duplicate Charges at Scale](./phase-1/01-novapay/ch02-idempotency-and-duplicate-prevention.md) | NovaPay | 1 |
| 03 | [The Slow Query Nobody Wrote a Ticket For](./phase-1/01-novapay/ch03-database-indexing-for-financial-queries.md) | NovaPay | 1 |
| 04 | [Design the Merchant Balance Cache](./phase-1/01-novapay/ch04-write-through-cache-design.md) | NovaPay | 1 |
| 07 | [Build the HIPAA-Compliant Audit Trail](./phase-1/02-meridian-health/ch07-hipaa-audit-trails.md) | MeridianHealth | 1 |
| 08 | [Data Residency — The State That Wants Its Data Back](./phase-1/02-meridian-health/ch08-data-residency-and-hipaa-architecture.md) | MeridianHealth | 1 |
| 09 | [Scale the EHR Query Layer with Read Replicas](./phase-1/02-meridian-health/ch09-read-replicas-and-query-routing.md) | MeridianHealth | 1 |
| 10 | [Scale 10x by Friday — The Northview Surge](./phase-1/02-meridian-health/ch10-ehr-integration-platform-design.md) | MeridianHealth | 1 |
| 15 | [The Delivery State Machine at Scale](./phase-1/03-velotrack/ch15-kafka-exactly-once-and-delivery-state-machine.md) | VeloTrack | 1 |
| 23 | [Design the Time-Series Sensor Data Platform](./phase-1/05-agrosense/ch23-time-series-data-design.md) | AgroSense | 1 |
| 24 | [Build the ETL Pipeline and Change Data Capture](./phase-1/05-agrosense/ch24-etl-pipeline-and-cdc.md) | AgroSense | 1 |
| 28 | [Multi-Tenant Architecture and Tenant Isolation](./phase-1/06-cloudstack/ch28-multi-tenancy-architecture.md) | CloudStack | 1 |
| 31 | [Design the Tenant Billing and Usage Metering System](./phase-1/06-cloudstack/ch31-tenant-billing-and-usage-metering.md) | CloudStack | 1 |
| 35 | [The N+1 Query Disaster Eating Your Database](./phase-1/07-pulsecommerce/ch35-n-plus-one-queries-and-dataloader.md) | PulseCommerce | 1 |
| 37 | [Flash Sale — Inventory Overselling Under Pressure](./phase-1/07-pulsecommerce/ch37-inventory-management-and-consistency.md) | PulseCommerce | 1 |
| 44 | [Database Collapse During Finals Week — Read Replicas at Scale](./phase-1/09-neurolearn/ch44-read-replicas-and-connection-pooling-second-appearance.md) | NeuroLearn | 1 |
| 46 | [Design the Adaptive Quiz Engine at Scale](./phase-1/09-neurolearn/ch46-adaptive-quiz-engine-design.md) | NeuroLearn | 1 |
| 47 | [Design Review Ambush — The Academic Integrity Architecture](./phase-1/09-neurolearn/ch47-academic-integrity-monitoring.md) | NeuroLearn | 1 |
| 95 | [Secret Management at Scale](./phase-2/05-sentinelops/ch95-secret-management-at-scale.md) | SentinelOps | 2 |
| 165 | [The Ledger That Cannot Lie](./phase-3/03-carbonledger/ch165-the-ledger-that-cannot-lie.md) | CarbonLedger | 3 |
| 167 | [SPIKE — Scale 10x by Friday](./phase-3/03-carbonledger/ch167-scale-10x-spike.md) | CarbonLedger | 3 |
| 168 | [Event Sourcing for Regulators](./phase-3/03-carbonledger/ch168-event-sourcing-for-regulators.md) | CarbonLedger | 3 |
| 169 | [The Cost of Green](./phase-3/03-carbonledger/ch169-the-cost-of-green.md) | CarbonLedger | 3 |
| 170 | [The Permanence Problem](./phase-3/03-carbonledger/ch170-the-permanence-problem.md) | CarbonLedger | 3 |
| 216 | [21 CFR Part 11](./phase-3/10-pharmasync/ch216-21-cfr-part-11.md) | PharmaSync | 3 |
| 217 | [Lab Data Integration](./phase-3/10-pharmasync/ch217-lab-data-integration.md) | PharmaSync | 3 |
| 218 | [Feature Stores for Drug Discovery](./phase-3/10-pharmasync/ch218-feature-stores-drug-discovery.md) | PharmaSync | 3 |
| 219 | [The Six-Week Submission](./phase-3/10-pharmasync/ch219-clinical-data-management.md) | PharmaSync | 3 |
| 220 | [Gerald Must Die](./phase-3/10-pharmasync/ch220-the-lims-deprecation.md) | PharmaSync | 3 |
| 221 | [The Two Phantom Patients](./phase-3/10-pharmasync/ch221-production-down-clinical-trial.md) | PharmaSync | 3 |
| 230 | [Fifty Years of Data](./phase-3/12-nexuswealth/ch230-fifty-years-of-data.md) | NexusWealth | 3 |

</details>

<a name="tag-database-internals"></a>

<details>
<summary><strong>#database-internals</strong> — 5 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 67 | [Database Internals — WAL and MVCC](./phase-2/01-stratum-systems/ch67-database-internals-wal-and-mvcc.md) | Stratum Systems | 2 |
| 71 | [Database Internals — B-Tree vs LSM-Tree](./phase-2/02-nexacare/ch71-database-internals-btree-vs-lsm.md) | NexaCare | 2 |
| 78 | [Database Internals — LSM-Tree Compaction Storm](./phase-2/03-tradespark/ch78-database-internals-deep-dive.md) | TradeSpark | 2 |
| 106 | [Time-Series at Scale — 50M Meters, 200M Events Per Hour](./phase-2/07-crestline-energy/ch106-time-series-at-scale-advanced.md) | Crestline Energy | 2 |
| 141 | [The Vacuum That Wasn't Running](./phase-2/12-prismhealth/ch141-database-internals-postgres-tuning.md) | PrismHealth | 2 |

</details>

<a name="tag-deduplication"></a>

<details>
<summary><strong>#deduplication</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 51 | [Idempotency at Scale — When Retries Become Dangerous](./phase-1/10-omnilogix/ch51-idempotency-at-scale.md) | OmniLogix | 1 |
| 138 | [14 Million Device Readings Per Minute](./phase-2/12-prismhealth/ch138-iot-at-scale-advanced.md) | PrismHealth | 2 |

</details>

<a name="tag-deployment"></a>

<details>
<summary><strong>#deployment</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 88 | [Canary Deployments and Feature Flags](./phase-2/04-giggrid/ch88-canary-deployments-and-feature-flags.md) | GigGrid | 2 |
| 147 | [GitOps at 2,000 Engineers](./phase-2/13-vertexcloud/ch147-advanced-deployment-pipeline.md) | VertexCloud | 2 |

</details>

<a name="tag-design-review"></a>

<details>
<summary><strong>#design-review</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 163 | [The Exit Brief](./phase-3/02-ironwatch/ch163-the-exit-brief.md) | IronWatch | 3 |
| 173 | [Where Are the Embeddings?](./phase-3/04-lexcore/ch173-design-review-ambush-spike.md) | LexCore | 3 |
| 206 | [What Do You Mean, Read-Only?](./phase-3/08-forgesense/ch206-design-review-ambush-brownfield.md) | ForgeSense | 3 |

</details>

<a name="tag-differential-privacy"></a>

<details>
<summary><strong>#differential-privacy</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 133 | [Genomic Data Is Personal — Differential Privacy and Federated Computation](./phase-2/11-axiom-labs/ch133-privacy-preserving-systems.md) | Axiom Labs | 2 |
| 256 | [The 23 Percent](./phase-3/15-mindscale/ch256-re-identification-risk-mitigation.md) | MindScale | 3 |

</details>

<a name="tag-dispatch"></a>

<details>
<summary><strong>#dispatch</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 267 | [The 500ms Problem](./phase-3/17-glaciergrid/ch267-grid-edge-intelligence.md) | GlacierGrid | 3 |
| 270 | [The Demand Cliff](./phase-3/17-glaciergrid/ch270-real-time-grid-stability.md) | GlacierGrid | 3 |

</details>

<a name="tag-distributed"></a>

<details>
<summary><strong>#distributed</strong> — 69 chapters across 24 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 01 | [Design the Payment Processing Pipeline](./phase-1/01-novapay/ch01-payment-processing-pipeline.md) | NovaPay | 1 |
| 02 | [Production is DOWN — Duplicate Charges at Scale](./phase-1/01-novapay/ch02-idempotency-and-duplicate-prevention.md) | NovaPay | 1 |
| 08 | [Data Residency — The State That Wants Its Data Back](./phase-1/02-meridian-health/ch08-data-residency-and-hipaa-architecture.md) | MeridianHealth | 1 |
| 10 | [Scale 10x by Friday — The Northview Surge](./phase-1/02-meridian-health/ch10-ehr-integration-platform-design.md) | MeridianHealth | 1 |
| 11 | [Untangle the Kafka Mess](./phase-1/03-velotrack/ch11-kafka-deep-dive-consumer-groups.md) | VeloTrack | 1 |
| 12 | [Production is DOWN — The Notification Consumer Won't Stop Crashing](./phase-1/03-velotrack/ch12-dead-letter-queues-and-error-handling.md) | VeloTrack | 1 |
| 13 | [Design the Real-Time Driver Tracking System](./phase-1/03-velotrack/ch13-real-time-delivery-tracking.md) | VeloTrack | 1 |
| 14 | [Design the Load Balancing Strategy for a Heterogeneous Fleet](./phase-1/03-velotrack/ch14-load-balancing-strategies.md) | VeloTrack | 1 |
| 15 | [The Delivery State Machine at Scale](./phase-1/03-velotrack/ch15-kafka-exactly-once-and-delivery-state-machine.md) | VeloTrack | 1 |
| 18 | [Fix the CDN — $2M/Month and Falling](./phase-1/04-beacon-media/ch18-blob-storage-and-cdn-architecture.md) | Beacon Media | 1 |
| 21 | [Scale 10x — The New Content Deal is Going to Break Transcoding](./phase-1/04-beacon-media/ch21-video-transcoding-pipeline.md) | Beacon Media | 1 |
| 26 | [Design for Unreliable Sensors at the Edge](./phase-1/05-agrosense/ch26-edge-processing-and-sensor-reliability.md) | AgroSense | 1 |
| 28 | [Multi-Tenant Architecture and Tenant Isolation](./phase-1/06-cloudstack/ch28-multi-tenancy-architecture.md) | CloudStack | 1 |
| 29 | [Design the API Gateway and Rate Limiting System (First Appearance)](./phase-1/06-cloudstack/ch29-api-gateway-and-rate-limiting.md) | CloudStack | 1 |
| 31 | [Design the Tenant Billing and Usage Metering System](./phase-1/06-cloudstack/ch31-tenant-billing-and-usage-metering.md) | CloudStack | 1 |
| 32 | [Solve the Noisy Neighbor Problem for Good](./phase-1/06-cloudstack/ch32-noisy-neighbor-and-resource-quotas.md) | CloudStack | 1 |
| 33 | [Design the Product Search System (First Appearance)](./phase-1/07-pulsecommerce/ch33-search-inverted-index-elasticsearch.md) | PulseCommerce | 1 |
| 34 | [Payment Integration at Scale — Scaling Challenges (Second Appearance)](./phase-1/07-pulsecommerce/ch34-payment-processing-second-appearance.md) | PulseCommerce | 1 |
| 36 | [Cache Invalidation Hell](./phase-1/07-pulsecommerce/ch36-cache-invalidation-patterns.md) | PulseCommerce | 1 |
| 37 | [Flash Sale — Inventory Overselling Under Pressure](./phase-1/07-pulsecommerce/ch37-inventory-management-and-consistency.md) | PulseCommerce | 1 |
| 40 | [Rate Limiting for Government APIs — The Enumeration Attack](./phase-1/08-civicos/ch40-rate-limiting-second-appearance.md) | CivicOS | 1 |
| 42 | [Design for Accessibility, Resilience, and the Citizen Who Can't Retry](./phase-1/08-civicos/ch42-accessibility-and-resilience-for-government-systems.md) | CivicOS | 1 |
| 43 | [Notification System at Scale — The Finals Week Problem (Second Appearance)](./phase-1/09-neurolearn/ch43-notification-system-second-appearance.md) | NeuroLearn | 1 |
| 45 | [Search Over Academic Content — Scaling Challenges (Second Appearance)](./phase-1/09-neurolearn/ch45-search-second-appearance.md) | NeuroLearn | 1 |
| 46 | [Design the Adaptive Quiz Engine at Scale](./phase-1/09-neurolearn/ch46-adaptive-quiz-engine-design.md) | NeuroLearn | 1 |
| 49 | [CAP Theorem in Practice — When the Network Fails](./phase-1/10-omnilogix/ch49-cap-theorem-and-eventual-consistency.md) | OmniLogix | 1 |
| 50 | [Design the Multi-Step Supply Chain Transaction with Saga](./phase-1/10-omnilogix/ch50-distributed-transactions-and-saga-pattern.md) | OmniLogix | 1 |
| 51 | [Idempotency at Scale — When Retries Become Dangerous](./phase-1/10-omnilogix/ch51-idempotency-at-scale.md) | OmniLogix | 1 |
| 52 | [Kafka Exactly-Once at the Application Level](./phase-1/10-omnilogix/ch52-kafka-exactly-once-application-level.md) | OmniLogix | 1 |
| 53 | [Global Inventory Consistency — The Full Design](./phase-1/10-omnilogix/ch53-global-inventory-consistency-synthesis.md) | OmniLogix | 1 |
| 62 | [Rate Limiting — Defending Against the GDS Giants](./phase-1/12-skyroute/ch62-rate-limiting-third-appearance.md) | SkyRoute | 1 |
| 63 | [Seat Inventory — The Overbooking Problem](./phase-1/12-skyroute/ch63-seat-inventory-and-overbooking.md) | SkyRoute | 1 |
| 87 | [Platform API Rate Limiting (4th Appearance)](./phase-2/04-giggrid/ch87-rate-limiting-fourth-appearance.md) | GigGrid | 2 |
| 113 | [Real-Time at Scale — 500k Concurrent Fans](./phase-2/08-venueflow/ch113-real-time-at-scale-pub-sub.md) | VenueFlow | 2 |
| 116 | [Ticket Inventory Consistency — The Taylor Swift Problem](./phase-2/08-venueflow/ch116-ticket-inventory-consistency.md) | VenueFlow | 2 |
| 126 | [CQRS and Advanced Event Sourcing for Project State](./phase-2/10-buildright/ch126-cqrs-event-sourcing-advanced.md) | BuildRight | 2 |
| 150 | [Signal and Noise](./phase-3/01-orbitcore/ch150-signal-and-noise.md) | OrbitCore | 3 |
| 151 | [The Cell Problem](./phase-3/01-orbitcore/ch151-the-cell-problem.md) | OrbitCore | 3 |
| 152 | [Sovereignty in Orbit](./phase-3/01-orbitcore/ch152-sovereignty-in-orbit.md) | OrbitCore | 3 |
| 153 | [The Latency Ladder](./phase-3/01-orbitcore/ch153-the-latency-ladder.md) | OrbitCore | 3 |
| 154 | [Ground Truth](./phase-3/01-orbitcore/ch154-ground-truth.md) | OrbitCore | 3 |
| 155 | [Capacity at Constellation Scale](./phase-3/01-orbitcore/ch155-capacity-at-constellation-scale.md) | OrbitCore | 3 |
| 165 | [The Ledger That Cannot Lie](./phase-3/03-carbonledger/ch165-the-ledger-that-cannot-lie.md) | CarbonLedger | 3 |
| 166 | [Article 6 Compliance](./phase-3/03-carbonledger/ch166-article-6-compliance.md) | CarbonLedger | 3 |
| 167 | [SPIKE — Scale 10x by Friday](./phase-3/03-carbonledger/ch167-scale-10x-spike.md) | CarbonLedger | 3 |
| 168 | [Event Sourcing for Regulators](./phase-3/03-carbonledger/ch168-event-sourcing-for-regulators.md) | CarbonLedger | 3 |
| 169 | [The Cost of Green](./phase-3/03-carbonledger/ch169-the-cost-of-green.md) | CarbonLedger | 3 |
| 170 | [The Permanence Problem](./phase-3/03-carbonledger/ch170-the-permanence-problem.md) | CarbonLedger | 3 |
| 178 | [The Signal](./phase-3/05-neuralbridge/ch178-the-signal.md) | NeuralBridge | 3 |
| 181 | [The Systemic Failure](./phase-3/05-neuralbridge/ch181-the-systemic-failure.md) | NeuralBridge | 3 |
| 188 | [Claims at Scale](./phase-3/06-shieldmutual/ch188-claims-at-scale.md) | ShieldMutual | 3 |
| 193 | [V2X at Scale](./phase-3/07-automesh/ch193-v2x-at-scale.md) | AutoMesh | 3 |
| 194 | [The Safety-Critical Edge](./phase-3/07-automesh/ch194-the-safety-critical-edge.md) | AutoMesh | 3 |
| 195 | [OTA at Fleet Scale](./phase-3/07-automesh/ch195-ota-at-fleet-scale.md) | AutoMesh | 3 |
| 200 | [Five Nines for a Moving Vehicle](./phase-3/07-automesh/ch200-the-partnership-aftermath.md) | AutoMesh | 3 |
| 202 | [The Digital Twin](./phase-3/08-forgesense/ch202-the-digital-twin.md) | ForgeSense | 3 |
| 216 | [21 CFR Part 11](./phase-3/10-pharmasync/ch216-21-cfr-part-11.md) | PharmaSync | 3 |
| 218 | [Feature Stores for Drug Discovery](./phase-3/10-pharmasync/ch218-feature-stores-drug-discovery.md) | PharmaSync | 3 |
| 219 | [The Six-Week Submission](./phase-3/10-pharmasync/ch219-clinical-data-management.md) | PharmaSync | 3 |
| 220 | [Gerald Must Die](./phase-3/10-pharmasync/ch220-the-lims-deprecation.md) | PharmaSync | 3 |
| 221 | [The Two Phantom Patients](./phase-3/10-pharmasync/ch221-production-down-clinical-trial.md) | PharmaSync | 3 |
| 245 | [50 Million Events Per Second](./phase-3/14-novasports/ch245-50-million-events-per-second.md) | NovaSports | 3 |
| 246 | [The World is Watching (And They're All in London)](./phase-3/14-novasports/ch246-multi-region-active-active.md) | NovaSports | 3 |
| 260 | [The Hybrid Architecture](./phase-3/16-quantumedge/ch260-the-hybrid-architecture.md) | QuantumEdge | 3 |
| 262 | [Circuit Job Orchestration](./phase-3/16-quantumedge/ch262-circuit-job-orchestration.md) | QuantumEdge | 3 |
| 263 | [Error Mitigation at Scale](./phase-3/16-quantumedge/ch263-error-mitigation-at-scale.md) | QuantumEdge | 3 |
| 264 | [Hybrid Pipelines — Chemistry Simulation](./phase-3/16-quantumedge/ch264-hybrid-pipelines-chemistry-simulation.md) | QuantumEdge | 3 |
| 265 | [Quantum Platform Economics](./phase-3/16-quantumedge/ch265-quantum-platform-economics.md) | QuantumEdge | 3 |
| 267 | [The 500ms Problem](./phase-3/17-glaciergrid/ch267-grid-edge-intelligence.md) | GlacierGrid | 3 |

</details>

<a name="tag-distributed-computing"></a>

<details>
<summary><strong>#distributed-computing</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 232 | [The March 31st Problem](./phase-3/12-nexuswealth/ch232-actuarial-calculation-at-scale.md) | NexusWealth | 3 |
| 233 | [Ten Minutes to Exposure](./phase-3/12-nexuswealth/ch233-real-time-portfolio-risk.md) | NexusWealth | 3 |

</details>

<a name="tag-distributed-systems"></a>

<details>
<summary><strong>#distributed-systems</strong> — 12 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 64 | [Distributed Consensus and Raft](./phase-2/01-stratum-systems/ch64-distributed-consensus-and-raft.md) | Stratum Systems | 2 |
| 65 | [Event Ordering and Lamport Timestamps](./phase-2/01-stratum-systems/ch65-event-ordering-and-lamport-timestamps.md) | Stratum Systems | 2 |
| 66 | [CQRS and Event Sourcing](./phase-2/01-stratum-systems/ch66-cqrs-and-event-sourcing.md) | Stratum Systems | 2 |
| 68 | [Multi-Region Active-Active Architecture](./phase-2/01-stratum-systems/ch68-multi-region-active-active-intro.md) | Stratum Systems | 2 |
| 84 | [Multi-Region Conflict Resolution](./phase-2/04-giggrid/ch84-multi-region-conflict-resolution.md) | GigGrid | 2 |
| 85 | [CRDTs — Conflict-Free Replicated Data Types](./phase-2/04-giggrid/ch85-crdts-introduction.md) | GigGrid | 2 |
| 93 | [Service Mesh Deep Dive — Istio in Production](./phase-2/05-sentinelops/ch93-service-mesh-deep-dive.md) | SentinelOps | 2 |
| 95 | [Secret Management at Scale](./phase-2/05-sentinelops/ch95-secret-management-at-scale.md) | SentinelOps | 2 |
| 98 | [Kafka at Scale — 40 Billion Security Events/Day](./phase-2/05-sentinelops/ch98-advanced-kafka-security-events.md) | SentinelOps | 2 |
| 234 | [VIX 80 — The ERISA Clock Is Ticking](./phase-3/12-nexuswealth/ch234-scale-10x-market-volatility-spike.md) | NexusWealth | 3 |
| 236 | [One Platform, Three Regulators](./phase-3/12-nexuswealth/ch236-data-sovereignty-pension-funds.md) | NexusWealth | 3 |
| 238 | [Twelve Sources, One Truth](./phase-3/13-propiq/ch238-the-property-data-mesh.md) | PropIQ | 3 |

</details>

<a name="tag-distributed-tracing"></a>

<details>
<summary><strong>#distributed-tracing</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 69 | [Distributed Tracing with OpenTelemetry](./phase-2/01-stratum-systems/ch69-distributed-tracing-intro.md) | Stratum Systems | 2 |
| 96 | [Advanced Observability — Distributed Tracing at 50M Spans/Day](./phase-2/05-sentinelops/ch96-advanced-observability-distributed-tracing.md) | SentinelOps | 2 |
| 121 | [200 Million Spans Per Day — Tail Sampling and Cost Control](./phase-2/09-telanova/ch121-distributed-tracing-opentelemetry.md) | TeleNova | 2 |

</details>

<a name="tag-dlq"></a>

<details>
<summary><strong>#dlq</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 12 | [Production is DOWN — The Notification Consumer Won't Stop Crashing](./phase-1/03-velotrack/ch12-dead-letter-queues-and-error-handling.md) | VeloTrack | 1 |
| 114 | [Concert Cancellation Notification Cascade (4th Appearance)](./phase-2/08-venueflow/ch114-notifications-fourth-appearance.md) | VenueFlow | 2 |

</details>

<a name="tag-edge"></a>

<details>
<summary><strong>#edge</strong> — 4 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 197 | [The Fractured Map](./phase-3/07-automesh/ch197-hd-map-distribution.md) | AutoMesh | 3 |
| 203 | [OPC-UA to Cloud](./phase-3/08-forgesense/ch203-opc-ua-to-cloud.md) | ForgeSense | 3 |
| 210 | [Three Weeks at Sea](./phase-3/09-deepocean/ch210-offline-first-for-ships.md) | DeepOcean | 3 |
| 213 | [Designing for the Speed of Light](./phase-3/09-deepocean/ch213-extreme-latency-architecture.md) | DeepOcean | 3 |

</details>

<a name="tag-edge-computing"></a>

<details>
<summary><strong>#edge-computing</strong> — 6 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 26 | [Design for Unreliable Sensors at the Edge](./phase-1/05-agrosense/ch26-edge-processing-and-sensor-reliability.md) | AgroSense | 1 |
| 182 | [Edge Intelligence](./phase-3/05-neuralbridge/ch182-edge-intelligence.md) | NeuralBridge | 3 |
| 194 | [The Safety-Critical Edge](./phase-3/07-automesh/ch194-the-safety-critical-edge.md) | AutoMesh | 3 |
| 202 | [The Digital Twin](./phase-3/08-forgesense/ch202-the-digital-twin.md) | ForgeSense | 3 |
| 207 | [The 200ms Window](./phase-3/08-forgesense/ch207-real-time-control-layer.md) | ForgeSense | 3 |
| 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | HarvestAI | 3 |

</details>

<a name="tag-edge-ml"></a>

<details>
<summary><strong>#edge-ml</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 212 | [2,000 Miles from Anywhere](./phase-3/09-deepocean/ch212-edge-processing-at-sea.md) | DeepOcean | 3 |
| 225 | [The Model in the Field](./phase-3/11-harvestai/ch225-edge-ml-inference.md) | HarvestAI | 3 |

</details>

<a name="tag-elasticsearch"></a>

<details>
<summary><strong>#elasticsearch</strong> — 8 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 33 | [Design the Product Search System (First Appearance)](./phase-1/07-pulsecommerce/ch33-search-inverted-index-elasticsearch.md) | PulseCommerce | 1 |
| 45 | [Search Over Academic Content — Scaling Challenges (Second Appearance)](./phase-1/09-neurolearn/ch45-search-second-appearance.md) | NeuroLearn | 1 |
| 60 | [Flight Search — The Hardest Search Problem](./phase-1/12-skyroute/ch60-flight-search-third-appearance.md) | SkyRoute | 1 |
| 89 | [Advanced Search — Facets and Typeahead](./phase-2/04-giggrid/ch89-advanced-search-facets-typeahead.md) | GigGrid | 2 |
| 115 | [Vector Search — Personalized Event Recommendations](./phase-2/08-venueflow/ch115-search-fourth-appearance.md) | VenueFlow | 2 |
| 127 | [Advanced Search — Material Catalog and Vendor Discovery](./phase-2/10-buildright/ch127-advanced-search-typeahead-facets.md) | BuildRight | 2 |
| 239 | [Search for Property Intelligence](./phase-3/13-propiq/ch239-search-for-property-intelligence.md) | PropIQ | 3 |
| 243 | [The Garbage Search](./phase-3/13-propiq/ch243-search-at-property-scale.md) | PropIQ | 3 |

</details>

<a name="tag-email"></a>

<details>
<summary><strong>#email</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 20 | [Design the Notification System (First Appearance)](./phase-1/04-beacon-media/ch20-notification-system-first-appearance.md) | Beacon Media | 1 |
| 43 | [Notification System at Scale — The Finals Week Problem (Second Appearance)](./phase-1/09-neurolearn/ch43-notification-system-second-appearance.md) | NeuroLearn | 1 |

</details>

<a name="tag-embeddings"></a>

<details>
<summary><strong>#embeddings</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 132 | [Finding Genetic Needles in Petabyte Haystacks](./phase-2/11-axiom-labs/ch132-vector-search-deep-dive.md) | Axiom Labs | 2 |
| 172 | [The Chunking Pipeline](./phase-3/04-lexcore/ch172-the-chunking-pipeline.md) | LexCore | 3 |
| 173 | [Where Are the Embeddings?](./phase-3/04-lexcore/ch173-design-review-ambush-spike.md) | LexCore | 3 |

</details>

<a name="tag-erisa"></a>

<details>
<summary><strong>#erisa</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 230 | [Fifty Years of Data](./phase-3/12-nexuswealth/ch230-fifty-years-of-data.md) | NexusWealth | 3 |
| 231 | [ERISA and SEC Compliance](./phase-3/12-nexuswealth/ch231-erisa-and-sec-compliance.md) | NexusWealth | 3 |

</details>

<a name="tag-error-budget"></a>

<details>
<summary><strong>#error-budget</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 74 | [Advanced Observability — SLOs, SLIs, and Error Budgets](./phase-2/02-nexacare/ch74-advanced-observability-slos.md) | NexaCare | 2 |
| 109 | [Grid Reliability SLOs — When 99.9% Is Not Good Enough](./phase-2/07-crestline-energy/ch109-slo-sli-error-budget-design.md) | Crestline Energy | 2 |

</details>

<a name="tag-escalation"></a>

<details>
<summary><strong>#escalation</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 140 | [Life-Safety Alerting — When False Positives Kill Trust](./phase-2/12-prismhealth/ch140-advanced-observability-alerting.md) | PrismHealth | 2 |
| 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | MindScale | 3 |

</details>

<a name="tag-etl"></a>

<details>
<summary><strong>#etl</strong> — 6 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 24 | [Build the ETL Pipeline and Change Data Capture](./phase-1/05-agrosense/ch24-etl-pipeline-and-cdc.md) | AgroSense | 1 |
| 55 | [ETL to Streaming — Evolving the Data Pipeline](./phase-1/11-luminaryai/ch55-etl-to-streaming-pipeline-evolution.md) | LuminaryAI | 1 |
| 103 | [Data Warehousing — OLAP vs OLTP](./phase-2/06-lightspeedretail/ch103-data-warehousing-olap-vs-oltp.md) | LightspeedRetail | 2 |
| 217 | [Lab Data Integration](./phase-3/10-pharmasync/ch217-lab-data-integration.md) | PharmaSync | 3 |
| 238 | [Twelve Sources, One Truth](./phase-3/13-propiq/ch238-the-property-data-mesh.md) | PropIQ | 3 |
| 242 | [The Real-Time Lie](./phase-3/13-propiq/ch242-real-time-market-aggregation.md) | PropIQ | 3 |

</details>

<a name="tag-event-sourcing"></a>

<details>
<summary><strong>#event-sourcing</strong> — 11 chapters across 9 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 66 | [CQRS and Event Sourcing](./phase-2/01-stratum-systems/ch66-cqrs-and-event-sourcing.md) | Stratum Systems | 2 |
| 70 | [GDPR Deletion Pipelines](./phase-2/02-nexacare/ch70-gdpr-deletion-pipelines.md) | NexaCare | 2 |
| 77 | [Low-Latency Event Sourcing for Trade Execution](./phase-2/03-tradespark/ch77-low-latency-event-sourcing.md) | TradeSpark | 2 |
| 104 | [CQRS Advanced Projections — Real-Time Inventory](./phase-2/06-lightspeedretail/ch104-cqrs-advanced-projections.md) | LightspeedRetail | 2 |
| 112 | ["What Was the Grid State at 14:32:07 on March 3rd?"](./phase-2/07-crestline-energy/ch112-cqrs-temporal-queries.md) | Crestline Energy | 2 |
| 116 | [Ticket Inventory Consistency — The Taylor Swift Problem](./phase-2/08-venueflow/ch116-ticket-inventory-consistency.md) | VenueFlow | 2 |
| 126 | [CQRS and Advanced Event Sourcing for Project State](./phase-2/10-buildright/ch126-cqrs-event-sourcing-advanced.md) | BuildRight | 2 |
| 131 | [GDPR Erasure in Event-Sourced Systems](./phase-2/10-buildright/ch131-compliance-gdpr-erasure-and-event-logs.md) | BuildRight | 2 |
| 142 | [A Patient's Complete Vital History](./phase-2/12-prismhealth/ch142-cqrs-for-patient-timelines.md) | PrismHealth | 2 |
| 165 | [The Ledger That Cannot Lie](./phase-3/03-carbonledger/ch165-the-ledger-that-cannot-lie.md) | CarbonLedger | 3 |
| 168 | [Event Sourcing for Regulators](./phase-3/03-carbonledger/ch168-event-sourcing-for-regulators.md) | CarbonLedger | 3 |

</details>

<a name="tag-eventual-consistency"></a>

<details>
<summary><strong>#eventual-consistency</strong> — 7 chapters across 6 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 49 | [CAP Theorem in Practice — When the Network Fails](./phase-1/10-omnilogix/ch49-cap-theorem-and-eventual-consistency.md) | OmniLogix | 1 |
| 53 | [Global Inventory Consistency — The Full Design](./phase-1/10-omnilogix/ch53-global-inventory-consistency-synthesis.md) | OmniLogix | 1 |
| 54 | [The Feature Store — Feeding Models the Right Data](./phase-1/11-luminaryai/ch54-feature-store-and-ml-pipeline.md) | LuminaryAI | 1 |
| 66 | [CQRS and Event Sourcing](./phase-2/01-stratum-systems/ch66-cqrs-and-event-sourcing.md) | Stratum Systems | 2 |
| 85 | [CRDTs — Conflict-Free Replicated Data Types](./phase-2/04-giggrid/ch85-crdts-introduction.md) | GigGrid | 2 |
| 104 | [CQRS Advanced Projections — Real-Time Inventory](./phase-2/06-lightspeedretail/ch104-cqrs-advanced-projections.md) | LightspeedRetail | 2 |
| 166 | [Article 6 Compliance](./phase-3/03-carbonledger/ch166-article-6-compliance.md) | CarbonLedger | 3 |

</details>

<a name="tag-exactly-once"></a>

<details>
<summary><strong>#exactly-once</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 15 | [The Delivery State Machine at Scale](./phase-1/03-velotrack/ch15-kafka-exactly-once-and-delivery-state-machine.md) | VeloTrack | 1 |
| 51 | [Idempotency at Scale — When Retries Become Dangerous](./phase-1/10-omnilogix/ch51-idempotency-at-scale.md) | OmniLogix | 1 |
| 52 | [Kafka Exactly-Once at the Application Level](./phase-1/10-omnilogix/ch52-kafka-exactly-once-application-level.md) | OmniLogix | 1 |

</details>

<a name="tag-experimentation"></a>

<details>
<summary><strong>#experimentation</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 117 | [A/B Testing Infrastructure](./phase-2/08-venueflow/ch117-a-b-testing-infrastructure.md) | VenueFlow | 2 |
| 128 | [Experimentation Platform — Contradictory A/B Results](./phase-2/10-buildright/ch128-a-b-testing-and-experimentation.md) | BuildRight | 2 |

</details>

<a name="tag-faceted-search"></a>

<details>
<summary><strong>#faceted-search</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 127 | [Advanced Search — Material Catalog and Vendor Discovery](./phase-2/10-buildright/ch127-advanced-search-typeahead-facets.md) | BuildRight | 2 |
| 243 | [The Garbage Search](./phase-3/13-propiq/ch243-search-at-property-scale.md) | PropIQ | 3 |

</details>

<a name="tag-failover"></a>

<details>
<summary><strong>#failover</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 151 | [The Cell Problem](./phase-3/01-orbitcore/ch151-the-cell-problem.md) | OrbitCore | 3 |
| 156 | [SPIKE — Solar Flare](./phase-3/01-orbitcore/ch156-solar-flare-spike.md) | OrbitCore | 3 |

</details>

<a name="tag-fan-out"></a>

<details>
<summary><strong>#fan-out</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 20 | [Design the Notification System (First Appearance)](./phase-1/04-beacon-media/ch20-notification-system-first-appearance.md) | Beacon Media | 1 |
| 43 | [Notification System at Scale — The Finals Week Problem (Second Appearance)](./phase-1/09-neurolearn/ch43-notification-system-second-appearance.md) | NeuroLearn | 1 |
| 59 | [Notification System — 8 Million in 40 Minutes](./phase-1/12-skyroute/ch59-notification-system-third-appearance.md) | SkyRoute | 1 |

</details>

<a name="tag-fda"></a>

<details>
<summary><strong>#fda</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 72 | [Advanced Audit Trails at Scale](./phase-2/02-nexacare/ch72-advanced-audit-trails-at-scale.md) | NexaCare | 2 |
| 179 | [FDA 510(k) Architecture](./phase-3/05-neuralbridge/ch179-fda-510k-architecture.md) | NeuralBridge | 3 |

</details>

<a name="tag-feature-flags"></a>

<details>
<summary><strong>#feature-flags</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 88 | [Canary Deployments and Feature Flags](./phase-2/04-giggrid/ch88-canary-deployments-and-feature-flags.md) | GigGrid | 2 |
| 117 | [A/B Testing Infrastructure](./phase-2/08-venueflow/ch117-a-b-testing-infrastructure.md) | VenueFlow | 2 |

</details>

<a name="tag-feature-store"></a>

<details>
<summary><strong>#feature-store</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 54 | [The Feature Store — Feeding Models the Right Data](./phase-1/11-luminaryai/ch54-feature-store-and-ml-pipeline.md) | LuminaryAI | 1 |
| 134 | [Drug Target Discovery — ML Pipelines at Research Scale](./phase-2/11-axiom-labs/ch134-ml-recommendation-pipeline.md) | Axiom Labs | 2 |

</details>

<a name="tag-feature-stores"></a>

<details>
<summary><strong>#feature-stores</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 204 | [The Webb Keynote](./phase-3/08-forgesense/ch204-the-webb-keynote.md) | ForgeSense | 3 |
| 247 | [35 Million Opinions, Updated in Real Time](./phase-3/14-novasports/ch247-personalization-at-scale.md) | NovaSports | 3 |

</details>

<a name="tag-federated-governance"></a>

<details>
<summary><strong>#federated-governance</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 136 | [200 Universities, 200 Data Silos](./phase-2/11-axiom-labs/ch136-data-mesh-intro.md) | Axiom Labs | 2 |
| 208 | [The Twelve Factories](./phase-3/08-forgesense/ch208-factory-data-mesh.md) | ForgeSense | 3 |

</details>

<a name="tag-fedramp"></a>

<details>
<summary><strong>#fedramp</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 39 | [Design the Data Sovereignty Architecture for Government Clients](./phase-1/08-civicos/ch39-data-sovereignty-and-compliance-architecture.md) | CivicOS | 1 |
| 41 | [SSO for 23 State Governments — Federated Identity at Scale](./phase-1/08-civicos/ch41-sso-and-identity-federation-for-government.md) | CivicOS | 1 |

</details>

<a name="tag-financial-systems"></a>

<details>
<summary><strong>#financial-systems</strong> — 4 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 232 | [The March 31st Problem](./phase-3/12-nexuswealth/ch232-actuarial-calculation-at-scale.md) | NexusWealth | 3 |
| 233 | [Ten Minutes to Exposure](./phase-3/12-nexuswealth/ch233-real-time-portfolio-risk.md) | NexusWealth | 3 |
| 234 | [VIX 80 — The ERISA Clock Is Ticking](./phase-3/12-nexuswealth/ch234-scale-10x-market-volatility-spike.md) | NexusWealth | 3 |
| 236 | [One Platform, Three Regulators](./phase-3/12-nexuswealth/ch236-data-sovereignty-pension-funds.md) | NexusWealth | 3 |

</details>

<a name="tag-finops"></a>

<details>
<summary><strong>#finops</strong> — 8 chapters across 8 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 110 | [Three Years of Growth, Two Years of Budget](./phase-2/07-crestline-energy/ch110-capacity-planning-and-forecasting.md) | Crestline Energy | 2 |
| 146 | [The $4.2M Infrastructure Bill Nobody Owned](./phase-2/13-vertexcloud/ch146-platform-observability-and-cost.md) | VertexCloud | 2 |
| 155 | [Capacity at Constellation Scale](./phase-3/01-orbitcore/ch155-capacity-at-constellation-scale.md) | OrbitCore | 3 |
| 169 | [The Cost of Green](./phase-3/03-carbonledger/ch169-the-cost-of-green.md) | CarbonLedger | 3 |
| 198 | [The $2.3M Conversation](./phase-3/07-automesh/ch198-the-cfo-presentation.md) | AutoMesh | 3 |
| 224 | [Design for 100x, Spend 2x](./phase-3/11-harvestai/ch224-the-seasonal-spike.md) | HarvestAI | 3 |
| 235 | [The Trustees Want Answers](./phase-3/12-nexuswealth/ch235-the-board-presentation.md) | NexusWealth | 3 |
| 265 | [Quantum Platform Economics](./phase-3/16-quantumedge/ch265-quantum-platform-economics.md) | QuantumEdge | 3 |

</details>

<a name="tag-fmea"></a>

<details>
<summary><strong>#fmea</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 194 | [The Safety-Critical Edge](./phase-3/07-automesh/ch194-the-safety-critical-edge.md) | AutoMesh | 3 |
| 199 | [Every Way This Can Kill Someone](./phase-3/07-automesh/ch199-fmea-driven-architecture.md) | AutoMesh | 3 |

</details>

<a name="tag-forensics"></a>

<details>
<summary><strong>#forensics</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 97 | [Security Incident Forensics Pipeline](./phase-2/05-sentinelops/ch97-security-incident-forensics-pipeline.md) | SentinelOps | 2 |
| 160 | [Inherited Disaster — The Data That Crossed the Line](./phase-3/02-ironwatch/ch160-inherited-disaster-data-breach.md) | IronWatch | 3 |
| 162 | [Audit Everything](./phase-3/02-ironwatch/ch162-audit-everything.md) | IronWatch | 3 |

</details>

<a name="tag-gdpr"></a>

<details>
<summary><strong>#gdpr</strong> — 9 chapters across 8 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 70 | [GDPR Deletion Pipelines](./phase-2/02-nexacare/ch70-gdpr-deletion-pipelines.md) | NexaCare | 2 |
| 73 | [Data Residency Across Multiple Jurisdictions](./phase-2/02-nexacare/ch73-data-residency-multi-jurisdiction.md) | NexaCare | 2 |
| 111 | [Energy Data Sovereignty — When Regulators Disagree](./phase-2/07-crestline-energy/ch111-multi-region-data-residency-advanced.md) | Crestline Energy | 2 |
| 131 | [GDPR Erasure in Event-Sourced Systems](./phase-2/10-buildright/ch131-compliance-gdpr-erasure-and-event-logs.md) | BuildRight | 2 |
| 133 | [Genomic Data Is Personal — Differential Privacy and Federated Computation](./phase-2/11-axiom-labs/ch133-privacy-preserving-systems.md) | Axiom Labs | 2 |
| 143 | [A German Patient's Heartbeat Cannot Leave Germany](./phase-2/12-prismhealth/ch143-multi-region-data-sovereignty-healthcare.md) | PrismHealth | 2 |
| 152 | [Sovereignty in Orbit](./phase-3/01-orbitcore/ch152-sovereignty-in-orbit.md) | OrbitCore | 3 |
| 180 | [Brain Data Privacy](./phase-3/05-neuralbridge/ch180-brain-data-privacy.md) | NeuralBridge | 3 |
| 257 | [Privacy by Forgetting](./phase-3/15-mindscale/ch257-the-data-minimization-architecture.md) | MindScale | 3 |

</details>

<a name="tag-genomics"></a>

<details>
<summary><strong>#genomics</strong> — 3 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 132 | [Finding Genetic Needles in Petabyte Haystacks](./phase-2/11-axiom-labs/ch132-vector-search-deep-dive.md) | Axiom Labs | 2 |
| 133 | [Genomic Data Is Personal — Differential Privacy and Federated Computation](./phase-2/11-axiom-labs/ch133-privacy-preserving-systems.md) | Axiom Labs | 2 |
| 137 | [Genomic PII in the Application Logs](./phase-2/11-axiom-labs/ch137-hipaa-compliance-remediation.md) | Axiom Labs | 2 |

</details>

<a name="tag-geospatial"></a>

<details>
<summary><strong>#geospatial</strong> — 7 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 13 | [Design the Real-Time Driver Tracking System](./phase-1/03-velotrack/ch13-real-time-delivery-tracking.md) | VeloTrack | 1 |
| 127 | [Advanced Search — Material Catalog and Vendor Discovery](./phase-2/10-buildright/ch127-advanced-search-typeahead-facets.md) | BuildRight | 2 |
| 209 | [The Ocean's Heartbeat](./phase-3/09-deepocean/ch209-ais-the-oceans-heartbeat.md) | DeepOcean | 3 |
| 223 | [When the Earth Won't Fit in a B-Tree](./phase-3/11-harvestai/ch223-geospatial-at-scale.md) | HarvestAI | 3 |
| 226 | [The Three-Meter Problem](./phase-3/11-harvestai/ch226-carbon-credits-integration.md) | HarvestAI | 3 |
| 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | HarvestAI | 3 |
| 243 | [The Garbage Search](./phase-3/13-propiq/ch243-search-at-property-scale.md) | PropIQ | 3 |

</details>

<a name="tag-gitops"></a>

<details>
<summary><strong>#gitops</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 129 | [Progressive Delivery with Argo Rollouts](./phase-2/10-buildright/ch129-canary-deployments-advanced.md) | BuildRight | 2 |
| 147 | [GitOps at 2,000 Engineers](./phase-2/13-vertexcloud/ch147-advanced-deployment-pipeline.md) | VertexCloud | 2 |

</details>

<a name="tag-government"></a>

<details>
<summary><strong>#government</strong> — 4 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 39 | [Design the Data Sovereignty Architecture for Government Clients](./phase-1/08-civicos/ch39-data-sovereignty-and-compliance-architecture.md) | CivicOS | 1 |
| 40 | [Rate Limiting for Government APIs — The Enumeration Attack](./phase-1/08-civicos/ch40-rate-limiting-second-appearance.md) | CivicOS | 1 |
| 41 | [SSO for 23 State Governments — Federated Identity at Scale](./phase-1/08-civicos/ch41-sso-and-identity-federation-for-government.md) | CivicOS | 1 |
| 42 | [Design for Accessibility, Resilience, and the Citizen Who Can't Retry](./phase-1/08-civicos/ch42-accessibility-and-resilience-for-government-systems.md) | CivicOS | 1 |

</details>

<a name="tag-grid"></a>

<details>
<summary><strong>#grid</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 266 | [The Virtual Power Plant](./phase-3/17-glaciergrid/ch266-the-virtual-power-plant.md) | GlacierGrid | 3 |
| 269 | [Three ISOs, One Morning](./phase-3/17-glaciergrid/ch269-production-down-grid-cascade.md) | GlacierGrid | 3 |

</details>

<a name="tag-healthcare"></a>

<details>
<summary><strong>#healthcare</strong> — 6 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 138 | [14 Million Device Readings Per Minute](./phase-2/12-prismhealth/ch138-iot-at-scale-advanced.md) | PrismHealth | 2 |
| 139 | [mTLS for HIPAA — Every Service Call Is a PHI Flow](./phase-2/12-prismhealth/ch139-service-mesh-healthcare-compliance.md) | PrismHealth | 2 |
| 140 | [Life-Safety Alerting — When False Positives Kill Trust](./phase-2/12-prismhealth/ch140-advanced-observability-alerting.md) | PrismHealth | 2 |
| 141 | [The Vacuum That Wasn't Running](./phase-2/12-prismhealth/ch141-database-internals-postgres-tuning.md) | PrismHealth | 2 |
| 143 | [A German Patient's Heartbeat Cannot Leave Germany](./phase-2/12-prismhealth/ch143-multi-region-data-sovereignty-healthcare.md) | PrismHealth | 2 |
| 254 | [The Consent That Can't Be a Toggle](./phase-3/15-mindscale/ch254-42-cfr-part-2-compliance.md) | MindScale | 3 |

</details>

<a name="tag-hipaa"></a>

<details>
<summary><strong>#hipaa</strong> — 16 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 06 | [Design the Authentication and Authorization System](./phase-1/02-meridian-health/ch06-jwt-oauth2-auth-system.md) | MeridianHealth | 1 |
| 07 | [Build the HIPAA-Compliant Audit Trail](./phase-1/02-meridian-health/ch07-hipaa-audit-trails.md) | MeridianHealth | 1 |
| 08 | [Data Residency — The State That Wants Its Data Back](./phase-1/02-meridian-health/ch08-data-residency-and-hipaa-architecture.md) | MeridianHealth | 1 |
| 09 | [Scale the EHR Query Layer with Read Replicas](./phase-1/02-meridian-health/ch09-read-replicas-and-query-routing.md) | MeridianHealth | 1 |
| 10 | [Scale 10x by Friday — The Northview Surge](./phase-1/02-meridian-health/ch10-ehr-integration-platform-design.md) | MeridianHealth | 1 |
| 133 | [Genomic Data Is Personal — Differential Privacy and Federated Computation](./phase-2/11-axiom-labs/ch133-privacy-preserving-systems.md) | Axiom Labs | 2 |
| 137 | [Genomic PII in the Application Logs](./phase-2/11-axiom-labs/ch137-hipaa-compliance-remediation.md) | Axiom Labs | 2 |
| 139 | [mTLS for HIPAA — Every Service Call Is a PHI Flow](./phase-2/12-prismhealth/ch139-service-mesh-healthcare-compliance.md) | PrismHealth | 2 |
| 142 | [A Patient's Complete Vital History](./phase-2/12-prismhealth/ch142-cqrs-for-patient-timelines.md) | PrismHealth | 2 |
| 143 | [A German Patient's Heartbeat Cannot Leave Germany](./phase-2/12-prismhealth/ch143-multi-region-data-sovereignty-healthcare.md) | PrismHealth | 2 |
| 178 | [The Signal](./phase-3/05-neuralbridge/ch178-the-signal.md) | NeuralBridge | 3 |
| 180 | [Brain Data Privacy](./phase-3/05-neuralbridge/ch180-brain-data-privacy.md) | NeuralBridge | 3 |
| 254 | [The Consent That Can't Be a Toggle](./phase-3/15-mindscale/ch254-42-cfr-part-2-compliance.md) | MindScale | 3 |
| 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | MindScale | 3 |
| 256 | [The 23 Percent](./phase-3/15-mindscale/ch256-re-identification-risk-mitigation.md) | MindScale | 3 |
| 257 | [Privacy by Forgetting](./phase-3/15-mindscale/ch257-the-data-minimization-architecture.md) | MindScale | 3 |

</details>

<a name="tag-human-in-the-loop"></a>

<details>
<summary><strong>#human-in-the-loop</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 253 | [The Precision-Recall Dilemma](./phase-3/15-mindscale/ch253-crisis-detection-ml.md) | MindScale | 3 |
| 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | MindScale | 3 |

</details>

<a name="tag-idempotency"></a>

<details>
<summary><strong>#idempotency</strong> — 12 chapters across 8 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 01 | [Design the Payment Processing Pipeline](./phase-1/01-novapay/ch01-payment-processing-pipeline.md) | NovaPay | 1 |
| 02 | [Production is DOWN — Duplicate Charges at Scale](./phase-1/01-novapay/ch02-idempotency-and-duplicate-prevention.md) | NovaPay | 1 |
| 15 | [The Delivery State Machine at Scale](./phase-1/03-velotrack/ch15-kafka-exactly-once-and-delivery-state-machine.md) | VeloTrack | 1 |
| 31 | [Design the Tenant Billing and Usage Metering System](./phase-1/06-cloudstack/ch31-tenant-billing-and-usage-metering.md) | CloudStack | 1 |
| 34 | [Payment Integration at Scale — Scaling Challenges (Second Appearance)](./phase-1/07-pulsecommerce/ch34-payment-processing-second-appearance.md) | PulseCommerce | 1 |
| 50 | [Design the Multi-Step Supply Chain Transaction with Saga](./phase-1/10-omnilogix/ch50-distributed-transactions-and-saga-pattern.md) | OmniLogix | 1 |
| 51 | [Idempotency at Scale — When Retries Become Dangerous](./phase-1/10-omnilogix/ch51-idempotency-at-scale.md) | OmniLogix | 1 |
| 52 | [Kafka Exactly-Once at the Application Level](./phase-1/10-omnilogix/ch52-kafka-exactly-once-application-level.md) | OmniLogix | 1 |
| 53 | [Global Inventory Consistency — The Full Design](./phase-1/10-omnilogix/ch53-global-inventory-consistency-synthesis.md) | OmniLogix | 1 |
| 56 | [Payment Infrastructure — Billing the ML Platform](./phase-1/11-luminaryai/ch56-payment-processing-third-appearance.md) | LuminaryAI | 1 |
| 81 | [DTCC Clearing Integration and Financial Settlement](./phase-2/03-tradespark/ch81-payments-fourth-appearance.md) | TradeSpark | 2 |
| 99 | [POS Terminal Payments and P2PE](./phase-2/06-lightspeedretail/ch99-payments-fifth-appearance.md) | LightspeedRetail | 2 |

</details>

<a name="tag-identity"></a>

<details>
<summary><strong>#identity</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 41 | [SSO for 23 State Governments — Federated Identity at Scale](./phase-1/08-civicos/ch41-sso-and-identity-federation-for-government.md) | CivicOS | 1 |
| 61 | [Auth for the Most Adversarial User Base](./phase-1/12-skyroute/ch61-auth-third-appearance.md) | SkyRoute | 1 |

</details>

<a name="tag-immutability"></a>

<details>
<summary><strong>#immutability</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 162 | [Audit Everything](./phase-3/02-ironwatch/ch162-audit-everything.md) | IronWatch | 3 |
| 165 | [The Ledger That Cannot Lie](./phase-3/03-carbonledger/ch165-the-ledger-that-cannot-lie.md) | CarbonLedger | 3 |
| 183 | [The Clinical Data Platform](./phase-3/05-neuralbridge/ch183-the-clinical-data-platform.md) | NeuralBridge | 3 |

</details>

<a name="tag-incident-management"></a>

<details>
<summary><strong>#incident-management</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | HarvestAI | 3 |
| 250 | [The Postmortem (And the Architecture We Should Have Built)](./phase-3/14-novasports/ch250-blameless-postmortem-and-redesign.md) | NovaSports | 3 |

</details>

<a name="tag-incident-response"></a>

<details>
<summary><strong>#incident-response</strong> — 15 chapters across 13 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 02 | [Production is DOWN — Duplicate Charges at Scale](./phase-1/01-novapay/ch02-idempotency-and-duplicate-prevention.md) | NovaPay | 1 |
| 12 | [Production is DOWN — The Notification Consumer Won't Stop Crashing](./phase-1/03-velotrack/ch12-dead-letter-queues-and-error-handling.md) | VeloTrack | 1 |
| 97 | [Security Incident Forensics Pipeline](./phase-2/05-sentinelops/ch97-security-incident-forensics-pipeline.md) | SentinelOps | 2 |
| 149 | [The Game Day That Became a Real Incident](./phase-2/13-vertexcloud/ch149-chaos-engineering-intro.md) | VertexCloud | 2 |
| 156 | [SPIKE — Solar Flare](./phase-3/01-orbitcore/ch156-solar-flare-spike.md) | OrbitCore | 3 |
| 160 | [Inherited Disaster — The Data That Crossed the Line](./phase-3/02-ironwatch/ch160-inherited-disaster-data-breach.md) | IronWatch | 3 |
| 188 | [Claims at Scale](./phase-3/06-shieldmutual/ch188-claims-at-scale.md) | ShieldMutual | 3 |
| 191 | [Production DOWN — Claims Cascade Failure](./phase-3/06-shieldmutual/ch191-production-down-claims-cascade.md) | ShieldMutual | 3 |
| 205 | [The Ghost in the Machine](./phase-3/08-forgesense/ch205-inherited-disaster-scada.md) | ForgeSense | 3 |
| 221 | [The Two Phantom Patients](./phase-3/10-pharmasync/ch221-production-down-clinical-trial.md) | PharmaSync | 3 |
| 234 | [VIX 80 — The ERISA Clock Is Ticking](./phase-3/12-nexuswealth/ch234-scale-10x-market-volatility-spike.md) | NexusWealth | 3 |
| 240 | [Inherited Disaster — The Data Quality Crisis](./phase-3/13-propiq/ch240-inherited-disaster-data-quality-crisis.md) | PropIQ | 3 |
| 249 | [The Game Day That Went Wrong](./phase-3/14-novasports/ch249-the-game-day-that-went-wrong.md) | NovaSports | 3 |
| 266 | [The Virtual Power Plant](./phase-3/17-glaciergrid/ch266-the-virtual-power-plant.md) | GlacierGrid | 3 |
| 269 | [Three ISOs, One Morning](./phase-3/17-glaciergrid/ch269-production-down-grid-cascade.md) | GlacierGrid | 3 |

</details>

<a name="tag-industrial"></a>

<details>
<summary><strong>#industrial</strong> — 3 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 205 | [The Ghost in the Machine](./phase-3/08-forgesense/ch205-inherited-disaster-scada.md) | ForgeSense | 3 |
| 206 | [What Do You Mean, Read-Only?](./phase-3/08-forgesense/ch206-design-review-ambush-brownfield.md) | ForgeSense | 3 |
| 207 | [The 200ms Window](./phase-3/08-forgesense/ch207-real-time-control-layer.md) | ForgeSense | 3 |

</details>

<a name="tag-infrastructure"></a>

<details>
<summary><strong>#infrastructure</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 110 | [Three Years of Growth, Two Years of Budget](./phase-2/07-crestline-energy/ch110-capacity-planning-and-forecasting.md) | Crestline Energy | 2 |
| 155 | [Capacity at Constellation Scale](./phase-3/01-orbitcore/ch155-capacity-at-constellation-scale.md) | OrbitCore | 3 |
| 198 | [The $2.3M Conversation](./phase-3/07-automesh/ch198-the-cfo-presentation.md) | AutoMesh | 3 |

</details>

<a name="tag-inherited-disaster"></a>

<details>
<summary><strong>#inherited-disaster</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 160 | [Inherited Disaster — The Data That Crossed the Line](./phase-3/02-ironwatch/ch160-inherited-disaster-data-breach.md) | IronWatch | 3 |
| 205 | [The Ghost in the Machine](./phase-3/08-forgesense/ch205-inherited-disaster-scada.md) | ForgeSense | 3 |
| 240 | [Inherited Disaster — The Data Quality Crisis](./phase-3/13-propiq/ch240-inherited-disaster-data-quality-crisis.md) | PropIQ | 3 |

</details>

<a name="tag-insurance"></a>

<details>
<summary><strong>#insurance</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 186 | [The Risk Engine](./phase-3/06-shieldmutual/ch186-the-risk-engine.md) | ShieldMutual | 3 |
| 187 | [The Actuarial Pipeline](./phase-3/06-shieldmutual/ch187-the-actuarial-pipeline.md) | ShieldMutual | 3 |

</details>

<a name="tag-integration"></a>

<details>
<summary><strong>#integration</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 217 | [Lab Data Integration](./phase-3/10-pharmasync/ch217-lab-data-integration.md) | PharmaSync | 3 |
| 226 | [The Three-Meter Problem](./phase-3/11-harvestai/ch226-carbon-credits-integration.md) | HarvestAI | 3 |

</details>

<a name="tag-inventory"></a>

<details>
<summary><strong>#inventory</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 37 | [Flash Sale — Inventory Overselling Under Pressure](./phase-1/07-pulsecommerce/ch37-inventory-management-and-consistency.md) | PulseCommerce | 1 |
| 63 | [Seat Inventory — The Overbooking Problem](./phase-1/12-skyroute/ch63-seat-inventory-and-overbooking.md) | SkyRoute | 1 |
| 104 | [CQRS Advanced Projections — Real-Time Inventory](./phase-2/06-lightspeedretail/ch104-cqrs-advanced-projections.md) | LightspeedRetail | 2 |
| 116 | [Ticket Inventory Consistency — The Taylor Swift Problem](./phase-2/08-venueflow/ch116-ticket-inventory-consistency.md) | VenueFlow | 2 |

</details>

<a name="tag-iot"></a>

<details>
<summary><strong>#iot</strong> — 10 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 23 | [Design the Time-Series Sensor Data Platform](./phase-1/05-agrosense/ch23-time-series-data-design.md) | AgroSense | 1 |
| 26 | [Design for Unreliable Sensors at the Edge](./phase-1/05-agrosense/ch26-edge-processing-and-sensor-reliability.md) | AgroSense | 1 |
| 106 | [Time-Series at Scale — 50M Meters, 200M Events Per Hour](./phase-2/07-crestline-energy/ch106-time-series-at-scale-advanced.md) | Crestline Energy | 2 |
| 138 | [14 Million Device Readings Per Minute](./phase-2/12-prismhealth/ch138-iot-at-scale-advanced.md) | PrismHealth | 2 |
| 182 | [Edge Intelligence](./phase-3/05-neuralbridge/ch182-edge-intelligence.md) | NeuralBridge | 3 |
| 202 | [The Digital Twin](./phase-3/08-forgesense/ch202-the-digital-twin.md) | ForgeSense | 3 |
| 203 | [OPC-UA to Cloud](./phase-3/08-forgesense/ch203-opc-ua-to-cloud.md) | ForgeSense | 3 |
| 225 | [The Model in the Field](./phase-3/11-harvestai/ch225-edge-ml-inference.md) | HarvestAI | 3 |
| 267 | [The 500ms Problem](./phase-3/17-glaciergrid/ch267-grid-edge-intelligence.md) | GlacierGrid | 3 |
| 268 | [The Numbers Are Wrong](./phase-3/17-glaciergrid/ch268-distributed-solar-wind-aggregation.md) | GlacierGrid | 3 |

</details>

<a name="tag-iso26262"></a>

<details>
<summary><strong>#iso26262</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 194 | [The Safety-Critical Edge](./phase-3/07-automesh/ch194-the-safety-critical-edge.md) | AutoMesh | 3 |
| 199 | [Every Way This Can Kill Someone](./phase-3/07-automesh/ch199-fmea-driven-architecture.md) | AutoMesh | 3 |

</details>

<a name="tag-isolation"></a>

<details>
<summary><strong>#isolation</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 32 | [Solve the Noisy Neighbor Problem for Good](./phase-1/06-cloudstack/ch32-noisy-neighbor-and-resource-quotas.md) | CloudStack | 1 |
| 181 | [The Systemic Failure](./phase-3/05-neuralbridge/ch181-the-systemic-failure.md) | NeuralBridge | 3 |

</details>

<a name="tag-istio"></a>

<details>
<summary><strong>#istio</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 83 | [Service Mesh and mTLS Introduction](./phase-2/03-tradespark/ch83-service-mesh-mtls-intro.md) | TradeSpark | 2 |
| 93 | [Service Mesh Deep Dive — Istio in Production](./phase-2/05-sentinelops/ch93-service-mesh-deep-dive.md) | SentinelOps | 2 |
| 120 | [Service Mesh at Production Scale — 60 Services, 18 Months of Istio](./phase-2/09-telanova/ch120-service-mesh-production.md) | TeleNova | 2 |
| 139 | [mTLS for HIPAA — Every Service Call Is a PHI Flow](./phase-2/12-prismhealth/ch139-service-mesh-healthcare-compliance.md) | PrismHealth | 2 |

</details>

<a name="tag-itar"></a>

<details>
<summary><strong>#itar</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 152 | [Sovereignty in Orbit](./phase-3/01-orbitcore/ch152-sovereignty-in-orbit.md) | OrbitCore | 3 |
| 156 | [SPIKE — Solar Flare](./phase-3/01-orbitcore/ch156-solar-flare-spike.md) | OrbitCore | 3 |

</details>

<a name="tag-jaeger"></a>

<details>
<summary><strong>#jaeger</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 69 | [Distributed Tracing with OpenTelemetry](./phase-2/01-stratum-systems/ch69-distributed-tracing-intro.md) | Stratum Systems | 2 |
| 96 | [Advanced Observability — Distributed Tracing at 50M Spans/Day](./phase-2/05-sentinelops/ch96-advanced-observability-distributed-tracing.md) | SentinelOps | 2 |

</details>

<a name="tag-kafka"></a>

<details>
<summary><strong>#kafka</strong> — 19 chapters across 14 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 11 | [Untangle the Kafka Mess](./phase-1/03-velotrack/ch11-kafka-deep-dive-consumer-groups.md) | VeloTrack | 1 |
| 12 | [Production is DOWN — The Notification Consumer Won't Stop Crashing](./phase-1/03-velotrack/ch12-dead-letter-queues-and-error-handling.md) | VeloTrack | 1 |
| 13 | [Design the Real-Time Driver Tracking System](./phase-1/03-velotrack/ch13-real-time-delivery-tracking.md) | VeloTrack | 1 |
| 15 | [The Delivery State Machine at Scale](./phase-1/03-velotrack/ch15-kafka-exactly-once-and-delivery-state-machine.md) | VeloTrack | 1 |
| 20 | [Design the Notification System (First Appearance)](./phase-1/04-beacon-media/ch20-notification-system-first-appearance.md) | Beacon Media | 1 |
| 24 | [Build the ETL Pipeline and Change Data Capture](./phase-1/05-agrosense/ch24-etl-pipeline-and-cdc.md) | AgroSense | 1 |
| 52 | [Kafka Exactly-Once at the Application Level](./phase-1/10-omnilogix/ch52-kafka-exactly-once-application-level.md) | OmniLogix | 1 |
| 55 | [ETL to Streaming — Evolving the Data Pipeline](./phase-1/11-luminaryai/ch55-etl-to-streaming-pipeline-evolution.md) | LuminaryAI | 1 |
| 59 | [Notification System — 8 Million in 40 Minutes](./phase-1/12-skyroute/ch59-notification-system-third-appearance.md) | SkyRoute | 1 |
| 79 | [Advanced Kafka — Log Compaction for Order Book State](./phase-2/03-tradespark/ch79-advanced-kafka-log-compaction.md) | TradeSpark | 2 |
| 98 | [Kafka at Scale — 40 Billion Security Events/Day](./phase-2/05-sentinelops/ch98-advanced-kafka-security-events.md) | SentinelOps | 2 |
| 122 | [Six Hours of Consumer Lag](./phase-2/09-telanova/ch122-advanced-kafka-consumer-lag.md) | TeleNova | 2 |
| 130 | [Data Pipeline Schema Evolution](./phase-2/10-buildright/ch130-data-pipeline-schema-evolution.md) | BuildRight | 2 |
| 178 | [The Signal](./phase-3/05-neuralbridge/ch178-the-signal.md) | NeuralBridge | 3 |
| 181 | [The Systemic Failure](./phase-3/05-neuralbridge/ch181-the-systemic-failure.md) | NeuralBridge | 3 |
| 188 | [Claims at Scale](./phase-3/06-shieldmutual/ch188-claims-at-scale.md) | ShieldMutual | 3 |
| 209 | [The Ocean's Heartbeat](./phase-3/09-deepocean/ch209-ais-the-oceans-heartbeat.md) | DeepOcean | 3 |
| 245 | [50 Million Events Per Second](./phase-3/14-novasports/ch245-50-million-events-per-second.md) | NovaSports | 3 |
| 251 | [Before Next Season](./phase-3/14-novasports/ch251-streaming-architecture-review.md) | NovaSports | 3 |

</details>

<a name="tag-kubernetes"></a>

<details>
<summary><strong>#kubernetes</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 32 | [Solve the Noisy Neighbor Problem for Good](./phase-1/06-cloudstack/ch32-noisy-neighbor-and-resource-quotas.md) | CloudStack | 1 |
| 92 | [Zero-Trust Architecture](./phase-2/05-sentinelops/ch92-zero-trust-architecture.md) | SentinelOps | 2 |

</details>

<a name="tag-latency"></a>

<details>
<summary><strong>#latency</strong> — 6 chapters across 6 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 120 | [Service Mesh at Production Scale — 60 Services, 18 Months of Istio](./phase-2/09-telanova/ch120-service-mesh-production.md) | TeleNova | 2 |
| 153 | [The Latency Ladder](./phase-3/01-orbitcore/ch153-the-latency-ladder.md) | OrbitCore | 3 |
| 186 | [The Risk Engine](./phase-3/06-shieldmutual/ch186-the-risk-engine.md) | ShieldMutual | 3 |
| 207 | [The 200ms Window](./phase-3/08-forgesense/ch207-real-time-control-layer.md) | ForgeSense | 3 |
| 213 | [Designing for the Speed of Light](./phase-3/09-deepocean/ch213-extreme-latency-architecture.md) | DeepOcean | 3 |
| 267 | [The 500ms Problem](./phase-3/17-glaciergrid/ch267-grid-edge-intelligence.md) | GlacierGrid | 3 |

</details>

<a name="tag-legal"></a>

<details>
<summary><strong>#legal</strong> — 7 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 126 | [CQRS and Advanced Event Sourcing for Project State](./phase-2/10-buildright/ch126-cqrs-event-sourcing-advanced.md) | BuildRight | 2 |
| 131 | [GDPR Erasure in Event-Sourced Systems](./phase-2/10-buildright/ch131-compliance-gdpr-erasure-and-event-logs.md) | BuildRight | 2 |
| 163 | [The Exit Brief](./phase-3/02-ironwatch/ch163-the-exit-brief.md) | IronWatch | 3 |
| 171 | [Privileged Information](./phase-3/04-lexcore/ch171-privileged-information.md) | LexCore | 3 |
| 174 | [The RFC That Failed](./phase-3/04-lexcore/ch174-the-rfc-that-failed.md) | LexCore | 3 |
| 176 | [Find the Precedent](./phase-3/04-lexcore/ch176-search-at-legal-scale.md) | LexCore | 3 |
| 240 | [Inherited Disaster — The Data Quality Crisis](./phase-3/13-propiq/ch240-inherited-disaster-data-quality-crisis.md) | PropIQ | 3 |

</details>

<a name="tag-load-balancing"></a>

<details>
<summary><strong>#load-balancing</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 14 | [Design the Load Balancing Strategy for a Heterogeneous Fleet](./phase-1/03-velotrack/ch14-load-balancing-strategies.md) | VeloTrack | 1 |
| 58 | [SPIKE — Scale 10x by Friday](./phase-1/11-luminaryai/ch58-recommendation-serving-scale.md) | LuminaryAI | 1 |

</details>

<a name="tag-lsm-tree"></a>

<details>
<summary><strong>#lsm-tree</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 71 | [Database Internals — B-Tree vs LSM-Tree](./phase-2/02-nexacare/ch71-database-internals-btree-vs-lsm.md) | NexaCare | 2 |
| 78 | [Database Internals — LSM-Tree Compaction Storm](./phase-2/03-tradespark/ch78-database-internals-deep-dive.md) | TradeSpark | 2 |

</details>

<a name="tag-maritime"></a>

<details>
<summary><strong>#maritime</strong> — 3 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 210 | [Three Weeks at Sea](./phase-3/09-deepocean/ch210-offline-first-for-ships.md) | DeepOcean | 3 |
| 212 | [2,000 Miles from Anywhere](./phase-3/09-deepocean/ch212-edge-processing-at-sea.md) | DeepOcean | 3 |
| 214 | [Flag State to Port State](./phase-3/09-deepocean/ch214-maritime-compliance.md) | DeepOcean | 3 |

</details>

<a name="tag-medical-device"></a>

<details>
<summary><strong>#medical-device</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 179 | [FDA 510(k) Architecture](./phase-3/05-neuralbridge/ch179-fda-510k-architecture.md) | NeuralBridge | 3 |
| 182 | [Edge Intelligence](./phase-3/05-neuralbridge/ch182-edge-intelligence.md) | NeuralBridge | 3 |

</details>

<a name="tag-mental-health"></a>

<details>
<summary><strong>#mental-health</strong> — 3 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 252 | [The Most Sensitive Data in the Room](./phase-3/15-mindscale/ch252-the-most-sensitive-data.md) | MindScale | 3 |
| 253 | [The Precision-Recall Dilemma](./phase-3/15-mindscale/ch253-crisis-detection-ml.md) | MindScale | 3 |
| 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | MindScale | 3 |

</details>

<a name="tag-messaging"></a>

<details>
<summary><strong>#messaging</strong> — 8 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 11 | [Untangle the Kafka Mess](./phase-1/03-velotrack/ch11-kafka-deep-dive-consumer-groups.md) | VeloTrack | 1 |
| 12 | [Production is DOWN — The Notification Consumer Won't Stop Crashing](./phase-1/03-velotrack/ch12-dead-letter-queues-and-error-handling.md) | VeloTrack | 1 |
| 114 | [Concert Cancellation Notification Cascade (4th Appearance)](./phase-2/08-venueflow/ch114-notifications-fourth-appearance.md) | VenueFlow | 2 |
| 150 | [Signal and Noise](./phase-3/01-orbitcore/ch150-signal-and-noise.md) | OrbitCore | 3 |
| 167 | [SPIKE — Scale 10x by Friday](./phase-3/03-carbonledger/ch167-scale-10x-spike.md) | CarbonLedger | 3 |
| 193 | [V2X at Scale](./phase-3/07-automesh/ch193-v2x-at-scale.md) | AutoMesh | 3 |
| 245 | [50 Million Events Per Second](./phase-3/14-novasports/ch245-50-million-events-per-second.md) | NovaSports | 3 |
| 262 | [Circuit Job Orchestration](./phase-3/16-quantumedge/ch262-circuit-job-orchestration.md) | QuantumEdge | 3 |

</details>

<a name="tag-metering"></a>

<details>
<summary><strong>#metering</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 31 | [Design the Tenant Billing and Usage Metering System](./phase-1/06-cloudstack/ch31-tenant-billing-and-usage-metering.md) | CloudStack | 1 |
| 56 | [Payment Infrastructure — Billing the ML Platform](./phase-1/11-luminaryai/ch56-payment-processing-third-appearance.md) | LuminaryAI | 1 |

</details>

<a name="tag-metrics"></a>

<details>
<summary><strong>#metrics</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 25 | [Build Observability Into the Sensor Platform](./phase-1/05-agrosense/ch25-observability-and-monitoring.md) | AgroSense | 1 |
| 57 | [Advanced Observability — When Models Lie Silently](./phase-1/11-luminaryai/ch57-advanced-observability-ml-systems.md) | LuminaryAI | 1 |

</details>

<a name="tag-ml-systems"></a>

<details>
<summary><strong>#ml-systems</strong> — 11 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 54 | [The Feature Store — Feeding Models the Right Data](./phase-1/11-luminaryai/ch54-feature-store-and-ml-pipeline.md) | LuminaryAI | 1 |
| 55 | [ETL to Streaming — Evolving the Data Pipeline](./phase-1/11-luminaryai/ch55-etl-to-streaming-pipeline-evolution.md) | LuminaryAI | 1 |
| 56 | [Payment Infrastructure — Billing the ML Platform](./phase-1/11-luminaryai/ch56-payment-processing-third-appearance.md) | LuminaryAI | 1 |
| 57 | [Advanced Observability — When Models Lie Silently](./phase-1/11-luminaryai/ch57-advanced-observability-ml-systems.md) | LuminaryAI | 1 |
| 58 | [SPIKE — Scale 10x by Friday](./phase-1/11-luminaryai/ch58-recommendation-serving-scale.md) | LuminaryAI | 1 |
| 115 | [Vector Search — Personalized Event Recommendations](./phase-2/08-venueflow/ch115-search-fourth-appearance.md) | VenueFlow | 2 |
| 204 | [The Webb Keynote](./phase-3/08-forgesense/ch204-the-webb-keynote.md) | ForgeSense | 3 |
| 218 | [Feature Stores for Drug Discovery](./phase-3/10-pharmasync/ch218-feature-stores-drug-discovery.md) | PharmaSync | 3 |
| 247 | [35 Million Opinions, Updated in Real Time](./phase-3/14-novasports/ch247-personalization-at-scale.md) | NovaSports | 3 |
| 253 | [The Precision-Recall Dilemma](./phase-3/15-mindscale/ch253-crisis-detection-ml.md) | MindScale | 3 |
| 263 | [Error Mitigation at Scale](./phase-3/16-quantumedge/ch263-error-mitigation-at-scale.md) | QuantumEdge | 3 |

</details>

<a name="tag-monitoring"></a>

<details>
<summary><strong>#monitoring</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 25 | [Build Observability Into the Sensor Platform](./phase-1/05-agrosense/ch25-observability-and-monitoring.md) | AgroSense | 1 |
| 57 | [Advanced Observability — When Models Lie Silently](./phase-1/11-luminaryai/ch57-advanced-observability-ml-systems.md) | LuminaryAI | 1 |
| 240 | [Inherited Disaster — The Data Quality Crisis](./phase-3/13-propiq/ch240-inherited-disaster-data-quality-crisis.md) | PropIQ | 3 |

</details>

<a name="tag-mtls"></a>

<details>
<summary><strong>#mtls</strong> — 5 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 83 | [Service Mesh and mTLS Introduction](./phase-2/03-tradespark/ch83-service-mesh-mtls-intro.md) | TradeSpark | 2 |
| 92 | [Zero-Trust Architecture](./phase-2/05-sentinelops/ch92-zero-trust-architecture.md) | SentinelOps | 2 |
| 93 | [Service Mesh Deep Dive — Istio in Production](./phase-2/05-sentinelops/ch93-service-mesh-deep-dive.md) | SentinelOps | 2 |
| 120 | [Service Mesh at Production Scale — 60 Services, 18 Months of Istio](./phase-2/09-telanova/ch120-service-mesh-production.md) | TeleNova | 2 |
| 139 | [mTLS for HIPAA — Every Service Call Is a PHI Flow](./phase-2/12-prismhealth/ch139-service-mesh-healthcare-compliance.md) | PrismHealth | 2 |

</details>

<a name="tag-multi-jurisdiction"></a>

<details>
<summary><strong>#multi-jurisdiction</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 192 | [State Regulation as Architecture](./phase-3/06-shieldmutual/ch192-state-regulation-as-architecture.md) | ShieldMutual | 3 |
| 214 | [Flag State to Port State](./phase-3/09-deepocean/ch214-maritime-compliance.md) | DeepOcean | 3 |

</details>

<a name="tag-multi-party"></a>

<details>
<summary><strong>#multi-party</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 174 | [The RFC That Failed](./phase-3/04-lexcore/ch174-the-rfc-that-failed.md) | LexCore | 3 |
| 241 | [The $2M Data Room](./phase-3/13-propiq/ch241-multi-party-data-rooms.md) | PropIQ | 3 |

</details>

<a name="tag-multi-region"></a>

<details>
<summary><strong>#multi-region</strong> — 8 chapters across 8 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 68 | [Multi-Region Active-Active Architecture](./phase-2/01-stratum-systems/ch68-multi-region-active-active-intro.md) | Stratum Systems | 2 |
| 73 | [Data Residency Across Multiple Jurisdictions](./phase-2/02-nexacare/ch73-data-residency-multi-jurisdiction.md) | NexaCare | 2 |
| 84 | [Multi-Region Conflict Resolution](./phase-2/04-giggrid/ch84-multi-region-conflict-resolution.md) | GigGrid | 2 |
| 125 | [Five Country Launches in One Quarter](./phase-2/09-telanova/ch125-multi-region-active-active-advanced.md) | TeleNova | 2 |
| 143 | [A German Patient's Heartbeat Cannot Leave Germany](./phase-2/12-prismhealth/ch143-multi-region-data-sovereignty-healthcare.md) | PrismHealth | 2 |
| 200 | [Five Nines for a Moving Vehicle](./phase-3/07-automesh/ch200-the-partnership-aftermath.md) | AutoMesh | 3 |
| 236 | [One Platform, Three Regulators](./phase-3/12-nexuswealth/ch236-data-sovereignty-pension-funds.md) | NexusWealth | 3 |
| 246 | [The World is Watching (And They're All in London)](./phase-3/14-novasports/ch246-multi-region-active-active.md) | NovaSports | 3 |

</details>

<a name="tag-multi-tenancy"></a>

<details>
<summary><strong>#multi-tenancy</strong> — 9 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 28 | [Multi-Tenant Architecture and Tenant Isolation](./phase-1/06-cloudstack/ch28-multi-tenancy-architecture.md) | CloudStack | 1 |
| 29 | [Design the API Gateway and Rate Limiting System (First Appearance)](./phase-1/06-cloudstack/ch29-api-gateway-and-rate-limiting.md) | CloudStack | 1 |
| 30 | [Auth at Scale — SSO, Service Accounts, and API Keys (Second Appearance)](./phase-1/06-cloudstack/ch30-auth-system-second-appearance.md) | CloudStack | 1 |
| 31 | [Design the Tenant Billing and Usage Metering System](./phase-1/06-cloudstack/ch31-tenant-billing-and-usage-metering.md) | CloudStack | 1 |
| 32 | [Solve the Noisy Neighbor Problem for Good](./phase-1/06-cloudstack/ch32-noisy-neighbor-and-resource-quotas.md) | CloudStack | 1 |
| 41 | [SSO for 23 State Governments — Federated Identity at Scale](./phase-1/08-civicos/ch41-sso-and-identity-federation-for-government.md) | CivicOS | 1 |
| 171 | [Privileged Information](./phase-3/04-lexcore/ch171-privileged-information.md) | LexCore | 3 |
| 208 | [The Twelve Factories](./phase-3/08-forgesense/ch208-factory-data-mesh.md) | ForgeSense | 3 |
| 227 | [One Account, Eight Hundred Farms](./phase-3/11-harvestai/ch227-multi-tenant-farm-management.md) | HarvestAI | 3 |

</details>

<a name="tag-multi-tenant"></a>

<details>
<summary><strong>#multi-tenant</strong> — 6 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 94 | [Advanced Authorization — RBAC vs ABAC and Policy-as-Code](./phase-2/05-sentinelops/ch94-auth-fourth-appearance.md) | SentinelOps | 2 |
| 176 | [Find the Precedent](./phase-3/04-lexcore/ch176-search-at-legal-scale.md) | LexCore | 3 |
| 189 | [SOC2 Under Pressure](./phase-3/06-shieldmutual/ch189-soc2-under-pressure.md) | ShieldMutual | 3 |
| 190 | [The Multi-Line Model](./phase-3/06-shieldmutual/ch190-the-multi-line-model.md) | ShieldMutual | 3 |
| 239 | [Search for Property Intelligence](./phase-3/13-propiq/ch239-search-for-property-intelligence.md) | PropIQ | 3 |
| 243 | [The Garbage Search](./phase-3/13-propiq/ch243-search-at-property-scale.md) | PropIQ | 3 |

</details>

<a name="tag-notifications"></a>

<details>
<summary><strong>#notifications</strong> — 5 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 20 | [Design the Notification System (First Appearance)](./phase-1/04-beacon-media/ch20-notification-system-first-appearance.md) | Beacon Media | 1 |
| 43 | [Notification System at Scale — The Finals Week Problem (Second Appearance)](./phase-1/09-neurolearn/ch43-notification-system-second-appearance.md) | NeuroLearn | 1 |
| 59 | [Notification System — 8 Million in 40 Minutes](./phase-1/12-skyroute/ch59-notification-system-third-appearance.md) | SkyRoute | 1 |
| 114 | [Concert Cancellation Notification Cascade (4th Appearance)](./phase-2/08-venueflow/ch114-notifications-fourth-appearance.md) | VenueFlow | 2 |
| 255 | [The Scarcest Resource](./phase-3/15-mindscale/ch255-the-notification-system-final-evolution.md) | MindScale | 3 |

</details>

<a name="tag-oauth2"></a>

<details>
<summary><strong>#oauth2</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 06 | [Design the Authentication and Authorization System](./phase-1/02-meridian-health/ch06-jwt-oauth2-auth-system.md) | MeridianHealth | 1 |
| 30 | [Auth at Scale — SSO, Service Accounts, and API Keys (Second Appearance)](./phase-1/06-cloudstack/ch30-auth-system-second-appearance.md) | CloudStack | 1 |
| 61 | [Auth for the Most Adversarial User Base](./phase-1/12-skyroute/ch61-auth-third-appearance.md) | SkyRoute | 1 |
| 123 | [Auth at 120 Million — Device Authentication and B2B2C Identity](./phase-2/09-telanova/ch123-auth-fifth-appearance.md) | TeleNova | 2 |

</details>

<a name="tag-observability"></a>

<details>
<summary><strong>#observability</strong> — 15 chapters across 13 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 07 | [Build the HIPAA-Compliant Audit Trail](./phase-1/02-meridian-health/ch07-hipaa-audit-trails.md) | MeridianHealth | 1 |
| 24 | [Build the ETL Pipeline and Change Data Capture](./phase-1/05-agrosense/ch24-etl-pipeline-and-cdc.md) | AgroSense | 1 |
| 25 | [Build Observability Into the Sensor Platform](./phase-1/05-agrosense/ch25-observability-and-monitoring.md) | AgroSense | 1 |
| 57 | [Advanced Observability — When Models Lie Silently](./phase-1/11-luminaryai/ch57-advanced-observability-ml-systems.md) | LuminaryAI | 1 |
| 69 | [Distributed Tracing with OpenTelemetry](./phase-2/01-stratum-systems/ch69-distributed-tracing-intro.md) | Stratum Systems | 2 |
| 74 | [Advanced Observability — SLOs, SLIs, and Error Budgets](./phase-2/02-nexacare/ch74-advanced-observability-slos.md) | NexaCare | 2 |
| 93 | [Service Mesh Deep Dive — Istio in Production](./phase-2/05-sentinelops/ch93-service-mesh-deep-dive.md) | SentinelOps | 2 |
| 96 | [Advanced Observability — Distributed Tracing at 50M Spans/Day](./phase-2/05-sentinelops/ch96-advanced-observability-distributed-tracing.md) | SentinelOps | 2 |
| 121 | [200 Million Spans Per Day — Tail Sampling and Cost Control](./phase-2/09-telanova/ch121-distributed-tracing-opentelemetry.md) | TeleNova | 2 |
| 140 | [Life-Safety Alerting — When False Positives Kill Trust](./phase-2/12-prismhealth/ch140-advanced-observability-alerting.md) | PrismHealth | 2 |
| 187 | [The Actuarial Pipeline](./phase-3/06-shieldmutual/ch187-the-actuarial-pipeline.md) | ShieldMutual | 3 |
| 204 | [The Webb Keynote](./phase-3/08-forgesense/ch204-the-webb-keynote.md) | ForgeSense | 3 |
| 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | HarvestAI | 3 |
| 248 | [Seven Seconds to National Television](./phase-3/14-novasports/ch248-real-time-sports-analytics.md) | NovaSports | 3 |
| 264 | [Hybrid Pipelines — Chemistry Simulation](./phase-3/16-quantumedge/ch264-hybrid-pipelines-chemistry-simulation.md) | QuantumEdge | 3 |

</details>

<a name="tag-offline"></a>

<details>
<summary><strong>#offline</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 99 | [POS Terminal Payments and P2PE](./phase-2/06-lightspeedretail/ch99-payments-fifth-appearance.md) | LightspeedRetail | 2 |
| 212 | [2,000 Miles from Anywhere](./phase-3/09-deepocean/ch212-edge-processing-at-sea.md) | DeepOcean | 3 |

</details>

<a name="tag-offline-first"></a>

<details>
<summary><strong>#offline-first</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 210 | [Three Weeks at Sea](./phase-3/09-deepocean/ch210-offline-first-for-ships.md) | DeepOcean | 3 |
| 213 | [Designing for the Speed of Light](./phase-3/09-deepocean/ch213-extreme-latency-architecture.md) | DeepOcean | 3 |

</details>

<a name="tag-olap"></a>

<details>
<summary><strong>#olap</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 103 | [Data Warehousing — OLAP vs OLTP](./phase-2/06-lightspeedretail/ch103-data-warehousing-olap-vs-oltp.md) | LightspeedRetail | 2 |
| 107 | [Grid Analytics — Apache Iceberg and Columnar Storage](./phase-2/07-crestline-energy/ch107-data-warehousing-columnar-storage.md) | Crestline Energy | 2 |

</details>

<a name="tag-opc-ua"></a>

<details>
<summary><strong>#opc-ua</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 203 | [OPC-UA to Cloud](./phase-3/08-forgesense/ch203-opc-ua-to-cloud.md) | ForgeSense | 3 |
| 206 | [What Do You Mean, Read-Only?](./phase-3/08-forgesense/ch206-design-review-ambush-brownfield.md) | ForgeSense | 3 |

</details>

<a name="tag-opentelemetry"></a>

<details>
<summary><strong>#opentelemetry</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 69 | [Distributed Tracing with OpenTelemetry](./phase-2/01-stratum-systems/ch69-distributed-tracing-intro.md) | Stratum Systems | 2 |
| 96 | [Advanced Observability — Distributed Tracing at 50M Spans/Day](./phase-2/05-sentinelops/ch96-advanced-observability-distributed-tracing.md) | SentinelOps | 2 |
| 121 | [200 Million Spans Per Day — Tail Sampling and Cost Control](./phase-2/09-telanova/ch121-distributed-tracing-opentelemetry.md) | TeleNova | 2 |

</details>

<a name="tag-optimization"></a>

<details>
<summary><strong>#optimization</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 35 | [The N+1 Query Disaster Eating Your Database](./phase-1/07-pulsecommerce/ch35-n-plus-one-queries-and-dataloader.md) | PulseCommerce | 1 |
| 212 | [2,000 Miles from Anywhere](./phase-3/09-deepocean/ch212-edge-processing-at-sea.md) | DeepOcean | 3 |
| 270 | [The Demand Cliff](./phase-3/17-glaciergrid/ch270-real-time-grid-stability.md) | GlacierGrid | 3 |

</details>

<a name="tag-ot-it-convergence"></a>

<details>
<summary><strong>#ot-it-convergence</strong> — 3 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 205 | [The Ghost in the Machine](./phase-3/08-forgesense/ch205-inherited-disaster-scada.md) | ForgeSense | 3 |
| 206 | [What Do You Mean, Read-Only?](./phase-3/08-forgesense/ch206-design-review-ambush-brownfield.md) | ForgeSense | 3 |
| 207 | [The 200ms Window](./phase-3/08-forgesense/ch207-real-time-control-layer.md) | ForgeSense | 3 |

</details>

<a name="tag-ota"></a>

<details>
<summary><strong>#ota</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 195 | [OTA at Fleet Scale](./phase-3/07-automesh/ch195-ota-at-fleet-scale.md) | AutoMesh | 3 |
| 197 | [The Fractured Map](./phase-3/07-automesh/ch197-hd-map-distribution.md) | AutoMesh | 3 |
| 225 | [The Model in the Field](./phase-3/11-harvestai/ch225-edge-ml-inference.md) | HarvestAI | 3 |

</details>

<a name="tag-parquet"></a>

<details>
<summary><strong>#parquet</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 103 | [Data Warehousing — OLAP vs OLTP](./phase-2/06-lightspeedretail/ch103-data-warehousing-olap-vs-oltp.md) | LightspeedRetail | 2 |
| 107 | [Grid Analytics — Apache Iceberg and Columnar Storage](./phase-2/07-crestline-energy/ch107-data-warehousing-columnar-storage.md) | Crestline Energy | 2 |

</details>

<a name="tag-partitioning"></a>

<details>
<summary><strong>#partitioning</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 79 | [Advanced Kafka — Log Compaction for Order Book State](./phase-2/03-tradespark/ch79-advanced-kafka-log-compaction.md) | TradeSpark | 2 |
| 98 | [Kafka at Scale — 40 Billion Security Events/Day](./phase-2/05-sentinelops/ch98-advanced-kafka-security-events.md) | SentinelOps | 2 |
| 209 | [The Ocean's Heartbeat](./phase-3/09-deepocean/ch209-ais-the-oceans-heartbeat.md) | DeepOcean | 3 |

</details>

<a name="tag-payments"></a>

<details>
<summary><strong>#payments</strong> — 9 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 01 | [Design the Payment Processing Pipeline](./phase-1/01-novapay/ch01-payment-processing-pipeline.md) | NovaPay | 1 |
| 02 | [Production is DOWN — Duplicate Charges at Scale](./phase-1/01-novapay/ch02-idempotency-and-duplicate-prevention.md) | NovaPay | 1 |
| 03 | [The Slow Query Nobody Wrote a Ticket For](./phase-1/01-novapay/ch03-database-indexing-for-financial-queries.md) | NovaPay | 1 |
| 04 | [Design the Merchant Balance Cache](./phase-1/01-novapay/ch04-write-through-cache-design.md) | NovaPay | 1 |
| 05 | [Design Review Ambush — PCI-DSS or the Audit Fails](./phase-1/01-novapay/ch05-pci-dss-compliance-architecture.md) | NovaPay | 1 |
| 34 | [Payment Integration at Scale — Scaling Challenges (Second Appearance)](./phase-1/07-pulsecommerce/ch34-payment-processing-second-appearance.md) | PulseCommerce | 1 |
| 56 | [Payment Infrastructure — Billing the ML Platform](./phase-1/11-luminaryai/ch56-payment-processing-third-appearance.md) | LuminaryAI | 1 |
| 81 | [DTCC Clearing Integration and Financial Settlement](./phase-2/03-tradespark/ch81-payments-fourth-appearance.md) | TradeSpark | 2 |
| 99 | [POS Terminal Payments and P2PE](./phase-2/06-lightspeedretail/ch99-payments-fifth-appearance.md) | LightspeedRetail | 2 |

</details>

<a name="tag-pci-dss"></a>

<details>
<summary><strong>#pci-dss</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 05 | [Design Review Ambush — PCI-DSS or the Audit Fails](./phase-1/01-novapay/ch05-pci-dss-compliance-architecture.md) | NovaPay | 1 |
| 56 | [Payment Infrastructure — Billing the ML Platform](./phase-1/11-luminaryai/ch56-payment-processing-third-appearance.md) | LuminaryAI | 1 |
| 81 | [DTCC Clearing Integration and Financial Settlement](./phase-2/03-tradespark/ch81-payments-fourth-appearance.md) | TradeSpark | 2 |
| 99 | [POS Terminal Payments and P2PE](./phase-2/06-lightspeedretail/ch99-payments-fifth-appearance.md) | LightspeedRetail | 2 |

</details>

<a name="tag-performance"></a>

<details>
<summary><strong>#performance</strong> — 15 chapters across 13 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 03 | [The Slow Query Nobody Wrote a Ticket For](./phase-1/01-novapay/ch03-database-indexing-for-financial-queries.md) | NovaPay | 1 |
| 09 | [Scale the EHR Query Layer with Read Replicas](./phase-1/02-meridian-health/ch09-read-replicas-and-query-routing.md) | MeridianHealth | 1 |
| 14 | [Design the Load Balancing Strategy for a Heterogeneous Fleet](./phase-1/03-velotrack/ch14-load-balancing-strategies.md) | VeloTrack | 1 |
| 18 | [Fix the CDN — $2M/Month and Falling](./phase-1/04-beacon-media/ch18-blob-storage-and-cdn-architecture.md) | Beacon Media | 1 |
| 22 | [The Recommendation API is Melting](./phase-1/04-beacon-media/ch22-content-recommendation-and-caching.md) | Beacon Media | 1 |
| 23 | [Design the Time-Series Sensor Data Platform](./phase-1/05-agrosense/ch23-time-series-data-design.md) | AgroSense | 1 |
| 32 | [Solve the Noisy Neighbor Problem for Good](./phase-1/06-cloudstack/ch32-noisy-neighbor-and-resource-quotas.md) | CloudStack | 1 |
| 35 | [The N+1 Query Disaster Eating Your Database](./phase-1/07-pulsecommerce/ch35-n-plus-one-queries-and-dataloader.md) | PulseCommerce | 1 |
| 44 | [Database Collapse During Finals Week — Read Replicas at Scale](./phase-1/09-neurolearn/ch44-read-replicas-and-connection-pooling-second-appearance.md) | NeuroLearn | 1 |
| 78 | [Database Internals — LSM-Tree Compaction Storm](./phase-2/03-tradespark/ch78-database-internals-deep-dive.md) | TradeSpark | 2 |
| 98 | [Kafka at Scale — 40 Billion Security Events/Day](./phase-2/05-sentinelops/ch98-advanced-kafka-security-events.md) | SentinelOps | 2 |
| 153 | [The Latency Ladder](./phase-3/01-orbitcore/ch153-the-latency-ladder.md) | OrbitCore | 3 |
| 223 | [When the Earth Won't Fit in a B-Tree](./phase-3/11-harvestai/ch223-geospatial-at-scale.md) | HarvestAI | 3 |
| 262 | [Circuit Job Orchestration](./phase-3/16-quantumedge/ch262-circuit-job-orchestration.md) | QuantumEdge | 3 |
| 263 | [Error Mitigation at Scale](./phase-3/16-quantumedge/ch263-error-mitigation-at-scale.md) | QuantumEdge | 3 |

</details>

<a name="tag-pipeline"></a>

<details>
<summary><strong>#pipeline</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 172 | [The Chunking Pipeline](./phase-3/04-lexcore/ch172-the-chunking-pipeline.md) | LexCore | 3 |
| 262 | [Circuit Job Orchestration](./phase-3/16-quantumedge/ch262-circuit-job-orchestration.md) | QuantumEdge | 3 |
| 264 | [Hybrid Pipelines — Chemistry Simulation](./phase-3/16-quantumedge/ch264-hybrid-pipelines-chemistry-simulation.md) | QuantumEdge | 3 |

</details>

<a name="tag-postgres"></a>

<details>
<summary><strong>#postgres</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 67 | [Database Internals — WAL and MVCC](./phase-2/01-stratum-systems/ch67-database-internals-wal-and-mvcc.md) | Stratum Systems | 2 |
| 141 | [The Vacuum That Wasn't Running](./phase-2/12-prismhealth/ch141-database-internals-postgres-tuning.md) | PrismHealth | 2 |

</details>

<a name="tag-postmortem"></a>

<details>
<summary><strong>#postmortem</strong> — 3 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | HarvestAI | 3 |
| 249 | [The Game Day That Went Wrong](./phase-3/14-novasports/ch249-the-game-day-that-went-wrong.md) | NovaSports | 3 |
| 250 | [The Postmortem (And the Architecture We Should Have Built)](./phase-3/14-novasports/ch250-blameless-postmortem-and-redesign.md) | NovaSports | 3 |

</details>

<a name="tag-privacy"></a>

<details>
<summary><strong>#privacy</strong> — 6 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 47 | [Design Review Ambush — The Academic Integrity Architecture](./phase-1/09-neurolearn/ch47-academic-integrity-monitoring.md) | NeuroLearn | 1 |
| 70 | [GDPR Deletion Pipelines](./phase-2/02-nexacare/ch70-gdpr-deletion-pipelines.md) | NexaCare | 2 |
| 133 | [Genomic Data Is Personal — Differential Privacy and Federated Computation](./phase-2/11-axiom-labs/ch133-privacy-preserving-systems.md) | Axiom Labs | 2 |
| 180 | [Brain Data Privacy](./phase-3/05-neuralbridge/ch180-brain-data-privacy.md) | NeuralBridge | 3 |
| 252 | [The Most Sensitive Data in the Room](./phase-3/15-mindscale/ch252-the-most-sensitive-data.md) | MindScale | 3 |
| 256 | [The 23 Percent](./phase-3/15-mindscale/ch256-re-identification-risk-mitigation.md) | MindScale | 3 |

</details>

<a name="tag-production-down"></a>

<details>
<summary><strong>#production-down</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 156 | [SPIKE — Solar Flare](./phase-3/01-orbitcore/ch156-solar-flare-spike.md) | OrbitCore | 3 |
| 269 | [Three ISOs, One Morning](./phase-3/17-glaciergrid/ch269-production-down-grid-cascade.md) | GlacierGrid | 3 |

</details>

<a name="tag-progressive-delivery"></a>

<details>
<summary><strong>#progressive-delivery</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 129 | [Progressive Delivery with Argo Rollouts](./phase-2/10-buildright/ch129-canary-deployments-advanced.md) | BuildRight | 2 |
| 147 | [GitOps at 2,000 Engineers](./phase-2/13-vertexcloud/ch147-advanced-deployment-pipeline.md) | VertexCloud | 2 |

</details>

<a name="tag-projections"></a>

<details>
<summary><strong>#projections</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 66 | [CQRS and Event Sourcing](./phase-2/01-stratum-systems/ch66-cqrs-and-event-sourcing.md) | Stratum Systems | 2 |
| 104 | [CQRS Advanced Projections — Real-Time Inventory](./phase-2/06-lightspeedretail/ch104-cqrs-advanced-projections.md) | LightspeedRetail | 2 |

</details>

<a name="tag-pubsub"></a>

<details>
<summary><strong>#pubsub</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 86 | [Real-Time at Scale — WebSockets, SSE, and Pub/Sub](./phase-2/04-giggrid/ch86-real-time-at-scale-websockets-sse.md) | GigGrid | 2 |
| 113 | [Real-Time at Scale — 500k Concurrent Fans](./phase-2/08-venueflow/ch113-real-time-at-scale-pub-sub.md) | VenueFlow | 2 |

</details>

<a name="tag-push"></a>

<details>
<summary><strong>#push</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 20 | [Design the Notification System (First Appearance)](./phase-1/04-beacon-media/ch20-notification-system-first-appearance.md) | Beacon Media | 1 |
| 43 | [Notification System at Scale — The Finals Week Problem (Second Appearance)](./phase-1/09-neurolearn/ch43-notification-system-second-appearance.md) | NeuroLearn | 1 |

</details>

<a name="tag-rate-limiting"></a>

<details>
<summary><strong>#rate-limiting</strong> — 9 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 29 | [Design the API Gateway and Rate Limiting System (First Appearance)](./phase-1/06-cloudstack/ch29-api-gateway-and-rate-limiting.md) | CloudStack | 1 |
| 40 | [Rate Limiting for Government APIs — The Enumeration Attack](./phase-1/08-civicos/ch40-rate-limiting-second-appearance.md) | CivicOS | 1 |
| 59 | [Notification System — 8 Million in 40 Minutes](./phase-1/12-skyroute/ch59-notification-system-third-appearance.md) | SkyRoute | 1 |
| 62 | [Rate Limiting — Defending Against the GDS Giants](./phase-1/12-skyroute/ch62-rate-limiting-third-appearance.md) | SkyRoute | 1 |
| 87 | [Platform API Rate Limiting (4th Appearance)](./phase-2/04-giggrid/ch87-rate-limiting-fourth-appearance.md) | GigGrid | 2 |
| 114 | [Concert Cancellation Notification Cascade (4th Appearance)](./phase-2/08-venueflow/ch114-notifications-fourth-appearance.md) | VenueFlow | 2 |
| 118 | [Adaptive Rate Limiting and Ticket Bot DDoS](./phase-2/08-venueflow/ch118-advanced-rate-limiting-ddos.md) | VenueFlow | 2 |
| 124 | [5G QoS — Fair Use Without Unfair Enforcement](./phase-2/09-telanova/ch124-network-rate-limiting-qos.md) | TeleNova | 2 |
| 261 | [The Quantum API Gateway and Auth (Auth System: 6th Evolution)](./phase-3/16-quantumedge/ch261-quantum-api-gateway-and-auth.md) | QuantumEdge | 3 |

</details>

<a name="tag-rbac"></a>

<details>
<summary><strong>#rbac</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 94 | [Advanced Authorization — RBAC vs ABAC and Policy-as-Code](./phase-2/05-sentinelops/ch94-auth-fourth-appearance.md) | SentinelOps | 2 |
| 227 | [One Account, Eight Hundred Farms](./phase-3/11-harvestai/ch227-multi-tenant-farm-management.md) | HarvestAI | 3 |

</details>

<a name="tag-read-replicas"></a>

<details>
<summary><strong>#read-replicas</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 09 | [Scale the EHR Query Layer with Read Replicas](./phase-1/02-meridian-health/ch09-read-replicas-and-query-routing.md) | MeridianHealth | 1 |
| 44 | [Database Collapse During Finals Week — Read Replicas at Scale](./phase-1/09-neurolearn/ch44-read-replicas-and-connection-pooling-second-appearance.md) | NeuroLearn | 1 |

</details>

<a name="tag-real-estate"></a>

<details>
<summary><strong>#real-estate</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 238 | [Twelve Sources, One Truth](./phase-3/13-propiq/ch238-the-property-data-mesh.md) | PropIQ | 3 |
| 239 | [Search for Property Intelligence](./phase-3/13-propiq/ch239-search-for-property-intelligence.md) | PropIQ | 3 |

</details>

<a name="tag-real-time"></a>

<details>
<summary><strong>#real-time</strong> — 17 chapters across 11 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 11 | [Untangle the Kafka Mess](./phase-1/03-velotrack/ch11-kafka-deep-dive-consumer-groups.md) | VeloTrack | 1 |
| 13 | [Design the Real-Time Driver Tracking System](./phase-1/03-velotrack/ch13-real-time-delivery-tracking.md) | VeloTrack | 1 |
| 46 | [Design the Adaptive Quiz Engine at Scale](./phase-1/09-neurolearn/ch46-adaptive-quiz-engine-design.md) | NeuroLearn | 1 |
| 47 | [Design Review Ambush — The Academic Integrity Architecture](./phase-1/09-neurolearn/ch47-academic-integrity-monitoring.md) | NeuroLearn | 1 |
| 153 | [The Latency Ladder](./phase-3/01-orbitcore/ch153-the-latency-ladder.md) | OrbitCore | 3 |
| 178 | [The Signal](./phase-3/05-neuralbridge/ch178-the-signal.md) | NeuralBridge | 3 |
| 186 | [The Risk Engine](./phase-3/06-shieldmutual/ch186-the-risk-engine.md) | ShieldMutual | 3 |
| 193 | [V2X at Scale](./phase-3/07-automesh/ch193-v2x-at-scale.md) | AutoMesh | 3 |
| 202 | [The Digital Twin](./phase-3/08-forgesense/ch202-the-digital-twin.md) | ForgeSense | 3 |
| 207 | [The 200ms Window](./phase-3/08-forgesense/ch207-real-time-control-layer.md) | ForgeSense | 3 |
| 233 | [Ten Minutes to Exposure](./phase-3/12-nexuswealth/ch233-real-time-portfolio-risk.md) | NexusWealth | 3 |
| 242 | [The Real-Time Lie](./phase-3/13-propiq/ch242-real-time-market-aggregation.md) | PropIQ | 3 |
| 247 | [35 Million Opinions, Updated in Real Time](./phase-3/14-novasports/ch247-personalization-at-scale.md) | NovaSports | 3 |
| 248 | [Seven Seconds to National Television](./phase-3/14-novasports/ch248-real-time-sports-analytics.md) | NovaSports | 3 |
| 266 | [The Virtual Power Plant](./phase-3/17-glaciergrid/ch266-the-virtual-power-plant.md) | GlacierGrid | 3 |
| 267 | [The 500ms Problem](./phase-3/17-glaciergrid/ch267-grid-edge-intelligence.md) | GlacierGrid | 3 |
| 270 | [The Demand Cliff](./phase-3/17-glaciergrid/ch270-real-time-grid-stability.md) | GlacierGrid | 3 |

</details>

<a name="tag-realtime"></a>

<details>
<summary><strong>#realtime</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 86 | [Real-Time at Scale — WebSockets, SSE, and Pub/Sub](./phase-2/04-giggrid/ch86-real-time-at-scale-websockets-sse.md) | GigGrid | 2 |
| 113 | [Real-Time at Scale — 500k Concurrent Fans](./phase-2/08-venueflow/ch113-real-time-at-scale-pub-sub.md) | VenueFlow | 2 |
| 245 | [50 Million Events Per Second](./phase-3/14-novasports/ch245-50-million-events-per-second.md) | NovaSports | 3 |

</details>

<a name="tag-redis"></a>

<details>
<summary><strong>#redis</strong> — 11 chapters across 10 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 04 | [Design the Merchant Balance Cache](./phase-1/01-novapay/ch04-write-through-cache-design.md) | NovaPay | 1 |
| 13 | [Design the Real-Time Driver Tracking System](./phase-1/03-velotrack/ch13-real-time-delivery-tracking.md) | VeloTrack | 1 |
| 29 | [Design the API Gateway and Rate Limiting System (First Appearance)](./phase-1/06-cloudstack/ch29-api-gateway-and-rate-limiting.md) | CloudStack | 1 |
| 36 | [Cache Invalidation Hell](./phase-1/07-pulsecommerce/ch36-cache-invalidation-patterns.md) | PulseCommerce | 1 |
| 40 | [Rate Limiting for Government APIs — The Enumeration Attack](./phase-1/08-civicos/ch40-rate-limiting-second-appearance.md) | CivicOS | 1 |
| 62 | [Rate Limiting — Defending Against the GDS Giants](./phase-1/12-skyroute/ch62-rate-limiting-third-appearance.md) | SkyRoute | 1 |
| 82 | [Advanced Caching — Probabilistic Early Expiration](./phase-2/03-tradespark/ch82-advanced-caching-probabilistic-expiration.md) | TradeSpark | 2 |
| 86 | [Real-Time at Scale — WebSockets, SSE, and Pub/Sub](./phase-2/04-giggrid/ch86-real-time-at-scale-websockets-sse.md) | GigGrid | 2 |
| 87 | [Platform API Rate Limiting (4th Appearance)](./phase-2/04-giggrid/ch87-rate-limiting-fourth-appearance.md) | GigGrid | 2 |
| 100 | [Advanced Caching — Black Friday Thundering Herd](./phase-2/06-lightspeedretail/ch100-advanced-caching-thundering-herd.md) | LightspeedRetail | 2 |
| 113 | [Real-Time at Scale — 500k Concurrent Fans](./phase-2/08-venueflow/ch113-real-time-at-scale-pub-sub.md) | VenueFlow | 2 |

</details>

<a name="tag-regulatory"></a>

<details>
<summary><strong>#regulatory</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 192 | [State Regulation as Architecture](./phase-3/06-shieldmutual/ch192-state-regulation-as-architecture.md) | ShieldMutual | 3 |
| 214 | [Flag State to Port State](./phase-3/09-deepocean/ch214-maritime-compliance.md) | DeepOcean | 3 |

</details>

<a name="tag-relevance"></a>

<details>
<summary><strong>#relevance</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 33 | [Design the Product Search System (First Appearance)](./phase-1/07-pulsecommerce/ch33-search-inverted-index-elasticsearch.md) | PulseCommerce | 1 |
| 45 | [Search Over Academic Content — Scaling Challenges (Second Appearance)](./phase-1/09-neurolearn/ch45-search-second-appearance.md) | NeuroLearn | 1 |
| 89 | [Advanced Search — Facets and Typeahead](./phase-2/04-giggrid/ch89-advanced-search-facets-typeahead.md) | GigGrid | 2 |

</details>

<a name="tag-reliability"></a>

<details>
<summary><strong>#reliability</strong> — 6 chapters across 6 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 26 | [Design for Unreliable Sensors at the Edge](./phase-1/05-agrosense/ch26-edge-processing-and-sensor-reliability.md) | AgroSense | 1 |
| 109 | [Grid Reliability SLOs — When 99.9% Is Not Good Enough](./phase-2/07-crestline-energy/ch109-slo-sli-error-budget-design.md) | Crestline Energy | 2 |
| 199 | [Every Way This Can Kill Someone](./phase-3/07-automesh/ch199-fmea-driven-architecture.md) | AutoMesh | 3 |
| 203 | [OPC-UA to Cloud](./phase-3/08-forgesense/ch203-opc-ua-to-cloud.md) | ForgeSense | 3 |
| 228 | [The Harvest Postmortem](./phase-3/11-harvestai/ch228-the-harvest-postmortem.md) | HarvestAI | 3 |
| 264 | [Hybrid Pipelines — Chemistry Simulation](./phase-3/16-quantumedge/ch264-hybrid-pipelines-chemistry-simulation.md) | QuantumEdge | 3 |

</details>

<a name="tag-resilience"></a>

<details>
<summary><strong>#resilience</strong> — 4 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 42 | [Design for Accessibility, Resilience, and the Citizen Who Can't Retry](./phase-1/08-civicos/ch42-accessibility-and-resilience-for-government-systems.md) | CivicOS | 1 |
| 149 | [The Game Day That Became a Real Incident](./phase-2/13-vertexcloud/ch149-chaos-engineering-intro.md) | VertexCloud | 2 |
| 151 | [The Cell Problem](./phase-3/01-orbitcore/ch151-the-cell-problem.md) | OrbitCore | 3 |
| 156 | [SPIKE — Solar Flare](./phase-3/01-orbitcore/ch156-solar-flare-spike.md) | OrbitCore | 3 |

</details>

<a name="tag-rocksdb"></a>

<details>
<summary><strong>#rocksdb</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 71 | [Database Internals — B-Tree vs LSM-Tree](./phase-2/02-nexacare/ch71-database-internals-btree-vs-lsm.md) | NexaCare | 2 |
| 78 | [Database Internals — LSM-Tree Compaction Storm](./phase-2/03-tradespark/ch78-database-internals-deep-dive.md) | TradeSpark | 2 |

</details>

<a name="tag-rollback"></a>

<details>
<summary><strong>#rollback</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 88 | [Canary Deployments and Feature Flags](./phase-2/04-giggrid/ch88-canary-deployments-and-feature-flags.md) | GigGrid | 2 |
| 102 | [Zero-Downtime Deploys for 120,000 POS Terminals](./phase-2/06-lightspeedretail/ch102-blue-green-deployments.md) | LightspeedRetail | 2 |

</details>

<a name="tag-safety-critical"></a>

<details>
<summary><strong>#safety-critical</strong> — 6 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 193 | [V2X at Scale](./phase-3/07-automesh/ch193-v2x-at-scale.md) | AutoMesh | 3 |
| 194 | [The Safety-Critical Edge](./phase-3/07-automesh/ch194-the-safety-critical-edge.md) | AutoMesh | 3 |
| 195 | [OTA at Fleet Scale](./phase-3/07-automesh/ch195-ota-at-fleet-scale.md) | AutoMesh | 3 |
| 197 | [The Fractured Map](./phase-3/07-automesh/ch197-hd-map-distribution.md) | AutoMesh | 3 |
| 199 | [Every Way This Can Kill Someone](./phase-3/07-automesh/ch199-fmea-driven-architecture.md) | AutoMesh | 3 |
| 207 | [The 200ms Window](./phase-3/08-forgesense/ch207-real-time-control-layer.md) | ForgeSense | 3 |

</details>

<a name="tag-saga"></a>

<details>
<summary><strong>#saga</strong> — 5 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 50 | [Design the Multi-Step Supply Chain Transaction with Saga](./phase-1/10-omnilogix/ch50-distributed-transactions-and-saga-pattern.md) | OmniLogix | 1 |
| 53 | [Global Inventory Consistency — The Full Design](./phase-1/10-omnilogix/ch53-global-inventory-consistency-synthesis.md) | OmniLogix | 1 |
| 63 | [Seat Inventory — The Overbooking Problem](./phase-1/12-skyroute/ch63-seat-inventory-and-overbooking.md) | SkyRoute | 1 |
| 80 | [Distributed Transactions — 2PC vs Saga](./phase-2/03-tradespark/ch80-distributed-transactions-2pc-vs-saga.md) | TradeSpark | 2 |
| 116 | [Ticket Inventory Consistency — The Taylor Swift Problem](./phase-2/08-venueflow/ch116-ticket-inventory-consistency.md) | VenueFlow | 2 |

</details>

<a name="tag-sampling"></a>

<details>
<summary><strong>#sampling</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 69 | [Distributed Tracing with OpenTelemetry](./phase-2/01-stratum-systems/ch69-distributed-tracing-intro.md) | Stratum Systems | 2 |
| 96 | [Advanced Observability — Distributed Tracing at 50M Spans/Day](./phase-2/05-sentinelops/ch96-advanced-observability-distributed-tracing.md) | SentinelOps | 2 |
| 121 | [200 Million Spans Per Day — Tail Sampling and Cost Control](./phase-2/09-telanova/ch121-distributed-tracing-opentelemetry.md) | TeleNova | 2 |

</details>

<a name="tag-scaling"></a>

<details>
<summary><strong>#scaling</strong> — 12 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 10 | [Scale 10x by Friday — The Northview Surge](./phase-1/02-meridian-health/ch10-ehr-integration-platform-design.md) | MeridianHealth | 1 |
| 21 | [Scale 10x — The New Content Deal is Going to Break Transcoding](./phase-1/04-beacon-media/ch21-video-transcoding-pipeline.md) | Beacon Media | 1 |
| 22 | [The Recommendation API is Melting](./phase-1/04-beacon-media/ch22-content-recommendation-and-caching.md) | Beacon Media | 1 |
| 33 | [Design the Product Search System (First Appearance)](./phase-1/07-pulsecommerce/ch33-search-inverted-index-elasticsearch.md) | PulseCommerce | 1 |
| 34 | [Payment Integration at Scale — Scaling Challenges (Second Appearance)](./phase-1/07-pulsecommerce/ch34-payment-processing-second-appearance.md) | PulseCommerce | 1 |
| 37 | [Flash Sale — Inventory Overselling Under Pressure](./phase-1/07-pulsecommerce/ch37-inventory-management-and-consistency.md) | PulseCommerce | 1 |
| 43 | [Notification System at Scale — The Finals Week Problem (Second Appearance)](./phase-1/09-neurolearn/ch43-notification-system-second-appearance.md) | NeuroLearn | 1 |
| 44 | [Database Collapse During Finals Week — Read Replicas at Scale](./phase-1/09-neurolearn/ch44-read-replicas-and-connection-pooling-second-appearance.md) | NeuroLearn | 1 |
| 45 | [Search Over Academic Content — Scaling Challenges (Second Appearance)](./phase-1/09-neurolearn/ch45-search-second-appearance.md) | NeuroLearn | 1 |
| 58 | [SPIKE — Scale 10x by Friday](./phase-1/11-luminaryai/ch58-recommendation-serving-scale.md) | LuminaryAI | 1 |
| 155 | [Capacity at Constellation Scale](./phase-3/01-orbitcore/ch155-capacity-at-constellation-scale.md) | OrbitCore | 3 |
| 196 | [Scale 10x — Fleet Doubles](./phase-3/07-automesh/ch196-scale-10x-fleet-doubles-spike.md) | AutoMesh | 3 |

</details>

<a name="tag-scheduling"></a>

<details>
<summary><strong>#scheduling</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 260 | [The Hybrid Architecture](./phase-3/16-quantumedge/ch260-the-hybrid-architecture.md) | QuantumEdge | 3 |
| 262 | [Circuit Job Orchestration](./phase-3/16-quantumedge/ch262-circuit-job-orchestration.md) | QuantumEdge | 3 |

</details>

<a name="tag-schema-evolution"></a>

<details>
<summary><strong>#schema-evolution</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 130 | [Data Pipeline Schema Evolution](./phase-2/10-buildright/ch130-data-pipeline-schema-evolution.md) | BuildRight | 2 |
| 190 | [The Multi-Line Model](./phase-3/06-shieldmutual/ch190-the-multi-line-model.md) | ShieldMutual | 3 |

</details>

<a name="tag-search"></a>

<details>
<summary><strong>#search</strong> — 10 chapters across 8 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 33 | [Design the Product Search System (First Appearance)](./phase-1/07-pulsecommerce/ch33-search-inverted-index-elasticsearch.md) | PulseCommerce | 1 |
| 45 | [Search Over Academic Content — Scaling Challenges (Second Appearance)](./phase-1/09-neurolearn/ch45-search-second-appearance.md) | NeuroLearn | 1 |
| 60 | [Flight Search — The Hardest Search Problem](./phase-1/12-skyroute/ch60-flight-search-third-appearance.md) | SkyRoute | 1 |
| 89 | [Advanced Search — Facets and Typeahead](./phase-2/04-giggrid/ch89-advanced-search-facets-typeahead.md) | GigGrid | 2 |
| 115 | [Vector Search — Personalized Event Recommendations](./phase-2/08-venueflow/ch115-search-fourth-appearance.md) | VenueFlow | 2 |
| 127 | [Advanced Search — Material Catalog and Vendor Discovery](./phase-2/10-buildright/ch127-advanced-search-typeahead-facets.md) | BuildRight | 2 |
| 176 | [Find the Precedent](./phase-3/04-lexcore/ch176-search-at-legal-scale.md) | LexCore | 3 |
| 177 | [The Privilege Log](./phase-3/04-lexcore/ch177-the-privilege-log.md) | LexCore | 3 |
| 239 | [Search for Property Intelligence](./phase-3/13-propiq/ch239-search-for-property-intelligence.md) | PropIQ | 3 |
| 243 | [The Garbage Search](./phase-3/13-propiq/ch243-search-at-property-scale.md) | PropIQ | 3 |

</details>

<a name="tag-secret-management"></a>

<details>
<summary><strong>#secret-management</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 75 | [Secret Management with HashiCorp Vault](./phase-2/02-nexacare/ch75-secret-management-intro.md) | NexaCare | 2 |
| 95 | [Secret Management at Scale](./phase-2/05-sentinelops/ch95-secret-management-at-scale.md) | SentinelOps | 2 |

</details>

<a name="tag-security"></a>

<details>
<summary><strong>#security</strong> — 25 chapters across 17 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 05 | [Design Review Ambush — PCI-DSS or the Audit Fails](./phase-1/01-novapay/ch05-pci-dss-compliance-architecture.md) | NovaPay | 1 |
| 06 | [Design the Authentication and Authorization System](./phase-1/02-meridian-health/ch06-jwt-oauth2-auth-system.md) | MeridianHealth | 1 |
| 07 | [Build the HIPAA-Compliant Audit Trail](./phase-1/02-meridian-health/ch07-hipaa-audit-trails.md) | MeridianHealth | 1 |
| 08 | [Data Residency — The State That Wants Its Data Back](./phase-1/02-meridian-health/ch08-data-residency-and-hipaa-architecture.md) | MeridianHealth | 1 |
| 19 | [Presigned URLs and the Pirates Problem](./phase-1/04-beacon-media/ch19-presigned-urls-and-content-security.md) | Beacon Media | 1 |
| 28 | [Multi-Tenant Architecture and Tenant Isolation](./phase-1/06-cloudstack/ch28-multi-tenancy-architecture.md) | CloudStack | 1 |
| 30 | [Auth at Scale — SSO, Service Accounts, and API Keys (Second Appearance)](./phase-1/06-cloudstack/ch30-auth-system-second-appearance.md) | CloudStack | 1 |
| 39 | [Design the Data Sovereignty Architecture for Government Clients](./phase-1/08-civicos/ch39-data-sovereignty-and-compliance-architecture.md) | CivicOS | 1 |
| 40 | [Rate Limiting for Government APIs — The Enumeration Attack](./phase-1/08-civicos/ch40-rate-limiting-second-appearance.md) | CivicOS | 1 |
| 75 | [Secret Management with HashiCorp Vault](./phase-2/02-nexacare/ch75-secret-management-intro.md) | NexaCare | 2 |
| 83 | [Service Mesh and mTLS Introduction](./phase-2/03-tradespark/ch83-service-mesh-mtls-intro.md) | TradeSpark | 2 |
| 92 | [Zero-Trust Architecture](./phase-2/05-sentinelops/ch92-zero-trust-architecture.md) | SentinelOps | 2 |
| 93 | [Service Mesh Deep Dive — Istio in Production](./phase-2/05-sentinelops/ch93-service-mesh-deep-dive.md) | SentinelOps | 2 |
| 94 | [Advanced Authorization — RBAC vs ABAC and Policy-as-Code](./phase-2/05-sentinelops/ch94-auth-fourth-appearance.md) | SentinelOps | 2 |
| 95 | [Secret Management at Scale](./phase-2/05-sentinelops/ch95-secret-management-at-scale.md) | SentinelOps | 2 |
| 97 | [Security Incident Forensics Pipeline](./phase-2/05-sentinelops/ch97-security-incident-forensics-pipeline.md) | SentinelOps | 2 |
| 118 | [Adaptive Rate Limiting and Ticket Bot DDoS](./phase-2/08-venueflow/ch118-advanced-rate-limiting-ddos.md) | VenueFlow | 2 |
| 152 | [Sovereignty in Orbit](./phase-3/01-orbitcore/ch152-sovereignty-in-orbit.md) | OrbitCore | 3 |
| 157 | [The Briefing Room](./phase-3/02-ironwatch/ch157-the-briefing-room.md) | IronWatch | 3 |
| 179 | [FDA 510(k) Architecture](./phase-3/05-neuralbridge/ch179-fda-510k-architecture.md) | NeuralBridge | 3 |
| 189 | [SOC2 Under Pressure](./phase-3/06-shieldmutual/ch189-soc2-under-pressure.md) | ShieldMutual | 3 |
| 195 | [OTA at Fleet Scale](./phase-3/07-automesh/ch195-ota-at-fleet-scale.md) | AutoMesh | 3 |
| 205 | [The Ghost in the Machine](./phase-3/08-forgesense/ch205-inherited-disaster-scada.md) | ForgeSense | 3 |
| 216 | [21 CFR Part 11](./phase-3/10-pharmasync/ch216-21-cfr-part-11.md) | PharmaSync | 3 |
| 261 | [The Quantum API Gateway and Auth (Auth System: 6th Evolution)](./phase-3/16-quantumedge/ch261-quantum-api-gateway-and-auth.md) | QuantumEdge | 3 |

</details>

<a name="tag-service-mesh"></a>

<details>
<summary><strong>#service-mesh</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 83 | [Service Mesh and mTLS Introduction](./phase-2/03-tradespark/ch83-service-mesh-mtls-intro.md) | TradeSpark | 2 |
| 93 | [Service Mesh Deep Dive — Istio in Production](./phase-2/05-sentinelops/ch93-service-mesh-deep-dive.md) | SentinelOps | 2 |
| 120 | [Service Mesh at Production Scale — 60 Services, 18 Months of Istio](./phase-2/09-telanova/ch120-service-mesh-production.md) | TeleNova | 2 |
| 139 | [mTLS for HIPAA — Every Service Call Is a PHI Flow](./phase-2/12-prismhealth/ch139-service-mesh-healthcare-compliance.md) | PrismHealth | 2 |

</details>

<a name="tag-settlement"></a>

<details>
<summary><strong>#settlement</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 80 | [Distributed Transactions — 2PC vs Saga](./phase-2/03-tradespark/ch80-distributed-transactions-2pc-vs-saga.md) | TradeSpark | 2 |
| 81 | [DTCC Clearing Integration and Financial Settlement](./phase-2/03-tradespark/ch81-payments-fourth-appearance.md) | TradeSpark | 2 |

</details>

<a name="tag-sla"></a>

<details>
<summary><strong>#sla</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 153 | [The Latency Ladder](./phase-3/01-orbitcore/ch153-the-latency-ladder.md) | OrbitCore | 3 |
| 200 | [Five Nines for a Moving Vehicle](./phase-3/07-automesh/ch200-the-partnership-aftermath.md) | AutoMesh | 3 |

</details>

<a name="tag-sli"></a>

<details>
<summary><strong>#sli</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 74 | [Advanced Observability — SLOs, SLIs, and Error Budgets](./phase-2/02-nexacare/ch74-advanced-observability-slos.md) | NexaCare | 2 |
| 109 | [Grid Reliability SLOs — When 99.9% Is Not Good Enough](./phase-2/07-crestline-energy/ch109-slo-sli-error-budget-design.md) | Crestline Energy | 2 |

</details>

<a name="tag-slo"></a>

<details>
<summary><strong>#slo</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 74 | [Advanced Observability — SLOs, SLIs, and Error Budgets](./phase-2/02-nexacare/ch74-advanced-observability-slos.md) | NexaCare | 2 |
| 109 | [Grid Reliability SLOs — When 99.9% Is Not Good Enough](./phase-2/07-crestline-energy/ch109-slo-sli-error-budget-design.md) | Crestline Energy | 2 |
| 140 | [Life-Safety Alerting — When False Positives Kill Trust](./phase-2/12-prismhealth/ch140-advanced-observability-alerting.md) | PrismHealth | 2 |

</details>

<a name="tag-spike"></a>

<details>
<summary><strong>#spike</strong> — 5 chapters across 5 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 97 | [Security Incident Forensics Pipeline](./phase-2/05-sentinelops/ch97-security-incident-forensics-pipeline.md) | SentinelOps | 2 |
| 156 | [SPIKE — Solar Flare](./phase-3/01-orbitcore/ch156-solar-flare-spike.md) | OrbitCore | 3 |
| 173 | [Where Are the Embeddings?](./phase-3/04-lexcore/ch173-design-review-ambush-spike.md) | LexCore | 3 |
| 196 | [Scale 10x — Fleet Doubles](./phase-3/07-automesh/ch196-scale-10x-fleet-doubles-spike.md) | AutoMesh | 3 |
| 240 | [Inherited Disaster — The Data Quality Crisis](./phase-3/13-propiq/ch240-inherited-disaster-data-quality-crisis.md) | PropIQ | 3 |

</details>

<a name="tag-spike-design-review"></a>

<details>
<summary><strong>#spike-design-review</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 128 | [Experimentation Platform — Contradictory A/B Results](./phase-2/10-buildright/ch128-a-b-testing-and-experimentation.md) | BuildRight | 2 |
| 149 | [The Game Day That Became a Real Incident](./phase-2/13-vertexcloud/ch149-chaos-engineering-intro.md) | VertexCloud | 2 |

</details>

<a name="tag-spike-inherited-disaster"></a>

<details>
<summary><strong>#spike-inherited-disaster</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 108 | [The 18-Hour Batch Job](./phase-2/07-crestline-energy/ch108-streaming-vs-batch-flink.md) | Crestline Energy | 2 |
| 122 | [Six Hours of Consumer Lag](./phase-2/09-telanova/ch122-advanced-kafka-consumer-lag.md) | TeleNova | 2 |
| 137 | [Genomic PII in the Application Logs](./phase-2/11-axiom-labs/ch137-hipaa-compliance-remediation.md) | Axiom Labs | 2 |

</details>

<a name="tag-spike-production-down"></a>

<details>
<summary><strong>#spike-production-down</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 114 | [Concert Cancellation Notification Cascade (4th Appearance)](./phase-2/08-venueflow/ch114-notifications-fourth-appearance.md) | VenueFlow | 2 |
| 129 | [Progressive Delivery with Argo Rollouts](./phase-2/10-buildright/ch129-canary-deployments-advanced.md) | BuildRight | 2 |
| 188 | [Claims at Scale](./phase-3/06-shieldmutual/ch188-claims-at-scale.md) | ShieldMutual | 3 |
| 249 | [The Game Day That Went Wrong](./phase-3/14-novasports/ch249-the-game-day-that-went-wrong.md) | NovaSports | 3 |

</details>

<a name="tag-spike-scale-10x"></a>

<details>
<summary><strong>#spike-scale-10x</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 118 | [Adaptive Rate Limiting and Ticket Bot DDoS](./phase-2/08-venueflow/ch118-advanced-rate-limiting-ddos.md) | VenueFlow | 2 |
| 125 | [Five Country Launches in One Quarter](./phase-2/09-telanova/ch125-multi-region-active-active-advanced.md) | TeleNova | 2 |
| 167 | [SPIKE — Scale 10x by Friday](./phase-3/03-carbonledger/ch167-scale-10x-spike.md) | CarbonLedger | 3 |
| 234 | [VIX 80 — The ERISA Clock Is Ticking](./phase-3/12-nexuswealth/ch234-scale-10x-market-volatility-spike.md) | NexusWealth | 3 |

</details>

<a name="tag-spot-instances"></a>

<details>
<summary><strong>#spot-instances</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 101 | [Immutable Infrastructure and Black Friday Scaling](./phase-2/06-lightspeedretail/ch101-immutable-infrastructure-and-scaling.md) | LightspeedRetail | 2 |
| 110 | [Three Years of Growth, Two Years of Budget](./phase-2/07-crestline-energy/ch110-capacity-planning-and-forecasting.md) | Crestline Energy | 2 |

</details>

<a name="tag-sql"></a>

<details>
<summary><strong>#sql</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 03 | [The Slow Query Nobody Wrote a Ticket For](./phase-1/01-novapay/ch03-database-indexing-for-financial-queries.md) | NovaPay | 1 |
| 09 | [Scale the EHR Query Layer with Read Replicas](./phase-1/02-meridian-health/ch09-read-replicas-and-query-routing.md) | MeridianHealth | 1 |
| 35 | [The N+1 Query Disaster Eating Your Database](./phase-1/07-pulsecommerce/ch35-n-plus-one-queries-and-dataloader.md) | PulseCommerce | 1 |

</details>

<a name="tag-sre"></a>

<details>
<summary><strong>#sre</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 74 | [Advanced Observability — SLOs, SLIs, and Error Budgets](./phase-2/02-nexacare/ch74-advanced-observability-slos.md) | NexaCare | 2 |
| 109 | [Grid Reliability SLOs — When 99.9% Is Not Good Enough](./phase-2/07-crestline-energy/ch109-slo-sli-error-budget-design.md) | Crestline Energy | 2 |

</details>

<a name="tag-sso"></a>

<details>
<summary><strong>#sso</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 30 | [Auth at Scale — SSO, Service Accounts, and API Keys (Second Appearance)](./phase-1/06-cloudstack/ch30-auth-system-second-appearance.md) | CloudStack | 1 |
| 41 | [SSO for 23 State Governments — Federated Identity at Scale](./phase-1/08-civicos/ch41-sso-and-identity-federation-for-government.md) | CivicOS | 1 |

</details>

<a name="tag-staff-engineering"></a>

<details>
<summary><strong>#staff-engineering</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 235 | [The Trustees Want Answers](./phase-3/12-nexuswealth/ch235-the-board-presentation.md) | NexusWealth | 3 |
| 237 | [A Letter to 2074](./phase-3/12-nexuswealth/ch237-the-50-year-architecture.md) | NexusWealth | 3 |

</details>

<a name="tag-state-machine"></a>

<details>
<summary><strong>#state-machine</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 15 | [The Delivery State Machine at Scale](./phase-1/03-velotrack/ch15-kafka-exactly-once-and-delivery-state-machine.md) | VeloTrack | 1 |
| 46 | [Design the Adaptive Quiz Engine at Scale](./phase-1/09-neurolearn/ch46-adaptive-quiz-engine-design.md) | NeuroLearn | 1 |

</details>

<a name="tag-statistics"></a>

<details>
<summary><strong>#statistics</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 117 | [A/B Testing Infrastructure](./phase-2/08-venueflow/ch117-a-b-testing-infrastructure.md) | VenueFlow | 2 |
| 128 | [Experimentation Platform — Contradictory A/B Results](./phase-2/10-buildright/ch128-a-b-testing-and-experimentation.md) | BuildRight | 2 |

</details>

<a name="tag-storage"></a>

<details>
<summary><strong>#storage</strong> — 7 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 18 | [Fix the CDN — $2M/Month and Falling](./phase-1/04-beacon-media/ch18-blob-storage-and-cdn-architecture.md) | Beacon Media | 1 |
| 19 | [Presigned URLs and the Pirates Problem](./phase-1/04-beacon-media/ch19-presigned-urls-and-content-security.md) | Beacon Media | 1 |
| 21 | [Scale 10x — The New Content Deal is Going to Break Transcoding](./phase-1/04-beacon-media/ch21-video-transcoding-pipeline.md) | Beacon Media | 1 |
| 169 | [The Cost of Green](./phase-3/03-carbonledger/ch169-the-cost-of-green.md) | CarbonLedger | 3 |
| 170 | [The Permanence Problem](./phase-3/03-carbonledger/ch170-the-permanence-problem.md) | CarbonLedger | 3 |
| 218 | [Feature Stores for Drug Discovery](./phase-3/10-pharmasync/ch218-feature-stores-drug-discovery.md) | PharmaSync | 3 |
| 230 | [Fifty Years of Data](./phase-3/12-nexuswealth/ch230-fifty-years-of-data.md) | NexusWealth | 3 |

</details>

<a name="tag-streaming"></a>

<details>
<summary><strong>#streaming</strong> — 9 chapters across 7 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 55 | [ETL to Streaming — Evolving the Data Pipeline](./phase-1/11-luminaryai/ch55-etl-to-streaming-pipeline-evolution.md) | LuminaryAI | 1 |
| 108 | [The 18-Hour Batch Job](./phase-2/07-crestline-energy/ch108-streaming-vs-batch-flink.md) | Crestline Energy | 2 |
| 178 | [The Signal](./phase-3/05-neuralbridge/ch178-the-signal.md) | NeuralBridge | 3 |
| 209 | [The Ocean's Heartbeat](./phase-3/09-deepocean/ch209-ais-the-oceans-heartbeat.md) | DeepOcean | 3 |
| 233 | [Ten Minutes to Exposure](./phase-3/12-nexuswealth/ch233-real-time-portfolio-risk.md) | NexusWealth | 3 |
| 242 | [The Real-Time Lie](./phase-3/13-propiq/ch242-real-time-market-aggregation.md) | PropIQ | 3 |
| 245 | [50 Million Events Per Second](./phase-3/14-novasports/ch245-50-million-events-per-second.md) | NovaSports | 3 |
| 248 | [Seven Seconds to National Television](./phase-3/14-novasports/ch248-real-time-sports-analytics.md) | NovaSports | 3 |
| 251 | [Before Next Season](./phase-3/14-novasports/ch251-streaming-architecture-review.md) | NovaSports | 3 |

</details>

<a name="tag-supply-chain"></a>

<details>
<summary><strong>#supply-chain</strong> — 4 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 50 | [Design the Multi-Step Supply Chain Transaction with Saga](./phase-1/10-omnilogix/ch50-distributed-transactions-and-saga-pattern.md) | OmniLogix | 1 |
| 51 | [Idempotency at Scale — When Retries Become Dangerous](./phase-1/10-omnilogix/ch51-idempotency-at-scale.md) | OmniLogix | 1 |
| 52 | [Kafka Exactly-Once at the Application Level](./phase-1/10-omnilogix/ch52-kafka-exactly-once-application-level.md) | OmniLogix | 1 |
| 53 | [Global Inventory Consistency — The Full Design](./phase-1/10-omnilogix/ch53-global-inventory-consistency-synthesis.md) | OmniLogix | 1 |

</details>

<a name="tag-sync"></a>

<details>
<summary><strong>#sync</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 210 | [Three Weeks at Sea](./phase-3/09-deepocean/ch210-offline-first-for-ships.md) | DeepOcean | 3 |
| 213 | [Designing for the Speed of Light](./phase-3/09-deepocean/ch213-extreme-latency-architecture.md) | DeepOcean | 3 |

</details>

<a name="tag-system-design"></a>

<details>
<summary><strong>#system-design</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 20 | [Design the Notification System (First Appearance)](./phase-1/04-beacon-media/ch20-notification-system-first-appearance.md) | Beacon Media | 1 |
| 42 | [Design for Accessibility, Resilience, and the Citizen Who Can't Retry](./phase-1/08-civicos/ch42-accessibility-and-resilience-for-government-systems.md) | CivicOS | 1 |
| 47 | [Design Review Ambush — The Academic Integrity Architecture](./phase-1/09-neurolearn/ch47-academic-integrity-monitoring.md) | NeuroLearn | 1 |
| 253 | [The Precision-Recall Dilemma](./phase-3/15-mindscale/ch253-crisis-detection-ml.md) | MindScale | 3 |

</details>

<a name="tag-system-evolution"></a>

<details>
<summary><strong>#system-evolution</strong> — 4 chapters across 4 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 10 | [Scale 10x by Friday — The Northview Surge](./phase-1/02-meridian-health/ch10-ehr-integration-platform-design.md) | MeridianHealth | 1 |
| 32 | [Solve the Noisy Neighbor Problem for Good](./phase-1/06-cloudstack/ch32-noisy-neighbor-and-resource-quotas.md) | CloudStack | 1 |
| 34 | [Payment Integration at Scale — Scaling Challenges (Second Appearance)](./phase-1/07-pulsecommerce/ch34-payment-processing-second-appearance.md) | PulseCommerce | 1 |
| 220 | [Gerald Must Die](./phase-3/10-pharmasync/ch220-the-lims-deprecation.md) | PharmaSync | 3 |

</details>

<a name="tag-temporal-queries"></a>

<details>
<summary><strong>#temporal-queries</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 77 | [Low-Latency Event Sourcing for Trade Execution](./phase-2/03-tradespark/ch77-low-latency-event-sourcing.md) | TradeSpark | 2 |
| 112 | ["What Was the Grid State at 14:32:07 on March 3rd?"](./phase-2/07-crestline-energy/ch112-cqrs-temporal-queries.md) | Crestline Energy | 2 |
| 142 | [A Patient's Complete Vital History](./phase-2/12-prismhealth/ch142-cqrs-for-patient-timelines.md) | PrismHealth | 2 |

</details>

<a name="tag-third-appearance"></a>

<details>
<summary><strong>#third-appearance</strong> — 4 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 59 | [Notification System — 8 Million in 40 Minutes](./phase-1/12-skyroute/ch59-notification-system-third-appearance.md) | SkyRoute | 1 |
| 60 | [Flight Search — The Hardest Search Problem](./phase-1/12-skyroute/ch60-flight-search-third-appearance.md) | SkyRoute | 1 |
| 61 | [Auth for the Most Adversarial User Base](./phase-1/12-skyroute/ch61-auth-third-appearance.md) | SkyRoute | 1 |
| 62 | [Rate Limiting — Defending Against the GDS Giants](./phase-1/12-skyroute/ch62-rate-limiting-third-appearance.md) | SkyRoute | 1 |

</details>

<a name="tag-thundering-herd"></a>

<details>
<summary><strong>#thundering-herd</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 82 | [Advanced Caching — Probabilistic Early Expiration](./phase-2/03-tradespark/ch82-advanced-caching-probabilistic-expiration.md) | TradeSpark | 2 |
| 100 | [Advanced Caching — Black Friday Thundering Herd](./phase-2/06-lightspeedretail/ch100-advanced-caching-thundering-herd.md) | LightspeedRetail | 2 |

</details>

<a name="tag-time-series"></a>

<details>
<summary><strong>#time-series</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 23 | [Design the Time-Series Sensor Data Platform](./phase-1/05-agrosense/ch23-time-series-data-design.md) | AgroSense | 1 |
| 106 | [Time-Series at Scale — 50M Meters, 200M Events Per Hour](./phase-2/07-crestline-energy/ch106-time-series-at-scale-advanced.md) | Crestline Energy | 2 |
| 202 | [The Digital Twin](./phase-3/08-forgesense/ch202-the-digital-twin.md) | ForgeSense | 3 |

</details>

<a name="tag-toil"></a>

<details>
<summary><strong>#toil</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 74 | [Advanced Observability — SLOs, SLIs, and Error Budgets](./phase-2/02-nexacare/ch74-advanced-observability-slos.md) | NexaCare | 2 |
| 109 | [Grid Reliability SLOs — When 99.9% Is Not Good Enough](./phase-2/07-crestline-energy/ch109-slo-sli-error-budget-design.md) | Crestline Energy | 2 |

</details>

<a name="tag-trading"></a>

<details>
<summary><strong>#trading</strong> — 3 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 77 | [Low-Latency Event Sourcing for Trade Execution](./phase-2/03-tradespark/ch77-low-latency-event-sourcing.md) | TradeSpark | 2 |
| 79 | [Advanced Kafka — Log Compaction for Order Book State](./phase-2/03-tradespark/ch79-advanced-kafka-log-compaction.md) | TradeSpark | 2 |
| 80 | [Distributed Transactions — 2PC vs Saga](./phase-2/03-tradespark/ch80-distributed-transactions-2pc-vs-saga.md) | TradeSpark | 2 |

</details>

<a name="tag-transactions"></a>

<details>
<summary><strong>#transactions</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 50 | [Design the Multi-Step Supply Chain Transaction with Saga](./phase-1/10-omnilogix/ch50-distributed-transactions-and-saga-pattern.md) | OmniLogix | 1 |
| 67 | [Database Internals — WAL and MVCC](./phase-2/01-stratum-systems/ch67-database-internals-wal-and-mvcc.md) | Stratum Systems | 2 |

</details>

<a name="tag-typeahead"></a>

<details>
<summary><strong>#typeahead</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 89 | [Advanced Search — Facets and Typeahead](./phase-2/04-giggrid/ch89-advanced-search-facets-typeahead.md) | GigGrid | 2 |
| 127 | [Advanced Search — Material Catalog and Vendor Discovery](./phase-2/10-buildright/ch127-advanced-search-typeahead-facets.md) | BuildRight | 2 |

</details>

<a name="tag-v2x"></a>

<details>
<summary><strong>#v2x</strong> — 3 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 193 | [V2X at Scale](./phase-3/07-automesh/ch193-v2x-at-scale.md) | AutoMesh | 3 |
| 199 | [Every Way This Can Kill Someone](./phase-3/07-automesh/ch199-fmea-driven-architecture.md) | AutoMesh | 3 |
| 200 | [Five Nines for a Moving Vehicle](./phase-3/07-automesh/ch200-the-partnership-aftermath.md) | AutoMesh | 3 |

</details>

<a name="tag-vault"></a>

<details>
<summary><strong>#vault</strong> — 2 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 75 | [Secret Management with HashiCorp Vault](./phase-2/02-nexacare/ch75-secret-management-intro.md) | NexaCare | 2 |
| 95 | [Secret Management at Scale](./phase-2/05-sentinelops/ch95-secret-management-at-scale.md) | SentinelOps | 2 |

</details>

<a name="tag-vector-search"></a>

<details>
<summary><strong>#vector-search</strong> — 3 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 115 | [Vector Search — Personalized Event Recommendations](./phase-2/08-venueflow/ch115-search-fourth-appearance.md) | VenueFlow | 2 |
| 132 | [Finding Genetic Needles in Petabyte Haystacks](./phase-2/11-axiom-labs/ch132-vector-search-deep-dive.md) | Axiom Labs | 2 |
| 239 | [Search for Property Intelligence](./phase-3/13-propiq/ch239-search-for-property-intelligence.md) | PropIQ | 3 |

</details>

<a name="tag-vpp"></a>

<details>
<summary><strong>#vpp</strong> — 2 chapters across 1 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 266 | [The Virtual Power Plant](./phase-3/17-glaciergrid/ch266-the-virtual-power-plant.md) | GlacierGrid | 3 |
| 270 | [The Demand Cliff](./phase-3/17-glaciergrid/ch270-real-time-grid-stability.md) | GlacierGrid | 3 |

</details>

<a name="tag-websockets"></a>

<details>
<summary><strong>#websockets</strong> — 4 chapters across 3 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 13 | [Design the Real-Time Driver Tracking System](./phase-1/03-velotrack/ch13-real-time-delivery-tracking.md) | VeloTrack | 1 |
| 14 | [Design the Load Balancing Strategy for a Heterogeneous Fleet](./phase-1/03-velotrack/ch14-load-balancing-strategies.md) | VeloTrack | 1 |
| 86 | [Real-Time at Scale — WebSockets, SSE, and Pub/Sub](./phase-2/04-giggrid/ch86-real-time-at-scale-websockets-sse.md) | GigGrid | 2 |
| 113 | [Real-Time at Scale — 500k Concurrent Fans](./phase-2/08-venueflow/ch113-real-time-at-scale-pub-sub.md) | VenueFlow | 2 |

</details>

<a name="tag-zero-trust"></a>

<details>
<summary><strong>#zero-trust</strong> — 4 chapters across 2 company/companies</summary>

| Ch | Title | Company | Phase |
|----|-------|---------|-------|
| 92 | [Zero-Trust Architecture](./phase-2/05-sentinelops/ch92-zero-trust-architecture.md) | SentinelOps | 2 |
| 93 | [Service Mesh Deep Dive — Istio in Production](./phase-2/05-sentinelops/ch93-service-mesh-deep-dive.md) | SentinelOps | 2 |
| 95 | [Secret Management at Scale](./phase-2/05-sentinelops/ch95-secret-management-at-scale.md) | SentinelOps | 2 |
| 158 | [Zero Trust, Zero Margin](./phase-3/02-ironwatch/ch158-zero-trust-zero-margin.md) | IronWatch | 3 |

</details>

**Single-occurrence tags** (each appears in exactly one chapter):

`#21cfr11`, `#2pc`, `#42CFR-Part2`, `#5g`, `#ABAC`, `#E2E`, `#Elasticsearch`, `#FIDO2`, `#HIPAA`, `#SLA`, `#UK-GDPR`, `#WORM`, `#academic`, `#accessibility`, `#actuarial`, `#adaptive`, `#aggregation`, `#ais`, `#algorithm`, `#algorithm-design`, `#anonymization`, `#anycast`, `#apache-flink`, `#apache-parquet`, `#api-governance`, `#api-integration`, `#api-keys`, `#apns`, `#appi`, `#argo-rollouts`, `#article6`, `#audit-escrow`, `#authority-levels`, `#automated-rollback`, `#automation`, `#autovacuum`, `#avro`, `#b2b2c`, `#backstage`, `#behavioral-fingerprinting`, `#bidirectional-control`, `#bloat`, `#breach-notification`, `#breaking-changes`, `#broadcast`, `#brownfield`, `#btree`, `#bulkhead`, `#bulkheads`, `#cache`, `#carbon-credits`, `#career-arc`, `#causal-consistency`, `#cell-based-architecture`, `#chargeback`, `#chunking`, `#citation-graph`, `#clearance-tiers`, `#clearing`, `#client-side-encryption`, `#clinical-review`, `#clinical-trials`, `#cold-start`, `#columnar`, `#columnar-storage`, `#command-audit`, `#compartmentalization`, `#compensating-transactions`, `#competitive-sensitivity`, `#compute`, `#concurrency`, `#conflict-free`, `#conflict-of-interest`, `#connectivity`, `#connectivity-tiers`, `#consensus`, `#consent`, `#consent-management`, `#constraint-programming`, `#consumer-vehicles`, `#cost-attribution`, `#cost-modeling`, `#cross-domain-solution`, `#data-access`, `#data-architecture`, `#data-breach`, `#data-catalog`, `#data-deletion`, `#data-diode`, `#data-flow`, `#data-infrastructure`, `#data-modeling`, `#data-tiering`, `#database-design`, `#database-migrations`, `#dataloader`, `#ddos`, `#de-identification`, `#debezium`, `#debugging`, `#demand-response`, `#deployments`, `#deprecation`, `#developer-experience`, `#device-authentication`, `#device-identity`, `#device-registry`, `#differential-updates`, `#digital-twin`, `#discoverability`, `#distributed-lock`, `#distributed-transactions`, `#distribution`, `#document-processing`, `#documentation`, `#domain-ownership`, `#downsampling`, `#drm`, `#drug-discovery`, `#dtcc`, `#edge-processing`, `#education`, `#encryption`, `#energy`, `#energy-analytics`, `#envoy`, `#error-handling`, `#eu-energy`, `#event-log-deletion`, `#event-ordering`, `#event-streaming`, `#evidence-preservation`, `#external-data`, `#facets`, `#fair-use`, `#fairness`, `#false-positive`, `#fan-in-ingestion`, `#fcm`, `#fda-validation`, `#federated-analytics`, `#federated-computation`, `#federated-data`, `#federated-ml`, `#fencing-tokens`, `#fiduciary`, `#fifth-appearance`, `#financial`, `#firmware`, `#fleet`, `#flink`, `#forecasting`, `#fraud`, `#frequency-regulation`, `#game-day`, `#gaps`, `#gds`, `#geosearch`, `#global-expansion`, `#golden-paths`, `#graceful-degradation`, `#grafana-tempo`, `#gremlin`, `#grid-edge`, `#grid-stability`, `#ground-station`, `#hardware`, `#hashicorp`, `#heterogeneous-protocols`, `#hierarchical-control`, `#hierarchical-permissions`, `#high-availability`, `#hnsw`, `#hybrid-pipeline`, `#iac`, `#idp`, `#immutable-infra`, `#immutable-storage`, `#indexing`, `#influxdb`, `#ingestion`, `#inverted-index`, `#iot-at-scale`, `#isolation-levels`, `#job-monitoring`, `#jwt`, `#k-anonymity`, `#key-escrow`, `#key-management`, `#lamport-clocks`, `#late-arriving-events`, `#leader-election`, `#legal-AI`, `#legal-ai`, `#legal-hold`, `#life-safety`, `#linkerd`, `#log-compaction`, `#log-remediation`, `#long-polling`, `#long-term-thinking`, `#low-latency`, `#manufacturing`, `#mapping`, `#marcus-webb`, `#market-data`, `#mentor`, `#mentorship`, `#mergers-acquisitions`, `#message-ordering`, `#migration`, `#ml`, `#ml-deployment`, `#ml-pipeline`, `#ml-serving`, `#mlops`, `#mobile`, `#model-monitoring`, `#model-serving`, `#model-updates`, `#model-versioning`, `#mqtt`, `#multi-channel`, `#multi-criteria`, `#multi-env`, `#multi-source`, `#multilingual`, `#mvcc`, `#n-plus-one`, `#narrative`, `#network-policy`, `#network-security`, `#network-segmentation`, `#nhtsa`, `#notification-fatigue`, `#oceanography`, `#oltp`, `#opa`, `#optimistic-updates`, `#orbital-mechanics`, `#order-book`, `#ota-updates`, `#outbox-pattern`, `#overbooking`, `#p2pe`, `#packer`, `#partition-reassignment`, `#patient-safety`, `#patient-timelines`, `#per-subscriber`, `#per-tenant-ranking`, `#performance-optimization`, `#personalization`, `#petabyte-scale`, `#pgvector`, `#pharma`, `#phi`, `#pii`, `#pinecone`, `#pipeda`, `#platform-design`, `#platform-economics`, `#platform-engineering`, `#policy-as-code`, `#policy-engine`, `#postgis`, `#precision-recall`, `#predictive-maintenance`, `#presigned-urls`, `#presto`, `#privacy-by-design`, `#probabilistic-expiration`, `#production`, `#protocol-migration`, `#purpose-bound-storage`, `#qos`, `#query-optimization`, `#queues`, `#race-conditions`, `#raft`, `#re-identification`, `#real-time-inventory`, `#rebalancing-storm`, `#recommendation`, `#recommendations`, `#reconciliation`, `#redshift`, `#reflection`, `#regulatory-audit`, `#relevance-tuning`, `#research`, `#reserved-instances`, `#retention`, `#retention-policy`, `#retries`, `#rfc`, `#right-to-erasure`, `#rightsizing`, `#risk`, `#risk-analysis`, `#risk-scoring`, `#rolling-windows`, `#routing`, `#saas`, `#safety`, `#saml`, `#scada`, `#scale-10x`, `#schema-registry`, `#scim`, `#seasonal-scaling`, `#sec`, `#secret-rotation`, `#self-service-infra`, `#shadow-mode`, `#showback`, `#sidecar`, `#sim-auth`, `#similarity-search`, `#sixth-appearance`, `#sliding-window`, `#slo-breach`, `#snapshot`, `#soc2`, `#spatial`, `#spatial-indexing`, `#spike-scale10x`, `#sse`, `#staff-engineer`, `#staff-patterns`, `#staff-skills`, `#staff-work`, `#stakeholder-management`, `#star-schema`, `#strangler-fig`, `#structured-logging`, `#substance-use`, `#synthesis`, `#tagging`, `#tail-sampling`, `#telemetry`, `#tempo`, `#temporal-alignment`, `#tenant-isolation`, `#terraform`, `#threat-intel`, `#time-travel`, `#timescaledb`, `#traffic-management`, `#traffic-steering`, `#travel-agent-bots`, `#travel-agents`, `#typescript`, `#ux`, `#vacuum`, `#vector-clocks`, `#versioning`, `#wal`, `#watermarks`, `#worker-assignment`, `#worm`, `#write-amplification`, `#write-stall`, `#z-ordering`, `#zero-downtime`, `#zero-knowledge`

</details>

---

## How to Contribute

Contributions are welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide.

**Welcome:**
- New company arcs or chapters in industries not yet covered
- Typo and link fixes
- Translations
- Clarifications that improve ambiguity without removing it

**Not welcome:**
- Reference solutions (intentionally absent — this is a feature, not a bug)
- Opinionated rewrites of existing chapters
- Changes that reduce technical depth

Fork → create a branch → submit a PR with a clear description of what changed and why.

---

## License

MIT License — [YOUR NAME]

---

## Author

Built by [YOUR NAME] · [YOUR GITHUB] · [YOUR LINKEDIN]

*If this would have helped past you, please star the repo and share it with someone who would benefit.*