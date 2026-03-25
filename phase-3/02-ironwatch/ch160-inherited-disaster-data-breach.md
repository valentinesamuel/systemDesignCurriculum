---
**Title**: IronWatch — Chapter 160: Inherited Disaster — The Data That Crossed the Line
**Level**: Staff
**Difficulty**: 10
**Tags**: #incident-response #data-breach #audit #forensics #compliance #inherited-disaster
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 157, Ch. 162, Ch. 163
**Exercise Type**: Incident Response
---

### Story Context

It is 6:47 AM on a Thursday. Your phone — personal, UNCLAS — rings with a number you don't recognize. You answer.

"This is Osei. I'm sorry to call before hours." She does not sound sorry. She sounds the way people sound when they've been awake since 3 AM. "We found something last night. I need you in the SCIF by 8."

You arrive at 7:52. Tuck is already there. He has the look of someone who has been staring at log files for hours — which is exactly what he has been doing. On the table is a printed stack of network flow logs, four inches thick, with certain lines highlighted in yellow.

"Twelve days ago," Tuck begins, "a scheduled job on the SECRET network copied a batch of threat indicator files to a staging directory for processing. Standard procedure. But the staging directory had a misconfigured mount point. It was mapped to a network share that, under certain conditions, is also accessible from the UNCLAS tier."

You feel the temperature in the room drop.

"We don't know the full scope. We don't know how long the mount point was misconfigured — the config file shows a modification timestamp but we can't confirm it wasn't tampered with. We have one confirmed case of a file being read from that share by an UNCLAS-tier analyst workstation. We believe the analyst — user record shows them as 'Delta-7' in the access logs, an anonymized identifier — did not know what they were reading. But we cannot confirm that."

Osei: "Our regulatory notification window is 72 hours from when we confirmed the breach. We confirmed it at 0215 this morning. The clock is running."

You look at the logs. The timestamps are inconsistent — some are UTC, some appear to be local time for the server's configured timezone. You note this.

---

