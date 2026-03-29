# Vigilant ISSO EKS Service and Component Backlog

## Goal

Define the service/component backlog for building Vigilant ISSO on **Amazon EKS** as a disciplined control plane for hybrid **AWS GovCloud + on-prem CMMC Level 2** operations.

This is not a generic microservice wishlist. It is a sequencing-oriented backlog focused on:
- what must exist first
- what each component is responsible for
- what it integrates with
- what should remain out of scope until the core is trustworthy

---

## Verified facts

1. **Amazon EKS is a managed Kubernetes service**, and AWS manages the Kubernetes control plane. [1]
2. **Amazon EKS is available in AWS GovCloud (US), but some capabilities present in commercial regions are unavailable there**, including EKS on Fargate, Amazon Managed Service for Prometheus, EKS Anywhere, and EKS Hybrid Nodes. [2]
3. **AWS GovCloud uses distinct partitions and service assumptions** that must be reflected in IAM, ARNs, and integration code. [3]
4. **Amazon Bedrock is a managed service for foundation models, but model availability is region-specific.** Bedrock-backed services should therefore be optional behind clear feature gates until target-region availability is verified. [4]
5. **OSCAL supports machine-readable representations of SSPs, assessment plans/results, and POA&Ms**, making it relevant as an external interchange target rather than necessarily the internal runtime schema. [5]

---

## Working assumptions

1. The first serious build should use **a modular monolith or a small set of coarse-grained services**, not dozens of tiny services.
2. Connectors should be independently deployable workers, because they fail and evolve at different rates.
3. The first release should optimize for **durable workflow state and provenance**, not maximal throughput.
4. Search, AI, and analytics should be **consumers of authoritative state**, not alternate systems of record.
5. Policy enforcement should be centralized enough that notification, retrieval, and AI services do not each invent their own rules.

---

## Namespace model

Suggested namespaces, matching the existing architecture direction:

- `vigilant-edge` — ingress, API edge, auth helpers
- `vigilant-core` — system of record and workflow services
- `vigilant-connectors` — AWS and on-prem ingestion workers
- `vigilant-search` — indexing and retrieval services
- `vigilant-notify` — digesting and delivery workers
- `vigilant-ai` — Bedrock orchestration and guardrailed drafting
- `vigilant-ops` — observability, audit jobs, admin/support tasks

---

## Deployment sequence summary

### Wave 1 — platform minimum
Stand up first:
- ingress / API edge
- authn/authz layer
- workflow service
- case/finding/evidence/remediation core service
- relational database
- object storage conventions
- event/queue backbone
- audit/event logging

### Wave 2 — source connectivity
Add next:
- connector framework
- AWS-native connectors
- on-prem import adapters
- asset/identity resolution
- normalization and deduplication workers

### Wave 3 — operator usefulness
Add next:
- control mapping service
- evidence freshness service
- notification service
- snapshot and packet services
- search/retrieval service

### Wave 4 — bounded AI and advanced analytics
Add later:
- Bedrock orchestration
- contradiction analysis
- recurrence analytics
- graph projection
- OSCAL import/export helpers

That order matters. Building Wave 4 before Wave 2 or 3 would be compliance theater.

---

## Backlog by component area

## A. Platform and edge services

### A1. `api-service`
**Namespace:** `vigilant-edge`  
**Priority:** P0  
**Why it exists:** stable entry point for UI, operator tooling, and controlled machine access.

**Responsibilities**
- expose operator and admin APIs
- expose connector ingestion endpoints where pull-based integration is not used
- validate requests, scopes, and schemas
- attach request identity and trace IDs
- publish domain commands/events to the workflow/core layer

**Critical integrations**
- auth service / identity provider
- workflow service
- policy service
- audit-event service

**Sequencing notes**
- build before specialized services so APIs do not sprawl inconsistently
- keep it thin; do not bury business logic here

---

### A2. `authz-service` or auth middleware layer
**Namespace:** `vigilant-edge`  
**Priority:** P0

**Responsibilities**
- enforce role/scope-aware authorization
- map users and service identities to enclaves, systems, and functions
- support approval actions with stronger policy checks
- expose authz decisions for audit logs

**Critical integrations**
- enterprise IdP or IAM-backed auth path
- policy service
- audit-event service

**Sequencing notes**
- must exist before review/approval workflows are trusted
- may start as middleware plus policy rules, then evolve into its own service if needed

---

