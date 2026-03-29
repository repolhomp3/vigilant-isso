# POA&M and Remediation Workflow

## Goal

Define a serious remediation workflow for Vigilant ISSO that connects findings, controls, owners, evidence, due dates, exceptions, and closure validation.

This should support POA&M-like discipline without pretending every environment uses the exact same federal artifact format.

---

## Principles

1. **Not every finding becomes a POA&M item.** Duplicates, accepted noise, and low-value ephemeral alerts should not become governance debris.
2. **Every remediation item must be traceable to source observations.**
3. **Closure requires validation evidence, not just status changes.**
4. **Exceptions and compensating measures must be visible.**
5. **Aging matters.** Old unresolved items are posture signals in their own right.

---

## When to create a remediation item

Create a remediation item when at least one of the following is true:
- the finding affects an in-scope or supporting asset/system
- the finding implicates one or more 800-171/CMMC requirements
- the issue is recurring or systemic
- the issue requires tracked ownership and due date
- the issue is likely to matter in an assessment or customer review

Do **not** create a separate item when:
- the finding is a duplicate instance already covered by an open systemic item
- the alert is informational and does not require action
- the source already provides sufficient tracked workflow and Vigilant ISSO is only mirroring state

---

## Canonical remediation record

Fields:
- remediation_id
- title
- description
- source_findings[]
- affected_systems[]
- affected_assets[]
- implicated_controls[]
- risk_rating
- priority
- owner
- supporting_owners[]
- planned_completion_date
- actual_completion_date
- status
- status_reason
- compensating_control_text
- exception_id
- residual_risk_statement
- closure_evidence_ids[]
- verification_reviewer
- verification_date
- aging_days

---

## Lifecycle

### 1. Proposed
Created automatically or by analyst from findings.

Required:
- linked source findings
- affected scope
- suggested owner
- initial control implications
- rationale

### 2. Accepted / triaged
Human confirms the item should be tracked.

Required:
- owner assigned
- due date assigned
- severity/priority reviewed
- duplicate/systemic relationship resolved

### 3. In progress
Work is underway.

Useful updates:
- work notes
- blocker notes
- change request refs
- maintenance window refs
- interim mitigation evidence

### 4. Pending validation
Technical change is believed complete but not yet verified.

Required for entry:
- implementation notes
- post-change scan/check requested
- candidate closure evidence attached

### 5. Closed - validated
Only after evidence shows the issue is actually remediated or an approved compensating measure/exemption path has been accepted.

Required:
- validation evidence
- reviewer identity
- validation date
- residual risk note if relevant

### 6. Deferred / accepted risk / exception
Not closed as fixed, but governance decision recorded.

Required:
- decision authority
- rationale
- expiration/review date if applicable
- compensating controls

---

## Systemic issue handling

Many security findings are symptoms of one underlying process failure.

Example:
- 40 servers missing the same patch baseline

Recommended pattern:
- create one **parent systemic remediation item**
- optionally keep child instance links for asset-specific tracking
- measure both systemic and instance closure

This keeps the workflow useful instead of burying the ISSO in duplicate tickets.

---

## Closure criteria patterns

### Vulnerability item
- affected asset inventory confirmed
- patch or mitigation applied
- verification scan shows resolved or risk-reduced state
- exception documented if asset cannot yet be fixed

### Access control item
- identity/group/policy changed
- change approval captured if required
- validation export confirms access removed or corrected
- follow-up review scheduled if temporary

### Logging/monitoring item
- configuration enabled and retained
- sample events generated/observed
- retention target verified
- review procedure or owner confirmed

---

## Relationship to controls

A remediation item should not just say “fix server.” It should carry control context.

Useful derived views:
- open remediation items by control family
- overdue items affecting multiple controls
- recurring items tied to the same requirement
- closed items lacking validating evidence

---

## Exception handling

Exception/accepted risk records should include:
- scope and affected assets
- linked controls
- business/mission rationale
- approving authority
- start and end dates
- compensating measures
- review cadence
- explicit statement that exception ≠ compliant implementation

This last point matters a lot. Too many systems blur “accepted risk” into “resolved.”

---

## Suggested automation policy

### Safe automation
- propose remediation from findings
- calculate aging and overdue status
- group duplicates
- route to suggested owner
- request validation evidence after change window

### Approval-gated automation
- create formal remediation record in authoritative workflow
- change owner or due date for materially significant items
- mark item as exception/accepted risk
- close item

---

## Reporting outputs

- overdue remediation report
- high-risk open items by system/enclave
- items with no owner
- items with no control mapping
- items closed without validation evidence
- recurring issue report by root-cause category
- assessor-facing POA&M packet with sources and status history

---

## Assumptions

1. Customers will vary between classic POA&M language and more general remediation register language.
2. A strong product should support both without losing rigor.
3. Validation evidence will often come from the same telemetry systems that created the finding, but should be stored as a distinct event/evidence object.

---

## Sources

1. NIST, “SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations.” https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
2. DoD CIO, “CMMC Resources & Documentation.” https://dodcio.defense.gov/CMMC/Resources-Documentation/
3. AWS, “What is Amazon Inspector?” https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html
4. AWS, “AWS Systems Manager Compliance.” https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-compliance.html
5. AWS, “Using IAM Access Analyzer.” https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
