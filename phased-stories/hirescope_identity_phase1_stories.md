# HireScope — Identity and Access — Phase 1 User Stories

## Scope and rules

This phase establishes account, organisation, invitation, and role boundaries needed before a person can act in HireScope. `C-07` is an opaque per-opening application/packet ID used as a stable workflow reference; it is not an account ID.

Authoritative behavior: `website-user-flow.md` sections 2, 7, and 8; `PROJECT_WORKFLOW_QA.md` sections 2, 7, 8, and 13. Authentication mechanics remain a documented decision in [REQUIREMENTS_GAPS.md](../REQUIREMENTS_GAPS.md).

## Roles and permissions

| Role | Permitted in this phase | Prohibited in this phase |
|---|---|---|
| Applicant | Register interest, verify/sign in, redeem only their invitation, manage own profile/application | Redeem another person's invite; see internal scores or rationale |
| Hiring Manager | Sign in to own organisation's authorised requisitions/openings | Approve their own requisition/change; access HR review evidence without explicit permission |
| HR/Authorized Approver | Sign in to authorized queues and applicant records needed for eligibility | Alter AI scores; access records outside authorization scope |
| Assigned Interviewer | Sign in to assigned interview events only | Change candidate status or access unrelated applications |

# US-IAM-01: Register interest for a specific opening

## User Story

As an applicant, I want to register interest in an active opening, so that HireScope can evaluate whether I may be invited to the assessment.

## Description

The public interest step collects the minimum data only: first name, last name, email, selected opening, and application source. It displays the approved eligibility criteria and requires an explicit acknowledgement. It creates an application interest record, not assessment access.

## Key Capabilities

- Show only an active opening and its approved eligibility-criteria version.
- Capture one of the supported sources: LinkedIn, Naukri, employee referral, direct registration, HR invitation, or other.
- Store the acknowledgement timestamp and criteria version; do not collect Aadhaar in the default MVP.

## Acceptance Criteria

- AC-01: The form requires first name, last name, email, opening, and source before it creates `INTEREST_REGISTERED`.
- AC-02: The opening's eligibility criteria, including applicable experience and salary range, are displayed before submission and require explicit confirmation.
- AC-03: The system stores source type, optional source reference, criteria version, confirmation time, and first-touch time with the interest record.
- AC-04: The default form does not request Aadhaar or expose an assessment link.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| IAM-01-TC-01 | Required minimum details | Active opening exists | Submit with each required field in turn missing, then complete form | Missing field is rejected; complete form creates `INTEREST_REGISTERED` (AC-01) |
| IAM-01-TC-02 | Criteria acknowledgement | Opening has versioned criteria | Attempt submit without, then with acknowledgement | Submission is blocked until acknowledgement; version/time are retained (AC-02) |
| IAM-01-TC-03 | Source traceability | Interest form is available | Submit a referral with reference | Source and first-touch metadata are available for audit (AC-03) |
| IAM-01-TC-04 | Minimum-data guardrail | Interest form is available | Inspect and submit form | Aadhaar and assessment credential are absent (AC-04) |

# US-IAM-02: Redeem a secure assessment invitation and complete profile

## User Story

As a qualified applicant, I want to verify my identity and redeem my invitation, so that only I can continue the application for the intended opening and FairPack version.

## Description

An authorised HR qualification decision creates a unique, time-limited assessment invitation bound to the verified email, organisation, application, opening, opening-configuration version, and FairPack version. Candidate details and résumé were collected before AI/HR pre-screening. On valid redemption, the applicant signs in or verifies an account and proceeds to the configured readiness page.

## Key Capabilities

- Issue, deliver, expire, revoke, resend, recover, and audit invitations.
- Refuse a token used by a different authenticated account.
- Assign an opaque per-opening packet ID for stable assessment/result references without using the global account ID.

## Acceptance Criteria

