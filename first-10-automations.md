# First 10 Automations

## Goal

Define the **first ten automations that Vigilant ISSO should actually implement** for a post-certification **CMMC Level 2 / AWS GovCloud / on-prem** continuous-compliance operating model.

These are intentionally not "cool demo" automations. They are the ones most likely to reduce real ISSO workload, preserve evidence quality, and catch posture drift before it becomes an assessor problem.

---

## Core position

The first automation set should do four things well:

1. detect meaningful change
2. preserve evidence and provenance
3. force timely human review where compliance consequence is material
4. reduce duplicate manual triage and follow-up work

If an automation cannot clearly improve one of those four outcomes, it should not be in the first wave.

---

## Verified facts

1. The CMMC Program establishes assessment and safeguarding requirements for contractor information systems that process, store, or transmit FCI or CUI; provide security protections for systems that process, store, or transmit CUI; or are not logically or physically isolated from such systems. [1]
2. CMMC Level 2 uses NIST SP 800-171 Rev. 2 as the baseline cybersecurity standard in the current rule. [1]
3. NIST SP 800-171 applies to components of nonfederal systems that process, store, or transmit CUI, or provide protection for such components. [2]
4. NIST SP 800-171A provides assessment procedures and methodology for self-assessments, third-party assessments, and government-sponsored assessments, and the findings and evidence produced during assessments support risk-based decisions. [3]
5. AWS Security Hub CSPM aggregates and normalizes findings across AWS services and standards and supports automation via EventBridge and automation rules. [4]
6. AWS Config records resource configuration state and history over time and can continuously evaluate resources using rules. [5]
7. GuardDuty continuously analyzes AWS data sources and logs to identify suspicious or malicious activity. [6]
8. Amazon Inspector continuously discovers and scans eligible workloads for vulnerabilities and unintended network exposure. [7]
9. IAM Access Analyzer generates findings about external and internal access exposure and can validate IAM policies. [8]
10. AWS Systems Manager can manage nodes across AWS and on-prem environments and provides inventory, patch, and compliance-related capabilities. [9]
11. AWS CloudTrail records management and other events for governance, compliance, operational auditing, and risk auditing, and CloudTrail Lake provides immutable event data stores with long retention options. [10]
12. AWS GovCloud service availability differs from commercial AWS regions, including EKS feature differences that must be accounted for in design and automation expectations. [11][12]

---

## Selection criteria for the first wave

An automation belongs in the first ten if it:
- touches a high-friction ISSO workflow
- produces durable evidence or better evidence routing
- catches post-certification drift early
- works across hybrid GovCloud + on-prem operations
- has clear human-approval boundaries
- can be implemented incrementally without pretending the platform is an assessor

Excluded from the first wave:
- fully autonomous remediation of materially significant security issues
- anything that declares a system or control "compliant"
- automations that only generate polished summaries without improving evidence, routing, or validation quality

---

## Automation autonomy classes

### Advisory
The platform detects, correlates, drafts, or recommends, but does not change authoritative workflow state without human action.

### Approval-gated
The platform prepares a consequential action, but a human must approve before the authoritative state changes.

### Automatic
The platform may perform the action automatically because the consequence is bounded and reversible, and it improves discipline without changing the organization's compliance truth.

---

## Summary table

| # | Automation | Primary purpose | Default autonomy |
|---|---|---|---|
| 1 | In-scope finding intake and deduplication | Turn raw findings into governed work candidates | Automatic |
| 2 | Control-impact mapping proposal | Attach likely CMMC/NIST control implications | Advisory |
| 3 | Evidence freshness and supersedence monitor | Detect evidence decay and invalidate stale narratives | Automatic |
| 4 | Contradiction detector | Surface mismatch between telemetry, SSP, exceptions, and narratives | Advisory |
| 5 | Remediation candidate creation and routing | Create draft remediation/POA&M candidates with owners and due-date logic | Approval-gated |
| 6 | Remediation aging and SLA escalation | Prevent silent drift through overdue work | Automatic |
| 7 | Closure validation evidence check | Block weak or theater-style closure | Approval-gated |
| 8 | Boundary and inheritance drift review | Catch architecture/scope changes that alter control obligations | Approval-gated |
| 9 | Collector-health and visibility-gap monitor | Detect blind spots in telemetry/evidence coverage | Automatic |
| 10 | Monthly readiness snapshot assembly | Produce immutable, reviewable monthly control-health state | Approval-gated |

---

# Automation 1: In-scope finding intake and deduplication

## Why this is in the first ten
ISSO teams lose enormous time to duplicate alert handling, inconsistent scope classification, and manual stitching of AWS/on-prem observations into one queue.

