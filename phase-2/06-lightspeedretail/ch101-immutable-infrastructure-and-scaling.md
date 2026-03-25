---
**Title**: LightspeedRetail — Chapter 101: Immutable Infrastructure and Black Friday Scaling
**Level**: Staff
**Difficulty**: 8
**Tags**: #immutable-infra #auto-scaling #spot-instances #terraform #packer #iac #black-friday
**Estimated Time**: 5 hours
**Related Chapters**: Ch. 99, Ch. 100, Ch. 102, Ch. 110 (Crestline capacity planning)
**Exercise Type**: System Design / Spike — Scale 10x
---

### Story Context

**#engineering-all-hands** — Slack, Wednesday October 9, 14:00

**Beatriz Santos**: Quick announcement before the product sync. I've set a Black Friday readiness requirement I want everyone to hear directly from me.

**Beatriz Santos**: I want to be able to burn every server to the ground at 11PM Wednesday before Black Friday and have the full stack back up at 4AM Thursday. Clean. Reproducible. No manual steps. No snowflake servers. No one-off configurations.

**Beatriz Santos**: If we can do that — if we can prove we can rebuild from nothing in 5 hours — then I will have confidence that when something breaks at 9AM Black Friday, we can recover it in 30 minutes instead of 4 hours.

**Beatriz Santos**: @[you] — you're owning this. I want a design by Monday.

---

**Slack DM — Ravi Patel (DevOps Lead) → [you]** — Wednesday October 9, 14:31

**Ravi Patel**: Did you see the all-hands message?

**[you]**: Yes.

**Ravi Patel**: Just so you know what you're walking into: we have 47 "unique" EC2 instance configurations right now. 12 of them were set up by engineers who no longer work here. Three of them have configuration changes that were made by SSH-ing directly into the running instance. Nobody knows what those changes were.

**[you]**: Three servers with unknown manual changes?

**Ravi Patel**: At least three. Probably more. We have no IaC. Some servers were created from the AWS console in 2019.

**[you]**: Okay. What's our current peak capacity?

**Ravi Patel**: Normal peak: 50,000 RPS across the payment gateway, product catalog API, and inventory service. Black Friday target: 250,000 RPS. 5x, not 10x — but Beatriz rounded up in the all-hands.

**[you]**: What's our scaling mechanism right now?

**Ravi Patel**: Manually increasing instance count the week before Black Friday. We provision 3x our normal fleet. We keep it running for 10 days before and after because we're afraid to change anything close to the event.

**[you]**: How much does that cost?

**Ravi Patel**: Last year: $340,000 for the 10-day over-provisioned window. That's on top of our $180,000/month baseline.

**[you]**: So $340k for a 10-day insurance policy against a 4-hour recovery window.

**Ravi Patel**: Correct. And we still had an incident.

---

**1:1 — [you] and Beatriz Santos** — Thursday October 10, 09:00

**Beatriz Santos**: Give me the honest picture first. What are we actually dealing with?

**[you]**: Three categories of problem. First: no infrastructure as code — everything is a snowflake. Second: no auto-scaling — we manually provision before Black Friday and hope. Third: no immutable deployment pipeline — engineers SSH into running servers and make undocumented changes. Any one of these would be a problem. All three together means we can't recover quickly when something breaks.

**Beatriz Santos**: How long to fix all three?

**[you]**: Properly? Six months. But we can get to the 4AM rebuild target in 4 weeks if we focus on the critical path.

**Beatriz Santos**: What's the critical path?

**[you]**: Build golden AMIs for our three critical services — payments gateway, product catalog API, inventory service. Put them behind Auto Scaling Groups with target-tracking policies. Write Terraform to provision the full stack from scratch. Run the burn-and-rebuild drill the Wednesday before Black Friday.

**Beatriz Santos**: Cost?

**[you]**: Spot instances for the stateless services with on-demand fallback. My estimate: Black Friday window drops from $340k over 10 days to $80k over 3 days. Plus $30k engineering time for the tooling build.

**Beatriz Santos**: So you're telling me immutable infra saves us $230k per year on Black Friday alone.

**[you]**: Minimum. And we stop paying the hidden cost of unknown server configurations failing at 3AM.

**Beatriz Santos**: Do it. Monday design review. Full proposal.

---

**Slack DM — Marcus Webb → [you]** — Thursday October 10, 12:15