- AC-01: Only an HR-qualified application receives a unique expiring invitation bound to verified email, organisation, application, opening, opening-configuration version, and FairPack version.
- AC-02: A token cannot be redeemed by another account, after expiry, or after revocation; the failed event is audited.
- AC-03: A valid redemption supports account verification/sign-in, retains the previously collected application/résumé association, and routes to the pinned readiness page without recollecting or bypassing intake.
- AC-04: Issue, delivery, expiry, resend, revocation, redemption, and identity-mismatch events are auditable; resend/recovery is controlled.
- AC-05: The application receives an opaque per-opening packet ID and never uses the global account ID as the assessment workflow key.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| IAM-02-TC-01 | HR-qualified invitation | Authorised HR set `QUALIFIED_FOR_ASSESSMENT` | Trigger invitation | Unique, organisation/application/configuration-bound expiring invitation is issued (AC-01) |
| IAM-02-TC-02 | Misuse protection | Invite exists | Redeem from another account, expired token, and revoked token | Each redemption is refused and audited (AC-02) |
| IAM-02-TC-03 | Valid invited activation | Valid invite and verified email; intake complete | Redeem and sign in | Existing application/résumé link remains and pinned readiness opens (AC-03) |
| IAM-02-TC-04 | Invitation event history | Invite lifecycle has activity | View audit history | All specified lifecycle events appear; controlled resend/recovery is available (AC-04) |
| IAM-02-TC-05 | Application packet identifier | Invited application exists | Start readiness/assessment and inspect result references | Packet ID, not global account ID, is used (AC-05) |

# US-IAM-03: Enforce role-scoped sign-in and protected entry

## User Story

As a HireScope user, I want to sign in to only the areas appropriate to my assigned role, so that confidential hiring data and decisions are protected.

## Description

HireScope supports Applicant, Hiring Manager, HR/Authorized Approver, and Assigned Interviewer access across multiple organisations. Every workforce identity is scoped to one or more explicitly authorised organisations. A shared opening-summary screen may be reused, but every server-side projection and action is role-, organisation-, ownership-, application-, and assignment-scoped.

## Key Capabilities

- Authenticate users and establish the applicable role/session.
- Limit resources by ownership, authorization scope, or interview assignment.
- Log denied confidential access and permit sign-out/session end.

## Acceptance Criteria

- AC-01: An authenticated user is admitted only to routes and actions permitted for their role and scope.
- AC-02: Applicants can access only their own application/status and cannot see exact AI scores, interviewer scores, rankings, reviewer/interviewer comments, or internal rationale.
- AC-03: Hiring Managers cannot approve their own requisitions/changes, publish without approval, or access HR candidate-review evidence outside explicit assigned permission.
- AC-04: Interviewers can access only assigned interview events and cannot change eligibility, final status, or AI results.
- AC-05: HR access is limited to authorized scope; allowed and denied confidential access attempts are audited.
- AC-06: A workforce user cannot discover, read, change, aggregate, or associate requisitions, openings, configurations, applications, résumés, assessments, interviews, decisions, or audit data belonging to an organisation outside their explicit scope.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| IAM-03-TC-01 | Route/action authorization | One account for each role | Attempt each role's permitted and unpermitted entry points | Only permitted entries/actions succeed (AC-01) |
| IAM-03-TC-02 | Applicant confidentiality | Applicant has submitted application | Open internal score/ranking/review URLs | Access is denied; applicant sees no internal decision data (AC-02) |
| IAM-03-TC-03 | Manager separation of duties | Hiring Manager owns pending item | Attempt self-approval, pre-approval publish, and unauthorised HR review | All are refused (AC-03) |
| IAM-03-TC-04 | Assignment-scoped interviewer | Interviewer is assigned to one event | Open assigned and unrelated events; attempt status/AI edit | Only assigned event opens; prohibited changes fail (AC-04) |
| IAM-03-TC-05 | HR audit | HR user makes allowed and denied access attempts | Inspect audit record | Scope enforcement and both attempts are recorded (AC-05) |
| IAM-03-TC-06 | Organisation isolation | Equivalent records exist in two organisations | Query/read/change/cross-associate the other organisation's records | Every operation is denied without data leakage and audited (AC-06) |
