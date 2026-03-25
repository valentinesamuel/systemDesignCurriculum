---
**Title**: HarvestAI — Chapter 226: The Three-Meter Problem
**Level**: Staff
**Difficulty**: 8
**Tags**: #carbon-credits #geospatial #compliance #integration #article6
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 165 (CarbonLedger carbon credit registry), Ch. 223 (HarvestAI geospatial at scale), Ch. 209 (DeepOcean data sovereignty)
**Exercise Type**: System Design / System Evolution
---

### Story Context

**Direct Message — Carlos Mendes → You, Thursday 8:44 AM**

---

**Carlos Mendes** [8:44 AM]
hey — got a message from a farm operator I need to loop you in on
can you read this and tell me if what I think is happening is actually happening

**Carlos Mendes** [8:44 AM]
*(forwarded message, translated from Portuguese)*

> From: Henrique Salgado, Fazenda Boa Esperança, Mato Grosso do Sul
>
> "Hello Carlos, I planted cover crops (brachiaria) on Field 7A and Field 7B last October and November, as recommended by the HarvestAI carbon protocol. Your platform confirmed satellite verification of the cover crops in January. Your app says my estimated sequestration for those fields is 142 tonnes CO₂e. I was expecting to see carbon credits registered on the CarbonLedger system but it has been three weeks since the verification completed and nothing has appeared. My buyer in the Netherlands is asking me for the credit serial numbers. When will I see my credits?"

**Carlos Mendes** [8:45 AM]
I looked at our CarbonLedger submission logs
we submitted his verification data two weeks ago
CarbonLedger's API returned HTTP 200
but the credits never appeared in their registry
I called their support line and they said the submission was rejected at the validation layer
reason: "boundary mismatch — submitted field polygon does not match registered field boundary within tolerance"
the tolerance is 1 meter
our submission was off by... they wouldn't say exactly but they said "more than acceptable tolerance"

**You** [8:52 AM]
what coordinate reference system are we submitting in?

**Carlos Mendes** [8:54 AM]
...oh no
we're submitting in WGS84
HarvestAI's entire geospatial stack is WGS84
I just checked the CarbonLedger API docs
they require SIRGAS 2000 for Brazilian properties
it's the Brazilian geodetic datum
IBGE mandated it for all land registrations in Brazil after 2005
our farm boundary data for Brazil was imported from INCRA — the Brazilian land registry
INCRA publishes in SIRGAS 2000
we stored it in WGS84
there's a coordinate transformation involved and... I don't know if we did it correctly when we imported the data two years ago

**You** [8:58 AM]
SIRGAS 2000 and WGS84 are close but not identical
the difference is usually sub-meter for most of South America
but in some areas of Brazil it can be up to 3 meters depending on the local realization
for a carbon credit boundary calculation, 3 meters off on a field polygon edge means you're attributing sequestration to land that might not be yours

**Carlos Mendes** [9:01 AM]
wait
3 meters off on a field polygon EDGE
across the PERIMETER of a field
how much area does that affect?

**You** [9:03 AM]
depends on field geometry. For a typical rectangular field: (perimeter × 3 meters) / 2 ≈ strip of uncertain land along every edge.
For a 50-hectare field with 1400m perimeter: that's about 2100 square meters = 0.21 hectares of uncertain attribution.
At typical soybean carbon sequestration rates (0.5 tCO₂e/hectare for cover crops), that's about 0.1 tonnes CO₂e per 50ha field.
Sounds small. But multiply by the number of fields and the number of years and the credit price ($35–45/tonne).
And more importantly: CarbonLedger's 1-meter tolerance rejects any boundary that doesn't match within 1 meter.
3-meter error = every Brazilian field boundary we've ever submitted is potentially wrong.

**Carlos Mendes** [9:06 AM]
how many Brazilian fields do we have in the system?

**You** [9:07 AM]
I'm looking. 4,200 Brazilian farms. Average 8 fields each. That's roughly 33,600 field polygons.
And if the coordinate reference system conversion was wrong in our import pipeline, every one of those polygons is potentially mis-stored.