### A3. `audit-event-service`
**Namespace:** `vigilant-ops`  
**Priority:** P0

**Responsibilities**
- record API mutations, approvals, connector actions, packet publication, and model invocations
- preserve actor, timestamp, object references, and decision context
- support immutable or tamper-evident export patterns

**Critical integrations**
- every mutating service
- packet service
- Bedrock orchestration service

**Sequencing notes**
- should be introduced at the start, not retrofitted later

---

## B. Core control-plane services

### B1. `workflow-service`
**Namespace:** `vigilant-core`  
**Priority:** P0

**Responsibilities**
- manage durable state machines for triage, evidence refresh, remediation, exceptions, and packet review
- issue tasks, approvals, escalations, and review checkpoints
- enforce allowed transitions
- coordinate timers/SLAs and recurring review loops

**Critical integrations**
- all core domain services
- notification service
- policy service
- queue/event backbone

**Sequencing notes**
- this is one of the first services to build
- consequential state changes should route through it

---

### B2. `record-service` (coarse-grained domain core)
**Namespace:** `vigilant-core`  
**Priority:** P0

**Why coarse-grained first:** early-stage Vigilant ISSO probably benefits from a single authoritative domain service before splitting into many microservices.

**Responsibilities**
- store and manage findings, evidence metadata, assets, systems, mappings, remediation items, exceptions, snapshots, packet manifests, and review metadata
- enforce canonical schemas and referential integrity
- expose query APIs for operator views and downstream services

**Critical integrations**
- relational database
- object store references
- workflow service
- search projection workers

**Sequencing notes**
- start here instead of prematurely creating ten domain services
- split later only if scaling, team topology, or change velocity justify it

---

### B3. `policy-service`
**Namespace:** `vigilant-core`  
**Priority:** P0

**Responsibilities**
- define advisory vs approval-gated actions
- define who can view which evidence classes and scopes
- decide notification channel eligibility and redaction rules
- gate retrieval and AI prompt assembly by policy
- centralize approval rules for packet publication and exception handling

**Critical integrations**
- API edge
- workflow service
- retrieval service
- notification service
- Bedrock orchestration service

**Sequencing notes**
- build early; otherwise every service will hard-code its own policy logic

---

### B4. `snapshot-service`
**Namespace:** `vigilant-core`  
**Priority:** P1

**Responsibilities**
- create point-in-time readiness snapshots
- freeze referenced object versions and packet scope
- support monthly and pre-assessment review cycles
- expose deltas against prior snapshots

**Critical integrations**
- record service
- packet service
- retrieval service
- workflow service

**Sequencing notes**
- build after the record and workflow core are stable
- becomes very important once pilot operations begin

---

### B5. `packet-service`
**Namespace:** `vigilant-core`  
**Priority:** P1

**Responsibilities**
- assemble immutable packet manifests
- render exports for review/audit support
- preserve linkage between packet contents and source records
- publish approved packet versions to object storage

**Critical integrations**
- snapshot service
- evidence service capabilities within record service
- retrieval service
- audit-event service

**Sequencing notes**
- only useful after evidence and mappings are stable enough to trust

---

---

## C. Connector and ingestion backlog

### C1. `connector-runtime`
**Namespace:** `vigilant-connectors`  
**Priority:** P0

**Responsibilities**
- provide shared job execution, retries, secrets handling, connector health reporting, and dead-letter routing
- standardize pull/push connector patterns
- stamp source metadata and ingestion provenance

**Critical integrations**
- queue backbone
- record service
- audit-event service

**Sequencing notes**
- build this before individual connectors so they share reliability primitives

---

### C2. `connector-securityhub`
**Priority:** P0

**Responsibilities**
- ingest Security Hub findings and relevant metadata
- preserve ASFF/raw references
- trigger normalization and triage workflows

**Critical integrations**
- connector runtime
- normalization worker
- workflow service

**Sequencing notes**
- one of the first AWS-native connectors because it is high-value for posture aggregation

---

### C3. `connector-config`
**Priority:** P0

**Responsibilities**
- ingest Config rule/resource compliance state and history references
- support drift-oriented evidence and timeline reconstruction

**Critical integrations**
- connector runtime
- asset resolution
- evidence freshness logic

---

### C4. `connector-inspector`
**Priority:** P0

**Responsibilities**
- ingest vulnerability and exposure findings
- preserve remediation recommendation metadata
- correlate findings to assets and remediation items

---

