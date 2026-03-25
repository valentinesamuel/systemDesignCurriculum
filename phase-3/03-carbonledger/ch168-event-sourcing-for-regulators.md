---
**Title**: CarbonLedger — Chapter 168: Event Sourcing for Regulators
**Level**: Staff
**Difficulty**: 8
**Tags**: #event-sourcing #database #compliance #distributed #audit
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 165 (Immutable Registry), Ch. 52 (OmniLogix event sourcing intro), Ch. 76 (Stratum bi-temporal modeling), Ch. 119 (VenueFlow CQRS)
**Exercise Type**: System Design
---

### Story Context

**Incident Bridge — #incidents, Monday 10:04 AM**

**PagerDuty Bot**: [P2] UN Audit Request — Credit History Reconstruction Required
Assigned: @you
Description: UNFCCC audit team has issued a formal data request under Article 6.6(c). They require the complete transaction history of carbon credit serial number CR-2024-0000012847. Response deadline: 4 hours from now (14:04 GMT). SLA breach triggers contract penalty of $50,000.

---

**#incidents — 10:07 AM**

**You**: what does the current system have for CR-2024-0000012847?

**Kwesi Ampah**: checking... okay. so. the current state table shows:
- Credit ID: CR-2024-0000012847
- Status: RETIRED
- Current holder: Meridian Carbon Fund
- Retirement date: 2024-09-03
- Project: Amazon Cerrado Restoration Project, Block 7

**You**: what's the history? who held it before Meridian?

**Kwesi Ampah**: ...that's the problem. the history table only goes back 90 days. there's a cron job that "archives" older records to S3. i'm looking at the S3 bucket now.

**Kwesi Ampah**: there are 47 CSV files in this bucket. no index. no schema documentation. the oldest one is labeled "archive_2023_q1_FINAL_v3_ACTUALLY_FINAL.csv.gz"

**You**:

**Kwesi Ampah**: yeah.

---

**#incidents — 10:23 AM**

**You**: i found something. in the 2023 Q1 archive. CR-2024-0000012847 appears... wait. this serial number is in the 2023 Q1 archive? the credit was issued in 2024.

**Kwesi Ampah**: ...it was reissued. look for CR-2023-0000012847. i bet that existed, got cancelled, and then the sequence counter was reused when we upgraded the issuance system in early 2024.

**You**: the sequence counter was reused?!

**Kwesi Ampah**: nobody documented that. i found a Jira ticket from February 2024 titled "Fix issuance counter after DB migration" and the resolution is "counter reset to starting value." so yes. we have two credits with effectively the same serial number, one from 2023, one from 2024, and the archive doesn't distinguish between them cleanly.

**You**: Kwesi, the UN audit team is going to ask for the full chain of custody. what am I telling them in 4 hours?

**Kwesi Ampah**: ...the truth, which is that we can reconstruct it but it's going to take manual work across 47 CSV files and a PostgreSQL database and we need the afternoon.

---

**Email — 10:41 AM**

**From**: Dr. Amara Ndiaye, UNFCCC Supervisory Body
**To**: compliance@carbonledger.io; You
**Subject**: Formal Audit Request — CR-2024-0000012847

This is a formal request under the CarbonLedger Article 6 Registry Agreement, Section 12.3 (Audit Rights).

Please provide the complete chain of custody for carbon credit serial number CR-2024-0000012847, including:
1. Date and details of original issuance
2. All transfers (with dates, transferor, transferee, consideration if applicable)
3. Any corrections, cancellations, or status changes
4. The retirement event and proof of retirement
5. Verification that this credit has NOT been used to satisfy NDC commitments in any country other than the stated retirement jurisdiction

Please note that under our audit framework, we will also ask for the **system-level evidence** that no retroactive modification of this record has occurred. A read-only database export is not sufficient — we require either a Merkle proof or a cryptographic hash chain demonstrating record integrity.

Response required by 14:04 GMT today.

Dr. Amara Ndiaye

---

**Slack DM — You → Priya Nair, 10:53 AM**

**You**: we have a problem

**Priya Nair**: how bad

**You**: UNFCCC audit wants a Merkle proof or hash chain for a credit that our current system cannot reconstruct because history was archived to 47 unlabeled CSVs

**Priya Nair**: ah. so the "immutable registry" we sold them is mutable PostgreSQL with a CSV archive strategy

**You**: yep

**Priya Nair**: okay. two problems. today: manually reconstruct the chain for this one credit, put together whatever you can, communicate proactively with the auditor. that's damage control.

**Priya Nair**: tomorrow: design event sourcing properly. not just "append rows to a table." i mean real event sourcing where the events ARE the truth and current state is just a projection. where you can replay any credit's history from event 1. and where the hash chain is built into the write path, not bolted on later.

**You**: the UN requires 5-year lookback. we need to replay any credit in under 5 minutes.

**Priya Nair**: bi-temporal too. valid time vs. transaction time. the auditors will ask: "what did your system *believe* about this credit at 3pm on June 15th?" that's transaction time. "when was this credit *actually valid*?" that's valid time. you need both.

---

**1:1 — You and Yara, 11:30 AM**

**Yara**: "Can we send them something by 2pm?"

**You**: "We can reconstruct the history manually for this one credit. But we can't provide a Merkle proof because we don't have one."

**Yara**: "What do we tell them?"

