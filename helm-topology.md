# Helm and Service Topology for the MVP Prototype

## Purpose

Define the **Helm release model** and **small EKS-hosted service topology** for the first bounded Vigilant ISSO prototype.

This document answers a practical question:

**If the prototype uses 3 small EKS clusters, what should actually run where, and how should Helm package it?**

---

## Design constraints

### Verified facts from this repo
- Existing docs recommend an **EKS-hosted application control plane** with clear separation between core state, connectors, search/retrieval, notifications, AI, and operations.
- Existing docs recommend the following namespace model as the target direction: `vigilant-edge`, `vigilant-core`, `vigilant-connectors`, `vigilant-search`, `vigilant-notify`, `vigilant-ai`, and `vigilant-ops`.
- Existing docs also recommend a **coarse-grained initial posture** rather than a giant microservice farm.

### Working assumptions
- For the prototype, Helm should optimize for **operability and clarity**, not maximum chart cleverness.
- The three clusters should have distinct operational roles even if some services could technically be co-located.
- The prototype can start with a combination of:
  - a few shared base charts
  - a small number of workload charts
  - one environment values tree for the first bounded deployment

---

## Recommended cluster roles

The prototype has **3 small EKS clusters**. A practical topology is:

### Cluster 1 — `vigilant-core`
Primary authoritative control-plane cluster.

Runs:
- API edge
- workflow/state services
- policy service
- packet/snapshot services
- relational DB clients
- evidence metadata services

### Cluster 2 — `vigilant-workers`
Connector, normalization, and async workload cluster.

Runs:
- AWS-native connectors
- import workers
- normalization workers
- dedupe/retry/background jobs
- notification workers
- scheduled jobs that do not need to live on the authoritative core cluster

### Cluster 3 — `vigilant-support`
Search, optional AI, and operations/support cluster.

Runs:
- search indexer
- retrieval service
- optional Bedrock orchestration and narrative drafting
- observability stack components
- admin/support tooling that should not burden the core cluster

### Why this split works for the prototype
- keeps **authoritative state** concentrated
- isolates **failure-prone workers** from the main control plane
- gives search/AI/ops a separate blast radius
- still stays small enough to understand and operate

---

## Namespace model by cluster

Not every namespace must exist on every cluster.

### Cluster 1 — `vigilant-core`
Namespaces:
- `vigilant-edge`
- `vigilant-core`

Primary workloads:
- ingress controller
- `api-service`
- auth helpers / middleware sidecars if needed
- `workflow-service`
- `record-service`
- `policy-service`
- `snapshot-service`
- `packet-service`
- `evidence-catalog-service`
- `mapping-service`
- `remediation-service`

### Cluster 2 — `vigilant-workers`
Namespaces:
- `vigilant-connectors`
- `vigilant-notify`

Primary workloads:
- `connector-runtime`
- `connector-securityhub`
- `connector-config`
- `connector-inspector`
- `connector-guardduty`
- `connector-accessanalyzer` if included
- `connector-cloudtrail-ref` if included
- `connector-ssm-compliance` if included
- `importer-vulnscan` placeholder or adapter
- `importer-asset-inventory` placeholder or adapter
- `normalization-worker`
- `asset-resolution-service` or worker variant
- `dedupe-recurrence-worker`
- `notification-service`
- `digest-worker`
- recurring workflow trigger jobs

### Cluster 3 — `vigilant-support`
Namespaces:
- `vigilant-search`
- `vigilant-ai`
- `vigilant-ops`

Primary workloads:
- `search-indexer`
- `retrieval-service`
- optional vector/index support if used later
- `bedrock-orchestrator` if enabled
- `narrative-draft-service` if enabled
- observability stack
- admin maintenance jobs
- backup/restore runners where Kubernetes-hosted

---

## MVP service cut

To keep the prototype tight, the actual first release should use this smaller service set.

### Must run in the prototype
- `api-service`
- `workflow-service`
- `record-service`
- `policy-service`
- `snapshot-service`
- `packet-service`
- `evidence-catalog-service`
- `mapping-service`
- `remediation-service`
- `connector-runtime`
- selected AWS connectors
- `normalization-worker`
- `asset-resolution-service`
- `notification-service`
- `search-indexer`
- `retrieval-service`
- observability components

### May run if already justified
- `digest-worker`
- `dedupe-recurrence-worker`
- `connector-accessanalyzer`
- `connector-cloudtrail-ref`
- `connector-ssm-compliance`

### Should be feature-flagged or deferred
- `bedrock-orchestrator`
- `narrative-draft-service`
- contradiction-analysis service
- graph projection worker
- assessor-question prep service

---

## Helm packaging model

## Recommended chart structure

Use a **small chart family** instead of one chart per tiny component.

```text
helm/
  charts/
    vigilant-base/
    vigilant-core/
    vigilant-workers/
    vigilant-support/
  env/
    prototype/
      core-values.yaml
      workers-values.yaml
      support-values.yaml
```

### Why this is the right prototype shape
- fewer release objects to manage
- easier coordinated versioning across related services
- avoids chart sprawl while the service boundaries are still evolving

---

## Chart responsibilities

## 1. `vigilant-base`

### Purpose
Shared primitives used by all clusters/releases.

### Contents
- namespace templates
- common labels/annotations
- network policies baseline
- service accounts
- common secrets/config references
- PodDisruptionBudget / priority-class helpers where needed

### Notes
This should stay small and boring.

---

## 2. `vigilant-core`

### Targets
Deploy to the `vigilant-core` cluster.

