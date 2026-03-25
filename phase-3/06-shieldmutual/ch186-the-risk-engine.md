---
**Title**: ShieldMutual — Chapter 186: The Risk Engine
**Level**: Staff
**Difficulty**: 8
**Tags**: #risk-scoring #real-time #api-design #caching #external-data #compliance #insurance #latency
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 29 (CloudStack rate limiting), Ch. 56 (LuminaryAI metering), Ch. 149 (VertexCloud IDP)
**Exercise Type**: System Design
---

### Story Context

The email arrives on your third day.

---

**From**: Gabrielle Okonkwo, CTO
**To**: Tomás Rivera, Adwoa Sarfo, You
**CC**: Patrick Hess, VP of Underwriting; Renata Szymanski, Chief Actuary
**Subject**: Re: Re: Re: NCI contract — URGENT — underwriting timeline
**Received**: Tuesday 8:47 AM

Team —

Quick update from my call with NCI (National Construction Infrastructure — the client) yesterday. They've moved up the timeline. They want preliminary quotes on all 10,000 properties by end of week.

Current state of our quoting system: manual underwriter review for commercial property, 2-4 days per complex property, 20-30 minutes for simpler ones. That's fine for our current volume. It is not fine for 10,000 properties in 48 hours.

Patrick estimates we have 6 underwriters available. At best, that's 6 × 8 hours × 3 quotes/hour = 144 quotes. We need 10,000.

I need a plan by EOD today for how we can automate the preliminary risk scoring. This is a $40M contract. It does not close if we can't demonstrate the quoting capability.

— G

---

You read it twice. Then you walk to Tomás's desk.

"The NCI contract," he says before you can speak. "I heard."

"What does the current quoting system actually do?"

Tomás swivels in his chair. He has the deliberate movements of someone who has explained complicated things many times and has decided that slowness is efficiency. "ARIA pulls data from four sources: our internal claims history, a commercial credit bureau, COPE data from the property — that's Construction, Occupancy, Protection, Exposure — and ISO industry benchmarks. An underwriter reviews all of it and assigns a manual risk score. The automation we have today runs the data pulls in parallel but still requires a human to score it."

"What are the data sources' latency profiles?"

"Claims history: internal, fast. Credit bureau: 200–400ms. COPE pull: it's actually a third-party data aggregator, Verisk, 800ms–2 seconds depending on their load. ISO benchmarks: we cache those, they update quarterly."

"And for 10,000 properties?"

"We've never done it. Our current throughput is maybe 200 quotes per hour in batch mode when the underwriters aren't in the way. We have no idea what happens at 2,000 per hour."

You go back to your desk. You open a blank document.

By 11am you have a Slack thread with Patrick Hess that changes the shape of the problem.

---

**#underwriting-tech** | Tuesday 11:03 AM

**Patrick Hess**: Quick question on the NCI properties — did anyone pull the list yet? I want to see the state distribution.

**Tomás Rivera**: Pulling now. Give me 5 min.

**Tomás Rivera**: [attached: nci_properties.csv]

**Patrick Hess**: Hm. 2,847 Florida properties. That's... a lot.

**Patrick Hess**: Someone needs to flag this to legal before we quote those.

**You**: Flag what specifically?

**Patrick Hess**: Florida has restrictions on what data sources can be used in commercial property pricing. There's a 2021 regulation — section 626 something — that limits the use of certain federal flood zone data and prohibits credit-based insurance scoring for commercial lines in certain property categories.

**Patrick Hess**: We've never quoted this many Florida properties at once. Our manual underwriters know to check, but if we're automating this...

**Renata Szymanski**: @Patrick is correct. I'm pulling the Florida reg now. The short version: for commercial property in flood-prone zones, we cannot use the FEMA FIRM maps in our pricing model. We have to use our own actuarial tables or approved state sources. The federal source is explicitly prohibited.

**You**: So we have two different scoring paths. Florida properties in flood zones use different data sources than everywhere else.

**Renata Szymanski**: At minimum. There are also coastal property rules for wind exposure. And a handful of categories that require manual underwriter review regardless of automation.

**Patrick Hess**: Yeah. So this isn't just "build a fast risk scoring API." It's "build a fast risk scoring API that knows the rules of 50 different states."

---

By afternoon, you have a meeting with Gabrielle.

**1:1 Transcript — Tuesday 3:15 PM**
*Gabrielle Okonkwo, CTO | You | Conference Room B*

**Gabrielle**: "Where are we?"

**You**: "I have a design in my head. Before I describe it, I need to understand one thing: is this system going to live beyond the NCI contract?"

**Gabrielle**: "Yes. This is the foundation of our automated underwriting platform. We've been talking about it for two years. NCI just forced the timeline."

**You**: "Then we need to build it correctly, not just fast. That means accepting that we won't have all 10,000 properties quoted by Friday. I can get you 7,000. The other 3,000 — the Florida coastal and flood-zone properties — need a second pass with compliant data sources or human review."

**Gabrielle**: *[pause]* "Patrick is going to hate that answer."

**You**: "Patrick is going to hate a regulatory fine more."

**Gabrielle**: *[longer pause]* "Okay. What does 'built correctly' look like?"

---

### Problem Statement

