# Terraform Module Map for the MVP Prototype

## Purpose

Define a Terraform module and repository structure appropriate for the **first bounded Vigilant ISSO prototype** only.

This is intentionally a prototype-oriented map, not a grand unified enterprise platform layout.

---

## Design intent

### Verified facts from this repo
- Existing docs recommend an **EKS-hosted control plane** with a relational store, object-backed evidence store, workflow services, and bounded AI integration.
- Existing docs also note GovCloud-specific differences and the need to avoid dependencies that are unavailable there.
- The first prototype is bounded to **1 AWS account, 2 VPCs, 1 data lake, and 3 small EKS clusters**.

### Working assumptions
- Terraform should provision the prototype platform infrastructure and shared environment scaffolding, while Helm handles in-cluster application deployment.
- The first prototype should prefer **coarse-grained modules** and a small number of stacks over deeply abstracted, reusable enterprise module libraries.
- It is acceptable for the first environment to keep some existing resources as inputs rather than forcing Terraform to own every pre-existing AWS object on day one.

---

## Recommended repo shape

### Option A — single infra repo for the prototype
This is the cleanest shape for the first bounded environment.

```text
infra/
  modules/
    aws-account-baseline/
    network-scope/
    eks-cluster/
    data-services/
    object-evidence-store/
    queue-backbone/
    iam-platform-access/
    observability-foundation/
    app-secrets-config/
  environments/
    prototype/
      main.tf
      variables.tf
      outputs.tf
      terraform.tfvars
      backend.tf
```

### Why this is recommended
- one bounded environment does not need a sprawling multi-repo platform taxonomy yet
- easier to review and evolve while the domain model is still moving
- supports fast iteration without pretending module boundaries are fully settled

---

## Recommended top-level module composition

The `environments/prototype` stack should compose a small number of coarse modules:

1. `aws-account-baseline`
2. `network-scope`
3. `eks-cluster` (called 3 times, once per cluster)
4. `data-services`
5. `object-evidence-store`
6. `queue-backbone`
7. `iam-platform-access`
8. `observability-foundation`
9. `app-secrets-config`

That is enough for the prototype.

---

## Module-by-module map

## 1. `aws-account-baseline`

### Purpose
Provision account-level shared settings and tagging conventions needed by the prototype.

### Responsibilities
- common tags and naming conventions
- KMS keys or references for platform encryption needs
- baseline IAM roles or shared policy attachments used across prototype infrastructure
- partition-aware settings for GovCloud-compatible naming/ARN handling

### Inputs
- environment name
- account metadata
- tagging map
- partition/region inputs

### Outputs
- shared KMS key ARNs/IDs
- common tags
- shared account-level role references

### Notes
Keep this narrow. Do not turn it into a giant organizational landing-zone module.

---

## 2. `network-scope`

### Purpose
Represent the prototype’s two-VPC boundary and shared connectivity assumptions.

### Responsibilities
- either create or reference the **2 prototype VPCs**
- manage subnets required for platform deployment where Terraform is the owner
- manage security groups and routing constructs directly needed by the platform
- expose network outputs for the 3 EKS clusters and supporting services

### Inputs
- VPC CIDRs or existing VPC IDs
- subnet definitions or existing subnet IDs
- private/public placement rules
- cluster-to-VPC mapping

### Outputs
- VPC IDs
- subnet IDs
- security group IDs
- cluster placement inputs

### Working assumption
For the first prototype, this module may support a mixed mode:
- **create mode** for net-new bounded environments
- **adopt/reference mode** for pre-existing VPCs

---

## 3. `eks-cluster`

### Purpose
Provision one small EKS cluster suitable for a single Vigilant ISSO role slice.

### Call pattern
Instantiate this module **three times**:
- `eks_cluster_core`
- `eks_cluster_tools`
- `eks_cluster_edge` or equivalent prototype role naming

### Responsibilities
- EKS control plane
- managed node groups or compatible EC2-backed worker groups
- IAM/OIDC integration for service accounts
- cluster security groups and required networking
- baseline add-ons required by the prototype

