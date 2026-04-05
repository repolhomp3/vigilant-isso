# Vigilant ISSO One-Week MVP Cost Model
## AWS GovCloud (US-West) / us-gov-west-1

## Purpose

Provide a **tight, practical cost model** for running a **lean Vigilant ISSO MVP** in **AWS GovCloud (US-West)** for **one week** in order to:
- exercise the architecture
- gather runtime and workflow stats
- validate signal flow and operator usefulness
- avoid confusing this with a full production deployment

This model assumes a **short-lived measurement-oriented MVP**, not a comfort-sized long-term environment.

---

## 1. Scope assumptions

This estimate assumes:
- one small EKS-hosted Vigilant ISSO runtime
- monitoring one bounded environment
- lightweight workflow/evidence persistence
- object-backed evidence storage
- basic notifications
- optional but restrained Bedrock use for summaries/playbooks
- one-week runtime only
- low-to-moderate log volume
- no heavy search tier
- no broad on-prem connector expansion

---

## 2. One-week ROM summary

| Scenario | Weekly ROM | Notes |
|---|---:|---|
| **Ultra-lean prototype** | **$75–$200** | Tight runtime, small nodes, low logs, minimal Bedrock |
| **Lean realistic MVP** | **$150–$350** | Best planning range for a useful 1-week test |
| **Comfortable / slightly overprovisioned MVP** | **$350–$800+** | More logs, more headroom, more AI usage, more retention |

## Recommended planning number
For leadership or internal planning, use:

**$150–$350 for one week in us-gov-west-1**

That is the cleanest “credible but not inflated” answer.

---

## 3. Cost drivers

## 3.1 Fixed-ish or semi-fixed drivers
- EKS control plane cost
- small database or state backend cost
- secrets / KMS / basic control-plane services

## 3.2 Variable drivers
- worker node size and count
- CloudWatch log volume
- S3/object storage volume
- Bedrock calls
- retention choices
- whether notification/testing volume is noisy

## 3.3 Biggest wildcard for a one-week MVP
The two biggest cost wildcards are usually:
- **CloudWatch log volume**
- **Bedrock usage**

Those are also the easiest things to control deliberately.

---

## 4. Lean weekly planning posture

If the goal is to gather stats cheaply, the MVP should:
- keep node sizes small
- keep service count low
- use one small runtime cluster
- retain only the logs actually needed for the test
- keep Bedrock use bounded to a few workflows
- avoid broad indexing and heavy search
- avoid overprovisioning “just in case”

This is not the week to build a deluxe platform.

---

## 5. Weekly estimate by operating posture

## Ultra-lean prototype
### Target weekly cost
**$75–$200**

### Assumptions
- one small EKS footprint
- minimal persistent state
- S3/object usage stays small
- log volume stays lean
- Bedrock is used only for a few summary operations
- no broad analytics or heavy replay work

### Interpretation
This is the “prove the bones” mode.
Good for:
- runtime testing
- evidence flow testing
- workflow proving

Not ideal for:
- realistic long-term observability posture
- generous retention
- broad usage simulation

---

## Lean realistic MVP
### Target weekly cost
**$150–$350**

### Assumptions
- small but sane EKS runtime
- modest state backend
- evidence store in S3
- CloudWatch logs kept under control
- some notification traffic
- bounded Bedrock summaries or playbook support
- enough logging and metrics to learn something useful

### Interpretation
This is the best default planning range.
It is lean without being fake.

---

## Comfortable / slightly overprovisioned MVP
### Target weekly cost
**$350–$800+**

### Assumptions
- extra headroom in node sizing
- more aggressive logging
- more generous retention
- more Bedrock use
- more services turned on than the prototype strictly needs

### Interpretation
Easy to drift into this range if nobody guards scope.
This is the range where people accidentally prototype like they’re already in production.

---

## 6. Best planning answer

If someone asks:

> “What will it cost us to run the Vigilant ISSO MVP in us-gov-west-1 for one week and gather stats?”

A strong answer is:

> **Roughly $150–$350 for a lean useful week, with a lower bound near $75–$200 if we keep it extremely tight.**

That is the practical answer.

---

## 7. Bottom line

### One-week GovCloud MVP ROM
- **Very lean:** $75–$200
- **Lean realistic:** $150–$350
- **Overprovisioned / noisier:** $350–$800+

### My recommendation
Plan around:

**$150–$350 for a one-week us-gov-west-1 MVP run**

and deliberately control:
- log volume
- Bedrock usage
- service count
- retention

That will keep the test informative without turning it into a money leak.
