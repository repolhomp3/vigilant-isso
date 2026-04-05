# Audit Readiness and Reporting Workflow

## Goal

Define how Vigilant ISSO should help teams stay assessment-ready for CMMC Level 2 / NIST SP 800-171-oriented reviews across AWS GovCloud and on-prem environments.

The product should make audits less painful by making daily operations cleaner.

---

## Core position

Audit readiness is not a once-a-year export.

It is the ongoing ability to answer, for a given assessment boundary and time period:
- what controls/practices are in scope
- how they are implemented
- what evidence supports that claim
- what findings remain open
- what remediation is underway
- what exceptions or accepted risks exist
- what changed since the last review

### ISSO/CISO accountability framework

This readiness model reflects explicit accountability requirements:

| Role | Accountability |
|------|---------------|
| **ISSO** | Maintains daily operational awareness; validates evidence quality; routes findings to owners; prepares assessment packages; attests to accuracy of control implementation status |
| **CISO** | Accountable for organizational security posture; approves material exceptions and risk acceptances; ensures adequate resourcing for remediation; reports to executive leadership on compliance status |
| **Control Owner** | Responsible for implementing and maintaining assigned controls; provides evidence on demand; remediates findings within SLA; attests to control operation |
| **Assessor** | Independent evaluation of control implementation against requirements; provides findings and recommendations |

*NIST SP 800-37 RMF roles: ISSO, Authorizing Official, Common Control Provider, System Owner*
*AWS Well-Architected: define responsibilities and ownership for security across the organization*

---

## Verified facts

1. NIST SP 800-171 applies to nonfederal systems and organizations that process, store, or transmit CUI, and NIST provides a CUI SSP template alongside the publication. [1]
2. DoD CIO publishes official CMMC Program documentation, including the Level 2 Assessment Guide and Level 2 Scoping Guidance. [2]
3. AWS CloudTrail supports audit, governance, and compliance by recording user, role, and service actions, and can retain/query long-lived data via CloudTrail Lake. [3]
4. AWS Config records resource configuration state/history over time, which is valuable for showing temporal posture and drift. [4]
5. Security Hub CSPM centralizes control/security findings and can drive automation and reporting workflows. [5]

---

## Audit-readiness workflow

### 1. Define the assessment boundary
Maintain explicit records for:
- assessed system/enclave
- in-scope assets and users
- supporting systems/services
- inherited/shared services
- cloud accounts/regions/sites included
- assessment period start/end

### 2. Define required evidence sets by control
For each control/practice, define:
- required evidence categories
- optional supporting evidence
- freshness expectation
- responsible owner
- preferred source systems

### 3. Continuously evaluate evidence completeness
For each control, calculate:
- evidence present/not present
- evidence freshness
- evidence strength
- unresolved contradictions
- open findings linked to the control
- open remediation linked to the control

### 4. Produce periodic readiness snapshots
Suggested cadence:
- weekly operational snapshot
- monthly compliance readiness snapshot
- pre-assessment freeze snapshot

Snapshots should be immutable and reproducible.

### 5. Prepare assessor support packets
For each control/domain/family/system, generate a packet containing:
- control text and implementation summary
- linked evidence inventory
- current and historical findings
- remediation/POA&M status
- exceptions/compensating controls
- open questions / unresolved gaps
- citations to exact source objects

---

## Required report types

### ISSO operations views
- stale evidence dashboard
- controls with weak evidence only
- open high-risk findings by system
- remediation aging by owner
- contradictions between SSP claims and observed state

### Engineering / operations views
- actionable findings by team
- overdue fixes
- recurring baseline drift
- validation-needed items

### Executive views
- trend of open high-risk items
- audit readiness scorecard by system
- aging and exception trends
- top control families with unresolved gaps

### Assessor-facing views
- point-in-time evidence packet
- control implementation narrative with citations
- open issues register
- closure validation record
- scope and inheritance summary

---

## Readiness scoring

Avoid a fake single "compliant/not compliant" number.

Better model:
- **evidence completeness score**
- **evidence freshness score**
- **open finding pressure**
- **remediation aging pressure**
- **exception burden**
- **narrative confidence**

This creates a much more honest readiness picture.

### Scoring alignment with NIST and AWS guidance

| Dimension | NIST Reference | AWS Well-Architected Alignment |
|-----------|---------------|-------------------------------|
| Evidence completeness | SP 800-171A assessment objectives | Security — ensure traceability |
| Evidence freshness | SP 800-53 CA-7 continuous monitoring | Operational Excellence — evolve processes |
| Open finding pressure | SP 800-53 CA-5 POA&M | Security — automate detection and response |
| Remediation aging | SP 800-53 SI-2 flaw remediation | Operational Excellence — respond to events |
| Exception burden | SP 800-53 CA-6 authorization | Security — implement a strong risk management program |
| Narrative confidence | SP 800-171A assessment methodology | Operational Excellence — understand operational health |

*CISO discipline: readiness is a leading indicator, not a lagging report; these dimensions surface problems before assessors do*

---

## Contradiction detection

One of the most valuable features:

Flag when any of the following disagree:
- SSP statement
- policy statement
- technical telemetry
- control narrative
- remediation status
- exception register

Examples:
- policy says centralized logging retained 1 year; CloudTrail/Config retention settings do not match
- narrative says all critical findings remediated in SLA; open items are overdue
- access review evidence says quarterly reviews complete; linked repositories have stale timestamps

---

## Recommended packet structure

### Packet header
- packet id
- system/scope
- generated at
- generated by
- assessment period
- catalog version

### Section 1: scope
- systems, accounts, enclaves, sites, inherited services

### Section 2: control summary
- requirement text
- implementation narrative
- reviewer notes

### Section 3: evidence index
- table of evidence ids, type, source, observed date, freshness, strength

### Section 4: findings and remediation
- active findings
- linked remediation items
- exceptions / accepted risk

### Section 5: open questions
- missing artifacts
- stale evidence
- contradictory data

### Section 6: attestations and approvals
- reviewer sign-offs
- packet hash / immutable reference

---

## Human-in-the-loop policy

The platform may:
- prepare packets
- draft narratives
- identify missing evidence
- propose questions the assessor is likely to ask

The platform should not:
- claim final certification outcome
- silently exclude inconvenient evidence
- rewrite history after packet generation

---

## Assumptions

1. Customers care more about reducing scramble and surprise than about glossy dashboards.
2. The strongest product behavior is to surface uncertainty explicitly.
3. Control narratives will often need both machine evidence and curated procedural evidence.

---

## Sources

1. NIST, “SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations.” https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
2. DoD CIO, “CMMC Resources & Documentation.” https://dodcio.defense.gov/CMMC/Resources-Documentation/
3. AWS, “What Is AWS CloudTrail?” https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
4. AWS, “What Is AWS Config?” https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
5. AWS, “Introduction to AWS Security Hub CSPM.” https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
