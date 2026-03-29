# Sprint 1 Backlog for the MVP Prototype

## Purpose

Define a **practical Sprint 1 backlog** for the first bounded Vigilant ISSO prototype.

This sprint is not trying to finish the platform. It is trying to create the smallest coherent slice that proves the prototype is real and pointed in the right direction.

---

## Sprint 1 goal

Stand up the **first end-to-end skeleton** of the prototype for the bounded environment so the team can demonstrate:
- the environment and service topology are nailed down
- infrastructure/release scaffolding exists
- a minimal core service path exists
- one or two real source signals can enter the system shape cleanly
- the backlog for Sprint 2 is driven by evidence instead of speculation

---

## Verified facts from this repo
- The broader architecture already favors an **EKS-hosted, evidence-grounded control plane** with workflow, record, evidence, packet, retrieval, and notification layers.
- Existing docs recommend building the platform in sequence: core platform first, then connectors/normalization, then evidence/control workflows, then notifications/snapshots, then bounded AI.
- The first prototype boundary is explicitly constrained to **1 AWS account, 2 VPCs, 1 data lake, and 3 small EKS clusters**.

## Working assumptions
- Sprint 1 should establish the **prototype skeleton**, not production-grade completeness.
- The team has enough access to at least inspect or model the bounded environment even if not every source integration is fully wired yet.
- The fastest useful Sprint 1 outcome is one that creates a stable platform shape and one narrow ingestion-through-record flow.

---

## Sprint 1 success statement

By the end of Sprint 1, the team should be able to say:

**“We have a defined prototype boundary, Terraform and Helm scaffolding for the three-cluster topology, a running core service skeleton, authoritative record schemas for the first objects, and at least one narrow finding ingestion path landing in the bounded environment model.”**

---

## Sprint 1 backlog

## Epic A — Prototype boundary and architecture lock

### A1. Confirm first-environment inventory
**Priority:** P0  
**Outcome:** named inventory of the single AWS account, 2 VPCs, 1 data lake, and 3 EKS clusters used by the prototype.

**Tasks**
- document prototype environment identifiers and intended ownership model
- confirm which cluster plays which prototype role
- document whether VPCs/data lake are Terraform-managed or referenced inputs
- capture unknowns and external dependencies explicitly

**Deliverable**
- environment inventory note or appendix used by infra/app work

---

### A2. Freeze MVP in-scope vs out-of-scope list
**Priority:** P0

**Tasks**
- approve first source set for sprint-1/sprint-2 work
- approve which features are explicitly deferred
- confirm AI is optional/feature-flagged

**Deliverable**
- accepted prototype scope baseline

---

## Epic B — Infrastructure and release scaffolding

### B1. Create Terraform repository skeleton
**Priority:** P0  
**Outcome:** working repo/tree for the module structure proposed in `terraform-module-map.md`.

**Tasks**
- create `infra/modules/` structure
- create `infra/environments/prototype/` root stack
- add provider/version/backend placeholders
- add README with prototype infra usage notes

**Acceptance criteria**
- repo/tree exists and is reviewable
- root stack composes placeholder modules
- no hand-wavy “we’ll figure it out later” gaps in the structure

---

### B2. Stub coarse Terraform modules
**Priority:** P0

**Tasks**
- add module skeletons for:
  - account baseline
  - network scope
  - EKS cluster
  - data services
  - object evidence store
  - queue backbone
  - IAM platform access
  - observability foundation
  - app secrets/config
- define inputs/outputs contracts even if implementation is partial

**Acceptance criteria**
- every planned module has a clear interface
- cluster module can be instantiated three times in the prototype root stack

---

### B3. Create Helm chart family skeleton
**Priority:** P0  
**Outcome:** chart tree exists for `vigilant-base`, `vigilant-core`, `vigilant-workers`, and `vigilant-support`.

**Tasks**
- create chart.yaml/templates/values skeletons
- define environment values structure for prototype
- wire namespaces, service accounts, and placeholder deployments

**Acceptance criteria**
- helm template works for the skeleton charts
- cluster-role deployment intent is visible in the chart layout

---

## Epic C — Core application skeleton

### C1. Define canonical schema v0 for prototype objects
**Priority:** P0

**Tasks**
- define initial objects for:
  - environment/system
  - VPC
  - EKS cluster
  - finding
  - evidence metadata
  - remediation item
  - snapshot
- define required provenance fields
- define object IDs and minimal relationships

**Acceptance criteria**
- schema v0 is documented and implementable
- provenance fields are mandatory where appropriate
- derivative narrative content is explicitly distinct from evidence

---

### C2. Stand up minimal `record-service`
**Priority:** P0

**Tasks**
- create service skeleton with health endpoint
- define minimal persistence layer
- support create/read for first object set
- support recording source references on findings

**Acceptance criteria**
- service runs in dev/prototype environment
- at least one finding record can be written and read back

---

### C3. Stand up minimal `workflow-service`
**Priority:** P0

**Tasks**
- define initial workflow states for finding triage
- create endpoint or command path for opening a triage item
- persist actor/time/status metadata

**Acceptance criteria**
- a finding can trigger creation of a triage workflow item
- state transitions are explicit and auditable

---

### C4. Stand up minimal `api-service`
**Priority:** P0

**Tasks**
- expose health and version endpoints
- expose minimal routes for finding ingestion and record lookup
- add request ID / trace propagation scaffolding

