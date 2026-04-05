# Vigilant ISSO Seven-Day Budget Profile
## AWS GovCloud (US-West) / us-gov-west-1

## Purpose

Define a **practical 7-day deployment profile** for the lean Vigilant ISSO MVP that:
- stays under a disciplined budget target
- still gathers useful runtime and operator data
- produces enough evidence to judge whether the MVP is worth continuing

This document is more specific than the earlier one-week cost model.
It describes the **recommended operating posture for the evaluation week**, not just a ROM range.

---

## 1. Recommended budget target

### Primary target
**Target the 7-day evaluation run at $150-$250 total AWS spend.**

That range is low enough to keep the MVP honest and high enough to support:
- one real EKS-hosted control runtime
- modest workflow/evidence persistence
- useful logging and metrics
- a small amount of bounded Bedrock-assisted summarization if enabled

### Hard stop threshold
**Treat $350 for the 7-day run as the no-surprises ceiling.**

If projected spend trends above that line during the week, the team should:
- reduce log volume
- reduce node headroom
- disable optional AI features
- shorten retention and export artifacts needed for analysis
- avoid adding more services or connectors mid-run

### Why this target is the right one
The repo already frames a useful one-week MVP as roughly **$150-$350**.
For the evaluation layer, the better discipline is:
- **plan to land in the lower half of that range**
- keep enough observability to learn something real
- avoid “production comfort” decisions that hide the true lean operating shape

---

## 2. What the 7-day run is trying to prove

This is not a scale test.
It is a **controlled learning week** intended to answer:

1. Can the lean GovCloud runtime stay operational without babysitting?
2. Do the selected AWS-native integrations provide enough useful findings and evidence motion?
3. Can the workflow/evidence path generate reviewable artifacts?
4. Do operators find the output meaningfully better than spreadsheet archaeology?
5. Can all of that happen without cost drift?

If the run does not answer those questions, extra spend is waste.

---

## 3. Recommended 7-day deployment profile

## 3.1 Runtime profile
Run the MVP as a **single small evaluation environment** with:
- **1 EKS cluster** for the Vigilant ISSO runtime
- **1 small node group** sized for the bounded service set
- **only the minimum services required** for intake, workflow state, evidence registration, notification, and evaluation reporting
- **no extra environments** for staging/perf during the 7-day measurement window unless they already exist outside this budget

### Services that should be on
- ingestion for the selected AWS-native sources
- canonical finding/evidence/workflow services
- object-backed evidence storage
- basic notification path
- basic health and cost telemetry
- scorecard data capture

### Services that should stay off unless clearly required
- heavy search/indexing tiers
- broad replay pipelines
- always-on analytics jobs
- large dashboard stacks beyond what is needed for operators
- experimental connectors unrelated to the bounded environment

---

## 3.2 Data and logging posture
Use a **learn enough, not log everything** stance.

### Keep
- service health metrics
- ingest counts and failure counts
- workflow state transitions
- evidence registration events
- remediation actions and closure checks
- enough application logs to debug normal failures

### Limit aggressively
- verbose debug logging by default
- full-payload duplication across multiple stores
- high-cardinality custom metrics unless they answer a known evaluation question
- long retention on operational logs for a one-week test

### Recommended retention stance for the run
- keep detailed operational logs only as long as needed for the 7-day evaluation plus short post-run review
- export or summarize the metrics needed for the scorecard before teardown or retention reduction
- retain primary evidence and final evaluation outputs longer than raw debug exhaust

---

## 3.3 Bedrock / AI posture
Bedrock should be treated as **optional, bounded, and measurable**.

### Good uses during the 7-day run
- draft readiness-summary text
- draft evidence-gap summaries
- draft operator-oriented triage summaries

### Bad uses during the 7-day run
- broad always-on summarization for every event
- repeated regenerate-until-it-looks-good prompting
- AI in the hot path for ingestion or workflow correctness
- autonomous closure or remediation decisions

### Budget posture
- set a small explicit weekly AI budget
- disable or throttle Bedrock if the team is already getting sufficient value from non-AI outputs
- log prompt/use counts at a coarse level so usefulness can be weighed against spend

**Recommended stance:** keep Bedrock spend in the **single-digit to low-double-digit dollar range** for the week unless there is a clear evaluation reason to spend more.

---

## 4. Cost envelope by component

This profile assumes a disciplined run aimed at **$150-$250 total**.

