---
**Title**: LuminaryAI — Chapter 55: ETL to Streaming — Evolving the Data Pipeline
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #data-pipeline #etl #streaming #kafka #cdc #debezium #ml-systems
**Estimated Time**: 3.5 hours
**Related Chapters**: Ch. 24, Ch. 26, Ch. 52, Ch. 54
**Exercise Type**: System Design
---

### Story Context

**SPIKE — Inherited Disaster**

**Monday 9 AM — Slack #incidents**

```
[09:03] Priya Subramaniam [Data Eng]: ALERT: batch pipeline failed at 03:42 UTC.
  The nightly feature computation job did not complete. Current feature vectors
  in Redis are from Sunday night. It is now Monday 9 AM UTC.
  Impact: all clients are serving stale features. Inventory features are
  33 hours old.

[09:08] Jeroen van der Berg [Infra]: Prediction volume is up — Monday morning
  shopping peak. We are serving recommendations based on Sunday inventory state.
  How many products went out of stock between Sunday midnight and now?

[09:11] Priya Subramaniam: Running query...

[09:14] Priya Subramaniam: 14,847 product SKUs across all clients went from
  in-stock to out-of-stock since Sunday midnight. We are actively recommending
  all of them.

[09:15] Jeroen van der Berg: Client SLAs. TechMart's contract has a P1 clause
  for out-of-stock recommendations. If they see this, it's a contract event.

[09:16] You: What broke in the batch pipeline?

[09:19] Priya Subramaniam: The Spark job ran out of memory. The data warehouse
  query pulled 31 days of events instead of 30 — a date boundary bug. The
  extra day pushed the in-memory join past the cluster's memory limit.

[09:22] You: Timeline to fix?

[09:24] Priya Subramaniam: Fix the Spark query + re-run: 2.5 hours minimum.
  Features won't be current until noon UTC. 3 more hours of stale recommendations.

[09:25] Jeroen van der Berg: We cannot wait 3 hours. TechMart peak is 9-11 AM UTC.

[09:27] You: Can we run a partial job? Just inventory features — not the full
  behavioral computation?

[09:29] Priya Subramaniam: Not in the current batch architecture. The job
  computes all features in one pass. We'd need to redesign to support
  partial runs.

[09:30] You: Add that to the list. For now — Jeroen, can we pull live inventory
  data from the clients' product APIs and write directly to Redis while the
  batch job reruns?

[09:33] Jeroen van der Berg: In theory. But we'd need a one-time script and
  someone to verify each client's API response format. We have 47 clients.

[09:35] You: Not all 47. TechMart, RetailCo, and FashionHub are our top 3 by
  volume. How many products do they have?

[09:38] Priya Subramaniam: TechMart: 2.1M SKUs. RetailCo: 840K. FashionHub: 410K.

[09:40] You: Not practical as a one-time script. Let me think.

[09:41] You: Okay. Alternative: can we push the inventory feature update latency
  to 60 seconds by subscribing to client inventory webhook events? Three clients
  have webhook endpoints. That gets us 60% of the problem.

[09:44] Priya Subramaniam: TechMart has a webhook. RetailCo has a Kafka stream
  we can subscribe to. FashionHub uses polling — they send daily exports.

[09:46] You: TechMart webhook + RetailCo Kafka stream. I'll write a consumer.
  We patch Redis directly with inventory status changes. FashionHub we accept
  as stale for now.

[09:48] Jeroen van der Berg: Do it. I'll hold TechMart from contacting us.
  How long to build the consumer?

[09:50] You: 90 minutes for TechMart and RetailCo. FashionHub we'll fix in
  the streaming pipeline redesign.
```

---

**Post-incident retrospective — Monday 4 PM**

**Priya Subramaniam**: The immediate issue is fixed. TechMart received a
patch of inventory updates at 11:23 AM. RetailCo was patched by 11:45.
FashionHub had 412 out-of-stock recommendations we couldn't fix in time.
Their contract team has been notified.

**You**: The root cause isn't the Spark memory bug. That's a symptom. The
root cause is that our entire feature freshness depends on a single nightly
batch job. One failure, and we're blind for hours.

