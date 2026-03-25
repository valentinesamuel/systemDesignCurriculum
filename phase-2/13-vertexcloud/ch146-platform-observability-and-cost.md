---
**Title**: VertexCloud — Chapter 146: The $4.2M Infrastructure Bill Nobody Owned
**Level**: Staff
**Difficulty**: 8
**Tags**: #finops #showback #chargeback #cost-attribution #rightsizing #platform-economics #tagging
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 110 (Crestline capacity planning), Ch. 56 (LuminaryAI billing)
**Exercise Type**: System Design
---

### Story Context

**Budget Review — Tuesday 14:00**
**Attendees**: Reza Tehrani (VP Engineering), Yael Cohen (Head of Platform), [you]

Reza opens an AWS Cost Explorer dashboard on the projector. Monthly cloud bill: $4.2M. He draws a red circle around one line item.

**Reza Tehrani**: "Unallocated compute — $1.1M/month." That's 26% of our cloud bill that we cannot attribute to any team, product, or business unit. We don't know who is responsible for it, which means we cannot hold anyone accountable for it, which means nobody is going to optimize it.

**[you]**: How did $1.1M of compute become unallocated?

**Reza Tehrani**: Three reasons I've identified. First: 6 teams are provisioning infrastructure directly in AWS without going through the IDP, so cost attribution tags are not being applied. Second: when engineers leave the company, their personal test environments sometimes stay running. We have 23 instances that are over 6 months old with no associated active employee. Third: one team provisioned a GPU cluster for a 2-week ML experiment in Q2. The experiment ended. The GPU cluster kept running. That cluster alone is $38,000/month.

**Yael Cohen**: The GPU cluster is the one that bothers me most. It's been running for 4 months. $152,000 in compute that generated zero value after week two.

**Reza Tehrani**: *(to you)* I need three things. One: cost attribution for everything — I should be able to see every dollar attributed to a team and a product. Two: automated detection of idle resources. Three: a rightsizing recommendation system that tells teams when they're massively over-provisioned. I need this before Q4 planning in 8 weeks.

---

**Slack DM — Amara Chen (VertexCloud Staff Engineer) → [you] — Wednesday 09:00**

**Amara Chen**: I pulled the resource tagging audit you asked for. Here's what we have:

- Resources with complete tags (team, product, environment): 23%
- Resources with partial tags (some fields missing): 31%
- Resources with no tags: 46%

**[you]**: 46% of our infrastructure has no tags at all?

**Amara Chen**: The teams that went off-path (provisioning directly in AWS) never went through a tagging enforcement step. AWS doesn't enforce tags by default. So you can create 100 EC2 instances with zero tags and it just works.

**[you]**: And the IDP's templates?

**Amara Chen**: Every IDP template applies team, product, environment, and cost-center tags at provisioning time. It's enforced — you can't deploy through the IDP without tags. The problem is the 14 teams that aren't using the IDP.

**[you]**: So the cost attribution problem is a symptom of the IDP adoption problem.

**Amara Chen**: Yes. Every team that adopts the IDP automatically gets cost attribution. Every team that doesn't, doesn't.

**[you]**: That's a powerful IDP adoption argument. Can you quantify the cost waste attributable to teams not using the IDP?

**Amara Chen**: The 23 orphan instances + GPU cluster + untagged compute = roughly $1.3M/month in waste we could eliminate with proper attribution and lifecycle management.

---

### Problem Statement

$1.1M/month of VertexCloud's cloud spend is unattributable to any team because 46% of resources have no cost allocation tags. Teams provision directly in AWS, bypassing IDP tag enforcement. Orphan resources (departed employees' test environments, forgotten GPU clusters) persist for months with no detection. Design the cost attribution and idle resource management system.

---

### Explicit Requirements

1. 100% of cloud resources must have attribution tags (team, product, environment, cost-center) within 6 months
2. Automated idle resource detection: any resource running for > 30 days with CPU < 5% and no associated active user is flagged
3. Team-level cost dashboards: every Engineering Manager sees their team's monthly cloud spend
4. Rightsizing recommendations: for over-provisioned resources (< 20% average CPU), suggest smaller instance type
5. New resource provisioning: IDP-enforced tagging as mandatory step; direct AWS provisioning must also enforce tags (via SCP)
6. Idle resource lifecycle: automated notification at 30 days idle, automated shutdown at 60 days idle (with override mechanism)

---

### Hidden Requirements

- **Hint**: Reza Tehrani wants to hold teams accountable for their cloud costs. There's a political dimension: the current culture is "engineering doesn't worry about cloud costs." Introducing chargeback (teams actually pay from their budget) vs showback (teams see the cost but don't pay) has different political implications. Re-read the requirements: Reza said "hold accountable" — does he mean chargeback or showback? Your design should address this distinction explicitly.

- **Hint**: The GPU cluster: $38,000/month for 4 months = $152,000 total waste. The engineer who provisioned it left the company. The engineering manager whose team the engineer was on is now responsible for this cost — but didn't know the cluster existed. Your idle resource detection must send notifications not just to the resource owner (who may have left) but to the team's current engineering manager.

---

### Constraints

- $4.2M/month total cloud spend
- Unattributable: $1.1M/month (target: $0 unattributable within 6 months)
- Current tagging compliance: 23% fully tagged, 31% partial, 46% none
- Orphan resources: 23 known, estimated 40-60 total
- GPU cluster: $38,000/month, 4 months old, no active owner
- Target waste reduction: $1.3M/month → ≤ $200k/month "acceptable unattributed" waste

---

### Your Task

Design VertexCloud's cloud cost attribution and idle resource management system.

---

### Deliverables

- [ ] Tag taxonomy: required tags, optional tags, tag validation schema
- [ ] Enforcement mechanism: AWS SCP (Service Control Policy) that prevents untagged resource creation
- [ ] Retroactive tagging campaign: strategy for tagging the existing 46% untagged resources
- [ ] Showback vs chargeback design: recommendation with political/organizational justification
- [ ] Team cost dashboard: what data each EM sees, update frequency, drill-down capability
- [ ] Idle resource detection pipeline: criteria, notification workflow, automated shutdown design
- [ ] Rightsizing recommendation engine: how to generate actionable recommendations (not just "this is over-provisioned")
- [ ] GPU/expensive resource lifecycle: special handling for high-cost resources with 14-day idle threshold (not 30)
- [ ] Tradeoff analysis (minimum 3):
  - Chargeback vs showback — accountability vs culture change impact
  - AWS Cost Explorer vs third-party FinOps tools (Apptio, CloudHealth) vs custom — make vs buy
  - IDP-enforced tags vs AWS SCP enforcement — the difference in what each catches