### C5. `connector-guardduty`
**Priority:** P0

**Responsibilities**
- ingest threat findings
- support urgent workflow routing and risk summaries
- preserve detector/source context

---

### C6. `connector-accessanalyzer`
**Priority:** P1

**Responsibilities**
- ingest policy and access findings
- support identity/access review evidence and contradiction checks

---

### C7. `connector-cloudtrail-ref`
**Priority:** P1

**Responsibilities**
- ingest audit-reference metadata and query links rather than indiscriminately copying logs
- support packet citations and change reconstruction

**Sequencing notes**
- keep this reference-oriented; do not accidentally turn Vigilant ISSO into a log platform

---

### C8. `connector-ssm-compliance`
**Priority:** P1

**Responsibilities**
- ingest patch/compliance state where SSM is part of the environment
- support evidence freshness and remediation workflows

---

### C9. `importer-vulnscan`
**Priority:** P0

**Responsibilities**
- accept structured exports from on-prem vulnerability scanners
- normalize scanner-specific severities and identifiers
- correlate to assets and systems

**Sequencing notes**
- likely more realistic for MVP than deep live API integrations on day one

---

### C10. `importer-asset-inventory`
**Priority:** P0

**Responsibilities**
- import CMDB or asset inventory data
- seed ownership, system membership, and scoping data

---

### C11. `importer-ticket-mirror`
**Priority:** P1

**Responsibilities**
- mirror selected ticket/ITSM data for remediation status reconciliation
- avoid making the ticket system a hidden dependency for core state

---

### C12. `importer-doc-metadata`
**Priority:** P1

**Responsibilities**
- register policy, SSP, procedure, diagram, and review-record metadata
- support freshness, provenance, and retrieval without necessarily ingesting full document contents initially

---

## D. Normalization and identity workers

### D1. `normalization-worker`
**Namespace:** `vigilant-core` or `vigilant-connectors`  
**Priority:** P0

**Responsibilities**
- map source records into canonical finding/evidence/event shapes
- preserve source-native references and important source fields
- assign initial control hints and scope hints

**Critical integrations**
- connector runtime
- record service
- asset resolution service

---

### D2. `asset-resolution-service`
**Namespace:** `vigilant-core`  
**Priority:** P0

**Responsibilities**
- reconcile cloud resources, hosts, scanner targets, accounts, enclaves, and ownership
- support many-to-one identity linking with confidence/provenance
- expose match confidence and unresolved collisions

**Sequencing notes**
- absolutely critical; weak identity resolution poisons everything upstream

---

### D3. `dedupe-recurrence-worker`
**Namespace:** `vigilant-core`  
**Priority:** P1

**Responsibilities**
- group repeated or overlapping findings
- identify recurrence patterns across cycles
- preserve lineage so suppressed duplicates remain traceable

---

## E. Evidence and control services

### E1. `evidence-catalog-service`
**Namespace:** `vigilant-core`  
**Priority:** P0/P1 boundary

**Responsibilities**
- register evidence metadata and storage references
- manage provenance, hash/integrity metadata, collection timestamps, and evidence type classification
- distinguish primary evidence from derivative summaries

**Critical integrations**
- object storage
- record service
- packet service
- retrieval service

---

### E2. `freshness-engine`
**Namespace:** `vigilant-core`  
**Priority:** P1

**Responsibilities**
- evaluate freshness policies by evidence type/control/source
- open warning or stale tasks
- feed control-health views and notifications

---

### E3. `mapping-service`
**Namespace:** `vigilant-core`  
**Priority:** P1

**Responsibilities**
- store mappings between findings/evidence and 800-171/CMMC/internal control structures
- support many-to-many and versioned mappings
- record mapping rationale and approval state

**Sequencing notes**
- do not auto-finalize mappings without review

---

### E4. `control-health-service`
**Namespace:** `vigilant-core`  
**Priority:** P2

**Responsibilities**
- compute multi-dimensional health indicators for controls
- incorporate evidence completeness/freshness, open findings, remediation pressure, exception burden, and narrative confidence

**Sequencing notes**
- useful after evidence, mapping, and remediation primitives exist
- should never collapse into a fake one-number “compliance score” without qualification

---

## F. Remediation and exception services

### F1. `remediation-service`
**Namespace:** `vigilant-core`  
**Priority:** P1

**Responsibilities**
- manage remediation items and POA&M-like records
- track owner, due date, SLA, status, validation evidence, and closure history
- support mirrored ticket references where needed

