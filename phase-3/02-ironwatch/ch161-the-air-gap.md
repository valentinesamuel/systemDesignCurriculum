---
**Title**: IronWatch — Chapter 161: The Air Gap
**Level**: Staff
**Difficulty**: 9
**Tags**: #air-gapped #cross-domain-solution #data-diode #network-security #threat-intel #hardware
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 157, Ch. 160, Ch. 162
**Exercise Type**: System Design
---

### Story Context

The data breach in Chapter 160 has a specific upstream cause: encrypted physical media — hard drives delivered by courier — was ingested via a transfer workstation connected to both the SECRET and TS/SCI networks via USB. That USB path was a known risk. The breach in Chapter 160 was a separate issue, but Osei's post-incident review identified the USB transfer process as "the next breach waiting to happen" and moved it to the top of the remediation list.

She schedules a technical review for the following Monday. Present: you, Tuck, a hardware engineer named Sergeant First Class (Ret.) Darnell Wiggs who now works as a civilian systems integrator, and a woman introduced only as "Dr. Lena Bauer, NSA cross-domain solutions team, here in an advisory capacity." Dr. Bauer has a government-issue laptop and says very little. When she does speak, it's precise.

You walk through the current process: partner agency ships a hard drive via cleared courier. Drive arrives in a secure mailroom. Tuck picks it up, takes it to the transfer workstation in SCIF room 2, plugs in the drive, manually copies the files, then carries a separate drive to the TS/SCI enclave and repeats the process. It takes 45-90 minutes per delivery. Deliveries happen twice daily.

**Darnell Wiggs:** "The USB path is a malware vector. Always has been. We had an incident at a previous contract — an agency I can't name — where a compromised drive installed firmware-level malware on the transfer workstation that persisted across reboots. The workstation was connected to both networks. You see where that goes."

**Dr. Bauer:** "A cross-domain solution needs to be hardware-enforced. Not software. Software can be exploited, patched incorrectly, or misconfigured. A hardware data diode is physically incapable of transmitting data in the reverse direction. That's not a policy — it's physics."

You ask about the specific threat intel feed format. Tuck explains: partner agencies send STIX/TAXII formatted intelligence bundles — structured JSON/XML packages. The files are 10-50MB each. Twice daily. The system ingesting them on the SECRET and TS/SCI sides needs to parse the STIX format, validate it, extract indicators (IPs, hashes, domains, TTPs), and insert them into the threat intelligence database.

**Dr. Bauer:** "The validation step is where most CDS designs fail. The receiving side must validate that what came through the data diode is actually a STIX bundle — not an exploit payload disguised as one. Content inspection at the receiving end, before any database write. The diode passes bytes. Your software decides what those bytes mean."

**Tuck:** "And we also need to handle the case where the sending system — the external courier media — is compromised. Stale intelligence is better than a compromised ingestion pipeline."

You sketch the architecture. The core question is: how do you get data from the external courier media to the air-gapped TS/SCI network without any physical USB path that could carry malware, while still supporting automated, twice-daily, structured feed ingestion?

Dr. Bauer speaks again, unprompted: "One more thing. The data diode transfers bytes only in one direction. If you need the ingestion system to send acknowledgments — 'I received this bundle, here is the hash of what I received' — you can't do that through the diode. There is no back-channel. Design for that."

She closes her laptop and stands. "My advisory capacity ends here. Good luck." She leaves the room.

Wiggs: "She's actually very helpful compared to most NSA people."

---

### Problem Statement

IronWatch's current threat intel feed ingestion relies on a physical USB transfer process that is a known malware vector and an operational bottleneck. Two deliveries per day, 45-90 minutes each, manual handling by a single engineer.

You must design the automated cross-domain solution (CDS) architecture to replace this process. The CDS must use hardware-enforced one-way data transfer (data diode) from the delivery system into the SECRET and TS/SCI networks. The design must address: malware prevention at the ingestion boundary, STIX/TAXII feed validation before database writes, elimination of back-channel acknowledgments (the diode is unidirectional), operational monitoring of feed health without violating the one-way constraint, and software update distribution to air-gapped systems.

---

### Explicit Requirements

