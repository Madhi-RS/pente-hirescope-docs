# HireScope — Requisition and Opening Management — Phase 1 User Stories

## Scope and rules

An opening starts only from an approved workforce requisition and is owned by exactly one organisation. Organisation membership limits access, but configuration belongs to the individual opening: two openings owned by the same organisation may have different modalities, sections, timing, re-record limits, pre-screen criteria, and live-interview settings. Published versions are immutable and changes require approval. The seven work-assessment areas remain separate from résumé pre-screening criteria.

### Requisition lifecycle

```text
REQUISITION_DRAFT
  → (Hiring Manager submits) → REQUISITION_PENDING_APPROVAL
      → (HR approves) → REQUISITION_APPROVED → OPENING_DRAFT
      → (HR rejects/returns with required comment) → REQUISITION_REJECTED
          → (Hiring Manager revises + resubmits) → REQUISITION_PENDING_APPROVAL
```

Each submission locks an immutable version; a returned requisition is revised and resubmitted as a new version, and only `REQUISITION_APPROVED` may create an opening. Approval/rejection outcomes are notified to the raising Hiring Manager by email.

# US-REQ-01: Create, submit, and resubmit a workforce requisition

## User Story

As a Hiring Manager, I want to create, submit, and — when returned — revise and resubmit a workforce requisition, so that an HR Reviewer can decide whether a job opening may be created.

## Description

The manager owns a draft through submission. Required business information is captured before status changes from `REQUISITION_DRAFT` to `REQUISITION_PENDING_APPROVAL`. Submission locks an immutable submitted version. A returned/rejected requisition is released back to the manager as an editable draft; revision and resubmission create a new immutable submitted version for re-review.

## Key Capabilities

- Save, edit, or discard a draft before submission.
- Capture role/title, position count, business justification, job description, required experience, salary range, employment type, and eligibility criteria.
- Make each submitted version read-only to the manager and audit lifecycle actions.
- Revise and resubmit a returned requisition as a new immutable version for re-review.

## Acceptance Criteria

- AC-01: A requisition cannot be submitted without all required business and eligibility fields.
- AC-02: A manager can save/edit/discard only a draft owned by their organisation; submission changes status to `REQUISITION_PENDING_APPROVAL` and locks that version.
- AC-03: Creation, edits, submission, discard, and resubmission are audited with organisation, actor, and time.
- AC-04: A returned/rejected requisition is released to the manager as an editable draft; revision and resubmission create a new immutable submitted version and return status to `REQUISITION_PENDING_APPROVAL`, while every prior submitted version remains read-only and audited.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| REQ-01-TC-01 | Required requisition data | Manager signed in | Omit each required field, then submit complete data | Incomplete submission fails; complete submission succeeds (AC-01) |
| REQ-01-TC-02 | Draft ownership and submission | Two organisations and one draft exist | Edit as owner and non-owner; submit as owner | Owner actions work; cross-organisation action fails; submitted version is pending/read-only (AC-02) |
| REQ-01-TC-03 | Requisition audit | Requisition has lifecycle activity | Inspect audit history | Organisation, actor, time, and all listed actions are retained (AC-03) |
| REQ-01-TC-04 | Return and resubmit version | A requisition was returned | Revise, resubmit, then inspect status and versions | Resubmission creates a new immutable submitted version, returns status to `REQUISITION_PENDING_APPROVAL`, and prior submitted versions remain read-only/audited (AC-04) |

# US-REQ-02: Approve or return a requisition and re-review resubmissions

## User Story

As an HR Reviewer, I want to approve or return a requisition with a comment and re-review any resubmission, so that only approved headcount becomes an opening.

## Description

An authorised HR Reviewer for the owning organisation reviews each submitted requisition. Approval sets `REQUISITION_APPROVED`; rejection/return sets `REQUISITION_REJECTED`, requires a comment, and releases the requisition to the manager for revision. A resubmitted requisition re-enters the organisation's approval queue for re-review through the same path.

## Key Capabilities

- Organisation-scoped approval queue and separation of duties.
- Approved headcount becomes the opening limit.
- Email notification of the decision to the raising Hiring Manager.
- Re-review of each resubmitted requisition.
- Audit of every decision cycle.

## Acceptance Criteria

