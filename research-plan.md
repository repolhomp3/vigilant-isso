# Vigilant ISSO Research Plan

## Objective

Define a credible, evidence-grounded ISSO operations agent for hybrid environments that span **AWS GovCloud (US)** and **on-prem CMMC Level 2 / NIST SP 800-171** scope.

The near-term goal is not autonomous compliance. It is a disciplined platform concept that helps ISSOs and security teams:
- maintain current posture awareness
- keep evidence organized and fresh
- connect findings to controls and systems
- track remediation with accountability
- prepare audit-ready narratives backed by source artifacts

---

## Verified facts

The following points are directly supported by current official documentation:

1. **AWS GovCloud (US) is an isolated AWS region family for regulated/sensitive workloads.** AWS states it is designed for U.S. government agencies and customers with requirements including FedRAMP High, DoD SRG IL4/5, and CJIS, and that the regions are administered only by AWS personnel who are U.S. citizens. AWS also states customers can run workloads containing all categories of CUI in GovCloud (US). [1]
2. **GovCloud differs from commercial AWS in concrete operational ways.** AWS documents that GovCloud ARNs use the `arn:aws-us-gov` partition and region identifiers such as `us-gov-west-1` and `us-gov-east-1`. [2]
3. **NIST SP 800-171 Rev. 2 remains the normative source for the 110 CUI security requirements.** NIST explicitly says the PDF is the authoritative source and describes the requirements as applying to nonfederal systems that process, store, or transmit CUI. [3]
4. **DoD CMMC documentation currently provides Level 2 scoping and assessment guidance.** The DoD CIO CMMC resources page links the CMMC Program Model Overview, Level 2 Scoping Guidance, and Level 2 Assessment Guide. The page also notes phased implementation began in November 2025, with early focus on Level 1 and Level 2 self-assessments. [4]
5. **AWS-native telemetry sources are already structured enough to seed an ISSO operations fabric.**
   - Security Hub CSPM aggregates findings across AWS services and standards, and supports automation through EventBridge and automation rules. [5]
   - AWS Config tracks resource configuration state and history over time. [6]
   - GuardDuty continuously analyzes AWS data sources and logs for threats. [7]
   - Inspector continuously scans workloads for vulnerabilities and network exposure and creates findings with remediation recommendations. [8]
   - IAM Access Analyzer generates findings for external/internal/unused access and policy issues. [9]
   - CloudTrail records management and API activity for audit, governance, and security analysis. [10]
   - Systems Manager Compliance can aggregate patch/configuration compliance across accounts and regions and supports remediation workflows. [11]
6. **NIST OSCAL provides machine-readable formats for control catalogs, baselines, implementations, and assessment content.** It is a credible anchor for a control/evidence interchange strategy even if Vigilant ISSO stores its own internal schema. [12]

---

## Highest-value ISSO tasks to tackle first

### Tier 1: immediate value, low controversy

1. **Finding normalization and de-duplication**
   - Merge repeated findings across Security Hub, GuardDuty, Inspector, Config, and on-prem scanners.
   - Suppress noise without deleting provenance.
   - Group by asset, control, severity, owner, and recurring cause.

2. **Control-aware evidence inventory**
   - For each control/practice, show what evidence exists, where it came from, when it was collected, and when it goes stale.
   - Separate “evidence exists” from “evidence is persuasive.”

3. **POA&M / remediation tracking with compliance context**
   - Convert accepted findings into remediation work items.
   - Track status, aging, exceptions, compensating measures, due dates, and re-test evidence.

4. **Audit-ready posture snapshots**
   - Generate review packets by control family, enclave, system, and assessment period.
   - Include source citations, open issues, stale evidence, and unresolved assessor questions.

### Tier 2: strong value, requires disciplined human review

5. **Control mapping recommendations**
   - Suggest likely 800-171 / CMMC mappings for findings and evidence.
   - Require human approval for final mappings where one finding touches multiple requirements.

