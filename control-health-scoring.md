# Control Health Scoring Model

## Goal

Define a **control-health scoring model** for Vigilant ISSO that is useful for real post-certification operations and resistant to fake-compliance theater.

The model should help ISSOs answer:
- which controls are drifting
- which controls are poorly evidenced
- which controls are being contradicted by current telemetry
- which controls are carrying too much unresolved remediation or exception debt
- how much confidence we should place in the current narrative

It should **not** answer:
- "you are 92% compliant"
- "this control is fine because the dashboard is green"
- "a convincing summary is equivalent to evidence"

---

## Core position

A serious continuous-compliance platform should score **control health**, not "compliance percentage."

That means:
- score dimensions separately
- preserve negative signals even when the average looks decent
- expose confidence and contradictions directly
- require human determination for materially consequential readiness conclusions

The right output is a **profile**, not a vanity number.

---

## Verified facts

1. The CMMC Program uses NIST SP 800-171 Rev. 2 as the baseline standard for Level 2 in the current rule. [1]
2. NIST SP 800-171 applies to components that process, store, or transmit CUI and to components that provide protection for such components. [2]
3. NIST SP 800-171A provides assessment procedures and methodology whose findings and evidence support risk-based decisions; assessments may vary in depth and coverage. [3]
4. AWS Config records resource configuration state and history over time, enabling drift and temporal evidence review. [4]
5. AWS CloudTrail records actions taken by users, roles, and AWS services and supports long-retention immutable event-data storage through CloudTrail Lake. [5]
6. Security Hub, GuardDuty, Inspector, Access Analyzer, and Systems Manager provide machine-observable inputs useful for detecting control pressure, validation evidence, and visibility gaps. [6][7][8][9][10]

---

## Design principles

1. **No single score can hide a critical failure.** A control with stale evidence and open high-severity findings should never look healthy because other dimensions are strong.
2. **Evidence quality matters as much as evidence presence.** Attestation-only support should not score like direct machine observation.
3. **Contradictions are first-class signals.** Conflict between narrative and telemetry is a major readiness issue even when no one has yet marked the control failed.
4. **Aging matters.** Old unresolved findings and remediation debt are evidence of weak control operation.
5. **Confidence must be explicit.** If telemetry coverage is degraded or narratives are derivative, the platform should say so.
6. **Human review remains the decision point for material readiness claims.**

---

## The seven dimensions

Each control gets a dimension score from **0 to 4**.

- **4 = strong / current / low concern**
- **3 = acceptable but watchable**
- **2 = weakened**
- **1 = poor**
- **0 = failed, missing, or materially unreliable**

### Dimension 1: Evidence freshness

Question:
**How current is the evidence supporting this control for the present assessment period?**

Inputs:
- age of primary evidence
- freshness policy by evidence type
- whether change events occurred after the evidence was collected
- whether the evidence was superseded or revoked

Suggested scoring:
- **4**: all required primary evidence current
- **3**: minor warning threshold crossed, but no primary evidence stale
- **2**: at least one primary evidence item in warning/stale transition, limited impact
- **1**: primary evidence stale for meaningful part of control story
- **0**: no current primary evidence or current evidence invalidated by change

### Dimension 2: Evidence strength

Question:
**How strong is the evidence basis behind the control narrative?**

Inputs:
- ratio of direct machine observations to manual/attestation artifacts
- whether evidence chain includes technical validation for implementation claims
- source provenance quality
- derivative-summary dependence

Suggested scoring:
- **4**: direct machine evidence plus supporting documentation and provenance intact
- **3**: mostly direct/documentary evidence with minor gaps
- **2**: mixed strength, substantial dependence on screenshots/manual uploads/derived summaries
- **1**: attestation-heavy or weak documentary basis
- **0**: narrative effectively unsupported by acceptable evidence

### Dimension 3: Contradiction pressure

Question:
**How much active contradiction exists between what the control claims and what current observations show?**

Inputs:
- narrative vs telemetry conflicts
- policy/standard vs observed-state conflicts
- stale-evidence vs confident-summary conflicts
- exception register vs reported implementation conflicts

Suggested scoring:
- **4**: no unresolved contradictions
- **3**: minor contradiction under active review, low materiality
- **2**: one material contradiction or several low-materiality contradictions
- **1**: persistent contradiction across review cycle
- **0**: major contradiction unresolved or actively worsening

### Dimension 4: Open findings pressure

