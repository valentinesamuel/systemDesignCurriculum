---
**Title**: GigGrid — Chapter 87: Platform API Rate Limiting (4th Appearance)
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #rate-limiting #fairness #sliding-window #distributed #redis #api-gateway
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 118, Ch. 124
**Exercise Type**: System Design / Incident Response
---

### Story Context

**#enterprise-support — GigGrid**
`Monday 10:15` **kenji.mori** *(Head of Enterprise Relations)*: I need engineering urgently. We have a problem with our London market.
`10:16` **you**: what's happening?
`10:17` **kenji.mori**: a JP Morgan Chase operations team has been running an automated bot that accepts all available jobs in the London area the instant they're posted. they grab the job, hold it for 5-10 minutes, then cancel if the pay rate isn't high enough. effectively they're squatting on jobs for premium-rate negotiation.
`10:18` **amara.chen**: that's clever. and infuriating.
`10:19` **kenji.mori**: smaller agencies can't compete. by the time a human dispatcher or a slower system sees the job, it's been grabbed by JPM. we've had 3 agencies in London threaten to cancel their contracts this week.
`10:20` **you**: how many jobs is JPM's bot accepting per minute?
`10:21` **kenji.mori**: up to 340 per minute. peak London dispatching is about 500 per minute. they're grabbing 68% of all jobs.
`10:22` **amara.chen**: and they're within their contract limits?
`10:23` **kenji.mori**: their contract says "unlimited job acceptance." we didn't anticipate bots.
`10:24` **you**: we need rate limiting. specifically: fairness-based rate limiting, not just throughput limiting.

**Slack DM — Marcus Webb → You**
`Same afternoon`
**marcus.webb**: I saw the London situation in the enterprise channel. you've seen this before.
**you**: I designed rate limiting at CloudStack (token bucket, per-client) and CivicOS (behavioral, per-ASN) and SkyRoute (multi-class, partner overrides). this is different.
**marcus.webb**: how?
**you**: the JPM bot isn't violating any rate limit. it's working within its contract. the problem isn't rate — it's fairness. they're out-competing smaller players who have an equal right to jobs. this is a market fairness problem wearing a rate limiting costume.
**marcus.webb**: good framing. what's the difference between "rate limiting" and "fair queuing"?
**you**: rate limiting: you can't do more than X per second. fair queuing: everyone gets an equal share of capacity. even if you CAN go faster, you only get your fair share.
**marcus.webb**: and what's weighted fair queuing?
**you**: weighted fair queuing: each client gets a share proportional to their weight. bigger clients get a bigger share, but the share is bounded. the JPM bot gets, say, 15% of job capacity. smaller agencies get 5% each.
**marcus.webb**: that's the design. the question is whether your contracts allow you to cap JPM at 15%.
**you**: that's Kenji's problem. my problem is building the system that enforces it.

---

### Problem Statement

GigGrid's platform API has no fairness controls. A single enterprise client (JP Morgan Chase) is using an automated bot to accept 68% of all London-area jobs instantly, preventing smaller agencies from accessing the platform. The issue is not throughput violation but market fairness — their contract currently allows unlimited acceptance. You need to design a fairness-based rate limiting system (weighted fair queuing) while the contracts are renegotiated.

This is the 4th appearance of the rate limiting system in this curriculum. Evolution: CloudStack (token bucket, per-client throughput) → CivicOS (behavioral, attack-based) → SkyRoute (multi-class, partner overrides) → GigGrid (fairness/weighted fair queuing).

---

### Explicit Requirements

1. Each client must receive a fair share of available jobs in their geographic market
2. Larger clients may receive a proportionally larger share (weighted), but the share is bounded
3. Rate limits must be enforced per geographic market (London market separate from Manchester market)
4. Burst tolerance: clients can briefly exceed their fair share if capacity is unused by others
5. Rate limiting state must be distributed across all API gateway nodes (consistent enforcement)
6. Clients must be able to observe their current rate limit usage via an API response header

---

### Hidden Requirements

- **Hint**: Re-read Marcus Webb's point: "the contracts allow unlimited acceptance." Kenji said contracts need to be renegotiated. But what if JPM refuses to renegotiate? The rate limiting system needs a mechanism to enforce fairness *within* the existing "unlimited" contract — by framing it as "job queue access fairness" rather than "rate limiting." How do you implement this contractually defensible distinction?

- **Hint**: The bot is accepting jobs and cancelling them within 5-10 minutes. This means jobs are being locked up temporarily, reducing effective supply for other agencies. Your rate limiting design might also need a "hold time" component: jobs that are accepted and then cancelled within a short window reduce the acceptor's quota more than normal cancellations. Is this "rate limiting" or "penalty scoring"?

---

### Constraints

- London job market: ~500 jobs/minute peak
- JPM contract: "unlimited" acceptance (must renegotiate to add fairness terms)
- 12 enterprise clients in the London market, varying sizes
- Rate limiting must be enforced across all API gateway nodes (3 nodes in London)
- Redis Cluster available for distributed rate limit state
- Sliding window log from SkyRoute work is already implemented — can be reused

---

### Your Task

Design the weighted fair queuing system for GigGrid's job acceptance API. Build on the sliding window rate limiting from prior chapters but extend it with fairness semantics.

---

### Deliverables

- [ ] Weighted fair queuing design: How to calculate each client's fair share of jobs per minute in their geographic market
- [ ] Mermaid diagram: Rate limiting architecture (API gateway → Redis rate limit state → enforcement decision)
- [ ] Redis Lua script sketch: Fair queuing enforcement — check current usage, compare to fair share weight, allow/deny
- [ ] Burst tolerance design: When unused capacity is available, allow clients to burst above fair share by what amount?
- [ ] Contractual framing: How to implement this as "job queue access fairness" (contractually defensible) rather than "rate limiting" (may violate existing contracts)
- [ ] Hold time penalty: Design for reducing a client's quota when they accept-then-cancel jobs within a short window
- [ ] Comparison to previous rate limiting designs: What changed between CloudStack (token bucket), CivicOS (behavioral), SkyRoute (multi-class), and this design (fair queuing)?
- [ ] Tradeoff analysis (minimum 3):
  - Token bucket vs weighted fair queuing for this use case
  - Per-market vs global fair queuing
  - Hard limits (reject at limit) vs soft limits (delay at limit)
