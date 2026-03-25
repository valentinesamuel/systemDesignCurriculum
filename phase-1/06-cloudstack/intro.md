# CloudStack — Company Introduction

CloudStack is a B2B SaaS developer platform — the "Heroku for enterprises." They
provide managed compute, database-as-a-service, container orchestration, and
networking infrastructure for companies that want AWS-grade capabilities without
the operational overhead of managing AWS directly. Their 3,200 tenants range from
Series A startups running a few containers to Fortune 500 teams running dozens of
microservices with compliance requirements.

The platform is shared infrastructure: all 3,200 tenants run on the same physical
clusters, managed databases, and networking fabric. CloudStack's core engineering
promise is that tenants are completely isolated — one tenant's noisy workload
cannot affect another's performance, and under no circumstances can one tenant
access another's data.

The Platform Engineering team is 45 people. **Seo-yeon Park**, Head of Platform
Engineering, runs a tight ship. She has a rule: "If a tenant can tell they share
infrastructure with another tenant, that's a bug. Fix it."

You join as a **Senior Backend Engineer** on the Platform team. Your first project:
a complete review of the rate limiting architecture. CloudStack has had three
incidents in the past year where one "noisy neighbor" tenant degraded the experience
for 200+ other tenants. The root cause in all three: no API rate limiting.