Question:
**How much negative operational telemetry is currently touching this control?**

Inputs:
- open findings count
- severity weighting
- in-scope asset count affected
- recurrence of the same issue family
- source diversity of corroborating findings

Suggested scoring:
- **4**: no active findings or only low-risk isolated issues
- **3**: low/moderate findings with limited affected scope
- **2**: repeated moderate findings or one high finding with bounded scope
- **1**: multiple high or recurring findings affecting in-scope assets
- **0**: critical or systemic unresolved finding pressure

### Dimension 5: Remediation aging pressure

Question:
**How much unresolved corrective-action debt has built up against this control?**

Inputs:
- average and max aging of open remediation items
- overdue counts
- reopened items
- closure requests blocked for poor validation

Suggested scoring:
- **4**: no overdue items; aging within policy
- **3**: some aging but within acceptable watch thresholds
- **2**: one overdue item or rising aging trend
- **1**: multiple overdue or repeatedly reopened items
- **0**: severe unresolved debt materially affecting readiness credibility

### Dimension 6: Exception burden

Question:
**How much of the control posture depends on temporary exceptions, waivers, or compensating measures?**

Inputs:
- count of active exceptions
- duration of exceptions
- proximity to expiry
- compensating-control evidence strength
- percentage of control narrative resting on exception logic

Suggested scoring:
- **4**: no active exceptions or only tightly bounded low-impact exception
- **3**: one active exception with strong compensating evidence and good review discipline
- **2**: multiple or long-lived exceptions, but still governed
- **1**: exception-heavy posture or weak compensating evidence
- **0**: expired, unreviewed, or effectively permanent exception dependence

### Dimension 7: Narrative confidence

Question:
**How much should a human trust the current control summary without reopening the evidence set from scratch?**

Inputs:
- recency of human review
- completeness of citations in current narrative
- telemetry coverage confidence
- evidence freshness/strength roll-up
- contradiction state
- proportion of narrative generated from derivative artifacts vs primary evidence

Suggested scoring:
- **4**: recently human-reviewed, well-cited, evidence-backed, and visibility coverage strong
- **3**: trustworthy but due for routine review soon
- **2**: narrative usable with caution; some dependencies on weaker or aging inputs
- **1**: narrative confidence low because review is old, citations weak, or visibility incomplete
- **0**: current narrative should not be relied on for readiness or assessor support

---

## Why these are separate dimensions

These dimensions are intentionally not interchangeable.

Examples:
- a control may have **strong evidence strength** but **poor freshness** because last quarter's exports were solid but now stale
- a control may have **current evidence** but **high contradiction pressure** because new telemetry conflicts with the narrative
- a control may have low current finding pressure but still have heavy **exception burden** and weak **narrative confidence**

That is exactly why a single green number is misleading.

---

## Recommended derived outputs

## 1. Control health profile

Primary UI object:

```text
AC.L2-3.1.x
Freshness: 2
Strength: 3
Contradiction: 1
Findings: 2
Remediation: 2
Exceptions: 4
Narrative confidence: 2
```

This should be the default view.

## 2. Operational health state

Derived state focused on day-to-day posture:
- **healthy**
- **watch**
- **degraded**
- **critical**

Suggested logic:
- **critical** if any of the following:
  - contradiction pressure = 0
  - open findings pressure = 0
  - remediation aging pressure = 0
- **degraded** if two or more dimensions are 1 or lower, or any one of freshness/strength/narrative confidence is 0
- **watch** if no critical conditions but one or more dimensions are 2
- **healthy** only if all dimensions are 3 or 4

## 3. Assessment readiness state

Separate from operational health:
- **reviewable**
- **reviewable_with_gaps**
- **not_ready**
- **pending_human_determination**

Suggested logic:
- **not_ready** if freshness = 0, strength = 0, narrative confidence = 0, or contradiction pressure = 0
- **reviewable_with_gaps** if any dimension is 1 or 2 but no hard-stop dimension is 0
- **reviewable** only if freshness, strength, contradiction pressure, and narrative confidence are all at least 3 and no severe aging/exception issue exists
- **pending_human_determination** whenever the control relies on compensating controls, active boundary ambiguity, or unresolved material exception review

This separation prevents the platform from confusing operational calm with assessment readiness.

---

## Optional aggregate index: use carefully

If leadership insists on one roll-up number, use a **control health index** only as a secondary summary, never as the primary truth.

### Suggested formula

