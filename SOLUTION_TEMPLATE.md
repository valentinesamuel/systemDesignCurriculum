# Chapter XX — [Title]

**Company**: &nbsp; | &nbsp; **Phase**: &nbsp; | &nbsp; **Date Attempted**:

---

## My Understanding of the Problem

*(2–3 sentences — restate the problem in your own words before designing. Do not copy the Problem Statement verbatim. Paraphrasing forces you to actually understand it.)*

---

## Assumptions I'm Making

*(List explicit assumptions — don't skip this. This is what separates Staff from Senior. If you don't state your assumptions, your architecture is built on air.)*

-
-
-

---

## Architecture Diagram

*(ASCII diagram or link to Excalidraw / Miro / draw.io export. A photo of a whiteboard is fine. The point is: draw it before you describe it.)*

```
[paste ASCII diagram here, or replace with: [Diagram](link-to-your-diagram)]
```

---

## Core Components

*(Brief description of each major service or component in your design. One paragraph or bullet per component — not an essay, not a one-liner.)*

| Component | Responsibility | Technology Choice | Why |
|-----------|---------------|------------------|-----|
| | | | |
| | | | |
| | | | |

---

## Database Schema

*(Key tables / collections with field types and indexes. You don't need every field — focus on the ones that drive your design decisions.)*

```sql
-- Example format:
CREATE TABLE payments (
  id          UUID PRIMARY KEY,
  merchant_id UUID NOT NULL,
  amount      BIGINT NOT NULL,         -- cents
  status      TEXT NOT NULL,           -- 'pending' | 'settled' | 'failed'
  idempotency_key TEXT UNIQUE NOT NULL,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_payments_merchant_status ON payments (merchant_id, status);
```

---

## Scaling Estimation

*(Show your math. Don't just state numbers — derive them.)*

| Metric | Estimate | Reasoning |
|--------|----------|-----------|
| DAU / MAU | | |
| Peak RPS (reads) | | |
| Peak RPS (writes) | | |
| Storage / year | | |
| Cache hit rate target | | |
| Replication factor | | |
| Bandwidth | | |

**Cost estimate** *(Phase 2+)*: $___/month
*(If this is a Phase 2 or Phase 3 chapter, include a rough infrastructure cost estimate.)*

---

## Trade-offs

*(Minimum 3 explicit trade-offs. For each: what you chose, what you rejected, and the honest reason why — including what you'd lose by choosing the rejected option.)*

| Decision | Chose | Rejected | Why | What You'd Lose |
|----------|-------|----------|-----|----------------|
| | | | | |
| | | | | |
| | | | | |

---

## Hidden Requirements I Found

*(List any requirements buried in the story context — in Slack messages, email chains, incident logs, or character dialogue. If you found none, re-read the story. They're always there.)*

-
-

---

## Compliance & Security Considerations

*(Only fill this in if the chapter touches compliance. Delete the section if not relevant.)*

- **Regulation(s) in scope**:
- **Key architectural impact**:
- **Audit trail requirements**:
- **Data residency constraints**:

---

## What I'd Do Differently Next Time

*(The most important section. Be honest. What did you get wrong? What would you change with another hour? What did you miss on first read?)*

---

## Questions I Still Have

*(Don't skip this. Unresolved questions are data about where to study next. They're not a sign of weakness — they're a sign of intellectual honesty.)*

-
-

---

## Self-Review Checklist

Before marking this chapter Done on your board, verify:

- [ ] Architecture diagram drawn before writing prose
- [ ] Scaling math is explicit (showed derivation, not just stated a number)
- [ ] At least 3 explicit trade-offs documented
- [ ] Hidden requirements section is not empty
- [ ] Assumptions are stated, not assumed
- [ ] I re-read the story context looking for clues after completing my design
- [ ] I compared my design to the chapter's Constraints — does it actually satisfy them?