## Trigger
- new finding from Security Hub, GuardDuty, Inspector, IAM Access Analyzer, AWS Config, Systems Manager Compliance, approved on-prem vulnerability/configuration source, or ticket import
- status change on an existing finding

## Inputs
- raw source payload and native id
- source timestamp(s)
- asset/resource identifiers
- system/enclave metadata
- current boundary membership
- known duplicate/systemic-remediation relationships
- finding severity/confidence

## Processing
- normalize the finding into the canonical schema
- resolve asset, system, owner, and in-scope/supporting/out-of-scope classification
- deduplicate against open findings and parent systemic issues
- preserve raw payload, native ids, and ingestion provenance

## Outputs
- normalized finding record
- scope classification proposal or confidence score
- duplicate/systemic linkage
- routing key set for downstream workflows
- evidence object referencing raw payload and metadata

## Guardrails
- never discard the original source payload
- never auto-mark ambiguous boundary cases as out-of-scope
- never collapse repeated findings into a parent without keeping child-instance traceability

## Default autonomy
**Automatic** for normalization and duplicate proposal acceptance when confidence is high and boundary context is already established.

**Escalate to human** when:
- the asset is newly discovered
- scope is ambiguous
- the finding would be hidden by deduplication into an old systemic item

---

# Automation 2: Control-impact mapping proposal

## Why this is in the first ten
Analysts repeatedly perform the same translation: "what control family or practice does this likely affect?" That translation is useful when it is treated as a proposal with citations, not as truth.

## Trigger
- new normalized finding
- change in a finding's severity, scope, or asset classification
- new evidence object attached to a control or remediation item

## Inputs
- normalized finding type
- asset/system metadata
- control catalog and mapping rules
- historical accepted mappings
- SSP/policy references if available

## Processing
- propose likely CMMC Level 2 / NIST SP 800-171 control mappings
- provide rationale tied to finding type, affected asset, and control text
- score mapping confidence
- detect whether the issue appears operational only, evidence-related, or both

## Outputs
- mapping assertion draft(s)
- rationale text with source references
- confidence band
- suggested reviewer queue

## Guardrails
- mappings remain assertions until accepted or overridden by a human
- the platform must show control text and rationale, not just an opaque label
- low-confidence mappings must not drive automatic readiness downgrades by themselves

## Default autonomy
**Advisory**

---

# Automation 3: Evidence freshness and supersedence monitor

## Why this is in the first ten
Evidence decay is one of the fastest ways to fall out of credible readiness after certification.

## Trigger
- scheduled freshness evaluation
- significant change event on a system/control
- new evidence object uploaded or collected
- evidence object marked superseded, revoked, or invalid

## Inputs
- evidence metadata
- freshness policy by evidence type/control
- relationship graph of evidence to controls, packets, and narratives
- change-event feed (for architecture changes, baseline changes, policy updates)

## Processing
- recalculate freshness state: current, warning, stale
- identify superseded evidence chains
- find narratives, packets, and control summaries that still depend on stale or revoked evidence
- generate refresh tasks where replacement is required

## Outputs
- evidence freshness status change event
- impacted controls/packets list
- refresh request queue item
- narrative invalidation flag where needed

## Guardrails
- stale evidence is preserved, not overwritten or deleted
- a generated narrative must visibly show stale-evidence warnings when affected
- evidence cannot be treated as refreshed merely because a summary was regenerated

## Default autonomy
**Automatic**

---

# Automation 4: Contradiction detector

## Why this is in the first ten
The most valuable continuous-compliance signal is often not a new alert, but a contradiction between what the organization says and what telemetry shows.

## Trigger
- new finding on a control with existing implementation narrative
- stale/revoked evidence on a control currently narrated as implemented
- new exception or exception expiry
- SSP/control narrative update
- monthly readiness recalculation

## Inputs
- current control narrative
- SSP/policy excerpts
- active findings and exceptions
- evidence freshness and strength
- prior reviewer decisions

## Processing
- compare current evidence and observations against implementation claims
- classify contradiction type:
  - telemetry vs narrative
  - policy vs observed state
  - exception vs reported health
  - stale evidence vs confident summary
- score contradiction pressure

## Outputs
- contradiction record
- affected controls/systems list
- explanation with citations
- required-review recommendation

## Guardrails
- contradictions must not be silently suppressed because a prior human marked a control healthy
- the system must distinguish contradiction from proof of noncompliance
- contradiction records need explicit disposition: accepted, explained, under review, or resolved

## Default autonomy
**Advisory**

---

# Automation 5: Remediation candidate creation and routing

