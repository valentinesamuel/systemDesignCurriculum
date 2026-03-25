# Leaving LuminaryAI

Five months. You arrived to fix a feature pipeline and left having redesigned
the foundation of how LuminaryAI understands what its models are doing.

The streaming inventory pipeline shipped on schedule. FashionHub's out-of-stock
recommendation rate dropped to 0.3% — from 8.4%. Dr. Nadia Osei ran the A/B
test herself: the real-time feature pipeline improved click-through rates by
22% for the clients with Kafka streams and 14% for the webhook clients.
"The model was right all along," she told you. "We just kept feeding it lies."

The observability system caught three more data quality incidents in the two
months after you built it — all before they reached the feature store. The
FashionHub incident was the last one that slipped through.

The TechMart expansion went live on April 15th. Not 40,000 RPS — TechMart's
traffic ramped more slowly than predicted, peaking at 28,000 RPS in the first
week. The infrastructure held. Jeroen sent an all-company message: "We proved
we can scale. Now let's prove we can do it without 9-day sprints."

The usage-based billing model generated $340,000 in the first month from
overages alone. Lena Kirchhoff sent you a coffee card.

Marcus Webb sent a DM the day after the TechMart go-live:

**Marcus Webb**: ML infrastructure is not ML. It's the same problems: pipelines,
caches, queues, consistency. Different data types, same failure modes. You know
that now. It was always the same problems.

You leave because SkyRoute calls. A different kind of scale problem.
Not predictions per second. Passengers per day.

---

**Slack DM — Marcus Webb**

**Marcus Webb**
SkyRoute is a global airline reservation and scheduling platform.
380 million passengers per year. 24,000 daily flights across 6 continents.

The problems you'll see there are the synthesis of everything you've built:
notifications at their third and final scale, search at its third and most
complex appearance, auth for the most adversarial user population you've
encountered, and rate limiting against travel agent bots that have been
probing APIs for years.

You've been building skills independently. SkyRoute is where they integrate.

---

**SkyRoute**

**SkyRoute** is a global airline reservation and operations platform. Their
software manages flight scheduling, seat inventory, passenger check-in, and
real-time flight status notifications for 47 airline partners operating
24,000 daily flights.

380 million passengers per year use SkyRoute-powered booking systems, often
without knowing it — when a passenger books through an airline's website,
books through a travel aggregator like Kayak or Expedia, or receives a
gate change notification on their phone, that's SkyRoute's infrastructure.

The scale is deceptive. The number that actually matters isn't 380 million
passengers — it's the 90-minute window around departure time when 50% of
all passenger interactions happen simultaneously. Check-in, seat selection,
gate notifications, delay alerts, boarding passes: all of them in 90 minutes
for 24,000 flights per day.

**VP of Engineering Amina Diallo** (who you've heard of but never met) has a
reputation for one thing: she does not tolerate systems that fail during
the 90-minute window. "Everything else," she told an interviewer once,
"is forgivable. That window is not."

You join as **Senior Backend Engineer** on the Passenger Experience team.

"We have a notification system," Amina tells you on your first day.
"It sends 8 million notifications per day. During irregular operations —
mass delays, weather cancellations — it sends 8 million in 40 minutes.
That's when it breaks."
