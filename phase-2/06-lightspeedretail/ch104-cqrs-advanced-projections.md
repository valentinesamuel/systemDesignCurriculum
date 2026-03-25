---
**Title**: LightspeedRetail — Chapter 104: CQRS Advanced Projections — Real-Time Inventory
**Level**: Staff
**Difficulty**: 8
**Tags**: #cqrs #event-sourcing #inventory #projections #eventual-consistency #snapshot
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 66 (Stratum CQRS intro), Ch. 77 (TradeSpark CQRS), Ch. 99, Ch. 100
**Exercise Type**: System Design / Performance Optimization
---

### Story Context

**#store-operations** — Slack, Thursday November 1, 08:14

**[08:14] Keiko Tanaka** (Regional Manager, APAC): URGENT. Tokyo flagship. Customer purchased the last 3 units of Hermès scarf (SKU: HE-SC-2847-PLAT). The system showed 47 in stock. The stockroom had 3. We don't know where the other 44 went. Customer is waiting.

**[08:16] Ji-woo Kim** (Store Systems Lead): I'm looking at the inventory audit log. SKU HE-SC-2847-PLAT. Inventory table says 47. But there are 23 transactions in the last 4 hours that sold units of that SKU. I count 44 deductions. Plus 3 just now = 47 total sold. The inventory table never decremented.

**[08:21] [you]**: Are you saying the inventory read is returning stale data?

**[08:23] Ji-woo Kim**: Yes. The inventory table and the transactions table are in the same Postgres instance. Under high write load, the inventory updates are lagging. We're doing an UPDATE on `inventory` for every transaction — and those UPDATEs are losing to the transaction INSERT throughput under Black Friday write load. Specifically: we have a SELECT FOR UPDATE on the inventory row at transaction time. Under high concurrency, the lock wait queue is 2-3 seconds long. Some UPDATEs are being dropped when the lock times out.

**[08:27] [you]**: Dropped silently?

**[08:29] Ji-woo Kim**: They're logged as errors in the app layer but there's no compensating mechanism. The inventory count is wrong and no one recovers it.

**[08:33] Beatriz Santos**: This is the Black Friday nightmare. If a customer buys 1 item and the inventory doesn't decrement, we oversell. Tokyo is a canary for what happens to us globally on Black Friday.

**[08:36] [you]**: I need to redesign the inventory system. Not patch it.

**[08:38] Beatriz Santos**: You have 26 days.

---

**Design Session — [you] and Marcus Delgado** — Thursday November 1, 14:00

**Marcus Delgado**: Walk me through what you're thinking.

**[you]**: The core problem is that inventory is a mutable counter in the same Postgres table that's serving both high-throughput writes (transactions) and high-throughput reads (inventory checks for 120k terminals). These two workloads have completely different consistency and performance requirements.

**Marcus Delgado**: CQRS.

**[you]**: Yes. Command side: every sale, every restock, every return becomes an inventory event — appended to an event log. No UPDATE on a shared counter. Query side: a materialized projection of current stock levels, rebuilt from the event log. The projection can be eventually consistent with a known, bounded lag.

**Marcus Delgado**: How bounded?

**[you]**: Under normal load, sub-second. Under Black Friday peak, we target under 3 seconds. The key insight is that a 3-second lag on inventory reads is acceptable — cashiers can be shown a "sold in last 3 seconds" indicator — but a dropped UPDATE that corrupts the count is not acceptable.

**Marcus Delgado**: What triggers the projection rebuild?

**[you]**: Event consumer. Each inventory event triggers a projection update. The projection is a Redis hash: `inventory:{sku_id}:{store_id}` → `{quantity: N, last_event_id: UUID}`. The `last_event_id` ensures idempotent projection updates.

**Marcus Delgado**: Multiple projections?

**[you]**: At least three. The stock level projection — what we just described. A low-stock alert projection — triggers reorder notifications when quantity drops below threshold. A slow-mover projection for the analytics warehouse — daily rollup by SKU for merchandising reports.

**Marcus Delgado**: Same event stream feeds all three?

**[you]**: Yes. That's the elegance. The event log is the source of truth. Projections are derived views. If a projection gets corrupted — or we need a new report — we replay the event log from any point.

**Marcus Delgado**: What about the Tokyo scenario? 44 sold, inventory shows 47?

**[you]**: Under the new model: each sale emits an `InventoryDecrement` event. Those 44 events are durably committed to the event log before any projection update. Even if the projection consumer crashes, the events are there. Replay = correct inventory.

---

**Slack DM — Marcus Webb → [you]** — Thursday November 1, 17:41

**Marcus Webb**: Heard you're redesigning inventory for Tokyo. What's the approach?

**[you]**: CQRS with event log as source of truth. Materialized projection in Redis for read path.

**Marcus Webb**: Good. Watch out for snapshot stale reads. What happens when a terminal queries inventory and the projection consumer is 15 seconds behind due to a partition on the event consumer?

