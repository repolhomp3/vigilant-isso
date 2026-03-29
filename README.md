# Vigilant ISSO

## Concept

Vigilant ISSO is a governed security and compliance operations concept for **AWS GovCloud and on-prem CMMC Level 2 environments**.

The goal is to create an expert ISSO agent that reduces compliance and security-operations tedium without pretending that AI itself is the authorizing official, assessor, or source of truth.

The right framing is:

**an ISSO operations agent that continuously monitors security posture, maps technical reality to control frameworks, gathers evidence, tracks remediation, and helps regulated teams stay inspection-ready.**

---

## Why this project exists

ISSO work is full of repetitive, high-friction tasks such as:
- reviewing findings
- mapping issues to controls
- gathering and organizing evidence
- tracking POA&Ms
- identifying stale artifacts
- following up on remediation owners
- checking drift against approved baselines
- preparing audit support packages
- translating technical findings into compliance language
- comparing policy claims against operational reality

That work matters, but it is tedious, fragmented, and often manual.

Vigilant ISSO should help close that gap.

---

## Core idea

The system should continuously ingest signals from AWS GovCloud and on-prem environments, interpret them against CMMC L2 / NIST-oriented control expectations, organize supporting evidence, and help teams keep security/compliance work current.

It should be able to:
1. collect findings and evidence
2. map findings to control families and practices
3. identify missing, stale, or contradictory evidence
4. create and track remediation / POA&M items
5. summarize current posture in operator and compliance language
6. prepare audit-ready artifacts and briefings
7. support human decision-making without overclaiming compliance truth

---

## Scope

### Environment scope
- **AWS GovCloud** security, audit, and configuration signals
- **On-prem CMMC L2** signals, evidence, and remediation data
- hybrid environments where control evidence is split across multiple systems

### Functional scope
- continuous monitoring support
- evidence collection and cataloging
- control mapping
- POA&M/remediation tracking
- audit-readiness reporting
- narrative generation grounded in source evidence
- notification and escalation for overdue or high-risk issues

---

## What Vigilant ISSO should do

### 1. Posture monitoring
- ingest findings from AWS-native security and audit services
- ingest on-prem vulnerability/configuration/compliance outputs
- identify drift from expected baselines
- correlate issues by system, owner, control family, and severity

### 2. Control-aware reasoning
- map technical findings to:
  - NIST SP 800-171
  - CMMC L2 practices/domains
  - NIST SP 800-53 families where useful
  - AWS guidance / GovCloud service realities
  - internal policy references

### 3. Evidence management
- organize evidence artifacts by control/system/date
- identify stale, missing, or weak evidence
- preserve provenance and source references
- support evidence bundle generation for review/audit

### 4. Remediation / POA&M support
- create and update remediation items
- track owners, due dates, status, and aging
- summarize overdue or high-risk items
- support POA&M-like workflows and reporting

### 5. Audit readiness
- generate control summaries
- prepare briefing views by control family/system/environment
- identify mismatches between documented posture and observed reality
- produce reviewable, source-backed narratives

### 6. Human-in-the-loop compliance ops
- support analysts and ISSOs
- never claim final compliance authority
- always distinguish:
  - verified facts
  - inferred analysis
  - recommendations
  - unresolved questions

---

## Design principles

- evidence first
- citations over vibes
- retrieval-grounded control interpretation
- hybrid-environment aware
- human review for consequential conclusions
- deterministic remediation tracking
- auditable by default
- least privilege for any operational integrations
- no fake “you are compliant” output

---

## Candidate data sources

### AWS GovCloud
- AWS Security Hub CSPM
- AWS Config
- Amazon GuardDuty
- Amazon Inspector
- IAM Access Analyzer
- AWS CloudTrail
- Amazon CloudWatch
- Systems Manager inventory/patch/compliance where applicable

