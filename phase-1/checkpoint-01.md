---
**Title**: Checkpoint 1 — Mid → Senior Interview @ Helios Systems
**Level**: Senior
**Difficulty**: 8
**Tags**: #checkpoint #interview #system-design #promotion
**Estimated Time**: 3 hours (treat as a real interview session)
**Related Chapters**: Ch. 1–16
**Exercise Type**: Promotion Checkpoint
---

> **This is a high-stakes system design interview.** Treat it as real.
> No hints until you've attempted the design. The brief is intentionally ambiguous.
> You must ask clarifying questions as part of the exercise.

---

### Story Context

**The job listing you applied to — 3 weeks ago**

```
Helios Systems — Senior Backend Engineer, Platform
Location: Remote / Sydney preferred
Stack: Node.js, PostgreSQL, Redis, Kafka

We're building the financial infrastructure layer for Southeast Asian SMEs.
Think: multi-currency accounts, payment rails, treasury management.
Fast-moving team. High standards. Real production load from day one.

Apply: [form link]
```

You applied. Two technical screens. Now you're at the final round: a 3-hour
system design interview with the engineering panel.

---

**Zoom call — 10:00 AM, Thursday**

The interview begins. There are three people on the panel:

**Interviewer 1: Preethi Nair** — Staff Engineer, Helios Systems. Crisp, direct.
She runs the interview.

**Interviewer 2: Tomas Herrera** — Senior Engineer. Takes notes. Asks follow-up
questions about scaling math.

**Interviewer 3:** Camera off. Name not shown yet.

**Preethi**: Thanks for joining. We'll skip intros and dive straight in. We have
three hours. Here's your brief:

> **"Design a multi-currency payment ledger system for SMEs."**

That's it. What questions do you have?

---

**Interview transcript (partial — you fill in the rest)**

**You**: Before I start designing, I'd like to ask some clarifying questions.

**Preethi**: Please do.

**You**: First — when you say "multi-currency," how many currencies? Are we talking
10 major currencies or 180+ ISO 4217 currencies?

**Preethi**: Good question. Start with 15 currencies: SGD, MYR, PHP, THB, IDR, VND,
USD, EUR, GBP, AUD, JPY, CNY, HKD, KRW, INR. But architect for extensibility.

**You**: What's the read-to-write ratio? Are SMEs primarily initiating payments,
or do they also need reporting, reconciliation, and auditing?

**Tomas**: Both. Heavy read for reporting. They reconcile end-of-day. The write
path is the payment initiation and settlement.

**You**: Scale — how many SMEs, how many transactions per day?

**Preethi**: 50,000 SMEs at launch, targeting 500,000 within 18 months.
Average 200 transactions per SME per month. Peak daily volume — think about it
in terms of business days.

**You**: Compliance requirements? Financial data in Southeast Asia often has
regulatory requirements by country.

**Preethi**: Good catch. Singapore MAS guidelines. PCI-DSS if we're handling card
data. Each SME operates in at least one jurisdiction; some operate across multiple.

*[Preethi turns to the third panelist.]*

**Preethi**: Do you have anything to add before they begin?

*[The third interviewer turns on their camera.]*

**Interviewer 3**: Yeah. Just one thing.

