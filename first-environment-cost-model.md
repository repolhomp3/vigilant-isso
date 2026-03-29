# Vigilant ISSO First-Environment Cost Model

## Purpose

Provide a tighter first-environment cost model for the updated Vigilant ISSO direction:

**a lean EKS-hosted control service using MCP-style integrations to monitor one bounded AWS environment.**

This estimate assumes the monitored environment is:
- **1 AWS account**
- **2 VPCs**
- **1 data lake**
- **3 small EKS clusters**

And that Vigilant ISSO itself is a **small platform footprint**, not a large multi-cluster internal platform.

---

## 1. Separate the two questions

### A. One-time build cost
What does it cost to stand up the first useful lean prototype?

### B. Monthly AWS run cost
What does it cost to run the small Vigilant ISSO control service each month?

These are different numbers.

---

## 2. First-environment one-time build ROM

### Lean proof / narrow MVP
**$15K–$50K**

Assumes:
- highly disciplined scope
- small service set
- MCP-based AWS integrations only
- minimal UI
- no broad on-prem work
- very light AI or no AI initially

### Credible first implementation
**$50K–$125K**

Assumes:
- solid Terraform/Helm setup
- trustworthy workflow/evidence state
- notifications and monthly snapshot output
- closure validation and freshness logic
- enough polish that an ISSO can actually use it

### Better-but-still-bounded first environment
**$125K–$225K**

Assumes:
- stronger operator usability
- better packet/snapshot handling
- more robust MCP integrations
- optional bounded Bedrock summarization

## Honest planning number
For the updated lean direction, a good first planning number is:

**$50K–$125K** for a credible first implementation.

---

## 3. Monthly AWS run cost ROM

### Ultra-lean prototype
**$100–$300/month**

Possible if:
- one small EKS footprint
- tiny DB or similarly lean state backend
- small object storage use
- low logs and retention
- minimal or no AI calls

### Lean but credible
**$300–$700/month**

Assumes:
- small EKS runtime
- modest persistent state
- object storage
- notifications
- logs/backups
- light Bedrock use if enabled

### More comfortable first environment
**$700–$1,500/month**

Assumes:
- more retention
- more logs
- more search/retrieval
- more frequent AI use
- more comfort margin in sizing

## Honest monthly target
For this updated architecture, the best planning range is:

**$300–$700/month**

### Stretch-low stance
If you want to chase your intuition that this can be very cheap:

**$200/month is now much more plausible** as an aggressive low-end prototype target.

But that still assumes:
- strict scope discipline
- light usage
- minimal AI spend
- prototype-grade resilience expectations

---

## 4. Why the number dropped

The number drops because the architecture changed.

We are no longer pricing:
- a large generalized compliance platform
- a broad multi-cluster hosting footprint for Vigilant ISSO
- heavy always-on search/analytics infrastructure
- a broad integration estate

We are now pricing:
- one small EKS-hosted control layer
- MCP pods/services that query AWS-native sources directly
- a small persistent workflow/evidence core
- notifications and monthly readiness outputs

That is a much lighter system.

---

## 5. Cheap-by-design architecture posture

To stay in the low monthly range, keep:
- one small EKS runtime
- one lightweight structured state backend
- S3/object-backed evidence storage
- AWS-native MCP integrations only
- Teams/email notifications
- monthly snapshots
- optional, tightly bounded Bedrock usage

Avoid early:
- broad on-prem connectors
- graph analytics
- large search infrastructure
- rich dashboards for every audience
- expensive always-on AI features
- generalized platform behavior beyond this first environment

---

## 6. Boss-ready framing

### One-time build
> “For one bounded AWS environment and a lean MCP-based architecture, the first Vigilant ISSO implementation looks more like roughly $50K to $125K than a giant internal platform spend.”

### Monthly AWS cost
> “For the first scoped environment, the lean target is roughly $300 to $700 a month, with $200 plausible if we keep the prototype very small.”

---

## 7. Bottom line

### First-environment one-time build ROM
**$50K–$125K** for a credible first implementation

### First-environment monthly AWS run cost ROM
**$300–$700/month** as the realistic lean target

### Aggressive low-end prototype target
**~$200/month** is now a plausible stretch goal under the revised architecture

That is the tighter answer for the new direction.
