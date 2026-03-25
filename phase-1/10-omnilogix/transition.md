# Leaving OmniLogix

Four months at OmniLogix. The hardest systems work you've done so far.

The global inventory consistency redesign shipped in three phases over eight
weeks. Phase 1: the quorum-based allocation layer, rolled out DC by DC with
a feature flag. Zero double-allocation incidents since go-live. Phase 2:
the saga orchestrator with the outbox relay. Adaora's team went from 300
manual fixes per day to 41. She sent a Slack message: "You automated the
boring half of my job. I'm spending the rest of the time on the interesting
half. This is better." Phase 3: the idempotency proxy for SupplierAPI and
CarrierAPI. PM-1442 can't happen anymore.

The BMW double-allocation incident that started the whole project cost $240,000
in penalties. The triple-billing incident from the non-idempotent saga cost
$84,000. The engineering investment to fix both: three months, two engineers.
Kwame put those numbers in the all-hands deck. "This is what distributed
systems work looks like when it ships."

Marcus Webb sent a DM the day the Phase 3 rollout completed:

**Marcus Webb**: Good work. Not because it worked. Because you understood
why each piece was necessary before you built it. A lot of engineers would
have built the saga and called it done. You understood that the saga without
idempotency is a more expensive way to break things. That's the difference.

You leave when Kwame gets a new budget approved. Two junior hires are joining
the distributed systems team. Your work is becoming their foundation. Time
for a new ceiling.

---

**Slack DM — Marcus Webb**

**Marcus Webb**
Next arc. You're going into machine learning infrastructure.

Not model training — data pipelines. Feature stores. The infrastructure
that keeps models fed and honest.

ML systems fail in a specific way: silently. The model doesn't crash.
It gives wrong answers. And no one knows until a business metric drops
and someone investigates. That's your next problem.

At LuminaryAI, they're serving 140 million predictions per day and they
have no idea if half of them are correct.

---

**LuminaryAI**

**LuminaryAI** is an AI-powered marketing personalization platform. Their
system processes user behavior signals from e-commerce clients and serves
real-time product recommendations. 140 million predictions per day. 47
e-commerce clients, each with different user bases, product catalogs,
and behavioral patterns.

The data team, led by **Dr. Nadia Osei**, has built a world-class set of
recommendation models. The infrastructure team, led by **Jeroen van der Berg**,
has built a world-class serving layer. What nobody has built is the pipeline
between them: the feature store, the training data pipelines, the model
monitoring infrastructure.

The models are getting stale. Behavioral patterns shift. Campaigns run for
a week and then stop. Product inventory changes daily. The models don't know.

You join as the **Senior Backend Engineer** on the ML Infrastructure team —
the bridge between the data scientists and the serving infrastructure.

"The models are right," Dr. Nadia Osei tells you on your first day.
"When we trained them. Last month."

That's the problem.
