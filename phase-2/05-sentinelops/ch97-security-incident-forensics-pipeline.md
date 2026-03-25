---
**Title**: SentinelOps — Chapter 97: Security Incident Forensics Pipeline
**Level**: Staff
**Difficulty**: 9
**Tags**: #incident-response #forensics #immutable-storage #worm #evidence-preservation #security #compliance #spike
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 92 (Zero-Trust Architecture), Ch. 94 (Auth/ABAC), Ch. 7 (MeridianHealth HIPAA Audit Trails)
**Exercise Type**: Incident Response (SPIKE — Production DOWN / Live Attack)
---

### Story Context

**Date**: Friday, August 22 — 3:07 AM PDT
**Channel**: #incidents-sev0

> **SPIKE TYPE: Production DOWN — Live Active Attack**
> This chapter begins as a routine on-call page. It is not routine.

---

**Incident Bridge Log — #incidents-sev0**
*SentinelOps Internal Incident Response System*
*Incident ID: INC-2024-0822-001 | Severity: SEV-0 | Status: ACTIVE*

---

**[03:07:14] PagerDuty-Bot:** ALERT FIRED — `anomalous_outbound_data_transfer` on tenant `ftn-0291` (Fortress National Bank). 47GB outbound transfer in 14 minutes to IP range 185.220.x.x (known Tor exit nodes). Auto-escalating to SEV-0.

**[03:07:22] You (On-Call SRE):** Ack'd. Pulling logs now. @Nadia Cho @Yusuf Okafor paging you — need eyes.

**[03:08:45] Nadia Cho:** On the bridge. What do we have?

**[03:08:52] You:** Customer Fortress National Bank. Their data is being exfiltrated. The traffic is OUTBOUND from our threat intel aggregation service, which has read access to FTN's ingested network logs. Whoever is in there has compromised the aggregation service credentials.

**[03:09:31] Yusuf Okafor:** Oh no. Pull the access log for `intel-aggregation-svc` RIGHT NOW. Check what authenticated to that service in the last 6 hours.

**[03:10:18] You:** I see it. There's a service account `svc-aggregator-worker-03` that has been making API calls since 01:23 AM. Normal behavior. But at 02:49 AM there are calls from a different source IP — same service account token but the IP is 45.142.x.x. That's not any of our cluster nodes.

**[03:10:55] Yusuf Okafor:** The token was stolen. They have a valid service account token and they're using it from outside our network. They authenticated successfully through our API gateway. This is not a breach of the customer — WE are the attack vector.

**[03:11:30] Nadia Cho:** Revoke that token NOW.

**[03:11:44] You:** Token revoked. Outbound transfer stopped.

**[03:11:58] Yusuf Okafor:** Good. Now DO NOT touch anything else. Do not delete logs. Do not rotate the compromised service account. Do not restart the affected pods. Freeze everything. We are now in forensic preservation mode.

**[03:12:15] Dr. Amara Kamara:** I'm on the bridge. I just spoke with Fortress National Bank's CISO. They are aware. FBI Cyber Division has been notified by the customer. We will be receiving a call from Special Agent Theresa Huang within the hour. Evidence preservation is now a legal obligation.

**[03:14:40] You:** Understood. What's the SLA?

**[03:15:02] Dr. Kamara:** 72 hours to package and deliver evidence to the FBI. This is not negotiable. It is federal law under 18 U.S.C. § 2703.

**[03:15:44] Yusuf Okafor:** Here's the problem. Our logs are mutable. They live in CloudWatch Logs and Elasticsearch. A sufficiently determined attacker — and this looks like a sophisticated actor — could have already modified logs in the window between compromise and detection. The logs we're looking at right now may not be admissible as evidence.

**[03:16:20] You:** When did we implement log immutability?

**[03:16:35] Yusuf Okafor:** We did not. It was on the roadmap. We were going to do it after the Vault deployment.

*[12-second silence on the bridge.]*

**[03:16:47] Dr. Kamara:** Design it. Tonight. What we cannot change, we must work around — but going forward, this cannot happen again. Yusuf, work with our counsel to determine what we CAN deliver to the FBI from our current mutable store. Make the chain of custody as defensible as we can. I want a written forensic methodology from you by 6AM.

---

**FBI Call — Special Agent Theresa Huang**
*Friday, August 22 — 4:22 AM*
*Participants: Dr. Kamara, Yusuf Okafor, You, FBI SA Huang, FBI digital forensics specialist Marcus Chen*

