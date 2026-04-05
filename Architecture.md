# Vigilant ISSO Architecture

## Purpose

This document describes the **updated target technical architecture** for Vigilant ISSO in enough detail that a professional technical writer, solution architect, or diagrammer can produce:
- a high-level architecture diagram
- a logical component diagram
- a deployment diagram
- sequence/data-flow diagrams
- and a trust-boundary view

This version reflects the current direction:

**Vigilant ISSO is a lean, EKS-hosted compliance-operations control layer that uses MCP-style server/client components to interact with AWS-native services, bounded Bedrock reasoning, and a small persistent evidence/workflow core.**

It is no longer described as a heavy platform fielded across the monitored environment. Instead, it is a **small monitoring/orchestration system** pointed at a target AWS environment.

---

![Vigilant ISSO Architecture](./vigilant-isso-2.png)

The image above is the current architecture drawing derived from this document and reflects the revised lean architecture described here.

---

## 1. System intent

Vigilant ISSO is an **evidence-grounded ISSO operations service** for helping a team stay compliant after initial CMMC Level 2 certification.

Its primary mission is to:
- continuously assess the health of a monitored AWS environment
- detect drift and evidence decay
- correlate findings to controls and owners
- route remediation and review work
- generate readiness outputs and monthly snapshots
- support human ISSOs with bounded AI summaries and playbooks

### Important framing
The AWS account, VPCs, data lake, and EKS clusters being discussed are the **environment being monitored**, not the infrastructure footprint Vigilant ISSO itself necessarily requires.

That distinction drives the new design.

---

## 2. Architecture principles

These principles align with NIST guidance (SP 800-53, SP 800-171), AWS Well-Architected Framework pillars (Security, Reliability, Operational Excellence, Cost Optimization), and practical ISSO/CISO discipline.

1. **Lean control layer, not giant platform**
   - use the minimum persistent platform required to maintain trustworthy workflow and evidence state
   - *AWS Well-Architected: Cost Optimization — right-size resources and avoid over-provisioning*

2. **AWS-native first**
   - rely on AWS-native security/compliance sources wherever possible rather than copying or recreating their functions
   - *AWS Well-Architected: Security — use AWS-managed services for defense in depth; Operational Excellence — reduce operational burden through managed services*

3. **MCP-mediated service interactions**
   - use EKS-hosted MCP-style server/client components to query and operate on approved service surfaces
   - *NIST SP 800-53 AC-4, SC-7: enforce information flow control and boundary protection through defined interfaces*

4. **Evidence before narrative**
   - authoritative state comes from findings, evidence metadata, workflow records, and approval history
   - *NIST SP 800-171 3.3.1, 3.3.2: create and retain system audit logs; ISSO discipline: evidence provenance is the foundation of audit defense*

5. **AI as bounded reasoning, not source of truth**
   - Bedrock is used for summarization, mapping drafts, readiness narratives, and playbook assistance only
   - *CISO responsibility: AI-assisted outputs require human validation before consequential decisions*

6. **Human-in-the-loop for consequential compliance decisions**
   - no automatic compliance declarations, closure approvals, or exception approvals
   - *NIST SP 800-53 AU-6, CA-7: human review of audit records and continuous monitoring outputs; ISSO accountability for attestations*

7. **Cheap by design**
   - constrain persistent infrastructure, avoid unnecessary always-on components, and keep the first version narrow
   - *AWS Well-Architected: Cost Optimization — implement expenditure awareness, match supply to demand*

8. **Least privilege for integrations**
   - MCP components and service accounts use scoped IAM roles with minimum required permissions
   - *NIST SP 800-53 AC-6, NIST SP 800-171 3.1.5-3.1.8: least privilege and separation of duties*

9. **Defense in depth**
   - layered security controls across network, identity, application, and data tiers
   - *AWS Well-Architected: Security — apply security at all layers; NIST SP 800-53 SC-7, PL-8*

10. **Operational visibility and resilience**
    - infrastructure and application telemetry enable rapid incident detection and recovery
    - *AWS Well-Architected: Reliability — design for failure recovery; Operational Excellence — anticipate and respond to operational events*

---

## 3. Target deployment model

