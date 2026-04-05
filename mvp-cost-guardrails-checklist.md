# Vigilant ISSO MVP Cost Guardrails Checklist
## AWS GovCloud (US-West) / us-gov-west-1

## Purpose

Provide a **practical cost-control checklist** for the lean Vigilant ISSO MVP so the evaluation run does not quietly drift from a bounded experiment into a more expensive quasi-production deployment.

This checklist is meant to be used:
- before launch
- during the 7-day run
- during closeout and before any extension

---

## 1. Guardrail policy summary

### Budget intent
- [ ] We have a named target budget for the run, not just a vague estimate.
- [ ] The team agrees the **7-day target is $150-$250** and the **ceiling is $350** unless explicitly re-approved.
- [ ] We can identify who is responsible for watching spend during the run.

### Core principle
- [ ] Every cost line item must justify itself with evaluation value.
- [ ] If a service does not improve trust, workflow proof, or operator usefulness during the MVP, it should stay off.

---

## 2. Scope guardrails

### Boundary control
- [ ] The MVP still monitors only the bounded first environment: **1 account, 2 VPCs, 1 data lake, 3 small EKS clusters**.
- [ ] No new monitored environments were added just because the platform technically could.
- [ ] No extra staging/perf environments were created inside the same evaluation budget without explicit approval.

### Feature control
- [ ] Only services required for ingestion, workflow state, evidence handling, notifications, and evaluation reporting are enabled.
- [ ] Heavy search, broad analytics, and speculative helper services remain off.
- [ ] Optional AI features are feature-flagged and can be disabled quickly.
- [ ] Nice-to-have dashboard or UX work is not being funded with infrastructure sprawl.

### Change control
- [ ] Any mid-run service addition requires a clear reason tied to the MVP scorecard.
- [ ] Any mid-run scope increase is documented with expected cost impact.
- [ ] “While we're here” additions are treated as a warning sign, not free progress.

---

## 3. Compute and platform guardrails

### EKS discipline
- [ ] There is exactly one small Vigilant ISSO runtime cluster for the evaluation unless an exception was approved.
- [ ] Node groups are sized for the bounded service set, not for imagined future load.
- [ ] Requests/limits are set well enough to prevent accidental oversizing.
- [ ] Autoscaling, if used, has conservative bounds.
- [ ] Idle workloads are not burning capacity overnight just because the week is short.

### Workload discipline
- [ ] Nonessential jobs and cron-like tasks are disabled.
- [ ] Resource-hungry background processing is not running continuously without a metric-backed reason.
- [ ] The team has identified the few pods/services most likely to cause compute drift.

### Environment hygiene
- [ ] Temporary test resources are tagged and easy to remove.
- [ ] Orphaned load balancers, volumes, snapshots, and leftover jobs are checked at least once mid-week and once at closeout.
- [ ] Teardown steps are identified before the week ends.

---

## 4. Logging and observability guardrails

### Log-volume control
- [ ] Default log level is info/warn, not debug, unless debugging a live issue.
- [ ] Debug logging has a time-boxed use and gets turned back down after investigation.
- [ ] We are not duplicating the same payload into multiple logs/stores without need.
- [ ] High-chatter services are identified before they surprise the bill.

### Metrics discipline
- [ ] Only metrics tied to runtime health, workflow performance, or scorecard questions are enabled.
- [ ] High-cardinality metrics are avoided unless there is a specific decision they support.
- [ ] Retention and dashboarding are kept minimal for the one-week run.

### Review cadence
- [ ] Spend and log-volume are reviewed daily during the run.
- [ ] If CloudWatch cost becomes a top driver, corrective action happens the same day.

---

## 5. Data and storage guardrails

### Evidence storage discipline
- [ ] Primary evidence and metadata needed for trust are retained.
- [ ] Raw debug exhaust is not retained like high-value evidence.
- [ ] S3/object storage paths are organized well enough to prevent duplicate copies and confusion.

