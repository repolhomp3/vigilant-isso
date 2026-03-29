# Vigilant ISSO Implementation Roadmap

## Goal

Define a practical build path for turning Vigilant ISSO from a concept into a usable, evidence-grounded platform for **AWS GovCloud + on-prem CMMC Level 2 / NIST-oriented security operations**.

This roadmap is intentionally conservative:
- prove data fidelity before claiming automation value
- prove evidence discipline before scaling AI drafting
- prove operator usefulness before broad rollout
- keep consequential control-state decisions human-approved

---

## Verified facts

1. **Amazon EKS is a managed Kubernetes service, and EKS is available in AWS GovCloud (US).** AWS also documents notable GovCloud differences, including lack of EKS on Fargate, Amazon Managed Service for Prometheus, EKS Anywhere, and EKS Hybrid Nodes. [1][2]
2. **AWS GovCloud uses a distinct partition and service realities** that affect ARNs, integrations, and deployment assumptions. [3]
3. **NIST SP 800-171 Rev. 2 remains the authoritative source for the CUI security requirements used by many CMMC Level 2 programs today**, and NIST explicitly states the PDF is the authoritative source. [4]
4. **NIST SP 800-171A Rev. 3 provides assessment procedures and methodology** for assessing the 800-171 security requirements, including use in independent third-party and government-sponsored assessments. [5]
5. **The DoD CIO CMMC resources page publishes current Level 2 scoping and assessment guidance** and notes phased implementation began in November 2025, with early focus on Level 1 and Level 2 self-assessments. [6]
6. **NIST OSCAL provides machine-readable formats for catalogs, profiles, system security plans, assessment plans, assessment results, and plans of action and milestones.** That makes it relevant as an interchange target for control/evidence exports. [7]
7. **Amazon Bedrock is a managed service for foundation models, but model availability is region-specific.** Final architecture choices must therefore validate target-region availability rather than assume parity. [8]

---

## Working assumptions

These points are design assumptions, not verified facts:

1. The strongest first deployment is an **internal operator platform** for ISSOs and security/compliance teams, not a broad end-user SaaS product.
2. The first release should optimize for **trust, provenance, and workflow discipline**, not flashy autonomous remediation.
3. The most credible MVP will start with a **small connector set and a narrow evidence model**, then expand after proving reliability.
4. A successful rollout depends more on **clear ownership, review queues, and good packet outputs** than on advanced analytics.
5. Bedrock-backed summarization and drafting are useful only after retrieval, citation, and policy gates are already working.
6. Many early customer environments will need **semi-structured on-prem imports** before they can support direct connectors.

---

## What must be proven before wider rollout

Before Vigilant ISSO should be used broadly across enclaves or organizations, the team should be able to demonstrate:

### 1. Source fidelity
- connectors preserve raw payload provenance
- normalization does not destroy source meaning
- asset/system identity resolution is accurate enough for triage
- duplicate handling suppresses noise without hiding evidence

### 2. Evidence discipline
- primary evidence and derivative narratives remain distinct
- freshness rules work predictably
- packet generation is reproducible
- reviewers can trace every summary claim to source artifacts

### 3. Workflow credibility
- finding triage and POA&M workflows are durable and auditable
- approvals are recorded with actor/time/context
- exceptions and compensating controls are visible, bounded, and expiring
- recurring review cadences produce smaller queues, not bigger piles

### 4. Human usefulness
- ISSOs can answer real readiness questions faster than before
- control owners can understand requests without translation overhead
- operators trust the summaries because the citations hold up
- the platform reduces scramble work ahead of internal/external assessments

### 5. Platform reliability and security
- ingestion is idempotent and resilient to partial failure
- queues, retries, and dead-letter paths are in place
- least-privilege access is enforced for connectors and services
- multi-tenant or multi-enclave boundaries are enforced before scaling outward

---

## Delivery strategy

### MVP boundary
The MVP should prove four things only:
1. telemetry can be ingested and normalized reliably
2. evidence can be cataloged and freshness-tracked
3. remediation work can be managed with durable workflow state
4. the system can generate reviewable, source-backed readiness outputs

