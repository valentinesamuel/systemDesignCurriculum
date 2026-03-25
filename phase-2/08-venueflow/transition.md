# VenueFlow — Job Transition

## After the Taylor Swift Onsale

The Taylor Swift onsale runs on a Tuesday morning in October. You are in the office at 8 AM. The team is in the office at 8 AM. Nobody asked anyone to come in — it just happened.

At 10:00 AM, the onsale opens. The virtual waiting room holds. The event-sourced inventory handles the concurrency without a single double-sell. The adaptive rate limiting catches and throttles 67% of bot traffic before it reaches the reservation layer. Latency on the seat map stays under 180ms throughout. Zero double-sells. Zero down alerts. The queue drains smoothly over forty minutes.

Zara watches from a chair at the back of the war room with her arms crossed and says nothing for a long time. Then she says: "You did what I came here to do."

She means it as a compliment. You take it as one.

The postmortem is quiet — five minor issues, none customer-facing. You close it in a single meeting. The bot incident retrospective goes to the legal team and, three weeks later, to a Senate subcommittee staffer who is drafting language for the TICKET Act. VenueFlow is cited as an example of what proactive anti-bot infrastructure looks like.

You spend another six weeks finishing the event-sourced inventory rebuild. Petra Holst runs it. You review the design. When it ships to production, the test suite includes a scenario called "Taylor Swift" — 2M concurrent simulated users, 80k seats, zero double-sells required. It passes on the first run.

Three months after the onsale, you get a LinkedIn message from the Head of Network Engineering at TeleNova — a carrier that uses VenueFlow APIs to power a fan-ticketing portal for three European mobile markets. His name is Lars Engström. He saw the engineering blog post about the onsale architecture. He writes: "We have problems at a different scale entirely, but the same shape. Would you be open to a conversation?"

You are.

---

**Next:** TeleNova — Europe's third-largest mobile carrier, 120 million subscribers, 8 countries, and a 5G launch in 90 days.
