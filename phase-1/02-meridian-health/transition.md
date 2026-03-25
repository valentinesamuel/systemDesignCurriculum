# Leaving MeridianHealth

Cascade Health went live on schedule. The load test hit 500 concurrent users with
P99 at 387ms — 13ms under the contractual SLA. Sandra sent a company-wide Slack
message: "Engineering is the reason we keep this deal." Ravi printed it and taped
it to the door of the server room, which is a thing he apparently does.

You stay another four months after the Cascade launch. The data residency project
gets completed. The audit trail passes Northview's compliance review. Nalini sends
you a thank-you note that feels like a hug in email form.

But the work starts to feel solved. You've put out the fires. The architecture is
stable. There's a part of you that needs to be somewhere moving faster.

A recruiter reaches out. **VeloTrack** — a Y Combinator-backed logistics startup
building real-time delivery tracking infrastructure. "Like Google Maps for last-mile
delivery," she says. "Except we're building the backend that powers it."

The CTO is the co-founder. The engineering team is nine people. The platform
processes 4 million delivery events per day and they can't keep up.

---

**Slack DM — Marcus Webb → You**

**Marcus Webb**
Logistics, huh. Different domain. Same problems. You'll see.
One prediction: within two weeks they'll show you a Kafka setup that's being
used as a database. I've seen it at three logistics companies.
Don't let them convince you it's fine.

---

**Day 1 at VeloTrack**

The CTO, **Emeka Eze**, gives you the tour. He's been coding since he was twelve
and he has the energy of someone who's just discovered that what he built is much
larger than he planned for. "We're processing 4 million events a day," he says.
"Last month it was 2 million. In three months, we think it's going to be 20 million."

He pauses at a whiteboard covered in boxes and arrows.

"We have a slight Kafka problem."

Marcus Webb was right. It's always Kafka.
