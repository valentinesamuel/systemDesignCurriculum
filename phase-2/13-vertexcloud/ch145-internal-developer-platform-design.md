---
**Title**: VertexCloud — Chapter 145: The Paved Road Nobody Walked On
**Level**: Staff
**Difficulty**: 8
**Tags**: #idp #golden-paths #backstage #platform-engineering #self-service-infra #developer-experience
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 147 (VertexCloud GitOps), Ch. 148 (API governance), Ch. 149 (Chaos engineering)
**Exercise Type**: System Design
---

### Story Context

**Your first two weeks: user research**

You don't write a line of architecture in your first two weeks. Instead, you interview 20 engineers from 6 different teams. You ask the same three questions to everyone:

1. "Why didn't you use the previous IDP?"
2. "What is the biggest friction point in your current deploy workflow?"
3. "If I could remove one thing from your day that wastes your time, what would it be?"

You compile the responses into a document and present it to Yael Cohen and Reza Tehrani.

---

**Platform Engineering Weekly — Week 3**
**Presenter**: [you]
**Audience**: Yael Cohen, Reza Tehrani, Lucas Oliveira (Engineering Manager, Pilot Team), and platform team

**[you]**: The first IDP failed because it required engineers to learn a new DSL to describe their infrastructure. Here is the feedback from Team Helios (one of the teams that tried it): "I spent two weeks learning the IDP's templating language. At the end of two weeks, I could do 60% of what I could do with Terraform in the first day. I gave up."

**[you]**: The second IDP failed because it was built without compliance input. Audit logging was added as an afterthought, the compliance team reviewed it six weeks into the rollout, found three GDPR gaps, and halted deployment. Six weeks of platform engineering work, discarded.

**Reza Tehrani**: So what's different this time?

**[you]**: Two changes in how we build it. One: we start with what engineers already know. The platform wraps Terraform, not replaces it. Engineers write Terraform. The platform handles the scaffolding, state management, secrets injection, and CI/CD wiring. They get the familiarity of Terraform with the consistency of a managed platform. Two: compliance and security review every design decision before implementation. Not after.

**Yael Cohen**: The compliance review requirement was in both previous attempts. What makes this time different?

**[you]**: Both previous attempts brought compliance in at "pre-launch review." I'm proposing compliance as a design partner from day 1. Different timing, different relationship.

**Lucas Oliveira**: *(representing the pilot team)* My team has 40 engineers. The reason we don't use the golden path is it doesn't cover scheduled jobs. We have 23 scheduled jobs. They're all snowflake configurations. If the IDP doesn't handle scheduled jobs, I can't adopt it — I'd have two systems.

**[you]**: I've inventoried all 14 teams' infrastructure types. Scheduled jobs are the third most common pattern, after web services and async workers. The golden path must cover at minimum: web services, async workers, scheduled jobs, and static sites. Those four cover 84% of all infrastructure deployed at VertexCloud. Everything else is an escape hatch.

**Lucas Oliveira**: And the escape hatch actually exists? I'm not blocked if I have a weird requirement?

**[you]**: The escape hatch actually exists. It's documented. It's not a dead end — it leads to a "bring your own Terraform" path that still gets the platform's CI/CD and observability integration. You don't get the opinionated guardrails, but you're not isolated from the platform.

---

### Problem Statement

VertexCloud's two previous IDP attempts failed due to DSL friction (engineers rejected a new language) and late compliance integration (audit logging gaps discovered post-launch). A third attempt must: use Terraform as the engineering interface, support the 4 most common infrastructure patterns natively, provide real escape hatches for non-standard patterns, and have compliance integrated from day 1 of design.

---

### Explicit Requirements

1. Engineer interface: Terraform (not a new DSL). Platform handles scaffolding, state, secrets, CI/CD.
2. Golden path must cover: web services, async workers, scheduled jobs, static sites (84% of all VertexCloud infrastructure)
3. Escape hatch: documented path for non-standard infrastructure that still integrates with CI/CD and observability
4. Compliance integration: GDPR audit logging and data residency controls built into every golden path template
5. Self-service: engineer can provision a new service through the IDP in ≤ 30 minutes without platform team intervention
6. Adoption metric: 60% of VertexCloud teams using the golden path within 12 months

---

### Hidden Requirements

- **Hint**: Re-read Lucas Oliveira's objection: "I'd have two systems." The previous IDPs failed partly because they didn't cover enough use cases, forcing teams to maintain the IDP AND their old tooling simultaneously. Your adoption analysis must show the minimum feature coverage required to be the "one system" for each team — and validate that 84% coverage is sufficient for that claim.

- **Hint**: Reza Tehrani asked "what's different this time?" He's asking an organizational question, not a technical one. What governance mechanism ensures the compliance review actually happens early? An informal promise to "bring compliance in earlier" is not governance. What is the structural change that makes early compliance review mandatory?

---

### Constraints

- 14 product teams, 2,000 engineers
- Previous IDP attempt 1: DSL-based, 0% adoption after 6 months
- Previous IDP attempt 2: Backstage-based, compliance shutdown after 6 weeks
- Golden path coverage target: 84% of infrastructure patterns (4 types)
- Self-service provisioning SLA: ≤ 30 minutes (from "I want a new service" to "service is deployable")
- Budget: Reza Tehrani has approved 12 months of platform team time + $200k/year for tooling

---

### Your Task

Design the VertexCloud Internal Developer Platform architecture and adoption strategy.

---

### Deliverables

- [ ] IDP architecture layers: self-service portal + Terraform module library + CI/CD pipeline + observability integration
- [ ] Golden path templates: Terraform module design for web service, async worker, scheduled job, static site
- [ ] Escape hatch design: "bring your own Terraform" integration with platform CI/CD and observability
- [ ] Compliance integration: how GDPR audit logging and data residency are built into every template (not retrofitted)
- [ ] Adoption strategy: which team to pilot with, pilot success criteria, rollout sequencing to remaining 13 teams
- [ ] Self-service portal options: Backstage vs Port vs custom portal — selection criteria based on previous failure analysis
- [ ] Success metrics: deployment frequency, incident rate, time-to-first-deployment for new services — baseline and 12-month target
- [ ] Tradeoff analysis (minimum 3):
  - Terraform module library vs custom DSL — adoption friction vs standardization enforcement
  - Backstage (open source, extensible) vs Port (SaaS, less customizable) vs custom — build vs buy for VertexCloud's requirements
  - Opinionated platform (high consistency, less flexibility) vs flexible platform (high adoption, less consistency) — the 80/20 rule application