**Critical integrations**
- workflow service
- notification service
- record service

---

### F2. `exception-service`
**Namespace:** `vigilant-core`  
**Priority:** P1

**Responsibilities**
- manage exceptions, waivers, and compensating controls
- record approvers, expiry dates, rationale, and linked evidence
- trigger renewal/review workflows before expiry

---

## G. Search, retrieval, and knowledge services

### G1. `search-indexer`
**Namespace:** `vigilant-search`  
**Priority:** P1

**Responsibilities**
- project approved metadata and text into search indexes
- support keyword retrieval over findings, evidence metadata, controls, and packet outputs

---

### G2. `retrieval-service`
**Namespace:** `vigilant-search`  
**Priority:** P1

**Responsibilities**
- assemble policy-approved context packs for operators and AI workflows
- return citations, excerpts, and source references
- enforce scope/classification and corpus allowlists

**Critical integrations**
- policy service
- search indexer
- record service
- Bedrock orchestration service

**Sequencing notes**
- build before AI drafting; otherwise prompt assembly will become ad hoc and unsafe

---

### G3. `graph-projection-worker`
**Namespace:** `vigilant-search` or `vigilant-core`  
**Priority:** P3

**Responsibilities**
- build relationship-heavy projections for advanced traversal and analytics
- support questions like “which controls and packets are touched by this recurring asset issue?”

**Sequencing notes**
- later-phase optimization, not MVP critical path

---

## H. Notification and operator loop services

### H1. `notification-service`
**Namespace:** `vigilant-notify`  
**Priority:** P1

**Responsibilities**
- send policy-approved Teams/email notifications and digests
- avoid over-notification through batching and suppression rules
- preserve delivery attempts and message provenance

**Critical integrations**
- workflow service
- policy service
- remediation service
- freshness engine

---

### H2. `digest-worker`
**Namespace:** `vigilant-notify`  
**Priority:** P2

**Responsibilities**
- aggregate daily hygiene, overdue work, stale evidence, and control-owner actions into digest outputs
- reduce noisy one-event-at-a-time messaging

---

## I. AI and bounded reasoning services

### I1. `bedrock-orchestrator`
**Namespace:** `vigilant-ai`  
**Priority:** P2

**Responsibilities**
- broker all model requests through one controlled path
- apply prompt templates, model selection, citation packaging, and policy checks
- record model/version, prompt references, and output metadata

**Critical integrations**
- policy service
- retrieval service
- audit-event service
- packet service

**Sequencing notes**
- do not build until retrieval and citations are already reliable
- keep feature-flagged until Bedrock availability is validated in target environment

---

### I2. `narrative-draft-service`
**Namespace:** `vigilant-ai`  
**Priority:** P2

**Responsibilities**
- draft control summaries, monthly snapshots, and packet narratives
- clearly label verified facts, inferences, open questions, and recommendations

---

### I3. `contradiction-analysis-service`
**Namespace:** `vigilant-ai`  
**Priority:** P3

**Responsibilities**
- compare SSP/policy claims against telemetry, evidence freshness, exceptions, and historical narratives
- propose contradiction findings for human review

---

### I4. `assessor-question-prep-service`
**Namespace:** `vigilant-ai`  
**Priority:** P3

**Responsibilities**
- draft likely assessor follow-up questions for weak or contradictory controls
- identify missing evidence likely to be challenged

---

## J. Ops and reliability services

### J1. `job-scheduler`
**Namespace:** `vigilant-ops`  
**Priority:** P1

**Responsibilities**
- trigger recurring evidence refresh, monthly snapshots, digest jobs, and connector syncs
- coordinate with workflow service rather than directly mutating state

---

### J2. `observability-stack`
**Namespace:** `vigilant-ops`  
**Priority:** P0

**Responsibilities**
- metrics, logs, traces, queue health, job health, and connector health
- alert on ingest failures, dead-letter growth, latency spikes, and missed recurring jobs

**Sequencing notes**
- because Amazon Managed Service for Prometheus is unavailable in GovCloud for EKS, plan for a self-managed or otherwise GovCloud-supported observability pattern. [2]

---

### J3. `backup-restore-runner`
**Namespace:** `vigilant-ops`  
**Priority:** P0/P1

**Responsibilities**
- coordinate tested backups/restores for relational data and packet manifests
- validate recovery procedures for pilot and production environments

---

