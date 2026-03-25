---
**Title**: Crestline Energy — Chapter 111: Energy Data Sovereignty — When Regulators Disagree
**Level**: Staff
**Difficulty**: 8
**Tags**: #data-residency #gdpr #eu-energy #data-sovereignty #cross-border #anonymization #federated-analytics
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 73 (NexaCare data residency), Ch. 143 (PrismHealth sovereignty), Ch. 111 cross-reference
**Exercise Type**: System Design
---

### Story Context

**Email Chain**
**From**: Lena Krause (Compliance Officer)
**To**: [you], Elena Vasquez
**CC**: Matthias Bauer (Crestline Legal, Paris)
**Subject**: FWD: CRE Preliminary Opinion — French Energy Data Classification
**Date**: Tuesday 07:18

Forwarding for immediate attention. French regulator (CRE — Commission de Régulation de l'Énergie) has issued a preliminary opinion classifying residential energy consumption data as "strategic national infrastructure data" under Article 7 of the French Energy Code. Translated summary attached (Matthias prepared it).

Key restriction: French residential energy consumption data at individual meter level cannot be transmitted outside French territory for any purpose, including analytics, machine learning, or cloud storage with replication to non-French servers.

This affects our planned France expansion. The contract with the French utility consortium (EDF Distribution + four regional operators) is valued at €14.2M over 3 years. Signing is planned for 60 days from today.

We need a technical solution that satisfies the CRE restriction within 60 days.

— Lena

---

**Email**
**From**: Matthias Bauer (Legal, Paris)
**To**: [you], Lena Krause, Elena Vasquez
**Subject**: RE: CRE Preliminary Opinion
**Date**: Tuesday 08:45

[you] —

To be precise about the regulatory constraint: the CRE opinion distinguishes between individual meter data and aggregated grid data:

- **Individual meter data** (consumption at one address, one household): cannot leave France. Not even for processing. Not even to a French cloud region owned by a US company with French data centers, if the US company's parent can legally compel production of the data under US law (CLOUD Act concern).
- **Aggregated grid zone data** (average consumption for a zone of 10,000+ meters): not restricted. Can be used for cross-border grid management.
- **Anonymized data** (provably cannot be re-identified to an individual): CRE is undecided. We should assume restricted unless they clarify.

Your existing architecture replicates all data to a central EU warehouse in Frankfurt. That needs to change for France.

Matthias

---

**1:1 — Elena Vasquez, Wednesday 10:00**

**Elena Vasquez**: You've read Matthias's email. What are the options?

**[you]**: Three options. One: don't do France. We lose €14.2M. Two: run a completely separate France stack with no data leaving the country. We duplicate infrastructure. Three: redesign the data architecture so that computation moves to the data instead of data moving to computation.

**Elena Vasquez**: Option three sounds like the right engineering answer but also sounds like "it'll take 18 months."

**[you]**: Option three is a federated analytics model. French meters feed French data stores. Analytics queries run inside France. Only aggregated results — not raw data — leave France to join our central analytics pipeline. If we design it right, it becomes the model for every country with data localization requirements. Germany's BDSG is heading in the same direction. So is Spain's LOPD.

**Elena Vasquez**: How long?

**[you]**: For France alone, 8 weeks. For the federated model that scales to all countries: 6 months.

**Elena Vasquez**: We sign the France contract in 60 days. Can we have a workable solution for France in 60 days even if the full federated model isn't done?

**[you]**: Yes. France-in-isolation first. Federated model second. The contract signing can reference the architecture plan for the full federated model as a future commitment.

**Elena Vasquez**: Design it. Matthias needs technical specifications to include in the contract annexe by end of month.

---

### Problem Statement

Crestline Energy's expansion into France is blocked by a French regulatory opinion restricting individual meter data to French territory. The current architecture replicates all data to a central EU warehouse. A federated analytics architecture must allow French data to remain in France while still enabling cross-border grid analytics using aggregated data.

---

### Explicit Requirements

1. Individual French meter data must not leave French territory under any circumstances
2. Aggregated grid zone data (≥10,000 meters) may leave France for cross-border analytics
3. Existing analytics pipeline (Presto + Iceberg in Frankfurt) must continue working for all non-France countries without modification
4. France architecture must be operational within 60 days for contract signing
5. Solution must be extensible to other countries with future data localization requirements (Germany, Spain)
6. ML models for demand forecasting must function for French meters without requiring French data to leave France

---

### Hidden Requirements

- **Hint**: Matthias flagged the CLOUD Act concern — a US cloud provider's French region may not satisfy CRE if the parent company is under US jurisdiction. Re-read his email. What does this mean for AWS/GCP/Azure as hosting options for the French data plane?

- **Hint**: The CRE opinion is undecided on "anonymized data." Your anonymization pipeline design will be reviewed by French regulators. What technical standard constitutes "provable anonymization" for energy data (k-anonymity, differential privacy, or something else)?

---

### Constraints

- France deployment: 8M meters at contract start, growing to 12M in year 2
- Cross-border analytics: aggregation must be at grid zone level (≥10,000 meters minimum)
- 60-day deployment timeline for France-in-isolation architecture
- Legal requirement: all data processing must happen on infrastructure with no US-parent CLOUD Act exposure
- Existing Frankfurt architecture cannot be modified (20+ countries already using it)

---

### Your Task

Design the federated data architecture for Crestline Energy's France expansion and the generalized data sovereignty model.

---

### Deliverables

- [ ] Mermaid architecture diagram: French data plane (isolated) → aggregation layer → central EU analytics (Frankfurt)
- [ ] Data classification model: individual meter data vs aggregated zone data vs anonymized data — what can cross the border?
- [ ] Federated analytics design: how queries run inside France and return only aggregated results
- [ ] Anonymization pipeline: the technical approach (differential privacy vs k-anonymity) for the "undecided" category
- [ ] Generalized sovereignty model: how the France architecture extends to Germany/Spain with country-specific configuration
- [ ] ML model sovereignty design: model weights can cross borders, training data cannot — how does this work for demand forecasting?
- [ ] Tradeoff analysis (minimum 3):
  - France-in-isolation vs full federated model — risk of signing contract before full model exists
  - Differential privacy vs k-anonymity for energy data — which gives stronger regulatory protection?
  - Compute-to-data vs data-to-compute for analytics — latency, cost, and governance tradeoffs
