---
**Title**: LightspeedRetail — Chapter 99: POS Terminal Payments and P2PE
**Level**: Staff
**Difficulty**: 9
**Tags**: #payments #pci-dss #idempotency #offline #p2pe #reconciliation
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 1 (NovaPay pipeline), Ch. 2 (NovaPay idempotency), Ch. 34 (PulseCommerce Stripe), Ch. 56 (LuminaryAI metering), Ch. 100
**Exercise Type**: System Design / Incident Response
---

### Story Context

**From**: Adyen Partner Relations <partner-relations@adyen.com>
**To**: Beatriz Santos <b.santos@lightspeedretail.com>
**CC**: Payments Engineering <payments-eng@lightspeedretail.com>
**Subject**: RE: UK Incident — Duplicate Charge Volume — Partnership Review Notice
**Date**: Monday, October 7, 09:14 GMT

Beatriz,

Following the network restoration incident on October 3rd, our reconciliation team has identified 4,847 duplicate charge events from UK POS terminals between 14:22 and 15:09 GMT. Total duplicated value: £312,400. We have already initiated chargebacks on behalf of affected cardholders.

We need to understand your store-and-forward implementation before we can continue processing at current volume. A technical review call is scheduled for October 14th. If we cannot reach alignment on your offline transaction handling architecture, we will need to initiate a partnership review which may affect your current processing rates.

Please have your payments engineering lead and a senior architect present.

Regards,
Miriam Holt
Senior Partner Manager, Adyen UK

---

**#payments-engineering** — Slack, Monday October 7

**[09:22] Beatriz Santos**: @everyone — read the Adyen email. All hands 11AM today. No exceptions.

**[09:31] Priya Nair** (Payments Lead): I pulled the logs. The terminal firmware has a retry loop. When the network came back, terminals re-submitted every pending transaction without checking whether the initial submission had landed. The idempotency key was generated at transaction *start* but the retry generated a *new* UUID each time.

**[09:34] Priya Nair**: So yes. Classic store-and-forward failure. The key was generated wrong.

**[09:38] Daniel Cho** (Platform Eng): Wait, the idempotency key is supposed to be deterministic from the transaction data. Why is it generating a new UUID on retry?

**[09:41] Priya Nair**: Because whoever wrote the retry logic didn't read the idempotency spec. The key generation is in a different module from the retry handler.

**[09:45] Beatriz Santos**: I'm bringing in @[you] on this. They own the redesign. Priya, Daniel, block your calendars for the week.

**[09:47] [you]**: On it. I'm reading the incident report now.

**[09:48] Priya Nair**: Fair warning — this is messier than it looks. The terminal firmware is compiled C++. We can push OTA updates but it takes 4-6 hours to propagate to all 13,400 UK terminals. And we have 120k terminals globally. Whatever we design has to work for the firmware we have *right now* until we can ship a fix.

**[09:52] [you]**: Noted. Let's look at the server side first. Why didn't the payments gateway detect the duplicate?

**[09:55] Priya Nair**: The Adyen idempotency key field. When the terminal sends a new UUID each retry, Adyen treats it as a new transaction. There's no server-side deduplication on our end that would catch it before it hits Adyen's API.

**[09:58] Daniel Cho**: So we need dedup at the gateway layer using something deterministic from the transaction itself.

**[10:02] [you]**: Right. And we need to rebuild the entire offline transaction model. Not just patch the UUID bug.

---

**Slack DM — Marcus Webb → [you]** — Monday October 7, 11:47

**Marcus Webb**: Saw your name on the Adyen call. Good luck.

**[you]**: Do you have any context on this class of problem?

**Marcus Webb**: I've solved it twice. Once in 2007 at a fuel retail chain — 4,000 pumps, same offline scenario. Once in 2014 at a grocery chain. 22,000 tills.

**Marcus Webb**: Both times the root cause was the same. Someone designed the idempotency key at the *application* layer instead of at the *transaction* layer. The transaction has a natural identity. Use it.