ShieldMutual needs a real-time risk scoring pipeline for commercial property insurance quotes. The pipeline must ingest a property's metadata (address, construction type, occupancy, size, use class), call up to 12 external data sources in parallel, synthesize the results through a configurable risk scoring model, and return a preliminary risk score and recommended premium range within 2 seconds at the 99th percentile.

The system must handle state-level regulatory variation in which data sources can be used for pricing — starting with Florida's restrictions on federal flood zone data for commercial lines. The architecture must be extensible to add new state rules, new data sources, and new scoring model versions without redeploying the core pipeline.

---

### Explicit Requirements

1. Risk score API: `POST /quotes/risk-score` returns score + confidence + data source audit within 2s p99
2. Support 10,000 concurrent quote requests during batch windows (NCI contract); 200 req/min steady-state
3. 12 external data sources; each call is independently optional — partial results are acceptable with degraded confidence score
4. State-level routing rules: different data sources allowed per state/property category combination
5. Risk model versioning: ability to run model V1 and V2 simultaneously (A/B or shadow mode) for actuarial validation
6. Every quote request must produce an audit record: which data sources were called, which returned data, which were skipped (and why), which model version was used, final score
7. Data source cache: sources that update infrequently (ISO benchmarks: quarterly, COPE base data: weekly) should be cached; real-time sources (credit, claims) must not be stale > 24 hours
8. Circuit breaker per external data source: if a source is down, the pipeline continues with degraded confidence rather than failing the entire request

---

### Hidden Requirements

1. **Hint**: Re-read Patrick's message about Florida properties. He says "we've never quoted this many Florida properties at once." What does this imply about the current system's state-awareness? Ask: does the risk scoring model itself need to know which state it's operating in, or is that a routing concern handled upstream? The answer changes where the compliance logic lives in the architecture.

2. **Hint**: Renata says "for commercial property in flood-prone zones, we cannot use the FEMA FIRM maps in our pricing model." What happens if a data source that is prohibited in Florida is cached globally? If the cache doesn't carry state-context metadata, compliant routing becomes impossible. The cache key design is a compliance requirement, not just a performance decision.

3. **Hint**: Gabrielle says "we've been talking about it for two years." This system is not a prototype — it will become the foundation of the underwriting platform. What does that imply about the audit log schema? It will need to serve future regulatory examinations, not just current ones. Re-read Pattern Summary 12, Pattern 66: bi-temporal modeling for regulatory audit.

4. **Hint**: Tomás mentions that Verisk (the COPE data aggregator) has latency of "800ms–2 seconds depending on their load." That's your entire latency budget for the worst case. If Verisk is slow, you cannot meet the 2s p99 SLA. What pre-fetching or speculative execution strategy resolves this without violating data freshness requirements?

---

### Constraints

- **Scale**: 10,000 concurrent quotes during batch windows; 200 req/min steady-state
- **Latency**: < 2s p99 end-to-end (including all data source calls)
- **External data sources**: 12 sources; latency range 50ms (internal) to 2000ms (Verisk COPE)
- **Cache TTL limits**: ISO benchmarks: 90 days; COPE base data: 7 days; credit scores: 24 hours; claims history: 1 hour; real-time weather/satellite: no cache (live only)
- **Team**: 4 backend engineers, 1 data scientist (risk model), 1 compliance engineer
- **Budget**: $15K/month infrastructure cap for the risk scoring service
- **Compliance**: Florida 626 regs, NAIC model regulations, state-specific data source restrictions (starting with FL, expanding to 50 states)
- **Audit retention**: 7 years per insurance record retention requirements
- **Existing system**: ARIA monolith (Java EE); risk scoring must integrate via REST or event queue — no direct DB access to ARIA

---

### Your Task

Design the ShieldMutual Real-Time Risk Scoring Pipeline. The system must handle the 10,000-property NCI batch, the steady-state quoting volume, and the compliance routing rules — while producing a tamper-evident audit trail for every quote.

Focus particularly on: (1) how data source fan-out and partial failure are handled within the 2s latency budget; (2) how state compliance rules are modeled and applied without hardcoding per-state logic into the scoring service; (3) how the audit record is structured to support future regulatory examination.

---

### Deliverables

- [ ] Mermaid architecture diagram (end-to-end: quote request → data fan-out → scoring → audit → response)
- [ ] Data source orchestration design: timeout budgets, circuit breaker states, partial result handling
- [ ] State compliance rule model: how do you represent "Florida commercial flood-zone properties cannot use FEMA FIRM data" as a configuration rather than code?
- [ ] Cache key design: how does the cache incorporate state context to prevent compliant/non-compliant data mixing?
- [ ] Database schema: quote audit table (with column types, indexes, retention design)
- [ ] Scaling estimation:
  - 10,000 concurrent requests × 12 data source calls = 120,000 outbound HTTP calls/batch window
  - What concurrency model handles this within budget?
  - Storage: 10,000 quotes/day × 365 × 7 years = 25.5M audit records; estimate storage size
- [ ] Cost modeling: estimate monthly infrastructure cost (API compute, cache layer, audit storage, data source API fees) — must stay under $15K/month
- [ ] Tradeoff analysis (minimum 4):
  - Fan-out-all vs. required-subset data source calls
  - Cache freshness vs. compliance (some sources cannot be cached in certain contexts)
  - Synchronous vs. asynchronous audit writing (latency vs. durability)
  - Monolithic scoring model vs. per-state model shards

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