6. **Narrative generation for ISSO workflows**
   - Draft control summaries, risk statements, and status updates grounded in evidence and findings.
   - Must label verified facts vs inference vs recommendation.

7. **Staleness / contradiction detection**
   - Flag when policy claims, SSP statements, diagrams, and operational evidence diverge.
   - Example: SSP says MFA enforced for admins; IAM/IdP exports show bypass groups or stale exceptions.

### Tier 3: valuable but riskier to automate

8. **Remediation prioritization using mission and audit impact**
9. **Suggested compensating controls or interim risk reduction plans**
10. **Assessor-question forecasting based on weak evidence patterns**

---

## Research questions

1. What source systems are mandatory for a credible first release in GovCloud and in on-prem CMMC L2 environments?
2. Which findings should remain source-native versus normalized into a common internal schema?
3. What is the minimum viable evidence ontology that still preserves provenance, freshness, and chain of custody?
4. How should one finding map to many controls without double-counting risk or work?
5. Which outputs belong to operators, ISSOs, executives, and assessors respectively?
6. What decisions can be automated, what should be approval-gated, and what must remain advisory only?
7. How should the platform represent scoping boundaries and inherited/shared controls in hybrid environments?
8. How should waivers, accepted risk, and compensating measures be modeled so they are visible but not mistaken for closure?

---

## Working assumptions

These are design assumptions, not verified facts:

1. Early buyers/users will care more about **evidence discipline and audit readiness** than fully autonomous remediation.
2. Most viable deployments will start as an **internal platform or trusted co-pilot** for ISSOs rather than a broad end-user product.
3. The most credible first architecture is **deterministic workflows + retrieval-grounded AI summaries**, not an open-ended agent with broad actuation rights.
4. Control mapping should be **many-to-many and versioned**, because one technical issue often affects several requirements and control texts evolve.
5. Hybrid customers will need **human-curated evidence sources** for policy, procedures, access reviews, diagrams, and training artifacts in addition to machine telemetry.

---

## Initial source categories

### AWS GovCloud
- Security Hub CSPM
- AWS Config
- GuardDuty
- Inspector
- IAM Access Analyzer
- CloudTrail / CloudTrail Lake
- Systems Manager Compliance / Patch Manager / State Manager
- Identity data from IAM Identity Center or other IdP exports where available

### On-prem / enterprise systems
- vulnerability scanners
- configuration compliance scanners
- endpoint posture tools
- directory/identity systems
- ticketing / ITSM / defect trackers
- CMDB / asset inventory
- evidence repositories and document stores
- SSP, procedures, diagrams, access review records, training records

---

## Research output plan

This project should answer the following with concrete documents:
- `architecture-options.md` — credible platform designs and tradeoffs
- `control-mapping.md` — control mapping method and confidence model
- `evidence-model.md` — evidence, provenance, and graph/data model
- `poam-workflow.md` — remediation lifecycle and governance model
- `audit-readiness.md` — reporting and assessment support workflows
- `pitch.md` — why this is serious and useful

---

## Sources

1. AWS, “What Is AWS GovCloud (US)?” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/whatis.html
2. AWS, “Amazon Resource Names (ARNs) in GovCloud (US) Regions.” https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/using-govcloud-arns.html
3. NIST, “SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations.” https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
4. DoD CIO, “CMMC Resources & Documentation.” https://dodcio.defense.gov/CMMC/Resources-Documentation/
5. AWS, “Introduction to AWS Security Hub CSPM.” https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
6. AWS, “What Is AWS Config?” https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
7. AWS, “What is Amazon GuardDuty?” https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html
8. AWS, “What is Amazon Inspector?” https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html
9. AWS, “Using IAM Access Analyzer.” https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
10. AWS, “What Is AWS CloudTrail?” https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
11. AWS, “AWS Systems Manager Compliance.” https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-compliance.html
12. NIST, “Open Security Controls Assessment Language (OSCAL).” https://pages.nist.gov/OSCAL/