**Acceptance criteria**
- API routes are sufficient to exercise the first vertical slice
- request identity/tracing scaffolding is visible

---

## Epic D — First ingestion slice

### D1. Define connector-runtime contract
**Priority:** P0

**Tasks**
- define input/output contract for source connector jobs
- define raw payload reference pattern
- define retry and dead-letter placeholders

**Acceptance criteria**
- connector implementation pattern is documented and reusable
- normalization handoff format is agreed

---

### D2. Build one narrow AWS connector path
**Priority:** P0  
**Recommended first choice:** Security Hub or Config.

**Tasks**
- implement one connector skeleton
- ingest sample/real payloads for the bounded environment
- persist raw reference metadata
- hand off to normalization path

**Acceptance criteria**
- at least one real source type lands in canonical finding shape
- provenance back to source payload/reference is preserved

---

### D3. Build normalization worker v0
**Priority:** P0

**Tasks**
- transform first connector payload into canonical finding object
- attach account/VPC/cluster scope hints where possible
- flag unknown mappings rather than inventing certainty

**Acceptance criteria**
- normalized output is stable and reviewable
- unresolved scope/mapping ambiguity is explicit, not hidden

---

### D4. Build asset/environment resolution v0
**Priority:** P1

**Tasks**
- map first source records to prototype environment entities
- support association to account, VPC, and cluster where feasible
- return confidence or unresolved state

**Acceptance criteria**
- first findings can be attached to the correct prototype boundary objects often enough to prove the model

---

## Epic E — Evidence and remediation skeleton

### E1. Define evidence metadata model v0
**Priority:** P1

**Tasks**
- define evidence object shape
- include source, observed_at, collected_at, integrity/reference fields, classification, and freshness metadata placeholders
- define raw vs derivative distinction

**Acceptance criteria**
- evidence model can support both source artifacts and future packet references

---

### E2. Define remediation item model v0
**Priority:** P1

**Tasks**
- define owner, due date, status, validation-evidence reference, and linked finding fields
- define closure states requiring validation

**Acceptance criteria**
- remediation shape supports the intended lifecycle without overbuilding ticket sync

---

## Epic F — DevEx and observability basics

### F1. Add local/dev deployment path
**Priority:** P1

**Tasks**
- define how developers run or template the prototype services locally
- add sample env files / values examples
- add makefile/scripts if useful

**Acceptance criteria**
- another engineer can stand up the skeleton without tribal knowledge

---

### F2. Add baseline observability hooks
**Priority:** P1

**Tasks**
- health endpoints
- structured logging
- request IDs / correlation IDs
- minimal connector/job status logging

**Acceptance criteria**
- first vertical slice is debuggable

---

## Sprint 1 stretch items

Only if the P0 backbone lands cleanly:

### S1. Add second AWS source path
- if Security Hub is first, add Config
- if Config is first, add Inspector or GuardDuty

### S2. Add notification-service skeleton
- create placeholder event-to-notification path
- no need for rich delivery logic yet

### S3. Add snapshot object/model skeleton
- define snapshot record even if packet generation waits for Sprint 2

---

## Explicitly not Sprint 1

Do **not** let Sprint 1 expand into:
- polished front-end experience
- full control mapping engine
- mature packet generation
- Bedrock integration unless the rest is already stable
- deep on-prem importer work
- broad search/indexing
- enterprise IAM perfectionism beyond what is needed to prove the skeleton

Sprint 1 should end with a coherent spine, not a pile of half-built “advanced” features.

---

## Suggested sprint board view

### P0 — must land
- A1 Confirm first-environment inventory
- A2 Freeze MVP scope
- B1 Create Terraform repo skeleton
- B2 Stub Terraform modules
- B3 Create Helm chart family skeleton
- C1 Define canonical schema v0
- C2 Stand up minimal record-service
- C3 Stand up minimal workflow-service
- C4 Stand up minimal api-service
- D1 Define connector-runtime contract
- D2 Build one narrow AWS connector path
- D3 Build normalization worker v0

### P1 — should land if P0 is on track
- D4 Build asset/environment resolution v0
- E1 Define evidence metadata model v0
- E2 Define remediation item model v0
- F1 Add local/dev deployment path
- F2 Add baseline observability hooks

### Stretch
- S1 second AWS source path
- S2 notification skeleton
- S3 snapshot skeleton

---

## Sprint 1 exit criteria

Sprint 1 is successful if the team has:
- a frozen prototype boundary
- Terraform/Helm scaffolding for the three-cluster model
- minimal running core services
- a canonical finding shape with provenance
- one real ingestion-to-record-to-workflow path
- enough observability to debug that path

If those are not true, Sprint 1 is not done, even if lots of code exists.

---

## Verified facts vs working assumptions summary

### Verified facts
- Existing repo docs already sequence the platform toward core infrastructure and workflows first.
- The prototype boundary is explicitly limited to one bounded AWS environment with 2 VPCs, 1 data lake, and 3 small EKS clusters.
- The architecture already emphasizes provenance, evidence discipline, and human-reviewed workflows.

### Working assumptions
- Sprint 1 should focus on the end-to-end spine, not breadth.
- Security Hub or Config is the best first AWS connector slice.
- Helm and Terraform scaffolding are worth doing in Sprint 1 because the three-cluster shape is part of the prototype definition, not an afterthought.

---

## Bottom line

Sprint 1 should produce:

**a real prototype skeleton with one narrow working ingestion path, not a slide deck disguised as engineering progress.**
