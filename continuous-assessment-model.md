# Continuous Assessment Model

## Goal

Define a continuous-compliance operating model for Vigilant ISSO that helps an organization **stay inside CMMC Level 2 expectations after initial certification**, rather than drifting back into a scramble-driven posture.

The model is intentionally operational, not theatrical. It assumes:
- a mixed **AWS GovCloud + on-prem** environment
- a CUI assessment boundary with inherited/shared services that must be tracked explicitly
- a security/compliance team that needs durable routines, not just dashboards
- human reviewers remain accountable for significant control and evidence decisions

---

## Core position

CMMC readiness should be treated as a **continuous assessment program** made of recurring observations, evidence refresh, triage, remediation verification, and control-owner review.

The platform should answer, every day:
1. What changed in the environment?
2. Which changes create new control risk or evidence decay?
3. Which open issues threaten ongoing compliance credibility?
4. Which controls have stale, weak, or contradictory evidence?
5. What must an ISSO review now versus later?

---

## Verified facts

1. The CMMC Program establishes requirements for contractors and subcontractors to implement prescribed cybersecurity standards for systems that process, store, or transmit FCI or CUI, or provide security protections for such systems. [1]
2. The CMMC Program uses NIST SP 800-171 Rev. 2 as the baseline for Level 2 and applies to scoped contractor information systems. [1]
3. NIST SP 800-171 states that the requirements apply to components of nonfederal systems and organizations that process, store, and/or transmit CUI, or provide security for such components. [2]
4. NIST SP 800-171A provides assessment procedures and methodology that can be used for self-assessments, independent third-party assessments, and government-sponsored assessments, with different depth and coverage. [3]
5. AWS Config records resource configuration state and history over time, and AWS CloudTrail records user, role, and service activity; both are useful for temporal evidence and drift detection. [4][5]
6. AWS GovCloud has service-availability differences that must be accounted for in architecture and operating procedures; for EKS specifically, some commercial-region features are unavailable in GovCloud. [6][7]

---

## Design principle

The right unit of work is not “run an annual assessment.”

It is:
- collect observations continuously
- re-evaluate control posture on a recurring cadence
- force evidence freshness discipline
- keep remediation and exceptions current
- create immutable monthly and pre-assessment snapshots

That is how an organization avoids falling out of compliance between certification events.

---

## Operating model

### 1. Four assessment loops

#### A. Event-driven loop
Triggered by new findings, drift, failed control checks, major changes, or evidence invalidation.

Examples:
- GuardDuty high-severity finding
- Config rule noncompliance on an in-scope asset
- on-prem scanner import showing critical vulnerabilities
- privileged access group membership change
- SSP-controlled setting changed without approved change record

Expected outcome:
- open or update an operational case
- propose impacted controls
- identify required evidence or remediation
- route to the correct owner and ISSO queue

#### B. Daily hygiene loop
Focused on keeping operational debt from accumulating.

Daily checks:
- newly ingested findings lacking owner or scope classification
- evidence that crossed warning/stale thresholds
- remediation items nearing SLA breach
- unresolved contradictions between telemetry, SSP claims, and prior narratives
- failed collector jobs or ingestion gaps

Expected outcome:
- a daily ISSO queue with small, concrete actions

#### C. Monthly control review loop
A structured review for control owners and the ISSO team.

Monthly review should answer:
- which controls have weak or decaying evidence?
- which controls depend on manual attestations too heavily?
- which open findings are aging into compliance risk?
- which exceptions are nearing expiration?
- which inherited/shared services need refreshed dependency review?

Expected outcome:
- monthly readiness snapshot
- control-owner attestations where required
- refreshed evidence plan for the next cycle

#### D. Quarterly / pre-assessment assurance loop
A deeper rehearsal against the assessment model.

Activities:
- boundary and scoping re-validation
- spot-check assessment objectives against stored evidence
- packet generation for selected Level 2 practices
- contradiction and stale-evidence review
- sampling of closed remediation items for validation quality

