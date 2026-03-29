# EKS-Hosted Architecture and Component Model

## Goal

Define a concrete EKS-hosted deployment model for Vigilant ISSO in a hybrid **AWS GovCloud + on-prem** environment, including where **Amazon Bedrock with Claude Sonnet 4.5** fits as a bounded reasoning/drafting layer.

This architecture is intentionally conservative:
- EKS is the application control plane, not the compliance authority
- workflow state and evidence provenance are first-class
- AI is a bounded drafting/reasoning subsystem behind policy gates
- the design must survive GovCloud service differences and hybrid-environment realities

---

## Verified facts

1. Amazon EKS is a managed Kubernetes service in which AWS manages the Kubernetes control plane. [1]
2. Amazon EKS is available in AWS GovCloud (US), but some capabilities available in commercial regions are not available there, including EKS on Fargate, Amazon Managed Service for Prometheus, EKS Anywhere, and EKS Hybrid Nodes. [2]
3. AWS GovCloud uses distinct partitions and service realities, which must be reflected in resource identifiers, integrations, and deployment assumptions. [3]
4. Amazon Bedrock is a fully managed service that provides access to foundation models; however, model availability is region- and service-specific and must be verified in the target environment. [4]
5. AWS Config, CloudTrail, Security Hub, Inspector, GuardDuty, Access Analyzer, and Systems Manager provide core telemetry sources needed for the Vigilant ISSO concept. [5][6][7][8][9][10][11]

---

## Architecture position

The best first serious architecture is:

**EKS-hosted workflow and API platform + relational system of record + object-backed evidence store + search/retrieval layer + bounded Bedrock reasoning services + policy-enforced human approval gates.**

This gives Vigilant ISSO:
- durable workflows
- isolation between deterministic state changes and AI-generated drafts
- good support for connectors and APIs
- portable service decomposition
- a clean boundary for evidence handling and review

---

## High-level deployment model

## Region and environment split

### AWS GovCloud (primary hosted control plane)
Host the main Vigilant ISSO application services in GovCloud when the organization wants the core operations platform close to the regulated cloud boundary.

Host here:
- EKS cluster(s)
- application APIs
- workflow services
- connector workers for AWS-native sources
- evidence metadata services
- object storage for approved evidence/artifacts
- reporting and packet generation services
- search/retrieval services
- policy engine

### On-prem environment
Run lightweight collectors/connectors or approved export jobs for:
- vulnerability scanners
- patch/compliance tools
- identity sources
- CMDB / asset inventory
- document repositories
- local log or configuration exports where direct API access is not possible

Preferred pattern:
- outbound-only or tightly controlled connector communication to the hosted control plane
- minimal privileges
- durable queue / retry behavior

---

## Core components

## 1. Ingress and API layer

### Components
- EKS ingress controller
- API gateway / internal API service
- authentication and authorization middleware
- tenant / enclave / scope-aware routing

### Responsibilities
- operator UI/API access
- connector ingestion endpoints
- workflow interaction endpoints
- packet/export requests
- notification dispatch requests

### Notes
- enforce least privilege and strong identity controls
- separate user-facing APIs from machine-ingestion paths
- preserve source-system identity on all connector submissions

---

## 2. Workflow orchestration layer

### Components
- workflow engine service
- task scheduler / durable job queue
- review-state manager

### Responsibilities
- evidence refresh tasks
- finding triage workflows
- remediation state machine
- exception review workflows
- monthly snapshot generation
- pre-assessment packet assembly

### Design rule
All consequential state transitions should pass through this layer.

That includes:
- accepting a control mapping
- marking primary evidence attached
- approving an exception
- closing a remediation item
- approving a readiness snapshot

---

## 3. System of record

### Recommended data stores
- relational database for authoritative state
- object storage for raw artifacts and immutable packet outputs
- optional graph projection or search index for relationship-heavy traversal

### Store authoritatively in the relational core
- systems and boundaries
- assets and owners
- findings
- evidence metadata
- mappings
- remediation items
- exceptions
- notifications
- reviews and approvals
- snapshots and packet manifests

### Store in object storage
- raw JSON payloads
- exports and reports
- screenshots/PDFs/docs where retained
- immutable packet bundles
- derivative drafts and narratives with provenance

---

## 4. Connector and normalization layer

### AWS-native connectors
- Security Hub / ASFF ingestor
- Config recorder/import processor
- CloudTrail / CloudTrail Lake reference collector
- Inspector ingestor
- GuardDuty ingestor
- Access Analyzer ingestor
- Systems Manager compliance/inventory ingestor

