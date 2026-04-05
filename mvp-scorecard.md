# Vigilant ISSO MVP Scorecard
## Lean GovCloud MVP Evaluation Framework

## Purpose

Define a **practical evaluation framework** for judging whether the lean Vigilant ISSO MVP is actually successful.

This scorecard is meant to evaluate the bounded GovCloud MVP using evidence gathered during the 7-day run, with emphasis on:
- runtime behavior
- documents and outputs produced
- drift detection quality
- remediation behavior
- operator usefulness
- cost discipline

This is deliberately stricter than asking whether the system merely deployed.

---

## 1. Core evaluation question

The MVP should be judged by this question:

**Did the bounded Vigilant ISSO runtime produce trustworthy operational value for one environment at a credible cost, or did it merely generate activity and architecture?**

---

## 2. Evaluation boundary

This scorecard applies only to the first bounded MVP:
- **1 AWS account**
- **2 VPCs**
- **1 data lake**
- **3 small EKS clusters**
- **one lean Vigilant ISSO runtime in us-gov-west-1**

A strong score here does **not** prove broad multi-environment readiness.
It only proves whether the lean MVP deserves continuation.

---

## 3. How to use this scorecard

Each category below should be scored on a **0-5 scale**:

| Score | Meaning |
|---|---|
| **0** | Not demonstrated / failed |
| **1** | Barely present; mostly not usable |
| **2** | Partial and fragile |
| **3** | Credible MVP-level performance |
| **4** | Strong for a lean MVP |
| **5** | Exceptional for this stage |

### Suggested interpretation
- **4.0-5.0 average:** strong MVP; proceed
- **3.0-3.9 average:** viable MVP; proceed with focused fixes
- **2.0-2.9 average:** weak or mixed MVP; narrow and re-run before expanding
- **below 2.0 average:** redesign or stop

### Scoring rule
Scores should be based on:
- direct runtime data
- linked evidence and outputs
- operator observations
- cost data
- explicit reviewer judgment

Not vibes.

---

## 4. Weighted scorecard categories

| Category | Weight | What it asks |
|---|---:|---|
| **Runtime reliability and ingest fidelity** | 20% | Did the system stay up and capture source signals reliably enough to matter? |
| **Evidence and document quality** | 15% | Did it produce traceable, reviewable artifacts instead of vague summaries? |
| **Drift detection usefulness** | 15% | Did it identify meaningful posture drift in a way operators could act on? |
| **Remediation workflow behavior** | 15% | Did it support ownership, aging, validation, and closure discipline? |
| **Operator usefulness and trust** | 20% | Did humans actually find it useful and credible? |
| **Cost discipline and efficiency** | 15% | Did the MVP stay inside the lean budget while still producing value? |

Total weight: **100%**

---

## 5. Category details and scoring prompts

## 5.1 Runtime reliability and ingest fidelity — 20%

### What success looks like
- selected AWS-native sources ingest on a repeatable basis
- failures are visible and debuggable
- normalized records preserve source traceability
- the platform is stable enough to gather a full week's data

### Measure with
- ingest success/failure counts by source
- service outage count and duration
- number of records traceable back to source references
- queue/backlog behavior if applicable
- operator time spent babysitting the system

### Scoring prompts
**0-1**
- ingestion regularly fails
- source traceability is weak or absent
- the system requires constant rescue

**2**
- some sources work, but reliability is inconsistent
- traceability exists in spots but is not dependable

**3**
- selected sources ingest reliably enough for a 7-day MVP
- traceability holds up in normal review
- failures are understandable, not mysterious

**4-5**
- ingestion is consistently stable
- traceability is obvious and defensible
- runtime operations are calm and low-drama

### Suggested evidence
- ingest dashboard or logs
- source-to-canonical walkthrough
- summary of runtime incidents and recoveries

---

## 5.2 Evidence and document quality — 15%

### What success looks like
- evidence metadata is captured with provenance and timestamps
- generated outputs stay distinguishable from primary evidence
- readiness outputs are reviewable and grounded
- final documents produced during the week are useful for follow-on work

### Measure with
- number of evidence items registered
- number of outputs/snapshots generated
- citation/source-link completeness in outputs
- reviewer feedback on whether documents are actually usable

### Documents and outputs that count here
- readiness snapshots
- triage summaries
- evidence gap summaries
- remediation-linked records
- final evaluation artifacts created during or after the run

### Scoring prompts
**0-1**
- outputs are vague or untraceable
- summaries blur evidence and narrative
- documents look polished but are not reviewable

**2**
- some useful documents exist, but they require too much manual explanation

**3**
- documents are reviewable, grounded, and useful at MVP level
- evidence provenance is preserved clearly enough to defend conclusions

**4-5**
- outputs are consistently strong and reduce manual reconstruction effort substantially
- reviewers would willingly reuse the artifacts

### Suggested evidence
- sample readiness snapshot
- sample evidence-linked workflow record
- reviewer comments on output quality

---

## 5.3 Drift detection usefulness — 15%

### What success looks like
- the MVP identifies meaningful deviations from expected posture
- drift is scoped to the right system, cluster, VPC, or shared service context
- the platform surfaces drift in a way that helps triage instead of creating noise

### Measure with
- number of meaningful drift cases surfaced
- false-positive/low-value drift cases noted by operators
- ability to tie detected drift to findings, controls, or remediation actions
- time from drift signal to triage-ready context

### Scoring prompts
**0-1**
- drift detection is absent, noisy, or untrusted

**2**
- some drift can be surfaced, but context or quality is inconsistent