**Dr. Nadia Osei**: We've been discussing moving to streaming for inventory
for six months. Today made the business case.

**You**: We need to redesign in two stages. Stage 1: streaming pipeline for
inventory features, running continuously alongside the batch job. Stage 2:
streaming pipeline for transaction features. Stage 3 — maybe Phase 2 of
this project — streaming behavioral features, which is much harder because
of the 30-day window computation.

**Priya Subramaniam**: Stage 1 is straightforward. Clients with Kafka streams
— subscribe directly. Clients with webhooks — convert to Kafka via an ingestion
service. Clients with polling APIs — Debezium CDC if they give us DB access,
or periodic polling if they don't.

**Jeroen van der Berg**: The ingestion heterogeneity is the hard part. We have
47 clients with 47 different ways of telling us about inventory changes.

**You**: We need an inventory ingestion abstraction layer. Every client,
regardless of how they deliver updates, results in a standardized Kafka topic:
`inventory_updates.{client_id}`. Everything downstream reads from that topic.

---

**Slack DM — Marcus Webb → You, Monday evening**

**Marcus Webb**
Good incident response. You found the workaround, you understood the root cause,
and you didn't confuse fixing the symptom with fixing the problem.

On the ETL-to-streaming evolution: the pattern you're about to design has a
specific failure mode that kills teams. Watch for it.

When you add a streaming pipeline alongside a batch pipeline, you have two
sources of truth writing to the same feature store. For a period of time,
both pipelines will run. The batch job writes full feature vectors. The
streaming pipeline writes deltas. You tested the streaming pipeline in staging.
But staging doesn't have 140 million predictions per day.

The failure mode: the streaming pipeline has a bug. It writes a corrupted
inventory feature for 0.1% of SKUs. The batch pipeline corrects it every
night. So the bug is only visible for a few hours per day — until Monday,
when the batch job fails again. Then the corrupted 0.1% is corrupted for
33 hours, and no one notices because the overall error rate was "low."

Build monitoring for feature freshness AND feature correctness from day one.
Not later. Not when there's time. Day one of the streaming pipeline.

---

### Problem Statement

LuminaryAI's batch-only ETL pipeline is a single point of failure for feature
freshness. A nightly batch job failure leaves 14,000+ out-of-stock products
actively recommended for hours, causing client SLA violations. The pipeline
must be redesigned to support streaming ingestion for inventory and transaction
features, with the batch pipeline retained as a full-vector refresh. The
heterogeneous client data delivery methods (Kafka streams, webhooks, polling)
must be normalized into a standard ingestion abstraction.

### Explicit Requirements

1. Inventory features must be updated within 60 seconds of a client inventory
   event, regardless of how the client delivers that event
2. Transaction features must be updated within 5 minutes of a purchase event
3. The batch pipeline must continue running nightly as a full-vector refresh
   (correctness baseline)
4. All client data delivery methods (Kafka, webhook, polling) must be normalized
   to a standard internal Kafka topic per client
5. The batch job must support partial runs — inventory feature recompute
   independently of behavioral feature recompute
6. Feature correctness monitoring must detect corrupted or stale feature vectors

### Hidden Requirements

- **Hint**: Marcus Webb raised the "two sources of truth" failure mode.
  The streaming pipeline writes deltas; the batch pipeline writes full vectors.
  If they run concurrently, what happens at the moment the batch job writes?
  It will overwrite the streaming pipeline's most recent delta with a
  30-minutes-old value (the batch job takes 4 hours; by the time it writes,
  its data is stale relative to the streaming pipeline). What is the write
  ordering strategy? Should the batch job skip writing fields that were
  recently updated by the streaming pipeline?

- **Hint**: Priya mentioned FashionHub uses "daily exports" — they send a
  CSV file once per day with their full inventory. This is not a streaming
  feed and not a webhook. For FashionHub, the 60-second inventory SLA is
  impossible to meet. How does the system handle clients that cannot
  meet the streaming SLA? Is there a tiered freshness SLA — different
  guarantees for different client data delivery capabilities?

