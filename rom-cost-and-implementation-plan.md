# Vigilant ISSO ROM Cost Estimate and Implementation Plan

## Purpose

Provide a rough order of magnitude estimate and implementation plan for Vigilant ISSO under the **current lean architecture direction**.

### Current direction
Vigilant ISSO is now framed as:

**a small EKS-hosted compliance-operations control service using MCP-style integrations to monitor a bounded AWS environment, not a large fielded platform spread across the monitored infrastructure.**

This matters because it materially changes both cost and implementation shape.

---

## 1. Scope note

The earlier broader platform-style estimate should now be treated as historical context for a much larger future platform scenario.

For the current direction, the preferred planning reference is:
- `first-environment-cost-model.md`

That file reflects the tighter first-environment and lower-footprint architecture.

---

## 2. Updated planning stance

For the current first implementation, assume:
- one bounded AWS environment first
- one small Vigilant ISSO runtime in EKS
- MCP-style access to AWS-native services
- lightweight persistent workflow/evidence state
- optional Bedrock use
- policy-aware Teams/email notifications

### Practical planning answer
For the current direction, use this as the working estimate:

- **One-time build:** roughly **$50K–$125K** for a credible first implementation
- **Monthly AWS run cost:** roughly **$300–$700/month** as the realistic lean target
- **Aggressive low-end prototype target:** around **$200/month** if the prototype stays very small

---

## 3. Implementation shape

The implementation plan should now focus on:

### Phase 0 — prototype truth model and scope
- lock monitored environment scope
- define MCP service set
- define minimal persistent state
- define notification and monthly snapshot outputs

### Phase 1 — small EKS runtime + Terraform/Helm baseline
- provision lean runtime footprint
- baseline state store and object store
- wire IAM, secrets, and observability

### Phase 2 — MCP integrations and workflow core
- build MCP service set for AWS-native sources
- add finding intake, workflow state, evidence freshness, remediation routing

### Phase 3 — readiness outputs
- add monthly snapshot generation
- add packet/export capability
- add notification and digest behavior

### Phase 4 — optional bounded AI
- add retrieval and Bedrock-backed drafting only if useful and low-risk

---

## 4. Bottom line

The current direction is cheaper because the system is smaller.

The right leadership story is now:

**Vigilant ISSO first proves value as a lean MCP-based monitoring and compliance-operations control layer for one AWS environment, then expands only if the team trusts it and wants more.**
