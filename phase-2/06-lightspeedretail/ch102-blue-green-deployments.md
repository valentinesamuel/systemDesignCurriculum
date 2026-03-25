---
**Title**: LightspeedRetail — Chapter 102: Zero-Downtime Deploys for 120,000 POS Terminals
**Level**: Staff
**Difficulty**: 8
**Tags**: #blue-green #deployments #database-migrations #zero-downtime #canary #rollback
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 101, Ch. 99, Ch. 104
**Exercise Type**: Incident Response / System Design
---

### Story Context

**#incidents** — Slack, Tuesday October 15, 14:02

**[14:02] PagerDuty** [ALERT]: `lightspeed-pos-api` health check failure rate > 50%. Triggered by: `health_check_failure_rate_critical`. Severity: P0.

**[14:03] Marcus Delgado**: I'm on. What happened?

**[14:04] Ravi Patel**: Deploy just went out. v2.14.3. Started at 14:00 exactly.

**[14:05] Marcus Delgado**: 120k terminals are getting 503s. The terminals are switching to "Maintenance Mode." Cashiers are telling customers to wait.

**[14:07] Beatriz Santos**: Revenue impact estimate?

**[14:08] Marcus Delgado**: $8.7M per hour is the Black Friday estimate. Tuesday afternoon is lower — estimating $720k per hour at current trading volume.

**[14:09] Ravi Patel**: I see what happened. The DB migration in v2.14.3 added a NOT NULL column to the `transactions` table. It ran an `ALTER TABLE` during the deploy window. That locked the table for 9 minutes.

**[14:11] Marcus Delgado**: 9 minutes of table lock on `transactions`. The main write path for every terminal.

**[14:13] [you]**: Can we roll back?

**[14:14] Ravi Patel**: Not easily. The migration already ran. The old binary doesn't know about the new column. Rolling back the binary while the new schema is live will cause write failures on the new column.

**[14:16] [you]**: Okay. We ride forward. Can we complete the migration faster?

**[14:18] Ravi Patel**: Migration is actually done — the column exists. The lock released 2 minutes ago. The issue now is that 120k terminals are in Maintenance Mode and need to reconnect.

**[14:21] Marcus Delgado**: Reconnect surge. Same as the cache problem. 120k terminals connecting simultaneously.

**[14:24] [you]**: Rate limit the reconnect. Stagger by region. Let them reconnect in batches.

**[14:26] Ravi Patel**: On it. Setting reconnect backoff in the terminal firmware... wait. We can't push firmware in the middle of an incident.

**[14:28] [you]**: Server-side backoff then. Return HTTP 429 with Retry-After header to terminals that try to reconnect before their region's window.

**[14:30] Marcus Delgado**: That will work. Writing the nginx rate limit config now.

**[14:47] Marcus Delgado**: Reconnect surge absorbed. Terminal error rate back below 0.1%.

**[14:48] Beatriz Santos**: Full outage duration: 12 minutes. Estimated revenue impact: $144,000. I want a root cause and a plan to make this impossible to happen again. @[you] owns the plan.

---

**Post-Incident Review — Email Chain**

**From**: Beatriz Santos <b.santos@lightspeedretail.com>
**To**: Engineering Leadership <eng-leadership@lightspeedretail.com>
**Subject**: Post-Incident Review — v2.14.3 Deploy Outage — Action Items
**Date**: Tuesday October 15, 17:00

Team,

Root cause is documented: an `ALTER TABLE ADD COLUMN NOT NULL` executed during the deploy window, locking the `transactions` table for 9 minutes.

This is not a one-time mistake. This is a symptom of having no deployment architecture. We are deploying to a live system with a single fleet, no blue/green separation, and no schema migration discipline.

Action items assigned to @[you]:
1. Design blue/green deployment architecture for all three critical services
2. Define expand/contract migration pattern — no migration that locks a table may ever run during a deploy
3. Automated rollback trigger: if health check SLO breaches within 5 minutes of deploy, roll back automatically
4. Canary release within blue/green: new version receives 5% traffic before full cutover

Black Friday is in 33 days. I want this design reviewed and approved before we deploy again.

Beatriz

---

**Slack DM — Marcus Webb → [you]** — Tuesday October 15, 18:22

**Marcus Webb**: $144k for a NOT NULL migration. Classic. What's the root cause beyond the obvious?

**[you]**: No deploy discipline. Single fleet. Schema migration runs during deploy, holds a table lock, all terminals queue up.

**Marcus Webb**: And you couldn't roll back because the schema was ahead of the binary.

**[you]**: Exactly. Trapped between two versions.

**Marcus Webb**: Expand/contract. Non-negotiable. You never write a migration that a previous version of the binary cannot read. Add the column as nullable, deploy the binary, then add the NOT NULL constraint, then backfill. Three separate deploys, three separate migrations. No locks.

