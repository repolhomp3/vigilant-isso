# Architecture Options for Vigilant ISSO

## Goal

Design an ISSO operations system that is:
- evidence-grounded
- traceable
- credible to auditors and ISSOs
- practical in AWS GovCloud + on-prem environments
- conservative about automation rights

---

## Non-negotiable requirements

1. **Every consequential claim must point back to source evidence.**
2. **Control interpretations must be versioned and reviewable.**
3. **Machine data and human evidence must coexist.**
4. **AI should draft and correlate, not silently adjudicate compliance truth.**
5. **The architecture must tolerate GovCloud differences** such as partition/ARN differences and service availability realities. [1][2]

---

## Shared logical architecture

Regardless of implementation style, the platform should contain these layers:

### 1. Source connectors
- AWS GovCloud connectors for Security Hub, Config, GuardDuty, Inspector, Access Analyzer, CloudTrail, Systems Manager
- on-prem connectors for scanners, CMDB, IdP, ticketing, document repositories
- batch and event-driven ingestion

### 2. Normalization and correlation
- normalize findings into a canonical internal schema
- preserve raw source payloads
- compute identities for asset, control, system, owner, and observation window
- correlate duplicates and recurring issues

### 3. Evidence/state store
- findings and observations
- evidence objects and evidence collections
- controls/practices/catalog versions
- systems, enclaves, and scope boundaries
- remediation items / POA&Ms
- exceptions and accepted risk
- generated narratives and snapshots

### 4. Workflow engine
- evidence collection requests
- review/approval steps
- reminder and escalation logic
- remediation state transitions
- pre-audit readiness checks

### 5. Retrieval-grounded reasoning layer
- draft mappings, summaries, control narratives, and audit packets
- only on bounded, source-linked context
- never write authoritative status without a workflow event or human approval

### 6. Reporting and exports
- operator dashboard
- ISSO work queue
- control coverage views
- aging and staleness reports
- assessor support packet export

---

## Option A: Workflow-first control plane with AI as an analyst copilot

### Shape
- relational system of record
- object storage for artifacts
- queue/event bus for ingestion and workflow triggers
- LLM used only at decision support boundaries

### Why it is strong
- best trust posture
- easiest to audit
- easiest to limit blast radius
- easiest to explain to compliance stakeholders

### Recommended components
- **System of record:** Postgres or equivalent relational store
- **Artifact store:** S3-compatible/object storage
- **Search/RAG index:** vector + keyword index over approved evidence and control texts
- **Workflow engine:** Temporal, Step Functions equivalent, or durable job orchestration
- **Event bus:** EventBridge/Kafka/SQS-style pattern

### Best use case
First production version.

### Main weakness
Less “agentic” and less flashy; requires careful workflow design up front.

---

## Option B: Graph-centered knowledge fabric with deterministic workflow wrapper

### Shape
- property graph or graph overlay connects findings, controls, assets, evidence, owners, packages, and remediation items
- workflows still control state transitions
- AI uses graph traversals + source retrieval to answer control/evidence questions

### Why it is strong
- naturally models many-to-many mappings
- excellent for “show me everything relevant to control X in enclave Y over last 90 days”
- useful for contradiction detection and blast-radius analysis

### Best use case
Second-generation architecture after the canonical model is stable.

### Main weakness
Graph stores are great for relationships but can tempt over-modeling. The operational truth still needs a durable transactional system of record.

### Recommendation
Use a relational core plus graph projection rather than graph-only.

---

## Option C: Lakehouse / analytics-heavy compliance data platform

### Shape
- land raw findings/events/artifacts in a lakehouse
- run transformations into analytical models
- use BI/reporting and ML/LLM on top

### Why it is strong
- good for large-scale history and trend analysis
- good for enterprise rollups and audit season exports
- matches environments with lots of heterogeneous data sources

### Main weakness
- weaker transaction semantics for workflow-heavy remediation management
- more work to preserve authoritative operational states

### Best use case
Enterprise reporting backend, not the only core system.

---

## Recommended architecture

## Option A + selective graph projection

This is the most credible shape for Vigilant ISSO.

### Recommended stack pattern
1. **Transactional core**
   - systems, assets, controls, findings, evidence, remediation items, attestations, exceptions
2. **Raw evidence store**
   - immutable copies or references to source artifacts and payloads
3. **Graph projection**
   - derived relationships for control/evidence/finding traversal
4. **Workflow engine**
   - governs approvals, evidence requests, due dates, and closure criteria
5. **RAG/LLM layer**
   - drafts summaries and suggestions using only approved source-linked corpora
6. **Policy/guardrail layer**
   - distinguishes advisory, approval-gated, and automated actions

---

## Data flow

1. Ingest source events/findings/evidence metadata.
2. Preserve raw payload and source identifiers.
3. Normalize into canonical internal objects.
4. Correlate to assets, systems, control candidates, owners, and open POA&M items.
5. Trigger workflows:
   - create review task
   - request missing evidence
   - suggest control mappings
   - propose POA&M
   - mark stale evidence
6. Human reviews suggestions where required.
7. Approved state becomes visible in dashboards and audit-readiness packets.

---

## AI boundaries

### Safe uses
- summarize findings
- recommend likely control mappings
- draft remediation narratives
- identify missing/stale evidence
- prepare assessor packet drafts
- generate daily/weekly ISSO digests

### Approval-gated uses
- creating formal POA&M entries
- changing control coverage status
- attaching evidence to a control as “primary” support
- marking a finding as compensated or accepted risk
- closing a remediation item

### Avoid / defer
- direct infrastructure changes
- declaring compliance achieved
- deleting evidence or findings
- silently overriding source severity or status

---

## Multi-environment modeling requirements

The architecture must explicitly represent:
- system/enclave boundaries
- CUI scope boundaries
- inherited/shared services
- cloud vs on-prem deployment context
- account/subscription/site hierarchy
- ownership hierarchy
- assessment period and evidence period

This matters because CMMC/NIST evaluation depends heavily on scope and whether a control is implemented directly, inherited, or partially inherited.

---

## Credibility features that matter

1. **Provenance everywhere** — source system, object id, collection time, collector, hash where possible.
2. **Versioned control text** — mappings must be traceable to the catalog version used.
3. **Temporal history** — what was true when the audit period began, not just now.
4. **Human sign-off events** — preserve who accepted a mapping, closure, exception, or narrative.
5. **Immutable snapshots** — audit packet generation should create reproducible point-in-time views.

---

## Assumptions

1. Not every evidence object will be stored directly; some will remain external references with hashes and metadata.
2. Customers will accept advisory AI outputs if provenance and reviewer accountability are explicit.
3. Most customers will prefer limited integration rights over broad autonomous permissions.

---

## Sources

1. AWS, “What Is AWS GovCloud (US)?” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/whatis.html
2. AWS, “Amazon Resource Names (ARNs) in GovCloud (US) Regions.” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/using-govcloud-arns.html
3. AWS, “Introduction to AWS Security Hub CSPM.” https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
4. AWS, “What Is AWS Config?” https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
5. AWS, “What is Amazon GuardDuty?” https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html
6. AWS, “What is Amazon Inspector?” https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html
7. AWS, “Using IAM Access Analyzer.” https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
8. AWS, “What Is AWS CloudTrail?” https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
9. AWS, “AWS Systems Manager Compliance.” https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-compliance.html
10. NIST, “Open Security Controls Assessment Language (OSCAL).” https://pages.nist.gov/OSCAL/
