---
**Title**: NovaSports — Chapter 251: Before Next Season
**Level**: Staff
**Difficulty**: 9
**Tags**: #architecture-review #streaming #cost-engineering #capacity-planning #kafka #performance-optimization
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 245, Ch. 248, Ch. 249, Ch. 250
**Exercise Type**: System Evolution
---

### Story Context

**From:** Kenji Watanabe <kenji.watanabe@novasports.com>
**To:** You
**CC:** Rosa Delgado
**Subject:** Joint Architecture Review — Event Streaming Platform
**Date:** Tuesday, 3 weeks after the game day incident

Hi,

I've been meaning to send this for two weeks, but the incident aftermath kept pushing it.

Here's where we are: since the game day incident, I've been running a continuous dependency graph analysis on our streaming infrastructure. What I found is more concerning than the one undiscovered dependency we found during the game day. There are eleven more.

Not all of them are dangerous. Most of the undiscovered dependencies are read-only connections from analytics to event streams. But three of them touch the betting critical path in ways that weren't designed — they grew organically as engineers added features over the last 18 months.

I've also been looking at the cost numbers. We're currently spending $1.8M/month on our Kafka and stream processing infrastructure. Based on the traffic analysis I've run, I believe we can get to $1.2M/month without degrading any SLA — if we're willing to do the architecture work.

Season opens in 8 weeks. I don't want to be back in an incident bridge because we didn't take this opportunity seriously.

I'm proposing we schedule a 3-day joint architecture review. You, me, Rosa, Chioma, and one external. I want to propose bringing in an external reviewer who's run streaming infrastructure at sports-scale before — not to validate our choices, but to find what we're missing before the season does.

Are you in?

— Kenji

---

**You [Slack DM to Rosa, 11:02 AM]:**
> "Kenji found 11 more undiscovered dependencies. Three touch the betting critical path."

**Rosa Delgado [Slack DM, 11:03 AM]:**
> "I know. He showed me last night. The cost analysis is also compelling. $600K/month delta. That's $7.2M/year."
> "Dev is going to want to hear that number."

**You [Slack DM, 11:04 AM]:**
> "I'm in for the architecture review. Who's the external reviewer?"

**Rosa Delgado [Slack DM, 11:05 AM]:**
> "Kenji wants to bring in someone from his previous company. Streaming infrastructure. Sports data background."
> "I suggested Marcus Webb."

**You [Slack DM, 11:06 AM]:**
> "Marcus is retired."

**Rosa Delgado [Slack DM, 11:06 AM]:**
> "He does occasional consulting. Kenji asked him directly. Marcus said: 'I'll do one day, not three. And I charge by finding problems you didn't know you had.'"

**You [Slack DM, 11:07 AM]:**
> "That sounds exactly like him."

---

**ARCHITECTURE REVIEW — DAY 1 — 9:00 AM**

The review room has four whiteboards, three laptops streaming Grafana dashboards, and a printed dependency graph that Kenji generated from three months of OpenTelemetry data. It's large enough to require taping two sheets of paper together.

Marcus Webb joins via video call at 9:05. He looks older than the last time you saw him. More relaxed. There's no urgency in his eyes — just the deep, practiced patience of someone who no longer needs to prove anything.

**Marcus Webb:** Good morning. Show me the dependency graph first. Before you show me anything else.

Kenji pins the dependency graph to the wall. Marcus is quiet for a long time.

**Marcus Webb:** Three things jump out at me before we talk about anything technical. First: your Kafka topics. How many do you have?

**Kenji Watanabe:** 847.

**Marcus Webb:** How many should you have?

Silence.

**Marcus Webb:** That's the question I'd start with. When I see 847 topics in a sports platform, I see 18 months of engineers creating topics without a topic governance policy. I see consumers that were written to read from the wrong topics. I see partitioning strategies that made sense for the first use case and became wrong for the second. How many of those 847 topics have exactly one consumer?

**Chioma Osei:** I don't know.

**Marcus Webb:** That's a system that's grown faster than your understanding of it. I've seen this exact pattern before. The scary part isn't the 847 topics. The scary part is that you're about to run your busiest season on it.

He pauses.

**Marcus Webb:** The good news: this is fixable in 8 weeks if you're disciplined. I've seen it fixed in 6. The cost savings Kenji found are real — I saw them in the numbers before I got on this call. But the cost savings are a side effect. The primary output of this review should be: a streaming architecture you can operate under pressure. The kind of pressure you saw six weeks ago, except at 3x the scale.

He looks directly at you.

**Marcus Webb:** You're the architect on this. What's the first question you want to answer?

---

**Day 3 — End of Review**

