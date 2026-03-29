# Terraform Module Map for the MVP Prototype

## Purpose

Define the Terraform structure for the **lean Vigilant ISSO prototype**.

This version assumes Vigilant ISSO itself is a **small EKS-hosted runtime** monitoring a bounded AWS environment, not a large fielded platform spread across the target infrastructure.

---

## Design intent

Terraform should provision only the infrastructure Vigilant ISSO itself needs to run and store trustworthy state.

That means Terraform should focus on:
- small EKS hosting footprint
- IAM and secrets
- object-backed evidence storage
- lightweight structured state backend
- queue/event plumbing if used
- observability basics

Terraform should **not** try to own the entire monitored AWS environment.

---

## Recommended repo shape

```text
infra/
  modules/
    vigilant-runtime/
    vigilant-state/
    vigilant-evidence-store/
    vigilant-iam/
    vigilant-observability/
    vigilant-secrets/
  environments/
    prototype/
      main.tf
      variables.tf
      outputs.tf
      backend.tf
      terraform.tfvars.example
```

This is intentionally small.

---

## Top-level module composition

### 1. `vigilant-runtime`
Provision the small EKS runtime for Vigilant ISSO.

Responsibilities:
- EKS cluster or reference to a pre-existing small cluster
- node groups / worker settings
- OIDC / service-account access plumbing
- baseline networking inputs

### 2. `vigilant-state`
Provision the lightweight structured data backend.

Responsibilities:
- relational database or equivalent managed state backend
- backup settings
- connection outputs and secret refs

### 3. `vigilant-evidence-store`
Provision object-backed storage for:
- evidence references
- packet exports
- derived summaries

### 4. `vigilant-iam`
Provision least-privilege roles for:
- core control service
- MCP-based AWS integrations
- notification components
- optional Bedrock access

### 5. `vigilant-observability`
Provision the minimum useful observability layer.

Responsibilities:
- logs
- alert hooks
- metrics support inputs
- retention policies

### 6. `vigilant-secrets`
Provision secret/config containers and references for Helm deployments.

---

## What Terraform should not own in the prototype

Terraform should not attempt to model all monitored resources across:
- the two VPCs
- the data lake
- the three small EKS clusters

Those are the **targets**.

The prototype may reference them as identifiers or discovery scope, but does not need to provision them.

---

## Environment inputs the prototype needs

The prototype environment stack should take inputs like:
- target AWS account ID
- list of monitored VPC IDs
- identifiers for the data lake boundary/resources
- list of monitored EKS cluster names/ARNs
- notification target config
- feature flags for Bedrock usage

This keeps the prototype pointed at the monitored environment without pretending to manage it.

---

## Bottom line

The right Terraform posture now is:

**provision only the small Vigilant ISSO runtime and its supporting state, while treating the larger AWS account as monitored scope rather than Terraform-owned platform infrastructure.**
