---
**Title**: NovaSports — Chapter 247: 35 Million Opinions, Updated in Real Time
**Level**: Staff
**Difficulty**: 9
**Tags**: #ml-systems #feature-stores #personalization #real-time #recommendation #cache
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 245, Ch. 246, Ch. 56
**Exercise Type**: System Design
---

### Story Context

**#data-science — Thursday, 2:14 PM**

**Yusuf Adeyemi [Head of Data Science]:** I need everyone's attention. We just finished the 90-day A/B test. Final numbers are in.

**Aiko Tanaka [Data Scientist]:** Go ahead.

**Yusuf Adeyemi:** Users with personalized feeds: 3.1x engagement versus rule-based. Time on platform: up 47%. Push notification open rate: up 82% when recommendation score is used for send timing. Revenue per user from personalized ad placement: up 2.3x.

**Dev Chatterjee [CTO]:** These are the best numbers we've ever seen.

**Yusuf Adeyemi:** I know. And they're from a test with 500K users. Our rule-based system serves 35 million. The revenue delta if we roll this out... I don't want to say the number in a channel.

**Dev Chatterjee:** Everyone who matters is in this channel, Yusuf. Say the number.

**Yusuf Adeyemi:** $80M ARR upside. Conservative.

Silence in the channel for eleven seconds. You count.

**Dev Chatterjee:** What do we need?

**Yusuf Adeyemi:** The model is ready. The training pipeline is ready. The problem is serving. I need to give every one of our 35 million users a personalized recommendation in under 50ms. And I need those recommendations to update in real time when something happens in a live match. Not batch. Not hourly. Real time.

**Rosa Delgado [Staff Eng]:** Define "real time."

**Yusuf Adeyemi:** Player scores a goal. Every user who follows that player gets their feed updated within 30 seconds. Their recommendation ranking changes. The users most likely to engage with that content move to the top of their queue.

**Rosa Delgado:** 30 seconds. Okay. How many users follow any given top-tier player?

**Yusuf Adeyemi:** Lionel Messi has 11 million followers on our platform.

Another silence.

---

You open your laptop and start sketching. The math hits immediately:

- 35M users
- Each user has a feature vector: 200 features (player affinities, team loyalties, engagement history, recency)
- Feature updates from live events: up to 500K users affected per goal (fan clusters overlap)
- If a top-tier player scores: 11M feature vectors need updating within 30 seconds
- Each feature vector update triggers a recommendation re-rank for that user
- Serving latency must stay under 50ms

You send Rosa a message.

**You [Slack DM to Rosa, 3:47 PM]:**
> "I've been thinking about the fan-out problem. 11M feature updates in 30 seconds is 366K writes/sec. That's before the re-rank compute."
> "The model produces a recommendation score for each <user, content_item> pair. If we precompute and cache all scores for all users, that's 35M × ~1000 content items = 35 billion scored pairs. Just in memory: 280GB at 8 bytes per score. Not viable."
> "We can't precompute everything. We can't compute at serve time for hot events. This is the thundering herd problem plus feature freshness problem at the same time."

**Rosa Delgado [Slack DM, 3:52 PM]:**
> "And you haven't even gotten to the EU problem yet."
> "EU users: we can't ship their behavioral features to US-East for scoring. The feature store has to live in EU-West. The model serving has to live in EU-West."
> "What does that do to your 50ms SLA when model inference happens in-region but player event data originates in US-East?"

**You [Slack DM, 3:54 PM]:**
> "It means we need event propagation latency under 10ms across regions before we even start scoring. And the model weights need to be replicated to every region."
> "This is four separate systems: feature engineering, feature store, model serving, recommendation cache. All have to talk to each other faster than a goal celebration."

**Rosa Delgado [Slack DM, 3:56 PM]:**
> "Yusuf's coming to you tomorrow morning. He's bringing the model card and wants an architecture proposal by EOD. I told him you'd have something."

**You [Slack DM, 3:57 PM]:**
> "You told him that before you told me."

**Rosa Delgado [Slack DM, 3:57 PM]:**
> "That's how I know you'll have something."

---

**From:** Yusuf Adeyemi <yusuf.adeyemi@novasports.com>
**To:** You
**Subject:** Personalization Architecture — Friday Meeting Prep
**Date:** Thursday, 6:01 PM

Hi,

Ahead of tomorrow: a few numbers that might help your architecture thinking.

Model characteristics:
- Input: 200-dimensional user feature vector + 50-dimensional content feature vector
- Output: Single score (0.0–1.0) representing engagement probability
- Model size: 180MB (can be replicated to edge)
- Inference time on GPU: 8ms per 1000 users (batch)
- Inference time on CPU: 45ms per 1000 users (batch)
- Top-K recommendation: we serve top 20 items per user per session