**SA Huang:** Let me be direct about what we need. We need a complete forensic timeline of all API calls made using the compromised credential from 01:23 AM to 03:11 AM. We need evidence of data staging: what was accessed, what volume was transferred, what destination endpoints were contacted. We need the authentication logs showing the token issuance and all subsequent uses. And we need everything cryptographically signed with a chain of custody that holds up in federal court. Can you provide that?

**Yusuf Okafor:** We can provide the timeline and the content. The chain of custody documentation on our current infrastructure has a weakness I need to be transparent about — our log storage is not currently write-once. We're implementing that tonight. For the evidence we're packaging now, I want to walk you through our methodology and have your forensics specialist review it for admissibility.

**Marcus Chen (FBI):** Walk me through your current log pipeline. Where do the logs originate, what's the transport, and where do they land?

**You:** Application logs are emitted via the OTel Collector to CloudWatch Logs and Elasticsearch. The OTel Collector pipeline is running in our Kubernetes cluster. The Collector has not been restarted since the incident began — that means its internal buffer still has the raw spans from the attack window, and those have not been written to any persistent storage yet. Those span records are our cleanest evidence because they haven't touched a mutable storage system.

**Marcus Chen:** Can you export those spans right now to an isolated, write-protected location before the Collector restarts or purges its buffer?

**You:** Yes. I can write them to S3 with Object Lock enabled in COMPLIANCE mode. That makes them legally immutable and WORM — Write Once Read Many. The bucket policy prevents deletion by anyone, including our root account, for the retention period.

**SA Huang:** Do that immediately. That is your forensic snapshot. Everything from persistent storage we will treat as supporting evidence, not primary evidence, unless you can demonstrate the hash chain of the log files.

---

**Slack DM — You → Yusuf Okafor**
*Friday, August 22 — 6:00 AM*

**You:** Collector spans exported to S3 Object Lock bucket. SHA-256 hash of the export bundle: [hash string]. I've recorded the hash in a separate, air-gapped document and sent it to legal. The methodology doc for FBI is done.

**Yusuf:** Good work tonight. Now. When the sun comes up and we're through the immediate crisis — we need to design the forensics pipeline this should have been running all along. WORM log storage from day one. Immutable audit trail. Chain of custody built into the architecture, not bolted on after a breach.

**You:** I know what we need to build. I've been thinking about it for the last three hours.

**Yusuf:** Write it up. This incident is the design brief.

---

### Problem Statement

The SentinelOps incident on August 22 exposed a fundamental gap: the platform processes security events for hundreds of enterprise customers, but its own internal logs are mutable. An attacker who compromises SentinelOps infrastructure can alter or delete the log evidence of their own actions. This is legally and ethically unacceptable for a managed security provider.

You must design a forensic log pipeline that ensures immutability from point of emission, supports 72-hour evidence packaging for law enforcement, maintains a cryptographic chain of custody, and enables forensic timeline reconstruction from first sign of compromise to containment.

### Explicit Requirements

1. All security-relevant logs (authentication events, authorization decisions, API calls, network connections, data transfer events) written to WORM (Write Once Read Many) storage within 30 seconds of emission
2. S3 Object Lock in COMPLIANCE mode for all forensic log buckets; retention period minimum 365 days; cannot be shortened or deleted by any IAM principal including root
3. Cryptographic hash chain: each log batch includes the SHA-256 hash of the previous batch, creating a tamper-evident chain detectable if any historical record is altered
4. Evidence packaging: within 72 hours of incident declaration, the system must produce a complete forensic bundle including all relevant logs, hash chain verification, and chain of custody documentation
5. Forensic timeline reconstruction: given a compromised service account or credential, the system must reconstruct all API calls, data accesses, and network connections for any time window within the retention period
6. Chain of custody log: every access to forensic evidence (reads, exports, shares) is itself logged immutably
7. Real-time alerting on log pipeline failures: if the immutable log write pathway falls behind by more than 60 seconds, a SEV-1 alert fires
8. Live attack protocol: during an active incident, a "forensic freeze" mode preserves all in-flight log buffers before any remediation actions

### Hidden Requirements

