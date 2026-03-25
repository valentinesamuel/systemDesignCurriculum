# Leaving AgroSense

Eleven months. You stay longer than you expected to.

The real-time alert pipeline went live and the Kenyan Agricultural Ministry
partnership was signed. The 3-minute SLA held in production — median alert
delivery was 97 seconds. Priya sent the engineering team a note with the words
"97 seconds could save a farmer's harvest" in the subject line. You don't usually
feel that kind of tangible impact in software engineering. You feel it here.

The edge processing architecture handled the dry season burst replay correctly.
Zero false positives for the overnight battery-saving windows. The data quality
quarantine caught 2,847 corrupted readings in the first month alone — including
one soil probe that had been reporting consistently wrong calibration data for
six months, affecting three farms' irrigation decisions. Fixing that retroactively
required re-running historical derived metrics. You built the backfill pipeline.

When you leave, Ananya asks if you'd consider staying and leading the data team.
You tell her she should lead it — she understands the domain better than you
do and the engineering team respects her. She looks surprised. A week later,
Priya emails you: "I promoted Ananya. Thank you for that nudge."

---

**Slack DM — Marcus Webb**

**Marcus Webb**
Time-series data. Edge computing. Data quality. You're building a good mental
toolkit. Now you need to work somewhere that will teach you multi-tenancy —
the art of building systems that serve many customers on shared infrastructure
without any customer knowing the others exist.
B2B SaaS is the domain for that. Find somewhere with a healthy paranoia about
tenant isolation.

---

**CloudStack reaches out**

**CloudStack** is a B2B SaaS company building a developer platform — think Heroku
or Render, but for enterprise clients. They provision compute, manage databases,
and run containerized workloads for companies who don't want to manage their own
AWS accounts. Their clients range from 10-person startups to Fortune 500 engineering
teams.

The infrastructure is shared. A startup on CloudStack runs its workloads on the
same physical machines as an enterprise client's regulated financial data. The
engineering problem is keeping those customers completely isolated — in compute,
in storage, in networking, in billing — while making the platform look seamless.

**Seo-yeon Park**, the Head of Platform Engineering, frames it simply on the call:
"We have 3,200 tenants on our platform. If one tenant can affect another tenant's
performance or, worse, access another tenant's data — we're done. That's our
primary engineering constraint."

You start the following Monday.
