# Vigilant ISSO Architecture

## Purpose

This document describes the target technical architecture for **Vigilant ISSO** in enough detail that a professional technical writer, solution architect, or diagrammer can create:
- a **high-level architecture diagram**
- a **logical component diagram**
- a **data-flow diagram**
- a **deployment diagram**
- and, if needed, a **sequence diagram** for core workflows

This is not a marketing overview. It is a drawing-ready technical architecture description.

---

![Vigilant ISSO Architecture](./vigilant-isso.png)

The image above is the current technical architecture drawing derived from this document and should be kept aligned with future architectural revisions.

## 1. System Intent

Vigilant ISSO is an **evidence-grounded ISSO operations platform** for organizations operating in **AWS GovCloud and on-prem CMMC Level 2 environments**.

Its purpose is to help the organization:
- continuously assess control health after initial certification
- detect drift and evidence decay before they become audit pain
- correlate findings, evidence, controls, remediation, and exceptions
- drive repeatable ISSO workflows
- generate reviewable readiness snapshots and assessor support packets
- keep human reviewers in control of consequential compliance decisions

The platform is explicitly designed to prevent the common post-certification failure mode:

**the environment remains technically active, but evidence freshness, control discipline, and remediation rigor gradually decay until the next assessment becomes painful.**

---

## 2. Architecture Principles

### 2.1 Foundational principles

1. **Evidence before narrative**
   - the system stores and reasons over evidence, not just prose summaries
2. **Deterministic state before AI assistance**
   - workflow state, evidence provenance, control mappings, and approvals must be authoritative without depending on an LLM
3. **Human-in-the-loop for material decisions**
   - control-state changes, exceptions, major remediation closures, and packet publication require human review/approval
4. **Hybrid-environment aware**
   - the platform must handle AWS GovCloud and on-prem data sources in one operating model
5. **Policy-mediated outputs**
   - notifications, retrieval, packet generation, and AI drafting all run through policy rules
6. **Auditability by design**
   - every consequential action must be attributable, reviewable, and reconstructable

### 2.2 What the system is not

Vigilant ISSO is **not**:
- an autonomous compliance authority
- a system that declares “you are compliant” without review
- a broad infrastructure mutation engine
- a log platform replacing CloudTrail or SIEM tools
- an LLM front-end with weak provenance controls

---

## 3. Recommended Architecture Pattern

The recommended first serious implementation is:

**EKS-hosted application control plane + relational system of record + object-backed evidence store + queue/workflow backbone + retrieval/search layer + policy engine + bounded Bedrock reasoning layer + in-boundary notification services**

This pattern is preferred because it gives the system:
- strong state management
- portable application deployment
- clean service boundaries
- manageable integration patterns for cloud and on-prem sources
- controlled insertion points for AI drafting
- compatibility with a GovCloud-centered deployment model

---

## 4. Deployment Scope and Environment Model

## 4.1 Primary hosted location

The primary control plane is hosted in **AWS GovCloud**, using **Amazon EKS** as the application platform.

This hosted control plane contains:
- API and operator services
- workflow orchestration
- canonical state and evidence metadata services
- notification and digest services
- packet and reporting services
- retrieval and search components
- bounded AI orchestration services
- AWS-native connectors

## 4.2 On-prem presence

On-prem environments contribute data through:
- lightweight collectors
- import jobs
- scheduled exports
- API adapters where allowed
- document/evidence metadata importers

The preferred on-prem connectivity pattern is:
- minimal privilege
- explicit source identity
- durable retry behavior
- tightly controlled communication back to the hosted control plane

## 4.3 Hybrid model

The architecture assumes **hybrid scope** where evidence and findings may originate from:
- GovCloud-native services
- on-prem security tools
- ticketing systems
- document repositories
- manual compliance artifacts
- identity and asset systems

This is critical because real CMMC L2 operations are almost never purely cloud-native.

---

## 5. High-Level Logical Architecture

A diagrammer should think of the system in **eight major layers**:

1. **Source Systems Layer**
2. **Connector and Ingestion Layer**
3. **Normalization and Identity Layer**
4. **Core Control Plane / System of Record Layer**
5. **Evidence and Packet Layer**
6. **Search / Retrieval / AI Drafting Layer**
7. **Notification and Human Review Layer**
8. **Platform Operations and Observability Layer**

### 5.1 Suggested high-level diagram shape

A high-level diagram should show left-to-right or top-to-bottom flow like this:

**Source Systems** → **Connectors/Ingestion** → **Normalization/Correlation** → **Core State + Workflow** → **Evidence/Packets + Search/AI** → **Notifications / Human Review / Reports**