| Cost element | 7-day target range | Budget posture |
|---|---:|---|
| **EKS control plane** | $50-$80 | Baseline fixed cost; accept it and control everything around it |
| **Worker compute / nodes** | $25-$60 | Keep node count and sizing lean; avoid comfort headroom |
| **State backend / persistence** | $15-$40 | Small but durable; enough for workflow truth |
| **S3 / evidence storage** | $1-$8 | Minor at MVP scale |
| **CloudWatch logs and basic metrics** | $10-$35 | Main controllable wildcard; tune early |
| **Queue / events / glue services** | $2-$10 | Usually modest |
| **Secrets / KMS / misc** | $2-$10 | Small but real |
| **Bedrock usage** | $0-$20 | Optional; keep tightly bounded |
| **Buffer / variance** | $10-$25 | Covers small surprises |
| **Expected total** | **$115-$288** | Aim to finish near the middle, not at the edge |

### Interpretation
- finishing around **$150-$250** means the profile stayed disciplined
- finishing around **$250-$350** means the run was still viable but scope control weakened
- finishing above **$350** means the MVP likely drifted toward comfort or noise rather than evaluation discipline

---

## 5. Day-by-day operating pattern

## Day 0 / preflight
Goal: prevent waste before the clock starts.

Do before or at launch:
- verify only required services are enabled
- set cost tags on all MVP resources
- enable budget alarms and daily spend review
- confirm log levels and retention settings
- confirm Bedrock feature flags default to off or bounded
- define the success questions for the week

## Day 1-2 / stability and ingest proof
Goal: prove the spine works.

Focus on:
- service stability
- connector health
- finding and evidence flow
- correcting obvious log-volume mistakes immediately

Do not add more scope unless a blocker requires it.

## Day 3-4 / workflow and remediation proof
Goal: prove the platform supports operational work, not just data collection.

Focus on:
- triage flow
- evidence freshness handling
- remediation record creation and aging
- closure validation behavior
- operator review experience

## Day 5-6 / readiness-output proof
Goal: produce and review useful outputs.

Focus on:
- readiness snapshot generation
- summary quality
- source traceability
- operator usefulness feedback
- drift and remediation evidence in context

## Day 7 / scorecard closeout
Goal: turn the week into a go/no-go input.

Produce:
- final cost snapshot
- final metric rollup
- scorecard judgment
- top lessons learned
- recommended keep/change/kill decisions for Sprint 2

---

## 6. Minimum useful data to collect during the week

The week should capture enough data to score the MVP across cost, usefulness, and trust.

### Runtime data
- service uptime / major outage count
- ingest success/failure counts by source
- backlog/queue depth if used
- average or median time from finding intake to triage-ready state
- major operational incidents and recovery time

### Workflow data
- number of findings ingested
- number of findings triaged
- number of remediation items created
- number of remediation items validated/closed
- number of evidence items registered
- number of stale-evidence cases detected

### Output data
- number of readiness snapshots produced
- number of reviewable summaries produced
- number of source-backed citations or linked records in outputs
- operator feedback notes on usefulness/trust

### Cost data
- daily spend trend
- top 3 cost drivers for the run
- any cost spikes and their cause

---

## 7. Exit criteria for a successful 7-day profile

The budget profile should be judged successful if most of the following are true:
- total spend stays within the **$150-$250 target** or at least under the **$350 ceiling**
- the platform remains stable enough to gather a full week of evaluation data
- ingest, workflow, and output paths all produce scorecard-worthy evidence
- operators can complete at least one credible finding-to-remediation-to-validation story
- readiness outputs are grounded enough to review, not just admire
- the week produces clear decisions about what to keep, simplify, or cut next

---

## 8. Failure signs

The 7-day deployment profile should be treated as a warning sign if:
- cost rises quickly without additional evaluation value
- logs dominate the bill because they were left untuned
- optional AI spend becomes easier to measure than actual operator value
- the team adds services during the week instead of learning from the bounded setup
- the environment becomes “production-shaped” before the MVP proves usefulness
- no one can explain what was learned for each major cost line item

---

## 9. Recommended decision rule

At the end of the week:

- **Proceed** if the MVP stayed near budget, produced trustworthy artifacts, and reduced operator friction in at least a few real tasks.
- **Proceed with narrowing** if the core value is visible but logging, compute sizing, or optional features caused cost drift.
- **Pause or redesign** if the team spent above the ceiling and still cannot demonstrate trustworthy workflow and useful outputs.

---

## Bottom line

The best 7-day GovCloud MVP profile is not the cheapest possible run.
It is the **cheapest run that still produces enough trustworthy runtime, workflow, and operator data to make a real continuation decision**.

For this repo's current lean direction, that means:
- **targeting $150-$250 for the week**
- **treating $350 as the ceiling**
- **optimizing for learning density, not platform comfort**
