---
**Title**: LexCore — Chapter 175: The RFC That Succeeded
**Level**: Staff
**Difficulty**: 10
**Tags**: #RFC #client-side-encryption #zero-knowledge #E2E #legal-hold #key-escrow #compliance
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 174, Ch. 171, Ch. 172
**Exercise Type**: System Design (MANDATORY STORY BEAT PART 2)
---

### Story Context

You spent the two weeks after the rejection reading. You read Zoom's E2EE architecture whitepaper. You read the Signal Protocol documentation. You read three law review articles on cloud computing and attorney-client privilege. You read NIST SP 800-57 Part 1 on key management. You called Yasmine at 10pm on a Wednesday and talked through the watermarking problem for forty-five minutes. You did not sleep enough.

The core insight crystallized somewhere around day four: the only way to make LexCore *technically incapable* of reading deal room documents is to ensure that the encryption and decryption happen exclusively on the client device — in the browser, or in a client application — using keys that never leave the client. LexCore's servers see only ciphertext. They can store it, move it, log access events against it, but they cannot read it.

This is client-side encryption (CSE). It's well-established in security architecture. The implementation challenge in a SaaS product is that "the client" is typically a browser, which is an untrusted environment — the browser runs JavaScript that LexCore serves, which means LexCore *could* modify that JavaScript to exfiltrate keys before encryption. This is the trusted client problem. You design around it: keys are generated and managed in a browser extension or dedicated desktop client that users control — not from LexCore-served JavaScript.

The harder problem is legal hold. A court order requires production of documents. Under platform-managed keys (the rejected v1), LexCore could produce documents in response to a subpoena. Under CSE, LexCore holds only ciphertext and cannot comply even if it wants to. This is the desired property *for attorney-client privilege* but it creates a different problem: if a court orders production of deal room documents, who produces them? The answer must be the *law firm* — the attorney/client relationship's holder. LexCore's role is to facilitate firm-level legal hold: the firm can produce their own documents in response to a court order, using their own keys.

Watermarking remains unsolved on Tuesday of the second week. Then Yasmine cracks it: client-side watermarking. The client application applies the watermark locally — using the user's identity from a platform-issued signed token — before rendering or downloading the document. The watermark is verifiable because the signed token is auditable. LexCore never sees the plaintext.

You write the revised RFC on day thirteen. You present it on day fourteen.

---

**[Revised RFC Review — Thursday, 14:00]**

The same room. Rosalind Achebe is present, this time in person.

You present for thirty minutes. The engineering team has questions. Achebe has questions. They are different questions than last time — they are not objections, they are probes for completeness.

**Achebe:** "What happens when a court orders LexCore to produce deal room documents?"

You explain: LexCore produces ciphertext. The court order must go to the law firm. The firm holds the keys. The firm produces plaintext. LexCore provides the ciphertext and the cryptographic audit trail showing access events.

**Achebe:** "What if the firm dissolves before the legal hold is executed?"

You explain the key escrow mechanism: each firm's deal room keys are escrowed with a court-appointed escrow agent (not LexCore) when a legal hold is filed. The escrow agent holds keys in a sealed envelope, opened only by court order. LexCore never touches the escrow.

**Achebe:** "What if LexCore is subpoenaed to modify its client application to exfiltrate keys going forward?"

You explain the trusted client architecture: the client application that manages keys is open-source, code-signed, and cannot be silently modified. A government order requiring a backdoor would need to be public (or LexCore would disclose it, similar to warrant canary architecture).

**Achebe:** "I'm not satisfied with the warrant canary answer. But I accept the rest." She pauses. "RFC approved. Proceed to implementation."

You exhale slowly. Sasha Petrenko gives you a brief nod.

After the meeting, you have a Slack notification on your phone. It's not Marcus Webb. It's a message from Priya Nair — Staff Engineer turned CTO, who hired you at PrismHealth during Phase 2.

**Priya Nair [DM]:** *"Heard you're at LexCore. I went through the exact same CSE design debate at a previous company. One thing you'll hit in implementation: key rotation for active deal rooms. Document [was encrypted with key v1, new version is key v2, mid-deal. How do you handle re-encryption without taking the deal room offline?"*

You look at the RFC you just got approved. Key rotation for active deal rooms is not in it.

You open a new document: "Addendum A: Key Rotation for Active Deal Rooms."

---

### Problem Statement

LexCore's multi-party deal room v2 must use client-side encryption (CSE) to make the platform technically incapable of accessing privileged attorney-client communications. Encryption and decryption must occur exclusively on client devices using keys that never pass through LexCore's servers. The platform stores only ciphertext. Watermarking, activity logging, and legal hold support must be redesigned to work without platform-side access to plaintext.

Additionally, Priya Nair's post-approval note identifies a gap: key rotation for active deal rooms. A deal room may be open for 90-180 days. Key rotation policy requires rotating keys every 90 days. You must design re-encryption of deal room content during an active deal without downtime and without requiring all participants to coordinate simultaneously.

---

### Explicit Requirements