### Packages
- ingress resources
- `api-service`
- `workflow-service`
- `record-service`
- `policy-service`
- `snapshot-service`
- `packet-service`
- `evidence-catalog-service`
- `mapping-service`
- `remediation-service`

### Values examples
- image tags
- replica counts
- ingress hosts
- DB secret refs
- queue endpoints
- object storage config refs
- resource limits for core services

---

## 3. `vigilant-workers`

### Targets
Deploy to the `vigilant-workers` cluster.

### Packages
- `connector-runtime`
- selected connectors
- `normalization-worker`
- `asset-resolution-service` or worker
- `dedupe-recurrence-worker` if enabled
- `notification-service`
- `digest-worker` if enabled
- scheduler/cronjob objects for connector sync or recurring maintenance tasks

### Why bundle these together
They are operationally similar:
- async-heavy
- queue-driven
- retry-prone
- more failure-tolerant than the core control plane

---

## 4. `vigilant-support`

### Targets
Deploy to the `vigilant-support` cluster.

### Packages
- `search-indexer`
- `retrieval-service`
- optional search backend release references if self-hosted in-cluster
- `bedrock-orchestrator` if enabled
- `narrative-draft-service` if enabled
- observability and maintenance jobs that belong in this cluster

### Notes
If AI is disabled for the first increment, this chart still makes sense because search and ops support likely remain useful.

---

## Release model

A clean first prototype release layout is:

```text
helm upgrade --install vigilant-base ...
helm upgrade --install vigilant-core ...
helm upgrade --install vigilant-workers ...
helm upgrade --install vigilant-support ...
```

One release family per cluster role is enough.

---

## Service topology details

## Core request path

### Primary synchronous path
1. operator or service calls `api-service`
2. `api-service` validates auth/scope
3. request flows into `workflow-service` and/or `record-service`
4. `policy-service` decides gating and visibility rules
5. state changes are persisted in the authoritative store
6. async events are emitted for workers/search/notifications as needed

### Design rule
Only the core cluster should be able to authoritatively mutate critical workflow and record state.

---

## Worker/event path

### Primary async path
1. connector or worker receives/pulls source data
2. writes normalized work items/events
3. `normalization-worker` transforms source payloads
4. `asset-resolution-service` enriches scope/ownership
5. workflow/task records are created through the core APIs or command path
6. `notification-service` reacts to approved workflow triggers

### Design rule
Workers do not become a shadow control plane. They feed the authoritative one.

---

## Search/retrieval path

1. core services publish approved searchable records/events
2. `search-indexer` updates indexes
3. `retrieval-service` assembles policy-approved context
4. optional AI services consume retrieval output, not raw stores

### Design rule
Retrieval is downstream of authoritative state and policy.

---

## AI path if enabled

1. user or workflow requests a draft summary
2. `policy-service` confirms allowed operation
3. `retrieval-service` assembles bounded context with citations
4. `bedrock-orchestrator` performs the model call
5. draft output is stored as derivative content with provenance
6. human reviews before publication/use in consequential contexts

### Design rule
AI is never allowed to bypass retrieval, policy, or human review.

---

## Suggested values structure

```text
helm/
  env/
    prototype/
      global.yaml
      core-values.yaml
      workers-values.yaml
      support-values.yaml
```

### `global.yaml`
Shared settings:
- environment name
- account/region metadata
- common image registry
- shared tags/labels
- object store endpoints/refs
- queue namespace/prefixes

### `core-values.yaml`
- ingress hosts
- API replicas
- core service resource requests/limits
- DB secret refs
- packet/snapshot feature toggles

### `workers-values.yaml`
- enabled connectors
- queue subscriptions
- cron schedules
- retry settings
- notification channel toggles

### `support-values.yaml`
- search config
- retrieval feature flags
- AI enablement flags
- observability component sizing

---

## Operational rules for the prototype topology

### Rule 1: Keep authoritative state centralized
Even with three clusters, the control plane should still feel like one system with one authoritative record path.

### Rule 2: Prefer a few coarse services
The prototype should not split every domain noun into its own deployment unless there is a real operational reason.

### Rule 3: Isolate noisy workers
Connectors, importers, and retry-heavy jobs should stay off the main core cluster.

### Rule 4: Treat search/AI as consumers
Search, retrieval, and AI services should consume approved state and approved corpora, not invent alternate truths.

### Rule 5: Make AI optional
The support cluster topology should still work if all AI services are disabled.

---

## Out of scope for this Helm topology

- multi-tenant chart overlays
- per-customer release isolation
- service mesh complexity unless the real environment already requires it
- canary and progressive delivery sophistication beyond prototype need
- one chart per connector unless operationally necessary later
- self-inflicted chart templating cleverness that makes reviews harder

---

## Verified facts vs working assumptions summary

### Verified facts
- The repo’s architecture direction already separates core, connectors, search, AI, notifications, and ops.
- The repo explicitly recommends a namespace model matching that separation.
- The repo also recommends a coarse-grained early architecture rather than an oversized microservice mesh.

### Working assumptions
- The 3 small EKS clusters should be split into core, workers, and support roles.
- Helm should group related services into a few chart families rather than many tiny charts.
- AI should be optional and naturally isolated onto the support cluster.

---

## Bottom line

For the first bounded prototype, the cleanest Helm/service posture is:

**one authoritative core cluster, one noisy worker cluster, one support/search/AI cluster, all deployed through a small family of coarse Helm charts with AI kept explicitly optional.**