With **Platform Operations** and **Policy** shown as cross-cutting layers.

---

## 6. Source Systems Layer

This layer represents systems external to Vigilant ISSO that produce authoritative data.

### 6.1 AWS GovCloud source systems

Recommended source systems to depict:
- **AWS Security Hub CSPM**
- **AWS Config**
- **Amazon Inspector**
- **Amazon GuardDuty**
- **IAM Access Analyzer**
- **AWS CloudTrail** / CloudTrail references
- **AWS Systems Manager Compliance / Inventory**

### 6.2 On-prem / enterprise source systems

Recommended source systems to depict:
- vulnerability scanners
- asset inventory / CMDB
- patch and compliance tools
- identity/access review exports
- ticketing/ITSM systems
- SSP / policy / procedure repositories
- diagram/document repositories

### 6.3 Diagram note

In a diagram, these should appear **outside** the Vigilant ISSO boundary and feed data inward through connectors or importers.

---

## 7. Connector and Ingestion Layer

This layer acquires source data while preserving provenance.

## 7.1 Primary components

Recommended components to draw:
- `connector-runtime`
- `connector-securityhub`
- `connector-config`
- `connector-inspector`
- `connector-guardduty`
- `connector-accessanalyzer`
- `connector-cloudtrail-ref`
- `connector-ssm-compliance`
- `importer-vulnscan`
- `importer-asset-inventory`
- `importer-ticket-mirror`
- `importer-doc-metadata`

## 7.2 Responsibilities

This layer is responsible for:
- acquiring findings/evidence metadata/import payloads
- preserving source IDs and raw payload references
- stamping ingestion provenance
- retry and dead-letter handling
- handing off records for normalization and workflow triggers

## 7.3 Diagram guidance

A logical component diagram should show these as **worker-style services**, not user-facing apps.

If the tech writer creates a deployment diagram, these should sit in a **connectors namespace** and feed a queue or ingestion API.

---

## 8. Normalization and Identity Layer

This layer makes heterogeneous source data usable.

## 8.1 Primary components

Recommended components to draw:
- `normalization-worker`
- `asset-resolution-service`
- `dedupe-recurrence-worker`

## 8.2 Responsibilities

### `normalization-worker`
- converts source-native records into canonical internal finding/evidence/event objects
- preserves source references and source nuance
- adds initial scope hints and likely mapping hints

### `asset-resolution-service`
- reconciles cloud resources, on-prem hosts, asset inventory objects, and ticket references
- determines when different source records represent the same practical asset or system
- preserves match confidence and unresolved collisions

### `dedupe-recurrence-worker`
- groups duplicate or repeated findings
- identifies recurring issue families
- avoids governance noise without losing lineage

## 8.3 Diagram guidance

This layer should appear between the connector layer and the core system-of-record layer.

It is important to visually separate this from both:
- raw source acquisition
- authoritative workflow/state management

because normalization is where trust can be damaged if done poorly.

---

## 9. Core Control Plane / System of Record Layer

This is the authoritative center of the platform.

## 9.1 Primary components

Recommended components to draw:
- `api-service`
- `authz-service` or auth middleware
- `workflow-service`
- `record-service`
- `policy-service`
- `audit-event-service`
- `snapshot-service`

## 9.2 Responsibilities

### `api-service`
- serves operator and integration APIs
- provides UI/API entry point
- routes requests into the core control plane

### `authz-service`
- enforces role/scope-aware access
- controls who may review, approve, publish, or view specific evidence classes

### `workflow-service`
- manages state transitions for:
  - finding triage
  - evidence refresh
  - remediation lifecycle
  - exception review
  - monthly snapshots
  - packet approval workflows

### `record-service`
- acts as the coarse-grained authoritative domain service
- manages findings, evidence metadata, mappings, remediation items, exceptions, snapshots, and packet manifests
- persists canonical business state to the relational store

### `policy-service`
- centralizes action policy
- determines:
  - advisory vs approval-gated actions
  - retrieval eligibility
  - notification channel rules
  - AI drafting permissions
  - evidence visibility and redaction behavior

### `audit-event-service`
- records every consequential action:
  - approvals
  - workflow transitions
  - packet publication
  - model invocations
  - notification deliveries

### `snapshot-service`
- creates point-in-time readiness freezes
- supports monthly and pre-assessment packet generation
- preserves stable references for later packet output

## 9.3 Diagram guidance

