# HireScope — Live Interview and Final Decision — Phase 3 User Stories

## Scope and rules

HR decides whether a candidate progresses to live interview after reviewing authorised candidate information and prior evidence. A live interview supports one or more assigned interviewers according to the pinned opening configuration. Each interviewer owns a separate immutable rubric submission; no score overwrites another interviewer's result or any AI result. AI, interviewer, reviewer-comment, aggregation (if approved), and final-HR-decision provenance remain distinct.

# US-INT-01: Assign one or more live interviewers with scoped access

## User Story

As HR, I want to assign the configured number of participants to a recommended candidate's live interview, so that every required interviewer can review the permitted brief and submit independent evidence.

## Description

HR creates an event only after its human `LIVE_INTERVIEW_RECOMMENDED` decision. It assigns one or more interviewers/participants and roles according to the application-pinned opening configuration. Assigned interviewers may review relevant candidate information authorised for the live stage, including résumé, candidate details, pre-screen information, previous assessment results, and read-only AI evidence where permitted. Access is event- and organisation-scoped.

## Key Capabilities

- Multi-participant event/role assignment and notification.
- Required-participant validation from opening configuration.
- Read-only, assignment-scoped candidate/prior-evidence brief.
- No unrelated data or candidate-status control.

## Acceptance Criteria

- AC-01: HR can create an event only for an HR-set `LIVE_INTERVIEW_RECOMMENDED` application and must assign the number/roles required by the pinned opening configuration.
- AC-02: The event records organisation, application/candidate, interview identity, each participant identity/role, required/optional status, and assignment history.
- AC-03: Only assigned participants can access the event; each assignment/reassignment/revocation sends the required notification and is audited.
- AC-04: Assigned interviewers may view only the read-only candidate details, résumé, pre-screen/assessment/AI evidence, role guidance, logistics, and non-medical accommodation instructions authorised for that event.
- AC-05: Interviewers cannot edit/reprocess AI results, access unrelated applications, submit for another interviewer, or change progression/final status.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| INT-01-TC-01 | Human-decision/configuration gate | Recommended/held applications exist; count=2 | Create events | Only recommended application accepts exactly required assignment set (AC-01) |
| INT-01-TC-02 | Participant identities/roles | Multi-participant event | Inspect assignment record | Interview/candidate and every participant/role/status are retained (AC-02) |
| INT-01-TC-03 | Assignment scope/notice | Assigned/unassigned users exist | Open/reassign/revoke | Only current assignees enter; notifications/audit exist (AC-03) |
| INT-01-TC-04 | Relevant read-only brief | Interviewer assigned | Open brief | Only authorised relevant candidate/prior evidence is visible read-only (AC-04) |
| INT-01-TC-05 | Interviewer limits | Assigned interviewer | Edit AI, open unrelated candidate, submit as peer, change status | All actions are denied/audited (AC-05) |

# US-INT-02: Capture each interviewer's structured ten-area evidence

## User Story

As an assigned interviewer, I want my own role-guided ten-area rubric with evidence comments, so that my evaluation remains attributable and separate from AI and other interviewers.

## Description

The existing ten categories and 1–5 scale apply. Scores 2 and 4 remain blocked until their anchor wording receives formal HR approval. Every category requires an evidence-based comment. An approved aggregation/display policy may produce a separate derived view, but it cannot overwrite individual results.

## Key Capabilities

- Existing ten-area, role-family-guided rubric.
- Per-interviewer 1–5 scores/comments and provenance.
- Separate AI/human/derived evidence blocks.
- Optional configured aggregation only through an approved versioned policy.

## Acceptance Criteria

- AC-01: Each assigned scorer receives the existing ten categories and approved role-family guidance from the pinned opening version.
- AC-02: Every category accepts a 1–5 score using approved anchors and requires an evidence-based comment; score-2/4 anchor use is blocked pending formal approval.
- AC-03: Every draft/submitted entry records organisation, candidate/application, interview, interviewer identity, category, score, comment, and status.
- AC-04: The interface separates résumé pre-screen AI, assessment AI, each named interviewer's evaluation, reviewer comments, configured derived aggregation, and final HR decision.
- AC-05: An aggregation is shown only when the opening selects an approved versioned policy; source scores remain visible/preserved and unapproved or missing policy produces no combined score.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| INT-02-TC-01 | Complete role rubric | Supported role-family event | Open rubric | Ten categories/correct guidance appear (AC-01) |
| INT-02-TC-02 | Score/comment/anchor validation | Rubric open | Submit incomplete and pending 2/4 anchor, then valid entries | Invalid entries fail; approved 1–5 evidence is retained (AC-02) |
| INT-02-TC-03 | Interviewer-level provenance | Two interviewers score same category | Inspect records | Both identity-linked results coexist (AC-03) |
| INT-02-TC-04 | Evidence separation | Review page open | Inspect labels | AI, each interviewer, derived, comment, and HR decision blocks are distinct (AC-04) |
| INT-02-TC-05 | Aggregation policy gate | Approved/unapproved/no policy cases exist | View results | Derived view appears only for approved selected policy; sources stay intact (AC-05) |

