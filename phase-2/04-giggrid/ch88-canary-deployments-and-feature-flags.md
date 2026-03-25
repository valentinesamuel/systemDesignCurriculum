---
**Title**: GigGrid — Chapter 88: Canary Deployments and Feature Flags
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #deployment #canary #feature-flags #blue-green #rollback #automation
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 102, Ch. 129
**Exercise Type**: System Design / Incident Response
---

### Story Context

**#incidents — GigGrid**
`Friday 18:07` **pagerduty-bot**: 🔴 P1 ALERT — job_dispatch_api: error_rate > 5%
`18:08` **kai.moreau**: we just deployed v2.14.3. deploy finished at 18:04.
`18:09` **you**: related?
`18:10` **kai.moreau**: yes. looking at logs now. the new job acceptance flow has a null pointer on the geo_restriction field for UK workers. we didn't test UK-specific geo restrictions in staging.
`18:11` **amara.chen**: how many workers affected?
`18:12` **kai.moreau**: about 40,000 workers in the UK can't accept jobs right now. they're getting 500 errors.
`18:13` **amara.chen**: that's 2% of our active worker base. and it's Friday evening in the UK — peak shift-seeking hours.
`18:14` **kai.moreau**: we need to roll back
`18:15` **you**: how?
`18:16` **kai.moreau**: ...we need to redeploy the previous version. we don't have a rollback script. the deploy team just ran kubectl apply with the new image tag.
`18:17` **you**: so we're re-deploying manually. how long?
`18:18` **kai.moreau**: 14 minutes. I'm starting now.
`18:19` **amara.chen**: 14 minutes × 40,000 workers unable to accept jobs on a Friday evening. this is our third deployment incident this quarter.

**Post-Incident Review — Monday Morning**
*Present: Amara Chen, You, Kai Moreau, Kenji Mori (Enterprise Relations)*

**Kenji**: Three deployment incidents this quarter. BuildReady Staffing called me Friday. Their London dispatch team works Friday evenings. They had 40,000 workers unable to accept shift assignments for 14 minutes. They're invoking the SLA breach clause in their contract. That's a €50,000 penalty.

**Amara**: What's the current deployment process?

**Kai**: We build a Docker image, run CI/CD tests in staging, and do a full rollout via `kubectl apply`. No staged rollout. No canary. No automated rollback.

**Amara**: *(to you)* Fix it.

**You**: I want to design this properly rather than patch it. The problem isn't just the rollback time — it's that we had no way to detect the geo_restriction bug before it hit 100% of UK workers. A canary deployment to 5% of UK workers would have caught this before it affected 40,000 people.

**Kai**: Our mobile app makes this complicated. Workers have the app installed. If we ship a breaking API change, the old app version calls the new API. The canary isn't just the backend — it has to account for the client version.

**You**: That's why we need feature flags. The geo_restriction feature should have been gated behind a flag. We ship the code to 100% of servers. We enable it for 5% of workers via a flag. We watch metrics. We roll out to 25%, then 100%, then retire the flag.

**Amara**: Feature flags add complexity. Who manages 200 feature flags across 12 services?

**You**: That's the operational challenge. The alternative is what we have now.

---

### Problem Statement

GigGrid has no staged deployment capability. Deployments are full rollouts with no canary, no feature flags, and manual rollback taking 14 minutes. Three deployment incidents this quarter have caused worker-facing outages during peak hours. You need to design a canary deployment system with automated rollback and a feature flag infrastructure for gradual feature rollouts.

---

### Explicit Requirements

1. New deployments must be routable to a percentage of workers (canary: 5% → 25% → 100%)
2. Automated rollback must trigger within 2 minutes of error rate or latency SLO breach
3. Feature flags must be per-worker (not per-server) to enable gradual rollout by worker segment
4. Feature flag changes must propagate to all servers within 10 seconds
5. Rollback must complete in < 2 minutes (vs current 14 minutes)
6. The solution must handle mobile API clients that may be using older app versions

---

### Hidden Requirements

- **Hint**: Re-read Kai's concern: "mobile app makes this complicated." Workers with older app versions are calling the new API. If you ship a new API version as a canary to 5% of *servers*, the routing is server-side — but old mobile clients can be routed to either the canary or the stable server. How does your canary strategy handle the client version dimension?

- **Hint**: Re-read Amara's question: "Who manages 200 feature flags?" Flag sprawl is a real operational problem. If flags are never retired, the codebase fills with dead code paths. What's the lifecycle management policy for feature flags in your design?

---

### Constraints

- 8M registered workers, 2M active daily
- 30 countries with country-specific features (geo restrictions, compliance rules)
- Mobile app: iOS + Android (multiple active versions in the wild at any time)
- Current infrastructure: Kubernetes, no existing feature flag system
- Error rate SLO: < 0.5%; rollback threshold: > 2%
- Kenji's mandate: no more deployment-related SLA breaches

---

### Your Task

Design the canary deployment and feature flag infrastructure for GigGrid. Include the traffic routing mechanism, automated rollback triggers, and feature flag lifecycle management.

---

### Deliverables

- [ ] Mermaid diagram: Canary deployment architecture (Kubernetes traffic splitting → canary pod → metrics → rollback trigger)
- [ ] Rollback automation: What metrics trigger automated rollback? What's the exact threshold and evaluation window?
- [ ] Feature flag architecture: Design for LaunchDarkly or equivalent (flag definition, evaluation, targeting rules by worker segment, country, or app version)
- [ ] Flag propagation: How changes to a flag reach all servers within 10 seconds
- [ ] Mobile API versioning: How to route old mobile clients to stable API while new clients get the canary
- [ ] Flag lifecycle management: Definition, rollout, stable, deprecation, removal stages — with automated warning for long-lived flags
- [ ] Database migration strategy during canary: The expand/contract pattern for schema changes alongside code canary
- [ ] Tradeoff analysis (minimum 3):
  - Kubernetes traffic splitting (weighted service) vs application-level canary (feature flag in code)
  - LaunchDarkly (managed) vs Unleash (self-hosted) for GigGrid's scale
  - Percentage-based canary vs segment-based canary (country, worker tier)