If the MVP cannot do those four things well, later AI and analytics layers will just decorate a weak core.

### Later-phase boundary
Only after the MVP is trustworthy should the platform expand into:
- contradiction detection across policy, telemetry, and prior narratives
- stronger relationship analytics and recurrence modeling
- broader on-prem connector coverage
- OSCAL import/export helpers
- more advanced assistant behaviors around packet drafting and assessor-question prep

---

## Implementation phases

## Phase 0 — Foundation, scope, and truth model

### Objective
Lock the operating assumptions, trust boundaries, and internal data model before building the main platform.

### Build
- define canonical entities: systems, enclaves, assets, findings, evidence, controls, mappings, remediation items, exceptions, snapshots, packets
- define source provenance and chain-of-custody rules
- define control-state vocabulary and review-state model
- define advisory-only vs approval-gated actions
- define initial success metrics and rollout gates

### Deliverables
- canonical schema v0
- evidence provenance rules
- workflow state model
- access model for operators, ISSOs, reviewers, and control owners
- approved source list for MVP

### Dependencies
- existing concept docs in this repo
- decision on GovCloud-hosted control plane vs alternate hosting boundary
- agreement on first assessment boundary / pilot environment

### Exit criteria
- the team can explain what the system will and will not claim
- approved MVP source list exists
- reviewers agree on the distinction between fact, inference, and recommendation

### Key risks
- trying to model every edge case up front
- blurring evidence objects with AI-authored summaries
- leaving scoping and inheritance ambiguous

---

## Phase 1 — Core platform skeleton on EKS

### Objective
Stand up the minimal control plane and system-of-record needed to support real workflows.

### Build
- EKS cluster baseline in GovCloud
- ingress/API layer
- relational system of record
- object-backed evidence store
- queue/event backbone
- workflow engine and review-state service
- authentication/authorization foundation
- baseline observability and audit logging

### Deliverables
- deployable platform skeleton
- namespace and service boundaries
- secure secrets/config handling
- backup/restore and disaster-recovery runbooks for core stores

### Dependencies
- Phase 0 data/workflow model
- GovCloud deployment prerequisites
- cluster security baseline and IAM design

### Exit criteria
- operators can authenticate
- services can persist findings/evidence/remediation state
- every API mutation is attributable and auditable
- a basic review workflow can run end to end

### Key risks
- overcomplicating cluster topology too early
- choosing managed dependencies not available in GovCloud
- weak audit logging for approval actions

---

## Phase 2 — MVP connectors and normalization

### Objective
Ingest the first real signals and prove that normalization preserves meaning.

### MVP source set
#### AWS-native
- Security Hub CSPM
- AWS Config
- Amazon Inspector
- GuardDuty
- IAM Access Analyzer
- CloudTrail references
- Systems Manager Compliance where available and relevant

#### On-prem / enterprise imports
- vulnerability scanner export
- asset inventory / CMDB export
- ticketing system import or mirror
- document/evidence metadata import for SSPs, procedures, diagrams, access reviews

### Build
- connector runtime pattern
- raw payload archival references
- canonical finding/evidence normalization
- identity/asset resolution service
- deduplication and recurrence rules
- ingest quality and drift monitoring

### Deliverables
- first wave of production-like connectors/importers
- normalized finding pipeline
- connector health dashboard
- ingest error and dead-letter handling

### Dependencies
- Phase 1 storage and queues
- source access and service account approvals
- canonical schema for findings/assets/evidence

### Exit criteria
- at least 80–90% of pilot-source records land cleanly without manual repair
- duplicate findings are grouped without losing provenance
- operators can trace normalized objects back to raw source records

### Key risks
- inconsistent asset identifiers across cloud and on-prem data
- over-normalization that erases source nuance
- underestimating manual prep required for on-prem data feeds

---

## Phase 3 — Evidence catalog, freshness, and control mapping MVP

### Objective
Turn ingested data into control-aware evidence and reviewable posture state.

