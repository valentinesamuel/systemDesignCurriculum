---
**Title**: Pattern Summary 4 — API Design, Rate Limiting & Load Distribution
**Level**: Reference
**Difficulty**: N/A
**Tags**: #pattern-summary #api-design #rate-limiting #load-balancing #auth #multi-tenancy
**Estimated Time**: 1 hour (review + reflection)
**Related Chapters**: Ch. 28–47
**Exercise Type**: Pattern Summary
---

> No deliverables. This is a reflection chapter.

---

### How This Arrived

**Slack DM — Marcus Webb → You**

**Marcus Webb**
You've now built rate limiting twice (CloudStack, CivicOS), auth twice (MeridianHealth,
CloudStack), and seen multi-tenancy up close. Plus the search systems at PulseCommerce
and NeuroLearn gave you a view of how APIs change when the data structure changes.

I want to talk about the API layer specifically — because it's the boundary between
your system and the outside world, and it's where most of the interesting design
decisions get made.

Also: you've now worked at companies where rate limiting, auth, and multi-tenancy
were afterthoughts. And you've seen what "afterthought" costs.

[Attachment: api-rate-limiting-load-patterns.md]

---

### Patterns Covered: Ch. 28–47

---

#### Pattern 23: Rate Limiting — Threat Model First

**The core insight**: Rate limiting is not primarily a performance control.
It's a security control. The threat model determines the algorithm.

**Threat Model 1: Runaway client (accidental)**
- A client has a bug that sends too many requests
- Defense: per-client token bucket. Simple, effective.
- Used at: CloudStack (Ch. 29)

**Threat Model 2: Coordinated distributed attack (intentional)**
- 847 IPs each sending 17 requests/hour. No individual IP is over limit.
- Defense: behavioral fingerprinting, per-ASN limits, per-subnet limits,
  response normalization (remove oracle)
- Used at: CivicOS (Ch. 40)

**Threat Model 3: Abuse of business logic**
- Not rate of requests, but pattern of requests. Flash sale inventory abuse,
  credential stuffing, enumeration.
- Defense: anomaly detection, CAPTCHA, session fingerprinting
- Used at: PulseCommerce flash sale (Ch. 37), CivicOS enumeration (Ch. 40)

**The lesson**: "Rate limiting" is not one pattern. It's a family of patterns.
Start with the threat model. The algorithm follows.

---

#### Pattern 24: The Three Rate Limiting Algorithms

**Token Bucket**:
- Bucket with N tokens; refills at R tokens/second; each request consumes 1 token
- Allows bursts (bucket can be full); prevents sustained overload
- Best for: API gateway rate limiting; allows legitimate burst behavior
- Redis implementation: atomic Lua script; check + decrement in one operation

**Sliding Window Log**:
- Store timestamp of every request in last N seconds; count is the rate
- Accurate; no burst problem at window boundary
- Expensive: stores N entries per client per window; Redis memory intensive
- Best for: high-accuracy, low-volume rate limiting (financial API, fraud signals)

**Fixed Window Counter**:
- Simple counter per time window (e.g., 100 requests per minute)
- "Double burst" problem: 100 requests at 0:59, 100 at 1:01 = 200 in 2 seconds
- Best for: simple quota systems where precision isn't critical

**Leaky Bucket**:
- Requests enter a queue (bucket); processed at a fixed rate
- Smooths bursts; requests can queue and wait
- Best for: protecting a downstream service with fixed capacity
- Not great for user-facing APIs (users wait in invisible queue)

---

#### Pattern 25: Multi-Tenant API Design

**The core requirement**: tenant A must not know about tenant B. Not their data,
not their existence, not their infrastructure placement.

**Four isolation layers**:
1. **Data isolation**: RLS, separate schemas, separate DBs per tier
2. **Performance isolation**: ResourceQuotas (K8s), connection pool limits per tenant,
   per-tenant rate limits
3. **Failure isolation**: blast radius bounded by isolation model (separate DB for Enterprise)
4. **Information isolation**: no internal IDs in API responses, no shared error messages,
   no infrastructure hints in headers

**Tenant context propagation**:
- Tenant context (tenant_id) must be passed through every layer of the system
- Typically: extracted from JWT at gateway → injected into request context →
  used in every DB query as a filter or routing key