- AC-01: Only an HR Reviewer authorised for the owning organisation can approve or reject/return a pending requisition; a manager cannot approve their own work.
- AC-02: Rejection/return requires a comment; every decision records organisation, actor, time, and comment/status.
- AC-03: No opening can be created or published from a requisition that is not approved.
- AC-04: Approval/rejection notifies the raising Hiring Manager by email (through the product notification channel) and is audited.
- AC-05: A rejected/returned requisition returns to an editable state; resubmission places it back in the organisation's approval queue, and each decision cycle (review, decision, required comment, notification) is independently audited.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| REQ-02-TC-01 | Organisation scope and separation | Pending requisition | Decide as its manager, another organisation's HR, and authorised HR | Only authorised HR succeeds (AC-01) |
| REQ-02-TC-02 | Return comment | Pending requisition | Reject without then with comment | Comment is required and retained with decision metadata (AC-02) |
| REQ-02-TC-03 | Approved-only opening | Approved and unapproved requisitions exist | Attempt opening creation/publish from both | Only approved requisition can progress (AC-03) |
| REQ-02-TC-04 | Decision email/audit | Decision made | Check email notification and audit | Manager receives an email and audit is complete (AC-04) |
| REQ-02-TC-05 | Resubmission re-review | A requisition was returned | Revise/resubmit as manager, then re-review as HR | Resubmission re-enters the queue, re-review is recorded, and each cycle is audited (AC-05) |

# US-OPN-01: Draft and approve an organisation-owned opening and FairPack

## User Story

As a Hiring Manager, I want to prepare an opening and FairPack from an approved requisition, so that HR can approve a consistent assessment for my organisation before applicants see it.

## Description

The system creates an `OPENING_DRAFT` owned by the requisition's organisation and tied to approved headcount. AI may draft a FairPack, but the manager edits and previews it. HR approves the opening, opening configuration, pre-screen criteria, and FairPack before publication. A returned package is released to the manager for revision and may be resubmitted for re-review.

## Key Capabilities

- Organisation ownership and isolation.
- AI draft plus manager edit/preview.
- Seven fixed online-assessment areas and an approval gate before publication.
- Return, revision, and resubmission of the opening package.
- Email notification of the decision to the Hiring Manager.

## Acceptance Criteria

- AC-01: Opening creation is available only from an approved requisition and records its owning organisation and approved position count.
- AC-02: The FairPack contains seven job-related areas, a designed-missing-input work sample, configured structured questions, enabled baselines/sections, and 1–4 anchors.
- AC-03: The manager can edit and preview the draft, but cannot make school, employer brand, gaps, culture fit, or grit an online-assessment score.
- AC-04: HR approval of the opening, pre-screen criteria, opening configuration, and FairPack is required before publication.
- AC-05: A user outside the owning organisation cannot read/change the opening or associate it with another organisation's candidates/configuration.
- AC-06: HR may reject/return the opening package with a recorded reason, releasing it to the manager for revision; a revised package may be resubmitted for re-review, and every decision cycle is audited.
- AC-07: Opening approval/rejection/return notifies the Hiring Manager by email (through the product notification channel).

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPN-01-TC-01 | Approved requisition linkage | Approved and rejected requisitions exist | Create opening from each | Only approved source creates an organisation-owned draft with headcount (AC-01) |
| OPN-01-TC-02 | Required FairPack instrument | Opening draft/configuration exists | Generate/inspect FairPack | Required components follow the opening configuration and seven 1–4 areas exist (AC-02) |
| OPN-01-TC-03 | Anti-gate guardrail | Manager edits FairPack | Try to score brand, gaps, culture fit, or grit | Scored-gate save is refused (AC-03) |
| OPN-01-TC-04 | Approval gate | Draft is ready | Publish before and after HR approval | Before is refused; approved draft can progress (AC-04) |
| OPN-01-TC-05 | Organisation isolation | Two organisations exist | Access/associate Organisation A's opening as Organisation B | Cross-organisation operation is refused and audited (AC-05) |
| OPN-01-TC-06 | Package return and resubmission | Draft package was returned | Revise, resubmit, then inspect state/audit | Resubmission re-enters review and the full cycle is audited (AC-06) |
| OPN-01-TC-07 | Decision email | HR decides the package | Check manager's email and audit | Manager receives an email; audit is complete (AC-07) |