**[you]**: Return the stale value with a lag indicator. The terminal UI shows "Stock count as of N seconds ago."

**Marcus Webb**: That's a UX decision. Has Beatriz approved it?

**[you]**: Not yet.

**Marcus Webb**: Go get that approval before you build the technical spec. The business needs to decide: is it acceptable to show a cashier a stock count that might be 15 seconds stale? Because if the answer is no — if they want strong consistency on every inventory read — then Redis projection is the wrong choice. You'd need synchronous event processing in the transaction commit path, which kills your throughput advantage.

**[you]**: You're right. Let me check with Beatriz.

**Marcus Webb**: And what's your snapshot strategy? If you have 18 months of inventory events for 500,000 SKUs × 120,000 stores, you're not replaying from the beginning every time a projection consumer restarts.

**[you]**: Periodic snapshots. Snapshot current state every 1,000 events or 24 hours, whichever comes first. Replay only from the last snapshot.

**Marcus Webb**: Good. Make sure the snapshot write is atomic with the last event ID it represents. You'll need that for exactly-once projection rebuild.

---

This is the most advanced CQRS design you have built. At Stratum (Ch. 66) you encountered CQRS as a concept while debugging a read-replica lag problem. At TradeSpark (Ch. 77) you applied it to financial order book projections with strict latency requirements. Here the domain is physical retail inventory — where the consequence of a stale read is not just a bad dashboard metric but a luxury customer standing at a counter in Tokyo while a manager scrambles to explain why the system said 47 and the stockroom held 3.

### Problem Statement

LightspeedRetail's inventory management system uses a mutable counter in Postgres (`SELECT FOR UPDATE` on inventory rows at transaction time). Under high write load — specifically during Black Friday simulation — lock wait queues cause UPDATE operations to time out and be silently dropped. The result: inventory counts diverge from reality. Tokyo flagship oversold 44 units of a luxury item because the inventory counter failed to decrement.

You must redesign inventory management using CQRS: the command side emits immutable inventory events to an event log; the query side maintains multiple materialized projections derived from the event stream. The design must provide bounded-lag eventual consistency on reads, support multiple projection consumers, handle projection rebuild from snapshots, and define exactly what consistency guarantee is offered to the cashier UI.

### Explicit Requirements

1. All inventory mutations (sale, restock, return, adjustment) emit immutable events to an event log — no direct UPDATE on a shared counter.
2. At least three projections: stock level (real-time), low-stock alert (threshold-based), slow-mover analytics (daily rollup).
3. Stock level projection must be eventually consistent with a maximum bounded lag of 3 seconds under normal load, 10 seconds under Black Friday peak.
4. Idempotent projection updates: each event must be applied exactly once per projection, even if the consumer restarts.
5. Projection rebuild must be possible from a snapshot + replay, not from full event log history.
6. Snapshots taken every 1,000 events or every 24 hours (whichever is first) per store+SKU combination.
7. Terminal UI must display lag indicator when stock count is > 3 seconds stale.
8. All inventory events must be retained for 7 years (audit requirement).

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's question: "What happens when a terminal queries inventory and the projection consumer is 15 seconds behind?" Marcus asked you to get business approval. There is a hidden requirement: you must define and document the consistency SLA as a business decision, not a technical default. The schema must include a `consistency_policy` flag per product category (luxury items vs commodity items may have different tolerances).

- **Hint**: Re-read Ji-woo Kim: "some UPDATEs are being dropped when the lock times out." The root cause was `SELECT FOR UPDATE` lock timeout. Under CQRS, you no longer use SELECT FOR UPDATE. But the command side still needs to prevent inventory going below zero. How do you enforce non-negative inventory without a lock? This is the hidden requirement: optimistic validation at the command layer (check projection before emitting event, accept some race condition risk) vs pessimistic validation (serialize writes per SKU via a single-writer queue).

- **Hint**: Re-read "500,000 SKUs × 120,000 stores." That is 60 billion possible combinations. Most will never have inventory events. The hidden requirement is sparse storage: the event log and projection storage must only contain SKU+store pairs that have actual inventory, not pre-allocate all 60B combinations.

- **Hint**: Re-read "a slow-mover projection for the analytics warehouse — daily rollup by SKU for merchandising reports." The hidden requirement is that this projection feeds the data warehouse designed in Ch. 103. The two systems must share the event schema — inventory events must be readable by the warehouse ETL pipeline without custom transformation.

### Constraints