In a high-level architecture diagram, this layer should be represented as the **central box** inside the Vigilant ISSO trust boundary.

In a logical diagram, this can be drawn as:
- a **coarse-grained modular monolith** initially, or
- a small cluster of tightly related services

The docs intentionally recommend a **coarse-grained initial posture** rather than a large microservice farm.

---

## 10. Data Stores Layer

The platform should show **three distinct persistent data classes**.

## 10.1 Relational system of record

### Purpose
Authoritative storage for structured state.

### Stores
- systems and enclaves
- assets and owners
- findings
- evidence metadata
- control/practice mappings
- remediation items / POA&M state
- exception records
- approvals and reviewer actions
- snapshots and packet manifests
- notifications and delivery state

### Diagram guidance
Show this as the **authoritative structured database** behind the record/workflow layer.

## 10.2 Object-backed evidence store

### Purpose
Preserve raw artifacts and immutable packet outputs.

### Stores
- raw JSON exports
- scanner imports
- configuration exports
- screenshots / PDFs / documents where retained
- packet bundles
- derivative narratives and generated summaries (clearly marked as derivative)

### Diagram guidance
Show this as separate from the relational store.

This separation matters because:
- raw evidence must not be confused with summaries
- packet outputs should be immutable
- derivative narratives are not primary evidence

## 10.3 Queue / event backbone

### Purpose
Decouple ingestion, workflows, notifications, indexing, and AI drafting.

### Uses
- connector handoff
- workflow triggers
- reminder and escalation jobs
- snapshot jobs
- indexing jobs
- digest generation

### Diagram guidance
Show this as a backbone connecting connectors, core services, notification workers, and optional AI workers.

---

## 11. Evidence and Packet Layer

This layer turns structured state into readiness artifacts.

## 11.1 Primary components

Recommended components to draw:
- `evidence-catalog-service`
- `freshness-engine`
- `mapping-service`
- `packet-service`
- `control-health-service` *(later phase or optional analytic layer)*

## 11.2 Responsibilities

### `evidence-catalog-service`
- registers evidence objects and metadata
- tracks provenance, source, collection method, hash/integrity, observed dates, and classification
- distinguishes primary evidence from derivative summaries

### `freshness-engine`
- evaluates freshness policies
- flags warning and stale evidence states
- triggers evidence recovery workflows

### `mapping-service`
- manages many-to-many relationships between findings, evidence, remediation items, and controls/practices
- preserves rationale, review status, versioning, and confidence

### `packet-service`
- assembles immutable packet manifests and exports
- supports assessor support packets and review packets
- only publishes packet outputs from approved snapshot state

### `control-health-service`
- computes multidimensional control-health signals
- should not be drawn as a fake compliance score engine
- should instead use dimensions such as:
  - evidence freshness
  - evidence strength
  - contradiction pressure
  - open findings pressure
  - remediation aging pressure
  - exception burden
  - narrative confidence

## 11.3 Diagram guidance

A technical writer should show this layer as downstream of authoritative state but upstream of reports and AI drafting.

This layer should be visually connected to both:
- the **object evidence store**
- the **snapshot/packet** processes

---

## 12. Search / Retrieval / AI Drafting Layer

This layer exists to improve operator understanding while staying bounded.

## 12.1 Primary components

Recommended components to draw:
- `search-indexer`
- `retrieval-service`
- optional `graph-projection-worker`
- `bedrock-orchestrator`
- `narrative-draft-service`
- `contradiction-analysis-service` *(later phase)*
- `assessor-question-prep-service` *(later phase)*

## 12.2 Responsibilities

### `search-indexer`
- projects approved metadata/text into searchable indexes
- supports operator retrieval and draft generation support

### `retrieval-service`
- assembles policy-approved context packs
- enforces corpus, scope, and classification constraints
- returns source citations and excerpts

### `graph-projection-worker`
- later-phase relationship analytics helper
- supports advanced traversal use cases
- not required in MVP

### `bedrock-orchestrator`
- centralizes all model access
- applies policy checks, prompt templates, citation packaging, and audit logging
- ensures model use is mediated, not ad hoc

### `narrative-draft-service`
- drafts:
  - control summaries
  - monthly readiness narratives
  - packet narrative text
  - evidence gap summaries
  - contradiction explanation drafts

### `contradiction-analysis-service`
- later-phase service
- compares SSP/policy claims, telemetry, findings, and prior narratives for mismatch patterns

### `assessor-question-prep-service`
- later-phase service
- drafts likely assessor questions and identifies weak evidence areas

## 12.3 Bedrock / Claude Sonnet 4.5 placement