**Marcus Webb**: Heard about the "burn it down" mandate. Good instinct from Beatriz. Most CTOs say they want immutable infra and then panic when they see what it takes to get there.

**[you]**: 47 unique EC2 configs. Three with unknown SSH-applied changes.

**Marcus Webb**: Classic. I called it "snowflake drift" at the grocery chain. Every week someone logs in to "just fix one thing" and now you have a configuration archaeology problem. You can't reproduce it, you can't scale it, and you definitely can't recover it at 3AM on Black Friday.

**Marcus Webb**: Packer for AMI baking. Non-negotiable. Your build pipeline produces the AMI. The AMI is the artifact. No human ever touches a running server again.

**[you]**: What about stateful services — database, Redis?

**Marcus Webb**: Those are different. Stateless services: fully immutable, auto-scale. Stateful: RDS Multi-AZ, ElastiCache with replication, managed services where possible. Your immutable pipeline applies to the application tier. Stop there.

**Marcus Webb**: One thing that bites people: spot instance interruptions during Black Friday. Design for it explicitly. 2-minute interruption notice. What does your drain handler look like?

**[you]**: I'll add a SIGTERM handler with a 90-second drain window.

**Marcus Webb**: Good. And stagger your Auto Scaling Group launch configs. If you need to replace the entire fleet at once and all your spot instances are in the same AZ, you'll starve the spot market.

---

The conversation Beatriz had in the all-hands was not metaphorical. She built her previous company's engineering culture around a practice called "chaos Wednesdays" — a weekly window where any engineer could trigger a random infrastructure failure. The companies that survive chaos Wednesdays are the ones that never need to pull an all-nighter before Black Friday.

### Problem Statement

LightspeedRetail's infrastructure is a collection of 47 manually-configured EC2 instances with no IaC, no auto-scaling, and no reproducible deployment process. Black Friday requires scaling from 50,000 RPS to 250,000 RPS. The current approach — manually provisioning 3x capacity for 10 days — costs $340,000 per year and still produces incidents. CTO Beatriz Santos has mandated an immutable infrastructure design with a target of full stack rebuild from zero in under 5 hours.

You must design the immutable infrastructure pipeline using Packer-baked AMIs, Terraform IaC, Auto Scaling Groups with target-tracking policies, spot instance strategy with on-demand fallback, and a burn-and-rebuild verification drill plan for the Wednesday before Black Friday.

### Explicit Requirements

1. All stateless services (payments gateway, product catalog API, inventory service) must run from Packer-baked golden AMIs.
2. No human may SSH into a running production instance — all changes go through the AMI pipeline.
3. Terraform must be able to provision the full application-tier stack from scratch.
4. Auto Scaling Groups must use target-tracking scaling policies keyed on CPU and custom RPS metrics.
5. Spot instances must be the primary compute for stateless services, with On-Demand instances as fallback for critical services.
6. Instance drain on spot interruption: 90-second SIGTERM drain before termination.
7. The full stack rebuild drill must complete in under 5 hours from scratch (zero to traffic-serving).
8. Cost for Black Friday scaling window must not exceed $90,000 (down from $340,000).
9. Stateful services (RDS, ElastiCache) are out of scope for immutable pipeline — use managed service features.

### Hidden Requirements

- **Hint**: Re-read Ravi Patel's message about "three servers with unknown SSH-applied changes." Before you can build golden AMIs, you must reverse-engineer the current state. There is a hidden requirement: a configuration audit phase before AMI baking begins. How do you reconstruct the known-good configuration of servers you cannot reproduce?

- **Hint**: Re-read Marcus Webb's warning about "spot instance interruptions during Black Friday." The Auto Scaling Group must span multiple AZs and use a diversified instance type strategy. Spot market exhaustion is a failure mode — what is your fallback priority order?

- **Hint**: Re-read Beatriz Santos: "burn every server to the ground at 11PM Wednesday and have the full stack back up at 4AM Thursday." There is a hidden requirement: DNS cutover or load balancer target group swap must be part of the Terraform plan. You cannot "burn it down" if traffic is still pointing at the old fleet.

- **Hint**: Re-read Ravi Patel: "We keep it running for 10 days before and after because we're afraid to change anything close to the event." There is a hidden requirement: a deployment freeze policy embedded in the pipeline — no AMI updates permitted in the 72h window before and 48h after Black Friday, enforced by CI/CD gates, not human discipline.

### Constraints