- **Scale**: 120,000 stores, 500,000 active SKUs (2,000-8,000 per store)
- **Transaction rate**: 40M transactions/day normal; 200M on Black Friday
- **Inventory event rate**: ~40M events/day (same as transaction rate)
- **Projection consumer lag budget**: 3 seconds normal, 10 seconds peak
- **Snapshot frequency**: per store+SKU: every 1,000 events or 24h
- **Event retention**: 7 years
- **Stock level projection storage**: Redis (hot), Postgres (warm/cold)
- **Event log storage**: Kafka (retention: 7 days hot) + S3 (7-year archive via Kafka Connect)
- **Concurrent projection consumers**: 3 (stock level, low-stock alert, analytics)
- **Team**: 3 backend engineers, 1 data engineer

### Your Task

Design the complete CQRS inventory architecture with event log, multiple projection consumers, snapshot strategy, and consistency policy framework. Produce the event schema and projection schemas.

### Deliverables

- [ ] **Mermaid architecture diagram** showing: transaction service → inventory command handler → event log (Kafka) → three projection consumers → Redis (stock level) + Postgres (low-stock threshold + snapshot) + S3/Warehouse (analytics). Include snapshot writer and replay path.

- [ ] **Inventory event schema** (TypeScript):
  ```typescript
  type InventoryEventType =
    | 'InventoryDecrement'   // sale
    | 'InventoryIncrement'   // restock
    | 'InventoryReturn'      // customer return
    | 'InventoryAdjustment'  // manual correction
    | 'InventorySnapshot';   // periodic snapshot

  interface InventoryEvent {
    event_id:        string;   // UUIDv7 (time-sortable)
    event_type:      InventoryEventType;
    store_id:        string;
    sku_id:          string;
    delta:           number;   // positive = increment, negative = decrement
    resulting_qty:   number;   // denormalized for fast projection rebuild
    transaction_id:  string | null;  // null for adjustments
    emitted_at:      string;   // ISO 8601 UTC
    sequence_no:     number;   // monotonic per store+sku partition
  }
  ```

- [ ] **Stock level projection design**:
  - Redis key: `inv:stock:{store_id}:{sku_id}` (HASH)
  - Fields: `quantity`, `last_event_id`, `last_updated_at`, `lag_ms`
  - Consumer logic: compare incoming `event.sequence_no` against stored `last_sequence_no`. If equal+1, apply. If less, skip (duplicate). If greater than +1, fetch missed events and apply in order.

- [ ] **Snapshot schema**:
  ```sql
  inventory_snapshots (
    snapshot_id     UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
    store_id        UUID          NOT NULL,
    sku_id          UUID          NOT NULL,
    quantity        INT           NOT NULL,
    sequence_no     BIGINT        NOT NULL,  -- last event included in this snapshot
    snapshotted_at  TIMESTAMPTZ   NOT NULL,
    UNIQUE(store_id, sku_id, sequence_no),
    INDEX(store_id, sku_id, snapshotted_at DESC)
  )
  ```
  Show: projection rebuild query (load latest snapshot, replay events with `sequence_no > snapshot.sequence_no`).

- [ ] **Non-negative inventory enforcement design**: Compare two approaches:
  - Optimistic: emit event, check projection after. If resulting_qty < 0, emit compensating `InventoryAdjustment`. What is the race condition window?
  - Pessimistic: serialize writes per store+SKU via a Redis-backed single-writer queue. What is the throughput cost?
  Define which approach you choose and why, with numerical justification.

- [ ] **Consistency policy schema** (per product category):
  ```sql
  product_consistency_policy (
    category_id           UUID    PRIMARY KEY,
    max_stale_seconds     INT     NOT NULL DEFAULT 3,
    show_lag_indicator    BOOL    NOT NULL DEFAULT TRUE,
    oversell_tolerance    INT     NOT NULL DEFAULT 0,  -- luxury: 0, commodity: 5
    validation_mode       ENUM('optimistic','pessimistic') NOT NULL DEFAULT 'optimistic'
  )
  ```

- [ ] **Scaling estimation**:
  - Event log throughput: 40M events/day ÷ 86,400 sec = 463 events/sec average. Black Friday: 200M/day = 2,315 events/sec. Show Kafka partition strategy (partition by `store_id` hash to maintain per-store ordering).
  - Redis memory: active 2,000 SKUs/store × 120,000 stores = 240M active keys. At ~100 bytes per key = 24GB. Show how to tier inactive SKUs to Postgres.
  - Snapshot storage: estimate rows per day and 36-month projection.

- [ ] **Tradeoff analysis** (minimum 3):
  1. CQRS eventual consistency vs synchronous inventory decrement: synchronous is simpler and strongly consistent but SELECT FOR UPDATE serializes all sales for a hot SKU. At what sales rate per SKU does CQRS become necessary?
  2. Redis projection vs Postgres projection: Redis gives sub-millisecond reads but is volatile (data loss on crash without persistence). How do you configure Redis to be durable enough for inventory without sacrificing performance?
  3. Multiple projection consumers vs a single multiplex consumer: separate consumers are independently scalable and fault-isolated, but each adds operational overhead. Define the criteria for splitting into a separate consumer vs adding to an existing one.
