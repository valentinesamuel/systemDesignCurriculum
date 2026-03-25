---
**Title**: NovaPay — Chapter 1: Design the Payment Processing Pipeline
**Level**: Mid
**Difficulty**: 4
**Tags**: #payments #api-design #database #idempotency #distributed
**Estimated Time**: 2 hours
**Related Chapters**: Ch. 2, Ch. 5, Ch. 34, Ch. 56
**Exercise Type**: System Design
---

### Story Context

**#payments-core — Slack, Day 1, 9:14 AM**

**Dani Osei** [9:14 AM]
Welcome to NovaPay. Before you get badge access sorted, I need a brain on this.
We're re-designing our payment initiation flow. Our current system is a 3-year-old
monolith that some contractor built. It works, mostly, but we're about to onboard
Kwik — Nigeria's largest e-commerce marketplace. 200k merchants, ~$40M GMV/day at
peak. We cannot afford to fail.

**Dani Osei** [9:15 AM]
Can you join the architecture session at 11am? I'll share the Confluence doc.
No prep needed — just come ready to think.

**You** [9:17 AM]
I'll be there.

---

**#payments-core — 11:02 AM (Architecture Session Notes, transcribed by Carlos Reyes)**

**Dani**: Okay, the current flow is dead simple. Merchant calls `/charge`, we hit the
bank API, we write to Postgres, we return success. Single service, synchronous all
the way down.

**Carlos Reyes (PM)** [11:03 AM]: The problem is Kwik sends us payment bursts. During
flash sales — think 11/11, Black Friday equivalents — we see 8,000 requests per minute.
Our current system falls over at around 1,200 RPM. We've been manually throttling Kwik
to 200 RPM. They noticed. Their CTO emailed Priya directly.

**Dani**: We need a new pipeline. Async processing, proper queue, status callbacks.
The whole thing. And it needs to be PCI-compliant by Q3 because we're going for
SOC2 Type II.

**Priya Sharma (CTO)** [11:09 AM]: One more thing. We had a double-charge incident
last month. Merchant hit retry on their end, our system processed it twice. Refunds
cost us $14,000 in fees and one merchant almost churned. Idempotency is not optional.

**Carlos** [11:11 AM]: Oh, and Kwik wants a webhook when payment status changes.
Their ops team manually checks our dashboard right now. It's embarrassing.

**Dani** [11:14 AM]: Also — their payments come in three currencies: NGN, KES, GHS.
We need to store the original currency AND settle in USD. Two amounts, two currencies,
immutable once written. Audit trail is non-negotiable.

---

**Slack DM — Marcus Webb → You, 11:47 AM**

**Marcus Webb**
Heard you're the new hire they threw into the Kwik problem. Good luck.
One question before you design anything: where does money live in your system?
Not the DB row. The *concept* of money. Answer that first. Everything else follows.

**You** [11:52 AM]
Not sure I follow?

**Marcus Webb** [11:53 AM]
You will. Think about it. Then think about what happens if your service crashes
between "bank says yes" and "you write the DB row."

---

**Confluence Doc: Kwik Integration Requirements (shared by Dani, 10:58 AM)**

```
- POST /v1/payments — initiate a payment
- GET /v1/payments/{id} — check payment status
- Webhook delivery: PENDING → PROCESSING → COMPLETED | FAILED | REVERSED
- Idempotency key required on all POST requests (Kwik provides it)
- Response SLA: < 200ms for initiation acknowledgement
- Webhook retry: at least 3 attempts with exponential backoff
- Currency: store original (NGN/KES/GHS) + settlement amount (USD)
- Audit log: every state change must be recorded with timestamp + actor
```

---

### Problem Statement

NovaPay's existing synchronous payment API cannot handle Kwik's peak load of 8,000
RPM and has already caused double-charge incidents. You need to design a new async
payment processing pipeline that decouples payment initiation from bank API execution,
guarantees idempotency, delivers webhook notifications, and maintains a complete audit
trail — all while meeting a < 200ms acknowledgement SLA.

### Explicit Requirements

1. Accept payment requests via `POST /v1/payments` with an idempotency key
2. Acknowledge receipt within 200ms (async — return a `payment_id` and `PENDING` status)
3. Process payments asynchronously via a queue (bank API calls happen out-of-band)
4. Deliver webhook notifications for every state transition: `PENDING → PROCESSING → COMPLETED | FAILED`
5. Support webhook retry with at least 3 attempts and exponential backoff
6. Store both original currency amount and USD settlement amount
7. Maintain an immutable audit log of every state change with timestamp and actor
8. Idempotency: same idempotency key must return the same result, never double-process

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's DM. What happens if your worker crashes *after*
  the bank API returns success but *before* you write to the database? How do you
  guarantee the outcome is recorded? (Hint: think about at-least-once delivery and
  idempotency together.)
- **Hint**: Carlos mentioned "two amounts, two currencies, immutable once written."
  What does *immutable* mean for your DB schema? Can you UPDATE a payment row, or
  must you use append-only writes?
- **Hint**: Priya said the previous double-charge cost $14,000 in fees. That implies
  the bank API *does not* have its own idempotency guard. Your system must be the
  last line of defense.

### Constraints

- **DAU**: 200,000 active merchants (Kwik alone); total platform ~400,000
- **Peak RPS**: ~135 RPS average, peaks at 8,000 RPM (~133 RPS) with Kwik flash sales
- **P99 initiation latency**: < 200ms
- **Bank API response time**: 800ms–3s (variable, external)
- **Webhook delivery SLA**: best-effort, within 30 seconds of state change
- **Storage**: payments data retained for 7 years (regulatory requirement)
- **Team size**: 3 backend engineers (including you), 1 DevOps
- **Infrastructure**: AWS, PostgreSQL 14, Redis, existing BullMQ setup
- **Budget constraint**: solution must run on current infra, no new managed services
  without VP approval

### Your Task

Design the complete async payment processing pipeline from API ingress to bank API
execution to webhook delivery. Your design must address idempotency at every layer,
define the exact state machine for a payment, and ensure no payment is lost even if
any single component crashes.

### Deliverables

- [ ] **Mermaid architecture diagram** — show the full pipeline: API → queue → worker
  → bank API → state update → webhook queue → webhook delivery
- [ ] **Payment state machine diagram** (Mermaid stateDiagram-v2) — all states and
  valid transitions
- [ ] **Database schema** — `payments` table and `payment_audit_log` table with column
  types, indexes, and constraints
- [ ] **Scaling estimation** — show math: at peak 8,000 RPM, how many queue workers
  do you need? What's your queue depth at peak?
- [ ] **Tradeoff analysis** — minimum 3 explicit tradeoffs:
  1. Sync vs async initiation
  2. At-least-once vs exactly-once delivery
  3. Single queue vs separate queues for payment processing and webhook delivery

### Diagram Format

All architecture diagrams must use Mermaid syntax.

```mermaid
graph LR
  A[Merchant API] --> B[POST /v1/payments]
  %% complete this diagram
```

### Code Task

Write the TypeScript interface for the payment initiation request and the payment
record stored in the database. Include the idempotency key, both currency fields,
and the state machine status enum.

```typescript
// Sketch only — not a full implementation
enum PaymentStatus {
  // ...
}

interface PaymentInitiationRequest {
  // ...
}

interface PaymentRecord {
  // ...
}
```
