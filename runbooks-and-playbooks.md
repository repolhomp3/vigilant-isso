# Runbooks and Interactive Playbooks

## Goal

Define concrete ISSO/compliance operations procedures for Vigilant ISSO in a hybrid **AWS GovCloud + on-prem CMMC Level 2** environment.

This document distinguishes between:
- **runbooks**: deterministic, repeatable operational procedures
- **interactive playbooks**: guided human-in-the-loop workflows where Vigilant ISSO prepares context, evidence, drafts, and decision points

---

## Verified facts

1. NIST SP 800-171A is built around assessment methods such as examine, interview, and test, and its procedures are meant to support self-assessments and third-party assessments with varying depth and coverage. [1]
2. AWS Security Hub, AWS Config, GuardDuty, Inspector, Access Analyzer, and Systems Manager provide machine-observable security and compliance signals relevant to continuous review and remediation workflows. [2][3][4][5][6][7]
3. The CMMC Program is concerned with systems that process, store, or transmit CUI and systems that provide security protection for those systems, so runbooks must preserve scoping decisions and evidence chains. [8]

---

## Operating principles

1. **Evidence before narrative.** The system gathers source data first, drafts second.
2. **Every playbook must expose citations.** The operator should always be able to open the source evidence behind a claim.
3. **No silent closure.** Material items cannot close without validation evidence and reviewer identity.
4. **Consequence determines autonomy.** The more a step affects readiness or audit posture, the more explicit human approval it needs.
5. **Scoping is part of the job.** A playbook that ignores scope boundaries will create compliance fiction.

---

## Standard work queues

Vigilant ISSO should maintain at least these queues:

- **New observations**
- **Scope clarification needed**
- **Control mapping review**
- **Evidence stale or missing**
- **Remediation pending owner**
- **Remediation pending validation**
- **Exception / compensating control review**
- **Monthly readiness review**
- **Pre-assessment packet prep**
- **Data-source / collector failure**

---

## Core runbooks

## Runbook 1: New high-severity finding triage

### Trigger
- high or critical finding from GuardDuty, Inspector, Security Hub, Access Analyzer, Config, or approved on-prem source
- or any finding mapped to an in-scope CUI asset with likely Level 2 impact

### Inputs
- raw finding payload
- asset and system metadata
- current scope classification
- existing related remediation items
- relevant control mappings

### Procedure
1. Validate collector success and payload integrity.
2. Confirm whether the affected asset/system is in-scope, supporting, or out-of-scope.
3. Check whether an open systemic remediation item already exists.
4. Attach likely control mappings with confidence and rationale.
5. Determine whether immediate containment coordination is needed.
6. Route to engineering owner and ISSO queue.
7. Create or update remediation record.
8. Record the triage decision and reviewer identity.

### Required outputs
- triage classification
- scope decision
- owner assignment
- severity / urgency confirmation
- remediation linkage
- evidence references

### Human approvals required
- downgrading severity when it affects prioritization materially
- marking the finding out-of-scope where the boundary is ambiguous
- deciding that a repeated finding is fully covered by an existing systemic item

---

## Runbook 2: Stale evidence recovery

### Trigger
- evidence freshness crosses warning or stale threshold
- primary evidence for a control packet becomes invalid or superseded

### Procedure
1. Identify the control(s), packet(s), and narratives that depend on the stale evidence.
2. Determine whether a stronger machine-generated source exists.
3. Open an evidence refresh task to the control owner.
4. Preserve the stale artifact and mark it stale; do not overwrite history.
5. If the stale evidence was primary support for a high-value control, downgrade readiness status pending refresh.
6. Regenerate affected summaries with explicit stale-evidence warning banners.

### Required outputs
- stale evidence event record
- replacement request
- impacted controls list
- readiness impact statement

### Guardrail
The platform must never quietly keep using stale evidence in a control narrative without labeling it.

---

## Runbook 3: Remediation closure validation

### Trigger
- remediation item moved to pending validation

### Procedure
1. Pull implementation notes and claimed completion date.
2. Collect validation evidence from the authoritative source system.
3. Verify the original observation is resolved, risk-reduced, or formally excepted.
4. Confirm no contradictory new findings exist.
5. Record validation reviewer, date, and evidence ids.
6. Close item only after validation evidence is attached.

### Validation patterns
- vulnerability item → post-change scan or agent check
- access item → group/policy export showing corrected access
- logging item → observed test events and retention confirmation
- baseline drift item → current configuration export + approved change reference

### Human approvals required
- closure when evidence is indirect or partial
- closure via compensating control rather than technical fix
- closure where residual risk remains material

---

## Runbook 4: Exception / compensating control review

### Trigger
- owner requests exception
- remediation cannot complete by due date
- operational necessity requires a temporary alternative control posture

### Procedure
1. Link the request to findings, assets, and controls.
2. Require business/mission rationale.
3. Require start date, end date, review date, and approver.
4. Require compensating-control evidence where claimed.
5. Evaluate whether the exception changes current readiness reporting.
6. Publish expiration warning schedule inside the platform state model.

### Required outputs
- exception record
- approval trail
- compensating-control evidence
- expiration and review date
- explicit statement that exception is not equivalent to full compliance

---

## Runbook 5: Monthly readiness snapshot

### Trigger
- month-end or formal monthly review event

### Procedure
1. Freeze current state into an immutable snapshot.
2. Recalculate control-health dimensions.
3. Summarize stale evidence, contradictions, overdue remediation, and exception burden.
4. Present draft snapshot to ISSO/compliance lead.
5. Capture sign-off and follow-up actions.
6. Store packet hash and reviewer identities.