- **Hint**: The incident was triggered by a date boundary bug in the Spark
  query ("31 days instead of 30"). The fix is to patch the query. But this
  class of bug — off-by-one in time windows — is common in data pipelines.
  What testing approach would have caught this? How do you write a pipeline
  that is robust to date boundary edge cases?

### Constraints

- **Client count**: 47 clients
- **Client data delivery methods**: 18 with Kafka streams, 21 with webhooks,
  8 with polling/batch exports
- **Inventory freshness SLA**: 60 seconds for Kafka/webhook clients;
  best-effort for polling clients
- **Prediction volume during incident window**: ~4,000/sec (morning peak)
- **Batch job window**: 2-4 AM UTC (4-hour window); must complete before
  morning peak
- **Feature store Redis memory**: estimated from Ch. 54 sizing exercise

### Your Task

Design the streaming ingestion pipeline that normalizes heterogeneous client
data delivery into a standard Kafka topic, supports delta updates to the
feature store alongside the batch pipeline, and includes feature freshness
monitoring.

### Deliverables

- [ ] **Ingestion abstraction architecture** (Mermaid) — show how Kafka,
  webhook, and polling clients are all normalized to `inventory_updates.{client_id}`
  and `transaction_updates.{client_id}` topics. What is the ingestion
  service responsible for?

- [ ] **Streaming pipeline design** — the consumer that reads from the
  normalized topics and writes delta updates to the feature store. What
  is the consumer's idempotency key? How does it handle duplicate events
  from the same client?

- [ ] **Batch + streaming coexistence strategy** — how do batch full-vector
  writes and real-time delta writes avoid producing inconsistent feature
  vectors? Extend the versioned write design from Ch. 54 to handle this
  specific scenario: batch job finishes writing at 6 AM; streaming pipeline
  was writing deltas throughout the night. What is the merge strategy?

- [ ] **Partial batch job redesign** — how to restructure the batch pipeline
  so that inventory feature recompute can run independently of behavioral
  feature recompute. What is the dependency graph? What jobs can run in
  parallel?

- [ ] **Feature freshness monitoring** — define the metrics and alerts:
  feature_age_seconds per client, stale_feature_count (features older than
  SLA), feature_update_lag (streaming pipeline delay). What is the alerting
  threshold for each? Who gets paged?

- [ ] **Tiered SLA design** — define the freshness SLA tiers:
  Tier 1 (Kafka/webhook, < 60s), Tier 2 (hourly polling, < 2 hours),
  Tier 3 (daily export, < 36 hours). How does the client contract reflect
  these tiers? How does the serving layer signal which tier a feature came from?

- [ ] **Tradeoff analysis** — minimum 3 tradeoffs:
  1. Streaming pipeline + batch pipeline (dual path) vs full streaming
     replacement (no batch at all)
  2. Per-client normalized Kafka topic vs shared topic with client_id field
  3. Debezium CDC for polling clients (requires DB access, more real-time)
     vs periodic polling (simpler, more latency)

### Diagram Format

```mermaid
graph TB
  subgraph "Client Data Delivery (Heterogeneous)"
    K_CLIENTS[18 Kafka clients\ndirect streams] --> KAFKA_NORM
    WH_CLIENTS[21 Webhook clients\nHTTP push] --> WH_SVC[Webhook ingestion\nservice]
    WH_SVC --> KAFKA_NORM
    POLL_CLIENTS[8 Polling/Export clients] --> POLL_SVC[Polling service\nScheduled fetch]
    POLL_SVC --> KAFKA_NORM
  end
  subgraph "Normalized Internal Topics"
    KAFKA_NORM[Kafka\ninventory_updates.{client_id}\ntransaction_updates.{client_id}]
  end
  subgraph "Feature Update Pipelines"
    KAFKA_NORM --> RT_PIPE[Real-time consumer\nDelta updater]
    DW[(Data Warehouse)] --> BATCH[Nightly Spark\nFull vector compute]
    RT_PIPE --> REDIS[(Redis\nFeature Store)]
    BATCH --> REDIS
  end
  subgraph "Monitoring"
    REDIS --> FRESHNESS[Feature freshness\nmonitor]
    FRESHNESS --> ALERT[PagerDuty\nalerts]
  end
```