**You**: "The truth. Our current architecture doesn't support cryptographic audit trails. We're designing the replacement now. For this specific credit, we can provide a complete manual reconstruction with our best attestation that no modification occurred."

**Yara**: "Will that satisfy them?"

**You**: "No. But it's better than lying. And it tells them exactly what we're fixing."

**Yara** [after a long pause]: "Send the manual reconstruction. Then write the design for the real system. I want it ready for internal review by end of week."

---

### Problem Statement

CarbonLedger's current architecture cannot satisfy a legitimate regulatory audit request: it cannot reconstruct the complete chain of custody for a carbon credit, cannot provide cryptographic proof that records were not retroactively modified, and has an archiving strategy that makes history reconstruction manual and error-prone. The system must be redesigned around event sourcing where events are the source of truth, current state is a derived projection, and cryptographic integrity is structural — not optional.

Additionally, the system must support bi-temporal modeling (valid time and transaction time) so that regulators can ask "what did your system believe about this credit at a specific moment in time" and receive a deterministic, auditable answer.

### Explicit Requirements

1. All credit lifecycle events (ISSUED, TRANSFERRED, RETIRED, CANCELLED, CORRECTED) must be stored as immutable events in an append-only event store
2. Current state of any credit must be derivable by replaying its event history from the beginning
3. Event history replay for any single credit must complete in < 5 minutes (5-year history)
4. Bi-temporal modeling: each event must record both valid_time (when the fact was true in the real world) and transaction_time (when it was recorded in the system)
5. Cryptographic integrity: each event must be hashed, and each event must include the hash of the previous event for this credit — forming a per-credit hash chain
6. The UN Supervisory Body must be able to verify the hash chain independently — without CarbonLedger cooperation
7. Cross-registry duplicate detection: when a credit is issued, the system must check whether any prior event references the same serial number from a prior issuance period

### Hidden Requirements

- **Hint**: Re-read Dr. Ndiaye's audit request carefully. She asks for "proof that this credit has NOT been used to satisfy NDC commitments in any country other than the stated retirement jurisdiction." This is a negative proof requirement — the system must not only record what DID happen, but must make it possible to prove what DID NOT happen. What does this require architecturally?
- **Hint**: Kwesi discovered that the sequence counter was reset after a DB migration, creating two credits with the same effective serial number from different years. A serial number alone is not a sufficient unique identifier. What is the globally unique identifier for an event sourcing system? (Hint: think about aggregate ID + version number, not just serial number.)
- **Hint**: Priya mentioned "what did your system *believe* about this credit at 3pm on June 15th?" This is a point-in-time query on the transaction time axis. What does this require of your query interface? The read model must support AS OF queries — reconstruct state as of any past transaction timestamp.
- **Hint**: Dr. Ndiaye said "a read-only database export is not sufficient." Why not? Because a database export can be altered before export. The Merkle proof must be independently verifiable from data that CarbonLedger cannot retroactively change. Where does that data live?

### Constraints

- **Event volume**: 10M events/day at scale; 5-year retention = ~18 billion events total
- **Per-credit replay**: < 5 minutes for any credit with up to 50 events in its lifecycle
- **Bi-temporal queries**: point-in-time state reconstruction must be < 10 seconds for any credit, any timestamp
- **Hash chain**: SHA-256 per event, chain must be verifiable in O(n) time where n = number of events for that credit
- **Storage**: Each event ~2KB; 10M/day × 365 × 5 years × 2KB = ~36TB over 5 years
- **Cold storage tiering**: Events older than 90 days → S3 Glacier; hot events in PostgreSQL; warm in S3 Standard
- **Independent verification**: The hash chain anchors must be published to a public, immutable location (options: IPFS, public blockchain, periodic government attestation)
- **Team**: 4 backend engineers, 2 weeks to production-ready event store
- **Compliance**: 21 CFR Part 11 analog for carbon registries; records must be tamper-evident

### Your Task

Design the event sourcing architecture for CarbonLedger's carbon credit registry, including the event store schema, hash chain mechanism, bi-temporal query model, and the independent verification path that satisfies regulatory audit requirements.

### Deliverables

- [ ] Mermaid architecture diagram: event store write path, state projection service, bi-temporal read model, hash chain anchor publication, and audit query interface
- [ ] Event store schema: `credit_events` table (with all columns including hash fields, valid_time, transaction_time, aggregate version), `credit_projections` read model table, `hash_chain_anchors` table
- [ ] Event catalog: define all 6-8 event types (CREDIT_ISSUED, CREDIT_TRANSFERRED, CREDIT_RETIRED, CREDIT_CANCELLED, CORRECTION_APPLIED, CORRESPONDING_ADJUSTMENT_RECORDED, etc.) with their required fields
- [ ] Hash chain design: exactly how is the per-credit hash computed? What fields are included? What happens at chain genesis (first event for a credit)?
- [ ] Bi-temporal query design: write the SQL (or pseudocode) for "give me the state of credit X as of transaction_time T, where valid_time <= V"
- [ ] Scaling estimation: at 10M events/day, what is the write throughput? How many partitions do you need in the event store? What is the storage cost at 5 years?
- [ ] Tradeoff analysis (minimum 3): per-credit hash chain vs. global Merkle tree; event sourcing vs. change data capture; on-chain anchor (blockchain) vs. periodic government notarization
- [ ] Independent verification protocol: step-by-step description of how a regulator (without CarbonLedger access) can verify that credit CR-2024-0000012847 has not been tampered with

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