### J4. `admin-maintenance-tools`
**Namespace:** `vigilant-ops`  
**Priority:** P2

**Responsibilities**
- controlled reprocessing of failed ingests
- schema migrations and repair workflows
- integrity verification and support tooling

---

## Data stores and infrastructure components

These are not application services, but they are backlog-critical dependencies.

### S1. Relational system of record
**Priority:** P0

**Responsibilities**
- authoritative store for systems, assets, findings, evidence metadata, mappings, remediation, exceptions, snapshots, and approvals

---

### S2. Object-backed evidence store
**Priority:** P0

**Responsibilities**
- store raw artifacts, packet exports, derivative narratives, and immutable manifests
- preserve hash/provenance and separation of raw vs derived content

---

### S3. Queue / event backbone
**Priority:** P0

**Responsibilities**
- decouple ingestion, workflows, notifications, and indexing
- support retries, dead-letter queues, and idempotent processing

---

### S4. Search index
**Priority:** P1

**Responsibilities**
- support keyword retrieval over approved corpora

---

### S5. Optional vector index
**Priority:** P2

**Responsibilities**
- support retrieval over approved text corpora for bounded AI use

**Sequencing notes**
- optional until narrative drafting needs it

---

## Recommended MVP service cut

If building the first usable version, do **not** implement the whole backlog at once.

### MVP application services
- `api-service`
- auth middleware / `authz-service`
- `audit-event-service`
- `workflow-service`
- `record-service`
- `policy-service`
- `connector-runtime`
- `connector-securityhub`
- `connector-config`
- `connector-inspector`
- `connector-guardduty`
- `importer-vulnscan`
- `importer-asset-inventory`
- `normalization-worker`
- `asset-resolution-service`
- `evidence-catalog-service`
- `freshness-engine`
- `mapping-service`
- `remediation-service`
- `notification-service`
- `snapshot-service`
- `packet-service`
- `search-indexer`
- `retrieval-service`
- `observability-stack`

### Defer until after MVP trust is proven
- `graph-projection-worker`
- `control-health-service` as a sophisticated analytics layer
- `bedrock-orchestrator` and AI drafting services
- `contradiction-analysis-service`
- `assessor-question-prep-service`
- rich live ticket sync beyond a mirror pattern
- full OSCAL import/export automation

---

## Critical integration notes

### 1. Policy service is the hinge point
If policy is distributed across API handlers, notification workers, and AI orchestration code, the platform will drift into inconsistent behavior. Centralize it.

### 2. Asset resolution is more important than fancy analytics
If the system cannot reliably tell that a scanner host, EC2 instance, Config resource, and ticket item refer to the same practical asset/system, everything above it gets shaky.

### 3. Packet service should consume approved snapshot state
Do not let packet generation re-query “live” mutable state at export time without a frozen manifest.

### 4. AI must consume retrieval, not bypass it
No direct free-form model access to raw stores. Retrieval and citation packaging should be mandatory.

### 5. Notification service must not become the workflow engine
Notifications should reflect workflow state, not decide it.

---

## Risks if component boundaries are wrong

### Too many services too early
- slow delivery
- integration thrash
- duplicated authorization and audit logic
- fragile deployments for a still-changing domain model

### Too little separation in connectors and AI
- source-specific bugs contaminate core state
- unsafe prompt assembly
- hard-to-audit model interactions
- difficult incident isolation

### Weak sequencing
- shiny summaries before trustworthy evidence
- packet generation before snapshot integrity
- notification spam before ownership models are stable

---

## Suggested backlog order

1. platform baseline + stores + audit
2. workflow + record + policy core
3. connector runtime + first AWS connectors + first on-prem importers
4. normalization + asset resolution
5. evidence catalog + freshness + mappings
6. remediation + notifications
7. snapshots + packets + retrieval/search
8. AI orchestration and later analytics

That is the path from concept to a serious platform.

---

## Sources

1. AWS, “What is Amazon EKS?” https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html
2. AWS, “Amazon Elastic Kubernetes Service in AWS GovCloud (US).” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-eks.html
3. AWS, “AWS GovCloud (US).” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/
4. AWS, “Overview - Amazon Bedrock.” https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html
5. NIST, “Open Security Controls Assessment Language (OSCAL).” https://pages.nist.gov/OSCAL/
6. AWS, “Amazon EKS Best Practices Guide.” https://docs.aws.amazon.com/eks/latest/best-practices/introduction.html
