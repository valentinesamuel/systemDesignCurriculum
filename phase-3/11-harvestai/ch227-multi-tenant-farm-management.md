---
**Title**: HarvestAI — Chapter 227: One Account, Eight Hundred Farms
**Level**: Staff
**Difficulty**: 8
**Tags**: #multi-tenancy #rbac #hierarchical-permissions #saas #data-isolation
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 30 (CloudStack multi-tenant federation), Ch. 187 (ShieldMutual multi-line insurance), Ch. 75 (VertexCloud platform multi-tenancy)
**Exercise Type**: System Design
---

### Story Context

**1:1 Meeting Transcript — Priya Sundaram and You**
*Thursday, 3:00 PM, Priya's office. Small room, whiteboard on one wall, a dead plant in the corner that nobody has removed. The afternoon sun makes the whiteboard glare.*

---

**Priya:** "Close the door. I want to run this by you before the sales team makes any more promises."

**You:** "What happened?"

**Priya:** "AgriCorp. You know them?"

**You:** "Commercial farming conglomerate. Listed on NYSE. Something like 800 farm operations across the Midwest and Brazil."

**Priya:** "812 operations, to be precise. They want to onboard their entire portfolio onto HarvestAI. It's a $2.4M annual contract — our largest by a factor of six. I've been working this deal for eight months. They signed an LOI last week."

**You:** "That's good news."

**Priya:** "It would be, except for one problem. Their procurement team sent over their data access requirements document. Fifty-two pages. I read it last night."

*(She slides a printed document across the desk. You flip to a highlighted section.)*

**You:** *(reading)* "'Each operating subsidiary must have organizational isolation from all other operating subsidiaries. Regional managers must have read access to all farms within their region. Farm managers must have read/write access to their farm only. Corporate ESG team must have read access to carbon credit data across all farms but cannot see operational farm data. External crop consultants must have time-limited read access to specific fields assigned by the farm manager. Third-party agronomists hired by AgriCorp corporate must have read access to all farms in a given crop type cohort, but only for the current season's data.'"

**Priya:** "Keep going. Paragraph three."

**You:** *(reading)* "'Billing must be segmented: each operating subsidiary has its own billing account. Some subsidiaries are on Premium tier; others are on Standard tier. Individual farm managers must be able to purchase add-on features for their farm only, which bill to the subsidiary account. Corporate-level purchases bill to the master account. AgriCorp corporate wants a consolidated invoice showing charges broken down by subsidiary, farm, and feature.'"

**You:** "Our current data model is one tenant equals one farm."

**Priya:** "Correct."

**You:** "And the billing model is one billing account per tenant."

**Priya:** "Also correct."

**You:** "So the entire multi-tenancy model has to change."

**Priya:** "Now you understand the problem."

*(Pause. You look at the whiteboard.)*

**You:** "Can I draw something?"

**Priya:** "Please."

*(You go to the whiteboard and draw a hierarchy.)*

```
AgriCorp (Master Account)
├── Midwest Operations (Subsidiary)
│   ├── Iowa Region (Regional Org)
│   │   ├── Farm A (Operation)
│   │   ├── Farm B (Operation)
│   │   └── Farm C (Operation)
│   └── Illinois Region (Regional Org)
│       ├── Farm D (Operation)
│       └── Farm E (Operation)
├── Brazil Operations (Subsidiary)
│   └── Mato Grosso Region (Regional Org)
│       ├── Fazenda 1 (Operation)
│       └── Fazenda 2 (Operation)
└── ESG Office (Functional Unit — cross-cutting)
```

**You:** "This is a four-level hierarchy. Account → Subsidiary → Regional Org → Farm Operation. And then there are cross-cutting functional units like the ESG team that don't fit neatly into the hierarchy — they need permissions that span multiple subsidiaries."

**Priya:** "How many other customers might have a structure like this?"

**You:** "If AgriCorp works, we'll sell to more commercial farming conglomerates. I'd estimate maybe 20 customers in the next two years with a similar structure. Smaller, maybe 50-200 farms each instead of 800."

**Priya:** "So this isn't a custom solution for one customer. It's a product decision."

**You:** "It's a product decision. We need to support hierarchical organizations as a first-class concept."

**Priya:** "What does that cost us in terms of existing customers?"

**You:** "Our current 7,999 other customers are single-farm or small multi-farm (2-5 farms, same owner). They should just become leaf nodes under a single organization with no sub-structure. Migration should be invisible to them."

**Priya:** "And the data isolation?"

**You:** "Right now we use PostgreSQL row-level security. Every table has a `farm_id` column, and RLS policies filter on the user's `farm_id` from the JWT. That model breaks for hierarchical access — a regional manager's JWT doesn't have a list of all their farm IDs embedded in it."

**Priya:** "How do other companies solve this?"

