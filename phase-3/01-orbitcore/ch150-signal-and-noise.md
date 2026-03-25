---
**Title**: OrbitCore — Chapter 150: Signal and Noise
**Level**: Staff
**Difficulty**: 7
**Tags**: #telemetry #ingestion #data-pipeline #distributed #messaging #data-sovereignty #capacity-planning
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 22 (AgroSense time-series ingestion), Ch. 55 (LuminaryAI streaming pipeline), Ch. 105 (Crestline Energy telemetry)
**Exercise Type**: System Design
---

### Story Context

It is 3:04pm on your first day.

You are in the middle of reading a Confluence page titled "Ground Station Architecture v2.1 — DRAFT" when your phone buzzes. Then buzzes again. Then a third time in rapid succession.

**[#ground-ops — Slack, 3:04pm]**

> **Niamh Okafor** [Ground Ops Engineer]
> GS-Singapore is throwing buffer overflow warnings on the ingestion queue again. 4th time this week.

> **Preet Kapila** [SRE Lead]
> @channel — staging is also affected. The queue depth on `telemetry-raw-ingest-sg` hit 2.1M messages, auto-scaled, then dropped back. No data loss confirmed but this is the third time this week we've had the auto-scaler racing the queue fill rate.

> **Tomás Reyes** [Staff Engineer, Infra]
> Yeah the Singapore cluster is undersized for the LEO window between 14:00–16:30 UTC. Three of our Kepler constellation satellites are all visible simultaneously from Singapore during that window. We sized for two.

> **Ingrid Solberg** [VP Engineering]
> @newstaff — yes, this is you now. Welcome to OrbitCore. Tomás will walk you through the current architecture. We need a proper design for the ingestion pipeline. Not a patch. A design.

You open a new Notion doc and start taking notes.

---

**[1:1 Slack DM — Tomás Reyes → You, 3:22pm]**

> **Tomás Reyes**
> Welcome to the deep end.
> Quick context: the current pipeline was designed for our first customer, Kepler Dynamics, who had 12 satellites. We now have 47 across 4 constellations. The architecture hasn't changed. We just kept adding Kafka partitions and hoping.

> **You**
> How bad is the "just adding partitions" approach right now?

> **Tomás Reyes**
> We're at 384 partitions on `telemetry-raw` and our consumer lag monitoring is a mess. We can't tell if we're actually behind or if the partition lag metrics are misleading us.
> Also, fun fact: the Vantage constellation (EU satellites) has a GDPR clause in their contract. EU telemetry cannot be processed on US infrastructure. It currently is. Legal knows. They are not happy.

> **You**
> Since when?

> **Tomás Reyes**
> Since we signed the contract 8 months ago. The EU processing constraint is in Appendix B, paragraph 3. Nobody told engineering.

---

**[Email chain]**

**From:** Ingrid Solberg <ingrid@orbitcore.io>
**To:** Engineering Leadership <eng-leads@orbitcore.io>
**CC:** James Whitfield <james.whitfield@orbitcore.io> [CEO]
**Subject:** RE: RE: RE: Series C Data Room — Technical Due Diligence Items
**Date:** March 8 (yesterday)

> James — the Series C lead investor (Orion Ventures) has flagged our telemetry infrastructure as a "technical risk item" in the due diligence review. Specifically they asked:
>
> 1. Can the pipeline handle 200 satellites (4x current load)?
> 2. How do you handle ground station failure?
> 3. What is your data residency story?
>
> I need concrete answers to all three before the follow-up call on March 23rd. That gives engineering 14 days.
>
> — Ingrid

**[Note in the email thread, handwritten and photographed, attached to the email]**
> "Also — confirm whether Singapore GS burst buffering is documented anywhere. Solar interference season starts April. If we lose Singapore during a Kepler LEO window we lose 40 minutes of telemetry. That's contractually unacceptable." — J.W.

---

**[#ground-ops — Slack, 4:55pm]**

> **Niamh Okafor**
> Update: Singapore queue cleared. No data loss. But I manually restarted the ingest workers. That shouldn't be a manual step.

> **Preet Kapila**
> Adding to the runbook. Again.

> **Ingrid Solberg**
> No. Stop adding to the runbook. Fix the system.

By 5pm, you have three pages of notes and a clear sense of what you are here to do.

### Problem Statement

OrbitCore's telemetry ingestion pipeline is a single logical Kafka cluster handling 2 million events per second from 47 satellites across 12 ground stations in 6 countries. The pipeline was designed for 12 satellites and has been scaled horizontally without architectural changes. It exhibits regular buffer overflows during peak orbital windows (when multiple satellites are simultaneously visible from a single ground station), has no documented burst-buffering strategy for solar interference events, and is in violation of GDPR data residency requirements for EU-constellation telemetry.

Design a new telemetry ingestion pipeline that can handle 2M events/sec today, scale to 10M events/sec (200 satellites) within 18 months, enforce per-constellation data sovereignty at the ingestion layer, and provide burst buffering for solar interference windows without data loss.

### Explicit Requirements

1. Ingest telemetry from 47 satellites (scaling to 200) at up to 2M events/sec sustained, 3M events/sec burst
2. Each ground station operates as an independent ingestion point — no single point of failure
3. EU constellation telemetry (currently Vantage constellation, 11 satellites) must never be processed outside EU infrastructure
4. US ITAR-controlled satellite telemetry must be processed only on US infrastructure
5. End-to-end latency from satellite signal receipt to downstream consumer: < 500ms P99
6. 6-month hot retention, 7-year cold retention for all telemetry
7. Zero data loss guarantee — RPO = 0
8. Consumer lag monitoring must accurately reflect actual processing delay

### Hidden Requirements

1. **Hint: re-read Tomás's DM about Singapore.** What specific orbital window causes the burst? What does this imply about scheduling burst buffer capacity — is it random or predictable?

2. **Hint: re-read James Whitfield's handwritten note attached to the email.** He mentions "solar interference season starts April." What does this mean for the pipeline? Can solar interference be predicted, and should the burst buffer be pre-warmed?

3. **Hint: re-read the Niamh/Preet exchange at 4:55pm.** Restarting ingest workers manually cleared the queue. What does this tell you about the current consumer group configuration — and what does it imply about your partition reassignment strategy under load?

4. **Hint: re-read Tomás's note about 384 partitions and misleading lag metrics.** At 384 partitions, what is the consumer group coordination overhead? Is Kafka the right tool at this partition count, or does the architecture need a different message bus topology?

### Constraints

- **Current load**: 2M events/sec sustained, 3M burst (peak LEO windows)
- **Target load (18 months)**: 10M events/sec (200 satellites)
- **Average event size**: 200 bytes
- **Ground stations**: 12, across US (3), EU (3), Kenya (1), Singapore (1), Australia (2), UK (2)
- **Constellations**: 4 (Kepler — US, Vantage — EU/GDPR, TerraWatch — unclassified US, ArcticLens — UK)
- **Retention**: 6 months hot, 7 years cold
- **Latency SLA**: < 500ms P99 end-to-end
- **Uptime SLA**: 99.99% (52 minutes downtime/year)
- **Team**: 2 SREs, 4 platform engineers, you
- **Budget signal**: Series C dependent — Ingrid wants a "responsible" design, not a blank check

### Your Task

Design the telemetry ingestion pipeline for OrbitCore. Your design must cover: per-ground-station ingestion architecture, the data sovereignty enforcement layer, burst buffering strategy, consumer group topology, retention tiers, and consumer lag monitoring. Address the 200-satellite scaling path explicitly.

### Deliverables

- [ ] Mermaid architecture diagram showing the full ingestion pipeline from ground station to downstream consumer, including sovereignty enforcement zones
- [ ] Database/schema design for telemetry event metadata (not the telemetry payload itself — just the routing, indexing, and sovereignty metadata)
- [ ] Scaling estimation (show math step by step):
  - Events/sec → MB/sec → GB/day → TB/6months
  - Partition count justification at 2M/sec and at 10M/sec
  - Consumer lag monitoring approach and why current approach fails
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs):
  - Per-ground-station Kafka cluster vs shared cluster with sovereignty tagging
  - Kafka vs Pulsar vs proprietary message bus at 10M events/sec
  - Burst buffer design: pre-warmed vs reactive auto-scaling
- [ ] Cost modeling ($X/month estimate):
  - Kafka/Pulsar cluster costs at current and target load
  - Storage costs for 6-month hot + 7-year cold retention
  - Network egress costs for cross-region data movement (and how sovereignty constraints reduce this)
- [ ] Capacity planning (18-month horizon):
  - Satellite count growth curve (47 → 100 → 200)
  - Infrastructure scaling triggers (when do you add ground stations? When do you shard further?)
  - Solar interference season planning — what changes in April?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).

Example structure to consider (not prescriptive):

```
graph TD
  subgraph EU_Zone ["EU Sovereignty Zone"]
    ...
  end
  subgraph US_Zone ["US / ITAR Zone"]
    ...
  end
```
