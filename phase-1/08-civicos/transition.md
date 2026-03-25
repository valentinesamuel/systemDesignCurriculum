# Leaving CivicOS

Nine months. Intense, important, methodical.

The data sovereignty violations were disclosed to all 7 affected states within
72 hours. Four states acknowledged the disclosure without incident. Three required
a remediation call with legal teams. No contract was terminated. The CloudFront
geographic restriction was deployed within 48 hours of the disclosure. The backup
encryption design — separate KMS keys per state — took 6 weeks to implement and
was completed without a single minute of downtime.

The enumeration attack investigation was harder. Forensic analysis determined that
~78,000 valid SSNs may have been confirmed by the attacker over 72 hours. Legal
counsel concluded it was a borderline breach notification case under most state laws
— the attacker confirmed existence of records but didn't access the content of those
records. Eight states required written disclosure anyway. None terminated their contracts.
The rate limiting redesign went live 12 days after discovery. Zero enumeration patterns
have been detected since.

FedRAMP Moderate authorization was submitted at the 4-month mark. The assessment
is pending. Jasmyn believes you'll pass.

The 89 citizens who lost form progress during the Wednesday outage — each received
a personal call from CivicOS's customer success team and a guided walkthrough to
complete their renewal. All 89 completed their renewals within a week.

You leave when the FedRAMP authorization is in final review. There's nothing more
for you to build at this stage — the architecture is sound, the compliance is in
order, and Tunde (who you've been mentoring) is ready to lead the security platform team.

---

**Slack DM — Marcus Webb**

**Marcus Webb**
Government infrastructure. Complex compliance. Serious consequences.
You've handled it well.
Next: EdTech. Completely different energy. The hardest users you'll ever serve
are students, because they care the least about the infrastructure and the most
about the outcome. Make the system invisible. That's success in education tech.
Also: notification systems. You've touched them at Beacon Media. You're going
to see the full complexity of notification infrastructure at scale in the next
chapter of your career.

---

**NeuroLearn**

**NeuroLearn** is a Series D EdTech company building adaptive learning infrastructure
for higher education. Their platform is used by 4.2 million students at 380 universities
across North America, UK, and Australia. Adaptive quizzes, study reminders, AI-powered
content recommendations, peer collaboration tools, academic integrity monitoring.

The VP of Engineering, **Obi Mensah**, has three problems he can't solve fast enough:
notification infrastructure that's failing during exam periods (sending 6 million
reminders in a 2-hour window), search that's returning increasingly irrelevant
results as the content library grows, and a database that's showing signs of collapse
under end-of-semester load.

"We have 4 million students," Obi says on the call. "When they're all cramming
at the same time, everything breaks. Fix everything."

You know that feeling. You start Monday.