**You:** "Two main approaches. One: encode the hierarchy in the token — include a list of all accessible resource IDs in the JWT. Problem: at AgriCorp's scale, the JWT becomes unwieldy. A regional manager in Iowa might have 200 farms in their scope; listing 200 farm UUIDs in a JWT is technically possible but ugly, and it doesn't handle dynamic changes to the hierarchy."

**Priya:** "And the second approach?"

**You:** "Replace the RLS model with an authorization service. JWT carries the user's identity and role. Authorization service knows the org hierarchy and computes what resources the user can access at query time. More flexible, but now you have a new critical dependency."

**Priya:** "Which do you recommend?"

**You:** "I need to think through the cardinality. Let me come back to you with a design by end of day Friday."

**Priya:** "You have until Friday noon. The sales team is presenting to AgriCorp's CTO on Monday."

*(She stands up, which means the meeting is over.)*

**Priya:** "One more thing. Their security team will want a data isolation audit. They'll want to know how we prevent someone at Fazenda 1 from seeing data at Farm A in Iowa. Make sure the design has a clear answer for that."

**You:** "Will do."

**Priya:** "Oh — and the external crop consultants with time-limited access. Their procurement doc says access must expire within 24 hours of revocation. Not eventual consistency. Hard expiration."

*(She looks at you steadily.)*

**You:** "I'll address that specifically."

---

**Direct Message — Carlos Mendes → You, Thursday 4:55 PM**

**Carlos Mendes** [4:55 PM]
I heard about the AgriCorp design meeting
one thing Priya probably didn't mention: AgriCorp's Brazil operations use a different login system
they're on Azure AD (Entra ID)
the Iowa farms use Google Workspace
some of the older family-operated farms that AgriCorp acquired still use a homegrown auth system with static usernames and passwords that someone wrote in 2014
so whatever we build has to federate identity from at least three different IdPs
and probably more as they acquire new farms

---

### Problem Statement

HarvestAI's current multi-tenancy model was designed for a flat structure: one tenant equals one farm, one billing account, one set of users. The design uses PostgreSQL row-level security (RLS) with `farm_id` as the isolation key in every table, and JWT claims that contain the user's `farm_id`. This model breaks for AgriCorp, a commercial farming conglomerate with a four-level organizational hierarchy: corporate account → subsidiaries → regional organizations → individual farm operations. Approximately 800 farms will be onboarded under this single contract.

The requirements include: sub-organization data isolation (Midwest Operations cannot see Brazil Operations data), hierarchical read access (regional managers see all farms in their region), cross-cutting functional access (ESG team reads carbon data across all subsidiaries), external consultant time-limited access (hard expiration within 24 hours of revocation), segmented billing (per-subsidiary, per-farm, with consolidated invoicing at corporate level), and multi-IdP federation (Azure AD, Google Workspace, legacy auth). The design must also handle the 7,999 existing single-farm customers without disruption — they must be silently migrated to a compatible model.

---

### Explicit Requirements

1. Support a four-level organizational hierarchy: Account → Subsidiary → Region → Farm; nodes at any level can have users with scoped permissions
2. Support cross-cutting functional access: a team (e.g., ESG Office) can be granted access to a specific data type (e.g., carbon_reports) across all nodes in a subtree, without granting access to other data types
3. External consultant access must support time-limited grants with hard expiration: once revoked or expired, the user must not be able to access the resource within 24 hours (not eventually consistent)
4. Billing must be hierarchical: charges can be attributed at farm level, roll up to subsidiary level, and consolidate at account level; each level can have its own payment method
5. The system must federate identity from multiple IdPs per account: an AgriCorp account can have users from Azure AD, Google Workspace, and legacy auth systems simultaneously
6. Data isolation must be provable: the system must be able to demonstrate (for an audit) that a user at Farm A cannot access Farm B's data, even within the same subsidiary
7. Existing single-farm customers must be migrated automatically and transparently; their user experience must not change

---

### Hidden Requirements

1. **Hint: re-read Carlos's message about AgriCorp's Brazil operations using Azure AD and the Iowa farms using Google Workspace.** He adds that legacy family farms use a "homegrown auth system with static usernames and passwords written in 2014." This homegrown system almost certainly does not support OIDC or SAML. How do you federate a user from a non-standard IdP into HarvestAI's authorization model without building a custom identity bridge? And how do you handle the case where AgriCorp acquires a new farm in the future that uses yet another IdP you haven't seen before?

