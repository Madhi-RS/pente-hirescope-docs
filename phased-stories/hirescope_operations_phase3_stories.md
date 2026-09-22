# HireScope — Operations, Reporting, and Closure — Phase 3 User Stories

## Scope and rules

The shared opening-summary module is reused by Hiring Manager and HR only as a user-interface concept. Each role receives a different server-authorized data projection. Closed openings are retained, read-only records and are never deleted. `POSITION_FILLED` is downstream headcount accounting, not an online-assessment score or automatic result of `ADVANCED`.

# US-OPS-01: View role-scoped opening progress and operational counts

## User Story

As an authorized Hiring Manager or HR user, I want to view opening progress appropriate to my role, so that I can monitor recruitment without exposing confidential candidate drill-down data.

## Description

The summary includes approved headcount, opening/configuration/FairPack version, and aggregate funnel counts. HR receives authorised candidate/pre-screen/assessment/interview drill-down, exception, decision, and closure controls; Hiring Managers receive their opening aggregates and only explicitly permitted candidate/interview visibility.

## Key Capabilities

- Aggregate counts across application and interview stages.
- Role-specific server projection from a reusable summary surface.
- Version/headcount status visibility.

## Acceptance Criteria

- AC-01: The opening summary shows authorized users approved headcount, active version, and aggregate counts for interest/applied, attended/submitted, live-interview recommendation, not proceeding/rejected, interview completion, final outcomes, and position-filled/remaining headcount.
- AC-02: Hiring Managers can view only own/authorised organisation opening aggregates and explicitly permitted interview information; HR receives authorised candidate/résumé/pre-screen/assessment/interview drill-down, access audit, final decisions, and closure controls.
- AC-03: Shared UI does not imply shared data: API/service authorization prevents unauthorized fields/actions even when a route is known.
- AC-04: Stage counts preserve the defined lifecycle and do not expose applicant scores or identities to roles without permission.
- AC-05: Counts, drill-downs, exports, and shared UI projections never combine or expose data from an organisation outside the viewer's authorised scope.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPS-01-TC-01 | Aggregate funnel | Opening has candidates in each stage | View authorized summary | Required counts/headcount/version are calculated and shown (AC-01) |
| OPS-01-TC-02 | Different role projections | Manager and HR can view same opening | Compare screens/API responses | Manager gets permitted aggregates; HR gets authorized drill-down/control (AC-02) |
| OPS-01-TC-03 | Known-route authorization | Manager knows HR endpoint | Request restricted resource/action | Server refuses it regardless of shared UI (AC-03) |
| OPS-01-TC-04 | Confidential count presentation | Summary contains scored candidates | Inspect manager/applicant/interviewer views | No unauthorized score/identity data appears (AC-04) |
| OPS-01-TC-05 | Cross-organisation reporting isolation | Two organisations have similar openings | Query summary/drill-down/export as one organisation | Only authorised organisation data appears; cross-scope access is denied/audited (AC-05) |

# US-OPS-02: Send operational notifications and handle failures

## User Story

As an authorized operations or HR user, I want timely, safe workflow notifications and failure queues, so that applicants and internal staff can act without technical errors becoming candidate outcomes.

## Description

Notifications cover approvals, HR pre-screen decisions, assessment invitations, interruption/grievance decisions, AI failures, interview assignment/completion, and applicant outcomes. Failure queues preserve source records and support authorized recovery. Applicant outcome email may communicate approved strengths and weaknesses, but never exact AI, interviewer, or aggregate scores.

## Key Capabilities

- Event-driven recipient routing.
- Neutral candidate communications.
- Failure queues with controlled retry/resend/revocation.

## Acceptance Criteria

- AC-01: Approval requests and approval/rejection decisions notify the appropriate approver or Hiring Manager by email (through the product notification channel) with permitted status/comment information.
- AC-02: Only candidates qualified by HR receive the secure assessment invitation; HR non-qualified candidates receive the approved outcome/reason email, and pending candidates receive no assessment link.
- AC-03: Invitation delivery failure and AI failure create alerts/queues while preserving original records and enabling authorized audited resend/revoke/retry.
- AC-04: Interview assignments notify all assigned participants; incomplete required participant results remain visibly incomplete; grievance decisions notify the applicant; outcome emails may state approved strengths/weaknesses but omit exact scores, rankings, internal comments, and confidential rationale.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPS-02-TC-01 | Approval email notifications | Requisition/change is submitted and decided | Process events | Correct internal recipient receives permitted information by email (AC-01) |
| OPS-02-TC-02 | HR-gated pre-screen messaging | Each HR pre-screen outcome exists | Process notifications | Only HR-qualified candidate receives link; non-qualified/pending messages follow policy (AC-02) |
| OPS-02-TC-03 | Failure queue recovery | Simulate delivery and AI failure | Inspect queue and recover | Records persist; authorized recovery action is available/audited (AC-03) |
| OPS-02-TC-04 | Grievance/interview/outcome notices | Grievance, multi-participant interview, and final decision exist | Decide grievance; assign, partially submit, then decide | Applicant/participants are notified; incomplete is visible; strengths/weaknesses omit exact/internal data (AC-04) |

