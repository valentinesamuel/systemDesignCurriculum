---
**Title**: ForgeSense — Chapter 203: OPC-UA to Cloud
**Level**: Strong Senior
**Difficulty**: 7
**Tags**: #iot #opc-ua #data-pipeline #brownfield #reliability #edge
**Estimated Time**: 3.5 hours
**Related Chapters**: Ch. 202, Ch. 206
**Exercise Type**: System Design
---

### Story Context

**Slack — #platform-team — Wednesday, 10:14 AM**

```
Nikolaj Brandt [10:14 AM]
Quick heads-up: just got a page. The Detroit factory connector
dropped again.

You [10:15 AM]
The Node.js bridge?

Nikolaj Brandt [10:16 AM]
Yeah. Third time this week. I restarted it. It's back.
But this is what I'm talking about — we have a pilot factory
in Germany (Ludwigshafen) onboarding in 6 weeks and that
bridge is the same codebase.

You [10:18 AM]
Who wrote the bridge?

Nikolaj Brandt [10:19 AM]
...an intern. Two years ago. I don't even remember his name.
It worked well enough for the pilot. Now it's in production
serving live traffic and nobody has touched it.

You [10:20 AM]
Is there a README?

Nikolaj Brandt [10:21 AM]
There is a comment in the index.js that says
"TODO: add error handling"
That's the documentation.

You [10:22 AM]
I'll look at it today.

Nikolaj Brandt [10:22 AM]
Good luck. The OPC-UA library it uses was deprecated in 2021.
```

---

**Slack — DM from Svetlana Volkov — Wednesday, 11:30 AM**

```
Svetlana Volkov [11:30 AM]
How's the bridge audit going?

You [11:42 AM]
It's worse than I thought. No retry logic. No reconnection
backoff. The OPC-UA session dies every 72 hours due to a
keep-alive timeout bug in the library and the process just
hangs instead of restarting.

Svetlana Volkov [11:43 AM]
How much data do we lose in the hang?

You [11:44 AM]
Depends. Average hang is 4-6 minutes before our monitoring
catches it and pages. At 50K sensors, that's 3-4.5 million
readings per hang. Completely lost. No buffer, no replay.

Svetlana Volkov [11:45 AM]
Karl Bauer knows about the outages. He's been patient
because it's pilot. When Detroit goes to production, he
won't be patient.

You [11:47 AM]
There's something else. I found a comment in the bridge code
from 8 months ago. It says:
"HACK: FDA audit mode — stop writes to this endpoint
during audit window. See email from Karl 2024-03-12."
There's no implementation. It's just a comment.
The audit mode was never built.

Svetlana Volkov [11:48 AM]
That email is in Karl's inbox, not ours. We don't have
a copy of what was agreed. You need to call Karl.

You [11:49 AM]
So we have an undocumented customer commitment embedded
in a comment in intern code, for a feature that doesn't exist,
in a bridge that crashes every 72 hours.

Svetlana Volkov [11:50 AM]
Welcome to legacy systems. Write up what you found.
We'll need it for the architecture review Thursday.
```

---

**Call transcript — You and Karl Bauer (VP Operations, Detroit Tier 1), Wednesday 2 PM (reconstructed from notes)**

**Karl**: "The audit mode thing — yes, I remember that conversation. When ISO TS 16949 auditors are on the floor, they want a static snapshot of sensor data. No incoming writes to the records they're reviewing. But the factory can't stop running. So we need writes to continue for the live dashboard, but the audit log view has to be frozen. It's a split-write requirement."

**You**: "How long do audit windows typically last?"

**Karl**: "Three to four hours per session. Maybe two sessions a day during a quarterly audit. And the data we're freezing — it's specific production line data, not all sensors. The auditors tell us in advance which lines they're reviewing."

**You**: "And does this need to cover the edge bridge layer, or just the dashboard query layer?"

**Karl**: "Both, ideally. But if you can only do one, the dashboard. The auditors never touch the bridge layer."

**You**: "Last question: does the frozen snapshot need to be cryptographically tamper-evident? Like, if we're claiming 'this was the state at 10:07 AM,' can we prove it hasn't been modified?"

**Karl**: [long pause] "Nobody has ever asked me that before. But... yes. If an auditor ever challenged our records, we'd need to prove they're authentic. Can you do that?"

**You**: "Yes. But it changes the design."

---

**Design review prep — your notes, Wednesday evening**

The bridge problem has two components: reliability (it crashes) and correctness (the audit mode commitment exists but was never built). Both need to be solved before the 3-factory onboarding in Q2.

The deeper architectural question: the bridge is currently the only path from factory floor to cloud. It's a single point of failure serving live production traffic. For 12 factories, that model doesn't work at all. Each factory needs an independent, self-healing bridge that can buffer locally if the cloud connection drops and drain the buffer when connectivity returns.

---

### Problem Statement