- **Current normal peak**: 50,000 RPS
- **Black Friday target**: 250,000 RPS (5x)
- **Black Friday duration**: 36 hours (Thursday midnight to Saturday noon)
- **Services in scope**: Payments gateway, product catalog API, inventory service (3 services)
- **Instance types**: Current: m5.2xlarge (8 vCPU, 32GB RAM); target: mix of c5 and m5 family
- **Spot discount**: ~70% off On-Demand for c5.2xlarge in us-east-1 (historical average)
- **Current Black Friday cost**: $340,000 for 10-day over-provisioned window
- **Target Black Friday cost**: ≤ $90,000 for 3-day scaled window
- **Rebuild target**: 0 to traffic-serving in under 5 hours
- **SIGTERM drain window**: 90 seconds
- **Team**: 3 DevOps engineers, 2 platform engineers, 4 weeks to Black Friday
- **Tooling**: Packer, Terraform, AWS Auto Scaling, AWS CodeBuild for AMI pipeline

### Your Task

Design the complete immutable infrastructure pipeline and Black Friday scaling strategy. Produce the full cost model and the burn-and-rebuild drill playbook.

### Deliverables

- [ ] **Mermaid architecture diagram** showing: AMI baking pipeline (code commit → CodeBuild → Packer → AMI → AMI registry) → Terraform provisioning (VPC, ALB, ASG, RDS, ElastiCache) → Auto Scaling Group scaling policy. Show spot and on-demand tiers separately.

- [ ] **Packer AMI baking pipeline design**:
  - Input: base Amazon Linux 2 AMI + application artifact (Docker image or binary)
  - Provisioners: system dependencies, application binary, systemd service, CloudWatch agent
  - Output: tagged golden AMI (include version, git SHA, build date in tags)
  - Pipeline: triggered on merge to main; AMI promoted to production via manual approval gate

- [ ] **Terraform resource plan** (list of resources, not full HCL):
  - VPC, subnets (multi-AZ), security groups
  - ALB, target groups, listeners, health check configuration
  - Launch template (AMI ID, instance type, spot config, user data)
  - Auto Scaling Group (min/desired/max, multi-AZ, mixed instances policy)
  - Target tracking scaling policy (target CPU 60%, custom metric: RPS per instance)
  - CloudWatch alarms for scale-out and scale-in

- [ ] **Spot instance strategy**:
  - Mixed instances policy: primary c5.2xlarge spot, fallback c5a.2xlarge spot, fallback m5.2xlarge on-demand
  - Interruption handler: SIGTERM → drain active requests → deregister from ALB → allow termination
  - Capacity rebalancing: enable proactive replacement before interruption notice
  - Show minimum on-demand capacity to maintain service during spot interruption cascade

- [ ] **Scaling estimation and cost model**:
  - Normal baseline: how many instances at 50k RPS? (assume 1,000 RPS per c5.2xlarge instance)
  - Black Friday: 250k RPS ÷ 1,000 RPS/instance = 250 instances
  - Spot cost: 250 instances × c5.2xlarge spot price ($0.085/hr) × 36 hours = show math
  - On-demand fallback (20% floor): 50 instances × $0.34/hr × 36 hours = show math
  - Total 3-day Black Friday cost vs $340k baseline. Show savings.

- [ ] **Burn-and-rebuild drill playbook** (bulleted steps):
  - Wednesday 23:00: drain traffic from old fleet
  - Wednesday 23:15: `terraform destroy` application tier
  - Wednesday 23:30: verify zero running instances
  - Rebuild: `terraform apply` — target 4:00 UTC complete
  - Smoke tests: automated health checks at 4:00 UTC
  - Traffic restore: 4:15 UTC
  - Show estimated time per phase and go/no-go criteria

- [ ] **Deployment freeze gate design**: CI/CD pipeline rule — block all AMI promotions to production if `NOW` is within 72h of Black Friday or 48h after. Implemented as a CodePipeline condition, not a calendar reminder.

- [ ] **Tradeoff analysis** (minimum 3):
  1. Packer + AMI vs Docker + ECS/EKS: AMIs are heavier to bake but simpler for teams unfamiliar with container orchestration at scale. When does ECS/EKS become worth the complexity?
  2. Target tracking vs step scaling: target tracking is reactive (lags on sudden spikes). For Black Friday with a known traffic shape, is pre-scheduled scaling better?
  3. Spot-only vs mixed fleet: pure spot is cheapest but has interruption risk. What is the minimum on-demand floor for each service to survive a spot interruption cascade?