# US-OPN-02: Publish, version, and change an opening safely

## User Story

As a Hiring Manager, I want approved changes to create a new version, so that active applicants remain on the opening configuration and assessment version they started.

## Description

Publication creates an immutable active version. A proposed edit creates a change request; HR approval activates it for future applicants without altering the active version, in-flight applications, or other openings. A rejected/returned change request may be revised and resubmitted; opening-level configuration cannot bypass this approval workflow.

## Key Capabilities

- Immutable published and historical versions.
- Approval workflow for opening, configuration, criteria, and FairPack changes; configuration cannot bypass HR approval.
- Change-request return, revision, and resubmission.
- Email notification of the decision to the Hiring Manager.
- Opening/version isolation and audit.

## Acceptance Criteria

- AC-01: Publication creates an immutable active opening/configuration/FairPack version only after approval.
- AC-02: Direct modification is unavailable; **Propose Change** creates a separate draft/change request.
- AC-03: Approval activates a new version only for future candidates; in-flight applications retain their started version.
- AC-04: Rejection leaves the active version unchanged, and all versions remain readable, immutable, and audited.
- AC-05: A change affects only its target opening and never another opening in the same or a different organisation.
- AC-06: A rejected/returned change request may be revised and resubmitted for re-review; opening-level configuration changes follow the same change-request → HR approval path and cannot be applied directly or bypass HR approval.
- AC-07: Approval/rejection/return of a change notifies the Hiring Manager by email (through the product notification channel).

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPN-02-TC-01 | Publish approved version | Approved opening | Publish and inspect version | Immutable active opening/configuration/FairPack version is created (AC-01) |
| OPN-02-TC-02 | Direct-edit prevention | Published opening | Attempt direct edit; propose change | Direct edit fails; separate change draft exists (AC-02) |
| OPN-02-TC-03 | In-flight pinning | Candidate started v1; v2 approved | Activate v2 and inspect packet | New applicants receive v2; existing application retains v1 (AC-03) |
| OPN-02-TC-04 | Rejected change/history | Active version and change request exist | Reject and inspect history | Active version is unchanged; history/audit remains (AC-04) |
| OPN-02-TC-05 | Opening isolation | Two openings exist | Change one and inspect both | Only target opening changes (AC-05) |
| OPN-02-TC-06 | Change return/resubmit and config approval | A change request exists | Reject, revise, resubmit, and try a direct configuration edit | Direct edit fails; resubmitted change re-enters approval and cannot bypass HR (AC-06) |
| OPN-02-TC-07 | Change decision email | A change is decided | Check manager's email and audit | Manager receives an email; audit is complete (AC-07) |

# US-OPN-03: Advertise an approved opening and retain source attribution

## User Story

As an authorised Hiring Manager or HR user, I want to record approved external advertising, so that candidate source information is available without granting assessment access publicly.

## Description

Only an approved opening may be advertised. An external posting points to an opening/application-intake link; it is not an assessment credential.

## Key Capabilities

- Capture source, campaign/posting reference, URL, and opening version.
- Retain audited corrections and organisation ownership.
- Keep public application intake separate from assessment access.

## Acceptance Criteria

- AC-01: An advertisement may be recorded only for an approved active opening by a user in its organisation.
- AC-02: The record stores source type, optional reference/URL, opening version, organisation, and corrections with actor/reason.
- AC-03: The external link routes to application intake and cannot start an assessment.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPN-03-TC-01 | Approved/owned advertisement | Draft and active openings exist | Record an ad as owner and outsider | Only owner on active opening succeeds (AC-01) |
| OPN-03-TC-02 | Attribution history | Active posting exists | Save and correct source | Required metadata and audited correction remain (AC-02) |
| OPN-03-TC-03 | Link boundary | External link exists | Open it and try to start assessment | Application intake opens; no assessment credential is granted (AC-03) |

# US-OPN-04: Configure opening-specific pre-screen criteria and review rules

## User Story

As a Hiring Manager, I want to propose pre-screen criteria and applicant questions while creating an opening, so that AI can produce advisory résumé analysis and HR can make the actual progression decision for that role.

## Description

