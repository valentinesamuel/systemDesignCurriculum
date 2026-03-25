---
**Title**: GigGrid — Chapter 85: CRDTs — Conflict-Free Replicated Data Types
**Level**: Staff
**Difficulty**: 9
**Tags**: #crdt #distributed-systems #eventual-consistency #conflict-free #availability
**Estimated Time**: 4 hours
**Related Chapters**: Ch. 84, Ch. 116
**Exercise Type**: Incident Response / System Design
---

### Story Context

**#incidents — GigGrid**
`Wednesday 14:22` **amara.chen**: worker availability is broken in regions 7, 9, and 11. I'm getting reports from clients in Stockholm, Madrid, and Warsaw.
`14:23` **you**: what kind of broken?
`14:24` **amara.chen**: workers who set themselves unavailable are still showing as available. workers who set themselves available aren't appearing in job dispatch queries.
`14:25` **you**: when did this start?
`14:26` **amara.chen**: unknown. I'm looking at the incident report from a nurse agency in Stockholm. they've been getting wrong availability data for "at least two weeks."

**[Inherited Disaster Investigation — Next 4 hours]**

You dig into the codebase. The previous tech lead, Tyler Wren, left three weeks ago. His farewell Slack (sent to the general channel the afternoon he left) said: "The availability sync thing is in there. I ran out of time. There's a TODO in worker_availability_sync.ts. Good luck." Nobody replied.

`worker_availability_sync.ts, line 247`:
```typescript
// TODO: This LWW merge is wrong for availability flags.
// If a worker sets themselves UNAVAILABLE on their phone (region 7)
// and the phone is offline for 30 min, then updates propagate from
// region 9 (where they were marked available by the system due to
// inactivity timeout), the LWW merge will overwrite the worker's
// explicit UNAVAILABLE with the system's stale AVAILABLE.
// Worker has no way to know. They keep getting job offers they can't take.
// Need CRDT here. G-counter or OR-Set probably.
// - Tyler (2024-09-15, his last day apparently)
```

**DM — Amara Chen → You**
`18:44`
**amara.chen**: found it?
**you**: tyler found it. three weeks before he left. didn't fix it.
**amara.chen**: what's the impact?
**you**: I've traced 2,847 availability flag inconsistencies over the past 6 weeks. that's the number of workers whose availability state in region 7, 9, or 11 is different from the correct state as they set it on their device. in 619 of those cases, a worker marked themselves unavailable but kept receiving job assignments.
**amara.chen**: 619 workers received unwanted job offers
**you**: and in 12 cases, they accepted because they didn't realize the assignment was wrong. one of them was a nurse who had marked herself unavailable for maternity leave.
**amara.chen**: ...
**you**: she's filed a formal complaint through a labor rights organization. legal has been notified.
**amara.chen**: fix this. Tyler's TODO says CRDT. do you know what that means?
**you**: I do. I need to explain it to you.
**amara.chen**: room 204. now.

---

**Whiteboard Session — Room 204**

**You**: CRDTs are Conflict-free Replicated Data Types. The "conflict-free" part means: the data structure is designed so that any two replicas, merged in any order, always converge to the same result. No coordination required. No conflict resolution needed. The math guarantees convergence.

**Amara**: Why haven't we been using this all along?

**You**: Because CRDTs only work for certain types of data with specific convergence properties. For job assignment — which has a "exactly one winner" constraint — CRDTs don't help. But for worker availability flags — which follow "worker's explicit choice beats system inference" semantics — there's a CRDT that fits perfectly.

**Amara**: Show me.

---

### Problem Statement

GigGrid's worker availability flags use LWW merge logic. When a worker explicitly marks themselves unavailable on a device in one region, and a system-generated availability event (e.g., inactivity timeout) occurs in a different region, the LWW merge can overwrite the explicit worker choice with the system's stale value. Workers marked as unavailable continue to receive job assignments they cannot take. 2,847 inconsistencies documented over 6 weeks, including one worker on maternity leave receiving job assignments.

You need to design a CRDT-based availability flag system where the worker's explicit intent always wins over system-generated events.

---

### Explicit Requirements

1. A worker's explicit "unavailable" declaration must never be overwritten by a system-generated "available" update, regardless of timestamp or region ordering
2. Availability changes must propagate across all 14 regions within 60 seconds
3. Workers must be able to set complex availability states: unavailable (reason: maternity leave), available-selective (will only accept certain job types), available-all
4. The CRDT merge must be associative, commutative, and idempotent (can be applied multiple times without changing the result)
5. Migration: 2,847 inconsistent records must be corrected without requiring workers to re-set their availability

---

### Hidden Requirements

- **Hint**: Re-read Tyler's TODO: "worker sets themselves UNAVAILABLE on their phone (region 7) and the phone is offline for 30 min." The offline case is important. If a worker sets themselves unavailable on their phone and the phone is offline, then comes back online — how does the CRDT ensure their availability state propagates correctly when there have been 30 minutes of system-generated events they didn't see?

- **Hint**: Amara mentioned "available-selective (will only accept certain job types)." This isn't a binary flag — it's a set of accepted job types. What CRDT models a "set that can only grow when the worker explicitly adds items, and can have items removed when the worker explicitly removes them"?

---

### Constraints

- 14 regions, 8M registered workers, 2M active at any time
- Availability states: unavailable | available-all | available-selective(job_types: string[])
- Workers update availability via mobile app (frequently offline for 5–30 minutes)
- System also generates availability events: inactivity timeout (after 4hr offline → mark unavailable)
- 2,847 records with confirmed inconsistency to fix
- No downtime for the fix — workers are actively using the platform

---

### Your Task

Design the CRDT-based worker availability system for GigGrid. Choose the specific CRDT type(s), define the merge function, and design the migration plan for fixing inconsistent records.

---

### Deliverables

- [ ] CRDT type selection: Evaluate G-Set, 2P-Set, LWW-Register, OR-Set, and MV-Register for this use case. Choose the best fit and explain why the others don't work.
- [ ] Availability CRDT design: Define the data structure for representing `available-selective(job_types)` as a CRDT. What does a merge look like when two regions have different job type sets?
- [ ] Worker intent vs system event: Define how the merge function distinguishes between a worker-generated update (higher authority) and a system-generated update (lower authority). What metadata enables this distinction?
- [ ] TypeScript interface sketch: `AvailabilityCRDT` with `merge()`, `add_job_type()`, `remove_job_type()`, `set_unavailable()` methods
- [ ] Convergence proof: Show that your merge function is associative (merge(A, merge(B, C)) = merge(merge(A, B), C)), commutative, and idempotent
- [ ] Migration plan: How to correct 2,847 inconsistent records by replaying worker-generated events with higher authority than system events
- [ ] Tradeoff analysis (minimum 3):
  - CRDT vs distributed lock for availability updates
  - OR-Set vs 2P-Set for the selective availability job type set
  - CRDT limitations: what types of availability semantics can't be expressed as CRDTs?