2. **Hint: re-read Priya's final line: "access must expire within 24 hours of revocation. Not eventual consistency. Hard expiration."** JWT-based auth systems typically have a token validity period — if you issue a JWT to a consultant that is valid for 24 hours, you cannot "revoke" it before it expires without either checking a revocation list on every request (adding latency) or using short-lived tokens. What is the tradeoff between token duration, revocation latency, and performance? At HarvestAI's scale (8,000 + AgriCorp's users), how many tokens are in flight at any moment, and what is the cost of a revocation list check per request?

3. **Hint: re-read the organizational hierarchy you drew on the whiteboard.** You propose replacing RLS with an authorization service. But the authorization service is now a critical dependency — every API request must call it. What happens when the authorization service is slow or unavailable? An authorization service outage means no user can access any farm data. How do you design for authorization service availability? Can you cache authorization decisions, and if so, what is the maximum cache TTL before you violate the 24-hour hard-expiration requirement for consultant access?

4. **Hint: re-read the billing requirement: "each operating subsidiary has its own billing account; farm managers can purchase add-ons that bill to the subsidiary account; corporate purchases bill to the master account."** This is a hierarchical billing model. When a farm manager in Iowa purchases an add-on, who approves it — the farm manager, the Iowa Regional manager, or AgriCorp corporate? The procurement doc implies farm managers have authority for their own farm's purchases. But what is the credit limit policy? What happens if a subsidiary's payment method fails and there are active add-on purchases at 50 farms underneath it?

---

### Constraints

- **Account scale**: 8,000 existing single-farm accounts + AgriCorp (812 farms, ~4 org levels deep); project 20 more enterprise accounts (50–200 farms each) within 24 months
- **User scale**: currently ~24,000 users (3 per farm average); AgriCorp adds ~3,000 users; 24-month projection: ~80,000 users total
- **Authorization decision latency**: must not add more than 15ms P99 to API response time; current P99 API latency is 120ms
- **Revocation hard expiration**: consultant access must be inaccessible within 24 hours of revocation (hard requirement from AgriCorp's security team)
- **JWT token validity**: currently 1 hour; cannot be longer than 1 hour for security compliance
- **Multi-IdP**: Azure AD (OIDC), Google Workspace (OIDC), legacy systems (no standard protocol — HTTP basic or custom token)
- **Billing hierarchy depth**: up to 4 levels; each level can have a separate payment method or inherit from parent
- **Database**: PostgreSQL 15 with RLS; existing RLS policies on 47 tables using `farm_id`
- **Team size**: 12 engineers total; you and Carlos will lead this design; estimated implementation time: 8–12 weeks
- **Migration constraint**: zero downtime for existing 7,999 customers during schema migration; no user-visible changes
- **AgriCorp contract SLA**: data isolation audit must pass within 60 days of onboarding
- **Authorization service availability requirement**: the auth decision path must have 99.95% uptime (higher than the main application's 99.9% SLA)

---

### Your Task

Design HarvestAI's hierarchical multi-tenancy architecture. The design must address: the organizational hierarchy data model, the authorization model (replacing or augmenting the current RLS approach), the multi-IdP identity federation strategy, the time-limited access grant mechanism with hard expiration, the hierarchical billing model, and the migration path from the current flat model.

The design must be practical for a 12-person team to implement in 8–12 weeks. It must not require a complete rewrite of the 47 RLS-protected tables. It must support AgriCorp's audit requirements.

---

### Deliverables

- [ ] **Mermaid architecture diagram**: full system showing org hierarchy model, authorization service, IdP federation layer, billing hierarchy, and how they interact with the existing PostgreSQL/RLS layer
- [ ] **Database schema**:
  - `organizations` table: hierarchical (adjacency list or LTREE), with org type, parent reference, billing tier
  - `org_members` table: user ↔ org membership with role and scope
  - `access_grants` table: time-limited cross-cutting grants (consultant access), with expiry, revocation timestamp, grant type
  - `billing_accounts` table: hierarchical billing with payment method inheritance
  - `idp_connections` table: per-org IdP configuration (provider type, OIDC/SAML metadata, legacy bridge config)
  - Migration plan: how existing `farm_id` columns are preserved while new org hierarchy is added
- [ ] **Scaling estimation** (show math):
  - Authorization decision volume: 80,000 users × average 50 API calls/day = X authorization decisions/day; what is the RPS?
  - Authorization cache sizing: if you cache decisions with 5-minute TTL, how many unique cache entries per user? Total cache memory at 80K users?
  - PostgreSQL LTREE traversal: for a 4-level hierarchy with 20,000 farms, what is the query cost of "find all farms in this regional org" compared to a CTE recursive query?
- [ ] **Tradeoff analysis** (minimum 3):
  - JWT-embedded hierarchy claims vs external authorization service: token bloat vs latency vs operational complexity
  - PostgreSQL LTREE vs adjacency list vs materialized path for org hierarchy: query performance vs update complexity vs depth limits
  - RLS preservation vs full authorization service replacement: migration risk vs long-term correctness vs performance
- [ ] **Hard-expiration mechanism design**: describe precisely how consultant time-limited access grants are enforced to within 24 hours of revocation, including the interaction between JWT token validity, revocation list, and cache invalidation
- [ ] **Migration plan**: step-by-step plan for migrating 7,999 existing single-farm customers to the new model with zero downtime, including the RLS policy update strategy for all 47 affected tables

### Diagram Format
All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
