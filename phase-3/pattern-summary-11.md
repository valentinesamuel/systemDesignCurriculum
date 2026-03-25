# Pattern Summary 11 — Staff Patterns I: Platform Architecture, Cell-Based Design, RFC Process, Air-Gapped Compliance

**Type**: Pattern Summary (no deliverables)
**Covers**: OrbitCore arc and IronWatch arc chapters
**Related Chapters**: Ch. 193–212 (OrbitCore and IronWatch arcs)

---

### How This Arrived

You're in between projects — three days between finishing the DeepOcean engagement and starting at PharmaSync — when a Slack notification appears. Not PagerDuty. Not a calendar invite. Marcus Webb.

You haven't heard from him in six weeks. The last time you saw his name in Slack was a one-line DM before the Meridian Exchange checkpoint: "You already see it before the postmortem." That was it. No explanation. You thanked him and he didn't reply.

---

**Slack DM — Marcus Webb → you**

```
Marcus Webb [2:14 PM]
Found something in my notes from 2019. Thought you'd appreciate it now.
Not then. Now.
```

*[Attachment: webb_arch_notes_2019_annotated.pdf — 4 pages, handwritten on legal pad, photographed]*

```
Marcus Webb [2:15 PM]
Don't overthink it. It's not a framework. It's just what I kept seeing.
```

---

The attachment is four pages of legal pad, photographed with an iPhone. The handwriting is angular, precise. Some sections have been annotated in red pen — clearly added later, possibly recently. At the top of page one, in red: *"These still apply. Especially #3 and #6. — M.W., 2024"*

What follows is a transcription of the document, with his annotations preserved.

---

### Webb's 2019 Notes — Transcribed

**On large-scale distributed systems, patterns I keep seeing:**

---

**Pattern 1: Cell-based architecture — blast radius reduction**

*Original (2019)*: The instinct when a system grows is to scale vertically, then horizontally across a shared cluster. The failure mode: one bad actor, one bad deploy, one traffic spike takes out the entire cluster. Solution that actually works: cells. Each cell is a complete, self-contained replica of the system serving a subset of customers. Cells share nothing at runtime — separate databases, separate queues, separate compute. Failure in Cell 4 does not affect Cell 7.

The engineering cost is real: you now have N cells to deploy, monitor, and operate. The business benefit is also real: your blast radius is 1/N of your total customer base. For enterprise SaaS where one customer going down triggers SLA penalties and a board conversation, that math is favorable.

*Red annotation (2024)*: "This is not just for SaaS. I saw this same pattern in aviation (seat inventory cells per flight), in insurance (claims processing cells per state), and in satellite operations (cells per orbital plane). The specific technology doesn't matter. The isolation principle is universal."

---

**Pattern 2: Data sovereignty at the infrastructure layer — routing based on jurisdiction, not just latency**

*Original (2019)*: Most routing decisions are made on latency: send traffic to the nearest healthy node. This breaks when the data in the request is legally restricted to a specific geography. EU personal data must not leave the EU. Defense-classified data must not touch unclassified networks. Healthcare data in some jurisdictions cannot be processed on foreign soil.

The solution is not a policy document. The solution is infrastructure-layer routing that makes the wrong path physically impossible — or at minimum, auditably detectable. Data plane routing rules that key on customer identifier (which carries jurisdiction metadata) rather than pure latency.

*Red annotation (2024)*: "I've seen engineers argue 'we can enforce this in application code.' They're always wrong. Application code has bugs. Infrastructure routing rules are harder to accidentally bypass. Put jurisdiction enforcement as close to the wire as possible."

---

**Pattern 3: Latency-aware routing with dynamic topology — when the network is not static**

*Original (2019)*: Most routing tables are static: here are the nodes, here are their addresses, route to the nearest one. This breaks when nodes move (satellites), when nodes appear and disappear on a schedule (spot instances), or when the latency between nodes is not fixed (cross-region under congestion). The solution: treat the routing table as a live data structure. Compute optimal routes dynamically from real-time topology data. This is expensive to build but the only correct solution when the network topology is genuinely dynamic.

*Red annotation (2024)*: "The satellite case is the extreme version of this. Orbital mechanics is just a routing constraint with a physics equation instead of a ping latency. Same pattern. Once you see it, you can't unsee it."

---

**Pattern 4: Air-gapped system design — what fundamentally changes**

*Original (2019)*: An air-gapped system is not just a system with no internet. It's a system where the assumptions of modern distributed systems engineering break. No NTP for clock synchronization. No certificate revocation checking. No external package registries. No SaaS monitoring tools. No paging a vendor for help. Everything the system needs to operate must be present inside the gap.

Implications: software updates become a physical operation (USB, tape, CD, removable drive through a physical security checkpoint). Cryptographic certificates have fixed expiration dates with no renewal path unless you operate your own PKI. Dependency on external APIs is not a bad practice — it's impossible.

*Red annotation (2024)*: "The hardest part of air-gapped design is not the technical constraints. It's the organizational discipline required to keep a parallel software supply chain running. The people who understand the supply chain matter as much as the engineers who build the system."

---

**Pattern 5: Zero-trust in high-threat environments — insider threat model differs**

*Original (2019)*: Commercial SaaS zero-trust is primarily about: don't trust the network perimeter (VPN is not security). The threat model is external attackers who have gotten in. In high-threat environments — defense, intelligence, critical infrastructure — the primary threat model is the trusted insider. The contractor with the right badge. The engineer with legitimate credentials doing something they shouldn't. Zero-trust for insider threats means: every action is logged, every access is justified, access rights are time-bounded, and humans cannot self-approve elevated access.

