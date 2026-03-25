---
**Title**: VenueFlow — Chapter 117: A/B Testing Infrastructure
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #ab-testing #experimentation #feature-flags #statistics #data-infrastructure
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 115 (VenueFlow vector search), Ch. 116 (VenueFlow inventory), Ch. 56 (LuminaryAI ML infrastructure)
**Exercise Type**: System Design
---

### Story Context

**Email**
**To:** eng-leads@venueflow.io, data-science@venueflow.io
**From:** priya.mehta@venueflow.io (Head of Data Science)
**Subject:** Experiment Results — We Have a Problem
**Date:** Wednesday, 8:44 AM

Team,

I need to escalate something before the Q4 product review on Friday.

In the last six weeks, the product team ran four experiments simultaneously:
- Experiment A: New checkout flow (control vs. variant)
- Experiment B: Recommendation carousel placement (top of page vs. below fold)
- Experiment C: Hold timer display (countdown visible vs. hidden)
- Experiment D: Price display format (fee-inclusive vs. fee-exclusive)

All four experiments reported statistically significant results. Experiment A showed the new checkout improved conversion by 11%. Experiment B showed the recommendation carousel at the top of the page reduced conversion by 8%. Experiment C showed the hidden hold timer increased abandonment by 14%. Experiment D showed fee-inclusive pricing increased conversion by 6%.

**Here is the problem.** These four experiments ran on overlapping user populations with no mutual exclusion. A user in the Experiment A variant could simultaneously be in the Experiment B variant and the Experiment C control. The "11% checkout improvement" could be entirely explained by interaction with Experiment D's fee-inclusive pricing. We cannot disentangle them.

Every number in that summary is meaningless. We are about to present these results to the board as evidence for a $2M checkout redesign investment.

I need an experimentation platform. Not a feature flag toggle. An actual platform. I'm available to talk through requirements whenever engineering is ready.

— Priya

---

**#data-science — Slack**
**Wednesday, 10:15 AM**

**[you]:** Priya — I read your email. Let's set up 30 minutes this afternoon.
**priya.mehta:** Thank you. I should tell you: I've had this conversation at two previous companies. Both times, the engineering response was "we'll add a flag to our feature flag system." That is not what I need.
**[you]:** Understood. Feature flags and experimentation are different tools. What are the three things an experiment platform needs to do that a feature flag system doesn't?
**priya.mehta:** One: consistent assignment. The same user must see the same variant every time they visit, even across sessions. Two: mutual exclusion. Running four experiments simultaneously on overlapping audiences invalidates all four. Three: statistical rigor. When the platform declares an experiment significant, that declaration must mean something — not just p < 0.05 on underpowered data.
**[you]:** Sample size calculation at experiment creation?
**priya.mehta:** Minimum detectable effect, desired power, significance threshold. Forced inputs before you can launch. If you want to detect a 2% conversion lift with 80% power at p=0.05, you need to know how long the experiment must run before it's valid to stop.
**[you]:** And the holdout groups?
**priya.mehta:** Yes. We need a global holdout — a percentage of users who are excluded from all experiments. They're the baseline that doesn't drift. If your global holdout shows metric improvement, it's not your experiments doing it. It's something else.

**3:18 PM**
**[you]:** One more thing I want to flag: consistent hashing for assignment. We need the same user to get the same variant without querying a database on every request.
**priya.mehta:** How does that work?
**[you]:** Hash(user_id + experiment_id) → bucket. Bucket 0–49 = control. Bucket 50–99 = variant. Deterministic. No state lookup. The assignment is recomputable anywhere from the user ID.
**priya.mehta:** And that doesn't drift over time?
**[you]:** It drifts if you change the traffic split mid-experiment. Which is one of the things the platform must prevent.

---

**1:1 — Zara Ahmed → you — Thursday, 9:30 AM**

"How bad is it?" Zara asks.

"The four experiments? Completely confounded. They'll need to rerun them cleanly with proper isolation. The Experiment A checkout results they're planning to present on Friday are not valid."

Zara closes her eyes briefly. "Can we tell the board that?"

"We have to tell the board that. The alternative is making a $2M investment on data we know is wrong." You pause. "But we can also tell them we're fixing it. Have a platform spec ready by Friday. Show them we know the right answer — we just need two months to build it."

"Two months," Zara repeats. "What do we ship in month one?"

"Consistent assignment service, mutual exclusion groups, sample size calculator. That's the foundation. Month two: statistical analysis dashboard, global holdout, automated significance monitoring with early stopping rules."

"Early stopping rules?"

"The temptation is to peek at experiment results every day and stop when you see significance. That inflates your false positive rate. The platform enforces a predetermined stopping rule — you either run to your planned sample size or you use a sequential test that accounts for peeking."

