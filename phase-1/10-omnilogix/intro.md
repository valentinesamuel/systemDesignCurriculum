# OmniLogix — Company Introduction

OmniLogix is a global supply chain management platform that coordinates manufacturing
orders, supplier procurement, inventory allocation, and last-mile fulfillment for
Fortune 500 manufacturers across 47 countries. Their clients — automotive OEMs,
pharmaceutical distributors, electronics manufacturers — cannot tolerate supply chain
failures. When a Toyota assembly plant in Kentucky runs out of a specific bolt, the
conveyor stops. When a hospital's pharmaceutical distributor fails to confirm a drug
shipment, patient care is at risk.

The platform runs 14 data centers across 6 continents: North America, Europe, Asia-Pacific,
Southeast Asia, South Asia, and Latin America. Each data center serves the regional
supply chain operations of that geography. But inventory is global — a shortage in
Asia might be solved by a surplus in Europe, if the system knows about it.

"The hard part," VP of Engineering **Kwame Asante** (who you know from VeloTrack)
explains on your first day, "is that our 14 data centers disagree about inventory
numbers. Each region processes orders locally, for speed. But globally, we sometimes
double-allocate the same inventory to two different regions. Clients don't find out
until a shipment is missing."

You join as a **Senior Backend Engineer** on the Distributed Systems team. You're
here to fix the most fundamental distributed systems problem: how to maintain
consistency across 14 data centers without sacrificing the availability that
supply chain operations demand.
