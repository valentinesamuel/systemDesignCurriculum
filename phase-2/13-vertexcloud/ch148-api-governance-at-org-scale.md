---
**Title**: VertexCloud — Chapter 148: 300 Internal APIs, Zero Deprecation Policy
**Level**: Staff
**Difficulty**: 7
**Tags**: #api-governance #versioning #deprecation #breaking-changes #api-gateway #contract-testing
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 136 (Axiom data mesh), Ch. 29 (CloudStack API gateway), Ch. 145 (VertexCloud IDP)
**Exercise Type**: System Design
---

### Story Context

**Incident Postmortem — #incidents**
**Tuesday 15:00 — Postmortem for P1 incident last Thursday**

**Amara Chen**: Root cause confirmed. Team Helix changed a field name in their internal API — `userId` to `user_id`. This is a snake_case standardization change. No announcement. No version bump. Seventeen downstream consumers broke simultaneously when Team Helix deployed at 14:23 Thursday.

**Reza Tehrani**: How many hours of outage?

**Amara Chen**: Three hours for all 17 affected services. Combined revenue impact estimate: $240,000 in SLA penalties.

**Reza Tehrani**: And this field rename — was it intentional?

**Amara Chen**: Intentional, yes. Breaking change, yes. But Team Helix says "we checked — nobody was using the old field name." They used a grep of the internal monorepo. They missed 5 of the 17 affected consumers because those consumers are in separate repositories.

**[you]**: This is the fourth breaking change incident in 3 months. Different teams, same root cause: undocumented API contracts, no versioning, no deprecation process, no detection.

**Amara Chen**: I've catalogued all 300 internal APIs. 12% have OpenAPI specs. 4% have versioning. 0% have a formal deprecation process.

**Reza Tehrani**: This is a systemic problem. [you] — I want a design for API governance by end of month.

---

**1:1 — Yael Cohen → [you] — Thursday 10:00**

**Yael Cohen**: The API governance problem has a political dimension. If we mandate OpenAPI specs for all 300 APIs before anyone can deploy changes, we stop the company. It takes time to write specs. Teams will resist.

**[you]**: We can't mandate retroactive specs for all 300 APIs. We can mandate specs going forward: any new API, any breaking change to an existing API.

**Yael Cohen**: And for the existing 300 that have no spec?

**[you]**: We take a traffic-based approach. The APIs with the most downstream consumers get specced first — those are the highest blast-radius risks. I can pull traffic data from the API gateway to rank by consumer count. Top 20 APIs by consumer count probably represent 70% of the integration surface area.

**Yael Cohen**: Who writes the specs?

**[you]**: The teams that own the APIs. But we give them tooling that makes it fast: spec generation from API gateway traffic logs (we can auto-generate an OpenAPI spec from observed request/response patterns), a linter that enforces naming conventions, and a CI check that fails on breaking changes.

**Yael Cohen**: The CI check for breaking changes — how does that work?

**[you]**: Schema diff tool. You commit a new version of your API spec. CI diffs it against the last published version. If any field is removed, renamed, or type-changed — that's a breaking change. CI fails. You must either add a version bump or prove there are no consumers.

**Yael Cohen**: Can we prove there are no consumers automatically?

**[you]**: Not definitively — we can only prove there are no *known* consumers in our API gateway logs. That would have caught the Helix incident — their 17 consumers were hitting the gateway.

---

### Problem Statement

VertexCloud has 300 internal APIs with no standardized contracts, no versioning, and no deprecation process. Four breaking change incidents in 3 months have caused $500k+ in SLA penalties. A governance system must: detect breaking changes in CI, mandate specs for new APIs and breaking changes, rank and address the highest-risk existing APIs, and provide a formal deprecation pipeline.

---

### Explicit Requirements

1. Breaking change detection in CI: any PR that modifies an API spec must pass a schema diff check
2. New APIs must have an OpenAPI spec before first deployment
3. Existing APIs: top 20 by consumer count must have specs within 90 days
4. Deprecation pipeline: announce (deprecation notice + migration guide) → monitor (track consumer migration) → enforce (reject calls) → remove (clean up spec)
5. API catalog: all 300 internal APIs discoverable with spec, owner, consumer count, and deprecation status
6. Consumer-driven contract testing: any team that consumes an API can publish a Pact contract that runs in the provider's CI

---

### Hidden Requirements

- **Hint**: Amara Chen said Team Helix used "a grep of the internal monorepo" to check for consumers and missed 5 in separate repositories. But even a complete API gateway log search has a gap: consumers that haven't called the API recently (dormant consumers, seasonal features) won't appear in recent traffic. Your "no known consumers" check must include a dormancy window: flag any API where consumers were active in the last 12 months, not just the last 30 days.

- **Hint**: The deprecation pipeline has a political challenge: a team that's deprecated can't block on a team that's slow to migrate. Your design needs a "forced migration" mechanism — after the announce and monitor phases, if a consumer hasn't migrated by the enforcement date, what happens? Cutting them off breaks their service. Extending the deadline rewards procrastination. What is the correct forcing function?

---

### Constraints

- 300 internal APIs, 12% have OpenAPI specs
- Top 20 APIs by consumer count: estimated 70% of total API call volume
- CI run time budget: breaking change detection must add ≤ 30 seconds to existing CI pipeline
- API gateway: Kong (already deployed, has traffic logs)
- Deprecation timeline standard: 90 days from announcement to enforcement minimum
- Consumer count data: available from API gateway logs (last 30 days); dormancy gap: need 12-month lookback

---

### Your Task

Design VertexCloud's API governance system for 300 internal APIs.

---

### Deliverables

- [ ] Breaking change detection: schema diff tool integrated in CI, detection rules (what constitutes breaking vs non-breaking)
- [ ] API spec generation: auto-gen from API gateway traffic logs — what's captured, what requires manual completion
- [ ] API catalog design: how 300 API specs are stored, versioned, and discovered
- [ ] Deprecation pipeline: 4-stage lifecycle with specific timelines, owner responsibilities, and forced-migration mechanism
- [ ] Consumer-driven contract testing: Pact integration in CI — provider test running against published consumer contracts
- [ ] Risk ranking: how to prioritize which of the 288 spec-less APIs to tackle first (consumer count × change frequency)
- [ ] Dormancy window: 12-month lookback for consumer detection, how dormant consumers are handled in deprecation
- [ ] Tradeoff analysis (minimum 3):
  - Additive-only breaking change rule (never remove, only add) vs versioned breaking changes — operational simplicity vs API hygiene
  - OpenAPI vs gRPC Protobuf for internal API contracts — the VertexCloud language/ecosystem fit
  - Pact consumer-driven contracts vs provider-published schemas — who owns the contract?
