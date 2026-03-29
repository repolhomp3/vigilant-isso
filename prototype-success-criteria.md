# Prototype Success Criteria

## Purpose

Define how the first bounded Vigilant ISSO prototype will be judged.

This is intentionally narrower than “is Vigilant ISSO a good idea?”

The question here is:

**Did the prototype prove that Vigilant ISSO can operate credibly and usefully in one bounded AWS environment?**

---

## Prototype boundary being evaluated

The success criteria in this document apply only to the first bounded environment:
- **1 AWS account**
- **2 VPCs**
- **1 data lake**
- **3 small EKS clusters**

Nothing in this document should be read as proof of readiness for broad rollout beyond that scope.

---

## Verified facts from this repo
- Existing project docs define Vigilant ISSO as an **evidence-grounded ISSO operations platform**, not an autonomous compliance authority.
- Existing docs emphasize:
  - provenance
  - durable workflow state
  - evidence freshness
  - remediation discipline
  - packet/readiness support
  - human approval for consequential decisions
- Existing roadmap docs state that the MVP must prove telemetry ingestion, evidence discipline, remediation workflow, and reviewable readiness outputs before wider rollout.

## Working assumptions
- A prototype is successful if it creates real operational trust for a narrow environment, even if many later-phase features are still absent.
- The most important success indicators are usefulness and credibility, not feature count.
- Qualitative operator trust matters alongside technical metrics.

---

## Primary success criteria

## 1. Source fidelity success

The prototype succeeds on source fidelity if:
- selected AWS-native signals for the bounded environment are ingested successfully on a repeatable basis
- normalized findings preserve source identifiers and provenance
- operators can trace a canonical finding back to its source record/reference
- obvious source nuance is not erased during normalization

### Evidence of success
- demo or test showing at least one finding traced from source to canonical record
- connector/error logs showing reliable repeatability
- reviewer confirmation that normalized outputs remain understandable and defensible

### Failure signs
- findings cannot be traced back cleanly
- normalization invents false certainty
- source-specific meaning is lost or mangled

---

## 2. Environment-scoping success

The prototype succeeds on environment scoping if:
- findings and evidence can be associated to the correct prototype boundary objects often enough to be operationally useful
- the platform can distinguish relevant context across the 2 VPCs and 3 clusters
- shared-service context for the data lake is represented cleanly enough to support triage and evidence discussion

### Evidence of success
- prototype records visibly linked to account/VPC/cluster/system entities
- operators can filter or review by cluster, VPC, or shared boundary
- unresolved scoping cases are explicitly marked instead of silently guessed

### Failure signs
- the prototype cannot reliably tell where a finding belongs
- shared-service assets cause persistent ambiguity with no visible confidence markers

---

## 3. Workflow credibility success

The prototype succeeds on workflow credibility if:
- a new finding can enter triage workflow cleanly
- remediation items can be opened and tracked with owners/status
- closure requires linked validation evidence or explicit human review
- material state changes are attributable

### Evidence of success
- live walkthrough of finding → workflow item → remediation → validation state
- audit trail entries for key mutations and approvals
- reviewers confirm the workflow is understandable and not theater

### Failure signs
- state changes happen without attribution
- remediation closure is just a status flip with no evidence discipline
- operators bypass the workflow because it is less useful than existing manual methods

---

## 4. Evidence-discipline success

The prototype succeeds on evidence discipline if:
- evidence metadata is stored separately from derivative narrative output
- freshness state exists and can identify stale or weak evidence
- the system can link evidence to findings, remediation, and snapshot outputs
- the prototype does not confuse generated summaries with primary evidence

### Evidence of success
- evidence records show provenance and timestamps
- stale evidence scenarios can be demonstrated
- packet/snapshot content references evidence objects rather than vague claims

### Failure signs
- summaries are treated as evidence
- no freshness model exists in practice
- packet output cannot explain what evidence supports it

---

## 5. Readiness-output success

The prototype succeeds on readiness-output usefulness if:
- it can produce a bounded readiness snapshot for the first environment
- the snapshot clearly shows open issues, stale evidence, and linked remediation
- summaries are reviewable and grounded in source-backed state
- the output is useful enough that an ISSO would prefer it to ad hoc spreadsheet archaeology