- **Hint**: Re-read Marcus Chen's question: "Where do the logs originate, what's the transport, and where do they land?" The critical insight is that the OTel Collector's in-memory buffer contained the cleanest forensic evidence because it hadn't touched mutable storage yet. The new architecture must guarantee that in-memory buffers are treated as primary evidence and are persisted to WORM storage before any other operation during a forensic freeze.
- **Hint**: Re-read SA Huang's statement: "Everything from persistent storage we will treat as supporting evidence, not primary evidence, unless you can demonstrate the hash chain of the log files." This implies that Elasticsearch (the current primary log store) cannot serve as primary forensic evidence. What is the relationship between the WORM store (primary forensic evidence) and Elasticsearch (operational query layer)? They must be synchronized but one is the source of truth.
- **Hint**: Re-read the 3:16 AM exchange: "When did we implement log immutability?" / "We did not." This means all historical logs in Elasticsearch and CloudWatch are potentially compromised for this incident. The chain of custody design must explicitly address what happens to pre-WORM-implementation logs — they need a different evidentiary classification.
- **Hint**: Re-read Yusuf's 3:16 AM statement: "A sufficiently determined attacker — and this looks like a sophisticated actor — could have already modified logs in the window between compromise and detection." The forensic pipeline itself is a high-value target. The OTel Collector agent running in the compromised cluster could itself be tampered with. How do you ensure that log collectors cannot be silenced or manipulated by an attacker who has compromised the cluster's control plane?

### Constraints

- **Log volume**: ~50GB/day of security-relevant events (authentication, API calls, network flows, authorization decisions)
- **WORM retention**: 365 days minimum; 7 years for events related to known incidents (regulatory requirement)
- **Evidence packaging SLA**: 72 hours from incident declaration
- **Log write latency**: < 30 seconds from emission to WORM persistence
- **Hash chain generation**: every 60 seconds, batch all logs from that window, compute SHA-256 of batch + previous hash, store hash in separate immutable record
- **Compliance**: SOC 2 Type II, 18 U.S.C. § 2703 (federal evidence preservation), potential FINRA requirements for financial sector customers (FTN is a bank)
- **Storage cost**: S3 Object Lock COMPLIANCE mode on 50GB/day × 365 days = ~18TB/year at $0.023/GB-month
- **Forensic freeze RTO**: < 5 minutes from incident declaration to all buffers preserved
- **Chain of custody doc**: must include accessor identity, timestamp, purpose, and data hash for every evidence access event
- **Attacker threat model**: attacker may have cluster admin access; log pipeline must be resilient to node compromise

### Your Task

Design the forensic log pipeline for SentinelOps. Your design must address: the immutable write path from log emission to WORM storage, the hash chain implementation, the evidence packaging workflow, the forensic freeze protocol for live attacks, and the chain of custody audit system. You must also address the architecture's resilience to an attacker who has compromised cluster control plane access.

### Deliverables

- [ ] **Mermaid architecture diagram** — Forensic log pipeline: service application → OTel Collector agent → OTel Collector gateway (with forensic freeze mode) → dual write to (1) WORM S3 bucket (Object Lock, COMPLIANCE mode) and (2) Elasticsearch (operational queries). Show the hash chain computation step and where chain-of-custody access logs are written.
- [ ] **Database schema** — Two tables: (1) forensic_log_batches (batch_id, time_window_start, time_window_end, log_count, sha256_hash, previous_batch_hash, s3_key, created_at, verified_at) for hash chain integrity; (2) evidence_access_log (access_id, incident_id, accessor_id, accessor_role, accessed_batch_ids[], purpose, exported_to, access_timestamp, data_hash) for chain of custody
- [ ] **Scaling estimation** — Show math for: (a) WORM storage cost: 50GB/day × 365 days = TB/year × S3 Object Lock pricing, (b) hash chain computation overhead: 60-second batch windows, average batch size, SHA-256 throughput, (c) evidence bundle size for a 2-hour incident window (how many GB of logs, how long to package and transfer to FBI)
- [ ] **Tradeoff analysis** — Minimum 3 explicit tradeoffs:
  - S3 Object Lock COMPLIANCE mode vs GOVERNANCE mode (COMPLIANCE cannot be overridden by root; GOVERNANCE can)
  - Log pipeline resilience: out-of-cluster log forwarder (attacker cannot silence it) vs in-cluster forwarder (lower latency, but vulnerable to cluster compromise)
  - Real-time WORM writes vs batch WORM writes (latency vs throughput vs S3 API cost)
- [ ] **Forensic freeze runbook** — Step-by-step procedure for activating forensic freeze mode during a live attack: what commands run, what systems pause, what gets preserved first, verification steps. Must execute in < 5 minutes.
- [ ] **Hash chain verification script** — TypeScript pseudocode for a function that verifies the integrity of the hash chain across a given time range: reads batch records, recomputes hashes, detects any gaps or tampering. Include the function signature and core logic flow.