## 3.1 Where Vigilant ISSO runs
Vigilant ISSO runs as a **small EKS-hosted control service** in AWS GovCloud.

This deployment hosts:
- a small orchestrator/control API
- MCP servers and MCP-capable workers for AWS service access
- a lightweight workflow/evidence state service
- notification services
- optional retrieval and Bedrock orchestration components

## 3.2 What Vigilant ISSO monitors
The first monitored environment is:
- **1 AWS account**
- **2 VPCs**
- **1 data lake**
- **3 small EKS clusters**

This is the **target environment under observation**, not the minimum infra required to host Vigilant ISSO itself.

## 3.3 What remains outside the platform boundary
These remain external authoritative sources:
- AWS Security Hub
- AWS Config
- Amazon Inspector
- Amazon GuardDuty
- IAM Access Analyzer
- CloudTrail references
- selected document/evidence sources
- Teams/email delivery endpoints

---

## 4. Recommended architecture pattern

The recommended first implementation is:

**small EKS deployment + MCP server/client layer + lightweight workflow/evidence store + object-backed evidence storage + policy-aware notifications + optional bounded Bedrock reasoning**

### Why this pattern
This pattern is preferred because it:
- keeps cost low
- preserves AWS-native control surfaces
- reduces platform sprawl
- enables targeted automation and playbooks
- still preserves authoritative workflow/evidence state where it matters

### What it avoids
- large multi-service compliance platform overhead
- broad microservice complexity
- unnecessary duplication of AWS-native services
- expensive always-on analytics and indexing layers before they are needed

---

## 5. High-level logical architecture

A tech writer should think of this architecture in **six layers**:

1. **Monitored AWS Environment**
2. **MCP Access Layer**
3. **Lean Vigilant ISSO Control Layer**
4. **Evidence and Workflow State Layer**
5. **Optional AI / Retrieval Layer**
6. **Notification and Human Review Layer**

### High-level diagram shape
A simple high-level drawing should show:

**Monitored AWS Services** → **MCP Servers/Clients in EKS** → **Vigilant ISSO Control Service** → **Workflow/Evidence State + Object Store** → **Notifications / Human Review**

With **Bedrock** shown as an optional side integration behind policy and retrieval gates.

---

## 6. Monitored AWS environment layer

This layer is the environment under observation.

### Depict these monitored elements
- AWS account boundary
- two VPCs
- one data lake boundary/service
- three small EKS clusters
- AWS-native security/compliance service sources

### AWS-native source systems to show
- Security Hub
- Config
- Inspector
- GuardDuty
- Access Analyzer
- CloudTrail references
- Systems Manager compliance/inventory if used

### Diagram note
These should be drawn as **external monitored systems**, not as parts of the Vigilant ISSO runtime.

---

## 7. MCP access layer

This is the key architectural change.

Rather than building a large dedicated connector platform first, Vigilant ISSO should use **MCP-style pods/services** to provide bounded access to monitored AWS services.

## 7.1 Primary MCP components
Recommended MCP servers/clients to depict:
- `mcp-securityhub`
- `mcp-config`
- `mcp-inspector`
- `mcp-guardduty`
- `mcp-access-analyzer`
- `mcp-cloudtrail-ref`
- `mcp-evidence-store`
- `mcp-notify`
- optional `mcp-doc-metadata`
- optional `mcp-bedrock`

## 7.2 Responsibilities
Each MCP component should:
- expose a bounded tool surface to the control layer
- query or retrieve approved source data
- return structured outputs with provenance
- avoid becoming a free-form privileged superclient

## 7.3 Why MCP matters here
MCP lets the platform stay lean by treating integrations as:
- explicit tool surfaces
- bounded service interactions
- reusable pods/components

instead of forcing a heavier bespoke integration platform on day one.

### Diagram guidance
A logical component diagram should show these MCP services as a **thin service/tool interaction layer** between the monitored AWS environment and the Vigilant ISSO control layer.

---

## 8. Lean Vigilant ISSO control layer

This is the core of the system.

## 8.1 Primary components
Recommended components to draw:
- `vigilant-api`
- `workflow-engine`
- `evidence-state-service`
- `policy-engine`
- `snapshot-service`
- `playbook-service`
- `audit-event-service`