1. Client-side encryption: all document encryption and decryption happens on client devices; LexCore servers never hold or process plaintext or keys
2. Trusted client architecture: key management runs in a code-signed, open-source desktop client or browser extension — not LexCore-served JavaScript
3. Zero-knowledge activity logging: LexCore logs access events (user ID, document ID, timestamp, action) without seeing document content
4. Client-side watermarking: watermarks applied locally on client using platform-issued signed identity tokens; verifiable without server-side content access
5. Legal hold support: firms can initiate a legal hold that escrows their deal room keys with a third-party escrow agent (not LexCore); LexCore facilitates but never holds escrow
6. Court-order response: LexCore can produce ciphertext + access logs in response to a subpoena; plaintext production is the firm's responsibility
7. Key rotation: deal room keys must be rotatable every 90 days without deal room downtime; documents encrypted under old key must be re-encrypted under new key without all participants online simultaneously
8. Multi-firm key distribution: all authorized participants across multiple firms must be able to decrypt documents; key distribution must handle participant additions and removals mid-deal

---

### Hidden Requirements

- **Hint**: Re-read the multi-firm key distribution requirement. If each participant has their own asymmetric key pair, how do you encrypt a document for 120 participants without encrypting it 120 times? What encryption pattern (e.g., envelope encryption with a per-document symmetric key encrypted to each participant's public key) solves this? What does this mean for the key rotation problem?
- **Hint**: Re-read the legal hold escrow design. The escrow agent holds keys in a "sealed envelope." What does this mean cryptographically — is the escrow a copy of the symmetric key, or is it a share of the key (Shamir's Secret Sharing), requiring multiple parties to reconstruct? Which provides better protection against a rogue escrow agent?
- **Hint**: Re-read Achebe's warrant canary objection: she's "not satisfied" but accepts the rest. The RFC is approved but has a known open issue. What is the actual vulnerability in the warrant canary architecture — specifically, can a government order require a software company to silently update a code-signed open-source application to add a key exfiltration backdoor without the company disclosing it? What would a more robust design look like?
- **Hint**: Re-read the re-encryption problem for key rotation. If documents are encrypted with a per-document symmetric key, and that symmetric key is encrypted to each participant's public key (envelope encryption), then key rotation does not require re-encrypting the document content — only re-encrypting the per-document symmetric key to the new key. What does this imply about the rotation cost? What is the remaining challenge?

---

### Constraints

- **Key architecture**: asymmetric per-user key pairs (RSA-4096 or X25519); per-document symmetric keys (AES-256); envelope encryption
- **Client architecture**: desktop client (Electron-based) or browser extension; code-signed; open-source
- **Deal room size**: up to 120 participants × 50,000 documents = 6M participant-document key envelopes per large deal
- **Key rotation cadence**: 90 days
- **Re-encryption during active deal**: must complete within 24 hours; deal room must remain accessible throughout
- **Legal hold escrow**: third-party escrow agent (not LexCore); court-order-only access
- **Latency SLA**: document decryption + render in < 3 seconds p95 (CSE adds latency vs. server-side decryption)
- **Team**: Dmitri + Yasmine + 2 engineers + external security review
- **Timeline**: 8 weeks to production; Pemberton Whitmore deal is watching

---

### Your Task

Design the CSE-based multi-party deal room v2. Produce the full RFC (accepted version), key distribution architecture, client-side watermarking design, legal hold escrow flow, and key rotation procedure.

---

### Deliverables

- [ ] Mermaid architecture diagram: CSE key flow (per-document symmetric key → envelope encryption to each participant → LexCore ciphertext store), client application, legal hold escrow path
- [ ] RFC document structure (accepted version):
  - **Title**: Multi-Party Deal Room Architecture v2 (Client-Side Encryption)
  - **Status**: APPROVED
  - **Changes from v1**: what changed and why
  - **Key Management Architecture**: envelope encryption design
  - **Client Application Architecture**: trusted client, code-signing, open-source
  - **Legal Hold Design**: escrow agent protocol
  - **Court-Order Response Procedure**: step-by-step
  - **Key Rotation Procedure**: per-document re-key process
  - **Open Issues**: warrant canary limitation (acknowledged)
- [ ] Database schema:
  - `deal_room`, `deal_room_participant`, `deal_room_document_metadata` (ciphertext location, NOT content), `document_key_envelope` (per-participant encrypted symmetric key), `activity_log`, `legal_hold_record`
- [ ] Scaling estimation:
  - Key envelope volume: 120 participants × 50,000 documents = 6M envelopes per large deal
  - Key rotation re-encryption: 50,000 documents × 120 participants = 6M envelope re-encryptions at 90-day cadence
  - Storage for envelopes: 6M × ~512 bytes (RSA-encrypted key) = X GB per large deal
  - Show math step by step
- [ ] Tradeoff analysis (minimum 3):
  - Envelope encryption vs. group key (per-participant keys vs. single shared key: forward secrecy vs. operational simplicity)
  - Client-side watermarking vs. server-side (privilege preservation vs. watermark tamperability)
  - Shamir Secret Sharing for escrow vs. full key copy (threshold security vs. recovery complexity)
- [ ] Cost modeling: CSE infrastructure (key envelope storage, client distribution), escrow agent fees, security audit ($X/month)
- [ ] Capacity planning: 12-month horizon (deal room volume growth, participant scale, key rotation infrastructure)
- [ ] Addendum A: Key Rotation procedure — step-by-step, including how documents encrypted under old key are re-keyed while deal room remains active

### Diagram Format

All architecture diagrams: Mermaid syntax (renders in GitHub Issues).