### Inputs
- cluster name
- VPC/subnet placement
- node sizing and count
- cluster role policies
- add-on toggles

### Outputs
- cluster endpoint
- cluster name
- cluster CA data
- OIDC provider details
- worker role references

### Why use one module called three times
The prototype needs consistency across the three clusters without inventing three separate cluster modules.

### Working assumption about cluster roles
A sensible first split is:
- one cluster for core Vigilant ISSO services
- one cluster for connector/worker and support services
- one cluster for edge/search/AI or controlled experimentation

If the environment already dictates different cluster roles, keep the same module and change the environment wiring.

---

## 4. `data-services`

### Purpose
Provision or reference the authoritative structured data layer for the prototype.

### Responsibilities
- relational database for system-of-record state
- subnet groups / security groups for the DB tier
- backup retention settings appropriate for prototype seriousness
- parameter groups or engine-level settings if needed

### Scope
This module should own the **authoritative structured database** used for:
- findings
- evidence metadata
- mappings
- remediation items
- snapshots
- audit/review state

### Inputs
- engine selection and version
- instance/class sizing
- storage settings
- network placement
- KMS key references

### Outputs
- DB endpoint
- secret references / connection metadata
- security group references

### Working assumption
The prototype should use one main relational backend rather than fragmenting state across many specialized datastores.

---

## 5. `object-evidence-store`

### Purpose
Provision the object-backed evidence and packet storage used by the prototype.

### Responsibilities
- bucket(s) for raw artifacts, packet outputs, and derivative narrative outputs
- lifecycle and retention defaults
- encryption settings
- access policies for application roles

### Suggested bucket separation
Either:
- one bucket with prefixes for `raw/`, `packets/`, `derived/`

or preferably:
- `vigilant-evidence-raw`
- `vigilant-packets`
- `vigilant-derived`

### Why separate
The broader repo repeatedly emphasizes separating:
- raw evidence
- authoritative metadata/state
- derivative narratives

The Terraform layout should reinforce that discipline.

---

## 6. `queue-backbone`

### Purpose
Provision the queue/event primitives needed to decouple ingestion, workflows, notifications, and snapshot jobs.

### Responsibilities
- main work queues
- dead-letter queues
- optional topic/event fan-out primitives where needed
- queue IAM policies for service access

### Suggested MVP queues
- `ingest-events`
- `normalize-events`
- `workflow-events`
- `notify-events`
- `snapshot-events`
- corresponding DLQs

### Notes
Keep the first prototype queue model simple and explicit. Avoid building a giant event taxonomy up front.

---

## 7. `iam-platform-access`

### Purpose
Centralize IAM roles and policies for the prototype platform.

### Responsibilities
- EKS service account roles for core services
- connector roles for AWS-native source access
- database/object/queue access policies
- least-privilege separations between:
  - core services
  - connectors
  - notifications
  - AI orchestration if used

### Why this deserves its own module
IAM sprawl becomes unreadable fast. Even for a prototype, centralizing role intent is worth it.

### Notes
This module should reflect the repo’s design principle that connectors, AI, notifications, and core state are distinct trust zones.

---

## 8. `observability-foundation`

### Purpose
Provision the minimum observability and audit-support infrastructure required for a credible prototype.

### Responsibilities
- log groups / sinks
- metrics and scrape-related base configuration where infrastructure-owned
- alerting destinations or integrations if they are provisioned via Terraform
- storage/retention for audit-support logs and platform telemetry

### Working assumption
Because the repo already notes GovCloud feature differences, this module should avoid assuming commercial-region-only managed observability dependencies.

---

## 9. `app-secrets-config`

### Purpose
Provision secret and configuration containers needed by Helm-deployed services.

### Responsibilities
- secrets manager entries or parameter-store entries
- shared application config references
- connector credential placeholders
- notification integration secrets
- AI integration config placeholders if enabled

### Why keep separate
This makes the Helm layer cleaner and helps preserve a clean line between infra provisioning and in-cluster release configuration.