## 8.2 Responsibilities

### `vigilant-api`
- serves operator/API entry points
- coordinates requests into the core logic
- mediates between UI/humans and workflow state

### `workflow-engine`
- manages triage, evidence refresh, remediation, monthly snapshot, and closure validation flows
- controls what becomes a task, review item, escalation, or approval point

### `evidence-state-service`
- stores findings metadata, evidence metadata, mappings, remediation items, exceptions, and readiness snapshots
- acts as the minimum authoritative system of record

### `policy-engine`
- centralizes rules for notification, retrieval, AI drafting, review requirements, and action gating

### `snapshot-service`
- creates point-in-time readiness views and packet manifests

### `playbook-service`
- assembles interactive playbook context for ISSOs and reviewers
- does not replace workflow state, but uses it

### `audit-event-service`
- records operator actions, approvals, workflow transitions, and AI invocations

### Diagram note
This should be shown as a **small central control plane**, not a huge application mesh.

---

## 9. Evidence and workflow state layer

This layer should be depicted separately from MCP services and separately from AI.

## 9.1 Structured state store
Use a small relational store for:
- findings metadata
- evidence metadata
- mappings
- remediation records
- approval/review state
- exception records
- monthly snapshots
- notification history

## 9.2 Object-backed evidence store
Use object storage for:
- raw exported evidence
- packet bundles
- derivative summaries
- snapshot artifacts

### Important separation
The drawing should make clear that:
- structured workflow state is authoritative for process truth
- object storage holds raw/derived artifacts
- derivative narratives are not the same as evidence

---

## 10. Optional AI / retrieval layer

The AI layer is now explicitly **optional and bounded**.

## 10.1 Primary components
Recommended components to depict if enabled:
- `retrieval-service`
- `bedrock-orchestrator`
- `narrative-draft-service`
- optional `playbook-assistant`

## 10.2 Responsibilities

### `retrieval-service`
- assembles policy-approved context for summaries and playbooks
- returns citations and evidence references

### `bedrock-orchestrator`
- centralizes model access
- ensures prompts are policy-bounded and auditable
- applies model feature flags and region checks

### `narrative-draft-service`
- drafts monthly readiness summaries
- drafts evidence-gap summaries
- drafts triage summaries and playbook context

### `playbook-assistant`
- helps explain degraded controls, recurring findings, and packet gaps
- always returns source-backed draft outputs for human review

## 10.3 Model placement
For drawing purposes:
- show **Amazon Bedrock** as the managed reasoning provider
- show **Claude Sonnet 4.5** only as a **candidate model assumption**, not guaranteed in-region fact

## 10.4 What AI must not do
- declare compliance
- auto-close remediation
- approve exceptions
- suppress evidence
- silently rewrite packet truth

---

## 11. Notification and human review layer

This layer closes the loop with humans.

## 11.1 Primary components
Recommended components to draw:
- `notification-service`
- `digest-service`
- operator UI / work queue
- reviewer approval interface
- Teams/email endpoints inside the boundary

## 11.2 Responsibilities
- send concise Teams/email notifications
- batch and digest lower-urgency items
- show actionable queue items to ISSOs and reviewers
- support review/approval for mapping, closure, and snapshot publication

### Diagram note
Notifications should be shown as **downstream channels**, not as system-of-record stores.

---

## 12. EKS topology

This architecture can be deployed very leanly.

## 12.1 Recommended first topology
A technical drawing should show **one small EKS-hosted Vigilant ISSO runtime** rather than implying three hosting clusters are required for the platform itself.

Within that runtime, depict:
- `vigilant-core` namespace or service group
- `vigilant-mcp` namespace or service group
- `vigilant-notify` namespace or service group
- optional `vigilant-ai` namespace or service group
- optional `vigilant-ops` namespace or service group

## 12.2 Why this changed
Earlier docs explored a heavier multi-cluster topology. The current direction is leaner:
- the three small EKS clusters are primarily **targets being monitored**
- Vigilant ISSO itself should start with a much smaller hosting footprint

That distinction should be explicit in the new diagram.

---

## 13. Core data flows

A tech writer should consider drawing these four flows.