**[you]**: What defines the natural identity of a POS transaction?

**Marcus Webb**: Think about what's immutable once the customer taps the card. Time, amount, card token, terminal ID. Those four things don't change on retry. If your key doesn't derive from those, you've already lost.

**Marcus Webb**: Also — look at the P2PE scope. You're probably dragging more systems into PCI scope than you need to be. That's a compliance liability your legal team hasn't found yet.

**[you]**: I'll look at scope reduction.

**Marcus Webb**: One more thing. Settlement batches. Reconcile at the terminal level, not just at the account level. You'll thank me at month-end close.

---

This is the fifth time payments have appeared in your career. You built the core pipeline at NovaPay from scratch. You wrangled Stripe idempotency keys at PulseCommerce. You built usage-based billing metering at LuminaryAI. You watched SkyRoute handle partner OAuth delegation. Now you are facing the hardest payments problem yet: offline-capable, hardware-constrained, dual-channel idempotency across 120,000 physical terminals. The Adyen partnership review call is in seven days.

### Problem Statement

LightspeedRetail's POS terminals support offline mode: they store transactions locally when network connectivity is lost and submit them when reconnected (store-and-forward). The current implementation generates a new idempotency key on each retry attempt, causing duplicate charges when terminals reconnect after an outage. The immediate incident involved 13,400 UK terminals; the systemic vulnerability affects all 120,000 terminals globally.

You must redesign the offline transaction model with deterministic idempotency keys, implement server-side deduplication at the payments gateway layer, reduce PCI-DSS scope via Point-to-Point Encryption (P2PE), and define a settlement reconciliation model that can detect discrepancies between terminal-side records and Adyen settlement reports.

### Explicit Requirements

1. Offline transactions must be stored locally on the terminal and submitted when connectivity is restored.
2. Idempotency keys must be deterministic — derived from immutable transaction attributes — so retries never generate a new key.
3. The server-side payments gateway must deduplicate submissions before forwarding to Adyen, even if the terminal sends the same transaction multiple times.
4. P2PE must be implemented to reduce PCI-DSS scope: card data encrypted at the terminal hardware level, never decrypted within LightspeedRetail systems.
5. Settlement reconciliation must run daily, comparing terminal transaction logs against Adyen settlement files to detect gaps, duplicates, and voids.
6. The solution must be compatible with existing terminal firmware (C++) without requiring an immediate firmware update (server-side defense until OTA ships).
7. EMV chip transaction flow (authorization, clearing, settlement) must be respected.
8. Audit log of all idempotency key collisions and deduplication events retained for 7 years (PCI requirement).

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's DM. What did he say about "settlement batches — reconcile at the terminal level, not just at the account level"? There is a hidden requirement for per-terminal settlement ledger, not just per-merchant. This matters for month-end close and chargeback attribution.

- **Hint**: Re-read Priya Nair's message: "The terminal firmware is compiled C++. We can push OTA updates but it takes 4-6 hours to propagate." The firmware update itself is a deployment problem — 13,400 → 120,000 terminals is a staged rollout. What happens if a terminal is mid-transaction when the OTA lands?

- **Hint**: Re-read the Adyen email: "may affect your current processing rates." There is an implicit SLA: Adyen's chargeback rate threshold. If duplicate charges exceed 1% of monthly transaction volume, Adyen can reclassify the merchant tier. That threshold needs to be a monitored metric.

- **Hint**: Re-read Daniel Cho's question about P2PE scope. Marcus confirmed it. If card data is never decrypted in LightspeedRetail systems, which components fall *out* of PCI-DSS scope? Define the CDE boundary before designing the architecture.

### Constraints