The **Eligibility and Pre-screen** section defines visible criteria, required candidate information, questions, validation, evidence expectations, effective version, and HR review instructions. HR approves it with the opening. Criteria guide AI analysis only after candidate information and résumé collection; they cannot automatically reject, qualify, or invite a candidate.

## Key Capabilities

- Configure opening-specific criteria/questions and required evidence.
- Freeze criteria with the opening version.
- Keep pre-screen criteria outside the seven work-assessment scores.
- Require human review and decision.

## Acceptance Criteria

- AC-01: The opening draft includes role-specific criteria, candidate information requirements, and questions.
- AC-02: Every question records label, help text, answer type, required status, validation, options/range where applicable, and rule version.
- AC-03: Every criterion records the facts/evidence to compare and the HR review procedure; AI output cannot directly set qualification/non-qualification or issue an assessment invitation.
- AC-04: HR approves the complete criteria version before publication; only the applicant's pinned version is used.
- AC-05: Approved experience and other job requirements may be compared with candidate/résumé facts for pre-screen analysis but never become seven-area assessment scores.
- AC-06: Active criteria are immutable; revisions use a change request and do not alter in-flight applications.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPN-04-TC-01 | Draft criteria | Approved requisition/opening draft exist | Add criteria/questions | Dedicated section retains opening-specific content (AC-01) |
| OPN-04-TC-02 | Question metadata | Draft exists | Configure several answer types | Required metadata/validation persist (AC-02) |
| OPN-04-TC-03 | Human decision boundary | Criteria exist | Bind AI result directly to qualification/invite | Automation is refused; HR review remains mandatory (AC-03) |
| OPN-04-TC-04 | Approval/versioning | Manager submits criteria | Use before approval, then approve | Unapproved criteria unavailable; approved pinned version is used (AC-04) |
| OPN-04-TC-05 | Assessment-score boundary | Résumé and criteria exist | Analyse and inspect seven-area scoring | Pre-screen criteria do not alter seven assessment scores (AC-05) |
| OPN-04-TC-06 | Active criteria change | Applicant is in flight | Direct edit then approved change | Direct edit fails; future version does not affect applicant (AC-06) |

# US-OPN-05: Configure assessment delivery for a specific opening

## User Story

As an authorised organisation user, I want to configure assessment delivery for each job opening, so that applicants receive only the sections, timing, recording rules, and interview workflow approved for their opening.

## Description

**Create Job Opening** includes an opening-level configuration step. The configuration is owned by that opening, versioned, validated before publication, and retrieved when an applicant enters the assessment. It is not a single organisation-wide configuration. Audio is currently supported. Video is planned only and cannot be activated yet. Network readiness is an acknowledgement; HireScope does not measure network speed.

## Key Capabilities

- Configure supported modality, sections, questions, expected duration/time limit, and recording/re-record rules.
- Require a maximum re-record count whenever re-recording is enabled.
- Configure required live-interviewer count and an approved aggregation/display policy when available.
- Persist, version, retrieve, validate, and isolate configuration.

## Acceptance Criteria

- AC-01: Each opening version has its own configuration linked to its owning organisation; saving one opening never changes another.
- AC-02: Supported fields include audio enabled/disabled, self-introduction/read-passage/question-answer enabled/disabled, question count/definitions, expected duration/time limit, and live-interviewer count.
- AC-03: When audio recording is enabled, re-recording can be enabled/disabled; enabling it requires a maximum re-record count that is enforced per configured section/answer.
- AC-04: Video is planned/unavailable and cannot be activated. General text modality, live-stage optionality, section timing, preparation time, and disabling core assessment AI remain decision-gated.
- AC-05: Validation rejects no usable response path, recording without its modality, re-recording without recording or a maximum, invalid timing/question counts, and unapproved video/aggregation settings.
- AC-06: Assessment entry retrieves the configuration pinned to the application/opening version and presents/enforces only enabled supported sections/modalities.
- AC-07: Configuration create/change/approval/publication/retrieval and denied cross-organisation/opening access are audited.
- AC-08: Opening-level configuration is approved with the opening package (OPN-01); any post-publication configuration change follows the same change-request → HR approval path (OPN-02) and cannot be applied directly or bypass HR approval.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPN-05-TC-01 | Independent configurations | One organisation owns two drafts | Save different values for each | Each version retains its own values (AC-01) |
| OPN-05-TC-02 | Supported fields | Draft exists | Configure supported fields | Values persist against the opening version (AC-02) |
| OPN-05-TC-03 | Re-record maximum | Audio enabled | Enable without/with maximum; exceed limit | Missing maximum fails; configured limit is enforced (AC-03) |
| OPN-05-TC-04 | Planned/decision-gated options | Draft exists | Enable video/general text/unapproved aggregation | Option cannot become active (AC-04) |
| OPN-05-TC-05 | Validation | Draft has inconsistent combinations | Validate/submit | Each invalid combination is refused (AC-05) |
| OPN-05-TC-06 | Applicant retrieval | Applications pinned to different versions | Enter each assessment | Correct configuration is displayed/enforced (AC-06) |
| OPN-05-TC-07 | Isolation/audit | Two organisations exist | Cross-access and inspect audit | Access fails and events are auditable (AC-07) |
| OPN-05-TC-08 | Configuration approval gate | Draft and published opening exist | Change configuration directly then through Propose Change | Direct change fails; only an HR-approved change request activates (AC-08) |

