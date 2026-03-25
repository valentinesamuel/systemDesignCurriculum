---
**Title**: BuildRight — Chapter 128: Experimentation Platform — Contradictory A/B Results
**Level**: Staff
**Difficulty**: 8
**Tags**: #experimentation #ab-testing #statistics #platform-design #spike-design-review
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 134 (Axiom Labs ML A/B), Ch. 88 (GigGrid progressive delivery)
**Exercise Type**: System Design / Design Review Ambush
---

### Story Context

**SPIKE — Design Review Ambush**

The calendar invite said: *"Experimentation Platform Design Review — 45 min."*
Attendees: you, Kenji Mori, Liam Kowalski, and two product managers.
You have slides. You have a Mermaid diagram. You are ready.

Dr. Priya Mehta joins the call three minutes late.

Kenji blinks. "Priya — I didn't know you were on this one."

**Priya**: "I wasn't. I saw the invite and thought I should be." She opens her laptop. "Before you present — can you explain why your current A/B framework is statistically invalid?"

Silence.

**You**: "...I haven't presented yet."

**Priya**: "I know. But you're about to propose expanding it. I've been looking at the last six months of experiment results. Something is wrong and I don't think anyone in this room knows what it is."

She shares her screen. A scatter plot. Thirty-one completed experiments. Fourteen showed positive results in the primary metric. Twelve showed no effect. Five showed contradictory results — positive in one segment, negative in another with no interaction term to explain it.

**Priya**: "Walk me through how your current system assigns users to variants."

**You**: "Hash of user ID modulo number of variants. Deterministic — same user always gets same variant."

**Priya**: "Good. Now walk me through how you ensure two concurrent experiments don't interfere with each other."

Beat.

**You**: "...They're run on different features."

**Priya**: "That's not isolation. That's hope."

She advances to the next slide — a timeline. Eight experiments running concurrently in November. The experiment testing the new RFI submission flow ran at the same time as an experiment testing the notification frequency for RFI updates. A user in variant A of the first experiment was more likely to land in variant B of the second, because both experiments used the same hash with no orthogonality check.

**Priya**: "SUTVA. Stable Unit Treatment Value Assumption. It's the foundational assumption of any causal experiment. It says: the treatment of one unit doesn't affect the outcome of another. You've been running experiments where the treatments are correlated. Your results are meaningless."

**Kenji**: "All of them?"

**Priya**: "The concurrent ones. Which is... most of them."

**Kenji** (quietly, to you): "How many product decisions were made based on these?"

**Liam**: "At least four major feature launches."

The room is quiet for eleven seconds. You counted.

**Priya**: "I'm not here to assign blame. I'm here because you're about to build a new experimentation platform and I want it to be correct this time. So yes — please present. And I will interrupt you when something is wrong."

---

**After the meeting — DM thread**

**Priya Mehta [DM to you]**: "Sorry for the ambush. I could have been more diplomatic."

**You**: "No. You were right. Should I have known about SUTVA before building this?"

**Priya**: "Yes and no. Most engineers building experiment infrastructure don't think about this until they've been burned by it. You're being burned by it in a meeting room, which is the best possible place. Not in a product decision that goes wrong at scale."

**You**: "What would a correct system look like?"

**Priya**: "Mutual exclusion for overlapping experiments. Or orthogonal assignment if you need concurrency. Plus holdout groups so you have a clean control. Plus power analysis before launch so we know if we even have enough traffic to detect the effect we care about. Plus proper statistical corrections for running multiple metrics."

**You**: "That sounds like a platform, not a feature."

**Priya**: "Yes. It is. That's the point."

---

**Marcus Webb [DM — that evening]**

**Marcus Webb**: "Heard about the Priya Mehta situation. A few questions. Why do most companies build the instrumentation before the statistics? What's the correct unit of randomization for a B2B platform — the user, or the company? Does it matter? Think before you design."

No sign-off. Just those three questions.

---

### Problem Statement

BuildRight's current A/B testing infrastructure assigns users to experiment variants using a simple hash-mod approach with no mutual exclusion between concurrent experiments. This violates the Stable Unit Treatment Value Assumption (SUTVA), producing correlated treatment assignments when multiple experiments run simultaneously. The result is six months of potentially invalid experiment results and four product decisions made on corrupted data.

The new experimentation platform must support concurrent experiments without SUTVA violations, provide proper holdout groups, enforce statistical power requirements before experiment launch, and handle the specific challenges of a B2B construction platform where the unit of randomization may be the company (GC organization) rather than the individual user.

---

### Explicit Requirements