*Red annotation (2024)*: "The architecture difference: commercial zero-trust is about authentication. High-threat zero-trust is about authorization and audit. Those require different systems."

---

**Pattern 6: Audit log cryptographic chaining — tamper-evidence via hash chaining**

*Original (2019)*: Append-only is necessary but not sufficient for a tamper-evident audit log. Append-only means you can't delete records (assuming you enforce it). But you can still: insert records in the past, modify existing records without changing their ID, or silently drop records from the middle of a sequence. Cryptographic hash chaining prevents this: each log entry includes the hash of the previous entry. Any modification to any past entry breaks the chain from that point forward. Combined with WORM storage (write-once-read-many), you get an audit log that is both append-only and cryptographically verifiable.

*Red annotation (2024)*: "I've seen this pattern required in three separate legal proceedings. The legal team always asks: 'Can you prove this log hasn't been modified?' With hash chaining: yes. Without it: no, and you're in discovery hell."

---

**Pattern 7: Cross-domain solutions (CDS) — hardware-enforced one-way data transfer**

*Original (2019)*: In environments where data flows from a high-classification domain to a low-classification domain (or vice versa), software-only controls are considered insufficient. The attack surface for software is too large. Hardware data diodes — physical devices that allow electrical signals to flow in only one direction — are used to enforce one-way data transfer at the hardware level. The data cannot flow backward because there is literally no return path at the hardware layer.

Engineering implication: all protocols assume bidirectional communication (TCP, HTTP, TLS handshakes). Data diodes break this assumption. You must use UDP-based one-way streaming protocols, or application-layer acknowledgment through a separate approved channel. This is exotic territory and most engineers never encounter it. But when you do, nothing in your toolkit works without modification.

*Red annotation (2024)*: "If you're ever told 'we use a data diode here,' the correct response is: 'walk me through the protocol design from scratch, because everything you learned about network programming doesn't apply.'"

---

**Pattern 8: RFC writing and stakeholder management — the politics of technical decisions**

*Original (2019)*: The RFC process exists to create a record of technical decisions, solicit feedback, and build consensus. In practice, it also does something more important: it surfaces political resistance early, before implementation begins. A rejected RFC is not a failure. It is information. It tells you whose objections you did not address, whose concerns you did not understand, and what the real constraints are that no one wrote down.

The Staff Engineer's skill is not writing a technically perfect RFC. It is writing an RFC that the right people will approve. Those are often different documents.

*Red annotation (2024)*: "The first version of your RFC explains what the system does. The second version explains what the business loses if you don't build it. The third version — which you should never need to write — names the people whose roadmaps get simpler if you do. Most engineers stop at version one."

---

### Patterns That Recur Across Industries

Looking across the OrbitCore and IronWatch arcs, three patterns appeared in every company in different forms:

**Blast radius management** appeared as: cell-based architecture (OrbitCore), bulkhead isolation (IronWatch), state-partitioned data processing (ShieldMutual), and vessel-by-vessel compliance isolation (DeepOcean). The topology changes. The principle — limit how much of your system can be damaged by a single failure — does not.

**Jurisdictional authority over infrastructure** appeared as: orbital slot licensing (OrbitCore), ITAR/EAR export control (IronWatch), state insurance department examinations (ShieldMutual), flag state/port state/EU tri-jurisdictional compliance (DeepOcean). In every case, a legal authority external to the company had the power to define where data could live and who could see it. In every case, enforcing that constraint in application code was insufficient.

**Tamper-evident audit logs** appeared as: cryptographic hash chaining (IronWatch), court-admissible privilege log provenance (LexCore), corrective action plan submission records (DeepOcean), claims processing control failure records (ShieldMutual). The requirement is the same: a third party — regulator, court, auditor — must be able to verify that the record has not been altered. The implementation varies, but hash chaining + WORM storage is the recurring answer.

---

### Three Reflection Questions

These questions have no answers in this document. They are for you.

**1.** Cell-based architecture reduces blast radius by creating isolation boundaries. But isolation has a cost: operational complexity multiplies by the number of cells, debugging cross-cell issues requires distributed tracing that spans artificially separate systems, and feature deployment requires coordinating across all cells. **When does cell-based architecture add more complexity than it removes?** At what company size, system complexity, or customer SLA level does the cost-benefit calculation flip?

**2.** Data sovereignty sounds like an infrastructure routing problem: send EU customer data to EU nodes. But physical infrastructure — undersea cables, satellite ground stations, CDN PoPs — crosses borders constantly. A packet from a Frankfurt data center to a Paris data center may route through a London exchange point. A satellite ground station uplink may be legally located in a third country. **How do you design for data sovereignty in a world where physical infrastructure crosses borders regardless of your intent?** What is the difference between "best effort sovereignty" and "legally enforceable sovereignty," and which does your architecture actually provide?

**3.** The IronWatch audit log had a specific constraint: it had to be verifiable by an external auditor, but the platform itself — the system being audited — could not be trusted as the sole source of truth. The auditor needed to be able to verify the log independently of the platform operator. **When does the auditor need to be independent of the system being audited?** What categories of system — by industry, by risk level, by relationship to the auditee — require architectural independence between the audit function and the system being audited? And what does "architectural independence" actually require at the infrastructure level?
