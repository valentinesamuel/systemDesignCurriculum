# Axiom Labs — Company Introduction

**Industry**: Biotech / Genomics Data Platform
**Your Role**: Principal Architect
**Arc**: Chapters 132–137

---

Axiom Labs is the infrastructure layer for academic genomics research. 200 universities across 14 countries store, compute on, and share genomic sequence data through Axiom's platform. When a researcher at MIT wants to compare a newly discovered cancer variant against every known sequence in the human genome reference library, they query Axiom. When a consortium of European universities wants to find genetic correlations across 500,000 patient records without sharing the underlying patient data, they use Axiom's federated computation layer.

The scale: 12 petabytes of genomic sequence data. 500TB added monthly. 4,000 active researchers running queries at any given time.

CTO **Dr. Priya Sharma** is a computational biologist who taught herself distributed systems from academic papers. She is brilliant, impatient with enterprise-style solutions, and fills all three of her office whiteboards before 10AM. She hired you because Dr. Yemi Okafor recommended you — and because Axiom's HIPAA audit is in 4 months and someone on the platform team left without transferring their security knowledge.

**Kofi Mensah** is in the lobby when you arrive. You know him from NexaCare, where he was Head of Platform Engineering before moving to Axiom 6 months ago. He shakes your hand: "I told Dr. Sharma to hire you. You're welcome."

The tour of the architecture takes 45 minutes. What you observe: a petabyte-scale S3 data lake organized by university name and date (no structured catalog), a custom vector similarity service for sequence matching that Kofi describes as "held together by caffeine and goodwill," and a compliance dashboard that Head of Security **Marcus Thompson** (hired 3 months ago after a HIPAA near-miss) describes as "mostly green, with areas of concern."

"Areas of concern" turns out to mean four compliance gaps, a log retention policy that has never been enforced, and a security review that identified 140 days of application logs containing partial research subject identifiers. Marcus Thompson found this on his second weekend at the company. He has not yet told Dr. Sharma.

On your first Monday, Marcus comes to your desk at 7:30 AM. He looks like he didn't sleep. He says: "Can you come to my office? Now."

You follow him. He shows you his screen.

You understand immediately.