1. Experiment assignment service: deterministic, consistent assignment across sessions and devices for the same user/entity.
2. Mutual exclusion layer: experiments marked as conflicting cannot co-enroll the same user/entity simultaneously.
3. Orthogonal assignment: for non-conflicting experiments, assignment must be statistically independent (no correlation between variant assignments).
4. Holdout groups: platform-level holdout (5% of users never in any experiment) for measuring cumulative experimentation effect.
5. Pre-launch power analysis: require minimum detectable effect, expected conversion rate, and desired power (0.8 default) before an experiment can launch; block launch if traffic is insufficient.
6. Metric definitions: experiments must declare primary metric (1) and secondary metrics (up to 5) before launch; automatic Bonferroni correction for multiple metrics.
7. Segment-level analysis: results can be broken down by company size, industry vertical, country — but with statistical correction for multiple comparisons.
8. Experiment lifecycle: draft → review → running → stopped → analyzed. No experiment can be stopped early based on peeking at results without triggering a sequential testing correction.
9. Results dashboard: show confidence intervals, not just p-values. Flag underpowered experiments.
10. Audit log: all experiment decisions (launch, stop, override) logged with actor and timestamp.

---

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's DM. He asks whether the unit of randomization is the user or the company. BuildRight is a B2B platform — a general contractor (GC) organization has dozens of users. If you randomize by user, two users at the same GC might see different variants of the same workflow, creating within-company inconsistency that confounds results and creates a bad product experience. What is the correct unit of randomization and what does that mean for traffic calculations?
- **Hint**: Re-read Priya's explanation of the November concurrent experiments. The notification frequency experiment affected users' engagement with RFIs, which was the primary metric of the RFI submission flow experiment. Mutual exclusion prevents co-enrollment, but it doesn't prevent *indirect* interference. What additional safeguards prevent one experiment's side effects from contaminating another's metrics?
- **Hint**: Re-read Kenji's reaction: "How many product decisions were made based on these?" Four feature launches happened based on corrupted data. Those features are live. Part of the platform design is a remediation path: how do you decide which past experiments need to be re-run? What metadata do you need to have stored to make that determination?
- **Hint**: Re-read the five "contradictory results — positive in one segment, negative in another." This is a real statistical phenomenon: heterogeneous treatment effects. Your platform must detect and surface interaction effects rather than hiding them in an aggregate metric. What does the schema for storing segment-level results look like?

---

### Constraints

- **Traffic**: 180,000 active users/month. 40,000 active GC organizations.
- **Concurrent experiments**: up to 25 running simultaneously at peak product velocity.
- **Minimum detectable effect**: platform must enforce MDE ≥ 1% for conversion metrics (smaller effects are not actionable at this traffic level).
- **Randomization unit**: must support both user-level and organization-level randomization (configurable per experiment).
- **Result latency**: experiment results must update within 1 hour of events being logged.
- **Statistical framework**: frequentist with p < 0.05 threshold as default; Bayesian option for fast-moving product teams.
- **Holdout size**: 5% platform-level holdout (9,000 organizations never enrolled in experiments).
- **Storage**: experiment event logs retained 2 years.
- **Infrastructure**: existing event pipeline (Kafka). Analytics warehouse (Redshift). Budget: $2,500/month additional infrastructure.
- **Team**: 1 data engineer, 1 backend engineer, you. Three months.

---

### Your Task

Design the experimentation platform for BuildRight. Address the assignment service (mutual exclusion + orthogonal layers), the statistical framework (power analysis, multiple testing correction, sequential testing guardrails), the organization-vs-user randomization problem, and the results pipeline. Provide a remediation plan for evaluating which of the six months of past experiments need to be re-run.

---

### Deliverables

- [ ] **Mermaid architecture diagram** — show: experiment registry, assignment service (with orthogonal/exclusion layer), event collection (Kafka), metrics computation pipeline (Redshift), and results dashboard
- [ ] **Database schema** — `experiments` table, `experiment_assignments` table (user_id or org_id + variant + timestamp), `experiment_metrics` table (pre-declared metric definitions), `experiment_results` table (per-segment confidence intervals); include column types and indexes
- [ ] **Scaling estimation** — calculate: assignment lookup volume at 180,000 MAU × 4 assignments/session; event log volume at 40 tracked events/user/day; Redshift compute cost for nightly metrics roll-up
- [ ] **Tradeoff analysis** — minimum 3: (1) user-level vs organization-level randomization (power vs consistency), (2) frequentist vs Bayesian (speed vs interpretability for PM audience), (3) mutual exclusion (low concurrency) vs orthogonal layers (higher complexity)
- [ ] **Orthogonal assignment algorithm** — describe the hashing strategy that guarantees independent assignment across two concurrent non-exclusive experiments; show a worked example with user IDs
- [ ] **Past experiment remediation checklist** — criteria for determining which past experiments had SUTVA violations severe enough to require re-running; what metadata you need to have stored to make this determination