### Build
- evidence catalog service
- freshness policy engine
- control and practice library loader
- mapping service for findings-to-controls and evidence-to-controls
- human review UX/API for approving mappings
- contradiction flags for obvious evidence gaps and stale artifacts

### Deliverables
- control view by practice/domain/family
- evidence completeness and freshness indicators
- approved mapping workflow
- initial control health profile

### Dependencies
- Phase 2 ingestion and identity resolution
- approved control crosswalk approach
- evidence type taxonomy

### Exit criteria
- reviewers can open a control and see linked findings, evidence, open remediation, and freshness state
- stale/missing evidence is surfaced automatically
- proposed mappings are reviewable with provenance

### Key risks
- pretending control mapping is deterministic when it is not
- letting weak evidence look equal to strong evidence
- producing fake precision in control status

---

## Phase 4 — Remediation, POA&M, and notification workflows

### Objective
Make the platform operationally useful by managing actual follow-up work.

### Build
- remediation item service
- POA&M lifecycle support
- SLA and aging logic
- exception/compensating-control workflow
- owner assignment and routing
- policy-aware Teams/email notifications and digests

### Deliverables
- triage queue
- owner dashboards
- overdue/aging reports
- closure validation workflow
- exception expiration and renewal handling

### Dependencies
- Phase 3 control and evidence model
- integration decision for ticketing system of record vs mirrored state
- notification policy and recipient rules

### Exit criteria
- accepted findings become trackable remediation items
- owners receive actionable requests with enough context to act
- closure requires verification evidence, not just status changes

### Key risks
- duplicating instead of integrating with existing ticketing workflows
- notification spam reducing trust and response rates
- weak closure controls turning POA&Ms into theater

---

## Phase 5 — Readiness snapshots, packets, and bounded AI drafting

### Objective
Produce the outputs that make the platform genuinely valuable to ISSOs and assessment prep teams.

### Build
- snapshot service for point-in-time readiness freezes
- packet manifest and export service
- retrieval/citation service over approved corpora
- Bedrock orchestration layer for draft summaries, evidence-gap analysis, and packet narratives
- generation logging with prompt/context metadata
- human approval workflow for published narratives and packets

### Deliverables
- monthly readiness snapshots
- pre-assessment packet assembly
- draft control summaries with citations
- contradiction summary drafts
- assessor-question prep drafts

### Dependencies
- trustworthy evidence catalog and workflow state
- approved retrieval corpora and classification rules
- validated Bedrock region/model path in target environment

### Exit criteria
- every generated narrative is citation-backed and reviewable
- published packets are immutable and reproducible
- the team can defend how a narrative was generated and approved

### Key risks
- using AI before retrieval discipline is mature
- overexposing sensitive evidence in prompts or notifications
- confusing drafted narratives with authoritative evidence

---

## Phase 6 — Pilot operation in a real boundary

### Objective
Run Vigilant ISSO against a limited but real assessment boundary long enough to discover operational truth.

### Recommended pilot shape
- one enclave or one well-bounded program
- limited AWS accounts and a constrained set of on-prem feeds
- named ISSO/operator/control-owner cohort
- monthly snapshots for at least 60–90 days

### Activities
- run daily hygiene loop
- run monthly control review loop
- measure ingestion reliability, queue volume, false-positive rate, mapping approval rate, and packet usefulness
- compare platform outputs with current manual process

### Success criteria
- measurable reduction in readiness scramble work
- measurable reduction in stale evidence and orphan findings
- reviewers trust the outputs enough to rely on them for internal reviews
- open issues cluster around source coverage gaps, not basic platform instability

### Risks to monitor
- review bottlenecks caused by poor workflow design
- ownership ambiguity for shared/inherited controls
- inability to reconcile external ticket state with platform state

---

## Phase 7 — Hardening and controlled expansion

### Objective
Expand only after the pilot proves trust, usefulness, and operational safety.

### Build next
- more connectors and import adapters
- richer relationship analytics / graph projection
- recurrence and root-cause reporting
- stronger contradiction analysis
- OSCAL import/export helpers
- cross-enclave policy templates and deployment automation