Feature freshness SLA from data science:
- Long-term affinity features (team loyalty, sport preferences): can be stale up to 24 hours
- Medium-term features (this week's engagement patterns): < 1 hour
- Real-time event features (player just scored, team just conceded): < 60 seconds

One more thing: we are contractually obligated to our Premier League partners to NOT use viewing data from one user to influence recommendations for another user who hasn't consented to data sharing. Consent is per user, per purpose.

See you tomorrow.
— Yusuf

---

### Problem Statement

NovaSports must replace its rule-based recommendation system with a real-time ML personalization system serving 35M users at < 50ms recommendation latency. The system must handle live event triggers — a top-tier player scoring a goal causes feature updates for up to 11M users within 30 seconds, which then triggers recommendation re-ranking. Feature freshness has three tiers with different staleness tolerances. EU users' behavioral features must not leave EU infrastructure (GDPR + contractual). Consent-gated data sharing affects what signals can be used across users.

### Explicit Requirements

1. Recommendation serving latency: < 50ms p99 per user
2. Feature update propagation for live events: < 60 seconds for real-time tier features
3. Top-K recommendation: 20 items per user per session refresh
4. Feature store tiers: 24-hour, 1-hour, 60-second staleness by feature type
5. Multi-region: feature stores and model serving in each region (EU features stay in EU)
6. Model can be replicated globally (180MB); inference GPU preferred, CPU fallback
7. Handle fan-out: 11M feature updates in 30 seconds for top-tier player goal event
8. 35M users total; 4.8M in EU with strict data residency

### Hidden Requirements

1. **Hint: re-read Yusuf's email, last paragraph.** Consent-per-purpose means some users have opted out of cross-user data sharing. Collaborative filtering (which uses patterns across users to recommend) is disabled for those users — the system must fall back to content-based signals only. The architecture must support two model paths per user.

2. **Hint: re-read the 11M followers number.** A goal by a top-tier player causes 11M feature updates. But those 11M users don't all request recommendations simultaneously — the fan-out on the *feature write* side is decoupled from the *read* side. The architecture must handle write fan-out without triggering a thundering herd on the recommendation cache.

3. **Hint: re-read the EU data residency constraint combined with model replication.** Model weights can be replicated (they contain no user PII), but feature vectors for EU users cannot leave EU-West. However, if a new content item is created in US-East (a new highlight, a new article), its content feature vector must reach EU-West model serving within the recommendation freshness SLA.

### Constraints

- **Users**: 35M total, 4.8M EU (data-residency constrained)
- **Model size**: 180MB, GPU inference 8ms/1K users batch, CPU 45ms/1K
- **Feature dimensions**: 200 user features + 50 content features per scoring pair
- **Top-K per user**: 20 recommendations per session
- **Live event fan-out peak**: 11M feature writes in 30 seconds = 366K writes/sec burst
- **Recommendation request rate**: 35M users × 10 sessions/day / 86,400s = 4,050 RPS baseline; peak match: 35,000 RPS
- **Feature store memory**: 35M users × 200 features × 8 bytes = 56GB just for user features
- **Content catalog**: ~50K active items, refreshed continuously
- **Latency SLA**: < 50ms p99 end-to-end (feature lookup + model inference + cache layer)
- **Infrastructure budget**: $220K/month for personalization stack

### Your Task

Design the real-time personalization architecture. Define the feature store design (tiered by freshness). Design the fan-out pipeline for live event feature updates without triggering thundering herd on the recommendation cache. Define the model serving architecture (where inference runs, how models are deployed per region). Explain how consent-gated users get a different model path. Show the EU data residency compliance for features.

### Deliverables

- [ ] Mermaid architecture diagram: feature engineering → feature store → model serving → recommendation cache → API layer
- [ ] Feature store schema (three freshness tiers, with partition strategy and TTLs)
- [ ] Fan-out design for live events: how 11M feature updates happen in 30 seconds without cache stampede
- [ ] Model serving design: GPU fleet sizing, CPU fallback, per-region deployment
- [ ] Consent-path routing: how users with restricted consent get a different inference path
- [ ] Database schema for user feature store (with column types, indexes, and region-sharding strategy)
- [ ] Scaling estimation (show math step by step):
  - Feature store write throughput at peak fan-out
  - Model serving RPS and GPU fleet sizing
  - Recommendation cache hit rate required to stay under 50ms SLA
- [ ] Tradeoff analysis (minimum 3 explicit tradeoffs)
- [ ] Cost modeling ($X/month for personalization stack)
- [ ] Capacity planning: 12-month horizon assuming 20% user growth + EU expansion

### Diagram Format

Mermaid syntax. Your diagram must show at minimum:
- Event ingestion (live match events → feature engineering pipeline)
- Feature store (three tiers, region-sharded for EU compliance)
- Model serving layer (GPU primary, CPU fallback, per-region)
- Recommendation cache (pre-computed vs just-in-time)
- API serving layer
- The consent-path fork (collaborative filtering vs content-based)
- Cross-region model weight replication (separate from feature replication)