**Carlos Mendes** [9:08 AM]
oh god
Priya is going to have a bad morning

---

**#eng-carbon — Slack, Thursday 10:15 AM**

**Priya Sundaram** [10:15 AM]
I've read the thread. I've called CarbonLedger. Here's what they told me:
1. All Brazilian credit submissions must use SIRGAS 2000 datum
2. Field boundaries must match within 1 meter of the INCRA-registered boundary
3. Article 6.2 of the Paris Agreement (which governs cross-border credit transfers from Brazil to EU buyers) requires a "corresponding adjustment" — both Brazil and the EU must record the credit transfer in their national registries. CarbonLedger handles this, but they need us to provide the correct boundary data first.
4. Any submission that was previously rejected can be resubmitted once corrected, but if a credit is issued under incorrect boundary data and later found wrong, it gets cancelled and the buyer gets burned. We need to fix this BEFORE any credits are issued, not after.

**Priya Sundaram** [10:16 AM]
so the problem is:
- Our import pipeline didn't handle the SIRGAS 2000 → WGS84 conversion correctly, or didn't record which datum the source data was in
- 33,600 Brazilian field boundaries may be incorrect by up to 3 meters
- We need to: (a) determine the actual error magnitude per field, (b) decide whether to re-import from INCRA, (c) fix the submission pipeline, (d) fix the verification pipeline
- And we need to do this without breaking the existing HarvestAI platform which uses WGS84 internally

**Priya Sundaram** [10:18 AM]
this is a data integrity incident with compliance implications
treat it as P1 until we know the scope

---

**Direct Message — Dr. Aisha Kamara → You, Thursday 11:00 AM**

**Dr. Aisha Kamara** [11:00 AM]
the carbon credit calculation pipeline is also affected beyond just the boundary issue
our satellite verification uses Sentinel-2 imagery
Sentinel-2 imagery is delivered in UTM (Universal Transverse Mercator) projection referenced to WGS84
the NDVI analysis we run on that imagery uses pixel coordinates
when we convert back to field boundaries for the "this pixel is inside this field" calculation, we're assuming WGS84
if the field boundary is actually SIRGAS 2000 and we haven't transformed it, the pixel-to-field assignment could attribute biomass measurements to the wrong field
in dense agricultural areas where fields are adjacent with no buffer strip, a 3-meter error could cause us to attribute satellite pixels from Field 7B to Field 7A
which means both fields' sequestration estimates are wrong
the error isn't just in the boundary polygon — it propagates through the entire carbon calculation

---

### Problem Statement

HarvestAI's carbon credit integration with CarbonLedger has surfaced a systemic data integrity issue: 33,600 Brazilian field polygons were imported using an incorrect geodetic datum transformation. The Brazilian national geodetic datum (SIRGAS 2000) and the global standard (WGS84) diverge by up to 3 meters across Brazil, which is within acceptable tolerance for most agricultural use cases but catastrophically outside CarbonLedger's 1-meter boundary matching requirement for carbon credit issuance. Every Brazilian farm's carbon credit submission is currently invalid.

The problem compounds: the satellite verification pipeline for carbon sequestration calculations uses the field polygons to assign satellite pixels to farms, meaning the biomass measurements used to calculate sequestration totals are also potentially wrong. And the compliance layer involves Article 6.2 of the Paris Agreement, which governs cross-border credit transfers from Brazilian farms to European buyers — a regulatory framework that requires matching geodetic precision between the Brazilian and EU national registries.

The system must be redesigned to: store geodetic datum provenance alongside all field polygons, support datum-aware coordinate transformations at query time, produce CarbonLedger-compliant SIRGAS 2000 submissions for Brazilian fields, and retroactively audit and correct existing submissions without cancelling already-issued credits.

---

### Explicit Requirements

