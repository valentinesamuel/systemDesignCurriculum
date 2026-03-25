# Arc 16: QuantumEdge — Quantum Computing / Hybrid Quantum-Classical Platform

**Chapters**: Ch. 260–265
**Role**: Founding Engineer #2
**Industry**: Quantum Computing / Hybrid Quantum-Classical Platform

---

## Company Introduction

QuantumEdge is a quantum computing startup operating at the bleeding edge of a technology that, by almost any measure, does not yet reliably work. They lease time on three superconducting QPUs (Quantum Processing Units) from a hardware vendor in Finland, run ten GPU nodes in a co-location facility in Amsterdam, and have fifty enterprise customers — pharmaceutical companies, financial firms, materials science labs — each paying $100,000 per year for access to compute that is simultaneously the most powerful and the most fragile in the world.

Dr. Yuki Tanaka, founder and CEO, has a PhD in quantum error correction from Caltech and the kind of focused intensity that comes from spending eight years solving a problem everyone else calls impossible. She built the first version of the platform herself: a Python script running on her laptop that accepts circuit submissions via email, validates them manually, and schedules QPU time in a spreadsheet.

It has been working fine with five customers. It will not work with fifty.

You join as Founding Engineer #2. The equity package is unusual. The challenges are more unusual still. On your first morning, Dr. Tanaka walks you to a whiteboard and draws a diagram: three QPUs, ten GPU nodes, fifty enterprise customers, and an arrow labeled "job scheduler" pointing to a drawing of a laptop with a cartoon flame underneath it.

"That's the system," she says. "We have a pharmaceutical customer who wants to submit ten thousand molecular simulation circuits by Friday. I need you to tell me what we build."

This is the kind of engineering that requires you to simultaneously understand the constraints of physics, the requirements of enterprise customers, and the economics of a technology in free fall toward mass adoption. The hardware will get better. The error rates will drop. The qubit counts will climb. You are building for a future state that doesn't exist yet.

That is the job.

---

**Study Note**: Chapters in this arc introduce concepts unique to quantum computing platforms: circuit validation, QPU scheduling, error mitigation pipelines, and hybrid quantum-classical orchestration. The underlying system design patterns — resource-based authorization, fair-use scheduling, pipeline orchestration, cost modeling under uncertainty — are applicable across all domains. Do not skip the domain context; understanding why QPU time is scarce is essential to reasoning about the tradeoffs.