ForgeSense's OPC-UA bridge — the component that connects factory floor PLC/OPC-UA systems to the cloud platform — is a brittle single point of failure. It crashes every 72 hours, has no retry logic or local buffering, and has a never-implemented audit mode that was verbally committed to a Tier 1 customer.

Design a production-grade OPC-UA-to-cloud bridge architecture that handles brownfield (existing machines with legacy OPC-UA endpoints) and greenfield (new installations) equipment, supports zero-downtime operation, includes a tamper-evident audit mode, and can be deployed independently per factory.

---

### Explicit Requirements

1. Bridge must self-heal on OPC-UA session drop — automatic reconnection with exponential backoff
2. Local buffer: if cloud connectivity is lost, the bridge buffers up to 4 hours of data locally and drains on reconnect
3. Bridge must be deployable as an independent unit per factory (not shared across factories)
4. Support audit mode: freeze specific sensor stream writes to the audit log view, while allowing live sensor reads to continue to the dashboard
5. Audit log snapshots must be tamper-evident (cryptographic hash chaining or equivalent)
6. Support both brownfield (existing OPC-UA servers) and greenfield (new OPC-UA endpoints)
7. The bridge must emit health metrics and expose a liveness/readiness probe for the factory gateway to monitor

---

### Hidden Requirements

- **Hint**: Re-read Karl's statement about the audit window — "the auditors tell us in advance which production lines they're reviewing." The audit freeze is line-specific, not factory-wide. What does this mean for the data model of the audit log? Can you freeze writes to a subset of sensor IDs without stopping writes to the others?

- **Hint**: Karl mentioned "the factory can't stop running." During an audit freeze, the live dashboard must still update. But the audit log must be frozen. This implies two write paths — one to a live state store, one to an audit log store. What does this mean for the bridge's publish semantics?

- **Hint**: The bridge runs on hardware at the factory edge — likely an industrial PC inside the factory network. The factory network is typically air-gapped from the internet (OT/IT network separation). What does this imply about how the bridge connects to the cloud? Can it use a standard REST API call, or does the architecture require something that traverses the OT/IT boundary?

---

### Constraints

- **OPC-UA polling rate**: Group A sensors every 10ms, Group B every 1 second
- **Local buffer capacity**: 4 hours at full sensor throughput per factory
- **Group A throughput**: 800K msg/sec per factory
- **Group B throughput**: 42K msg/sec per factory
- **OT/IT boundary**: factory floor is on an isolated OT network; cloud connectivity routes through a DMZ
- **Edge hardware**: industrial PC, 32GB RAM, 2TB NVMe SSD, 1Gbps uplink to DMZ
- **Audit mode duration**: up to 4 hours per session, up to 3 sessions per day during quarterly audits
- **Audit mode scope**: subset of sensor IDs (one production line = ~1,200 sensors out of 50,000)
- **Tamper evidence**: FIPS 140-2 equivalent integrity requirement (customer's compliance team language)
- **Deployment**: must deploy per-factory; no shared-bridge model across factories
- **Uptime SLA**: 99.99% availability for the bridge layer (52 minutes downtime/year maximum)

---

### Your Task

Design the OPC-UA-to-cloud bridge architecture. Your design must address:

1. The reliability problem: how does the bridge self-heal, buffer locally, and drain without data loss?
2. The OT/IT boundary: how does factory floor data cross the air gap to reach the cloud?
3. The dual write path: how does the bridge support both live state updates and the frozen audit log simultaneously?
4. Tamper evidence: what cryptographic mechanism makes the audit log tamper-evident?
5. The operational model: how do you deploy, monitor, and update the bridge across 12 factories without a factory visit?

---

### Deliverables

- [ ] Mermaid architecture diagram: OPC-UA endpoint → bridge → OT/IT boundary → cloud ingest (show both live path and audit log path)
- [ ] Bridge internal state machine diagram: normal operation → cloud disconnect → local buffer → drain → normal operation (plus audit mode transitions)
- [ ] Database schema for the local buffer (what is stored during cloud disconnect, with types and indexes)
- [ ] Audit log schema with tamper-evidence design (hash chain column, snapshot versioning)
- [ ] Scaling estimation:
  - Local buffer storage required for 4-hour outage at Group A + Group B throughput
  - Network bandwidth from factory edge to cloud (compressed vs. uncompressed)
  - Show your math
- [ ] Tradeoff analysis:
  - Tradeoff 1: OPC-UA push subscription vs. periodic polling for Group A sensors (latency vs. server load)
  - Tradeoff 2: Local SQLite buffer vs. embedded Kafka vs. flat file journal for the edge buffer
  - Tradeoff 3: Separate bridge process per factory vs. multi-factory bridge with factory namespacing
- [ ] Cost modeling: estimate cost per factory for the bridge-layer infrastructure (edge hardware amortized, cloud ingestion endpoint, monitoring)
- [ ] Capacity planning: how does the bridge design scale as you add Group A sensors from 8,000 to 20,000 per factory (contract option mentioned in the annex)?

### Diagram Format

All architecture diagrams must use Mermaid syntax.