Expected outcome:
- discovery of silent drift before an assessor does

---

## Assessment objects

Vigilant ISSO should continuously score and track six object types:

1. **Systems / enclaves**
   - assessment boundary membership
   - inherited/shared service dependencies
   - major change events

2. **Assets / resources**
   - configuration posture
   - vulnerability state
   - exposure state
   - ownership

3. **Controls / practices**
   - evidence completeness
   - evidence freshness
   - contradiction status
   - open issue pressure

4. **Evidence objects**
   - freshness
   - provenance integrity
   - suitability for current assessment period
   - strength class

5. **Remediation items**
   - aging
   - overdue state
   - closure validation quality
   - recurrence

6. **Exceptions / accepted risk records**
   - expiration
   - compensating-control evidence
   - approval provenance
   - review cadence

---

## Control health model

Do not reduce everything to a single green/yellow/red compliance light.

Use a control health profile with these dimensions:

### 1. Evidence completeness
Do we have the evidence types expected for this control?

States:
- complete
- partial
- weak
- missing

### 2. Evidence freshness
Is the evidence still current for its intended use?

States:
- current
- warning
- stale

### 3. Observation pressure
How much active negative telemetry is touching this control?

Inputs:
- count of open findings
- severity weighting
- affected in-scope assets
- recurrence/trend

### 4. Remediation pressure
How much unresolved corrective action is still open?

Inputs:
- overdue items
- average aging
- repeated re-openings
- pending validations not completed

### 5. Narrative confidence
How trustworthy is the platform’s current control summary?

Inputs:
- direct evidence strength
- contradiction count
- human review recency
- derivative-summary dependence

### 6. Exception burden
How much of the control posture depends on temporary exceptions or compensating measures?

---

## Recommended status model

For each control, maintain both:

### Operational health
- healthy
- watch
- degraded
- critical

### Assessment readiness
- reviewable
- reviewable_with_gaps
- not_ready
- pending_human_determination

This prevents the system from confusing “operationally quiet” with “assessment-ready.”

---

## Observation-to-action pipeline

### Step 1: Ingest and preserve
Collect:
- AWS Security Hub / ASFF findings
- Config snapshots and noncompliance results
- CloudTrail and CloudTrail Lake evidence references
- Inspector findings
- Access Analyzer findings
- Systems Manager inventory/patch/compliance data
- on-prem scanner/configuration outputs
- identity and access review outputs
- policy/procedure/evidence repository metadata

### Step 2: Scope and normalize
Determine:
- in-scope / supporting / out-of-scope
- system and enclave affiliation
- asset owner
- likely control implications
- evidence provenance

### Step 3: Evaluate rules
Rules should identify:
- required ISSO review
- new remediation recommendation
- evidence invalidation
- stale artifact replacement request
- possible contradiction with prior implementation claim

### Step 4: Route by consequence
- low-risk, high-confidence items → operational queue
- material compliance-impact items → ISSO review queue
- policy/process contradictions → compliance lead review
- boundary or inherited-service ambiguities → escalation queue

### Step 5: Create immutable state updates
Every meaningful state transition should be time-stamped and reviewable:
- proposed mapping
- accepted mapping
- evidence attached
- evidence marked stale
- remediation opened
- remediation validated
- exception approved
- snapshot generated

---

## Continuous-assessment cadences

### Near-real-time or event-driven
- security findings
- public exposure findings
- logging failures
- access anomalies
- collector failures

### Daily
- stale evidence warnings
- overdue remediation review
- ingestion failure review
- control contradiction triage

### Weekly
- issue trend review by system and control family
- review of high-risk controls
- root-cause clustering of repeated findings

### Monthly
- control-owner evidence review
- exception register review
- boundary change review
- immutable readiness snapshot generation

### Quarterly
- mini internal assessment using SP 800-171A-style sampling
- validate a sample of closed items and accepted mappings
- refresh playbooks and evidence expectations

---

## Internal assessment method

