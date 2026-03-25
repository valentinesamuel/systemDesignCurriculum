# NexaCare — Transition

Ten months at NexaCare. The FDA inspection concluded with zero Part 11 findings — a result Dr. Mia Torres described as "unprecedented in my 20 years of clinical trials compliance work." The cryptographic audit trail survived the inspector's live tamper-test. The deletion pipeline cleared 847 pending requests in the first weekend after launch.

The Vault migration hit a snag in week three: Sasha Petrov's resistance to dynamic credentials turned out to be justified — one of the database connection poolers (PgBouncer) didn't support credential rotation mid-session. Rachel Kim handled the conversation with Sasha exactly as predicted. The fix took one afternoon and a PgBouncer configuration change.

The SLO work paid off in month seven. P1 alert volume dropped from 200/week to 4/week. Nina Park sent a Slack message to the engineering channel: "I slept through the night four times this week. First time since I joined." It got 23 reactions.

Your exit interview with Fatima Al-Rashid: "You understand something that most engineers don't — that compliance and architecture are the same conversation. The lawyers aren't your adversaries. The regulations are constraints, like latency is a constraint. You engineer around them."

On your last day, Rachel Kim sends a LinkedIn recommendation that includes the phrase "helped save this company from a HIPAA breach that would have ended it." You appreciate the honesty.

The next morning: a recruiter from TradeSpark — an algorithmic trading firm. They have a production incident they can't diagnose. The runbook is three years out of date. The previous infrastructure engineer left to start a hedge fund.

You've heard this story before.