**[you]**: That's more operational complexity.

**Marcus Webb**: That's the price of zero-downtime. There is no shortcut. The alternative is explaining another $144k incident to the board.

**[you]**: What about long-lived POS terminal connections? Blue/green means the old fleet drains slowly if terminals hold open WebSocket connections.

**Marcus Webb**: Good question. You need a drain timeout. Give old terminals 5 minutes to naturally close and reopen on the green fleet. Force-close anything still on blue after the drain window. What's the terminal reconnect behavior?

**[you]**: Exponential backoff with jitter. Max 60-second reconnect interval.

**Marcus Webb**: Then your drain window is 5 minutes plus one 60-second reconnect interval. Call it 7 minutes. After 7 minutes, blue fleet is gone and every terminal has reconnected to green.

**[you]**: What triggers automatic rollback?

**Marcus Webb**: SLO breach. Not error count — error rate. If error rate on the new version exceeds your baseline by X% within 5 minutes of deploy, roll back. Make X a tunable parameter. Don't hardcode it.

---

This incident happened 33 days before Black Friday. The last one happened 2 weeks before Black Friday last year. The pattern is clear: every year, a deploy in the pre-Black-Friday window causes an outage that would not have happened with blue/green. The cost of the architecture is 3 weeks of engineering time. The cost of not building it is measured in six figures and board-level attention.

### Problem Statement

A database migration executed during a live deployment locked the `transactions` table for 9 minutes, causing a 12-minute outage across 120,000 POS terminals. The incident revealed the absence of blue/green deployment architecture, schema migration discipline, automated rollback triggers, and safe drain behavior for long-lived terminal connections. With 33 days to Black Friday, the deployment process must be redesigned end-to-end before the next release.

You must design a blue/green deployment architecture, define the expand/contract database migration pattern, build automated rollback triggers keyed to SLO breach, integrate canary traffic within the blue/green model, and handle the terminal drain problem for long-lived POS connections.

### Explicit Requirements

1. Blue/green deployment: blue fleet serves 100% traffic until green fleet passes smoke tests.
2. Zero breaking migrations: no migration may lock a production table during the deploy window.
3. Expand/contract migration pattern: three-phase (add nullable → deploy binary → add constraint/backfill).
4. Automated rollback: if green fleet error rate exceeds baseline + 2% within 5 minutes of cutover, automatically route 100% traffic back to blue.
5. Canary phase: new version receives 5% traffic for 10 minutes before full blue→green cutover.
6. POS terminal drain: blue fleet must serve existing terminal connections for 7 minutes after cutover begins. Force-close connections after drain window.
7. Smoke test suite must run automatically against green fleet before any traffic is routed.
8. Deploy pipeline must prevent any migration with an `ALTER TABLE ... LOCK` from running during the deploy window.

### Hidden Requirements

- **Hint**: Re-read Ravi Patel's message: "Rolling back the binary while the new schema is live will cause write failures." There is a hidden requirement: blue and green fleets must be able to run against the same schema simultaneously during the canary phase. This means the schema must be forward-compatible — the old binary must tolerate the new column. Expand/contract enforces this. Make it explicit in your migration gate.

- **Hint**: Re-read the description of the reconnect surge fix: "Return HTTP 429 with Retry-After header." The hidden requirement is that the rate limiter for terminal reconnects must persist across deploys — it is not a one-off incident fix but a permanent feature of the deploy drain process.

- **Hint**: Re-read Marcus Webb: "Make X a tunable parameter. Don't hardcode it." There is a hidden requirement: rollback threshold must be configurable per-service, not a global constant. The payments gateway has a lower tolerance than the product catalog API.

- **Hint**: Re-read the timeline: "Full outage duration: 12 minutes." 9 minutes was the table lock. What were the other 3 minutes? Terminals in Maintenance Mode reconnecting without the server-side rate limit. Hidden requirement: reconnect surge mitigation must be part of the blue/green cutover playbook, not just an incident response action.

### Constraints

- **Terminals**: 120,000 globally with long-lived connections (WebSocket / HTTP keep-alive)
- **Terminal reconnect behavior**: exponential backoff, max 60-second interval
- **Drain window**: 7 minutes (5-minute natural drain + 60-second reconnect window + 60-second buffer)
- **Canary traffic split**: 5% green, 95% blue for 10 minutes before full cutover
- **Rollback SLO trigger**: error rate on green > baseline + 2% sustained for 60 seconds within 5 minutes of cutover
- **Smoke test duration**: must complete in under 2 minutes
- **Deploy frequency**: 3-5 deploys per week to each service
- **Migration gate**: automated check in CI/CD pipeline using `pg_locks` analysis or EXPLAIN plan parsing
- **Team**: 3 DevOps, 2 platform engineers
- **Infrastructure**: ALB with weighted target groups (supports canary traffic splitting)

