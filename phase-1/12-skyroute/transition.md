# Leaving SkyRoute

Six months. You leave having touched every layer of a system that moves
380 million passengers per year.

The notification system redesign shipped two weeks before the next major
weather event — an East Coast nor'easter in October that disrupted 1,400
flights at 8 airports in 4 hours. Previous system: 87,000 undelivered
notifications. New system: 1.4 million notifications delivered in 38 minutes.
P1 delivery rate: 99.7%. Amina sent a company-wide message: "Zero SLA
violations. Seven airline partners are asking when we wrote the new system."

The phantom availability rate dropped from 7% to 0.3% after the soft hold
system and real-time Elasticsearch index updates shipped. The 12.6 million
annual phantom booking failures are now estimated at 540,000 — a 96% reduction.
The support cost reduction is estimated at $18-48M annually.

The triple-booking incident (AA102) was the last of its kind. The regional
authority model deployed without incident. Cross-region booking latency for
EU agents booking US flights settled at 290ms P99 — within the 800ms SLA.
Elena Vasquez ran the numbers: zero double-bookings in the 3 months after the
regional authority model deployed. "Zero," she said in the retrospective.
"Not 0.01%. Zero."

The seat inventory harvesting attack from the auth redesign chapter: by the time
behavioral detection and hold limits deployed, the attacker accounts had already
been identified through the conversion rate monitoring. 1,247 accounts suspended.
Kai Hoffmann's post-suspension analysis: "These accounts had a 0.02% booking
conversion rate. Normal passengers: 67%."

Marcus Webb sent his usual DM:

**Marcus Webb**: SkyRoute is where everything comes together. Notifications.
Search. Auth. Rate limiting. Inventory consistency. You've seen all of them in
isolation. Now you've seen them interact. The failure mode at AA102 wasn't a
single system failing — it was two systems (Redis soft holds, Postgres replicas)
that each worked correctly in isolation but failed at their interface.
That's the most common failure mode at this level. Not a bug in one system.
A gap between systems.

---

**Slack DM — Marcus Webb**

**Marcus Webb**
Phase 1 is almost complete.

You've seen: fintech, healthcare, logistics, media, IoT, multi-tenant SaaS,
e-commerce, government, education, supply chain, ML infrastructure, aviation.

Twelve companies. Twelve industries. And the same handful of patterns
showing up everywhere, wearing different clothes.

There's one more checkpoint. A real one. The kind that separates senior
from strong senior. No more warm-up problems.

Good luck. Not that you need it.