## Why this is in the first ten
Turning observations into tracked action is where many programs break down. This automation removes clerical work while keeping meaningful governance decisions human-owned.

## Trigger
- new high/critical finding in scope
- repeated medium finding crossing recurrence threshold
- stale primary evidence for a high-value control
- contradiction marked material by rule set

## Inputs
- finding(s) or contradiction record
- control mapping proposals
- asset/system owner metadata
- severity, risk, and recurrence rules
- existing remediation catalog

## Processing
- decide whether to propose a new remediation item or attach to existing systemic item
- suggest owner, due date policy, priority, and parent/child structure
- generate closure criteria template based on issue type

## Outputs
- draft remediation/POA&M record
- proposed owner and due date
- control and asset links
- duplicate/systemic relationship recommendation

## Guardrails
- do not auto-create governance clutter for every low-value alert
- do not auto-assign ambiguous ownership without visible confidence/override path
- do not create closure criteria that can be satisfied only by narrative or attestation when stronger technical validation should exist

## Default autonomy
**Approval-gated** for creation in the authoritative remediation register

The platform may automatically create a *draft candidate* in its own queue, but publishing to the authoritative workflow should require review.

---

# Automation 6: Remediation aging and SLA escalation

## Why this is in the first ten
Readiness drifts when open work ages quietly. Escalation discipline matters more than prettier dashboards.

## Trigger
- remediation item crosses warning, due-soon, overdue, or severely overdue threshold
- owner fails to acknowledge assigned work in policy-defined time
- repeated reopenings exceed threshold

## Inputs
- remediation metadata
- due dates and SLA policy
- exception records
- acknowledgment state
- prior escalation history

## Processing
- calculate aging and overdue state
- bundle reminders by owner/team/system
- escalate according to severity, control impact, and elapsed time
- update control pressure dimensions used in scoring

## Outputs
- reminder/escalation event
- updated remediation pressure metrics
- leadership queue item where needed

## Guardrails
- no reminder storms; bundle and rate-limit
- do not suppress aging because a ticket has recent comments but no real progress
- do not treat exception requests as equivalent to approved exceptions

## Default autonomy
**Automatic** for reminders, bundling, and escalation according to approved policy

---

# Automation 7: Closure validation evidence check

## Why this is in the first ten
A large chunk of fake compliance comes from weak closure: "ticket closed" replacing "problem fixed and evidenced."

## Trigger
- remediation item moved to pending validation or closed-requested state
- exception proposed as closure path

## Inputs
- original finding and remediation record
- claimed implementation notes
- post-change telemetry and validation evidence
- contradictory new findings
- closure criteria template

## Processing
- verify required evidence types are present
- test whether the original issue is actually resolved or materially reduced
- check for reopened or contradictory findings
- score closure evidence quality

## Outputs
- validation result: sufficient, partial, insufficient
- list of missing evidence
- reviewer package with citations
- recommendation to close, hold, or route to exception review

## Guardrails
- material items cannot close on attestation alone when direct validation should exist
- closure validation must record reviewer identity and evidence ids
- if the issue recurs before closure approval, the system must reopen or block closure automatically

## Default autonomy
**Approval-gated**

The platform can automatically block incomplete closure requests, but a human should approve closure of material items.

---

# Automation 8: Boundary and inheritance drift review

## Why this is in the first ten
Post-certification drift often begins with architecture changes that never get translated into control/evidence consequences.

## Trigger
- new AWS account, VPC, subnet, EKS cluster, node group, identity dependency, shared service, site, or server class
- major tag/classification change
- change record marked as affecting CUI enclave or supporting service

## Inputs
- asset and system inventory deltas
- CMDB or scope registry
- approved architecture/baseline records
- inherited/shared-service relationships
- change-management references

## Processing
- detect new or changed boundary-relevant objects
- identify likely impact on scope, inheritance, and required evidence
- open review for missing control coverage or onboarding evidence

## Outputs
- boundary drift case
- impacted systems/controls list
- evidence gap list
- recommended owner/reviewer routing

## Guardrails
- never automatically expand or shrink assessment scope without human approval
- inherited controls must remain explicitly linked; they cannot be assumed from similar architecture alone
- new supporting systems should not remain unreviewed just because they are not directly processing CUI

## Default autonomy
**Approval-gated**

---

# Automation 9: Collector-health and visibility-gap monitor

## Why this is in the first ten
A quiet dashboard can mean either good posture or broken telemetry. The system has to know the difference.

## Trigger
- collector job failure
- unexpected drop in finding volume from a known source
- asset appears in inventory but not in expected telemetry source
- CloudTrail/Config/SSM or on-prem collector lag crosses threshold