### Your Task

Design the complete blue/green deployment architecture and migration discipline. Produce the automated rollback specification and the migration safety gate design.

### Deliverables

- [ ] **Mermaid architecture diagram** showing: CI/CD pipeline → AMI build → green fleet provisioning → smoke tests → canary phase (ALB weighted target groups: 5/95) → full cutover → blue fleet drain → blue fleet decommission. Include automated rollback path.

- [ ] **Expand/contract migration pattern** — three-phase example using the `transactions.card_type` column:
  - Phase 1 (Expand): `ALTER TABLE transactions ADD COLUMN card_type VARCHAR(20)` (nullable, no lock)
  - Phase 2 (Deploy): deploy binary that writes `card_type` on new transactions; old binary ignores the column
  - Phase 3 (Contract): background backfill of historical rows; then `ALTER TABLE transactions ALTER COLUMN card_type SET NOT NULL` (safe after backfill)
  Show: which migrations are safe to run during deploy vs after deploy vs before deploy.

- [ ] **Migration safety gate design**: CI/CD pipeline step that analyzes migration SQL before allowing deploy to proceed. Define the ruleset:
  - Block: `ALTER TABLE ... ADD COLUMN ... NOT NULL` without DEFAULT
  - Block: `DROP TABLE`, `DROP COLUMN` (schema is ahead of binary — must use expand/contract)
  - Block: full-table `UPDATE` without batch size limit
  - Allow: `CREATE INDEX CONCURRENTLY`
  - Show how you implement this (a linter script, an ORM migration validator, or a custom CI step).

- [ ] **Blue/green cutover playbook** (ordered steps with timing):
  1. Build and bake green AMI
  2. Provision green Auto Scaling Group (identical to blue)
  3. Run smoke tests against green (2 min max)
  4. Canary phase: ALB weighted routing 5% → green, 95% → blue (10 min)
  5. Monitor: canary error rate vs baseline
  6. Go/no-go decision: automated (SLO check) or manual
  7. Full cutover: ALB weighted routing 100% → green
  8. Drain window: blue fleet continues serving existing connections for 7 min
  9. Blue fleet decommission

- [ ] **Automated rollback trigger design**: TypeScript interface for the rollback policy:
  ```typescript
  interface RollbackPolicy {
    serviceId: string;
    errorRateBaselinePct: number;
    errorRateThresholdDelta: number;   // e.g., 2.0 → trigger if error rate > baseline + 2%
    evaluationWindowSeconds: number;   // sustained for this long before triggering
    rollbackTimeoutSeconds: number;    // how long to wait for blue fleet to recover
    notifyChannels: string[];
  }
  ```
  Show how you integrate this with CloudWatch alarms and the ALB target group weight API.

- [ ] **Terminal drain handler design**: When blue fleet receives drain signal (ALB deregistration):
  - Continue serving existing terminal connections
  - Return `Connection: close` on new connection requests (redirect to green)
  - After 7-minute drain window: close all remaining connections with `1001 Going Away`
  - Show: NGINX or application-level drain config

- [ ] **Database schema** for deploy audit log:
  ```
  deploys (
    deploy_id        UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
    service_name     VARCHAR(64)   NOT NULL,
    version          VARCHAR(32)   NOT NULL,
    blue_asg_id      VARCHAR(64)   NOT NULL,
    green_asg_id     VARCHAR(64)   NOT NULL,
    status           ENUM('canary','live','rolled_back','complete') NOT NULL,
    canary_started   TIMESTAMPTZ,
    cutover_at       TIMESTAMPTZ,
    rollback_at      TIMESTAMPTZ,
    error_rate_at_cutover NUMERIC(5,2),
    notes            TEXT,
    INDEX(service_name, status),
    INDEX(cutover_at DESC)
  )
  ```

- [ ] **Scaling estimation**:
  - During canary phase: 5% of 120k terminals = 6,000 terminals on green. At Black Friday volume (250k RPS), 5% = 12,500 RPS on green. Does your smoke test fleet handle this? Show instance math.
  - Drain window load: during the 7-minute drain, blue handles all existing connections while green handles all new. What is the peak concurrent connection count on blue during drain?

- [ ] **Tradeoff analysis** (minimum 3):
  1. Blue/green vs rolling deploy: blue/green requires 2x fleet during transition, rolling deploy does not. What is the cost and what do you gain?
  2. Automated rollback vs human-in-the-loop: automated rollback is faster but can trigger false positives (transient spikes). How do you tune the threshold to minimize false positives without sacrificing safety?
  3. Expand/contract adds 3 deploys per schema change vs 1: is this always worth it? Define a category of changes where a simpler migration strategy is acceptable.
