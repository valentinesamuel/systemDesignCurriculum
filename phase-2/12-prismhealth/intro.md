# PrismHealth — Company Introduction

**Industry**: Digital Health / Remote Patient Monitoring
**Your Role**: Principal Architect
**Arc**: Chapters 138–143

---

PrismHealth monitors 8 million patients at home across 500 hospital systems. Pulse oximeters, continuous glucose monitors, blood pressure cuffs, ECG patches, fall detection devices — 10 million connected devices reporting to a central platform that nurses, physicians, and care coordinators use to watch for deterioration signals before they become emergencies.

CEO **Dr. Ngozi Obi** is a cardiologist who cofounded PrismHealth after a specific incident. Her patient Walter Reyes, 67, had a remote oximeter showing a concerning trend for six consecutive hours. The alert was in the queue. The on-call system deprioritized it. Walter's SpO2 continued declining. He arrived at the emergency department with a pulmonary embolism that required ICU-level intervention. He survived — barely. The investigation showed the alert had been categorized as "low priority" by the platform's triage algorithm, which weighted severity based on historical patterns from a predominantly younger patient population. Walter didn't fit the model.

Dr. Obi built PrismHealth so that wouldn't happen again.

On your first day, she tells you this story. Directly. Without softening it. She ends: "Every architectural decision we make is in service of Walter not making it to the emergency department too late. I need you to hold that in mind."

You meet **Priya Nair** in the engineering corridor — the same Priya Nair from Stratum Systems and the Apex Financial checkpoint panel. She has been CTO at PrismHealth for 8 months. She smiles when she sees you: "I wondered when our paths would cross again. I told Dr. Obi to reach out to you."

The platform processes 14 million device readings per minute. Two months ago, a Boston hospital experienced a 40-minute alert gap during a network partition. Three patients had elevated readings during that window. All three were fine. The retrospective review concluded the alert gap was acceptable "given the circumstances." Dr. Obi does not agree that any alert gap is acceptable.

**Staff Engineer Jake Huang** owns the IoT ingestion pipeline. He is good at what he does and knows it. He is not defensive about the gap incident — he's already redesigning the failover. He needs a partner who can see the full picture.

The HIPAA audit is in 6 months. You've been here before.