- Never: passed as a user-controlled parameter (users can't specify their own tenant_id)

**Where it appeared**:
- CloudStack: 3,200 tenants, full isolation matrix (Ch. 28)
- CivicOS: 23 state governments, each with different compliance requirements (Ch. 39)

---

#### Pattern 26: Authentication vs Authorization — and Where They Live

**Authentication**: Who is this? (Identity)
**Authorization**: What can they do? (Permissions)

They are different problems. They often get conflated in implementation.

**Authentication patterns by context**:
- Human users: OIDC/SAML federation to enterprise IdP (Ch. 30, Ch. 41)
- Machine-to-machine: OAuth2 client credentials, short-lived tokens (Ch. 30)
- API consumers: API keys with rate limiting (Ch. 29)

**Authorization models**:
- RBAC: "admin role can do everything; user role can read only"
  - Simple; works well for small role sets
  - Breaks down with many fine-grained permissions or complex conditions
- ABAC: "user X can do Y if attribute Z is true"
  - More flexible; handles complex conditions (tenant isolation, time-based access)
  - More complex to implement and debug
- ReBAC: "user X can do Y to resource R if there's a relationship between X and R"
  - Zanzibar model; Google, GitHub use this
  - Most flexible; most complex

**Key lesson**: Don't choose the most complex model. Choose the minimum model
that satisfies the requirements. RBAC with tenant scoping is sufficient for 80%
of enterprise SaaS.

---

#### Pattern 27: API Response Normalization (Security Pattern)

**What it is**: Return the same HTTP status code and response shape regardless of
whether a resource exists, to prevent information leakage.

**Example** (Ch. 40 — CivicOS):
- Bad: `/api/citizens/search?ssn=123-45-6789` returns 200 if exists, 404 if not
- This is an oracle — an attacker can enumerate which SSNs are in the system
- Good: return 200 in both cases (null payload if not found)
- Alternative: require authentication before any lookup (not-found becomes moot)

**Where it applies**:
- Any endpoint where the existence of a record is sensitive information
- User account existence (prevents account enumeration)
- Payment method existence (prevents card probing)
- Citizen data existence (prevents SSN enumeration)

**The tradeoff**: Normalization makes debugging harder (legitimate 404s are masked).
Mitigation: verbose error responses behind authentication, or in server-side logs.

---

#### Pattern 28: Search as a System Design Problem

**Search is not a database query**. It's a separate system with its own:
- Data store (inverted index, not relational)
- Sync pipeline (CDC from source of truth to search index)
- Relevance model (scoring function, not just match/no-match)
- Query language (Elasticsearch DSL, not SQL)
- Scaling model (horizontal shard scaling, not vertical)

**Sync latency SLA determines the architecture**:
- < 60s: Debezium CDC → Kafka → Elasticsearch (Ch. 33)
- < 15s: Application writes directly to Elasticsearch after DB write (dual write)
- Real-time: Elasticsearch as primary store (no sync, but loses transactional guarantees)

**User context in search**:
- Anonymous search: no user context; pure text relevance
- Personalized search: user context (institution, language, level, history) affects ranking
- The challenge: injecting user context at query time without adding latency

**Where it appeared**:
- PulseCommerce (Ch. 33): 150M products, relevance + merchant scoping
- NeuroLearn (Ch. 45): 48M academic documents, institution + language + level scoping

---

### Cross-Industry Observations

API design patterns are infrastructure patterns — they apply regardless of domain.
But the threat model and compliance requirements differ dramatically:

| Pattern | Commercial SaaS | Government | Education |
|---------|----------------|------------|-----------|
| Rate limiting | Prevent cost overrun | Prevent enumeration attacks | Prevent grade manipulation |
| Auth | Tenant isolation | FedRAMP/FISMA compliance | FERPA compliance |
| Multi-tenancy | Performance isolation | Data sovereignty | Institution isolation |
| Search | Relevance + conversion | Not typically needed | Personalized + level-aware |

---

### Reflection Questions

No answers provided.

1. You've now built rate limiting for two very different threat models: CloudStack
   (runaway client, accidental) and CivicOS (coordinated attack, intentional).
   If you were designing a rate limiting system for a new API gateway today,
   how would you architect it to handle both threat models with the same system?
   Is it possible to have a single rate limiting architecture that's correct for both?

2. Authentication was "simple" at MeridianHealth (one company, known users) and
   "complex" at CloudStack (3,200 tenants, each with their own IdP). At what point
   does the per-tenant IdP federation pattern become untenable? If you had 50,000
   tenants instead of 3,200, what would break first?

3. Multi-tenancy isolation exists on a spectrum: from "same everything, just filtered by tenant_id"
   to "completely separate infrastructure per tenant." Where on that spectrum should
   a new B2B SaaS start? And what's the trigger event (a customer ask? a compliance
   requirement? an incident?) that causes you to move further toward isolation?
