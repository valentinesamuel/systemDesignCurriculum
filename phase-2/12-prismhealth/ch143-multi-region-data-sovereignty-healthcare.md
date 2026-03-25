---
**Title**: PrismHealth — Chapter 143: A German Patient's Heartbeat Cannot Leave Germany
**Level**: Staff
**Difficulty**: 8
**Tags**: #data-sovereignty #gdpr #hipaa #multi-region #healthcare #cross-border #federated-ml
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 111 (Crestline data sovereignty), Ch. 133 (Axiom federated computation), Ch. 73 (NexaCare data residency)
**Exercise Type**: System Design
---

### Story Context

**Board Update Email — Dr. Ngozi Obi → Engineering Leadership**
**Subject**: Germany Expansion — €8.2M Contract
**Date**: Tuesday

Team,

I have good news and a technical challenge.

Good news: we have signed a Letter of Intent with a consortium of 40 German hospital systems — collectively, Deutsche Patientenverbund (DPV). Contract value: €8.2M annually. Patient population: approximately 700,000 monitored patients at launch, growing to 1.2M in year two.

The technical challenge: the German Data Protection Authority has issued a preliminary opinion that classifies continuous patient health monitoring data as "particularly sensitive personal data" under GDPR Article 9, with a specific requirement that this data may not be processed on infrastructure outside German territory. The DPA opinion goes further than standard GDPR data residency guidance — it specifically notes that cloud infrastructure operated by companies subject to US jurisdiction (CLOUD Act) does not satisfy the "German territory" requirement even if servers are physically located in Germany.

Our entire infrastructure runs on AWS. AWS is a US company. The DPA opinion may exclude AWS Frankfurt from being a compliant processing location.

Legal and engineering need to resolve this before the contract converts from an LOI to a signed agreement in 60 days.

— Ngozi

---

**Emergency Architecture Meeting — Wednesday 10:00**
**Attendees**: Dr. Obi, Priya Nair, [you], Kavita Rao (General Counsel), Michael Brandt (European Legal Counsel)

**Michael Brandt**: To be precise: the DPA's concern is jurisdiction, not physical location. An AWS server in Frankfurt is physically in Germany. But AWS, as a US company, can be compelled to produce data to US government agencies under the CLOUD Act. The DPA's position: this potential compelled production constitutes a data transfer outside Germany, regardless of where the data physically sits.

**Priya Nair**: What compliant options exist?

**Michael Brandt**: Two. One: use infrastructure operated by a European company with no US corporate parent. German cloud providers: IONOS (German), Hetzner (German), T-Systems (Deutsche Telekom subsidiary — German). These are not AWS. They are genuinely outside US jurisdiction. Two: host on German government cloud (Bundescloud) — available only for public sector entities. DPV is a private consortium. Option two is not available to them.

**[you]**: IONOS and Hetzner don't have the equivalent of AWS managed services — no managed Kubernetes, no managed RDS, no managed Kafka. We'd be operating raw infrastructure.

**Priya Nair**: How long to build on IONOS from scratch?

**[you]**: For a production-ready German data plane — 4 months minimum. We have 60 days.

**Dr. Ngozi Obi**: What can we deliver in 60 days?

**[you]**: In 60 days, we can deliver an architecture design and a deployment timeline commitment that satisfies the DPA preliminary opinion. We can't have a running system in 60 days — but we can sign the contract with an architecture annex showing exactly how compliance will be achieved. The DPA preliminary opinion is preliminary — the final opinion often allows reasonable timelines for compliance if the architectural plan is credible.

**Michael Brandt**: *(nodding)* That is consistent with how DPAs have handled similar situations. The question is whether your architecture plan is technically credible.

**[you]**: It will be.

---

### Problem Statement

PrismHealth's Germany expansion requires patient monitoring data to be processed exclusively on European-jurisdiction infrastructure, but the platform runs entirely on AWS, a US company subject to the CLOUD Act. A German data plane on non-US infrastructure (IONOS/Hetzner) must be designed with an ML inference capability that uses Priya's alert scoring model (trained in the US) without exporting German patient data.

---

### Explicit Requirements

1. German patient vital data must be stored and processed exclusively on infrastructure operated by entities under German/EU jurisdiction (no AWS, GCP, Azure)
2. Alert scoring ML model must function for German patients without German data leaving German infrastructure
3. German data plane must achieve the same clinical SLAs as the US/EU infrastructure: ≤ 100ms alert evaluation, ≤ 400ms device-to-alert p99
4. Cross-border data flow for the model: model weights (trained on non-German patient data) may be deployed to Germany; patient data may not leave Germany
5. Architecture design must be credible enough to satisfy DPA preliminary opinion within 60 days
6. German data plane must be operationally maintainable by PrismHealth's engineering team (not outsourced to the German cloud provider)

---

### Hidden Requirements

- **Hint**: Michael Brandt mentioned "IONOS and Hetzner" as compliant options, but also noted "they don't have managed services." Running production healthcare infrastructure on bare VMs (not managed Kubernetes, not managed databases) is a significant operational burden. Your architecture design must account for the "build our own managed layer" cost — Kubernetes cluster management, database operations, Kafka administration. What is the correct managed-service strategy for a 60-day timeline on unfamiliar infrastructure?

- **Hint**: The alert scoring model was trained on US patient data. The DPA opinion restricts processing of German patient data outside Germany. But what about model *improvement*? If PrismHealth wants to improve the model using German patient data in future (with consent), that data cannot be sent to the US for training. Your design must include a federated learning pathway that allows model improvement using German data without that data leaving Germany.

---

### Constraints

- 700,000 German patients at launch (1.2M in year 2)
- Non-US infrastructure: IONOS or Hetzner (evaluated, Hetzner preferred for Kubernetes-native support via Hetzner Cloud)
- Timeline: architecture design in 60 days; running system in 4 months
- ML model: 2,048-feature input, runs on CPU (no GPU inference required for this model)
- Cross-border latency: German data plane → US platform for non-PHI operational data (monitoring, alerting, billing) = acceptable; German patient PHI → US = not acceptable
- GDPR Article 9: heightened requirements for health data processing — DPA approval of processing activities required

---

### Your Task

Design PrismHealth's German data sovereignty architecture for the DPV deployment.

---

### Deliverables

- [ ] Architecture diagram: German data plane (Hetzner) isolated from US/EU platform (AWS) with cross-border data flow classification
- [ ] Data classification: German PHI (stays in Germany) vs operational data (allowed cross-border) vs model artifacts (allowed cross-border)
- [ ] German infrastructure stack: Kubernetes (self-managed on Hetzner), database (PostgreSQL on Hetzner), MQTT brokers (Hetzner), Kafka (self-managed)
- [ ] ML model deployment: model weights as artifacts deployed to German data plane, inference runs in-region, no patient data egress
- [ ] Federated learning pathway: how future model improvements can incorporate German patient data without exporting it
- [ ] DPA compliance documentation: the architecture annex for the 60-day contract commitment
- [ ] Operational burden analysis: what PrismHealth's SRE team must own for German infrastructure that AWS would normally manage
- [ ] Tradeoff analysis (minimum 3):
  - AWS Frankfurt (physically in Germany, legally US) vs Hetzner (physically and legally German) — compliance certainty vs operational burden
  - Federated ML vs local-only model for German patients — model accuracy vs data sovereignty
  - Single-region German data plane (simpler) vs multi-AZ German data plane (higher availability) — 60-day timeline vs reliability