### Evidence of success
- one or more prototype snapshots/packets generated and reviewed
- ISSO/reviewer feedback that the output improves readiness understanding
- snapshot contents are reproducible from known state

### Failure signs
- output is too vague to support action or review
- snapshot generation depends on manual heroics
- reviewers do not trust or use the result

---

## 6. Operational simplicity success

The prototype succeeds on operational simplicity if:
- the three-cluster topology remains understandable to the team
- Terraform and Helm structure is coherent enough that another engineer can work in it
- the system can be deployed, updated, and debugged without extraordinary tribal knowledge

### Evidence of success
- documented module/chart layout exists
- basic deployment and rollback/update path is understood
- logs/health checks/correlation IDs make troubleshooting possible

### Failure signs
- the prototype architecture is already too complex for the team operating it
- deployment requires manual incantations known by one person only
- cluster/service sprawl outpaces actual value delivered

---

## 7. Human-trust success

The prototype succeeds on human trust if:
- ISSO/reviewer users understand what the system knows vs infers
- users trust the citations and workflow history enough to act on them
- the platform is seen as reducing scramble work rather than creating another dashboard nobody believes

### Evidence of success
- qualitative feedback from ISSO/reviewer users
- repeated use during normal readiness work
- willingness to review prototype snapshots/queues as part of actual operations

### Failure signs
- users treat the platform as a demo artifact only
- users doubt the provenance or can’t interpret the outputs
- the platform adds work without reducing uncertainty

---

## Minimum quantitative gates

These are pragmatic prototype-level targets, not enterprise SLAs.

### Suggested gates
- **Ingestion reliability:** core selected source path succeeds consistently enough to support repeated demos and pilot usage
- **Traceability:** reviewers can trace sample findings back to source references in every prototype review session
- **Workflow completeness:** at least one full path from finding to remediation/validation can be demonstrated end to end
- **Snapshot usefulness:** at least one readiness snapshot/packet is generated and judged reviewable by intended users
- **Time-to-understand improvement:** ISSO/reviewer feedback indicates the prototype is faster than current manual state reconstruction for at least some questions

### Working assumption
Exact percentages should be set by the implementation team once real connector behavior is visible; this repo already avoids fake precision, and that restraint should continue here.

---

## Explicit non-successes

The prototype should **not** be called successful merely because:
- EKS clusters exist
- services are deployed
- dashboards look polished
- AI can generate impressive prose
- many connectors are partially wired
- a large backlog was written

Those things may be useful, but they are not proof that the prototype works.

---

## Go / no-go expansion criteria

The prototype is ready for Sprint 2+ expansion if most of the following are true:
- source fidelity is trusted
- the environment model is usable
- the core workflow is actually followed
- evidence freshness/provenance is visible
- readiness output is useful to at least one real ISSO/reviewer workflow
- architecture and deployment complexity are still under control

The prototype is **not** ready to expand if:
- provenance remains shaky
- the core workflow is bypassed
- scoping is persistently confusing
- packet outputs are not trusted
- the team is already drowning in platform complexity

---

## Recommended evaluation ceremony

A good prototype review should include:
1. bounded environment recap
2. demo of one real ingestion path
3. walkthrough of provenance and normalized record shape
4. walkthrough of finding triage and remediation path
5. walkthrough of evidence linkage and freshness state
6. walkthrough of one readiness snapshot or packet
7. operator/reviewer feedback collection
8. explicit decision: continue, narrow further, or redesign

---

## Verified facts vs working assumptions summary

### Verified facts
- The repo already defines the platform around provenance, evidence, workflow, snapshots, and human review.
- The roadmap already says the MVP must prove ingestion, evidence discipline, remediation workflow, and readiness outputs before wider rollout.
- The first prototype is intentionally bounded to one small AWS environment.

### Working assumptions
- Human trust and workflow adoption are first-class success measures.
- Prototype quantitative targets should remain pragmatic and avoid false certainty.
- A prototype can be a success even if AI and advanced analytics remain disabled.

---

## Bottom line

A successful prototype is one where an ISSO can honestly say:

**“For this one bounded environment, I can see what changed, trust where it came from, route the work, and review a readiness snapshot without the usual manual archaeology.”**