### On-prem / hybrid
- vulnerability scanner outputs
- patch/compliance tool outputs
- endpoint/server configuration baselines
- asset inventory systems
- identity/access review data
- ticketing/remediation systems
- document repositories for SSP/policies/procedures/evidence

---

## Likely architecture shape

### 1. Ingestion layer
Normalize findings, evidence references, and remediation data from cloud and on-prem sources.

### 2. Evidence/state layer
Store:
- findings
- evidence artifacts
- control mappings
- remediation items
- narrative history
- audit package references

### 3. Reasoning layer
Use bounded AI reasoning to:
- summarize findings
- map to controls
- identify evidence gaps
- draft POA&M entries
- generate audit-ready summaries

### 4. Workflow layer
Use deterministic workflows for:
- evidence collection requests
- control review cycles
- remediation follow-up
- notification and escalation
- periodic posture snapshots

### 5. Reporting layer
Produce:
- control dashboards
- evidence completeness reports
- remediation aging reports
- executive briefings
- audit support packets

---

## Key research questions

1. What are the highest-value ISSO tasks to automate first?
2. What should the canonical data model be for findings, controls, evidence, and POA&M items?
3. How should AWS GovCloud and on-prem signals be normalized together?
4. What is the best approach to mapping findings to CMMC L2 / NIST 800-171 practices?
5. How should evidence provenance and freshness be tracked?
6. What outputs are most valuable for auditors, ISSOs, and engineering teams respectively?
7. What actions should be advisory only vs workflow-driven vs approval-gated?
8. What architecture best preserves trust, traceability, and compliance credibility?

---

## Initial deliverables

This project directory should eventually contain:
- `README.md` — project overview
- `research-plan.md` — questions, sources, assumptions, and open issues
- `architecture-options.md` — viable platform designs
- `control-mapping.md` — CMMC L2 / NIST mapping strategy
- `evidence-model.md` — evidence and provenance data model
- `poam-workflow.md` — remediation and tracking model
- `audit-readiness.md` — reporting and audit-support model
- `pitch.md` — value proposition and narrative

---

## Initial take

This could be a very strong product or internal platform if kept disciplined.

The best version is not:
- “AI does compliance”

It is:

**an evidence-grounded ISSO operations agent that helps regulated teams maintain posture awareness, remediation discipline, and audit readiness across AWS GovCloud and on-prem CMMC L2 environments.**

That is serious. And useful.

---

## Current project documents

- `research-plan.md` — verified facts, assumptions, priority ISSO tasks, and research direction
- `architecture-options.md` — recommended platform shape and guardrails
- `control-mapping.md` — how findings/evidence map to CMMC L2 / NIST SP 800-171
- `evidence-model.md` — canonical evidence/provenance data model
- `poam-workflow.md` — remediation and POA&M lifecycle
- `audit-readiness.md` — readiness snapshots, evidence packets, and reporting model
- `continuous-assessment-model.md` — continuous-control review, anti-drift model, and recurring assessment cadence
- `runbooks-and-playbooks.md` — concrete ISSO runbooks and guided human-in-the-loop playbooks
- `notification-model.md` — Teams/email routing, escalation logic, and policy caveats
- `first-10-automations.md` — the first ten automations worth implementing, with triggers, inputs, outputs, guardrails, and autonomy boundaries
- `control-health-scoring.md` — multi-dimensional control-health model that avoids fake compliance percentages and emphasizes evidence, contradiction, debt, and confidence
- `eks-architecture.md` — EKS-hosted platform design and bounded Bedrock reasoning layer
- `implementation-roadmap.md` — phased build plan from concept to MVP, pilot, and controlled expansion
- `eks-component-backlog.md` — EKS service/component backlog, deployment sequencing, and integration notes
- `pitch.md` — sharper product narrative and positioning

## Notes on rigor

The project docs deliberately separate:
- **verified facts** from official sources
- **working assumptions** and design opinions

This matters because the whole concept only works if the platform is trusted as disciplined infrastructure, not compliance theater.
