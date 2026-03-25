# Arc 8: ForgeSense — Manufacturing / Smart Factory IIoT Platform

**Industry**: Manufacturing / Industrial IoT (IIoT)
**Arc Chapters**: Ch. 202–208
**Your Role**: New Hire — Senior Backend Engineer (Strong Senior track)
**GitHub Milestone**: `ForgeSense Arc — Chapters 202–208`

---

## Company Profile

ForgeSense is a Detroit-headquartered industrial IoT platform company serving Tier 1 automotive parts manufacturers. Founded in 2018 by a team of ex-GM and ex-Siemens engineers, ForgeSense builds the software layer that bridges the factory floor — Programmable Logic Controllers, SCADA systems, OPC-UA endpoints — with cloud analytics, digital twin simulations, and predictive maintenance pipelines.

The company currently serves 12 automotive parts suppliers across the US and Germany. Its flagship product, ForgeMind, is a real-time digital twin platform: every machine, sensor, and production line in a customer's factory is mirrored in software, with anomaly detection, failure prediction, and production optimization running continuously against the virtual model.

ForgeSense is not a startup anymore — it has 200 employees and a Series C under its belt — but it has the infrastructure of a startup. Most of the core platform was built by a founding engineering team of 8 people who were brilliant at industrial systems and less experienced with distributed systems at scale. The technical debt is specific: the cloud platform can't reliably handle more than 3 factories simultaneously, and the company just signed contracts to onboard 12 factories in 18 months.

---

## Your First Day

You fly into Detroit on a Sunday. The ForgeSense office is on the 11th floor of a building in Corktown, 10 minutes from the Ford Piquette Avenue Plant — the building where the first Model T was assembled. The irony isn't lost on you.

Your hiring manager, Tariq Hussain, meets you in the lobby. "You come recommended," he says. He doesn't say by whom. Later, you'll find out it was a brief reference from someone at VertexCloud. "We need someone who can think about systems the way a factory floor engineer thinks about machines — every failure mode planned for, every component replaceable, every alarm meaningful." He pauses at the elevator. "We've been building for clever. We need to start building for reliable."

Your onboarding buddy is Svetlana Volkov, a Staff Engineer who joined from Bosch 18 months ago. She has a habit of drawing architecture diagrams on whiteboards with a red marker and circling the parts she doesn't trust. On day one, the whiteboard in the team room has three large red circles on it. "Those," she says, pointing at them, "are what you're going to fix."

---

## Arc Theme

The ForgeSense arc is about the collision between industrial engineering culture and distributed systems thinking. Factory floor systems were designed with different assumptions than cloud systems: determinism over availability, safety over performance, long hardware lifecycles over continuous deployment. Bringing those two worlds together without breaking either one is the central challenge.

The arc also introduces Marcus Webb's second appearance in Phase 3 — not in person, but through a conference talk that a senior engineer references during a design review. Webb spoke publicly about the AgroSense incident from Phase 1. The lesson he drew from it — that data quality failures upstream destroy ML pipeline value downstream — is exactly the lesson ForgeSense needs to learn about its predictive maintenance system.