Use a practical adaptation of SP 800-171A assessment logic.

For each selected control/practice, examine:
- **specifications**: policy, procedure, SSP, standards, diagrams
- **mechanisms**: configurations, tooling, technical settings, logs
- **activities**: reviews, workflows, changes, approvals, ticket actions
- **individuals**: reviewer notes, attestation records, interview support where required

Then record:
- evidence used
- assessment period
- gaps or contradictions
- confidence level
- required follow-up

This keeps Vigilant ISSO aligned with an assessment mindset instead of a pure alerting mindset.

---

## Anti-drift controls

To specifically prevent post-certification drift, Vigilant ISSO should implement these checks:

### 1. Baseline drift detection
Flag configuration changes against approved baselines for:
- logging
- retention
- MFA / IAM guardrails
- encryption settings
- network exposure
- EKS cluster policy and add-on posture
- host hardening baselines on-prem

### 2. Evidence decay detection
Flag when a control is still narrated as implemented but its supporting evidence is stale, revoked, missing, or superseded.

### 3. Narrative contradiction detection
Compare:
- SSP statements
- policy claims
- current telemetry
- open findings
- exception register
- previously generated control narratives

### 4. Remediation recurrence detection
Detect when the same issue family repeatedly reappears on the same systems or business unit.

### 5. Inheritance drift detection
Detect when a shared service or inherited control source changed ownership, architecture, or evidence quality without downstream updates.

---

## Human review policy

### Must require human review
- control status changes that materially affect readiness
- use of compensating controls
- creation/approval of exceptions
- closure of significant remediation items
- attachment of primary evidence to an assessor packet
- approval of monthly readiness snapshot

### May be automated with review trail
- evidence freshness calculations
- issue deduplication proposals
- likely control mapping proposals
- daily digest generation
- control heatmaps and aging metrics

### Should not be automated
- declaring the organization compliant
- deleting inconvenient evidence
- downgrading risk solely because an LLM produced a persuasive summary
- suppressing contradictory evidence from packet generation

---

## Metrics that matter

Recommended program metrics:
- percent of controls with complete evidence sets
- percent of controls with stale primary evidence
- median age of open remediation items by severity
- percent of closed items with validating evidence
- number of active contradictions between narrative and telemetry
- count of expired or near-expiry exceptions
- number of collector/data-source failures affecting readiness visibility
- recurrence rate of previously closed issue families

These are better leading indicators than a fake “98% compliant” number.

---

## Verified facts vs assumptions for this model

### Verified facts
- CMMC Level 2 depends on scoped systems and uses NIST SP 800-171 Rev. 2 as the baseline in the current CMMC Program rule. [1]
- NIST SP 800-171A supports recurring assessments with variable depth and coverage. [3]
- AWS Config and CloudTrail provide time-based evidence relevant to ongoing review and drift detection. [4][5]
- AWS GovCloud service differences must be considered in platform design and operations. [6][7]

### Working assumptions
- A monthly immutable readiness snapshot is the right minimum cadence for most organizations nearing or just past initial certification.
- Most organizations will accept automated scoring of evidence freshness and control pressure if material conclusions remain review-gated.
- Continuous-assessment value comes more from contradiction detection and stale-evidence discipline than from inventing new alert streams.

---

## Sources

1. eCFR, “32 CFR Part 170 — Cybersecurity Maturity Model Certification (CMMC) Program.” https://www.ecfr.gov/current/title-32/subtitle-A/chapter-I/subchapter-G/part-170
2. NIST, “SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations.” https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
3. NIST, “SP 800-171A, Assessing Security Requirements for Controlled Unclassified Information.” https://csrc.nist.gov/pubs/sp/800/171/a/final
4. AWS, “What Is AWS Config?” https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
5. AWS, “What Is AWS CloudTrail?” https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
6. AWS, “AWS GovCloud (US).” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/
7. AWS, “Amazon Elastic Kubernetes Service in AWS GovCloud (US).” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-eks.html