---

## Environment stack wiring

A prototype root stack should look conceptually like this:

```hcl
module "account_baseline" {
  source = "../../modules/aws-account-baseline"
}

module "network" {
  source = "../../modules/network-scope"
}

module "data_services" {
  source = "../../modules/data-services"
}

module "evidence_store" {
  source = "../../modules/object-evidence-store"
}

module "queue_backbone" {
  source = "../../modules/queue-backbone"
}

module "iam_platform_access" {
  source = "../../modules/iam-platform-access"
}

module "observability" {
  source = "../../modules/observability-foundation"
}

module "app_secrets" {
  source = "../../modules/app-secrets-config"
}

module "eks_cluster_core" {
  source = "../../modules/eks-cluster"
}

module "eks_cluster_tools" {
  source = "../../modules/eks-cluster"
}

module "eks_cluster_edge" {
  source = "../../modules/eks-cluster"
}
```

---

## Proposed repo structure in more detail

```text
infra/
  modules/
    aws-account-baseline/
      main.tf
      variables.tf
      outputs.tf
    network-scope/
      main.tf
      variables.tf
      outputs.tf
    eks-cluster/
      main.tf
      variables.tf
      outputs.tf
      user-data/
    data-services/
      main.tf
      variables.tf
      outputs.tf
    object-evidence-store/
      main.tf
      variables.tf
      outputs.tf
    queue-backbone/
      main.tf
      variables.tf
      outputs.tf
    iam-platform-access/
      main.tf
      variables.tf
      outputs.tf
    observability-foundation/
      main.tf
      variables.tf
      outputs.tf
    app-secrets-config/
      main.tf
      variables.tf
      outputs.tf
  environments/
    prototype/
      backend.tf
      providers.tf
      versions.tf
      main.tf
      variables.tf
      outputs.tf
      terraform.tfvars.example
      README.md
```

---

## What should stay out of Terraform module scope for now

These belong elsewhere or should be deferred:
- detailed application service release definitions → Helm
- per-service Kubernetes manifests → Helm/chart templates
- deep CI/CD orchestration logic → separate delivery pipeline layer
- broad enterprise network landing zone abstraction
- multi-account orchestration framework
- reusable tenant factory abstractions
- advanced data analytics stack beyond prototype necessity

---

## Terraform ownership model

### Terraform should own
- foundational AWS infrastructure for the prototype
- cluster infrastructure
- DB/object/queue infrastructure
- IAM and secrets scaffolding
- observability foundations where infra-scoped

### Helm should own
- namespaces
- deployments/statefulsets/jobs
- services/ingress
- configmaps and app-level release wiring
- per-service autoscaling and rollout settings

### Working assumption
Where existing AWS resources already exist, the first prototype may consume them as data inputs rather than forcing immediate import/refactor work.

---

## State and promotion model

For the prototype, keep state simple:
- one Terraform state for the prototype environment is acceptable initially
- if separation is needed, split only into:
  - `foundation`
  - `clusters`
  - `app-shared`

Do **not** start with a large workspace matrix unless the real environment demands it.

---

## Verified facts vs working assumptions summary

### Verified facts
- The repo’s architecture direction calls for EKS, workflow/state services, relational data, object-backed evidence, and queues.
- The first prototype is bounded to **1 AWS account, 2 VPCs, 1 data lake, and 3 small EKS clusters**.
- Existing docs already warn against overcomplicated service sprawl and GovCloud-incompatible assumptions.

### Working assumptions
- A single prototype-focused infra repo is the best starting shape.
- Helm should be the primary in-cluster deployment layer.
- Three EKS clusters should share one reusable Terraform cluster module with role-specific inputs.
- Existing VPCs or data-lake components may be referenced rather than fully managed on day one.

---

## Bottom line

The right Terraform posture for this prototype is:

**a small set of coarse modules that provision the bounded AWS foundation cleanly, leave app rollout to Helm, and avoid pretending the first environment already needs enterprise-scale infrastructure taxonomy.**
