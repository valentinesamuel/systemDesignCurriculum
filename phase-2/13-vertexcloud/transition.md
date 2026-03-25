# VertexCloud — Transition

## Departure Scene

**Slack DM — Yael Cohen → [you] — Month 14, Friday 17:23**

**Yael Cohen**: I want you to know something before the all-hands next week. When you joined, we had two failed IDP attempts, $1.1M/month unallocated cloud spend, engineers bypassing GitOps with kubectl, and a chaos engineering test that accidentally became a production incident.

Fourteen months later: IDP adoption is at 63%. Unallocated cloud spend is $180k/month. Bypass rate is 3.2%. The auth service has a real fallback. The quarterly game day program ran its second iteration last week without a single unintended cascade.

You didn't build a platform. You built a culture change with a platform on top of it.

**[you]**: The teams did the work. I just made the paved road slightly less muddy.

**Yael Cohen**: That's the job. Don't underestimate it.

She pauses.

**Yael Cohen**: I heard you got a call from Meridian Exchange. I'm not going to try to counter-offer on behalf of Reza — that's not my place. But if you go, go knowing what you built here.

---

## What You Leave Behind

**VertexCloud — 14 months**

The platform is not finished. Platforms are never finished. But the trajectory has changed.

The IDP serves 8 of 14 teams on the golden path. The remaining 6 are on the migration roadmap, with scheduled job support (Lucas Oliveira's blocker) shipping in the next quarter. The escape hatch is documented and used — 3 teams have non-standard infrastructure that integrates cleanly with the platform's CI/CD and observability layers.

The cloud cost dashboard runs every morning at 07:00. Every Engineering Manager sees their team's spend. The GPU cluster incident — $152,000 in waste for a 2-week ML experiment — has not recurred. The 14-day idle threshold for high-cost resources has fired four times, recovering $290,000/year in projected waste.

The chaos engineering program survived its own near-disaster. The post-incident process produced an undocumented-dependency detection tool that now runs in CI: when a service is deployed with a new external HTTP call, the pipeline flags it for service catalog review. Forty-three previously undocumented dependencies have been surfaced in the three months since launch.

The 300-API governance system: 47 APIs now have OpenAPI specs (up from 12%). The top 20 by consumer count all have specs. Breaking change detection has blocked 9 PRs that would have been production incidents.

The bypass rate — 3.2% — is not zero. Marcus Webb's words are still in your notes: *"The right thing must be the easy thing."* You got it to 3.2%. The next engineer will get it lower.

---

## The Call

**Phone — Wednesday 19:47**

The voice on the other end is a VP of Engineering at Meridian Exchange, a real-time financial data platform. 50 million market events per second. 10,000 institutional subscribers. Sub-millisecond fanout SLAs written into legal contracts with investment banks.

The problem they want you to solve: their current distribution architecture was designed for 5 million events per second. They've grown 10x in 18 months. The system is not breaking yet, but the cracks are visible, and the next large client — a sovereign wealth fund in Singapore — will require a regional failover guarantee they cannot currently provide.

"We'd like to bring you in for a design review," they say. "Think of it as a conversation."

You know what it is. You've been on the other side of design review conversations that weren't conversations.

You say yes.

---

*Next: Checkpoint 4 — Meridian Exchange*
