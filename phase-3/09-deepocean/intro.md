# DeepOcean — Company Introduction

**Industry**: Oceanography / Marine Data Platform
**Your Role**: New Hire — Staff Engineer
**Arc**: Chapters 209–214

---

DeepOcean is the data infrastructure layer for the world's oceans. The platform ingests AIS vessel tracking signals, satellite altimetry, ocean buoy networks, research vessel telemetry, and government agency feeds from 47 countries — aggregating them into a unified marine data platform used by shipping companies, port authorities, coast guards, and oceanographic research institutions. The company has 340 employees spread across offices in Bergen, Singapore, and Miami. The data science team is world-class. The engineering team is chronically understaffed.

You join as a Staff Engineer on your second day in Bergen, Norway — jet-lagged, wearing the wrong coat for North Atlantic February. Your hiring manager, Erik Solberg (VP Engineering), greets you with a handshake and a Slack notification on his phone.

"Good timing," he says. "The AIS ingestion pipeline is losing messages again."

You haven't taken your coat off yet.

The platform serves 12,000 registered users across 800 organizations. At its core it tracks approximately 100,000 active vessels worldwide — cargo ships, tankers, fishing fleets, coast guard cutters, research vessels, ferries, and private yachts. Every vessel with AIS transponders broadcasts position, heading, speed, and identification data. At sea, that's one message every five seconds per vessel. In busy straits and port approaches, vessels transmit more frequently and terrestrial AIS receivers cluster the signal — creating geographic hotspots of message density that overwhelm the ingestion pipeline on a regular basis.

The AIS message loss during peak port traffic is the first problem. It will not be the last. The ocean, you will learn, is an edge environment — and edge environments are where all your assumptions about connectivity, consistency, and real-time data go to drown.

Dr. Adaeze Obi, the Chief Science Officer, will become the most interesting colleague you've had. She sketches architecture diagrams on flights and sends photos of handwritten notes with questions embedded in them that she never answers herself. She says she learned this habit from a mentor who never gave direct answers either.

You unpack your laptop. Erik sends you the AIS pipeline runbook. It is 400 words and was last updated in 2022.

Welcome to DeepOcean.