# US-OPS-03: Maintain a complete hiring audit trail

## User Story

As an authorized auditor or HR user, I want an append-only record of material hiring actions and protected-data access, so that the process is explainable and reviewable.

## Description

Audit spans requisitions, versions, sources, eligibility, invitations, protected résumé upload/access, assessment consent/submission, AI results/retries, HR decisions, interview evidence, headcount, closure, and allowed/denied confidential access. It records actor, action, target, time, and required reason/reference where the flow requires one.

## Key Capabilities

- Cross-domain audit lookup.
- Immutable evidence references rather than data overwrites.
- Protected-access audit.

## Acceptance Criteria

- AC-01: Requisition/opening creation, submission, approval/rejection, version activation, publication, change, and closure are audited.
- AC-02: Interest/intake, source correction, candidate/résumé collection and access, résumé AI extraction/matching/discrepancy/result versions, HR pre-screen decision, invitation, readiness/consent, assessment save/submission, recording versions/active selection, interruption/grievance/second-attempt decision, transcript correction, assessment AI result/retry/comment/selection, and HR live-progression decision are audited.
- AC-03: Multi-interviewer assignment/access/draft/individual submission/completion, aggregation policy/version/use, final HR decision, applicant communication, position-filled event, and headcount calculation are audited.
- AC-04: Allowed and denied confidential-data access attempts are audited, and audit history does not overwrite the immutable source result it describes.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPS-03-TC-01 | Opening lifecycle audit | Requisition/opening action history exists | Inspect audit | Every listed opening action is traceable (AC-01) |
| OPS-03-TC-02 | Intake/assessment audit | Candidate traverses intake and assessment | Inspect audit | Each specified event and evidence reference is retained (AC-02) |
| OPS-03-TC-03 | Interview/decision audit | Interview and final decision occur | Inspect audit | Assignment through headcount calculation is traceable (AC-03) |
| OPS-03-TC-04 | Access and immutability audit | User makes allowed/denied protected access | Inspect history and underlying result | Both attempts appear; historical audit did not mutate source result (AC-04) |

# US-OPS-04: Record filled positions and close an opening without deletion

## User Story

As HR or an authorized owner, I want to account for filled positions and close an opening deliberately, so that headcount is accurate and completed recruitment records remain available.

## Description

The opening calculates remaining headcount from approved requisition positions and active position-filled records. `ADVANCED` and onboarding start do not fill or close an opening automatically. HR/authorized owner records position fill and explicitly closes normally filled or exception/early-closed openings.

## Key Capabilities

- Separate `POSITION_FILLED` record and correction history.
- Approved/filled/remaining headcount calculation.
- Read-only `CLOSED` record with retention.

## Acceptance Criteria

- AC-01: A position-filled event is recorded separately from `ADVANCED`, with linked application where applicable, actor, date, reason/reference, and correction history.
- AC-02: Remaining positions equal approved position count minus filled-position count; starting onboarding or advancing one candidate does not automatically fill/close a multi-position opening.
- AC-03: HR/authorized owner can close when remaining count is zero; early closure with remaining headcount requires an authorized exception reason and audit.
- AC-04: A closed opening is visibly tagged `CLOSED`, read-only, cannot be deleted, and retains requisition, versions, applications, résumés, assessment, interview, decision, position-fill, and audit records under retention policy.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| OPS-04-TC-01 | Separate position-filled record | Candidate is `ADVANCED` | Inspect then create fill event | Advance alone does not fill; fill record contains required metadata/history (AC-01) |
| OPS-04-TC-02 | Headcount calculation | Multi-position opening with fills/onboarding | Calculate counts | Remaining count is correct; onboarding/advance do not auto-close (AC-02) |
| OPS-04-TC-03 | Normal/early closure authority | Zero-remaining and nonzero openings exist | Close as authorized; try early close without/with reason | Normal close succeeds; early close requires authorized reason/audit (AC-03) |
| OPS-04-TC-04 | Retained closed record | Opening is closed | Attempt edit/delete; inspect linked history | `CLOSED` tag/read-only state remains and history is retained (AC-04) |
