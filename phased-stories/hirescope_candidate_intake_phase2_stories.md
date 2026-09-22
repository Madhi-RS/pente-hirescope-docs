# HireScope — Candidate Intake and AI Résumé Pre-screening — Phase 2 User Stories

## Scope and rules

Candidate details and the résumé are collected before pre-screen evaluation. AI may extract facts, compare them with the applicant's opening-specific approved criteria, rank or recommend candidates when configured, and flag potential discrepancies. AI output is immutable advisory evidence: it never qualifies, rejects, or invites a candidate. An authorised HR Reviewer for the opening's organisation reviews the original résumé, extracted facts, analysis, recommendation, and discrepancies and makes the authoritative decision. Résumé pre-screening remains separate from the seven-area work-assessment score.

# US-CAN-01: Collect candidate information and résumé before pre-screening

## User Story

As an applicant, I want to submit the information and résumé required for my selected opening, so that the organisation can evaluate my complete application before deciding whether I receive an assessment link.

## Description

The application-intake step is bound to one organisation, opening, and approved opening version. It collects the configured required candidate fields and résumé before any AI pre-screen analysis. Failure to meet a manually defined criterion cannot reject the applicant before this collection is complete.

## Key Capabilities

- Opening-specific required fields and résumé upload.
- Protected storage, ownership, and access audit.
- Validation before pre-screen processing.
- No assessment credential at intake.

## Acceptance Criteria

- AC-01: Intake is bound to one active opening/version and collects the opening's configured required candidate details and résumé before creating `RESUME_SUBMITTED`.
- AC-02: Missing/invalid required details or résumé prevents submission with field-level guidance but does not create a rejection decision.
- AC-03: The original résumé and candidate details are linked to the application, protected by organisation/role scope, and every authorised or denied résumé access is audited.
- AC-04: Neither manual criteria nor AI can set a final pre-screen outcome before required details and résumé collection completes.
- AC-05: Intake never exposes or issues an assessment/test link.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| CAN-01-TC-01 | Complete opening-specific intake | Active opening has configured fields | Submit details and résumé | `RESUME_SUBMITTED` is linked to correct opening/version (AC-01) |
| CAN-01-TC-02 | Validation without rejection | Required field/résumé omitted or invalid | Submit intake | Validation appears; no HR/AI rejection outcome is created (AC-02) |
| CAN-01-TC-03 | Protected résumé | Two organisations and applications exist | Access own-scope and cross-scope résumé | Authorised access succeeds; cross-scope access fails; both are audited (AC-03) |
| CAN-01-TC-04 | Early-rule guardrail | Candidate answer appears not to meet a criterion | Attempt automatic rejection before upload completion | Decision is refused (AC-04) |
| CAN-01-TC-05 | Assessment-link boundary | Intake submitted | Inspect response/status/email | No assessment credential is present (AC-05) |

# US-CAN-02: Produce advisory AI résumé pre-screening and discrepancy analysis

## User Story

As an HR Reviewer, I want AI to analyse submitted candidate information and résumés against the approved opening criteria, so that I can review consistent advisory evidence across all applicants before making a decision.

## Description

For one or many applications to an opening, AI extracts relevant facts, compares them with the pinned criteria version, and produces an immutable analysis. It may provide match information, ranking/shortlist recommendation when configured, reasons, and potential discrepancies supported by the available submitted information. It must not claim that an applicant lied or use unspecified external verification sources.

## Key Capabilities

- Batch or individual analysis for all résumés in an opening.
- Candidate fact extraction and criteria comparison.
- Explainable advisory recommendation/ranking.
- Cross-field consistency checks and cautious discrepancy language.
- Processing failure/retry without candidate harm.

## Acceptance Criteria

- AC-01: AI processing starts only after required candidate data and résumé collection and uses the applicant's pinned organisation/opening/criteria version.
- AC-02: Each result records extracted qualifications/facts, criterion-by-criterion matching evidence, advisory analysis, recommendation/shortlist status when configured, reasons, model/configuration metadata, and time.
- AC-03: When multiple applications are processed together, every output remains attributable to its application and the system may present a comparative ranking/shortlist view without changing any application status.
- AC-04: Supported cross-field checks flag only `inconsistency detected`, `potential discrepancy`, `information requiring verification`, or equivalent cautious language and show the underlying conflicting facts/evidence.
- AC-05: AI never labels a candidate dishonest, invents verification sources, makes the final decision, issues an assessment invitation, or changes the seven-area work-assessment scores.
- AC-06: A processing failure preserves the application/résumé, creates a safe failure state, and permits authorised audited retry; retry creates a new result version rather than overwriting prior output.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| CAN-02-TC-01 | Complete-input prerequisite | Complete and incomplete applications exist | Request analysis for both | Only complete application is processed against pinned criteria (AC-01) |
| CAN-02-TC-02 | Explainable result | Résumé and approved criteria exist | Run analysis and inspect result | Facts, matches, analysis/recommendation, reasons, and metadata are present (AC-02) |
| CAN-02-TC-03 | Multiple résumé comparison | Ten submitted applications exist | Run configured batch pre-screen | Ten attributable outputs and advisory comparative view exist; no statuses change (AC-03) |
| CAN-02-TC-04 | Potential discrepancy | Graduation year and claimed experience conflict | Run analysis | Cautious flag identifies the conflicting facts and requests verification (AC-04) |
| CAN-02-TC-05 | Prohibited AI action/language | AI result exists | Attempt automatic decision/invite and inspect wording/scores | Action is refused; no dishonesty claim or seven-score change occurs (AC-05) |
| CAN-02-TC-06 | Failure and retry | Simulate processing failure | Retry as authorised user | Source remains; safe state/audit and a new immutable version are created (AC-06) |

