---
**Title**: BuildRight — Chapter 129: Progressive Delivery with Argo Rollouts
**Level**: Staff
**Difficulty**: 8
**Tags**: #progressive-delivery #canary #gitops #argocd #argo-rollouts #spike-production-down
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 88 (GigGrid canary deployments), Ch. 102 (LightspeedRetail blue/green)
**Exercise Type**: Incident Response / System Design
---

### Story Context

**SPIKE — Production is DOWN**

The PagerDuty alert fires at 2:17 AM AEST.

---

**#incidents** *(Slack — March 18, 2026)*

**[02:17 AEST] PagerDuty**: CRITICAL — `permit-submission-service` error rate 89% — Region: ap-southeast-2 (Sydney)

**[02:19 AEST] Liam Kowalski**: On bridge. I deployed at 10 PM. That was 4 hours ago and it was fine.

**[02:20 AEST] You**: What changed in the deploy?

**[02:21 AEST] Liam Kowalski**: New permit validation logic for AU/NSW state. Required field changes. Should be backward-compatible.

**[02:22 AEST] You**: Error message?

**[02:22 AEST] Liam Kowalski**: `ValidationError: 'site_classification_code' is required`. All permit submissions from AU construction sites.

**[02:23 AEST] You**: How many sites affected?

**[02:24 AEST] Liam Kowalski**: 2,000 Australian sites. Active time is 6 AM–6 PM AEST. It's 2 AM now. But some sites work night shift. And building inspectors in AU are on-site right now, trying to file approval forms.

**[02:25 AEST] You**: What's the rollback procedure?

**[02:26 AEST] Liam Kowalski**: Redeploy old image. Kubernetes. I have the tag.

**[02:27 AEST] You**: Do it.

**[02:28 AEST] Liam Kowalski**: Opening kubectl...

**[02:35 AEST] Liam Kowalski**: Okay I have the YAML open. Production YAML. Making the image tag change.

**[02:41 AEST] Liam Kowalski**: Applied. Rolling out.

**[02:44 AEST] Liam Kowalski**: Error rate dropping. 67%... 45%... 12%...

**[02:47 AEST] Liam Kowalski**: Back to baseline. Error rate 0.4%. Normal.

**[02:48 AEST] Thomas Adeyemi**: What happened.

**[02:49 AEST] Liam Kowalski**: Bad deploy. Rolled back.

**[02:50 AEST] Thomas Adeyemi**: How long were we down?

**[02:51 AEST] Liam Kowalski**: 30 minutes.

**[02:52 AEST] Thomas Adeyemi**: How many inspectors were on-site in Australia tonight.

**[02:53 AEST] Liam Kowalski**: Checking... 34 active sessions in the last hour.

**[02:54 AEST] Thomas Adeyemi**: So 34 inspectors couldn't file permits for 30 minutes. In the middle of site visits. Which means 34 contractors are now in a gray zone on their approval status.

Silence for three minutes.

**[02:57 AEST] Thomas Adeyemi**: Rollback took 23 minutes. Why.

**[02:58 AEST] Liam Kowalski**: I had to find the old image tag, pull up the YAML, edit it, apply it, wait for pods to terminate and restart.

**[02:59 AEST] Thomas Adeyemi**: You were editing Kubernetes YAML in production at 2 AM.

**[03:00 AEST] Liam Kowalski**: Yes.

**[03:01 AEST] Thomas Adeyemi**: That's a process failure, not a people failure. Fix the process.

**[03:02 AEST] Thomas Adeyemi**: Also — why did this hit ALL Australian sites? Why weren't we testing on a subset first?

**[03:03 AEST] Liam Kowalski**: We don't have that infrastructure.

**[03:04 AEST] Thomas Adeyemi**: Build it.

---

Post-incident, you write a two-page postmortem. The root causes:

1. No progressive rollout: the deploy went to 100% of Australian sites simultaneously.
2. No automated rollback trigger: a human had to notice the error rate, find an on-call engineer, and manually intervene.
3. Manual kubectl in production: rollback required editing live YAML files, introducing delay and human error risk.
4. No pre-deploy validation: the `site_classification_code` requirement was a breaking change that should have been caught by contract testing.

The Monday morning meeting has a new agenda item you didn't schedule.

**Thomas Adeyemi**: "I want progressive delivery for every deploy touching Australian permit workflows. If a new deploy breaks permits for 50 sites, I want it rolled back automatically before it reaches site 51."

**Liam**: "That's... a non-trivial system."

**Thomas Adeyemi**: "I know. That's why I hired a Staff Architect."

---

**Marcus Webb [DM — Tuesday]**

**Marcus Webb**: "Saw your postmortem. Good write-up. One question: you said 'canary to 5% of traffic.' For a B2B construction platform, what does 5% of traffic mean? 5% of requests? 5% of users? 5% of construction sites? Are those the same thing? What should the blast radius unit be?"

---

### Problem Statement

BuildRight's deployment process is fully manual: engineers edit Kubernetes YAML in production, rollbacks take 23 minutes, and every deploy targets 100% of production traffic simultaneously. A breaking change in permit submission logic affected 2,000 Australian construction sites during active hours. The platform needs a progressive delivery infrastructure that limits blast radius to a configurable percentage of construction sites (not just traffic percentage), enables automated rollback on SLO breach, and eliminates manual kubectl operations in production through a GitOps workflow.