- **Scale**: 120,000 POS terminals globally, 13,400 in UK
- **Transaction volume**: ~40M transactions/day globally at peak (Black Friday: ~200M)
- **Offline window**: Terminals can operate offline for up to 72 hours (approved EMV offline floor limit)
- **Offline queue depth**: Up to 2,000 transactions per terminal per offline session
- **Submission latency on reconnect**: All queued transactions must be submitted within 60 seconds of reconnect
- **Deduplication window**: Server must deduplicate within the same 60-second window at up to 26M submissions/hour (worst case: all UK terminals reconnect simultaneously)
- **P2PE**: Must use Adyen's P2PE HSM environment — no bring-your-own HSM
- **Settlement**: Adyen settlement files arrive T+1, 06:00 UTC. Reconciliation must complete by 08:00 UTC.
- **PCI scope reduction target**: Move from SAQ D to SAQ P2PE (reduces audit scope by ~70%)
- **Audit retention**: 7 years, immutable
- **Team**: 4 payments engineers, 1 security engineer, 1 DBA
- **Adyen review call**: 7 days

### Your Task

Design the offline-capable POS payment architecture with deterministic idempotency, server-side deduplication, P2PE scope reduction, and terminal-level reconciliation. Prepare the technical brief for the Adyen partnership review call.

### Deliverables

- [ ] **Mermaid architecture diagram** showing: terminal offline queue → reconnect submission flow → payments gateway deduplication layer → Adyen API → settlement reconciliation pipeline. Include P2PE encryption boundary and PCI CDE scope annotation.

- [ ] **Idempotency key schema**: Define the deterministic key derivation function. Format: `SHA-256(terminal_id || transaction_amount || card_token_last4 || transaction_date_utc || emv_arc)`. Show why each field is included and why it is immutable once the card is presented.

- [ ] **Database schema** for the payments gateway deduplication table:
  ```
  idempotency_keys (
    key_hash         CHAR(64)     PRIMARY KEY,
    terminal_id      UUID         NOT NULL,
    submitted_at     TIMESTAMPTZ  NOT NULL,
    adyen_psp_ref    VARCHAR(64),
    status           ENUM('pending','authorized','declined','duplicate') NOT NULL,
    raw_request_hash CHAR(64)     NOT NULL,
    INDEX(terminal_id, submitted_at),
    INDEX(status, submitted_at)  -- for reconciliation queries
  )
  ```
  Extend with the per-terminal settlement ledger table. Show all column types and indexes.

- [ ] **P2PE architecture**: Describe the encryption boundary. Where does card data enter? Where is it decrypted (Adyen's HSM, not yours)? Which LightspeedRetail components are now out of PCI scope? Draw the before/after CDE boundary.

- [ ] **Scaling estimation**:
  - Normal peak: 40M tx/day ÷ 86,400 sec = 463 TPS average. Peak 3x = ~1,400 TPS.
  - Reconnect surge: 13,400 terminals × 2,000 queued tx ÷ 60 sec = 446,667 submissions/minute = 7,444 TPS. Show how the dedup layer handles this burst.
  - Dedup table storage: 40M keys/day × 64 bytes × 365 days × 7 years = show math.
  - Black Friday: 200M tx/day. Recalculate TPS and storage.

- [ ] **Reconciliation pipeline design**: T+1 settlement file ingestion → per-terminal diff → exception queue → human review workflow. Show how you detect: (a) terminal submitted, Adyen not received; (b) Adyen charged, terminal has no record; (c) duplicate detected post-settlement.

- [ ] **Tradeoff analysis** (minimum 3):
  1. Deterministic key derivation vs opaque UUID: collision risk analysis. What happens if two legitimate transactions share all four fields?
  2. Server-side dedup vs fixing terminal firmware: server-side is safer short-term but you now own dedup forever. When do you remove it?
  3. P2PE reduces PCI scope but you lose visibility into card data for fraud analytics. How do you preserve fraud signal without decrypting?

- [ ] **EMV transaction flow summary**: Authorization → Clearing → Settlement — which stage does each of your components interact with? Where does store-and-forward sit in the EMV lifecycle?

- [ ] **Adyen review call brief** (1 page, bullet format): Technical summary of root cause, immediate server-side fix, 90-day roadmap to P2PE, reconciliation SLA commitment.