### Required outputs
- monthly readiness summary
- top risks list
- controls needing owner follow-up
- immutable snapshot reference

---

## Runbook 6: Assessment-boundary change review

### Trigger
- new AWS account, VPC, cluster, site, server class, shared service, or identity dependency
- significant architectural change in the CUI enclave

### Procedure
1. Determine whether the change affects systems processing/storing/transmitting CUI or systems providing security protection to those systems.
2. Update boundary record and inheritance relationships.
3. Identify newly required evidence or controls.
4. review whether existing narratives or packets are now incomplete.
5. open follow-up tasks for any missing onboarding evidence.

### Required outputs
- updated scope record
- inherited/shared service changes
- evidence gap list
- control impact summary

---

## Interactive playbooks

## Playbook 1: “Why is this control degraded?”

### Operator goal
Understand why a control moved from healthy/watch to degraded/critical.

### Vigilant ISSO should provide
- current control summary
- all active findings linked to the control
- stale or weak evidence list
- open remediation items
- exception burden
- contradiction view (policy vs telemetry vs narrative)
- citations to source evidence

### Human decision points
- accept the machine-prepared explanation?
- request deeper evidence collection?
- open additional remediation or policy review?
- revise control narrative?

---

## Playbook 2: “Prepare me for an assessor question”

### Operator goal
Generate a reviewable answer for a likely assessor question about one practice/control.

### Inputs
- control identifier
- assessment period
- target system / enclave

### Vigilant ISSO should provide
- requirement text
- current implementation summary draft
- evidence index with observed dates and strengths
- active findings/remediation related to the control
- known gaps or contradictions
- unresolved questions the human should be ready to answer

### Guardrail
The generated response should be labeled draft and should not hide weak evidence.

---

## Playbook 3: “Investigate recurring failure pattern”

### Operator goal
Determine whether repeated findings reflect one systemic control weakness.

### Vigilant ISSO should provide
- clustered findings over time
- affected assets/systems/business units
- common misconfiguration or process root-cause candidates
- prior remediation history
- reopened or repeated issue rates
- likely policy / standard / engineering ownership

### Human decision points
- consolidate into a parent systemic remediation item?
- revise baseline or standard?
- initiate management escalation?

---

## Playbook 4: “Validate a compensating control claim”

### Operator goal
Decide whether an asserted compensating measure is supported enough to present in readiness reporting.

### Vigilant ISSO should provide
- original deficiency and affected controls
- compensating-control narrative draft
- mapped evidence supporting the alternative measure
- gaps, assumptions, and ambiguities
- expiration / temporary-use context

### Human decision points
- accept, reject, or request more evidence
- require additional management approval
- downgrade readiness until stronger proof exists

---

## Playbook 5: “What changed since last month?”

### Operator goal
Brief leadership or assessors on meaningful posture changes.

### Vigilant ISSO should provide
- new high-risk findings
- controls whose readiness worsened or improved
- evidence sets that became stale or were refreshed
- newly approved or expired exceptions
- remediation aging deltas
- boundary or architecture changes affecting scope

---

## Human roles in the workflows

### ISSO / compliance analyst
- scope decisions
- mapping review
- packet review
- exception coordination
- readiness summary approval

### Control owner
- evidence refresh
- procedure/narrative validation
- compensating-control support
- monthly attestations where used

### Engineering / platform owner
- remediation execution
- implementation evidence
- root-cause correction
- change record linkage

### Security operations
- incident or finding triage
- validation signals
- monitoring coverage confirmation

### Authorizing / approving leadership
- exceptions
- accepted risk
- materially significant readiness decisions

---

## Suggested UI/interaction pattern

Each interactive playbook page should show:
- **Facts**: raw findings, evidence metadata, dates, source systems
- **Draft analysis**: system-prepared interpretation with citations
- **Unknowns**: scope ambiguity, missing evidence, conflicting signals
- **Required human decisions**: explicit buttons or workflow states
- **Audit trail**: who accepted what and when

That separation is crucial. It keeps the system useful without letting it pretend analysis is fact.

---

## Verified facts vs assumptions

### Verified facts
- NIST 800-171A supports examine/interview/test-style assessment procedures and variable rigor. [1]
- AWS security services cited here generate signals suitable for workflow triggers and validation evidence. [2][3][4][5][6][7]
- CMMC scope includes systems handling CUI and systems providing security protections to those systems. [8]

### Working assumptions
- Most organizations will get more value from a small number of deeply-governed playbooks than from dozens of shallow automations.
- The best first interactive playbooks are degradation analysis, assessor-question prep, recurrence analysis, and compensating-control validation.
- Monthly readiness review is the minimum viable ritual for keeping post-certification discipline.

---

## Sources

1. NIST, “SP 800-171A, Assessing Security Requirements for Controlled Unclassified Information.” https://csrc.nist.gov/pubs/sp/800/171/a/final
2. AWS, “Introduction to AWS Security Hub CSPM.” https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
3. AWS, “What Is AWS Config?” https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
4. AWS, “What is Amazon GuardDuty?” https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html
5. AWS, “What is Amazon Inspector?” https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html
6. AWS, “Using IAM Access Analyzer.” https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
7. AWS, “AWS Systems Manager Compliance.” https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-compliance.html
8. eCFR, “32 CFR Part 170 — Cybersecurity Maturity Model Certification (CMMC) Program.” https://www.ecfr.gov/current/title-32/subtitle-A/chapter-I/subchapter-G/part-170
