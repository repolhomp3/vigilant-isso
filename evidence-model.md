# Evidence Model

## Goal

Define how Vigilant ISSO should represent evidence, provenance, findings, controls, and relationships so the platform can produce trustworthy control narratives and audit-ready packages.

The key idea: **evidence is not a blob attached to a ticket.** It is a first-class object with provenance, freshness, scope, and relationship to findings and controls.

---

## Design principles

1. **Source first** — preserve original identifiers and payloads.
2. **Provenance always** — who collected it, from where, when, and how do we know it has not changed?
3. **Temporal truth** — evidence has an observation time and a collection time.
4. **Many-to-many relationships** — one artifact can support many controls; one control needs many evidence objects.
5. **Assessment-oriented** — model what an ISSO or assessor needs to decide whether implementation is credible.
6. **Machine-readable where possible** — align conceptually with OSCAL even if the operational schema is custom. [1]

---

## Core entity model

### 1. System
Represents an assessed boundary or subordinate system.

Fields:
- system_id
- name
- environment_type: aws_govcloud | on_prem | hybrid
- owner_org
- assessment_boundary_id
- cui_relevance: in_scope | supporting | out_of_scope
- inherited_control_sources
- metadata

### 2. Asset / resource
Represents a technical object.

Fields:
- asset_id
- system_id
- asset_type: ec2 | server | workstation | firewall | database | identity_group | document_repo | etc.
- native_id
- cloud_partition / region / account / site
- hostname_or_resource_name
- criticality
- owner
- tags

### 3. Finding
Represents a security/compliance observation from a source system.

Fields:
- finding_id
- source_system
- source_finding_id
- normalized_type
- title
- description
- severity
- confidence
- first_seen_at
- last_seen_at
- current_status
- raw_payload_ref
- asset_refs[]
- source_url_or_console_ref

### 4. Evidence object
Represents a specific piece of evidence.

Types:
- machine_snapshot
- event_log_extract
- configuration_export
- screenshot
- policy_document
- procedure_document
- access_review_record
- interview_note
- attestation
- training_record
- ticket_extract
- remediation_validation_result

Fields:
- evidence_id
- evidence_type
- title
- description
- source_system
- source_locator
- collected_at
- observed_at_start
- observed_at_end
- collector_identity
- collection_method: api | upload | agent | manual_link | import
- hash_sha256 (if content captured)
- storage_ref
- mime_type
- confidentiality_marking
- chain_of_custody_notes
- supersedes_evidence_id
- status: active | stale | superseded | revoked

### 5. Control catalog entry
- control_id
- framework
- framework_version
- family_or_domain
- statement_text
- assessment_objectives_text
- normative_reference

### 6. Mapping assertion
Connects findings/evidence/remediation to controls.
(See `control-mapping.md`.)

### 7. Remediation item / POA&M record
Represents governed work to address an issue.

Fields:
- remediation_id
- originating_finding_ids[]
- target_control_ids[]
- owner
- due_date
- status
- closure_criteria
- residual_risk_statement
- validation_evidence_ids[]

### 8. Evidence collection
Represents an audit packet, control packet, or review set.

Fields:
- collection_id
- scope
- purpose: periodic_review | internal_assessment | external_assessment | ad_hoc_investigation
- included_evidence_ids[]
- generated_at
- generated_by
- snapshot_hash

---

## Relationship model

Recommended relationship types:
- `FINDING_AFFECTS_ASSET`
- `FINDING_IMPLICATES_CONTROL`
- `EVIDENCE_SUPPORTS_CONTROL`
- `EVIDENCE_VALIDATES_REMEDIATION`
- `EVIDENCE_DESCRIBES_SYSTEM`
- `REMEDIATION_ADDRESSES_FINDING`
- `REMEDIATION_MITIGATES_CONTROL_GAP`
- `SYSTEM_INHERITS_CONTROL_FROM`
- `EVIDENCE_SUPERSEDES_EVIDENCE`
- `EXCEPTION_APPLIES_TO_CONTROL`
- `ASSESSMENT_PACKET_INCLUDES_EVIDENCE`

These can live in relational join tables, graph edges, or both.

---

## Provenance requirements

Every evidence object should answer:
1. Where did this come from?
2. When was it observed?
3. When was it collected into Vigilant ISSO?
4. Who or what collected it?
5. Is the content stored, linked, or both?
6. Can integrity be validated by hash or source identifier?
7. What system/scope/control does it relate to?
8. Is it still current enough for the intended assessment use?

This matters because audit support fails when teams cannot explain where a screenshot, export, or statement came from or whether it was current during the assessment period.

---

## Freshness model

Evidence should not be just present/absent. It needs freshness semantics.

### Suggested fields
- freshness_policy_id
- expected_refresh_interval_days
- stale_after
- freshness_status: current | warning | stale
- justification_if_stale

### Example policies
- daily config export: stale after 2 days
- monthly access review: stale after 45 days
- annual policy approval: stale after 395 days
- architecture diagram: stale after 180 days or significant change event

Freshness policy should be set by evidence type and sometimes by control.

---

## Evidence strength model

Add an `evidence_strength` dimension:
- direct_machine_observation
- direct_documentary_record
- derived_summary
- attestation_only
- interview_only
- inferred

This helps prevent a generated narrative from being mistaken for primary evidence.

---

## Recommended storage pattern

### Keep three layers
1. **Raw source layer**
   - original JSON, CSV, PDF, image, or export
2. **Normalized metadata layer**
   - structured fields for search/reporting/workflow
3. **Analytical narrative layer**
   - generated summaries, control notes, packet narratives

Never collapse these into one object.

---

## Example: control evidence chain

Requirement: flaw remediation / vulnerability handling

Possible chain:
- Inspector finding or on-prem scanner result
- patch policy document
- ticket showing assigned remediation
- maintenance/change record
- post-remediation scan result
- ISSO review note accepting closure

Vigilant ISSO should represent each link separately and connect them, rather than storing one big “proof” file.

---

## OSCAL alignment strategy

NIST OSCAL is useful as an interoperability target, especially for:
- catalog/control references
- system implementation statements
- assessment results
- plan/milestone exchange concepts [1]

Recommended approach:
- keep an internal operational schema optimized for workflow
- maintain export/import adapters for OSCAL-shaped data where valuable
- reuse OSCAL-like identifiers and concepts where they reduce translation pain

---

## Assumptions

1. Some evidence will remain external due to storage, sensitivity, or customer preference; metadata + locator + integrity proof should still be stored.
2. Screenshots alone should rarely be treated as primary evidence if stronger machine-readable exports exist.
3. LLM-generated summaries should be stored as derivative artifacts, not as primary evidence.

---

## Sources

1. NIST, “Open Security Controls Assessment Language (OSCAL).” https://pages.nist.gov/OSCAL/
2. NIST, “SP 800-171 Rev. 2, Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations.” https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final
3. AWS, “AWS Security Finding Format (ASFF).” https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-findings-format.html
4. AWS, “What Is AWS Config?” https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
5. AWS, “What Is AWS CloudTrail?” https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