### On-prem connectors
- scanner result importer
- baseline/config exporter
- identity review importer
- document metadata harvester
- ticketing/remediation mirror

### Responsibilities
- fetch or receive source data
- preserve raw payload references
- normalize into canonical schema
- resolve asset/system identities
- assign scope hints
- trigger workflows

---

## 5. Evidence and packet services

### Components
- evidence catalog service
- evidence freshness engine
- packet generation service
- integrity/hash service

### Responsibilities
- register evidence objects
- enforce freshness policies
- link evidence to controls/findings/remediation
- generate immutable packet manifests and exports
- track derivative summaries separately from primary evidence

---

## 6. Search and retrieval layer

### Components
- keyword search index
- vector index over approved corpora
- citation builder / context packager

### Approved corpora
- accepted control text / internal mappings
- approved policy and procedure docs
- accepted evidence metadata and selected evidence text extractions
- historical accepted narratives and packet summaries

### Exclusions by default
- raw inbound chat/email content unless intentionally approved
- unsupported or unaudited external content
- evidence objects lacking provenance or scope classification

---

## 7. Policy and guardrail engine

### Responsibilities
- decide which actions are advisory vs approval-gated
- enforce recipient/channel policy for notifications
- control which evidence can be shown in which workflow contexts
- gate AI prompts and outputs to approved source sets
- apply classification and scope policy

### Examples
- draft control summary → allowed
- mark control as reviewable → human approval required
- include exception in packet → approval required
- send detailed evidence excerpt to broad channel → blocked by policy

---

## 8. Notification service

### Responsibilities
- send Teams and email notifications
- use policy-aware message templates
- bundle reminders into digests where possible
- record delivery attempts and message provenance

### Design rule
Notifications must never become the primary evidence repository.

---

## 9. Bedrock reasoning/drafting layer

## Role in the architecture
Bedrock should be used as a **bounded reasoning and drafting subsystem**, not as an authoritative control-state engine.

### Recommended services
- control-mapping draft service
- evidence-gap analysis service
- assessor-question prep service
- packet narrative draft service
- contradiction summarization service
- daily/monthly digest drafting service

### Input constraints
Only provide:
- approved retrieval context
- bounded system prompts describing output schema and prohibitions
- explicit citations payloads
- minimized, policy-approved evidence excerpts

### Output constraints
Outputs must be structured as one of:
- draft summary
- proposed mapping
- proposed follow-up questions
- proposed packet narrative
- contradiction explanation draft

Each output should carry:
- source citations
- confidence statement
- explicit unresolved questions
- model/version and generation metadata

---

## Where Claude Sonnet 4.5 fits

### Working assumption
If **Claude Sonnet 4.5 is available through Bedrock in the target GovCloud deployment path or in an approved adjacent pattern**, it is a good fit for:
- synthesizing cross-source evidence
- drafting assessor-friendly narratives
- identifying contradictions across findings, policies, and prior summaries
- turning structured state into clear operator-facing explanations

### Important caveat
I did not verify a source here that specifically proves **Claude Sonnet 4.5 availability in AWS GovCloud**. Bedrock model availability varies by region/service. Therefore:
- **verified fact**: Bedrock provides access to foundation models and model availability is cataloged by AWS. [4]
- **working assumption**: the target deployment can use Claude Sonnet 4.5 through an approved Bedrock path
- **required implementation check**: verify Bedrock regional/model availability and applicable boundary/policy constraints before finalizing architecture

### Safe task profile for Sonnet-class model usage
- synthesize evidence across many linked records
- draft narratives with citations
- generate review questions and issue summaries
- compare current state with prior snapshot text

### Unsafe task profile
- final control adjudication
- automatic exception approval
- direct infrastructure mutation
- evidence deletion or suppression
- silent ticket closure

---

## Concrete namespace / service model on EKS

Suggested Kubernetes namespaces:
- `vigilant-edge` — ingress, API gateway, auth helpers
- `vigilant-core` — workflow, case management, mappings, packet services
- `vigilant-connectors` — AWS and on-prem ingestion workers
- `vigilant-search` — retrieval and indexing services
- `vigilant-notify` — notification workers
- `vigilant-ai` — Bedrock orchestration, prompt templates, citation packaging
- `vigilant-ops` — observability, audit, admin jobs

Suggested service breakdown:
- `api-service`
- `workflow-service`
- `finding-service`
- `evidence-service`
- `mapping-service`
- `remediation-service`
- `snapshot-service`
- `packet-service`
- `policy-service`
- `notify-service`
- `retrieval-service`
- `bedrock-orchestrator`
- `connector-*` workers

