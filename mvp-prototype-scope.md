# Vigilant ISSO MVP Prototype Scope

## Purpose

Define the **tight prototype MVP** for the first bounded environment only.

This document intentionally narrows the broader Vigilant ISSO concept to a small, credible prototype that can be built, operated, and judged without pretending the full platform already exists.

---

## Prototype boundary

### Verified facts from this repo
- The broader Vigilant ISSO concept targets **AWS GovCloud and on-prem CMMC Level 2 environments** with an emphasis on evidence-grounded workflows, human approval, and audit readiness.
- Existing architecture docs already recommend an **EKS-hosted control plane**, a **relational system of record**, **object-backed evidence storage**, **workflow state**, and **bounded AI drafting** instead of autonomous compliance decisions.
- The first-environment cost model in this repo narrows planning to **1 AWS account, 2 VPCs, 1 data lake, and 3 small EKS clusters**.

### Working assumptions for this prototype
- The first prototype should focus on the AWS portion of the bounded environment first, with on-prem support limited to a small number of import-style placeholders rather than deep live integrations.
- The prototype is an **internal operator tool** for ISSOs, compliance analysts, and engineering/security reviewers, not a customer-facing product.
- The prototype should prove operating discipline and usefulness before adding multi-account, multi-tenant, or large-scale analytics features.

---

## In-scope environment

The prototype covers exactly this first environment:
- **1 AWS account**
- **2 VPCs**
- **1 data lake**
- **3 small EKS clusters**

### Intended interpretation
- The account is treated as a single bounded pilot assessment environment.
- The 2 VPCs are sufficient to prove cross-VPC scoping, ownership, and evidence routing.
- The data lake is treated as a critical shared service boundary that may drive control relevance and evidence expectations.
- The 3 EKS clusters are enough to prove multi-cluster ingestion, recurring findings, and per-cluster evidence/remediation workflows without exploding scope.

---

## Prototype objectives

The prototype must prove five things:

1. **The platform can ingest and normalize first-environment AWS signals reliably.**
2. **The platform can maintain evidence provenance and freshness for the bounded environment.**
3. **The platform can route findings into governed remediation workflows with human review.**
4. **The platform can generate a small, reviewable readiness snapshot for this environment.**
5. **The platform can do all of the above without implying autonomous compliance authority.**

---

## What is in scope

## 1. AWS-native telemetry for the bounded environment

### In scope
- AWS Security Hub findings
- AWS Config compliance/drift signals
- Amazon Inspector findings
- Amazon GuardDuty findings
- IAM Access Analyzer findings
- CloudTrail references sufficient for traceability and packet citation
- Systems Manager compliance/inventory **only if already enabled and useful in this environment**

### Why
These sources are already identified in the repo as the highest-value AWS-native inputs for the concept, and they are enough to prove the intake, normalization, and triage loop.

---

## 2. Asset and system coverage for the prototype

### In scope
- Account-level environment record
- 2 VPC records
- 3 EKS cluster records
- data-lake record as a named protected system/service boundary
- asset resolution sufficient to associate findings and evidence to:
  - account
  - VPC
  - cluster
  - shared service boundary where applicable

### Not required in the prototype
- exhaustive CMDB-quality modeling for every subordinate AWS resource
- universal asset identity resolution across every possible source system

---

## 3. Core workflow capabilities

### In scope
- finding intake and normalization
- deduplication/recurrence grouping at a practical MVP level
- human-reviewed control mapping proposals
- evidence cataloging and freshness status
- remediation / POA&M-like workflow records
- closure validation requiring evidence
- monthly or on-demand readiness snapshot for the prototype boundary
- packet manifest / export for review use
- policy-aware notifications and digests

### Why
This is the minimum useful operating loop described across the existing roadmap and architecture docs.

---

## 4. Operator surfaces

### In scope
- API and simple operator-facing service/UI endpoints for:
  - queue views
  - finding detail
  - evidence detail
  - remediation detail
  - snapshot / packet review