### Rollout gates
Do not widen rollout until the pilot demonstrates:
- sustained ingest reliability
- defensible evidence provenance
- review SLAs that actually hold
- bounded AI usage with auditable outputs
- acceptable operator adoption and low rework rates

---

## Milestone view

| Milestone | Outcome | Proof required |
| --- | --- | --- |
| M0 Concept frozen | Canonical model and trust rules approved | schema, workflow states, action policy approved |
| M1 Core alive | EKS-hosted control plane and system of record running | auth, auditability, API mutations, backups validated |
| M2 Data landing | MVP connectors ingest real pilot data | provenance retained, dedupe/identity resolution acceptable |
| M3 Control-aware | Evidence freshness and mappings visible | reviewers can navigate from control to evidence/findings |
| M4 Operational | Remediation and notification loops functioning | accepted findings become actionable work with closure validation |
| M5 Assessment-useful | Snapshots, packets, and draft narratives available | packet claims trace back to evidence and approvals |
| M6 Pilot-proven | Real boundary operated for 60–90 days | reduced scramble, trusted outputs, manageable queues |
| M7 Expandable | Platform ready for wider rollout | security, reliability, adoption, and governance gates met |

---

## MVP vs later phases

## MVP
Must include:
- EKS-hosted core services
- relational record of findings/evidence/remediation/snapshots
- object-backed evidence manifests
- AWS-native connectors plus limited on-prem imports
- evidence freshness tracking
- control mapping review workflow
- remediation and notification workflow
- monthly readiness snapshots and packet manifests
- citation-backed drafting for narrow summary types only

Must not include:
- autonomous infrastructure mutation
- automatic control closure
- automatic exception approval
- broad “compliance score” claims without qualification
- wide connector sprawl before core reliability is proven

## Later phases
Add only after MVP proof:
- graph/relationship analytics
- recurrence clustering and root-cause insights
- assessor-question prediction
- richer on-prem live connectors
- OSCAL interchange helpers
- broader multi-tenant or multi-program deployment patterns

---

## Suggested sequence of workstreams

These can overlap, but should not be started all at once:

1. **Data and workflow model**
2. **EKS/core platform skeleton**
3. **Connectors and normalization**
4. **Evidence/freshness/control views**
5. **Remediation/notification workflows**
6. **Snapshots/packets/retrieval**
7. **Bounded AI drafting**
8. **Pilot hardening and expansion**

That sequencing matters because later layers depend on trustworthy lower layers.

---

## Recommended team shape for an initial build

### Minimum serious team
- platform lead / architect
- backend engineer for core services and data model
- integration engineer for connectors/imports
- security/compliance product owner or ISSO lead
- UX/operator workflow designer or strong product-minded engineer
- part-time cloud/platform engineer for EKS/IAM/observability hardening

### Critical non-code involvement
- actual ISSO/operator reviewers
- one assessment-savvy compliance lead
- source-system owners for AWS and on-prem telemetry

Without those people, the platform may be technically correct but operationally useless.

---

## Sources

1. AWS, “What is Amazon EKS?” https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html
2. AWS, “Amazon Elastic Kubernetes Service in AWS GovCloud (US).” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-eks.html
3. AWS, “AWS GovCloud (US).” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/
4. NIST, “SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations.” https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
5. NIST, “SP 800-171A Rev. 3, Assessing Security Requirements for Controlled Unclassified Information.” https://csrc.nist.gov/pubs/sp/800/171/a/r3/final
6. DoD CIO, “CMMC Resources & Documentation.” https://dodcio.defense.gov/CMMC/Resources-Documentation/
7. NIST, “Open Security Controls Assessment Language (OSCAL).” https://pages.nist.gov/OSCAL/
8. AWS, “Overview - Amazon Bedrock.” https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html
9. AWS, “Amazon EKS Best Practices Guide.” https://docs.aws.amazon.com/eks/latest/best-practices/introduction.html
