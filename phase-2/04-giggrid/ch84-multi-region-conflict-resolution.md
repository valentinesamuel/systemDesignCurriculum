---
**Title**: GigGrid — Chapter 84: Multi-Region Conflict Resolution
**Level**: Strong Senior
**Difficulty**: 8
**Tags**: #multi-region #active-active #conflict-resolution #distributed-systems #worker-assignment
**Estimated Time**: 3 hours
**Related Chapters**: Ch. 68, Ch. 85, Ch. 125
**Exercise Type**: System Design / Debugging
---

### Story Context

**Email Chain**

---
**From**: operations@buildready-staffing.co.uk
**To**: enterprise-support@giggrid.io
**Subject**: Double Assignment — 3rd Incident This Week
**Date**: [Week 1 at GigGrid]

To whom it may concern,

This morning at 07:14, two GigGrid workers — ID #GG-884271 and ID #GG-773054 — both reported to our Manchester construction site for the same role (Site Safety Officer, 07:00–19:00). Both had received a confirmed job assignment notification from your platform. Both had traveled to the site.

We paid both workers for the travel cost (as we are legally required to do under UK employment law), despite only needing one. We also now have a staffing shortage for a different role because one of our partner slots was consumed by this duplicate.

This is the third double assignment incident in two weeks. If this continues, we will be reviewing our contract.

BuildReady Staffing, Operations Team

---
**From**: amara.chen@giggrid.io
**To**: you@giggrid.io
**Subject**: FWD: BuildReady complaint — this is your root cause

The reason we have double assignments: active-active replication with LWW conflict resolution. Two workers in two different regions both accepted the same job within 200ms of each other. Both regions thought they were the "winner" because the replication lag was 340ms — longer than the acceptance window.

The LWW logic picked the wrong winner. But by the time the correct state propagated, both workers had been notified.

I've seen 47 of these this month. We're going to have a lawsuit before we have a solution.

— Amara

---

**#engineering — GigGrid**
`Monday 11:30` **you**: I've been looking at the 47 conflict incidents. the pattern is consistent: job acceptance from two workers in different regions, 50-400ms apart, with replication lag exceeding the acceptance gap. LWW resolves to one worker as the "correct" assignee but the other worker is already notified.
`11:32` **amara.chen**: what's wrong with LWW here?
`11:33` **you**: LWW is correct for some data types — like worker profile updates, where "latest update wins" is semantically meaningful. but for job assignment, it's the wrong semantics entirely. the conflict isn't about which update is "later" — it's about the fact that EXACTLY ONE worker should ever be assigned, and the system should prevent two workers from accepting simultaneously, not resolve it afterward.
`11:35` **amara.chen**: prevention rather than resolution
`11:36` **you**: exactly. but prevention requires coordination. and coordination introduces latency. at 30-country scale with 2M active workers, the coordination model matters.
`11:38` **amara.chen**: I've been thinking about this for 3 months. I didn't hire you to tell me what I already know. I hired you to design the solution.
`11:39` **you**: I need 48 hours
`11:39` **amara.chen**: you have 24

---

### Problem Statement

GigGrid's active-active multi-region replication uses last-write-wins (LWW) conflict resolution for job assignment records. This is semantically incorrect for job assignment: the constraint is "exactly one worker per job," not "the latest update wins." When two workers accept the same job in different regions within the replication lag window, both receive confirmation notifications before the conflict is resolved. You need to redesign the conflict resolution strategy for job assignments while maintaining the availability and latency properties of active-active replication.

---

### Explicit Requirements

1. A job must be assigned to exactly one worker — double assignments must be architecturally prevented, not resolved after the fact
2. Single-worker assignment must be achievable within 500ms P99 latency (workers expect near-instant confirmation)
3. The solution must work across 14 active-active regions
4. Workers who lose a race (attempted to accept a job that was taken) must receive a clear "already taken" response, not a silent failure
5. No single region should be a coordination bottleneck for all 14 regions
6. The conflict resolution change must not break profile update and availability flag updates (where LWW is still correct)

---

### Hidden Requirements

- **Hint**: Re-read Amara's framing: "prevention rather than resolution." Prevention requires knowing whether a job has already been accepted before you accept it. But in an active-active system, you can't always query all 14 regions synchronously before accepting — that's too slow. What's the data model change that makes prevention possible without full synchronous coordination?

- **Hint**: The BuildReady complaint mentions "both workers had traveled to the site." The notification was sent before the conflict was resolved. Even if your architecture prevents double-assignment at the data level, you need to change the notification timing. When should the confirmation notification be sent to a worker: immediately on acceptance, or after the assignment is durable across a quorum of regions?

---

### Constraints

- 14 active-active regions
- 2M active workers on any given day
- 50k job dispatches per day = ~35/minute average, up to 2,000/minute during surge periods
- Replication lag between regions: 50–340ms (varies by network path)
- Worker acceptance notification SLA: < 500ms
- LWW must remain for: worker profile updates, availability flag updates, preference settings

---

### Your Task

Design the conflict-free job assignment mechanism for GigGrid's active-active multi-region system. Identify the specific data types where LWW is correct and where it must be replaced, and design the replacement.

---

### Deliverables

- [ ] Conflict analysis: For job assignment, worker profile, and availability flags — classify each as "LWW safe" or "LWW unsafe" with justification
- [ ] Assignment reservation model: Design the "claim → reserve → confirm" state machine that prevents double-assignment
- [ ] Mermaid diagram: Job assignment flow across 14 active-active regions with reservation protocol
- [ ] Notification timing design: When confirmation notification is sent vs when assignment is durable
- [ ] Failure mode analysis: What happens if a worker reserves a job but the confirmation fails? How is the reservation eventually released?
- [ ] Scaling estimation: At 2,000 job dispatches/minute during surge, what's the reservation collision rate if 2M workers are competing? Show the math.
- [ ] Tradeoff analysis (minimum 3):
  - LWW vs reservation protocol for job assignment (availability vs correctness)
  - Regional coordinator (single authoritative DC per job) vs quorum-based assignment
  - Optimistic locking vs pessimistic locking for the assignment record