**[IRONCOMM — #incidents — SECRET tier — LIVE BRIDGE]**

**08:09 Tuck:** Scope: Confirmed 1 UNCLAS workstation accessed the staging share. Unconfirmed whether additional access occurred. Log retention on the UNCLAS share access controller is 30 days only — we may have lost earlier data.

**08:11 Osei:** That 30-day retention is a systemic failure we're addressing. For now — what can we reconstruct?

**08:13 Tuck:** Network flow logs on the SECRET-side router go back 90 days. We can cross-reference with UNCLAS endpoint EDR telemetry if the UNCLAS team pulls it. Problem: UNCLAS EDR is managed by a different contractor. Getting their data requires a procurement action. Lead time: 2-3 business days.

**08:14 [You]:** We need to preserve forensic integrity on everything we have right now before we start the reconstruction. The staging directory contents — do not modify, do not delete, do not run new scans that write to adjacent directories. Mount it read-only NOW.

**08:15 Tuck:** Done.

**08:17 [You]:** Second: Delta-7's workstation needs to be isolated from the UNCLAS network immediately. Not wiped. Isolated. Forensic copy before any investigation. Chain of custody documentation starts now.

**08:18 Osei:** Delta-7's supervisor has been notified. They're cooperating. No indication of malicious intent — Delta-7 appears to have opened one file and closed it, then filed an IT ticket saying "I think I got the wrong folder." The ticket is from 12 days ago. The ticket sat in the helpdesk queue unassigned.

**08:21 [You]:** The helpdesk ticket is evidence. Preserve it. It establishes Delta-7's intent and the 12-day institutional failure to respond.

**08:22 Osei:** Understood. What do we not know that we need to know before the regulatory notification goes out?

**08:24 [You]:** Three things: (1) Was the misconfigured mount point the only path? We need to audit all share configurations on the SECRET tier for similar patterns. (2) How many files were in that staging directory and what was their classification level? (3) Has the UNCLAS workstation that accessed the share been connected to any external devices or networks since the access event 12 days ago?

**08:26 Tuck:** Working on all three. (3) is going to take time.

**08:27 Osei:** The regulatory notification to DCSA goes at 11:15 regardless of what we know. We report what we know, what we don't know, and what we're doing to find out. That's the protocol.

---

You look up from the IRONCOMM log. Three hours. You need to reconstruct an audit trail from a system that wasn't designed for forensic investigation, using log data with inconsistent timezone configuration, for an incident that may have started before your retention window.

You've seen inherited disasters before. At Stratum Systems (Ch. 64), you inherited a route optimizer that was silently wrong for months. At SentinelOps, you rebuilt a threat detection pipeline after finding the previous engineer had disabled half the alert rules to "reduce noise." This is different. The stakes are federal.

The previous architect who left six months ago built the staging directory system. Their documentation is on a USB drive Tuck hands you. The USB drive has 47 files. Forty-three of them are blank Word documents.

---

### Problem Statement

A classified data exposure has been discovered at IronWatch: SECRET-tier threat indicator data was accessible from the UNCLAS network tier due to a misconfigured network share mount point of unknown duration. One confirmed access event has been identified; the full scope is unknown. The 72-hour regulatory notification window is running.

You must simultaneously: (1) design the forensic investigation and audit reconstruction architecture to establish scope and timeline of the breach, (2) contain the exposure, and (3) design the audit architecture that would have detected and prevented this — a system IronWatch does not currently have.

---

### Explicit Requirements

1. Forensic preservation: all potentially relevant data must be preserved with chain-of-custody documentation before any investigation activity modifies it
2. Scope determination: identify all files accessible via the misconfigured path, their classification levels, and all potential access events within available log retention windows
3. Timeline reconstruction: establish when the misconfiguration was introduced, using all available evidence including file system metadata, network flow logs, and EDR telemetry
4. 72-hour regulatory notification: notification to DCSA must be accurate about what is known vs. unknown
5. Containment: the misconfigured path must be closed; audit all SECRET-tier share configurations for similar patterns
6. Post-incident: design the monitoring architecture that would have detected this in real-time (not 12 days later)
7. Log retention: redesign retention policy — 30-day UNCLAS access logs are insufficient; establish minimum retention requirements

---

### Hidden Requirements

- **Hint**: Re-read the timestamp inconsistency note. Log timestamps in different timezones are not just an inconvenience — they affect the forensic timeline. What does this imply about the log collection architecture? Is a centralized log aggregator with normalized timestamps a forensic requirement, not just an operational convenience?
- **Hint**: Delta-7 filed a helpdesk ticket 12 days ago that sat unassigned. The ticket is evidence. What does this imply about the helpdesk system's role in the incident response architecture? Should anomalous access events auto-correlate with helpdesk tickets?
- **Hint**: Re-read Osei's 11:15 notification deadline: "We report what we know, what we don't know, and what we're doing to find out." The structure of a regulatory notification is itself a system design problem. What data model does a compliant notification require, and how must it be generated from audit data?
- **Hint**: Tuck says the staging directory configuration file shows a modification timestamp "but we can't confirm it wasn't tampered with." What does this imply about the trustworthiness of file system metadata as forensic evidence? What alternative evidence sources can corroborate or refute the timestamp?

---

### Constraints

- **Incident timeline**: breach confirmed at 02:15; notification deadline at 11:15 same day (9 hours)
- **Available log data**:
  - SECRET-tier router flow logs: 90-day retention
  - UNCLAS share access controller logs: 30-day retention (may have lost earlier data)
  - UNCLAS EDR telemetry: 2-3 business days to obtain from third-party contractor
  - File system metadata: available but potentially tampered
- **Scope of exposure**: 1 confirmed access; N unknown accesses
- **Analysts affected**: Delta-7 (UNCLAS) confirmed; others unknown
- **Regulatory requirement**: DCSA breach notification, DCID 6/4 compliance
- **Systems must stay operational**: classification tier separation must be maintained while investigation proceeds
- **Team available**: you + Tuck + 2 infrastructure engineers; UNCLAS contractor separately

---

### Your Task

Produce the incident response architecture and the audit system redesign. The incident response architecture must address forensic preservation, scope determination, timeline reconstruction, and regulatory notification. The audit system redesign must address the monitoring gap that allowed a 12-day detection delay.

---

### Deliverables

- [ ] Mermaid incident response flow diagram: preservation → investigation → containment → notification → remediation
- [ ] Forensic audit reconstruction schema: how to correlate network flow logs, access controller logs, EDR telemetry, file system metadata across inconsistent timestamps
- [ ] Scaling estimation:
  - Log volume: SECRET-tier network flow logs at X events/sec × 90 days = total events to process
  - EDR telemetry: 200 UNCLAS workstations × Y events/sec = total events
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3 explicit):
  - Forensic preservation vs. containment speed (which comes first and why)
  - Centralized log aggregation vs. per-tier log isolation (detection capability vs. security boundary)
  - Real-time cross-tier anomaly detection vs. air-gap boundary enforcement (operational tension)
- [ ] Real-time monitoring architecture: what must be deployed to detect future cross-tier data leakage within minutes, not 12 days
- [ ] Cost modeling: centralized log aggregation infrastructure, WORM storage for 7-year retention ($X/month estimate)
- [ ] Regulatory notification document template: structure of a compliant DCSA breach notification including known/unknown/action fields

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
