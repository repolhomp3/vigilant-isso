# Vigilant ISSO MVP Prototype Scope

## Purpose

Define the **tight prototype MVP** for Vigilant ISSO under the updated direction:

**a lean, low-cost EKS-hosted control layer using MCP-style server/client components to monitor one bounded AWS environment.**

This is intentionally smaller than the broader concept previously described.

---

## Prototype boundary

### Environment being monitored
The prototype monitors exactly:
- **1 AWS account**
- **2 VPCs**
- **1 data lake**
- **3 small EKS clusters**

### Platform footprint being fielded
The prototype fields only:
- one small Vigilant ISSO EKS runtime
- MCP pods/services for AWS-native access
- one lightweight workflow/evidence state service
- object-backed evidence storage
- notification/digest capability
- optional bounded Bedrock summarization

### Critical clarification
The monitored infrastructure is **not** the same thing as the infrastructure Vigilant ISSO requires to run.

That distinction is now part of the MVP definition.

---

## Prototype objectives

The prototype must prove five things:

1. It can monitor the bounded AWS environment without requiring a large platform footprint.
2. It can use MCP-style integrations to gather findings and validation data from AWS-native services.
3. It can preserve enough workflow and evidence state to be trustworthy.
4. It can support practical ISSO operations: triage, evidence freshness, remediation, closure validation, and monthly readiness snapshots.
5. It can do all of the above cheaply enough to justify real adoption.

---

## In scope

## 1. AWS-native sources
In scope:
- AWS Security Hub
- AWS Config
- Amazon Inspector
- Amazon GuardDuty
- IAM Access Analyzer
- CloudTrail references
- Systems Manager compliance/inventory only if already useful and enabled

These are accessed through **MCP-style service surfaces**, not a heavy connector platform.

## 2. Core workflow capabilities
In scope:
- finding intake
- normalization at a practical MVP level
- evidence metadata registration
- evidence freshness detection
- control mapping proposals
- remediation workflow records
- closure validation using authoritative source checks
- monthly readiness snapshot generation
- policy-aware Teams/email notifications

## 3. Minimal persistent state
In scope:
- findings metadata
- evidence metadata and references
- control mapping records
- remediation records
- snapshot records
- audit/review actions
- packet/export metadata

## 4. Optional bounded AI
In scope only if easy and low-risk:
- draft triage summary
- draft monthly readiness summary
- evidence gap summary

### AI guardrails
- advisory only
- citations required
- feature-flagged
- no autonomous closure or approval behavior

---

## Explicitly out of scope

- large multi-cluster hosting footprint for Vigilant ISSO itself
- enterprise-wide connector framework sprawl
- broad on-prem live integrations
- graph analytics
- broad semantic ingestion of all documents
- full assessor copilot features
- generalized platform architecture for many environments
- any feature that makes the system expensive before it is useful

---

## MVP definition of done

The prototype is “done enough” when it can:
- retrieve findings from the selected AWS-native sources through MCP-based integrations
- preserve provenance to the original source records
- maintain small but trustworthy workflow/evidence state
- show stale evidence and overdue remediation clearly
- support human-reviewed closure validation
- generate one reviewable monthly readiness snapshot for the bounded environment
- do this with a lean enough architecture that the operating cost remains credible

---

## Bottom line

This MVP is not “build the full Vigilant ISSO platform.”

It is:

**prove that a lean MCP-on-EKS control layer can monitor one bounded AWS environment and materially reduce ISSO compliance tedium without requiring a big expensive platform build.**
