# Helm and Service Topology for the MVP Prototype

## Purpose

Define the Helm release model and in-cluster topology for the **lean Vigilant ISSO prototype**.

This version reflects the updated direction:
- one small EKS-hosted Vigilant ISSO runtime
- MCP pods/services for AWS-native integrations
- small workflow/evidence core
- optional Bedrock support

---

## Recommended runtime topology

For the prototype, Vigilant ISSO itself should run in **one small EKS-hosted runtime**, not three platform clusters.

The three small EKS clusters described in scope are part of the **monitored environment**, not the Vigilant ISSO hosting topology.

### Suggested namespace/service grouping
- `vigilant-core`
- `vigilant-mcp`
- `vigilant-notify`
- `vigilant-ai` *(optional)*
- `vigilant-ops`

---

## Service groups

## 1. `vigilant-core`
Runs the minimum authoritative services:
- `vigilant-api`
- `workflow-engine`
- `evidence-state-service`
- `policy-engine`
- `snapshot-service`
- `playbook-service`
- `audit-event-service`

## 2. `vigilant-mcp`
Runs the MCP-style integrations:
- `mcp-securityhub`
- `mcp-config`
- `mcp-inspector`
- `mcp-guardduty`
- `mcp-access-analyzer`
- `mcp-cloudtrail-ref`
- `mcp-evidence-store`
- optional `mcp-doc-metadata`
- optional `mcp-bedrock`

## 3. `vigilant-notify`
Runs:
- `notification-service`
- `digest-service`

## 4. `vigilant-ai` *(optional)*
Runs:
- `retrieval-service`
- `bedrock-orchestrator`
- `narrative-draft-service`
- optional `playbook-assistant`

## 5. `vigilant-ops`
Runs:
- observability helpers
- scheduler/cron jobs
- backup/maintenance jobs

---

## Recommended chart structure

Use a small chart family:

```text
helm/
  charts/
    vigilant-base/
    vigilant-core/
    vigilant-mcp/
    vigilant-notify/
    vigilant-ai/
  env/
    prototype/
      global.yaml
      core-values.yaml
      mcp-values.yaml
      notify-values.yaml
      ai-values.yaml
```

### Why this works
- simple
- easy to operate
- no chart explosion
- keeps AI explicitly optional

---

## Release model

A clean prototype deployment looks like:

```text
helm upgrade --install vigilant-base ...
helm upgrade --install vigilant-core ...
helm upgrade --install vigilant-mcp ...
helm upgrade --install vigilant-notify ...
helm upgrade --install vigilant-ai ...   # only if enabled
```

---

## Design rule

The Helm topology should make this visually obvious:

- **core services hold workflow truth**
- **MCP services provide bounded AWS access**
- **notifications are downstream**
- **AI is optional and isolated**

---

## Bottom line

The right Helm posture now is:

**one small Vigilant ISSO runtime with a few coarse chart families, not a multi-cluster platform deployment.**