1. Hardware data diode (not software-enforced) between the delivery network and SECRET / TS/SCI networks
2. No USB path between networks in the production ingestion pipeline
3. STIX/TAXII bundle format validation at the receiving end before any database write
4. Feed ingestion: 2 deliveries/day, bundles of 10-50MB each, structured STIX/TAXII JSON/XML
5. No back-channel: the receiving system cannot send acknowledgments through the diode
6. Air-gapped TS/SCI: zero outbound connectivity from TS/SCI to any network
7. Software updates to air-gapped systems (OS patches, application updates) must follow the same one-way path as data
8. Feed health monitoring: operations team must be able to detect feed failures without violating the one-way constraint
9. Quarantine zone: content inspection of inbound bundles before SECRET/TS/SCI ingestion

---

### Hidden Requirements

- **Hint**: Re-read Dr. Bauer's statement that "the diode passes bytes — your software decides what those bytes mean." A STIX bundle is a structured format with a known schema. What does "content inspection at the receiving end" mean architecturally — is this a separate service, and what is its blast radius if it's compromised?
- **Hint**: Re-read Tuck's comment about stale intelligence. Feed ingestion failures will happen (corrupt bundles, format changes, network issues). What is the operational playbook when the SECRET feed goes 24 hours without a successful ingestion? How does the operations team detect this without a back-channel from the air-gapped network?
- **Hint**: Re-read Wiggs's incident about firmware-level malware that "persisted across reboots." The transfer workstation in the new architecture — even if it's only connected to the external delivery network — is a malware entry point. What does this imply about the transfer workstation's architecture? Should it be mutable (standard OS) or immutable (firmware-level locked)?
- **Hint**: Software updates must travel the same one-way path as data. What does this mean for the operational cadence? DISA STIG requires OS patches within 30 days of release. How do you maintain a patching cadence for an air-gapped system using only one-way transfer?

---

### Constraints

- **Feed volume**: 2 deliveries/day × 2 networks (SECRET + TS/SCI) = 4 ingestion events/day
- **Bundle size**: 10-50MB per bundle (STIX/TAXII JSON/XML)
- **Ingestion latency SLA**: feed available in threat intel DB within 2 hours of delivery
- **Data diode hardware**: physical hardware solutions (Waterfall Security, Owl Cyber Defense, or similar); procurement time 4-6 weeks
- **Air-gap constraint**: TS/SCI has zero outbound network path
- **Patching requirement**: DISA STIG mandates OS patches within 30 days of release
- **Team**: Wiggs (hardware), Tuck (software), 1 additional engineer
- **Budget signal**: hardware CDS solutions run $50K-$200K per deployment
- **Compliance**: NSA/CSS Policy Manual 9-12 (Cross Domain Solutions), DISA CDS requirements

---

### Your Task

Design the CDS architecture for automated threat intel feed ingestion into SECRET and TS/SCI networks. Address the quarantine/validation zone, software update distribution, and operational monitoring without back-channel.

---

### Deliverables

- [ ] Mermaid architecture diagram: delivery network → quarantine zone → data diode → SECRET ingestion → TS/SCI ingestion, with software update distribution path
- [ ] Database schema: STIX indicator ingestion (IP/hash/domain/TTP records), feed metadata (delivery timestamp, bundle hash, validation status), quarantine log
- [ ] Scaling estimation:
  - Daily ingest volume: 4 events/day × 50MB max = X MB/day; annual storage
  - Indicator extraction rate: estimate from STIX bundle density (X indicators per MB × Y MB/day)
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3):
  - Hardware data diode vs. software-enforced CDS (security guarantee vs. cost and flexibility)
  - Quarantine validation depth (schema only vs. content inspection) and performance impact
  - Immutable transfer workstation vs. standard mutable OS (patching complexity vs. attack surface)
- [ ] Cost modeling: data diode hardware ($50K-$200K), quarantine zone infrastructure, operational overhead ($X/month)
- [ ] Capacity planning: 12-month horizon (feed volume growth, additional partner agencies, software update cadence)
- [ ] Operational runbook: what does the on-call engineer do when a bundle fails validation? How do they detect a feed has been silent for >12 hours without a back-channel?

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
