# Control Mapping Strategy

## Goal

Map technical findings, evidence, and remediation work to **CMMC Level 2** and **NIST SP 800-171 Rev. 2** in a way that is reviewable, versioned, and grounded in source evidence.

The system should help with mapping. It should not pretend mapping is trivial or always one-to-one.

---

## Verified facts

1. NIST SP 800-171 Rev. 2 is the authoritative source for the security requirements, and NIST states the PDF is the normative reference. [1]
2. SP 800-171 Rev. 2 organizes 110 security requirements across 14 requirement families. [1]
3. The DoD CIO CMMC resources page provides the official Level 2 scoping guidance and Level 2 assessment guide. [2]
4. AWS Security Hub CSPM findings already include a `Compliance` structure and related requirements fields within the AWS Security Finding Format (ASFF), which is useful as an input signal but should not be treated as the final compliance truth for CMMC/800-171. [3]

---

## Mapping design principles

These principles reflect NIST SP 800-171A assessment methodology, CMMC assessment guidance, and practical ISSO/CISO discipline for defensible control mapping.

1. **Normative text beats heuristic tags.**
   - Source-native compliance tags are useful, but final mapping must be anchored to the relevant requirement text and assessment logic.
   - *NIST SP 800-171A: assessment objectives provide the authoritative basis for determining requirement satisfaction*

2. **Mappings are many-to-many.**
   - One technical issue may affect several requirements.
   - One requirement may need many technical and documentary evidence objects.
   - *ISSO practice: findings rarely map 1:1 to controls; accurate mapping requires understanding both the finding and the control implementation context*

3. **Mappings must be versioned.**
   - Catalog version, internal mapping rule version, reviewer, and review date all matter.
   - *NIST SP 800-53 CM-3: configuration change control; mappings are configuration items with change history requirements*

4. **Coverage is not compliance.**
   - A mapped finding tells you a requirement is implicated, not that the entire requirement passes or fails.
   - *CISO responsibility: avoid conflating "we looked at this control" with "this control is satisfied"*

5. **Scoping matters before mapping.**
   - If an asset/system/user population is out of scope, the finding may still matter operationally but not belong in a specific CMMC assessment boundary.
   - *CMMC Assessment Guide: scoping determines which assets and practices are in the assessment boundary; over-scoping or under-scoping introduces material risk*

6. **Traceability is mandatory.**
   - Every mapping assertion must link to source evidence, rationale, and reviewer accountability.
   - *NIST SP 800-53 AU-10, SI-12: non-repudiation and information management; accountability for compliance claims*

7. **Confidence levels are explicit.**
   - Distinguish high-confidence automated mappings from analyst-inferred mappings requiring review.
   - *AWS Well-Architected: Operational Excellence — define mechanisms to identify operational events and assess their impact*

---

## Canonical mapping objects

### 1. Control catalog entry
Represents a single requirement/practice, for example:
- framework: `NIST_SP_800_171`
- framework_version: `Rev2`
- control_id: `3.5.3`
- family/domain: `Identification and Authentication`
- statement_text
- assessment_objectives_reference
- cmmc_equivalent_id (if maintained)

### 2. Mapping assertion
Represents the claim that an object is relevant to a control.

Fields:
- mapping_id
- source_object_type: finding | evidence | procedure | attestation | exception | remediation_item
- source_object_id
- target_control_id
- relationship_type:
  - indicates_nonconformance
  - supports_implementation
  - supports_assessment
  - supports_compensating_measure
  - mitigates_risk
  - narrows_scope
- rationale_text
- mapping_method: rule | analyst | LLM_draft | imported
- confidence_score
- confidence_reason
- reviewer
- review_status: proposed | accepted | rejected | superseded
- valid_from / valid_to
- source_citations

### 3. Mapping bundle
Groups multiple assertions created during one review cycle, assessment period, or ingestion run.

---

## Recommended mapping workflow

### Step 1: establish scope context
Before any mapping:
- identify system/enclave
- identify whether the asset/data touches CUI processing, storage, or transmission
- identify whether the component is in-scope, supporting, or out-of-scope for the assessment boundary

### Step 2: normalize the technical observation
Example normalized observation:
- source: Inspector finding
- issue: internet-reachable EC2 instance with critical CVE
- resource: `arn:aws-us-gov:ec2:...`
- first_seen / last_seen
- severity
- remediation guidance