## Inputs
- collector run history
- expected source inventory and cadence
- asset inventory
- CloudTrail/Config/SSM/scan coverage metadata

## Processing
- detect missing or delayed collection
- infer likely visibility gaps by source, system, and control area
- assess which control-health dimensions are now less trustworthy

## Outputs
- collector incident record
- visibility-gap impact statement
- affected systems/controls list
- operator runbook trigger

## Guardrails
- do not report improved health when visibility dropped
- scoring must degrade narrative confidence or freshness confidence when evidence collection is impaired
- collector recovery should not retroactively erase the existence of a visibility gap

## Default autonomy
**Automatic**

---

# Automation 10: Monthly readiness snapshot assembly

## Why this is in the first ten
This is the control point that turns daily churn into a durable post-certification operating rhythm.

## Trigger
- approved month-end cadence
- pre-assessment preparation event
- on-demand management review request

## Inputs
- current control-health profiles
- stale evidence list
- contradiction register
- open findings/remediation/exception data
- reviewer notes and unresolved issues

## Processing
- freeze the current state into an immutable snapshot
- calculate dimension scores and readiness states
- assemble top risks, visibility gaps, and unresolved contradictions
- produce reviewer package with citations and change-since-last-snapshot summary

## Outputs
- immutable monthly snapshot record/hash
- control-health summary set
- management/ISSO review package
- explicit blocked-controls list

## Guardrails
- snapshot publication must not imply "certified compliant"
- blocked or degraded controls must remain visible in the published package
- a snapshot without adequate source coverage must be labeled visibility-limited

## Default autonomy
**Approval-gated** for publication/finalization

The draft package can be assembled automatically.

---

## Cross-automation implementation order

Recommended build order:

### Phase 1: observation discipline
1. In-scope finding intake and deduplication
2. Evidence freshness and supersedence monitor
3. Collector-health and visibility-gap monitor

### Phase 2: control-aware workflow
4. Control-impact mapping proposal
5. Remediation candidate creation and routing
6. Remediation aging and SLA escalation
7. Closure validation evidence check

### Phase 3: anti-drift governance
8. Contradiction detector
9. Boundary and inheritance drift review
10. Monthly readiness snapshot assembly

This order matters because a platform that cannot trust its inputs should not be generating high-confidence control narratives.

---

## What should remain out of scope for the first wave

Not first-wave automations:
- automatic patch deployment to in-scope production systems
- automatic risk acceptance or exception approval
- automatic control-status flips from healthy to critical based only on one model output
- autonomous assessor-packet publication without reviewer sign-off
- broad LLM-generated policy rewrites as a substitute for evidence collection

Those can exist later, if ever, but they are not the highest-trust starting point.

---

## Verified facts vs assumptions

### Verified facts
- CMMC scope includes systems handling CUI and systems providing security protection to those systems. [1][2]
- NIST 800-171A supports assessment methodologies that depend on evidence and findings, not just policy claims. [3]
- AWS Security Hub, Config, GuardDuty, Inspector, Access Analyzer, Systems Manager, and CloudTrail provide machine-observable signals useful for event-driven workflows and evidence preservation. [4][5][6][7][8][9][10]
- GovCloud feature differences must be reflected in design and automation expectations. [11][12]

### Working assumptions
- Most organizations will get more immediate value from evidence-governance and workflow automations than from direct technical auto-remediation.
- A monthly immutable readiness snapshot is the minimum viable governance rhythm for a post-certification operating model.
- Approval-gated creation and closure of remediation records is a better default than full automation for CMMC-sensitive environments.

---

## Sources

1. eCFR, "32 CFR Part 170 — Cybersecurity Maturity Model Certification (CMMC) Program." https://www.ecfr.gov/current/title-32/subtitle-A/chapter-I/subchapter-G/part-170
2. NIST, "SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations." https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
3. NIST, "SP 800-171A, Assessing Security Requirements for Controlled Unclassified Information." https://csrc.nist.gov/pubs/sp/800/171/a/final
4. AWS, "Introduction to AWS Security Hub CSPM." https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
5. AWS, "What Is AWS Config?" https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
6. AWS, "What Is Amazon GuardDuty?" https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html
7. AWS, "What Is Amazon Inspector?" https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html
8. AWS, "Using AWS Identity and Access Management Access Analyzer." https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
9. AWS, "What is AWS Systems Manager?" https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html
10. AWS, "What Is AWS CloudTrail?" https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
11. AWS, "AWS GovCloud (US) User Guide." https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/
12. AWS, "Amazon Elastic Kubernetes Service in AWS GovCloud (US)." https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-eks.html