1. The geospatial data model must record the source geodetic datum for every field polygon (WGS84, SIRGAS 2000, UTM zone, etc.) and the transformation applied during import
2. The carbon credit submission pipeline must apply the correct datum transformation at submission time, outputting the CRS required by CarbonLedger for each country (SIRGAS 2000 for Brazil, ETRS89 for EU, WGS84 for North America)
3. The satellite verification pipeline must perform pixel-to-field assignment using the same CRS as the imagery source (UTM/WGS84 for Sentinel-2), then convert results to the submission CRS — intermediate calculations must never mix coordinate systems
4. An audit pipeline must be built to calculate the error magnitude for all 33,600 existing Brazilian field polygons, compare them against INCRA source data, and identify which polygons exceed the 1-meter CarbonLedger tolerance
5. The carbon credit submission pipeline must implement idempotency: if a submission to CarbonLedger fails, it must be retryable without creating duplicate credit issuance requests
6. Article 6.2 corresponding adjustments must be tracked: when a Brazilian credit is transferred to an EU buyer, the system must record the transfer in a way that supports both Brazil's and the EU's national registry requirements
7. The system must handle partial correction: some polygons may be correctable from INCRA re-import; others may require manual survey correction — the pipeline must support a mixed state

---

### Hidden Requirements

1. **Hint: re-read Carlos's original question about Henrique's field.** He says Field 7A and Field 7B, cover crops planted in October and November. The carbon credit calculation depends on which carbon protocol was used. HarvestAI's "carbon protocol" referenced in Henrique's message is a specific methodology registered with CarbonLedger (Verra VM0042 or equivalent). The protocol specifies not just how to calculate sequestration but also how to define field boundaries — there are rules about buffer zones and adjacent field treatment. A 3-meter boundary error in a dense agricultural region may violate the protocol's adjacency rules, which could invalidate sequestration claims for multiple adjacent fields simultaneously, not just the field with the error.

2. **Hint: re-read Dr. Aisha's message about Sentinel-2 imagery.** She says "in dense agricultural areas where fields are adjacent with no buffer strip, a 3-meter error could cause us to attribute satellite pixels from Field 7B to Field 7A." This is not just a data correction problem — it is a double-counting problem. If Henrique's Field 7B pixels are being attributed to Field 7A, then Field 7A's sequestration is overstated and Field 7B's is understated. But both fields may have already had their credits calculated. The fix is not just to correct the boundaries — it is to recalculate sequestration for every affected field pair, determine which ones have net-positive vs net-negative delta, and handle the case where a field's corrected sequestration is lower than what was promised to a buyer.

3. **Hint: re-read Priya's line: "if a credit is issued under incorrect boundary data and later found wrong, it gets cancelled and the buyer gets burned."** This implies a priority ordering: fix the problem before any credits are issued for the affected fields. But some fields may already have credits issued (the CarbonLedger API returned HTTP 200 for other farms before this problem was discovered). The audit pipeline must distinguish: (a) fields with no credits issued yet (safe to correct), (b) fields with credits issued but not yet transferred to buyers (correctable with credit cancellation and re-issuance), (c) fields with credits issued and transferred to buyers (dangerous — cancellation affects third parties). How does the correction pipeline handle class (c)?

