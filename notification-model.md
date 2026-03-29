# Notification and Escalation Model

## Goal

Define how Vigilant ISSO should notify, route, and escalate operational and compliance issues using **Microsoft Teams and email** under the working assumption that those services are available **inside the organization’s CMMC Level 2 operating boundary**.

This document is deliberately conservative:
- notifications should move work forward, not create theater
- messages must preserve evidence links and scope context
- sensitive notifications should minimize copied content and prefer references into the authoritative platform
- policy and tenant configuration may restrict some channels or recipient types even when Teams and email are generally approved

---

## Verified facts

1. Microsoft Teams is built on Microsoft 365 groups, Microsoft Graph, SharePoint, and Exchange Online, and uses identities stored in Microsoft Entra ID. [1]
2. Microsoft states Teams uses enterprise security and compliance capabilities from Microsoft 365 / Office 365, with encryption in transit and at rest and customer-controlled tenant data. [2]
3. Microsoft Teams supports external access and guest access to collaborate with people outside the organization, which means a regulated deployment must deliberately govern or disable those paths if they are inconsistent with policy or boundary rules. [3]
4. Exchange Online is a hosted messaging solution in Microsoft datacenters and is typically reachable from inside the corporate network or over the Internet, so email-based workflows need classification and routing discipline. [4]
5. CMMC scope turns on systems that process, store, or transmit CUI and systems that provide security protection for those systems, so the organization must make its own formal scoping and policy determination about whether specific Teams/email uses are allowed for CUI-related workflows. [5]

---

## Boundary assumptions

### Working assumption for this concept pass
The organization has decided that:
- its Microsoft 365 tenant configuration used for Teams and email is part of, or authorized to support, the CMMC Level 2 operating environment
- access is limited to appropriately authorized internal users
- retention, audit, DLP, and external-sharing policies are configured to support the boundary

### Important caveat
Vigilant ISSO should not hard-code that all Teams chats or all email recipients are acceptable for all message types. That must remain policy-driven by tenant, enclave, data classification, and recipient class.

---

## Design principles

1. **Link, do not spill.** Prefer links to evidence, cases, and packets over dumping sensitive details into chat or email.
2. **Audience-bound messages.** Notifications should go only to the responsible queue, owner, or escalation role.
3. **Classification-aware content.** Message templates should vary by data sensitivity and channel policy.
4. **Stateful escalation.** Escalations should be tied to unresolved workflow states, not arbitrary timers alone.
5. **Two-way accountability.** Notifications should support acknowledgment, assignment, and review trails.

---

## Notification classes

## Class 1: Informational digests

Purpose:
- daily or weekly awareness
- posture summaries
- upcoming review reminders

Examples:
- daily ISSO summary
- weekly overdue remediation digest
- monthly readiness review reminder

Preferred channels:
- Teams channel post for operational groups
- email digest for leadership or control-owner groups

Data handling:
- summarize counts, trends, and links
- do not include raw sensitive payloads unless policy explicitly allows it

---

## Class 2: Action-required operational notifications

Purpose:
- assign work that needs a response

Examples:
- new high-severity finding in an in-scope enclave
- evidence refresh request
- remediation item assigned or due soon
- validation required before closure

Preferred channels:
- Teams message to owning queue/channel
- email to owner and backup owner where needed

Required elements:
- concise summary
- severity / priority
- system and scope label
- due date or SLA
- link to authoritative record
- acknowledgment action

---

## Class 3: Compliance-risk escalations

Purpose:
- surface issues likely to affect readiness materially

Examples:
- primary evidence for a critical control is stale
- exception expired without review
- repeated finding cluster indicates systemic control failure
- contradiction between SSP claim and telemetry remains unresolved
- monthly readiness snapshot blocked by open material gaps

Preferred channels:
- Teams escalation channel
- email to ISSO lead, compliance lead, and designated management roles

Required elements:
- issue statement
- readiness impact statement
- affected controls/systems
- actions already attempted
- explicit requested decision

---

## Class 4: Leadership / assessor-prep notifications

Purpose:
- prepare management or assessment support teams without oversharing

Examples:
- pre-assessment packet ready for review
- monthly readiness summary published
- significant architecture change affects scope

Preferred channels:
- email summary with packet link
- Teams channel post to approved leadership/compliance space

---

## Escalation triggers

### Immediate escalation
Use for:
- critical/high-risk issue on in-scope CUI assets
- logging or monitoring failure that blinds ongoing evidence collection
- public exposure or identity-control issue with material boundary impact
- collector failure that breaks required visibility for a major control area

### Escalate after missed response window
Use for:
- owner failed to acknowledge a high-priority item
- evidence refresh request not actioned in time
- remediation nearing or passing deadline

### Escalate after governance threshold
Use for:
- repeated reopenings
- expired exception
- stale primary evidence for high-value controls
- unresolved contradiction persisting across review cycle

---

## Routing model

Every notification should resolve routing from metadata, not free text.

Required routing keys:
- business unit
- system / enclave
- asset owner
- control owner
- severity / priority
- data sensitivity class
- workflow state

