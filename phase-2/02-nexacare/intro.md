# NexaCare — Company Introduction

NexaCare is a Series C startup building the "Stripe of clinical data" — the infrastructure layer that biotech companies use to collect, manage, and submit clinical trial data across 180 countries. If Stripe made payments invisible, NexaCare is trying to make trial data compliance invisible. Before NexaCare, a trial sponsor in Boston had to hire 40 data managers to wrangle regulatory submissions for a 3,000-patient, 12-country trial. NexaCare's platform reduces that to a team of 8.

You join as a **Senior Backend Engineer** eight months after your Stratum exit. VP Engineering **Fatima Al-Rashid** recruited you personally — your name came up in a distributed systems forum after the Stratum multi-region writeup circulated. "We need someone who can hold compliance and scale in their head at the same time," she wrote in the initial message. "Most engineers treat them as separate problems. They're not."

NexaCare's stack is Node.js/TypeScript throughout, with PostgreSQL, Kafka, and a custom event-sourcing layer that was written by three engineers who have since left the company. The system processes 50 million patient records across 180 countries, with 100,000 writes per day per active trial. There are currently 47 active trials.

Day one. You have read access to the production database. You find a table called `deletion_requests` with 847 rows, all marked `status: 'pending'`. The oldest is 14 months old.

You open Slack and type: "What is the deletion_requests table?"

Fatima's reply arrives in 30 seconds: "That's your first project."