---

### Problem Statement

VenueFlow's ad-hoc feature flag system has been used to run A/B experiments without mutual exclusion, consistent user assignment, or statistical power planning. Four simultaneous experiments produced confounded results that cannot be trusted. Design an experimentation platform that provides consistent user assignment, mutual exclusion between experiments, sample size validation, and statistically rigorous result analysis.

### Explicit Requirements

1. Consistent assignment: the same user must see the same variant across sessions and devices, without a database lookup on every request
2. Mutual exclusion groups: experiments in the same group cannot run on overlapping user populations
3. Sample size calculator: required input before experiment launch — minimum detectable effect, statistical power, significance threshold, and estimated experiment duration
4. Global holdout: a configurable percentage of users excluded from all experiments (default: 5%)
5. Traffic splitting: support arbitrary percentage splits (50/50, 90/10, 33/33/34) with variant-level tracking
6. Result dashboard: per-experiment metrics with confidence intervals, p-values, and run-to-completion progress
7. Prevent mid-experiment changes: once an experiment is launched, traffic split and targeting cannot be modified (requires stopping and restarting)
8. Audit log: all experiment creation, launch, pause, and stop events logged with actor and timestamp

### Hidden Requirements

- **Hint: re-read Priya's description of "interaction effects" between experiments.** Mutual exclusion prevents population overlap within an experiment layer. But is it possible for two experiments that touch the same metric (conversion rate) to interact even if their user populations are disjoint? What does this mean for how you define experiment layers?

- **Hint: re-read the discussion about early stopping rules.** The platform prevents mid-experiment traffic changes. But what about stopping the experiment early because results look good? What is the statistical cost of peeking, and what mechanism does the platform provide to allow valid early stopping?

- **Hint: re-read the consistent hashing assignment discussion.** Hash(user_id + experiment_id) → bucket. What happens to assignment consistency when an experiment's traffic split changes from 50/50 to 60/40? Does the same hash function produce different assignments? What is the correct behavior?

- **Hint: re-read Priya's requirement for a global holdout.** "If your global holdout shows metric improvement, it's not your experiments doing it." What is the platform responsibility when the global holdout shows metric degradation — and what does that signal to the engineering team?

### Constraints

- **MAU**: 5 million users eligible for experimentation
- **Concurrent experiments**: up to 20 running simultaneously
- **Assignment latency**: <5ms P99 (on every page render — cannot afford DB lookup)
- **Experiment layers**: at least 3 independent layers (checkout, recommendations, display)
- **Global holdout**: 5% of users (250k), excluded from all experiments
- **Minimum experiment duration**: enforced by sample size calculator (cannot launch if estimated duration > 90 days)
- **Result freshness**: metrics updated within 1 hour of event occurrence
- **Audit retention**: experiment configuration and results retained 3 years
- **Team**: 2 engineers (platform) + Priya (data science lead)
- **Timeline**: month 1 for core assignment + exclusion; month 2 for analysis dashboard

### Your Task

Design the VenueFlow experimentation platform. Define the user assignment service, experiment layer and mutual exclusion model, sample size validation logic, result aggregation pipeline, and global holdout implementation. The assignment service must be low-latency and stateless enough to call on every page render.

### Deliverables

- [ ] Mermaid architecture diagram: assignment service (consistent hash), experiment config store, event tracking pipeline, result aggregation, dashboard
- [ ] Database schema for:
  - Experiment definitions (id, layer, traffic split, variants, sample size target, status)
  - User assignment log (user_id, experiment_id, variant, assigned_at, bucket)
  - Experiment events (user_id, experiment_id, variant, event_type, value, timestamp)
- [ ] Scaling estimation (show math):
  - Assignment calls per second: 5M DAU × average page renders per session / seconds per day
  - Event ingestion rate: assignment events + conversion events + engagement events
  - Result aggregation: how frequently to recompute per-experiment statistics
- [ ] Tradeoff analysis (minimum 3):
  - Consistent hashing (stateless, no DB) vs. assignment DB (flexible, queryable)
  - Mutual exclusion via layers vs. global traffic budgets
  - Fixed-horizon testing vs. sequential testing for early stopping
- [ ] Sample size calculator spec: inputs (MDE, power, significance, baseline conversion rate), output (required sample size per variant, estimated days to completion at current traffic)
- [ ] TypeScript interface sketch for the assignment service:
  ```typescript
  interface ExperimentAssignment {
    experimentId: string;
    userId: string;
    variant: string | null; // null if in global holdout or not in experiment
    bucket: number;
  }

  interface AssignmentService {
    assign(userId: string, experimentId: string): ExperimentAssignment;
    getActiveExperiments(userId: string): ExperimentAssignment[];
  }
  ```

---
