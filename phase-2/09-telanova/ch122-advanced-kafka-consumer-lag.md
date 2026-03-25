---
**Title**: TeleNova — Chapter 122: Six Hours of Consumer Lag
**Level**: Staff
**Difficulty**: 9
**Tags**: #kafka #consumer-lag #rebalancing-storm #partition-reassignment #spike-inherited-disaster #consumer-groups
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 79 (TradeSpark Kafka log compaction), Ch. 98 (SentinelOps Kafka security events)
**Exercise Type**: Spike — Inherited Disaster
---

### Story Context

**Monday 06:45**

You're at your desk early. Jin-ho Park is on sick leave. You have no particular reason to check the Kafka dashboard — but something prompts you to open it. Maybe instinct. Maybe the fact that it's Monday morning and Monday mornings are when things break.

The consumer lag chart for `network-event-processors` shows a steep upward line. Current lag: 47 million messages. You calculate: at 3 million events/minute processing throughput, 47 million messages = 15 minutes of lag.

You watch the chart for 3 minutes. The lag is still climbing.

**Slack DM → Jin-ho Park — 06:48**

**[you]**: Jin-ho, are you around? The Kafka consumer lag for network-event-processors is climbing. I thought you said it catches up by morning.

*(no response)*

**[you]**: *(to yourself)* He's sick. No documentation. Figure it out.

---

**06:52 — Consumer group status check**

```
$ kafka-consumer-groups.sh --bootstrap-server kafka:9092 \
  --group network-event-processors --describe

GROUP                    TOPIC                  PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
network-event-processors network.events         0          847293711       893284100       45990389
network-event-processors network.events         1          REBALANCING     -               -
network-event-processors network.events         2          REBALANCING     -               -
network-event-processors network.events         3          847290001       893284200       45994199
...
[12 more partitions, 6 in REBALANCING state]
```

Six of 48 partitions in REBALANCING state. Those 6 partitions are not being consumed. They've been in REBALANCING for — you check the broker logs — 72 hours.

72 hours. This started Friday afternoon while you were in a design review.

---

**#incidents — 07:00**

**[you]**: Kafka consumer lag incident opened. network-event-processors consumer group has 6/48 partitions stuck in REBALANCING state for ~72 hours. Lag: 53M messages and growing. If not resolved, downstream SLA breach by 10:00 (network event processing backlog affects billing cycle completion for enterprise customers).

**alejandro.reyes**: On it. What do you need?

**[you]**: Access to the Kafka broker logs for the last 72 hours. And someone who knows what network-event-processors actually does. Jin-ho is sick.

**alejandro.reyes**: Fatima is on the call in 5 minutes.

---

**07:05 — Bridge**

**Fatima Al-Rashid (VP Engineering)**: The network-event-processors consumer group processes network events for billing triggers — data usage thresholds, voice minutes, SMS counts. Enterprise billing cycles close on Monday mornings. If events are 72 hours late, billing calculations for Friday-Sunday usage will be wrong.

**[you]**: How wrong?

**Fatima Al-Rashid**: €8-12M in billing adjustments, potentially. Enterprise accounts expect invoices by 10AM Monday. We can delay, but not more than 3 hours without SLA breach.

**[you]**: *(already reading broker logs)* I found it. Friday 14:37. One of the consumer instances got a `max.poll.interval.ms` timeout. The consumer was alive (still sending heartbeats) but not calling `poll()` frequently enough. Group coordinator initiated a rebalance. During rebalance, 3 other consumers hit memory pressure from the rebalance-triggered partition assignment shuffle and also timed out. Chain reaction.

**Fatima Al-Rashid**: Can we just kill and restart the consumers?

**[you]**: Yes, but we need COOPERATIVE rebalancing configured, not EAGER. If we restart with EAGER, we'll trigger another full rebalance and lose another 30 minutes. Let me patch the config first.

---

**07:48**

The consumer group is stable. 48/48 partitions consuming. Lag: 47M messages. Catchup rate: 3M/min. ETA to zero lag: 15 minutes.

**07:49 — Slack**

**[you]**: Consumer group stable. ETA to zero lag: 15 min. Billing cycle will close by 09:20 — within SLA.

**alejandro.reyes**: How did this stay hidden for 72 hours?

**[you]**: No consumer lag alerting configured. Jin-ho knew it happened occasionally and manually checked. When he got sick, nobody else knew to look.

**alejandro.reyes**: That's the real problem. Fix the architecture after we close the incident.

---

### Problem Statement

TeleNova's Kafka consumer group for network event processing has been in a partial rebalancing state for 72 hours after a `max.poll.interval.ms` timeout triggered a chain-reaction rebalance. Six of 48 partitions are unconsumed. The underlying issues: EAGER rebalancing configuration, no consumer lag alerting, and absence of documentation. Fix the immediate incident and redesign the consumer architecture to prevent recurrence.

---

### Explicit Requirements

1. Restore all 48 partitions to active consumption within 30 minutes (before 10:00 billing deadline)
2. Implement consumer lag alerting: alert at 5-minute lag, page at 15-minute lag
3. Configure COOPERATIVE incremental rebalancing to eliminate chain-reaction rebalances
4. Set `max.poll.interval.ms` and `session.timeout.ms` appropriately for the actual processing workload
5. Consumer group recovery runbook: documented procedure that any on-call engineer can follow
6. Partition reassignment strategy for planned consumer changes (not just failure recovery)

---

### Hidden Requirements

- **Hint**: Re-read the billing timeline: "Enterprise accounts expect invoices by 10AM Monday." The 47M-message lag at 3M/minute throughput = 15 minutes to catchup. But what if the lag has been building for 72 hours AND the upstream producers are still producing at normal rate (3M/min)? The consumer must process both the backlog AND the live incoming stream simultaneously. What is the actual catchup time if live events are still arriving at full rate?

- **Hint**: Jin-ho said "it catches up by morning" — this implies this rebalancing has happened before. It's a recurring issue, not a one-off. What does that tell you about the root configuration problem?

---

### Constraints

- 48 partitions, 12 consumer instances, `network.events` topic
- Processing rate: 3M events/minute per consumer group
- Incoming rate: 3M events/minute (at full production load)
- Billing deadline: enterprise invoices by 10:00 AM Monday
- `max.poll.interval.ms` current setting: 5 minutes (too short for heavy processing tasks)
- Rebalancing protocol: currently EAGER (all consumers drop partitions on rebalance)

---

### Your Task

Resolve the immediate Kafka consumer lag incident and redesign the consumer configuration to prevent recurrence.

---

### Deliverables

- [ ] Immediate recovery procedure: exact steps to restore 48/48 partitions without triggering another rebalance
- [ ] Catchup time math: with 47M backlog AND 3M/min live incoming rate, what is the actual time-to-zero lag?
- [ ] `max.poll.interval.ms` tuning: correct calculation based on actual processing time per batch
- [ ] COOPERATIVE rebalancing configuration: settings and behavioral explanation
- [ ] Consumer lag alerting design: Kafka consumer group metrics, alerting thresholds, PagerDuty escalation
- [ ] Consumer scaling strategy: how to add/remove consumers without triggering service disruption
- [ ] Runbook: 5-step emergency procedure for any on-call engineer to diagnose and resolve future rebalancing incidents
- [ ] Tradeoff analysis (minimum 3):
  - EAGER vs COOPERATIVE vs STATIC membership rebalancing — when each is appropriate
  - Single consumer group vs multiple consumer groups for the same topic (isolation vs throughput)
  - Auto-commit vs manual offset commit — exactly-once processing vs simplicity for billing events
