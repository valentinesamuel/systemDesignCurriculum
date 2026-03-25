# Leaving NeuroLearn

Ten months. The education domain stays with you in ways commercial software doesn't.

The notification system handled finals week without a single duplicate or dropped
notification. 6.4 million notifications in a 94-minute window — the largest
send event in NeuroLearn's history, triggered by a February overlap of MIT, McGill,
and LSE exam periods. Zero incidents. Obi sent a company-wide message: "The infrastructure
held. Our students got their reminders when they needed them."

The database read routing fix brought the primary DB CPU from 91% during peak to 42%.
The connection pool is no longer the bottleneck. The analytics team now exclusively
queries the read replica — they grumbled for a week and then forgot about it.

The academic integrity redesign took two months. It's the project you're most
intellectually satisfied by — not because it was technically the hardest (it wasn't),
but because it required thinking about harm and ethics alongside architecture.
Oxford, MIT, and University of Toronto all signed adoption agreements. Dr. Fitzgerald
sent a personal note: "This is what responsible educational technology looks like."

The quiz engine rearchitecture is in production. 40,000 concurrent sessions during
finals week. Average answer submission latency: 47ms. Down from 620ms.

You leave when you feel the ceiling. NeuroLearn's problems are mostly solved.
The next wave of work is incremental — tuning, optimization, monitoring. Important,
but not the kind of design work that stretches you.

---

**Slack DM — Marcus Webb**

**Marcus Webb**
You've now seen: fintech, healthcare, logistics, media, IoT, multi-tenant SaaS,
e-commerce, government, education.
Nine domains. Nine different sets of constraints. Nine different kinds of users.
But the same handful of problems, showing up in different clothes.
You're close to ready for the next level of difficulty.
Supply chain is next. It will break your brain in a specific way.
CAP theorem stops being theoretical there.

---

**OmniLogix**

**OmniLogix** is a global supply chain management platform — the backend infrastructure
that coordinates manufacturing orders, supplier procurement, inventory allocation,
and last-mile fulfillment across 47 countries. Their clients are Fortune 500
manufacturers who cannot afford supply chain failures.

The VP of Engineering, **Kwame Asante** (you recognize the name — you met him briefly
at VeloTrack), leads a team of 80. "We met at VeloTrack," he says on the intro call.
"You fixed the Kafka architecture I'd been complaining about for two years."

"We have a different kind of distributed systems problem now," he says.
"We have 14 data centers across 6 continents. And we need to agree on the same
inventory numbers at all times. We currently don't."

You know what that means. CAP theorem, in practice.