- enough UX to support real internal use by an ISSO and at least one reviewer/owner role

### Working assumption
The prototype may use a sparse internal UI and does **not** need polished product-grade front-end work.

---

## 5. Bounded AI use

### In scope
Only if retrieval, citation, and policy gates are already working:
- draft triage summary
- draft evidence-gap summary
- draft monthly readiness narrative
- draft packet summary text

### Strict constraints
- outputs are advisory drafts only
- every draft must carry citations
- no AI output becomes primary evidence
- no AI-driven automatic approval, closure, or exception handling

### Working assumption
AI may be feature-flagged or deferred if target-region/model availability or readiness is uncertain.

---

## Explicitly out of scope

The following are **not** part of this prototype MVP:

### Environment expansion out of scope
- multiple AWS accounts
- multi-organization / multi-tenant deployment
- broad on-prem live integrations
- enterprise-wide CMDB or universal identity reconciliation
- large-scale cross-enclave rollout

### Product/platform expansion out of scope
- autonomous remediation
- automatic control closure
- automatic exception approval
- final compliance adjudication
- broad “single compliance score” outputs
- rich assessor co-pilot features
- advanced contradiction analytics beyond obvious first-pass checks
- graph analytics / relationship engine
- OSCAL automation beyond future compatibility awareness
- deep ticket bi-directional synchronization
- full document-repository ingestion and semantic indexing of everything

### Operations hardening out of scope for MVP
- full HA/DR posture for a wide production rollout
- elaborate multi-region patterns
- massive dashboarding for every stakeholder type
- connector coverage beyond the bounded AWS-first set

---

## Prototype users

### Primary users
- ISSO / compliance analyst
- reviewer / compliance lead
- engineering or platform owner for remediation follow-up

### Secondary users
- security operations reviewer
- leadership consumer of periodic readiness summaries

### Not a target user for prototype
- external assessor using the platform directly as a daily tool
- general employee population
- external customers

---

## Prototype operating scenarios

The MVP should successfully support these scenarios:

1. **A new AWS finding appears** and is normalized, scoped to the right cluster/VPC/system, and routed into triage.
2. **A recurring finding reappears** and is linked to prior remediation history instead of creating meaningless duplicate noise.
3. **Evidence for a control area becomes stale** and the system opens a refresh workflow.
4. **A remediation item is marked ready for closure** and a human reviewer can verify closure against linked evidence.
5. **A monthly readiness snapshot is produced** for the bounded environment with evidence-backed summaries and clear open issues.

If the prototype cannot do those five things reliably, it is not ready to expand.

---

## MVP definition of done

The prototype is “done enough” when it can:
- ingest the selected AWS-native sources for the bounded environment
- preserve provenance back to source records
- maintain canonical records for findings, evidence, mappings, remediation items, and snapshots
- show stale evidence and overdue remediation clearly
- support human-reviewed triage and closure
- generate at least one reviewable readiness packet for this environment

---

## Verified facts vs working assumptions summary

### Verified facts
- The repo’s current architecture direction favors an EKS-hosted, evidence-grounded control plane with workflow, object-backed evidence storage, and bounded AI assistance.
- The first-environment planning model is explicitly scoped to **1 AWS account, 2 VPCs, 1 data lake, and 3 small EKS clusters**.
- Existing roadmap docs already treat human approval, provenance, and readiness packets as core parts of the intended platform.

### Working assumptions
- The prototype should prioritize AWS-native sources first and keep on-prem connectivity minimal for sprint 1.
- A sparse internal UI is sufficient for the prototype.
- AI drafting is optional and should remain behind feature flags until the rest of the stack is trustworthy.

---

## Bottom line

This prototype MVP is **not** “build Vigilant ISSO.”

It is:

**prove that Vigilant ISSO can operate credibly inside one bounded AWS environment with enough evidence discipline, workflow rigor, and snapshot usefulness that a real ISSO would trust it for daily readiness work.**
