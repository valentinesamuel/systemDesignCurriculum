---
**Title**: TeleNova — Chapter 123: Auth at 120 Million — Device Authentication and B2B2C Identity
**Level**: Staff
**Difficulty**: 8
**Tags**: #auth #oauth2 #device-authentication #sim-auth #b2b2c #scim #device-identity #fifth-appearance
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 6 (MeridianHealth auth), Ch. 30 (CloudStack SSO), Ch. 61 (SkyRoute auth), Ch. 94 (SentinelOps ABAC)
**Exercise Type**: System Design
---

### Story Context

**This is the 5th auth system in the curriculum — each appearance has evolved: OIDC (NexaCare) → SAML federation (CloudStack) → adversarial auth (SkyRoute) → RBAC/ABAC (SentinelOps) → device identity at scale (TeleNova)**

---

**Fatima Al-Rashid (VP Engineering) → [you] — Tuesday 14:30**

**Fatima Al-Rashid**: Do you have 30 minutes? There's a client situation.

Her office. A printed email on the table.

**Fatima Al-Rashid**: Deutsche Bahn — German railways — is deploying 50,000 IoT devices on trains. Telemetry sensors, passenger wifi access points, operational control systems. They want to authenticate each device to our network management APIs individually. Right now, our device onboarding portal lets you provision one device at a time through a web UI. Their procurement team has already signed a contract. The implementation team just discovered the one-at-a-time limitation. They need 50,000 devices provisioned in 72 hours. Launch date is fixed — railway schedules don't move.

**[you]**: What does "provision" mean in this context? SIM registration?

**Fatima Al-Rashid**: Not SIM. These are eSIM devices authenticating via client certificates. Each device needs a unique certificate issued by our PKI, registered in our device registry, and assigned to the Deutsche Bahn enterprise account with their specific permission scope. Our provisioning team does this manually. One device takes about 8 minutes.

**[you]**: 50,000 devices × 8 minutes = 400,000 minutes. 6,666 hours. Even with a team of 10 people working 24/7, that's 28 days. They have 72 hours.

**Fatima Al-Rashid**: Correct.

**[you]**: Bulk provisioning via SCIM. We need to build a SCIM endpoint that Deutsche Bahn's directory service can push to in bulk.

**Fatima Al-Rashid**: SCIM is for users. Can you do devices?

**[you]**: SCIM 2.0 has a generic Resource extension model. You can define a Device resource type. Deutsche Bahn's system sends us a structured batch of 50,000 device records. We process them asynchronously, issue certificates in parallel via our HSM, register in the device registry. I'd estimate 72 hours is achievable if we start immediately.

**Fatima Al-Rashid**: *(pause)* You know I worked at NexaCare for two years before this. You fixed the GDPR erasure pipeline there. Dr. Okafor mentioned you when I interviewed.

**[you]**: Small world.

**Fatima Al-Rashid**: *(standing)* You have until Friday to design the bulk provisioning system. The 50,000 device problem is the immediate need. But Alejandro wants a general solution — we have three more enterprise clients asking for similar deployments in Q2.

---

**Architecture Review — Thursday 10:00**

You've drafted the design. Fatima and Alejandro are in the room. You're explaining the B2B2C auth hierarchy:

- **TeleNova** (issuing authority) → **Deutsche Bahn** (enterprise client with delegated admin) → **50,000 devices** (end entities)

**Alejandro Reyes**: How does a device prove its identity at the API level?

**[you]**: Each device has a unique client certificate issued by our intermediate CA. The certificate contains the device ID, the enterprise client ID (Deutsche Bahn), and a capabilities scope encoded in the Subject Alternative Name extension. At the API gateway, we validate the certificate chain and extract the device identity.

**Alejandro Reyes**: What if a certificate is compromised? A device gets stolen from a train?

**[you]**: Certificate revocation via OCSP. Deutsche Bahn's admin portal can revoke a specific device certificate within seconds. Our API gateway queries our OCSP responder on every new connection. Revoked device = immediate rejection.

**Alejandro Reyes**: At 50,000 devices making continuous API calls — what's the OCSP latency?

**[you]**: With OCSP stapling, the gateway caches the OCSP response for the certificate's staple validity period — typically 4 hours. Most connections don't hit the OCSP responder directly. Revocation propagates in at most 4 hours.

**Alejandro Reyes**: Is 4-hour revocation propagation acceptable for a compromised device on a train network?

*(silence)*

**[you]**: No. That's a hidden requirement I need to address.

---

### Problem Statement

TeleNova needs to provision 50,000 Deutsche Bahn IoT devices with unique client certificates in 72 hours, and then design a scalable B2B2C device authentication architecture for future enterprise IoT deployments. Current one-at-a-time provisioning is unusable at this scale. Certificate revocation propagation of 4 hours is unacceptable for compromised device scenarios.

---

### Explicit Requirements

1. Bulk device provisioning: 50,000 devices in ≤ 72 hours via SCIM batch API
2. Each device receives a unique client certificate from TeleNova's PKI
3. B2B2C hierarchy: TeleNova → Enterprise Client → Devices (enterprise admin can manage their own devices)
4. Certificate revocation: compromised device must be blocked within 30 minutes (not 4 hours)
5. SCIM Device resource type: define schema, bulk operation support, asynchronous processing
6. The design must support at least 5 additional enterprise IoT deployments in Q2 without significant engineering work

---

### Hidden Requirements

- **Hint**: Alejandro identified the 4-hour revocation gap. Re-read his question: "Is 4-hour revocation propagation acceptable for a compromised device on a train network?" The answer depends on the device's capabilities. An operational control system device with network access could cause real harm in 4 hours. What's the correct revocation design for near-instant revocation at scale?

- **Hint**: Deutsche Bahn mentioned "passenger wifi access points" in the device types. These devices authenticate end-user passengers through their own SIM-based EAP-AKA. You have a chain: Device Certificate (TeleNova → DB → Device) AND end-user auth (passenger SIM). What happens when a passenger connects to a compromised access point? Your threat model needs to account for this.

---

### Constraints

- 50,000 devices, 72-hour provisioning deadline
- Certificate issuance rate: HSM can sign 100 certificates/minute (limited by HSM hardware)
- 50,000 ÷ 100/min = 500 minutes = ~8.3 hours for serial issuance (parallelizable with multiple HSMs)
- API call volume: 50,000 devices × average 10 API calls/hour = 500,000 API calls/hour
- Certificate validity: 2 years (industry standard for device certs)
- Revocation SLA: compromise detected → device blocked within 30 minutes

---

### Your Task

Design the TeleNova bulk device provisioning and B2B2C device authentication architecture.

---

### Deliverables

- [ ] SCIM Device resource schema: field definitions, required vs optional, bulk operation spec
- [ ] Bulk provisioning pipeline: SCIM input → certificate issuance → device registry → async status tracking
- [ ] HSM parallelism math: how many parallel HSM instances needed to provision 50,000 certs in ≤ 24 hours
- [ ] B2B2C auth hierarchy diagram: TeleNova PKI → Enterprise CA → Device certificate chain
- [ ] Near-instant revocation design: short-lived certificates (4-hour TTL) + re-issuance vs OCSP with push invalidation
- [ ] Enterprise admin portal design: what DB's provisioning team can do (bulk import, individual revoke, expiry alerts)
- [ ] Tradeoff analysis (minimum 3):
  - Client certificate auth vs JWT-based device auth at 500k API calls/hour
  - Short-lived certificates (fast revocation) vs long-lived (operational overhead)
  - SCIM for device provisioning vs custom API — standards compliance vs domain fit