# US-CAN-03: Review AI pre-screening and make the HR progression decision

## User Story

As an HR Reviewer, I want to review every candidate's résumé and advisory AI pre-screen result and record my decision, so that only a human determines who receives the assessment link.

## Description

The organisation-scoped review page shows candidate information, original résumé, extracted qualifications, pinned opening criteria, AI analysis/matches, discrepancies, recommendation, reasons, result version, and review status. HR may accept or reject the recommendation and add an opinion/comment, but cannot edit the AI output. HR records the final pre-screen decision and reason.

## Key Capabilities

- Complete pre-screen evidence view.
- Separate immutable AI output and HR comment/decision.
- Human-only progression decision.
- Assessment-link issuance only after qualifying HR action.

## Acceptance Criteria

- AC-01: Only HR authorised for the owning organisation may review the candidate information, original résumé, criteria, extracted facts, AI analysis/recommendation/reasons, discrepancies, version, and review status.
- AC-02: HR can accept or reject/disregard the AI recommendation and add a comment/opinion, but cannot edit, replace, or relabel AI-generated facts, reasons, or recommendation.
- AC-03: HR records `QUALIFIED_FOR_ASSESSMENT`, `NOT_CURRENTLY_QUALIFIED`, or an explicit pending/review state with decision reason, actor, selected AI version, and timestamp.
- AC-04: Only HR-set `QUALIFIED_FOR_ASSESSMENT` can generate/send the bound assessment invitation; AI output alone or a pending state cannot.
- AC-05: An HR non-qualification decision triggers the approved outcome/reason email and no assessment link; the full decision chain is auditable.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| CAN-03-TC-01 | Organisation-scoped evidence review | Analysis ready | Open as authorised and other-organisation HR | Full permitted evidence only for authorised HR; denial audited (AC-01) |
| CAN-03-TC-02 | Accept/reject without output mutation | AI recommends shortlist | Add comment and accept, then test reject on another candidate | HR action is separate; AI record remains unchanged (AC-02) |
| CAN-03-TC-03 | Human decision record | Analysis ready | Record each supported HR state | Actor, reason, selected version, and time are retained (AC-03) |
| CAN-03-TC-04 | Invitation gate | AI recommends candidate but HR has not qualified | Attempt issue before and after qualifying HR action | Only post-HR qualification issues the invitation (AC-04) |
| CAN-03-TC-05 | Non-qualification route | HR selects non-qualified | Inspect email, link state, and audit | Appropriate reason is sent without an assessment link; audit is complete (AC-05) |

# US-CAN-04: Show applicants only their allowed application state and outcome

## User Story

As an applicant, I want to see my permitted application state and receive useful outcome emails, so that I understand the result without seeing confidential internal scores or other candidates.

## Description

Applicants see their own intake, pre-screen, invitation, assessment, grievance, interview, and outcome status. Non-progression communications provide approved reason or evidence-based strengths/weaknesses where applicable, but never exact AI/interviewer scores, internal comments, rankings, or another applicant's data.

## Key Capabilities

- Own-application status and receipts.
- Email delivery for pre-screen and later outcomes.
- Strength/weakness summary without exact scores.
- Confidential-data suppression.

## Acceptance Criteria

- AC-01: An applicant can view only their own application and its permitted status/receipts.
- AC-02: Pre-screen qualification/non-qualification emails reflect the HR decision; only qualified email contains the secure assessment link.
- AC-03: Assessment/live-interview outcome email may state approved strengths and weaknesses but omits exact AI/interviewer scores, ranking, reviewer/interviewer comments, internal recommendation, and confidential rationale.
- AC-04: Status and email content never exposes other applicants, organisation-internal queues, or unauthorised identifiers.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| CAN-04-TC-01 | Own status | Applicant has application | Open own and another application's URL | Own state appears; other application is denied (AC-01) |
| CAN-04-TC-02 | Pre-screen email/link | HR records both decision types | Inspect emails | Only qualified candidate receives bound assessment link (AC-02) |
| CAN-04-TC-03 | Strength/weakness notification | Later outcome exists | Inspect applicant email | Narrative strengths/weaknesses may appear; exact scores/internal comments do not (AC-03) |
| CAN-04-TC-04 | Confidential isolation | Multiple applicants/organisations exist | Inspect status/email/API projection | No unrelated or internal data appears (AC-04) |
