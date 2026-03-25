---
**Title**: Axiom Labs — Chapter 133: Genomic Data Is Personal — Differential Privacy and Federated Computation
**Level**: Staff
**Difficulty**: 9
**Tags**: #differential-privacy #privacy #genomics #federated-computation #data-minimization #gdpr #hipaa
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 70 (NexaCare GDPR), Ch. 111 (Crestline data residency), Ch. 143 (PrismHealth sovereignty)
**Exercise Type**: System Design
---

### Story Context

**Email**
**From**: Prof. Aisha Okonkwo (University of Manchester, Department of Immunogenetics)
**To**: Dr. Priya Sharma
**Subject**: Consortium Study — Data Sharing Blocker
**CC**: [you]

Dr. Sharma,

I am writing on behalf of the MedGen Consortium — a research group of 12 universities studying a rare autoimmune condition affecting approximately 1 in 50,000 individuals of West African descent.

To achieve statistical power for our correlation analysis, we require a cohort of 500,000 patient records with genetic variant data and phenotypic information. No single institution in our consortium has more than 60,000 records. A cross-institution analysis is essential.

The problem: each institution's IRB (Institutional Review Board) prohibits sharing patient-level genetic data outside their institution. Cross-institution data sharing requires individual IRB approvals that take a minimum of 18 months. The Wellcome Trust funding for our study expires in 14 months.

We need access to aggregate genetic correlations across all 500,000 records without any individual-level data leaving each institution. We understand this may sound impossible. A colleague at MIT told us Axiom Labs is working on federated computation capabilities.

If you can enable this research, we are prepared to become an anchor customer for Axiom's enterprise tier.

Yours sincerely,
Prof. Aisha Okonkwo

---

**Design session — Thursday 14:00**
**Attendees**: [you], Dr. Priya Sharma, Kofi Mensah, Marcus Thompson

**Dr. Priya Sharma**: Prof. Okonkwo's consortium is not unique. I've been getting similar requests for 18 months. We can't help any of them because we don't have a federated computation capability. This is a €2M revenue opportunity in the next 12 months if we build it correctly.

**Marcus Thompson**: Before we discuss the product opportunity — I want to understand the legal architecture. If Axiom's platform enables federated computation on patient genomic data across 12 institutions in 6 countries, who is the data controller? Who is the processor? What happens when a compute node fails mid-analysis and we have to recover — does recovery involve touching patient data?

**[you]**: Those are the right questions. Let me sketch the architecture first, then Marcus and I can work through the legal mapping.

You draw on the whiteboard:

**Current model (impossible)**: Patient data from University A + University B → Axiom's central platform → analysis → result

**Federated model**: Analysis algorithm → University A's data plane (result A) + University B's data plane (result B) → Axiom aggregates results A + B → final correlation

**[you]**: In the federated model, patient data never leaves each university's infrastructure. Axiom's platform sends the analysis code to the data. The result — a statistical summary — comes back to Axiom. Individual patient records never leave.

**Marcus Thompson**: But the result itself could leak patient data. If a study has 3 patients with a rare variant and the result shows "3 patients at University of Manchester have variant X" — I've effectively identified those patients.

**[you]**: That's where differential privacy comes in. Before any result leaves a university's data plane, we add mathematically calibrated noise. The noise is small enough to preserve statistical utility but large enough to prevent individual identification.

**Kofi Mensah**: How do you calibrate "small enough" and "large enough"?

**[you]**: That's the ε parameter in ε-differential privacy. The tradeoff between privacy protection and statistical accuracy.

---

### Problem Statement

Axiom Labs needs to enable cross-institution genomic research studies where patient data cannot leave individual institutions (IRB restrictions). A federated computation model — where analysis code travels to data instead of data traveling to analysis — combined with differential privacy on result aggregation, can enable this research without violating data residency requirements.

---

### Explicit Requirements

1. Patient-level genetic data must never leave the originating institution's infrastructure
2. Analysis results returned to Axiom must be differentially private (formally provable, not just "anonymized")
3. Support cohort studies across 2-50 institutions simultaneously
4. Analysis types: allele frequency calculation, GWAS (genome-wide association study), phenotype-genotype correlation
5. Results must be statistically useful for the stated research purpose (precision loss from noise must be quantified)
6. Each institution must be able to audit what computation ran against their data

---

### Hidden Requirements

- **Hint**: Re-read Marcus Thompson's failure mode question: "What happens when a compute node fails mid-analysis and we have to recover?" In a federated model, Axiom's analysis code runs inside each university's infrastructure. If a node fails and Axiom needs to debug, does that require access to patient data? Your architecture must define exactly what Axiom can and cannot access during failure recovery.

- **Hint**: The differential privacy ε parameter creates a "privacy budget." Running 10 queries against the same dataset consumes 10x the budget. If a researcher runs the MedGen consortium study 50 times (iterating on their analysis), the privacy budget for those 500,000 patients is consumed. Your system must track and enforce per-dataset privacy budgets.

---

### Constraints

- 12 institutions, 6 countries (Germany, UK, Netherlands, Sweden, France, Canada)
- Each institution: 20,000–80,000 patient records
- Study duration: 14 months (Wellcome Trust funding window)
- Differential privacy ε target: 0.1–1.0 (standard for medical research; lower is stronger privacy)
- Statistical precision: correlation coefficients must be accurate to ±0.02 for GWAS utility
- Compute: analysis jobs run for 15 minutes–8 hours depending on cohort size
- IRB compliance: audit log of every computation against each institution's data required

---

### Your Task

Design Axiom Labs' federated computation and differential privacy platform for cross-institution genomic research.

---

### Deliverables

- [ ] Architecture diagram: federated computation model (code-to-data, not data-to-code)
- [ ] Differential privacy mechanism: Laplace mechanism for frequency queries, Gaussian mechanism for continuous statistics — with parameter derivation for the genomic use case
- [ ] Privacy budget tracking: per-dataset budget accounting, researcher-facing budget consumption display, enforcement when budget is exhausted
- [ ] Failure recovery design: what Axiom can access during compute failure without touching patient data
- [ ] IRB audit log: schema and retention — what each institution can audit about computations against their data
- [ ] Statistical utility analysis: for ε=0.1 on the MedGen 500,000-record cohort, what is the expected noise in allele frequency calculations?
- [ ] Secure aggregation: how individual institution results are combined at Axiom without revealing which institution contributed which result
- [ ] Tradeoff analysis (minimum 3):
  - Federated computation vs trusted execution environments (TEE/SGX) for data privacy
  - Laplace mechanism vs Gaussian mechanism for genomic correlation studies — precision vs privacy tradeoff
  - Per-query privacy budget vs per-study privacy budget — researcher usability vs privacy protection