This layer should be diagrammed as **bounded and optional**.

### Important design note
For drawing purposes, show:
- **Bedrock reasoning layer** as a service integration behind the `bedrock-orchestrator`
- **Claude Sonnet 4.5** as a **candidate model choice / working assumption**, not as a guaranteed region-verified dependency

### Why
The architecture intentionally treats model availability as region-dependent and subject to verification.

### What the AI layer may do
- synthesize evidence across approved sources
- draft narratives with citations
- identify likely contradictions
- prepare operator-friendly summaries

### What the AI layer may not do directly
- declare compliance
- silently close remediation
- approve exceptions
- delete or suppress evidence
- perform uncontrolled infrastructure mutation

---

## 13. Notification and Human Review Layer

This layer closes the operational loop.

## 13.1 Primary components

Recommended components to draw:
- `notification-service`
- `digest-worker`
- operator UI / dashboards
- reviewer approval interfaces
- Teams and email endpoints *(inside boundary assumptions apply)*

## 13.2 Responsibilities

### `notification-service`
- sends policy-approved Teams and email notifications
- routes alerts, review requests, stale evidence notices, and remediation escalations
- preserves delivery attempts and message provenance

### `digest-worker`
- aggregates daily and periodic issues into digest form
- reduces notification spam

### operator UI / dashboards
Should present:
- work queues
- control-health views
- evidence freshness views
- remediation aging
- exceptions nearing expiry
- monthly snapshot review surfaces

### reviewer approval interfaces
Must support:
- accepting/rejecting control mappings
- approving packet publication
- reviewing compensating controls and exceptions
- validating remediation closure

## 13.3 Diagram guidance

A high-level diagram should show this layer as the **human interaction layer** on the right side of the architecture.

Teams/email should be shown as **downstream delivery channels**, not as data stores.

---

## 14. Platform Operations and Observability Layer

This is the operational safety net.

## 14.1 Primary components

Recommended components to draw:
- `observability-stack`
- `job-scheduler`
- `backup-restore-runner`
- `admin-maintenance-tools`

## 14.2 Responsibilities

### `observability-stack`
- metrics, logs, traces, queue health, workflow health, connector health
- alerting on ingestion failures, latency spikes, dead-letter growth, and missed recurring jobs

### `job-scheduler`
- triggers recurring tasks:
  - evidence refresh checks
  - monthly snapshots
  - digest generation
  - connector sync cycles

### `backup-restore-runner`
- validates recoverability of relational state and packet manifests
- supports operational continuity

### `admin-maintenance-tools`
- schema migrations
- reprocessing support
- integrity checks
- controlled repair operations

## 14.3 Diagram guidance

Show this as a **cross-cutting platform layer**, supporting all namespaces/services.

---

## 15. Kubernetes Namespace and Deployment Topology

A deployment diagram should use these namespaces as the initial recommended topology:

- `vigilant-edge`
- `vigilant-core`
- `vigilant-connectors`
- `vigilant-search`
- `vigilant-notify`
- `vigilant-ai`
- `vigilant-ops`

### 15.1 Suggested deployment posture

#### `vigilant-edge`
- ingress
- API edge
- auth helpers

#### `vigilant-core`
- workflow service
- record service
- policy service
- snapshot service
- packet service

#### `vigilant-connectors`
- AWS-native connectors
- on-prem import workers
- normalization helpers

#### `vigilant-search`
- search indexer
- retrieval service
- optional graph workers

#### `vigilant-notify`
- notification service
- digest worker

#### `vigilant-ai`
- Bedrock orchestration
- narrative drafting
- later contradiction/question-prep workers

#### `vigilant-ops`
- observability
- scheduler
- backup/maintenance jobs

### 15.2 Diagram note

The deployment diagram should communicate that:
- connectors are independently failure-prone workers
- the core control plane is authoritative
- AI is segregated behind service boundaries
- notifications and ops are not part of the authoritative record

---

## 16. Core Data Flows

A technical writer should consider drawing at least **four sequence flows**.

## 16.1 Flow A — New finding ingestion

1. Source system emits or provides finding
2. Connector ingests finding
3. Raw payload reference preserved
4. Normalization worker creates canonical finding
5. Asset resolution links finding to asset/system
6. Workflow service opens triage/review task
7. Mapping service proposes control links
8. Notification service routes triage message
9. Optional AI draft summarizes finding with citations
10. Human reviewer confirms triage/action

## 16.2 Flow B — Evidence freshness decay