*[It's Marcus Webb.]*

**Marcus Webb**: Don't forget about currency precision. I've seen two fintech
companies blow up their ledger because of floating point. How are you representing
monetary amounts?

---

### The Design Challenge

You have the rest of the session. Design the system. Preethi, Tomas, and Marcus
Webb will probe your decisions.

**The brief (refined from clarifying questions)**:
Design a multi-currency payment ledger system for 50,000–500,000 SMEs processing
an average of 200 transactions/month each (peak ~33 transactions/business day/SME).
The system must support 15 currencies, MAS compliance, PCI-DSS scope awareness,
end-of-day reconciliation, and extensibility for new currencies and jurisdictions.

---

### Explicit Requirements

1. Store monetary amounts without floating-point precision loss
2. Support 15 currencies with extensibility for more; exchange rate storage and
   FX conversion audit trail
3. Double-entry bookkeeping: every transaction must balance (debits = credits)
4. Idempotency on all transaction writes (payment processing is at-least-once)
5. End-of-day reconciliation: each SME can generate a statement with opening balance,
   all transactions, and closing balance for any date range
6. MAS compliance: transaction records retained for 5 years; audit trail of all
   access to financial data
7. Multi-tenant isolation: SME A must never see SME B's data

### Clarifying Questions You Must Ask First

Before designing, write out the clarifying questions you would ask. The interview
answers are provided above, but practice formulating the questions yourself:

- Volume and scale (DAU, RPS, growth rate)
- Consistency requirements (strong vs eventual)
- Read vs write ratio
- Compliance jurisdiction
- Integration requirements (banking rails, card networks)
- Recovery time objective (RTO) if the system fails

### Hidden Requirements

These were embedded in the story. Did you catch them?

- **Marcus Webb's comment about floating point**: All monetary amounts must be
  stored as integers in the smallest currency unit (e.g., cents for USD, fils for AED).
  Never use FLOAT or DOUBLE for money. Hint: what PostgreSQL type do you use?
- **"Each SME operates in at least one jurisdiction; some operate across multiple"**:
  Multi-jurisdiction means multi-currency *accounts* for a single SME — an SME
  in Singapore might hold SGD, USD, and MYR balances simultaneously. Your
  account model must support this.
- **"End-of-day reconciliation"**: This is a heavy read at the same time every
  day — all 50,000 SMEs reconciling end of business. What does that spike look
  like on your infrastructure, and how do you handle it?

### Constraints

- **SMEs at launch**: 50,000; 18-month target: 500,000
- **Transactions/month/SME**: 200 average
- **Peak daily transactions**: 200/month ÷ 22 business days ≈ 9 transactions/SME/day
  → 50,000 SMEs × 9 = 450,000 transactions/day at launch; 4.5M/day at scale
- **Currencies**: 15 at launch, extensible
- **Retention**: 5 years (MAS requirement)
- **Reconciliation spike**: Assume 40% of SMEs reconcile in the same 30-minute window
  at day's end — that's 20,000 concurrent reconciliation queries

### Your Task

Produce a complete system design. The panel will probe:
1. Your ledger data model (schema, double-entry, currency precision)
2. Your transaction write path (idempotency, consistency)
3. How you handle the reconciliation spike
4. Your FX/exchange rate design
5. How you'd scope for MAS compliance

### Deliverables

- [ ] **Mermaid architecture diagram** — complete system: API → transaction service
  → ledger DB → reconciliation service → reporting layer
- [ ] **Ledger schema** — `accounts`, `transactions`, `ledger_entries` tables with
  column types (pay attention to monetary types and currency codes)
- [ ] **Double-entry verification query** — SQL query to verify that all ledger
  entries for a given transaction sum to zero
- [ ] **Reconciliation design** — how do you handle 20,000 concurrent end-of-day
  reconciliation queries without killing the primary DB?
- [ ] **Scaling math** — at 4.5M transactions/day: RPS, write throughput, 5-year
  storage requirements
- [ ] **Tradeoff analysis** — minimum 3 tradeoffs:
  1. Storing FX rates in your DB vs using a real-time FX API on every conversion
  2. NUMERIC(19,4) vs integer cents for monetary storage
  3. Separate reconciliation read replica vs materialized view strategy

---

### What Marcus Webb is Looking For

After the design session, as the call is wrapping up, Marcus Webb turns his camera
back on and says one thing:

**Marcus Webb**: Good. One final question, and you don't have to answer it now —
but I want you to think about it after:

> "You designed a ledger. Every transaction is a row. At 4.5 million transactions
> per day, your ledger has 1.6 billion rows per year. At 5 years — 8 billion rows.
> How do you query an SME's balance in under 100ms on an 8-billion-row table?"

That question is the beginning of the next chapter of your career. Think about it.

*[The call ends. You've passed.]*