# US-INT-03: Save and permanently submit participant-specific results

## User Story

As an assigned interviewer, I want to save my draft and submit my own immutable result, so that multiple interviewers can complete independently without overwriting one another or changing candidate status.

## Description

Drafts are editable by their owner. Submission validates and locks only that participant's result. The event reaches `INTERVIEW_COMPLETED` when all participants marked required by the pinned configuration have submitted; this is factual completion only.

## Key Capabilities

- Owner-editable drafts.
- Immutable interviewer-specific submission.
- Required-participant completion tracking.
- Preservation of individual results under aggregation.

## Acceptance Criteria

- AC-01: Each assigned scorer can save/edit only their own draft without changing application/interview completion status.
- AC-02: **Submit & Complete Interview** validates all required category scores/comments and permanently locks that interviewer's result with submission time.
- AC-03: A submitted result cannot be edited, overwritten by another interviewer, merged into AI output, or replaced by an aggregation.
- AC-04: `INTERVIEW_COMPLETED` is set only after all configured required participant submissions exist; optional/missing required statuses are visible and no candidate decision is set.
- AC-05: Reassignment/removal never deletes a prior submitted result; corrections, if later authorised, require a new attributed revision and reason.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| INT-03-TC-01 | Draft ownership | Two assigned scorers | Save own and edit peer draft | Own succeeds; peer edit fails; no completion (AC-01) |
| INT-03-TC-02 | Complete submission | Draft missing then complete | Submit both | Missing fails; complete submission locks with time (AC-02) |
| INT-03-TC-03 | No overwrite | Two submitted results/aggregation exist | Edit/overwrite/merge | Every mutation is refused; sources persist (AC-03) |
| INT-03-TC-04 | All-required completion | Two required and one optional participant | Submit incrementally | Completion only after both required; no HR outcome set (AC-04) |
| INT-03-TC-05 | Reassignment/history | Submitted scorer is reassigned | Inspect result and attempt correction | Original remains; only governed revision path is allowed (AC-05) |

# US-DEC-01: Make the final HR disposition and notify the applicant safely

## User Story

As HR, I want to record the final decision after reviewing separate AI and human evidence, so that HireScope's outcome remains human-owned, explainable, and appropriately communicated.

## Description

After `INTERVIEW_COMPLETED`, HR reviews résumé/candidate information, pre-screen analysis, selected immutable assessment AI version, reviewer comments, each immutable interviewer result, and any approved derived view. HR records `ADVANCED`, `HOLD`, or `REJECTED`. The applicant email communicates approved strengths and weaknesses without exact scores.

## Key Capabilities

- Final human-only status action.
- Separate provenance and audit record.
- Strength/weakness applicant email without exact scores.

## Acceptance Criteria

- AC-01: Final decision is available only after `INTERVIEW_COMPLETED` and authorised HR records `ADVANCED`, `HOLD`, or `REJECTED`.
- AC-02: Hiring Managers, interviewers, and AI may contribute evidence but cannot set final status.
- AC-03: The decision records organisation, actor, time, relevant source/version references, configured aggregation policy/version if used, and reason without modifying evidence.
- AC-04: Applicant email states the approved outcome and evidence-based strengths/weaknesses, but excludes exact AI/interviewer/aggregate scores, rankings, internal comments, and confidential rationale.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| DEC-01-TC-01 | Completion precondition | Incomplete/completed events | Decide both | Only completed event permits HR decision (AC-01) |
| DEC-01-TC-02 | HR authority | All roles exist | Attempt decision as each | Only authorised HR succeeds (AC-02) |
| DEC-01-TC-03 | Decision provenance | Evidence/versions exist | Decide and inspect | All references/reason retained; evidence unchanged (AC-03) |
| DEC-01-TC-04 | Applicant email | Final decision exists | Inspect email | Outcome and strengths/weaknesses appear without exact/internal data (AC-04) |