```text
BaseIndex =
  0.18 * Freshness
+ 0.16 * Strength
+ 0.18 * ContradictionPressure
+ 0.16 * OpenFindingsPressure
+ 0.12 * RemediationAgingPressure
+ 0.10 * ExceptionBurden
+ 0.10 * NarrativeConfidence

ScaledIndex = BaseIndex * 25   # 0..100
```

### But apply hard-stop penalties

If any of these conditions occur, cap the scaled index:
- freshness = 0 -> max 39
- strength = 0 -> max 39
- contradiction pressure = 0 -> max 34
- narrative confidence = 0 -> max 34
- more than two dimensions at 1 or lower -> max 49

### Display rule
If shown at all, the index must appear alongside the dimension profile and readiness state, for example:

```text
Control Health Index: 46 / 100 (capped due to stale primary evidence)
Assessment Readiness: not_ready
```

The cap explanation matters more than the number.

---

## Input normalization rules

## Evidence freshness normalization

Recommended freshness categories before 0-4 conversion:
- current
- warning
- stale
- invalidated

Map to score:
- current -> 4
- warning -> 3
- stale -> 1 or 2 depending on control criticality and evidence role
- invalidated -> 0

## Evidence strength normalization

Suggested evidence strength classes:
- direct_machine_observation
- direct_documentary_record
- remediation_validation_result
- controlled_manual_record
- attestation_only
- interview_only
- inferred_or_derived_summary

Example weighting for strength computation:
- direct_machine_observation = 1.00
- remediation_validation_result = 0.95
- direct_documentary_record = 0.80
- controlled_manual_record = 0.65
- attestation_only = 0.35
- interview_only = 0.25
- inferred_or_derived_summary = 0.15

Then combine using the strongest relevant evidence plus diversity bonus, not simple averaging. Otherwise a pile of weak artifacts can fake strength.

## Open findings pressure normalization

Recommended severity weights:
- informational = 0
- low = 1
- medium = 3
- high = 6
- critical = 10

Pressure inputs:
- weighted active finding sum
- number of in-scope assets affected
- recurrence multiplier
- corroboration multiplier if multiple sources point to the same issue

Example:

```text
FindingsPressureRaw =
  SeverityWeightedSum
  * ScopeMultiplier
  * RecurrenceMultiplier
  * CorroborationMultiplier
```

Then bucket into 0-4 scores using environment-tuned thresholds.

## Remediation aging normalization

Recommended inputs:
- oldest open item age
- median age
- overdue ratio
- reopen count
- blocked-closure count

Important rule:
A recently updated ticket is not automatically a low-aging-risk ticket. Aging pressure should follow elapsed unresolved time, not comment activity.

## Exception burden normalization

Recommended factors:
- active exception count
- percent of control narrative affected
- exception duration relative to approved window
- expiry proximity
- compensating-control evidence strength

Expired exceptions should heavily penalize the score even if compensating-control text exists.

## Narrative confidence normalization

Recommended factors:
- human review age
- citation coverage
- freshness score
- strength score
- contradiction score
- collector/visibility health

A simple practical formula:

```text
NarrativeConfidenceRaw =
  0.25 * HumanReviewRecency
+ 0.20 * CitationCoverage
+ 0.20 * EvidenceFreshness
+ 0.15 * EvidenceStrength
+ 0.10 * VisibilityCoverage
+ 0.10 * ContradictionHealth
```

Then downgrade to 0 immediately if the narrative depends on evidence already marked invalidated or if visibility coverage is critically impaired.

---

## Scoring workflow

### Step 1: Collect source facts
Collect current findings, evidence objects, remediation items, exceptions, scope state, collector health, and prior reviewer determinations.

### Step 2: Compute dimension candidates
Use deterministic rules first. Use model-assisted reasoning only to classify contradiction narratives or evidence-role interpretation where the rules need help.

### Step 3: Apply hard-stop rules
If certain evidence or contradiction conditions exist, apply caps or force readiness states.

### Step 4: Present for human review where material
If a control changes readiness state, enters not_ready, or crosses a policy-defined threshold, route for ISSO/control-owner review.

### Step 5: Preserve score history
Store prior dimension values and reasons so reviewers can answer:
- what changed
- why it changed
- what evidence caused the change
- who confirmed or overrode it

---

## Anti-gaming rules

To avoid fake compliance scoring, Vigilant ISSO should enforce these rules:

1. **No positive averaging away critical defects.** One major contradiction or invalidated primary evidence can cap the control's overall result.
2. **No summary-as-evidence substitution.** Generated narratives and derived summaries cannot raise evidence strength above the underlying evidence basis.
3. **No ticket-closure inflation.** Closed status without validation evidence does not improve remediation aging or open-findings pressure enough to restore health.
4. **No exception laundering.** Accepted risk does not mean the control is healthy; exception burden remains visible.
5. **No invisibility bonus.** Missing telemetry reduces confidence; it never improves health.
6. **No stale-but-green state.** A control with stale primary evidence cannot be shown as fully reviewable.
7. **No hidden overrides.** Human overrides must record rationale, reviewer, and expiration/review date.

---

## Example control profiles

## Example A: operationally noisy but well evidenced

```text
Freshness: 4
Strength: 4
Contradiction: 3
Findings: 1
Remediation: 2
Exceptions: 4
Narrative confidence: 4
Operational health: degraded
Assessment readiness: reviewable_with_gaps
```

Interpretation:
The evidence basis is strong, but active unresolved findings are hurting day-to-day posture.

## Example B: quiet dashboard, weak readiness

```text
Freshness: 1
Strength: 2
Contradiction: 2
Findings: 4
Remediation: 4
Exceptions: 3
Narrative confidence: 1
Operational health: degraded
Assessment readiness: not_ready
```

Interpretation:
There may be few active findings, but the control is not assessment-ready because the evidence and narrative are weak.

## Example C: exception-heavy stability

```text
Freshness: 3
Strength: 3
Contradiction: 3
Findings: 3
Remediation: 3
Exceptions: 1
Narrative confidence: 2
Operational health: watch
Assessment readiness: pending_human_determination
```

Interpretation:
The control is being propped up by exception logic and needs governance review.

---

## Recommended UI presentation

For each control, show:
- dimension profile
- operational health state
- assessment readiness state
- top three reasons for current state
- recent trend (30/90 days)
- cited evidence and contradictions
- explicit human overrides or pending decisions

Avoid:
- giant roll-up percentage on the first screen
- unlabeled green/yellow/red without reasons
- charts that hide visibility gaps or exception dependence

---

## Human review policy

Human review should be required when:
- readiness changes to **not_ready**
- contradiction pressure drops to 1 or 0
- primary evidence becomes invalidated/stale for a critical control area
- a control depends materially on compensating controls or exceptions
- the model proposes a status improvement after a long degraded period

The point is not to make scoring manual. The point is to keep consequential interpretation governed.

---

## Verified facts vs assumptions

### Verified facts
- CMMC Level 2 relies on NIST SP 800-171 Rev. 2 in the current program rule. [1][2]
- NIST 800-171A uses evidence and findings to support risk-based assessment decisions. [3]
- AWS Config and CloudTrail provide time-based evidence relevant to freshness, drift, and auditability. [4][5]
- AWS Security Hub, GuardDuty, Inspector, Access Analyzer, and Systems Manager provide machine-observable inputs relevant to findings pressure, validation evidence, and visibility monitoring. [6][7][8][9][10]

### Working assumptions
- A 0-4 dimension model is easier to explain to operators and leaders than a pseudo-precise decimal risk score.
- Contradiction pressure is one of the most valuable early indicators of post-certification drift and deserves equal weighting with evidence freshness.
- Narrative confidence should be a scored dimension because many compliance failures come from over-trusting stale or derivative summaries.
- An optional aggregate index can be useful for trend reporting if and only if hard-stop caps prevent false reassurance.

---

## Sources

1. eCFR, "32 CFR Part 170 — Cybersecurity Maturity Model Certification (CMMC) Program." https://www.ecfr.gov/current/title-32/subtitle-A/chapter-I/subchapter-G/part-170
2. NIST, "SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations." https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
3. NIST, "SP 800-171A, Assessing Security Requirements for Controlled Unclassified Information." https://csrc.nist.gov/pubs/sp/800/171/a/final
4. AWS, "What Is AWS Config?" https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
5. AWS, "What Is AWS CloudTrail?" https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
6. AWS, "Introduction to AWS Security Hub CSPM." https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
7. AWS, "What Is Amazon GuardDuty?" https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html
8. AWS, "What Is Amazon Inspector?" https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html
9. AWS, "Using AWS Identity and Access Management Access Analyzer." https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
10. AWS, "What is AWS Systems Manager?" https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html