## Flow A — Finding assessment
1. Security Hub / Config / Inspector / GuardDuty / Access Analyzer provides finding
2. Relevant MCP service retrieves or queries the finding
3. Control layer normalizes and stores finding metadata
4. Policy engine decides routing/notification
5. Workflow engine opens triage/remediation work
6. Human reviews and routes the work

## Flow B — Evidence freshness check
1. Scheduled workflow checks evidence freshness
2. Evidence-state service marks stale or warning status
3. Workflow engine opens refresh task
4. Notification service routes to owner/ISSO
5. New evidence is linked and validated

## Flow C — Closure validation
1. Remediation is marked pending closure
2. MCP services retrieve validation data from AWS-native sources
3. Evidence-state service links validation references
4. Reviewer approves or rejects closure
5. Audit event is recorded

## Flow D — Monthly readiness snapshot
1. Scheduler triggers snapshot
2. Snapshot service freezes current state references
3. Packet/export bundle is assembled
4. Optional retrieval + Bedrock draft summary is created
5. Human reviews and approves summary/packet output
6. Notification sends monthly readiness digest

---

## 14. Trust boundaries

A new architecture drawing should explicitly show these boundaries:

### Boundary A — Monitored environment boundary
The AWS account / VPCs / data lake / monitored EKS clusters

### Boundary B — MCP access boundary
Where bounded service access occurs

### Boundary C — Vigilant ISSO control boundary
Where workflow, evidence metadata, mappings, and approvals become authoritative

### Boundary D — AI drafting boundary
Where approved retrieval context is turned into draft narrative outputs

### Boundary E — Notification boundary
Where limited operational summaries are delivered to Teams/email

These boundaries are critical because they make the design understandable to both engineers and compliance reviewers.

---

## 15. Suggested diagram set for the technical writer

The writer/diagrammer should be able to create:

### Diagram 1 — High-level architecture
Show:
- monitored AWS environment
- MCP access layer
- Vigilant ISSO control layer
- evidence/workflow stores
- optional AI/retrieval layer
- Teams/email and human reviewers

### Diagram 2 — Logical component diagram
Show the small service set and MCP components.

### Diagram 3 — Deployment diagram
Show the lean EKS hosting footprint for Vigilant ISSO and the monitored environment separately.

### Diagram 4 — Finding-to-workflow sequence
Show source → MCP → workflow → remediation/notification path.

### Diagram 5 — Monthly snapshot sequence
Show snapshot → packet → optional AI summary → approval → notification.

### Diagram 6 — Trust boundary diagram
Show why monitored infrastructure and platform infrastructure are different concepts.

---

## 16. MVP cut for the updated direction

The MVP architecture should emphasize:
- small EKS deployment for Vigilant ISSO itself
- MCP services for AWS-native integrations
- one lightweight workflow/evidence store
- object-backed evidence storage
- notification service
- monthly snapshots
- optional, feature-flagged Bedrock summarization

### Defer
- heavy search/indexing infrastructure
- graph analytics
- broad document ingestion
- large-scale multi-cluster hosting patterns for Vigilant ISSO itself
- advanced assessor copilot features

---

## 17. Caption-ready summary

If the tech writer needs a short caption for the new main drawing, use:

**Vigilant ISSO is a lean EKS-hosted compliance-operations control layer that uses MCP-style service access to query AWS-native security and compliance sources, stores only the workflow and evidence state it must preserve, and optionally uses bounded Bedrock reasoning to prepare source-backed summaries for human ISSOs and reviewers.**

---

## 18. Final notes for the technical writer

### Visually emphasize
- monitored environment vs platform footprint
- MCP access layer as the integration mechanism
- small central workflow/evidence state core
- object-backed evidence storage
- optional AI layer behind policy and retrieval
- human approval points

### Avoid implying
- Vigilant ISSO is fielded across all monitored clusters as a large mesh
- Bedrock is always required
- AI is the source of compliance truth
- every monitored service requires a heavyweight custom connector

### Best visual framing
Think of Vigilant ISSO as:

**a small compliance-control and orchestration service pointed at a monitored AWS environment**

not:
- a giant platform fielded everywhere
- a SIEM replacement
- or an autonomous compliance engine

That framing should drive the new architecture drawing.
