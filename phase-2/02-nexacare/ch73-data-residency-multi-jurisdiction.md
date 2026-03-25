---
**Title**: NexaCare — Chapter 73: Data Residency Across Multiple Jurisdictions
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #data-residency #gdpr #compliance #multi-region #data-sovereignty #appi #pipeda
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 70, Ch. 111, Ch. 143
**Exercise Type**: System Design / Design Review Ambush
---

### Story Context

**Architecture Review — NexaCare Principal Architects**
*Tuesday 10:00 AM — Conference Room A*
*Present: You, Fatima Al-Rashid, Dr. James Osei (Principal Architect), Dr. Mia Torres (CCO)*

The learner has prepared a "simple" multi-region architecture: a primary region in us-east-1, read replicas in eu-west-1 (Frankfurt) and ap-southeast-1 (Singapore), with a global Kafka cluster handling event replication. Clean, familiar, well-understood.

The first two slides go smoothly. Then Dr. Osei raises his hand.

**Dr. Osei**: Before you go further — slide 2. You have Japanese patient data routing through Singapore and replicating to your US primary. Where does Japanese patient data *live*?

**You**: It... replicates to all three regions. For redundancy.

**Dr. Osei**: Under Japan's APPI — the Act on the Protection of Personal Information — health data can only be transferred to a third country if that country meets Japan's adequacy standards. Does the United States meet Japan's adequacy standards under APPI?

*(Silence.)*

**Dr. Torres**: The United States does not have a federal adequacy designation under APPI as of the current review period.

**Dr. Osei**: So your design currently moves Japanese patient health data to a country that is not APPI-adequate. That's a potential regulatory violation for every Japanese patient enrolled in a NexaCare-managed trial.

**Fatima**: *(to you, quietly)* This is why we do architecture reviews before we ship.

**Dr. Osei**: How many countries have you checked? GDPR has adequacy decisions. APPI has its own. Canada's PIPEDA has rules for health data. China's PIPL — Personal Information Protection Law — essentially prohibits health data from leaving China at all without explicit government approval. Brazil's LGPD mirrors GDPR but has differences in research exemptions. Your design needs to answer all of these.

**You**: I... only checked GDPR.

**Dr. Osei**: That's 14 countries you missed. This meeting is now a redesign session. We have 90 minutes. Let's start over.

---

**Break — Hallway Conversation**

**You**: *(to Fatima)* How does Dr. Osei know all of this?

**Fatima**: James spent 10 years as a data compliance architect at Roche before joining NexaCare. He's held in 6 countries. This is his specialty.

**You**: I feel like an idiot.

**Fatima**: Good. That's the right feeling. Now go back in there and lead the redesign. He's been brutal because he knows you can handle it.

---

**Redesign Session — Post-Break**

Over the next 90 minutes, the group works through the data residency requirements for NexaCare's 14 priority jurisdictions. What emerges is a set of constraints the architecture must satisfy:

- EU patients: data must stay within the EU (GDPR) — Frankfurt DC ✓
- Japanese patients: data must stay in Japan (APPI, no US adequacy) — no current Japan DC
- Canadian patients: data must stay in Canada (PIPEDA health data provisions)
- Chinese patients: data cannot leave China (PIPL) — requires in-country DC or no Chinese trials
- Brazilian patients: data requires consent for cross-border transfer (LGPD) — manageable
- US patients: no federal residency requirement, but state-level laws vary (HIPAA covers health data)

**Dr. Osei**: The hardest architectural question is this: if a Japanese patient enrolls in a global trial that also has EU and US patients, and you need to run cross-trial analytics, how do you compute aggregate statistics without moving Japanese data out of Japan?

**You**: *(thinking)* You'd need to run the computation *in Japan* and only send the aggregates — not the raw data — to the analytics system.

**Dr. Osei**: That's called federated analytics. Now design the system.

---

### Problem Statement

NexaCare manages clinical trials across 180 countries. Data residency laws in 14 priority jurisdictions place strict requirements on where patient data may be stored and processed. The current architecture replicates all data to three central regions, violating APPI (Japan), PIPL (China), and potentially PIPEDA (Canada). You need to redesign the multi-region architecture to enforce data residency per jurisdiction, while still enabling cross-trial analytics (without violating residency by moving raw data).

---

### Explicit Requirements

1. Patient data must be stored and processed within the patient's country of origin where legally required
2. Cross-country data transfers must only occur where an adequacy decision or appropriate safeguard exists
3. Cross-trial analytics must be possible without violating residency (federated computation where needed)
4. The architecture must accommodate "no transfer" jurisdictions (China under PIPL)
5. Encryption key custody must be jurisdiction-specific (keys for Japanese data held in Japan)
6. The system must be auditable: for any piece of patient data, you can prove which jurisdiction it has resided in

---

### Hidden Requirements

- **Hint**: Dr. Osei asked: "Does the United States meet Japan's adequacy standards under APPI?" APPI adequacy is bilateral. There's a deeper question: if NexaCare stores Japanese patient data in Japan, but their Japanese DC is managed by AWS Tokyo — and AWS is a US company subject to US law including CLOUD Act requests — does the data still "leave Japan" legally? This is an active legal debate. How does your architecture address it?

- **Hint**: NexaCare's business model involves trial sponsors (pharmaceutical companies) accessing their trial data. If a US pharma company wants to review data from their Japanese trial patients, and Japanese law prohibits export — how does the trial sponsor access their own data? This is a product constraint buried in the compliance requirement.

---

### Constraints

- 14 priority jurisdictions with varying residency rules
- Currently 0 DCs in Japan, 0 in China, 0 in Canada
- Global Kafka cluster for event replication (cannot be used across residency boundaries)
- Cross-trial analytics run by NexaCare's data science team in the US
- Trial sponsors need read access to their own trial data regardless of where it's stored
- Budget for new DCs: can add up to 4 new AWS regions in year 1

---

### Your Task

Design the multi-jurisdiction data residency architecture for NexaCare. Include the per-jurisdiction storage strategy, the cross-border analytics approach, and the trial sponsor access model.

---

### Deliverables

- [ ] Jurisdiction matrix: For each of the 14 priority jurisdictions, specify: residency requirement, adequacy status, allowed transfer conditions, NexaCare's proposed storage location
- [ ] Mermaid diagram: Multi-region architecture with data residency boundaries enforced at infrastructure level
- [ ] Federated analytics design: How to compute cross-trial aggregate statistics without moving raw Japanese or Chinese patient data
- [ ] Trial sponsor access model: How a US pharma company accesses Japanese trial data without violating APPI
- [ ] Encryption key custody: Per-jurisdiction key management (KMS in the patient's country)
- [ ] Tradeoff analysis (minimum 3):
  - Full data localization vs adequacy-based cross-border transfer
  - Federated analytics vs anonymization-then-transfer for cross-trial research
  - AWS-hosted regional DC vs co-location for "no transfer" jurisdictions (China)
- [ ] Phase 1 roadmap: Which 4 new DC regions to add first, ranked by trial volume and residency risk