1. Freshness engine detects warning/stale evidence
2. Record state updated with freshness status
3. Workflow service creates evidence-refresh task
4. Notification service routes to control owner/ISSO queue
5. New evidence collected/imported
6. Evidence catalog updates references
7. Snapshot/control-health views update
8. Optional AI draft updates control summary
9. Human approves updated narrative if needed

## 16.3 Flow C — Remediation closure validation

1. Remediation item moves to pending validation
2. Validation evidence requested from authoritative source
3. Evidence catalog stores validation evidence reference
4. Workflow service checks closure criteria
5. Contradictions and recurrence checks run
6. Human reviewer approves or rejects closure
7. Audit event recorded

## 16.4 Flow D — Monthly readiness snapshot

1. Scheduler triggers monthly snapshot workflow
2. Snapshot service freezes point-in-time manifest
3. Packet service assembles evidence and linked state
4. Retrieval service assembles approved context packs
5. Bedrock layer drafts narrative and top-risk summary
6. Human reviewer approves/rejects packet
7. Immutable packet bundle stored in object store
8. Notification service sends summary digest

---

## 17. Trust Boundaries and Security Boundaries

A drawing should explicitly show **trust boundaries**.

## 17.1 Boundary A — External source systems
Outside the Vigilant ISSO application boundary.

## 17.2 Boundary B — Connector ingress boundary
Where source identities and payload provenance are established.

## 17.3 Boundary C — Core authoritative state boundary
Where workflow state, evidence metadata, mappings, and approvals become authoritative.

## 17.4 Boundary D — AI drafting boundary
Where approved retrieval context is transformed into draft outputs.

## 17.5 Boundary E — Notification boundary
Where constrained message content leaves the platform toward approved in-boundary channels.

These boundaries matter because the platform’s credibility depends on keeping:
- source truth
- canonical state
- derivative narratives
- and notifications

clearly separated.

---

## 18. Suggested Diagram Set for the Technical Writer

A pro tech writer should be able to produce at least these diagrams from this document:

### Diagram 1 — Executive / high-level architecture
Show:
- source systems
- connectors
- core platform on EKS
- evidence stores
- retrieval/AI layer
- notification channels
- human reviewers

### Diagram 2 — Logical component diagram
Show services grouped by namespace / layer.

### Diagram 3 — Deployment diagram
Show EKS namespaces, shared stores, queue backbone, and external integrations.

### Diagram 4 — New finding sequence
Sequence from source finding to triage/review/notification.

### Diagram 5 — Monthly readiness snapshot sequence
Sequence from scheduler to snapshot to packet to narrative draft to approval to immutable packet.

### Diagram 6 — Trust boundary diagram
Show where provenance is established and where human approval is required.

---

## 19. MVP Architecture Cut

The technical writer may want to distinguish between **MVP** and **later phases** in the drawing.

## 19.1 MVP components to highlight

Must include:
- API edge
- workflow service
- record service
- policy service
- audit-event service
- connector runtime
- first AWS connectors
- first on-prem importers
- normalization worker
- asset resolution service
- evidence catalog
- freshness engine
- mapping service
- remediation service
- notification service
- snapshot service
- packet service
- search indexer
- retrieval service
- observability
- relational DB
- object store
- queue backbone

## 19.2 Later-phase components to gray out or annotate

Later additions:
- graph projection worker
- advanced contradiction analysis
- assessor-question prep service
- richer OSCAL helpers
- more advanced live on-prem connectors
- broader analytics layers

---

## 20. Architecture Summary for Caption Use

If the tech writer needs a short caption for the main architecture drawing, use something like:

**Vigilant ISSO is an EKS-hosted, evidence-grounded control plane for hybrid GovCloud and on-prem CMMC L2 operations. It ingests authoritative findings and evidence, normalizes them into a canonical system of record, drives human-reviewed remediation and readiness workflows, and uses bounded retrieval and AI drafting to prepare source-cited summaries and audit-ready packets.**

---

## 21. Final Notes for the Technical Writer

### Emphasize these visually
- evidence provenance
- workflow/state authority
- human approval points
- separation of raw evidence from derivative narratives
- hybrid GovCloud/on-prem source model
- bounded AI layer, not AI at the center

### Avoid implying
- the model is the source of truth
- notifications are evidence storage
- packet publication is automatic
- every service is required on day one
- the initial design is a giant microservice mesh

### Best visual framing
Think of Vigilant ISSO as:

**a compliance and security operations control plane**

not:
- a dashboard
- a chatbot
- a SIEM
- or a ticketing clone

That framing will make the drawing much more accurate.
