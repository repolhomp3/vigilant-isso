# Vigilant ISSO One-Week MVP Bill of Materials Cost Table
## AWS GovCloud (US-West) / us-gov-west-1

## Purpose

Provide a **bill-of-materials-style cost view** for a one-week lean Vigilant ISSO MVP run in `us-gov-west-1`.

This is a planning table, not a quote.
It is designed to show where the weekly spend likely comes from.

---

## 1. BOM-style weekly cost view

| Cost element | Lean weekly estimate | Notes |
|---|---:|---|
| **EKS control plane** | $50–$80 | Usually one of the fixed baseline costs |
| **Worker compute / nodes** | $20–$90 | Depends on node count/size and how lean the service set is |
| **State backend (small DB / managed persistence)** | $15–$60 | Small, but not zero |
| **S3 / evidence/object storage** | $1–$10 | Usually minor for a 1-week MVP |
| **CloudWatch logs and basic metrics** | $10–$75 | Can stay low or creep fast if left untuned |
| **Queue / events / glue services** | $2–$15 | Usually modest at prototype scale |
| **Secrets / KMS / misc managed services** | $2–$15 | Small but real |
| **Bedrock usage** | $0–$100+ | Very workload-dependent; easiest wildcard to cap |
| **Notifications / email / Teams support plumbing** | $0–$10 | Usually negligible if using existing channels |
| **Buffer / unexpected small charges** | $10–$45 | Good to leave some room |
| **Estimated weekly total** | **$110–$500+** | Practical planning range still narrows lower with discipline |

---

## 2. Interpreting the BOM

## Lowest credible posture
You can get close to the low end if you:
- keep the EKS footprint tiny
- minimize node count
- keep logs under control
- avoid broad indexing/search
- cap Bedrock usage aggressively

## Most likely cost leaks
The parts most likely to drift upward are:
- CloudWatch logs
- Bedrock calls
- node sizing / overprovisioning

## Least important line items
At one-week MVP scale, these are usually not the problem:
- S3
- notification plumbing
- secrets/KMS overhead

---

## 3. Practical planning scenarios

| Scenario | Expected weekly total | What that means |
|---|---:|---|
| **Tight prototype** | **$75–$200** | Very lean; enough to validate flow and gather basic stats |
| **Lean useful MVP** | **$150–$350** | Best default target; useful data without overbuilding |
| **Loose / comfortable MVP** | **$350–$800+** | Still not huge, but indicates weak scope control |

---

## 4. Recommended budget answer

If someone wants the quick BOM-style answer:

> **Expect the one-week us-gov-west-1 MVP to land around $150–$350, with the main cost coming from EKS baseline, worker compute, logging, and any Bedrock usage.**

That is the right concise answer.

---

## 5. Bottom line

The one-week MVP should be treated like a **controlled experiment**:
- small footprint
- bounded AI usage
- measured logs
- narrow scope

If we do that, the weekly AWS bill should stay in the **low hundreds of dollars**, not the thousands.