Recommended recipient groups:
- ISSO operations team
- control-owner groups
- engineering/platform queues
- security operations queue
- compliance leadership
- approving authorities for exceptions/accepted risk

---

## Teams model

## Recommended channel structure

Use Teams mainly for **team-visible workflow coordination**, not as the system of record.

Suggested spaces:
- `ISSO-Ops / Daily-Triage`
- `ISSO-Ops / Evidence-Refresh`
- `ISSO-Ops / Readiness-Review`
- `Security-Engineering / Remediation`
- `Compliance-Leadership / Escalations`
- `Assessment-Prep / Packet-Review`

### Recommended Teams behaviors
- post concise cards with links back to Vigilant ISSO
- use thread replies for case discussion
- require acknowledgment actions for material items
- avoid pasting full evidence documents into chat when a controlled link is enough

### Caveats
Because Teams supports guest and external access, the tenant’s external collaboration settings matter. If policy forbids external participants or certain chat modes for boundary-related work, Vigilant ISSO must enforce recipient restrictions accordingly. [3]

---

## Email model

Use email for:
- formal owner assignment
- escalation outside the immediate working queue
- leadership summaries
- packet review requests
- durable notice to stakeholders who do not live in Teams

### Recommended email practices
- subject line should encode severity, system, and workflow state
- include a short summary plus authoritative link
- do not embed large evidence payloads by default
- use distribution groups maintained by role, not individuals only
- preserve message classification/banner behavior required by policy

### Example subject patterns
- `[Vigilant ISSO][Action Required][High] GovCloud-Prod: Critical finding assigned`
- `[Vigilant ISSO][Escalation] Primary evidence stale for AC-3 packet`
- `[Vigilant ISSO][Monthly Review] March readiness snapshot awaiting approval`

---

## Notification content levels

### Level A: Metadata only
Suitable when policy is strict or audience is broad.

Includes:
- issue type
- severity
- system name
- due date
- link

### Level B: Controlled summary
Suitable for approved internal operational channels.

Includes:
- concise description
- affected controls
- owner
- readiness impact
- next action
- link

### Level C: Detailed evidence summary
Use only when policy and audience permit.

Includes:
- evidence ids
- source system names
- finding synopsis
- contradiction explanation
- packet references

Vigilant ISSO should choose the lowest sufficient level by policy.

---

## Escalation ladder

### Tier 0: System-generated alert to working queue
Used for new issues and reminders.

### Tier 1: Owner + ISSO reminder
Used when acknowledgment or action is missing.

### Tier 2: Control owner / team lead escalation
Used when SLA breach or evidence risk emerges.

### Tier 3: Compliance leadership escalation
Used when readiness is materially affected or exceptions expire.

### Tier 4: Executive / designated approving authority notice
Used for significant unresolved risk, scope-impacting issues, or blocked assessment readiness.

Escalation should include prior attempts and elapsed time so leaders see process discipline, not just noise.

---

## Interactive actions

Notifications should support structured responses such as:
- acknowledge
- accept owner assignment
- request scope review
- request exception review
- upload / link evidence
- mark remediation complete and request validation
- approve monthly snapshot

The response should update the authoritative workflow state, not live only in Teams or email.

---

## Guardrails

1. **No channel should become the evidence repository.** Teams and email are transport and coordination layers.
2. **No auto-escalation to external users.** External recipients should be blocked unless explicitly policy-approved.
3. **No hidden downgrades.** If an issue is materially affecting readiness, the notification must say so plainly.
4. **No silent reminder storms.** Use rate limits, bundling, and escalation states.
5. **Every notification needs provenance.** Record what was sent, to whom, when, and for which workflow object.

---

## Verified facts vs assumptions

### Verified facts
- Teams is built on Microsoft 365 services including Exchange Online, SharePoint, Entra ID, and Microsoft 365 groups. [1]
- Teams includes security/compliance features and also supports guest/external collaboration modes that require governance. [2][3]
- Exchange Online is Microsoft-hosted email and therefore requires message-classification and routing discipline for regulated use. [4]
- CMMC scoping obligations remain the organization’s responsibility. [5]

### Working assumptions
- The tenant used here is considered approved for internal CMMC-related operations.
- Teams channels and email distribution groups can be treated as inside-boundary communications only if external access, guest access, retention, and auditing are configured to policy.
- For most notifications, metadata-plus-link is safer and more scalable than embedding detailed evidence in the message body.

---

## Sources

1. Microsoft Learn, “Introduction to Microsoft Teams for admins.” https://learn.microsoft.com/en-us/microsoftteams/teams-overview
2. Microsoft Learn, “Overview of security and compliance - Microsoft Teams.” https://learn.microsoft.com/en-us/microsoftteams/security-compliance-overview
3. Microsoft Learn, “Use guest access and external access to collaborate with people outside your organization.” https://learn.microsoft.com/en-us/microsoftteams/communicate-with-users-from-other-organizations
4. Microsoft Learn, “Exchange Online service description.” https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-service-description
5. eCFR, “32 CFR Part 170 — Cybersecurity Maturity Model Certification (CMMC) Program.” https://www.ecfr.gov/current/title-32/subtitle-A/chapter-I/subchapter-G/part-170