### Persistence discipline
- [ ] The state backend is sized for workflow truth, not comfort-scale future growth.
- [ ] We are not using a larger managed persistence tier just to avoid making a sizing decision.
- [ ] Backups and snapshots are bounded to MVP needs.

### Retention discipline
- [ ] Operational log retention matches the evaluation window and short review period.
- [ ] Longer retention is reserved for final artifacts, evidence, and scorecard outputs.
- [ ] Temporary exports are removed after they have served their analysis purpose.

---

## 6. Bedrock / AI guardrails

### Activation control
- [ ] Bedrock features start disabled or tightly limited.
- [ ] There is an explicit reason for every AI-assisted workflow enabled during the MVP.
- [ ] AI is not used in the correctness path for ingestion, evidence registration, or closure decisions.

### Spend control
- [ ] A small weekly AI budget is set in advance.
- [ ] Prompt/use counts are tracked coarsely enough to relate cost to usefulness.
- [ ] Repeated regenerate-and-compare behavior is discouraged during the evaluation week.
- [ ] If AI output is not materially helping operators, it gets turned off instead of defended.

### Trust control
- [ ] AI outputs are clearly labeled as advisory.
- [ ] Citations or source references are required for any summary that influences an operator.
- [ ] No autonomous remediation or closure logic is hidden behind “assistive” language.

---

## 7. Budget visibility guardrails

### Cost visibility
- [ ] MVP resources carry consistent tags for cost allocation.
- [ ] Daily spend can be reviewed without manual detective work.
- [ ] The team can identify the top cost drivers at any point in the week.

### Alerting
- [ ] Budget alerts or equivalent thresholds are configured before launch.
- [ ] There is a threshold for “watch closely” and another for “take action now.”
- [ ] Alert recipients are known and available during the run.

### Accountability
- [ ] A single owner or clearly defined pair owns daily cost review.
- [ ] Cost decisions during the week are logged with brief rationale.
- [ ] If the team knowingly exceeds a guardrail, that decision is documented rather than forgotten.

---

## 8. Operational guardrails during the week

### Daily questions
Ask these each day:
- [ ] Did today's spend buy new learning, or just more exhaust?
- [ ] Did any service cost more than its evaluation value today?
- [ ] Are we gathering scorecard data, or just generating activity?
- [ ] Can we disable anything now without losing the week's core proof points?

### Mid-week correction triggers
Take corrective action if:
- [ ] log cost is rising faster than expected
- [ ] node utilization shows clear overprovisioning
- [ ] Bedrock usage is measurable but operator value is weak
- [ ] new services were added without improving scorecard coverage
- [ ] the team starts talking about “keeping this around” before the MVP is judged

### End-of-run discipline
- [ ] Final artifacts are exported or preserved.
- [ ] Nonessential resources are shut down promptly.
- [ ] The cost review is completed before extending the run.
- [ ] Extension beyond seven days requires a fresh decision, not inertia.

---

## 9. Anti-drift heuristics

Treat these as immediate drift signals:
- [ ] We are adding services because they might help later.
- [ ] We are raising retention “just to be safe.”
- [ ] We are using debug settings as the new normal.
- [ ] We are sizing for production instead of the bounded MVP.
- [ ] We are spending more time defending cost than explaining value.
- [ ] We are calling the environment “lean” while ignoring obvious idle spend.
- [ ] We cannot tell which costs support trust, workflow, or operator usefulness.

If two or more of those are true at once, the MVP is drifting.

---

## 10. Closeout checklist

At the end of the run:
- [ ] Total spend is captured.
- [ ] Top cost drivers are documented.
- [ ] Any surprise charges are explained.
- [ ] The team knows what to shrink, keep, or disable next time.
- [ ] The scorecard includes cost-effectiveness, not just technical function.
- [ ] Resources not needed for follow-on work are shut down or removed.

---

## Bottom line

A lean MVP does not stay lean by accident.
It stays lean because the team repeatedly asks:

**“Is this spend helping us prove trust, workflow, evidence discipline, or operator usefulness this week?”**

If the answer is no, cut it.