By the end of day 3, you have:
- A current-state architecture map (generated from trace data)
- Identified all 14 undiscovered dependencies
- A topic consolidation plan: 847 topics → 94 topics
- A partitioning strategy review
- A cost model showing path from $1.8M to $1.1M/month (Kenji's estimate was conservative)
- Three major architectural risks that survive all the changes
- A list of things you will NOT fix before the season starts (and why)

Marcus's final comment before signing off:

**Marcus Webb:** The list of things you're NOT fixing is the most important thing you produced today. I've seen more systems break in the sprint to fix everything than in the decision to fix only the right things. You know what you're doing. Ship what's safe. Park the rest.

He pauses.

**Marcus Webb:** Also — whoever wrote the postmortem for the game day incident: it was the best incident document I've read in three years. The section on blast radius amplification alone is worth keeping.

---

### Problem Statement

Following the game day incident and a 3-day architecture review, you must redesign NovaSports's event streaming platform for the upcoming season. The review has revealed 847 Kafka topics (requiring consolidation), 14 undiscovered service dependencies (3 touching the betting critical path), a cost overrun of $600K/month versus optimal, and three major architectural risks that must be mitigated before season start. The system must handle 50M events/sec sustained for 6 months, three major playoff events at 15M concurrent users, and reduce monthly cost from $1.8M to $1.2M — all within an 8-week implementation window.

### Explicit Requirements

1. Kafka topic consolidation: 847 → ~94 topics with formal governance policy
2. Partitioning strategy review and correction for all topics touching betting critical path
3. All 3 critical-path undiscovered dependencies must be documented and circuit-broken
4. Cost reduction: $1.8M/month → $1.2M/month for streaming infrastructure
5. Sustained capacity: 50M events/sec for 6 months (regular season)
6. Peak capacity: 15M concurrent users during 3 playoff events
7. Season start deadline: 8 weeks
8. Define what will NOT be fixed before season start (explicit parking lot with rationale)

### Hidden Requirements

1. **Hint: re-read Marcus Webb's comment about 847 topics.** Topic governance is not just about consolidation — it's about preventing the problem from recurring. The architecture review output must include a topic creation policy with approval gates, not just a one-time cleanup. Otherwise, by next season, you'll be back at 600+ topics.

2. **Hint: re-read the cost numbers carefully.** Kenji estimated $1.2M/month; Marcus's analysis suggested $1.1M is achievable. The difference is likely in partition count (over-partitioned topics waste broker memory and replication bandwidth) and consumer group count (idle consumer groups hold partition offsets and consume coordinator resources). The architecture review should model both.

3. **Hint: re-read "three major architectural risks that survive all the changes."** These three risks are not named in the story. You must derive them from the system context (50M events/sec, betting critical path, 3 playoff events). What are the top three risks that cannot be fully mitigated in 8 weeks? Your design must name them explicitly and define monitoring/alert strategy for each.

### Constraints

- **Current state**: 847 Kafka topics, 14 undiscovered dependencies, $1.8M/month
- **Target state**: ~94 topics, all dependencies documented, $1.2M/month
- **Sustained throughput**: 50M events/sec regular season
- **Peak throughput**: 15M concurrent users × assumed 5 events/sec/user = 75M events/sec peak
- **Playoff events**: 3 events, each expected to exceed regular season peak by 2x
- **Season start**: 8 weeks from review
- **Team**: 5 engineers available for streaming infrastructure work (others on feature development)
- **Kafka cluster**: currently 48 brokers, 847 topics, ~120K partitions total
- **Consumer groups**: ~2,400 consumer groups registered (many suspected idle)
- **Monthly cost breakdown**: $820K Kafka brokers, $540K stream processing compute, $440K storage/replication

### Your Task

Produce the streaming architecture redesign. Define the topic consolidation strategy and governance policy. Correct the partitioning strategy for critical-path topics. Document all 14 dependencies with circuit-breaker assignments. Show the cost reduction model. Define the 8-week migration plan including explicit parking lot items. Identify the three major architectural risks that survive the redesign.

### Deliverables

- [ ] Mermaid architecture diagram: redesigned Kafka topology (consolidated topics, correct partitioning, circuit breakers annotated)
- [ ] Topic consolidation plan: from 847 to target count — grouping rationale, migration strategy for each topic group
- [ ] Topic governance policy: creation requirements, approval gates, deprecation process
- [ ] Partitioning strategy specification: partition count formulas for each topic category, key selection rationale
- [ ] Dependency documentation: all 14 dependencies mapped, 3 critical-path ones with circuit-breaker configurations
- [ ] Cost model: line-item breakdown from $1.8M to $1.2M (show which changes produce which savings)
- [ ] 8-week migration plan: sequenced by risk, with week-by-week milestones
- [ ] Explicit parking lot: what is NOT being fixed and why, with monitoring strategy for known risks
- [ ] Three major architectural risks: named, described, and monitored
- [ ] Scaling estimation (show math):
  - 75M events/sec peak: broker count, partition count, consumer group sizing
  - Memory footprint with consolidated topic count
  - Replication factor and storage at scale
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs)
- [ ] Cost modeling ($X/month before and after, with line-item breakdown)
- [ ] Capacity planning: 6-month regular season + 3 playoff peaks

### Diagram Format

Mermaid syntax. Two diagrams:
1. **Current state**: 847-topic chaos with hidden dependencies (simplified representation, showing major groupings and the 3 dangerous paths)
2. **Target state**: consolidated topology with governance zones, circuit breakers, and explicit dependency contracts