---

## Data flow

### Flow 1: New finding
1. Connector ingests source payload.
2. Raw payload reference stored.
3. Finding normalized and scoped.
4. Mapping service proposes controls.
5. Workflow service opens triage/review task.
6. Notification service alerts owner/ISSO queue.
7. If requested, AI layer drafts triage summary with citations.

### Flow 2: Evidence refresh
1. Freshness engine flags stale primary evidence.
2. Workflow service creates evidence-refresh task.
3. Notification service routes to control owner.
4. New evidence submitted and cataloged.
5. AI layer may draft updated control narrative.
6. Human reviewer approves updated packet state.

### Flow 3: Monthly readiness snapshot
1. Snapshot workflow freezes a point-in-time state.
2. Packet service generates manifests and issue lists.
3. Retrieval service assembles supporting context.
4. Bedrock layer drafts snapshot narrative and top-risk summary.
5. ISSO/compliance lead reviews and approves.
6. Immutable packet bundle stored.

---

## Security and trust guardrails

## Evidence-grounding guardrails
- no model call without an explicit citation set
- no narrative persisted as primary evidence
- every generated statement links to source objects
- stale or weak evidence must remain labeled as such

## Human-in-the-loop guardrails
- approval required for material control-state changes
- approval required for exceptions and compensating controls
- approval required for packet publication
- approval required for closure of significant remediation items

## Data handling guardrails
- keep raw source payloads separate from derivative summaries
- enforce scope/classification policy before retrieval or notification
- use least-privilege IAM/service accounts for connectors and workers
- record every model invocation with prompt context references, not just output text

## Reliability guardrails
- durable queues for ingestion and workflows
- idempotent connector processing
- immutable snapshot and packet outputs
- retry with dead-letter handling for connector failures

---

## GovCloud-specific implications

Because GovCloud EKS lacks some commercial-region capabilities, the architecture should avoid depending on:
- EKS on Fargate
- Amazon Managed Service for Prometheus as a required managed dependency
- EKS Anywhere / Hybrid Nodes for the core design

Instead, prefer:
- EC2-backed worker/node groups
- self-managed observability stack if needed
- explicit validation of each dependency’s GovCloud support status

Also account for:
- `aws-us-gov` ARNs and partition-aware code paths
- export-controlled metadata caveats called out in GovCloud documentation
- separate availability validation for each AWS service in the final deployment pattern [2][3]

---

## Recommended first release architecture

### Phase 1
Build these first:
- EKS-hosted API/workflow services
- relational system of record
- object-backed evidence store
- AWS-native connectors
- basic on-prem import adapters
- notification service
- retrieval service over approved corpora
- Bedrock-assisted draft summaries and packet narratives

### Phase 2
Add:
- graph projection for relationship traversal
- contradiction detection improvements
- recurrence/root-cause analytics
- richer interactive playbooks

### Phase 3
Add:
- OSCAL interchange helpers
- broader source integrations
- stronger policy automation around packet assembly and review routing

---

## Verified facts vs assumptions

### Verified facts
- EKS is managed by AWS and available in GovCloud, with notable feature differences. [1][2]
- GovCloud requires partition-aware architecture. [3]
- Bedrock is a managed foundation-model service. [4]
- AWS security services listed here provide core telemetry for the concept. [5][6][7][8][9][10][11]

### Working assumptions
- EKS is the right application platform for a first serious Vigilant ISSO implementation because it balances portability, control, and integration flexibility.
- A dedicated Bedrock orchestration service is preferable to embedding model calls throughout the application.
- Claude Sonnet 4.5 is the right reasoning/drafting profile if available in the approved target environment.

---

## Sources

1. AWS, “What is Amazon EKS?” https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html
2. AWS, “Amazon Elastic Kubernetes Service in AWS GovCloud (US).” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-eks.html
3. AWS, “AWS GovCloud (US).” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/
4. AWS, “Overview - Amazon Bedrock.” https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html
5. AWS, “Introduction to AWS Security Hub CSPM.” https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
6. AWS, “What Is AWS Config?” https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
7. AWS, “What Is AWS CloudTrail?” https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
8. AWS, “What is Amazon Inspector?” https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html
9. AWS, “What is Amazon GuardDuty?” https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html
10. AWS, “Using IAM Access Analyzer.” https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
11. AWS, “AWS Systems Manager Compliance.” https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-compliance.html
