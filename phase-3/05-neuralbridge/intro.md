# NeuralBridge — Company Introduction

## The Company

NeuralBridge is a neuroscience technology company building the infrastructure layer for brain-computer interfaces (BCIs). Their platform sits between the raw electrode signals captured by implanted or non-invasive BCI devices and the applications that use that data — clinical trial software, neurofeedback consumer products, and academic research tools. Think of them as AWS for brain data: a cloud platform that handles signal ingestion, processing, storage, and analysis so that device manufacturers and application developers don't have to build it themselves.

The company is four years old, Series C, and has just received FDA Breakthrough Device Designation for their clinical trial platform. They have 180 employees, offices in San Francisco and Boston, and a small engineering team in Berlin managing European regulatory compliance. The platform currently serves 12 active clinical trials, 3 consumer neurofeedback products, and 8 university research programs.

## The Industry

BCI is experiencing its "smartphone moment." Non-invasive headset devices have become commodity hardware. The bottleneck is now software infrastructure. NeuralBridge is betting that no device manufacturer wants to build HIPAA-compliant, FDA-auditable, real-time signal processing pipelines — they want to ship devices and let the platform handle the rest.

Brain data is categorically different from other health data. It is simultaneously the most sensitive personal data ever collected (thoughts, emotional states, cognitive patterns) and a novel regulatory gray zone. HIPAA provides a floor, not a ceiling. International regulations are still being written. The civil liberties dimensions are live debates, not settled law.

## Your Role

You join as a new hire — a Senior Engineer moving into a Staff-level role, brought in to own the signal processing platform. The offer letter arrived on a Wednesday. Your first day is a Monday. By Friday of your first week, an FDA field reviewer will be on-site for a clinical trial milestone inspection.

You have five days to understand a system that three product teams depend on and that the regulatory team has been quietly worried about for six months.

## First Day

You badge in at 8:47am. Your manager, Dr. Lena Strauss (Head of Platform Engineering), is already in the all-hands war room with the regulatory affairs lead and two other engineers. She waves you over. The whiteboard has a hand-drawn pipeline diagram with three sections outlined in red marker. Someone has written "FDA FRIDAY" in the corner and underlined it twice.

Lena hands you a laptop and an access card. "Welcome. We'll do the tour later. Right now I need you to read this." She slides a printed incident report across the table. The header reads: *Signal Processing Pipeline — Stability Incident Log — Last 90 Days.*

There are 23 entries.

## Arc Overview

**Chapters 178–183** (6 chapters)

- **Ch. 178** — The Signal: Real-time neural signal processing pipeline
- **Ch. 179** — FDA 510(k) Architecture: Software as a Medical Device
- **Ch. 180** — Brain Data Privacy: HIPAA and beyond
- **Ch. 181** — The Systemic Failure: Cross-team architecture review (mandatory story beat)
- **Ch. 182** — Edge Intelligence: BCI edge processing and graceful degradation
- **Ch. 183** — The Clinical Data Platform: 21 CFR Part 11 compliance

**Milestone**: `NeuralBridge Arc — Chapters 178–183`