**3**
- drift detection is useful enough to support real triage and review in the bounded environment

**4-5**
- drift signals are consistently relevant, well-scoped, and materially helpful to operators

### Suggested evidence
- examples of detected drift
- examples of ignored/noisy drift
- operator notes on actionability

---

## 5.4 Remediation workflow behavior — 15%

### What success looks like
- findings can enter a durable workflow
- remediation items have owners, status, and aging
- validation evidence is linked before closure
- closure is disciplined rather than cosmetic

### Measure with
- number of remediation items opened
- number assigned to an owner
- number aged/updated during the week
- number validated and closed with linked evidence
- examples of blocked or ambiguous remediation paths

### Scoring prompts
**0-1**
- workflow is mostly a demo path or status theater
- closure happens without evidence or attribution

**2**
- workflow exists, but operators do not trust or follow it consistently

**3**
- at least one credible end-to-end finding-to-remediation-to-validation path is demonstrated
- workflow state is understandable and attributable

**4-5**
- remediation behavior feels operationally real, not staged
- the system clearly improves follow-through and closure discipline

### Suggested evidence
- workflow history for one or more findings
- closure validation examples
- operator feedback on usability of the workflow

---

## 5.5 Operator usefulness and trust — 20%

### What success looks like
- operators understand what the system knows vs infers
- they trust the citations and workflow history enough to act on them
- the MVP reduces scramble work and context reconstruction effort
- at least some users would want to keep using it

### Measure with
- operator interviews or survey notes
- time-to-understand comparisons versus current manual methods
- repeated voluntary use during the 7-day run
- examples where the MVP changed or accelerated a real decision

### Scoring prompts
**0-1**
- operators see it as a demo artifact or another dashboard
- they do not trust the output enough to use it

**2**
- some usefulness is visible, but trust or usability is weak

**3**
- operators find it meaningfully useful for at least a few recurring tasks
- trust is good enough for review and triage work

**4-5**
- operators clearly prefer it to current manual reconstruction for multiple tasks
- trust and usefulness are recurring themes in feedback

### Suggested evidence
- structured feedback notes
- examples of tasks made faster or clearer
- evidence of repeat use during the week

---

## 5.6 Cost discipline and efficiency — 15%

### What success looks like
- the MVP stays within the planned cost envelope
- the major cost drivers are understood
- the team can connect spend to learning and output value
- expensive noise is cut quickly

### Measure with
- total 7-day spend
- daily spend trend
- top cost drivers
- spend above/below target range
- examples of cost corrections during the week

### Scoring prompts
**0-1**
- spend is uncontrolled or poorly understood
- the run exceeds budget without proportional learning

**2**
- spend is partially understood, but guardrails were weak or reactive

**3**
- the run stays near the planned lean range and produces enough evidence to justify the spend

**4-5**
- the run is both cost-disciplined and highly informative
- the team can clearly explain which spend produced value and which did not

### Suggested evidence
- final spend summary
- top cost-driver table
- list of actions taken to control spend

---

## 6. Scorecard worksheet

Use this simple worksheet at the end of the run.

| Category | Weight | Score (0-5) | Weighted score |
|---|---:|---:|---:|
| Runtime reliability and ingest fidelity | 0.20 |  |  |
| Evidence and document quality | 0.15 |  |  |
| Drift detection usefulness | 0.15 |  |  |
| Remediation workflow behavior | 0.15 |  |  |
| Operator usefulness and trust | 0.20 |  |  |
| Cost discipline and efficiency | 0.15 |  |  |
| **Total** | **1.00** |  |  |

### Calculation
For each row:
- **weighted score = score × weight**
- sum the weighted scores for the final result

Example:
- score of 3 in a 20% category = **0.60** weighted points
- score of 4 in a 15% category = **0.60** weighted points

---

## 7. Minimum go / no-go gates

Regardless of the weighted average, the MVP should **not** be called successful if any of these fail badly:
- source traceability cannot be demonstrated
- evidence and narrative are mixed in a way reviewers cannot trust
- no credible remediation-to-validation path is shown
- operators do not find the outputs useful enough to revisit
- cost materially exceeds the agreed ceiling without a compelling reason

These are hard credibility gates.

---

## 8. Recommended evaluation ceremony

At the end of the 7-day run, review the MVP in this order:
1. final spend and cost-driver recap
2. runtime/ingest stability recap
3. walkthrough of one traceable finding
4. walkthrough of one drift case
5. walkthrough of one remediation path with validation evidence
6. review of readiness outputs and other documents produced
7. operator feedback review
8. category scoring and weighted total
9. explicit decision: proceed, narrow and rerun, or redesign

---

## 9. Decision guide

### Proceed
Use when:
- weighted score is roughly **3.0 or higher**
- credibility gates are intact
- operators saw real value
- cost stayed disciplined enough to justify continuation

### Narrow and rerun
Use when:
- the core idea looks sound
- some categories score well
- but cost, noise, or workflow gaps still obscure the real value

### Redesign or stop
Use when:
- trust remains weak
- outputs are not reviewable
- workflow adoption is poor
- or the budget is repeatedly exceeded without convincing operator benefit

---

## 10. Bottom line

A successful lean MVP is not one that simply ran for seven days.
It is one that can credibly show:

- the runtime worked
- the outputs were reviewable
- drift was surfaced usefully
- remediation behavior was disciplined
- operators found it helpful
- and the whole exercise stayed inside a believable budget envelope

If those things are true, the MVP earned the right to continue.