The challenge is that "5% of traffic" is meaningless for BuildRight — the blast radius unit must be construction sites (organizations), because a permit failure affects all users at that site simultaneously.

---

### Explicit Requirements

1. Progressive rollout using Argo Rollouts with canary strategy: start at 5% of construction sites, promote to 25%, 50%, 100% on metric gates.
2. Site-based canary routing (not request-percentage): a given construction site must be in either canary or stable — never split between them.
3. Automated rollback: if error rate > 1% or P99 latency > 2x baseline during canary period, automatically roll back without human intervention.
4. Argo Rollouts analysis templates: define the metrics that gate promotion (error rate, permit submission success rate, P99 latency).
5. GitOps workflow: all production deployments via ArgoCD. No direct kubectl access to production. Rollback = git revert + ArgoCD sync.
6. Rollback SLA: automated rollback must complete within 3 minutes of metric threshold breach (vs current 23-minute manual rollback).
7. Deployment audit log: every deploy, promotion step, and rollback logged with actor (human or automated), timestamp, and metric snapshot.
8. Canary smoke tests: before any promotion, run a suite of synthetic permit submission tests against the canary group.
9. Emergency override: on-call engineer can force immediate rollback via a single command (not YAML editing).
10. Notification: PagerDuty alert when canary auto-rollback fires; Slack notification on each promotion step.

---

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's DM. He asks what the "blast radius unit" should be. Site-based canary routing means you need a routing layer that maps `site_id` → canary/stable. Where does this mapping live? How is it maintained as sites are added to the canary group during promotion? What happens if a canary site has a user who is mid-session when promotion steps change the routing?
- **Hint**: Re-read the incident timeline. The building inspector was on-site during the outage. Thomas says "34 contractors are now in a gray zone on their approval status." This means the canary system must handle in-flight transactions: if a site moves from stable to canary mid-rollout, what happens to permit submissions that started on stable and are mid-processing?
- **Hint**: Re-read Thomas's final message: "5% of Australian sites." The canary should be geographically constrained initially. Australian permit workflows have AU-specific validation logic. A canary that includes US or EU sites does not test the AU code path. The canary selection strategy must be region-aware.
- **Hint**: Re-read the root cause analysis: "The `site_classification_code` requirement was a breaking change that should have been caught by contract testing." Progressive delivery reduces blast radius but does not prevent the class of bug that caused this incident. Contract testing (consumer-driven contracts, Pact) is a prerequisite. Your design must include this, even though it isn't asked for explicitly.

---

### Constraints

- **Scale**: 40,000 active construction sites. 2,000 Australian sites.
- **Canary increment**: 5% → 25% → 50% → 100% (sites, not requests).
- **Canary hold time**: minimum 15 minutes at each step before promotion (configurable).
- **Automated rollback trigger**: error rate > 1% OR P99 latency > 2x baseline for 5 minutes.
- **Rollback SLA**: < 3 minutes from threshold breach to 0% canary.
- **GitOps**: ArgoCD managing all production Kubernetes manifests. Git is the source of truth.
- **Analysis interval**: Prometheus metrics sampled every 30 seconds. Analysis runs every 60 seconds.
- **Team**: 2 engineers. Eight weeks to implement.
- **Infrastructure**: Kubernetes (EKS), Argo Rollouts, ArgoCD, Prometheus, PagerDuty, Slack.
- **Permit submission SLA (contractual)**: 99.5% success rate. Current: 99.97% when healthy.

---

### Your Task

Design the progressive delivery infrastructure for BuildRight. Address the site-based canary routing mechanism, the Argo Rollouts configuration (canary steps + analysis templates), the GitOps deployment workflow, and the automated rollback pipeline. Explicitly address the in-flight transaction problem and the region-aware canary selection strategy.

---

### Deliverables

- [ ] **Mermaid architecture diagram** — show: ArgoCD → Argo Rollouts → Kubernetes deployment; site routing layer (canary vs stable); Prometheus analysis; automated rollback flow
- [ ] **Argo Rollouts canary configuration** — define the rollout steps (percentages, pause durations, analysis template references), the AnalysisTemplate (metrics: error rate, P99 latency, permit success rate), and the rollback trigger thresholds
- [ ] **Site routing schema** — `canary_sites` table or Redis structure mapping `site_id` → `variant` (canary/stable) + `promoted_at` timestamp; describe the promotion algorithm (how sites are selected for canary)
- [ ] **Scaling estimation** — calculate: Prometheus metric cardinality for per-site error rate tracking at 40,000 sites; Kafka event volume for deployment audit log; storage for 1 year of rollout history
- [ ] **Tradeoff analysis** — minimum 3: (1) site-based vs request-based canary routing (correctness vs simplicity), (2) automated rollback (speed) vs human-in-the-loop (control), (3) GitOps purity vs emergency override capability
- [ ] **In-flight transaction handling** — describe what happens to a permit submission that starts on `stable` pod and receives a response from a `canary` pod mid-request; define the session affinity strategy
- [ ] **GitOps workflow** — step-by-step: from engineer merging a PR to production reaching 100% canary; include the rollback-as-git-revert flow

---

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