4. **Hint: re-read Carlos's datum question and your response about the 3-meter error.** You note the error is "up to 3 meters" — but it is not uniform. The actual SIRGAS 2000 vs WGS84 divergence varies by geographic location within Brazil. A field in Mato Grosso do Sul may have a 2.8-meter error; a field in Rio Grande do Sul may have only 0.4-meter error. The audit pipeline cannot assume a uniform correction — it must compute the actual transformation for each field's centroid coordinate and compare against the INCRA-registered boundary. This implies the audit pipeline needs access to the official SIRGAS 2000 → WGS84 transformation grid (IBGE's MAPGEO tool or a PROJ library grid shift file). What does it mean to run this transformation at scale for 33,600 polygons?

---

### Constraints

- **Brazilian field polygons**: 33,600 potentially affected
- **Total Brazilian farm operators**: 4,200
- **Farms with credits in progress** (submitted, not yet issued): ~800 farms, ~6,400 fields
- **Farms with credits already issued** (pre-incident): ~120 farms, ~960 fields — high risk
- **Farms with credits issued AND transferred to EU buyers**: estimated 40 farms, ~320 fields — critical risk
- **CarbonLedger boundary tolerance**: 1 meter
- **SIRGAS 2000 / WGS84 divergence in Brazil**: 0.1m to 3.1m depending on location (not uniform)
- **Sentinel-2 pixel resolution**: 10m × 10m (for vegetation bands); 20m × 20m (for red-edge)
- **Carbon credit price**: $35–45/tonne CO₂e (market rate)
- **Average sequestration per Brazilian farm per year**: ~85 tCO₂e (range: 12–840 depending on farm size and practices)
- **CarbonLedger API rate limit**: 200 requests/minute for submission endpoint; 60 requests/minute for verification status endpoint
- **Article 6.2 corresponding adjustment deadline**: must be recorded in both national registries within 90 days of credit transfer
- **PROJ library**: available in the HarvestAI stack (PostGIS uses it internally); IBGE grid shift file (MAPGEO2015.gsb) can be loaded into PROJ
- **Team available for remediation**: Carlos (primary), Raj (infrastructure support), you (design and oversight)
- **CarbonLedger re-submission SLA**: corrected submissions must be made within 30 days of the original rejection to preserve the filing date for Article 6.2 purposes

---

### Your Task

Design the carbon credit integration pipeline with full geodetic datum awareness, and design the remediation strategy for the existing 33,600 affected field polygons. The design must address: the data model changes (datum provenance storage), the transformation architecture (when and how CRS conversions happen), the audit pipeline (error magnitude per field, risk classification), the re-submission strategy (idempotent, rate-limited, priority-ordered by risk class), and the Article 6.2 compliance tracking layer.

The remediation plan must explicitly address the three risk classes: (a) no credits issued yet, (b) credits issued but not transferred, (c) credits issued and transferred to EU buyers. For class (c), you must propose a resolution path that does not burn the buyers.

---

### Deliverables

- [ ] **Mermaid architecture diagram**: full carbon credit pipeline from satellite verification → sequestration calculation → datum transformation layer → CarbonLedger submission → Article 6.2 registry; include the audit pipeline as a parallel path
- [ ] **Database schema**:
  - Modified `fields` table: add `source_datum`, `transformation_applied`, `last_datum_verified_at`, `incra_polygon_hash` columns
  - New `carbon_submissions` table: idempotency keys, submission status, CarbonLedger response, datum version at submission time
  - New `credit_audit_log` table: risk class, error magnitude, correction status, buyer notification status
  - `article6_transfers` table: bilateral registry entries for Brazil → EU transfers
- [ ] **Scaling estimation** (show math):
  - Audit pipeline throughput: 33,600 polygons × PROJ transformation time → estimated wall-clock time to audit all Brazilian fields
  - Re-submission rate: 33,600 re-submissions ÷ 200 requests/minute = time to complete full re-submission within rate limits
  - Sequestration recalculation: for ~6,400 high-risk fields, what is the compute cost of re-running the satellite pixel attribution pipeline?
- [ ] **Tradeoff analysis** (minimum 3):
  - Store coordinates in source CRS vs always transform to WGS84 at storage time vs store both: data model complexity vs query correctness vs storage overhead
  - Batch datum transformation at import time vs on-demand transformation at query/submission time: correctness risk vs performance vs pipeline complexity
  - Risk class (c) resolution options: credit cancellation with buyer compensation vs correction-in-place with tolerance negotiation with CarbonLedger vs voluntary disclosure without cancellation
- [ ] **Remediation runbook**: ordered steps for the 30-day correction sprint, including: audit pipeline execution, risk class triage, INCRA data re-import, polygon correction, re-submission, and buyer notification for class (c) affected credits
- [ ] **Idempotency design**: describe the idempotency key scheme for carbon credit submissions to CarbonLedger, ensuring that network retries or process restarts never create duplicate credit issuance requests

### Diagram Format
All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
