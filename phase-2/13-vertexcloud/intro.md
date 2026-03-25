# VertexCloud — Company Introduction

**Industry**: Cloud Infrastructure / Internal Developer Platform
**Your Role**: Principal Architect (Platform Engineering)
**Arc**: Chapters 145–149

---

VertexCloud builds enterprise cloud infrastructure products. 2,000 engineers. 380 microservices. 14 product teams shipping software that Fortune 500 companies depend on to run their cloud operations. By every external measure, VertexCloud is a success.

Internally, the engineering organization is in a state of productive chaos. Fourteen teams deploy in fourteen different ways. Some use Terraform. Some use Pulumi. Some use hand-rolled CloudFormation that only the original author understands — and the original author left in 2023. Two teams have never had a proper staging environment. Three teams deploy directly to production on Fridays, which the platform team calls "the Friday Prayer." Last quarter, six production incidents were traced directly to infrastructure misconfiguration. Not code bugs. Not logic errors. Infrastructure configurations that were wrong because there was no standard way to write infrastructure configuration.

**Head of Platform Engineering Yael Cohen** hired you because two previous attempts to build an Internal Developer Platform failed. The first attempt produced a platform nobody used because it required learning a proprietary DSL. The second produced a platform that the compliance team shut down mid-launch because it didn't support audit logging. Yael told you this in your first interview with the words: "I need someone who won't make the same mistake twice. Both mistakes."

You joined because you've spent the last several years building systems that other systems depend on — monitoring infrastructure, compliance layers, data pipelines. An IDP is the same problem at a different layer. The customer is the engineer. The uptime metric is deployment frequency. The SLA is developer happiness.

**VP Engineering Reza Tehrani** controls the budget. He is pragmatic and data-driven. He will approve investment in the IDP if — and only if — you can show a measurable improvement in deployment frequency, incident rate, or infrastructure cost. You have one year to show the number.

On your first day, Yael takes you to the Confluence graveyard: a section of their internal wiki called "IDP Attempts." Two folders. Each with complete architecture documents, project plans, launch materials, and retrospective notes. You read both folders in your first two hours. You understand why they failed before you design anything new.

That's the right starting point.