## Opening Configuration Matrix

| Configuration Category | Opening-Level? | Required? | Applicant Impact | HR Impact | Source | Story |
|---|---:|---|---|---|---|---|
| Audio modality | Yes | Explicit/current | Audio preparation and recording sections only when enabled | Approves per-opening use | New product requirement | OPN-05, ASM-02/03 |
| Video modality | Yes, future | **DECISION REQUIRED — planned only** | Future camera/lighting/framing preparation; absent now | Cannot activate yet | New product requirement clarification | OPN-05 |
| General text modality | No in baseline | **DECISION REQUIRED**; accommodation-only now | Typed route after approval | Approves accommodation | HireScope.md; new requirement | OPN-05, ASM-04 |
| Self-introduction | Yes | Explicit | Appears only when enabled | Reviews instrument | New product requirement | OPN-05, ASM-03 |
| Read-a-passage | Yes | Explicit | Appears only when enabled | Reviews instrument | New product requirement | OPN-05, ASM-03 |
| Question-answer section | Yes | Explicit | Appears only when enabled | Reviews instrument | New product requirement | OPN-05, ASM-03 |
| Number/question definitions | Yes | Explicit | Controls presented questions | Approves count/content | New product requirement | OPN-05 |
| Expected duration/time limit | Yes | Explicit | Shown at readiness and enforced | Approves timing | New product requirement | OPN-05, ASM-02 |
| Section-specific duration | Yes if adopted | **DECISION REQUIRED** | Would add section timers | Approves policy | New requirement (conditional) | OPN-05 |
| Preparation/instruction time | Yes if adopted | **DECISION REQUIRED** | Would add preparation time | Approves policy | New requirement (conditional) | OPN-05 |
| Re-record allowance/maximum | Yes | Explicit when recording enabled | Controls versioned attempts | Defines maximum | New product requirement | OPN-05, ASM-03 |
| AI résumé pre-screen criteria | Yes | Explicit | Advisory analysis; no automatic decision | Reviews criteria/output | New product requirement | OPN-04, CAN-02/03 |
| Seven-area response analysis | Pinned to version | Required by current workflow | Latest submitted answers analysed | Reviews immutable output | HireScope.md; new requirement | REV-01 |
| Disable core response analysis | Yes if adopted | **DECISION REQUIRED** | Would change assessment workflow | Product approval required | New requirement (conditional) | OPN-05 |
| Live interview required | Yes if adopted | **DECISION REQUIRED** | Determines whether live stage occurs | Determines progression path | New requirement (conditional) | OPN-05, INT-01 |
| Number of live interviewers | Yes | Explicit | No direct test impact | Drives required submissions | New product requirement | OPN-05, INT-01/03 |
| Live categories/scale | Pinned to role family | Required; existing ten areas/1–5 | Exact scores not emailed | Common rubric | Existing Flow/QA | INT-02 |
| Score aggregation/display | Yes | Configurable when used; methods **DECISION REQUIRED** | No exact aggregate emailed | Selects versioned policy | New product requirement | OPN-05, INT-02/03 |
| Network readiness | No measurement setting | Acknowledgement required | Confirms connectivity readiness | No speed result | New product requirement clarification | ASM-02 |