### Step 3: apply mapping rules
Use three layers:

1. **Deterministic rule mappings**
   - Example: IAM Access Analyzer external sharing finding strongly suggests relevance to AC and SC family requirements.
2. **Source-native hints**
   - Example: Security Hub `Compliance.RelatedRequirements` and ASFF metadata. [3]
3. **Retrieval-grounded analyst/LLM draft**
   - Use requirement text, prior accepted mappings, and local policy context.

### Step 4: human review where ambiguity is material
Required when:
- one finding maps to more than two substantive requirements
- documentary evidence is thin
- compensating control language is invoked
- the mapping could change reported posture materially

### Step 5: persist accepted mappings with rationale
Do not keep only the end result. Store why it was mapped.

---

## Mapping confidence model

### High confidence
- direct rule + direct evidence + low ambiguity
- example: CloudTrail disabled in a governed account, directly affecting logging/audit expectations

### Medium confidence
- plausible technical-control relationship but additional context needed
- example: patch lag implicates vulnerability management and possibly configuration/maintenance requirements

### Low confidence
- narrative or policy-heavy requirement with only indirect telemetry
- example: training, personnel screening, or some governance/process requirements

Low-confidence mappings can still be useful, but they should be labeled as analyst prompts, not final posture statements.

---

## Evidence classes by control type

### Strongly machine-observable
Good candidates for automation-assisted mapping:
- logging and event collection
- patch/compliance status
- vulnerability exposure
- external/public access exposure
- configuration drift
- encryption settings
- MFA/identity posture

### Partly machine-observable
Need mixed evidence:
- account management lifecycle
- incident response execution
- media protection handling
- maintenance controls
- change control

### Mostly documentary / procedural
Need human-curated evidence:
- security awareness training records
- personnel actions
- policy approval records
- SSP narratives
- management review artifacts

---

## Example mappings

### Example 1: public S3 bucket / cross-account exposure
- source: IAM Access Analyzer or Security Hub finding
- likely related controls:
  - AC family access enforcement / least privilege related requirements
  - SC family protections related to information flow or boundary handling
- confidence: medium to high depending on exact exposure
- required evidence: bucket policy, business justification, exception record, remediation or compensating measure

### Example 2: missing CloudTrail coverage
- source: Config / CloudTrail / Security Hub
- likely related controls:
  - AU family requirements for audit event generation, retention, and review
- confidence: high
- required evidence: trail settings, logging destinations, retention config, review procedures, sample review record

### Example 3: overdue critical patch on in-scope server
- source: Systems Manager Compliance or on-prem scanner
- likely related controls:
  - SI family related to flaw remediation
  - CM/MA related controls depending on local implementation model
- confidence: high for SI, medium for others
- required evidence: patch policy, scan result, approved exception if any, closure verification

---

## Anti-patterns to avoid

1. Treating one failed technical check as automatic failure of an entire requirement.
2. Treating a narrative policy document as sufficient implementation evidence by itself.
3. Trusting source-native compliance tags without review.
4. Losing history when mappings change.
5. Collapsing all related findings into one generic control issue without preserving individual observations.

---

## Recommended deliverables from the mapping layer

- proposed control mappings with confidence
- accepted control mappings with rationale and citations
- control coverage matrix by system/enclave
- “evidence exists but weak” list
- “finding mapped but no remediation owner” list
- requirement heatmap by open nonconformance count and evidence freshness

---

## Assumptions

1. Internal mapping tables will need to be curated over time from accepted analyst decisions.
2. Teams will want a local mapping layer that reflects their own SSP/control implementation, not just generic framework text.
3. CMMC Level 2 mapping will often be maintained as a linked projection of 800-171-based controls plus CMMC-specific assessment context.

---

## Sources

1. NIST, “SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations.” https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
2. DoD CIO, “CMMC Resources & Documentation.” https://dodcio.defense.gov/CMMC/Resources-Documentation/
3. AWS, “AWS Security Finding Format (ASFF).” https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-findings-format.html
4. AWS, “Introduction to AWS Security Hub CSPM.” https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
5. AWS, “Using IAM Access Analyzer.” https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
6. AWS, “What Is AWS CloudTrail?” https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
7. AWS, “AWS Systems Manager Compliance.” https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-compliance.html
