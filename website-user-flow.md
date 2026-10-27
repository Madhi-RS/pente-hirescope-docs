# HireScope Website User Flow

**Status:** Cross-role flow decisions reconciled through 2026-10-12, including the organization-scoped Super Admin ownership, onboarding, and Organization Admin appointment lifecycle.  
**Updated:** 2026-10-12  
**Architecture reference:** [System_Architecture.md](./System_Architecture.md)  
**Schema reference:** [pente_db_diagram.dbml](./pente_db_diagram.dbml)

> **Decisions recorded 2026-10-08 (Hiring Manager finalised):** the Hiring Manager configures the **stage → section → question hierarchy and per-stage weightage on the requisition**, before HR review; HR approval then lets that manager create the Job Opening, which carries the approved configuration. Stage weights total 100% but **rank only — never a composite score or an automatic threshold decision**. Blind review applies to everyone including the Hiring Manager (résumé + "distance traveled" context stay hidden until a stage's scores lock). AI evaluation is a **single 0–100 headline plus seven per-area 1–4 scores** (Task, Thinking, Follow-through, Learning, Judgment, People, Integrity). Voice integrity (intro/reading baselines, listen-don't-verdict flag, typed accommodation) is integrated. The Hiring Manager cannot edit any evaluation result and may add only a distinct **Hiring Manager Comment**. Sections AX–BK below are the Hiring Manager lifecycle.
>
> **Decisions recorded 2026-10-08 and reconciled 2026-10-12 (Interviewer finalised):** the Interviewer is a **job-opening-scoped participant** (reusing the existing `users`/Keycloak identity and `organization_member_roles` — never a separate interviewer account table and never a separate requisition-participant table). Being an interviewer for one opening grants access only to that organisation, opening, and assigned Live Interview stage(s). The Hiring Manager configures the Live Interview requirements on the requisition, then enters the required interviewers' names and email addresses while creating the opening. HR reviews and approves that panel package. Approval sends each interviewer a one-time Keycloak invitation from the verified organisation sender with the HR Reviewer as `Reply-To`; the invitee must sign in or register within 24 hours. HR approval authorises—but does not perform—publication: the Hiring Manager who created the opening remains the actor who publishes it. HR alone marks an applicant **Eligible for Live Interview** and schedules the date/time. Interviewers submit immutable evaluations, see completion status but never another interviewer's scores/comments, and never see AI scores or recommendations. Sections BP–CS below are the Interviewer lifecycle.
>
> **Decisions recorded 2026-10-09 (Applicant finalised):** an Applicant is a **job-application-level** identity — never an organisation member — associated to a job opening only through `job_applications`. Multi-organisation access uses **one Keycloak realm per organisation** (`master`/`onboarding`/`pente`/`careplus`) with **no public organisation directory** and **no cross-realm SSO**: the workspace URL (for example `pente.hirescope.com`) selects the organisation, and logging into one realm never silently authenticates the user in another. Canonical identity is the **verified email address**, with explicit identity-linking so the same person can hold different roles across organisations without becoming a duplicate user. Applying for a job **never grants organisation membership**, and an organisation ID supplied by the frontend is **never trusted without server-side validation**. The pre-screening/application form is **generated dynamically from the requisition's stage → section → question configuration**, never hardcoded. AI evaluation is immutable and advisory; the HR Reviewer owns every progression decision. **Multiple Live Interview stages are supported** (for example Live Interview 1 with panel A, Live Interview 2 with panel B), each with its own assigned interviewers, criteria, and completion/advancement conditions — "Live Interview" is never assumed to be a single stage. **HR Interview is a distinct configurable stage**, never merged with the HR review/decision activity that follows any stage. After the final interview, the authorised HR decision-maker records **final selection**, which transitions the application to **onboarding initiated** and consumes a headcount slot transactionally (target vs selected vs onboarding-in-progress vs onboarded vs remaining; over-fill is prevented). Sections CT–DM below are the Applicant lifecycle.

## A. Platform Admin bootstrap

1. During setup/deployment, the single Platform Admin identity is provisioned in the platform Keycloak realm.
2. Initial credentials are generated or set securely in Keycloak; no password is stored in PostgreSQL.
3. The public landing page presents **Login** and **Organization / User Registration** only. It never presents Platform Admin registration.
4. Platform Admin selects Login and Keycloak authenticates the identity.
5. If Keycloak requires an initial password change, it completes that action and returns the authenticated user to HireScope.
6. The backend validates the token and Platform Admin authority, then loads the Platform Admin dashboard.
7. Invalid credentials show an authentication error and leave the user at login.

Initial public registration uses the Keycloak **onboarding realm**. It creates a normal `users` record with `is_assigned = false`; it does not create a Platform Admin, organisation, or organisation membership. The `master` realm contains the Platform Admin, and each approved organisation has its own tenant realm (for example, `pente` and `careplus`).

## B. Organization onboarding

1. An Organization Admin requester registers/authenticates in the Keycloak onboarding realm; the existing `users` record starts with `is_assigned = false`.
2. The requester submits organisation information, organisation/admin proof, and a comment/reason for the request.
3. The backend creates `organization_onboarding_requests` with `PENDING` status. This record has no `organization_id`, because the workspace does not yet exist.
4. Platform Admin opens **Organization Registrations → Pending** and reviews the requester, proof, comment, and status.
5. Platform Admin chooses Approve or Reject.

### Rejection

1. Platform Admin records a rejection reason.
2. The request becomes `REJECTED`; the request and decision remain available as history.
3. The notification service sends a rejection email to the requester from the configured Platform Admin email identity.
4. No organisation, Keycloak tenant realm, membership, or role is created.

### Approval

1. The backend validates the pending request and approval authorization.
2. It creates the `organizations` record and organisation identifier.
3. It creates/configures the organisation Keycloak realm through the Keycloak Admin API and stores its identifier in `organizations.keycloak_realm`.
4. It creates/links the initial Organization Admin identity in that tenant realm, transactionally updates the application's Keycloak reference, creates an active `organization_memberships` row and one `ORGANIZATION_ADMIN` role with `job_opening_id = NULL`, and sets `users.is_assigned = true`.
5. The request becomes `APPROVED`, references the newly created organisation, and the workspace becomes `ACTIVE`.
6. The notification service sends an approval/onboarding email to the requester from the configured Platform Admin email identity.
7. The Organization Admin can log in to its organisation workspace.

## C. Existing workspace monitoring

1. Platform Admin opens **Workspaces** and sees onboarded organisations, lifecycle status (`ACTIVE`/`SUSPENDED`), Keycloak realm, creation date, membership count, and **View Workspace**. A suspend/activate toggle is shown beside the organisation.
2. Platform Admin opens a workspace in read-only monitoring mode.
3. The screen shows organisation ID/name, Keycloak realm, lifecycle status, creation date, total/active/pending members, and **View Users**.
4. Platform Admin opens Workspace Users to view member identity/contact data, membership status, Organization Admin status, registration/update dates, job-specific assignment summary, and role-verification status.
5. Platform Admin opens a user detail screen to inspect membership, Organization Admin flag, job opening association, current job-specific roles, proof/comment, and verification status.
6. Platform Admin can monitor these records but cannot assign Organization Admin, Hiring Manager, HR Reviewer, or Interviewer roles from these screens.

## D. Job-specific role verification and assignment

1. A public registrant who is not an Applicant enters through a known organisation workspace, requests an organisation-level role, uploads proof, and adds a comment. There is no public organisation directory.
2. After email verification, the backend creates a `PENDING_VERIFICATION` organisation membership and a `PENDING` `role_verification_requests` record.
3. An HR Reviewer request has no Job Opening. An initial Hiring Manager request also has no Job Opening: approval grants organisation-level `REQUISITION_CREATOR` authority so the user can raise a requisition before an opening exists. Interviewer access does not originate from a public role request; it originates from the Hiring Manager's panel nomination on a draft opening and HR's approval of that nomination.
4. The Organization Admin reviews the supporting information and approves the submitted role/scope or rejects it with a required reason.
5. A rejected requester can submit corrected proof/comment; this creates a new pending request linked to the rejected request, retaining history.
6. When a Hiring Manager's requisition is approved by HR and that manager creates its Job Opening, the service creates their job-specific Hiring Manager assignment for that opening.
7. Platform Admin can see the request and resulting assignment in monitoring views only.

`ORGANIZATION_ADMIN`, `HR_REVIEWER`, and `REQUISITION_CREATOR` are organisation-scoped and therefore have no job opening (`job_opening_id = NULL`). When an authorised requisition creator creates an opening from their HR-approved requisition, the service adds that person's job-opening-specific `HIRING_MANAGER` assignment. `INTERVIEWER` always requires a job opening and an HR-approved panel nomination. There is one HR Reviewer per organisation, subject to the same single-active-role constraint as the Organization Admin. An organisation can have many requisition creators/Hiring Managers, and one user can hold many assignments across openings and organisations.

## E. Applicant workflow visibility

An Applicant is a **job-application-level** identity, associated to a job opening only through `job_applications`; the organisation is derived from that opening (never from a client-supplied organisation ID). Applicants are **not organisation members** and never appear in the confirmed-members list.

1. The Applicant resolves the organisation from the **workspace URL** (for example `pente.hirescope.com`), authenticates in that organisation's Keycloak realm, and sees only that organisation's published openings.
2. The Applicant browses active openings, opens a job, and submits an application through a **configurable pre-screening form** generated from the requisition's stage → section → question configuration.
3. The Applicant's own application, current stage, history, HR communications, and pending actions are visible from the Applicant dashboard ("My Applications").
4. Organisation users (HR Reviewer, Hiring Manager, Interviewer) conduct the recruitment workflow within their workspace; the HR Reviewer owns every progression decision.
5. The complete Applicant lifecycle — multi-organisation login, application, pre-screening, online assessment, one or more Live Interview stages, HR Interview, final selection, and onboarding — is specified in sections CT–DM.

## F. Platform Admin screens and navigation

~~~text
Platform Admin
├─ Dashboard
├─ Organization Registrations
│  ├─ Pending
│  ├─ Approved
│  └─ Rejected
├─ Workspaces
│  └─ Workspace Details
│     └─ Users
│        └─ User Details / Role Verification Details
├─ Profile / Account
│  └─ Change Password (Keycloak)
└─ Logout
~~~

| Screen | Purpose and permitted action |
|---|---|
| Landing page | Branding plus Login and Organization/User Registration. |
| Login | Keycloak-backed login, recovery, and error state; no Platform Admin self-registration. |
| Dashboard | Summary counts, onboarded workspaces, and pending organisation registrations. |
| Registration details | Read submitted organisation/requester/proof/comment; approve or reject with a reason and email outcome. |
| Workspace details | Read-only organisation, realm, lifecycle, and member summary; navigate to users. |
| Workspace users | Read-only user, membership, Organization Admin, job-role, job-opening, and verification summary. |
| User / verification details | Read-only profile, role assignment/request proof, comment, and verification status. |
| Profile / password | Redirect to Keycloak-supported password change/recovery. |

## G. Logout

1. Platform Admin selects Logout from global navigation.
2. The application session is invalidated.
3. Keycloak end-session/logout is called.
4. The user is redirected to the landing page or Login screen.

## H. Guardrails

- A pending onboarding request never requires an organisation identifier. It is a business lifecycle record, not a separate registration-user table.
- The existing `users.is_assigned` flag is `false` in the onboarding realm and becomes `true` only with organisation membership; `organization_memberships` remains the authoritative relationship.
- Platform Admin has no normal role-assignment, job-management, candidate-management, interview-management, or recruitment-decision action.
- Keycloak owns credentials and session authentication; PostgreSQL owns application data, workflow, tenancy, memberships, and role scopes.
- Approval/rejection notifications are service-owned email outcomes, not manual-contact instructions.

## I. Organization Admin registration and pending state

An Organization Admin initially registers through the public HireScope onboarding flow, before any organization workspace or organization Keycloak realm exists. The registering user is therefore not yet an active member of an organization, and the registration enters a pending state.

1. From the landing page, the user selects **Registration**.
2. The user selects/indicates **Organization Admin registration**.
3. The user enters personal information.
4. The user enters organization information.
5. The user uploads the required organization/admin proof document.
6. The user adds a registration comment.
7. The user submits.

The registration is held in the Keycloak **onboarding** realm with the existing `users` record at `is_assigned = false`. The backend creates an `organization_onboarding_requests` record with `PENDING` status and no `organization_id`, because the workspace does not yet exist. There is no separate Organization Admin table and no separate pending-registration user table: Keycloak owns the identity, and `organization_onboarding_requests` is the pre-organization business record.

The registration remains pending until the Platform Admin reviews and approves it (see Organization onboarding (B) and the handoff below).

## J. Platform Admin to Organization Admin handoff

Approval is performed by the Platform Admin (see Organization onboarding (B)). From the Organization Admin's perspective, the handoff is:

1. Platform Admin reviews the pending registration (requester, proof, comment).
2. Platform Admin approves the organization.
3. The backend creates the `organizations` record.
4. The backend creates/configures the organization Keycloak realm (for example `pente`) and stores its identifier.
5. The pending user is associated with the organization: the identity is created/linked in the organization realm, an active `organization_memberships` row and one `ORGANIZATION_ADMIN` role (with `job_opening_id = NULL`) are created, and `users.is_assigned` is set to `true`.
6. The request becomes `APPROVED` and the workspace becomes active.
7. The notification service sends the verification/onboarding email.

Example transition:

~~~text
Pending User → Pente Organization → Pente Keycloak Realm → Organization Admin → Active Workspace Access
~~~

This handoff is distinct from Platform Admin's own authentication: Platform Admin authenticates in the `master` realm, while the Organization Admin will authenticate in the newly created organization realm.

## K. Organization Admin onboarding email and first login

After the handoff, the Organization Admin receives an email containing:

- Login ID
- A single-use, expiring Keycloak activation link
- Organization/workspace information
- Login/access instructions

The first-login flow is:

1. The Organization Admin opens HireScope and selects Login.
2. The application redirects to the organization's Keycloak realm (for example `pente`).
3. The single-use Keycloak action link verifies the invitation and lets the user set a password for this organisation realm. No password is sent by email.
4. Keycloak validates the newly established credentials and completes any other required first-login actions.
5. Authentication succeeds and the Organization Admin enters the Organization Admin workspace.

All password creation and later password changes are handled entirely by Keycloak; PostgreSQL never stores credentials.

## L. Organization Admin workspace and tenancy

After login, the Organization Admin enters the workspace belonging to their organization (for example `Pente`). The workspace displays:

- Organization name
- Organization Admin identity
- Organization context
- Navigation relevant to organization administration

All organization context is tenant-scoped. An Organization Admin authenticating in the `pente` realm sees only Pente data and must not see or reach another organization's workspace (for example `careplus`).

## M. Organization Admin dashboard

The workspace includes an organization-level dashboard with two member views.

### Pending / unverified users

Users who have registered/requested participation but whose organization/job-specific assignment has not yet been confirmed. Display:

- User name
- Email
- Registration date
- Requested role
- Requested Job Opening, if applicable
- Supporting document
- Comment
- Verification/assignment status
- View User action
- Verify/Assign action

These are the pending `role_verification_requests` described in Job-specific role verification and assignment (D), viewed from the Organization Admin's side. A request for `ORGANIZATION_ADMIN` or `HR_REVIEWER` has no Job Opening; a request for a job-specific role carries its Job Opening.

### Confirmed users

Users whose organization/job-specific assignment has already been verified. Display:

- User name
- Email
- Membership status
- Assigned role
- Job Opening, if applicable
- Status
- View User action

These correspond to confirmed `organization_member_roles` assignments (see Confirmed users and View User (O)).

## N. User verification and role assignment

When an Organization Admin opens a pending user, the flow is:

1. View User.
2. Review user details.
3. Review the submitted document.
4. Review the user comment.
5. Review the requested Job Opening (if any).
6. Review the requested role (job-specific for Hiring Manager / Interviewer, or Organization Admin / HR Reviewer at organization level).
7. Decide whether the assignment is valid.
8. If approved, assign the appropriate role; if rejected, record the rejection reason.
9. Confirm or reject the user.

On rejection the request becomes `REJECTED` (reason required) and the notification service emails the requester from the Organization Admin's configured email identity. On approval the user moves to the confirmed users list. The Organization Admin is the decision-maker; the user does not contact anyone else for verification.

### Role assignment model

**Organization Admin** and **HR Reviewer** are organisation-level authorities; there is one HR Reviewer per organisation. An approved requisition creator receives the organisation-level `REQUISITION_CREATOR` capability without a Job Opening. When that person creates an opening from their own HR-approved requisition, the service creates the opening-specific `HIRING_MANAGER` assignment. An **Interviewer** is always job-opening-specific and must originate from an HR-approved panel nomination.

When the Organization Admin verifies a public staff request, they approve or reject the exact requested organisation-level authority; the UI must not substitute a role. Job-opening-specific assignments are created by their owning workflow, for example:

- Approved requisition creator creates Job Opening: Software Engineer → service adds Hiring Manager assignment.
- Hiring Manager nominates a panel and HR approves it → service adds Interviewer assignments.

The job opening is required for an Interviewer assignment and for a Hiring Manager's operational opening assignment, but not for initial `REQUISITION_CREATOR` authority. The Organization Admin does not bypass the requisition/opening workflow by manually creating either job-specific assignment.

The Organization Admin and the HR Reviewer do not require a Job Opening:

- Organization: Pente, User: Jane, Role: Organization Admin, Job Opening: NULL
- Organization: Pente, User: Anita, Role: HR Reviewer, Job Opening: NULL

are valid organization-level assignments.

## O. Confirmed users and View User

After verification/assignment, the user moves from the pending/unverified list to the confirmed users list. A rejected request is retained in the rejected view with its rejection reason.

~~~text
Pending Users → Organization Admin reviews → Assignment confirmed → Confirmed Users
Pending Users → Organization Admin reviews → Rejected (reason required) → Rejected view (history)
~~~

The Confirmed Users screen shows: name, email, membership status, organization-level status (if applicable), job-specific role, Job Opening, assigned date, and a View User action. "Assigned date" is the assignment's creation date on `organization_member_roles`.

### View User

When the Organization Admin selects View User, a detailed page shows:

**User Details** — name, email, phone, registration date, membership status.

**Organization Details** — organization, organization membership status, Organization Admin status if applicable.

**Job Assignment Details** — for job-specific members: Job Opening, job-specific role, assignment status, assignment date; for Organization Admin: Job Opening = Not Applicable.

**Supporting Verification** — where applicable: uploaded document, user comment, verification status.

The page distinguishes organization-level status from job-specific assignment. A user can participate in more than one Job Opening (for example User A as Hiring Manager on Job Opening 1 and Interviewer on Job Opening 2). The existing `organization_member_roles` model already supports one membership holding multiple job-specific roles across multiple openings, with a unique (membership, role, job opening) constraint.

### Additional role assignment

After a user is confirmed, further assignments reuse the same identity and membership but follow the authority of the relevant workflow:

1. The Organization Admin may approve organisation-level `REQUISITION_CREATOR` authority after verification.
2. Creating an opening from the user's own HR-approved requisition adds that user's job-specific `HIRING_MANAGER` assignment.
3. While creating the opening, that Hiring Manager enters each required interviewer's name and email for the applicable Live Interview stage.
4. HR reviews the opening and panel package. Approval creates the job-specific `INTERVIEWER` assignments and sends invitations.
5. The Organization Admin may view and later deactivate assignments but cannot silently bypass HR panel approval with a manual Add Role action.

Example:

- Existing: User Arun → HR Reviewer (organization-level, Job Opening: Not Applicable).
- Panel nomination: Interviewer → Job Opening 002 → HR approval.
- Result: Arun holds HR Reviewer (organization) **and** Interviewer → Job 002.

The approved assignment is a new `organization_member_roles` row for the same membership and job opening. No second registration, second Keycloak identity, or second membership is created.

### Members with multiple roles and the role switcher

A confirmed member may hold several job-specific roles across one or more Job Openings:

- Arun Kumar → HR Reviewer (organization)
- Arun Kumar → Interviewer — Job 001
- Arun Kumar → Interviewer — Job 002

The members list shows this as a role/assignment summary. For a member with multiple active assignments, the Organization Admin (and the member, after login) sees a **role switcher/toggle** listing the member's assignments. The default view shows the first/primary assignment.

Switching the role changes only the workspace/UI context for that same member — it does not create a new user account, a new registration, or a new membership. The switcher is a context switch over the member's existing `organization_member_roles` rows.

## P. Organization Admin job opening management

The Organization Admin monitors the organization's Job Openings:

- View Job Openings, applicants/applications, assigned participants, applicant counts, filled positions, and remaining positions.
- Creation and configuration are deferred to the approved Hiring Manager flow; Organization Admin has no create/edit control in this phase.
- View assigned Hiring Managers.
- View the organization's HR Reviewer.
- View assigned Interviewers.
- View applicants/applications for the organization's Job Openings.

Applicants are not organization members (see Applicant workflow visibility (E)); the Organization Admin sees applicant information through the relevant Job Opening/recruitment workflow, and each applicant is associated to the opening through `job_applications`.

The older create/edit bullets above are superseded by the 2026-10-06 decision: Job Opening creation/configuration belongs to the approved Hiring Manager and is deliberately deferred to that actor's later flow.

## Q. Organization Admin navigation

~~~text
Organization Admin
├─ Dashboard
├─ Users / Members
│  ├─ Pending / To Be Verified
│  ├─ Confirmed Users
│  └─ User Details
├─ Job Openings
│  ├─ Job Opening List
│  ├─ Job Opening Details
│  └─ Assigned Members
└─ Profile / Account
   └─ Change Password (Keycloak)
~~~

Logout is available globally.

## R. Organization Admin logout

1. The Organization Admin selects Logout from global navigation.
2. The application session is terminated.
3. The Keycloak session is terminated.
4. The user is redirected to the login/landing page.

The user is not authenticated after logout.

## S. Organization Admin lifecycle (end to end)

~~~text
Registration
→ Pending Registration
→ Platform Admin Approval
→ Organization Created
→ Keycloak Organization Realm Created
→ User Associated With Organization
→ Email Sent
→ Organization Admin Login
→ Initial Password
→ Password Change
→ Organization Workspace
→ Manage Pending Users
→ Review User Details
→ Review Documents/Comments
→ Approve Organization Admin / HR Reviewer / REQUISITION_CREATOR authority as applicable
→ Confirm User
→ Confirmed Users
→ View User
→ View Job Opening Assignments
→ Monitor Job Openings and workflow-created assignments
→ View Applicants/Applications
→ Logout
~~~

## T. Organization Admin guardrails

- Organization Admin authority is organization-scoped. An Organization Admin must never see or act on another organization's workspace.
- Organization Admin has no platform-level responsibilities: it cannot create other organizations, create/configure Keycloak realms, approve organization onboarding, manage Platform Admin, or access another organization's workspace.
- Credentials and sessions are Keycloak-owned; PostgreSQL never stores a password or an initial password.
- A pending user is represented with existing status attributes — `users.is_assigned`, `organization_memberships.membership_status`, and `role_verification_requests.verification_status` — not a new pending-user table or a new `assigned` column.
- The existing design keeps one active Organization Admin per organization and one HR Reviewer per organization; the `ORGANIZATION_ADMIN` and `HR_REVIEWER` assignments (job opening NULL) are subject to that constraint.
- A workspace has a lifecycle status `ACTIVE` or `SUSPENDED`. Suspension is recorded from the workspace list and is enforced server-side: the backend/Keycloak rejects login for a suspended workspace; the UI toggle only records the state.
- Applicants are associated with the organization only through their job application; they are not organization members and never appear in the confirmed-members list.

## U. HR Reviewer role model and scope

The HR Reviewer is an **organization-level** role: there is exactly one HR Reviewer per organization (`job_opening_id = NULL`), operating across all of the organization's job openings. The same identity can still hold additional job-specific roles, for example:

~~~text
User A
→ Organization: Pente
→ Role: HR Reviewer (organization-level, job_opening_id = NULL)
→ Job Opening: 001 → Interviewer
→ Job Opening: 002 → Interviewer
~~~

The user registers once per organisation realm. Additional job-specific roles reuse that realm identity: a Hiring Manager assignment is created when an authorised requisition creator creates an opening from their approved requisition, and an Interviewer assignment is created after HR approves the Hiring Manager's panel nomination. There is no separate HR Reviewer user entity.

Role hierarchy (used consistently throughout):

~~~text
Platform level        → Platform Admin
Organization level    → Organization Admin, HR Reviewer
Job-opening level     → Hiring Manager, Interviewer
Job-application level → Applicant
~~~

Hiring Manager and Interviewer are never permanent organization-wide roles; they always carry a `job_opening_id`. Organization Admin and HR Reviewer are organization-level (`job_opening_id = NULL`); each organization has at most one of each.

## V. HR Reviewer registration

An HR Reviewer belongs to an existing organization (for example Pente already exists). The user selects **I am not registering as an applicant** during public registration, chooses the existing organization workspace, then submits proof and a comment:

1. From **Pente Workspace → Registration**, the user enters personal details.
2. The user provides the required proof/document.
3. The user requests role = **HR Reviewer** (organization-level; no Job Opening is selected).
4. The user adds a comment, for example: "I am the HR Reviewer for this organization. Please verify the attached document and assign me as the HR Reviewer."
5. The user submits.

After email verification, the backend creates a `PENDING_VERIFICATION` membership and a `PENDING` role-verification request. The user has no HR Reviewer authority until Organization Admin approval. A rejection shows its reason and supports corrected-proof resubmission through a new request linked to the rejected request.

## W. HR Reviewer login

After the Organization Admin assigns the HR Reviewer role:

1. The user receives the login/access information and a single-use, expiring Keycloak action link.
2. Authentication is handled by Keycloak.
3. A new realm user uses the link to establish a password; an existing realm user signs in with their existing password.
4. No plaintext or default password is sent by email or stored by HireScope.
5. The user is redirected to the HR Reviewer workspace in the organization's Keycloak realm.

PostgreSQL must not store the password or the initial password. Keycloak owns credentials and sessions.

Flow:

~~~text
Login/invitation link → Organization Keycloak realm → Establish password or authenticate → HR Reviewer workspace
~~~

## X. HR Reviewer workspace and tenancy

After login the HR Reviewer sees the organization workspace (for example Pente), which shows:

- User details.
- Organization/workspace details.
- Current active role.
- Job Openings in the organization requiring HR Reviewer action (the HR Reviewer is organization-level and sees all of the organization's openings).
- Requisitions requiring HR Reviewer action.
- Approved Job Openings.
- Applicant evaluation workflow.

The HR Reviewer only sees the organization's authorized data. Tenant scope is enforced per request; an HR Reviewer in the Pente realm must never reach CarePlus data.

## Y. HR Reviewer dashboard

The HR Reviewer dashboard contains at least:

### A. Pending Job Requisitions
Requisitions requiring HR Reviewer approval (including resubmitted requisitions, ordered first).

### B. Rejected / Returned Requisitions
Requisitions the HR Reviewer rejected or returned for modification.

### C. Approved Job Openings
Job Openings created/activated after requisition approval.

### D. Opening / Panel Approval
Draft openings submitted by their Hiring Manager, including the approved requisition version and the names/emails nominated for every Live Interview stage. HR approves for publication or returns the package with a reason. Approval sends interviewer invitations but leaves the final Publish action with the Hiring Manager.

### E. Applicant Evaluation
Applicants currently requiring HR Reviewer action at their current stage.

### F. Position Fulfillment
For example: Requested positions 2, Filled 1, Remaining 1. This is visible wherever Job Opening position status is relevant (see section AG).

## Z. Hiring Manager requisition creation and submission

A Hiring Manager creates a Job Requisition and submits it to the HR Reviewer.

Flow:

~~~text
Hiring Manager → Create Requisition → Submit → HR Reviewer
~~~

The HR Reviewer sees it in the pending requisition list.

Cross-reference: the finalized phased stories define the requisition lifecycle as `REQUISITION_DRAFT → REQUISITION_PENDING_APPROVAL → REQUISITION_APPROVED`, with `REQUISITION_REJECTED` for returns; each submission locks an immutable version. See section AW for reconciliation notes with the opening-package approval gate.

## AA. Requisition information

The HR Reviewer can view for each requisition:

- Job Requisition ID.
- Job Opening / Job Role (title).
- Hiring Manager.
- Organization.
- Number of positions.
- Salary range / expected salary.
- Expected applicant experience.
- Required skills.
- Job description.
- Job requirements.
- Location / work mode (where applicable).
- Employment type (where applicable).
- Requisition creation date.
- Last submitted date.
- Current status.
- Configured interview/assessment stages.
- Stage sections/configuration.
- Previous review comments.
- Previous rejection reason (if resubmitted).

Only standard requisition information that already exists in the HireScope requirements is included; no unnecessary fields are invented. (The requisition entity and its fields are business requirements needing database support — see section AW.)

## AB. Requisition stages and sections (configuration as data)

Stages are configurable during requisition creation. Predefined options include (but are not limited to):

- Pre-screening
- Online Assessment
- Live Interview
- HR Interview
- Other (the Hiring Manager enters a custom stage name)

These are not hardcoded as the only possible stages. The requisition stores the configured stage sequence. Example:

~~~text
Stage 1: Pre-screening
Stage 2: Online Assessment
Stage 3: Live Interview
Stage 4: HR Interview
~~~

Another requisition can have a different sequence.

Each stage can contain configurable sections. For example, Online Assessment may contain:

- Self Introduction / Recorded Self Interview
- Read a Passage
- Answer Questions
- Additional assessment sections

The Hiring Manager configures these sections when creating the requisition. The HR Reviewer reviews the configured stages and sections before approving. Stage configuration is treated as data, not hardcoded.

For **Other / Custom Stage**, the Hiring Manager must configure:

- Stage name and applicant-facing description.
- Mode (`ONLINE`, `IN_PERSON`, `REMOTE_MEETING`, or another configured mode).
- Who conducts the stage (role or named participant rule).
- Who evaluates it and the number of required evaluators.
- Sections/questions or evaluation criteria.
- Total marks, passing mark, and stage weightage.
- Whether scheduling, applicant submission, evaluator submission, or both are required.
- Completion condition and notification instructions.

The passing mark is decision support, not an automatic progression rule. The evaluator records the result and the HR Reviewer still makes the audited advance/hold/reject decision.

## AC. Requisition approval, rejection, and resubmission

**Approved decision:** HR approval changes the requisition to `REQUISITION_APPROVED`; it does not create or publish a Job Opening. The same approved requisition creator then creates one draft Job Opening. The opening carries the HR-approved stage/section/question/scoring configuration; the Hiring Manager adds opening-level operational information and enters the required interviewers' names and email addresses. HR reviews this opening/panel package and either returns it or marks it `APPROVED_TO_PUBLISH`. The Hiring Manager—not HR—performs the final Publish action.

The HR Reviewer opens a requisition and can:

### Approve

~~~text
HR Reviewer → Review Requisition → Approve → Requisition becomes Approved → Hiring Manager creates draft Job Opening from the approved configuration → names interviewers → submits opening/panel package to HR → HR approves/returns → Hiring Manager publishes
~~~

### Reject / Return for Modification

~~~text
HR Reviewer → Review Requisition → Add Comment → Reject/Return → Requisition status becomes Rejected/Needs Revision → Hiring Manager sees the comment → edits the requisition → resubmits → requisition returns to HR Reviewer
~~~

Example comment: "Please revise the expected salary range to a lower range before resubmitting."

The comment is stored as part of the requisition review history (append-only, audited) and is never lost across revisions.

## AD. Resubmitted requisition ordering

A resubmitted requisition appears above newly submitted requisitions in the pending list. The list shows:

- Requisition ID.
- Job role.
- Hiring Manager.
- Previous review date.
- Resubmission date.
- Current status.
- Priority/ordering indicator where appropriate.

Ordering rule:

1. Resubmitted / previously reviewed requisitions requiring action.
2. Newly submitted requisitions.

Within each group, use the most recent relevant date. Previous review history is never lost.

## AE. Approved job openings (HR Reviewer view)

An opening appears in HR's work queues as soon as its approved Hiring Manager creates the draft from the approved requisition. It appears in the **Approved Job Openings** list only after HR approves its opening/panel package. The opening is organisation-owned and job-specific to its creator.

Once the HR Reviewer approves the requisition:

~~~text
Requisition APPROVED → Hiring Manager creates DRAFT opening + names panel → PENDING_OPENING_APPROVAL → HR APPROVED_TO_PUBLISH → Hiring Manager PUBLISHES → PUBLISHED/ACTIVE
~~~

The HR Reviewer's **Approved Job Openings** list shows each Job Opening with:

- Job Opening ID.
- Job role/title.
- Hiring Manager.
- Number of positions.
- Filled positions.
- Remaining positions.
- Current status.
- Created date.
- Applicants count.
- View Job Opening.

The HR dashboard also contains a separate **Opening / Panel Approval** queue showing the approved requisition version, opening-level fields, every Live Interview stage, required interviewer count, nominated names/emails, membership/registration state, and **Approve for Publication** / **Return to Hiring Manager** actions. HR approval sends the interviewer invitations; it does not publish the opening.

## AF. View job opening

When the HR Reviewer clicks **View Job Opening**, show:

### Job details

- Job Opening ID.
- Job title.
- Job description.
- Hiring Manager.
- Number of positions.
- Filled positions.
- Remaining positions.
- Salary range.
- Expected experience.
- Skills/requirements.
- Location / work mode (where applicable).
- Employment type (where applicable).

### Configured interview process

Display the configured stages in sequence, each with its configured sections. Example:

~~~text
1. Pre-screening
2. Online Assessment
3. Live Interview
4. HR Interview
~~~

## AG. Position tracking and fulfillment

For every Job Opening, track headcount as **target, committed capacity, selected/onboarding, filled, and available to select**:

~~~text
Target:     2
Reserved:   1   (final selection recorded; committed capacity)
Onboarding: 1   (onboarding in progress)
Onboarded:  0
Available:  1   (target − reserved)
~~~

`Reserved` is committed capacity: it includes selected/onboarding reservations and completed fills. `Filled` is reported separately and increases only when onboarding completes. The system never uses `target − filled` as available selection capacity. `Reserved` cannot exceed `Target`, and `Available` cannot be negative.

Final selection and onboarding are **deliberate, separately recorded events** — not automatic consequences of advancing one candidate. A position is reserved when an authorised decision-maker records final selection; onboarding completion increments `Filled` without reopening capacity. Selection is refused transactionally when no capacity is available. A closed opening remains a retained read-only record. See sections DG and DH and AW.2 for the database support needed.

## AH. View applicants

From a Job Opening, **View Applicants** lists applicants with:

- Applicant ID.
- Applicant name.
- Email.
- Application date.
- Current stage.
- Application status.
- Relevant basic education information.
- Action menu (three-dot `⋮`).

Actions: **View Applicant** (other actions only if already supported by the existing workflow).

## AI. View applicant

**View Applicant** shows the applicant profile:

- Applicant ID.
- Name.
- Email.
- Phone (if available).
- College/institution.
- Degree/course.
- CGPA.
- Graduation information (where available).
- Resume.
- Relevant application information.

Do not expose unnecessary personal data.

## AJ. Applicant current stage

The applicant detail screen clearly shows the **Current Stage**. Example:

~~~text
Applicant 1001 → Job Opening 1001 → Current Stage: Pre-screening
~~~

As the applicant progresses:

~~~text
Pre-screening → Online Assessment → Live Interview → HR Interview
~~~

The actual sequence comes from the Job Opening's configured stage list; it is never hardcoded.

## AK. AI pre-screening evaluation (immutable)

For the pre-screening stage, AI evaluates the applicant on, for example:

- Resume relevance.
- Eligibility.
- Required skills.
- Experience.
- Job/category match.
- Other configured screening criteria.

The AI produces an evaluation/result.

CRITICAL RULE: **AI-generated evaluation results are immutable.**

The HR Reviewer can:

- View the AI result.
- Review supporting information (resume, eligibility, criteria, discrepancies).
- Add a human review comment.
- Approve the applicant for the next stage.
- Reject the applicant.

The HR Reviewer cannot edit the AI-generated evaluation itself; there is no Edit AI Result action.

### Blind-review visibility matrix

| Event | HR Reviewer | Hiring Manager | Interviewer | Organization / Platform Admin |
|---|---|---|---|---|
| Application submitted; AI/evaluator score not locked | Résumé and distance-travelled context hidden | Hidden | Hidden | Hidden |
| AI pre-screen result persisted and locked | Résumé/context visible with the immutable AI result for HR decision | Still hidden | Hidden | Hidden |
| HR pre-screen decision persisted | Visible | Visible read-only for the assigned opening | Hidden | Hidden |
| Live/custom human-evaluated stage | Sees authorised evidence and completed evaluations | Sees only evidence already released by the rule above | Sees only stage-required applicant information; no résumé/distance-travelled context, AI result, or prior evaluator scores | Hidden |
| Application manually closed by its Hiring Manager | Full historical assessment, immutable AI result, interviewer evaluations, HR decision and its confidential note during retention/legal hold | Read-only historical assessment, AI result, interviewer evaluations, and applicant-visible HR decision for the assigned opening; confidential HR note is hidden | No historical assessment, AI result, peer evaluation, or HR decision access; may retain only their own submitted-evaluation receipt where policy requires | SLA/audit metadata only; no recruitment evidence |

“Scores lock” means the AI result or required evaluator submission for that stage has been persisted immutably. A user must never receive hidden fields in the API response; this is not merely a visual mask.

## AL. Pre-screening decision

After the AI pre-screen result is persisted and locked, the HR Reviewer reviews it together with the now-released applicant details, résumé, and eligibility information, then:

### Approve

~~~text
Applicant → Approved for next configured stage → applicant notification email sent → applicant logs in → next stage becomes available
~~~

Existing finalized status: `QUALIFIED_FOR_ASSESSMENT`.

### Reject

~~~text
Applicant → Rejected → rejection comment stored → email sent → applicant can log in and see they are no longer active in the process
~~~

Existing finalized status: `NOT_CURRENTLY_QUALIFIED` or `REJECTED`.

The email body contains the HR Reviewer's decision/comment. The notification service sends it from the verified organisation address with the responsible HR Reviewer as `Reply-To`; no individual mailbox credentials are stored.

## AM. Rejected applicant experience

If the applicant logs in after rejection, show:

- Job Opening.
- Application status.
- Rejected / not-progressing status.
- HR Reviewer communication/comment where appropriate.

Do not display future assessment/interview screens. The applicant cannot continue the assessment after rejection.

## AN. Next-stage access

If the HR Reviewer approves the applicant:

~~~text
Applicant → Next configured stage becomes accessible
~~~

Example: Pre-screening approved → Online Assessment available. The applicant completes the configured sections. The exact next stage depends on the requisition's configured stage sequence.

## AO. Online assessment AI evaluation and consolidated result

If Online Assessment is configured, the applicant completes the configured sections; AI evaluates the submitted responses and produces a consolidated evaluation/result. Again, **AI evaluation is immutable**.

The HR Reviewer can view the consolidated AI result, review evidence/submissions, add a human comment, approve progression, or reject — but cannot modify the AI score/evaluation.

**View Consolidated Result** shows:

- AI evaluation.
- AI score/result where configured.
- Evaluation criteria.
- Section-level results where available.
- Overall consolidated result.
- Evaluation timestamp.
- Model/evaluation metadata where supported by the existing architecture.

AI-generated fields are not editable. The HR Reviewer adds a separate human review comment. The UI clearly distinguishes **AI Evaluation** from **HR Reviewer Comment**.

Existing finalized status for the decision: `LIVE_INTERVIEW_RECOMMENDED`, `LIVE_INTERVIEW_HOLD`, or `NOT_PROCEEDING` (see section AR).

## AP. Live interview

If the configured next stage is a Live Interview:

- The interview is conducted through the organization's configured external meeting process (for example Google Meet). The meeting itself is not assumed to happen inside HireScope unless the architecture says so.
- After the interview, the person who conducted/evaluated it logs in and submits the interview evaluation.
- The HR Reviewer reviews the evaluation and evaluator comments, then approves progression or rejects.

Finalized flow: the Hiring Manager nominates interviewers by name/email on the draft opening; HR approves or returns the opening/panel package; approval sends one-time 24-hour invitations; the Hiring Manager publishes the approved opening. Each interviewer submits an independent immutable rubric, and the stage completes when all required participants submit (`INTERVIEW_COMPLETED`). The full lifecycle is specified in BP–CS.

## AQ. HR interview

If HR Interview is configured as a stage, it follows the configured workflow. After completion:

- The evaluator submits the evaluation.
- The HR Reviewer reviews the evaluation.
- The HR Reviewer decides progression or rejection.

HR Interview is not assumed to be the final stage; the final stage comes from the requisition configuration.

## AR. Stage completion model

For every configured stage:

~~~text
Stage → Applicant completes stage → Evaluation generated/submitted → HR Reviewer reviews → Approve progression OR Reject → Applicant notified → Next configured stage enabled
~~~

- **AI-evaluated stages**: AI evaluation (immutable) → HR Reviewer review → human comment → decision.
- **Human-evaluated stages**: evaluator submits evaluation → HR Reviewer review → human comment/decision.
- **Custom stages**: use the configured mode, conductor, evaluator count, total marks, passing mark, evidence/submission requirements, and completion condition. Internally each custom stage instance follows `NOT_STARTED → READY/SCHEDULED → IN_PROGRESS → SUBMITTED/AWAITING_EVALUATIONS → EVALUATION_LOCKED → UNDER_HR_REVIEW → ADVANCED/HOLD/REJECTED`. The applicant-facing label comes from the custom stage configuration.

Decision-vocabulary mapping (task action → existing finalized status):

| Stage | Approve (progress) | Reject / not proceeding |
|---|---|---|
| Pre-screening | `QUALIFIED_FOR_ASSESSMENT` | `NOT_CURRENTLY_QUALIFIED` / `REJECTED` |
| Online assessment → live | `LIVE_INTERVIEW_RECOMMENDED` | `NOT_PROCEEDING` (or `LIVE_INTERVIEW_HOLD`) |
| Final interview (all configured stages complete) | `ADVANCED` | `REJECTED` (or `HOLD`) |
| Final selection (after `ADVANCED`) | `SELECTED` → `ONBOARDING_PENDING` → `ONBOARDING_COMPLETED` → `POSITION_FILLED` | `NOT_SELECTED` / `REJECTED` |

The HR Reviewer's decision at each stage is recorded with the reviewer, reason/comment, selected AI result version (where applicable), and timestamp. AI never changes application state or makes a decision.

`ADVANCED` means the applicant passed the final configured interview/evaluation stage; it does **not** yet select the applicant or fill a position. **Final selection** (`SELECTED`) is a distinct, authorised HR action that reserves a headcount slot. HR records external offer acceptance before `ONBOARDING_PENDING`; only `ONBOARDING_COMPLETED` creates `POSITION_FILLED`.

## AS. Email notifications

Where the workflow requires applicant communication:

### Approved for next stage

Send email to the Applicant communicating:

- Job Opening.
- Next stage.
- Required action.
- Relevant instructions.

### Rejected

Send email to the Applicant. The body contains the HR Reviewer's decision/comment where appropriate.

Emails are sent from a verified organisation address with the responsible HR Reviewer as `Reply-To`. Individual email credentials/passwords are never stored in PostgreSQL. The pre-screening qualification email includes the secure assessment link; a rejected/non-qualified applicant receives no assessment link.

## AT. HR Reviewer role switching

A user may hold, for example:

- HR Reviewer (organization-level).
- Interviewer → Job Opening 001.
- Interviewer → Job Opening 002.

The organization member UI lets the user switch between their assigned roles. Default: the first/primary assigned role. Selecting HR Reviewer shows the HR Reviewer workspace/functions (organization-wide); selecting Interviewer shows the Interviewer workspace/functions.

This is a context switch for the same user, not another account, registration, or membership.

## AU. HR Reviewer guardrails (what the HR Reviewer must not do)

The HR Reviewer must not have:

- Platform Admin capabilities.
- Organization creation.
- Keycloak realm creation.
- Cross-organization access.
- Organization Admin assignment.
- Arbitrary role assignment.
- Editing AI evaluations.
- Editing applicant submissions.

The HR Reviewer reviews requisitions, evaluates applicant progression, and reviews AI/human evaluations — the responsibilities defined above, scoped to the organization's job openings.

## AV. Complete HR Reviewer lifecycle (end to end)

~~~text
Register in Organization Workspace
→ Default Applicant/Unverified State
→ Upload Proof
→ Add Comment
→ Organization Admin Reviews
→ Organization Admin Assigns HR Reviewer
→ Login
→ Keycloak Authentication
→ Initial Password
→ Set New Password
→ HR Reviewer Workspace
→ View Requisitions
→ Review Requisition
→ Approve / Reject
→ If rejected: Hiring Manager edits → Resubmit → Requisition returns to top of HR Review queue
→ If approved: Job Opening
→ View Job Opening
→ View Applicants
→ View Applicant
→ Review AI Pre-screening
→ Approve / Reject
→ Applicant Notification
→ Applicant Next Stage
→ AI / Human Evaluation
→ HR Reviewer Review
→ Approve / Reject
→ Continue configured stages
→ Position Fulfillment Tracking
→ Final recruitment outcome
→ Logout
~~~

Logout terminates the application session and the Keycloak session and redirects to the login/landing page (same as Organization Admin logout, section R).

## AW. Contradictions and business requirements needing later database support

This section records (1) contradictions that require a product decision rather than silent resolution, and (2) business requirements this workflow implies that the supplied schema does not yet model. No tables/columns are added here; these are inputs for the later DBML/architecture update.

### AW.1 Contradictions / open reconciliation points

1. **Resolved — requisition approval → opening creation → publication.** HR approval changes the requisition to `REQUISITION_APPROVED`. The approved requisition creator creates a draft opening that carries the approved configuration, enters the stage-specific interviewer names/emails, and submits the opening/panel package to HR. HR approval changes it to `APPROVED_TO_PUBLISH` and sends the invitations; the Hiring Manager performs the final Publish action.

2. **Resolved — where stages/sections/questions/weights are configured.** The Hiring Manager configures the full stage → section → question hierarchy and per-stage weightage on the requisition (sections AZ–BB). The opening carries the approved configuration; assessment-delivery and pre-screen refinements happen without changing the approved sequence once applicants enter it (section BM). The DBML note marking `requisition_stages`/`requisition_stage_sections` as "legacy/proposed" is reversed.

3. **Resolved — Hiring Manager scope before an opening exists.** An Organization Admin can approve organisation-level `REQUISITION_CREATOR` authority (`job_opening_id = NULL`). That capability permits requisition creation only. Creating an opening from the creator's HR-approved requisition creates the job-specific `HIRING_MANAGER` assignment.

4. **"HR Interview" stage.** HR Interview remains a valid configurable stage (sections AB, AQ). The final HR disposition (`ADVANCED`/`HOLD`/`REJECTED`) after `INTERVIEW_COMPLETED` remains the closing decision; no candidate-performed stage is assumed beyond what the manager configured.

5. **Per-stage "Approve/Reject" vs finalized decision vocabulary.** This workflow uses "Approve/Reject" per stage; the finalized stories use richer per-stage statuses (section AR maps them). This is a vocabulary mapping, not a behavioral conflict, but the mapping must be applied consistently.

6. **Resolved — "consolidated" AI result vs the no-composite rule.** The agreed evaluation shape is a single headline 0–100 score plus seven per-area 1–4 scores with reasoning (section BJ). "Consolidated" means a consolidated view of the per-area immutable results, not a composite score/decision. Stage weights (section BB) rank only and never produce a hire score or a threshold decision.

7. **AI model/evaluation metadata.** This workflow (section AO) and the finalized stories (CAN-02, REV-01) require model/configuration metadata on AI results. The supplied DBML (`ai_stage_evaluations`) explicitly omits a model-version column.

8. **Resolved — email sender identity.** Notifications are service-owned and sent from a verified organisation address; the responsible HR Reviewer is set as `Reply-To`. Individual HR mailbox credentials are never stored.

### AW.2 Business requirements needing later database support

- **Job Requisition** entity with status (`REQUISITION_DRAFT`, `REQUISITION_PENDING_APPROVAL`, `REQUISITION_APPROVED`, `REQUISITION_REJECTED`), required fields, and version/revision history (each submission locks an immutable version; resubmission creates a new version).
- **Requisition review history**: append-only review comments, decisions, previous rejection reasons, review/resubmission dates, and ordering support (resubmitted first).
- **Requisition fields** not currently modelled: salary range, expected applicant experience, required skills, job requirements, location/work mode, employment type (the supplied `job_openings` has only title, description, `required_candidates`).
- **Position fulfillment**: committed capacity, selected/onboarding reservation, `Filled` (from a separate `POSITION_FILLED` record), and `Available to select = Target − Reserved`, all transactionally enforced; retained read-only `CLOSED` state.
- **AI result immutability + model metadata**: immutable AI evaluation rows plus model/configuration metadata.
- **Applicant stage history and notification history**: per-application stage progression and a record of applicant notifications (email outcomes) for audit.
- **Organisation notification identity + HR Reply-To**: verified organisation sender, responsible HR Reviewer reply address, delivery/retry audit; no individual mailbox credentials.
- **Role assignment history**: audited organisation-level authority approval, job-opening-specific Hiring Manager creation, HR-approved Interviewer assignment, activation, expiry, replacement, revocation, and same-realm role switching.
- **Opening approval/publication lifecycle**: `DRAFT → PENDING_OPENING_APPROVAL → RETURNED / APPROVED_TO_PUBLISH → PUBLISHED/ACTIVE → SUSPENDED / CLOSED`, preserving separate HR approval and Hiring Manager publication actors.
- **Interviewer nomination/invitation**: a pending workflow record keyed by opening + Live Interview stage + normalized nominee email, with name, existing-user match, HR decision, one-time action-link token reference, sent/activation/24-hour-expiry timestamps, replacement chain, and audit history. Activation creates/links the membership and role; no placeholder user/role and no plaintext password is stored.
- **Custom-stage execution model**: stage name, mode, conductor, evaluator rule/count, sections/questions/criteria, total marks, passing mark, weightage, schedule/submission requirements, completion condition, and generic stage-instance states.
- **Workflow action requests**: in-product HR request, target requisition/opening, requested action, reason, status, Hiring Manager decision/action, timestamps, and notifications for withdrawal/suspension/reactivation/closure; applicant action deadlines and automatic no-response suspension/release records.
- **Requisition stage → section → question configuration**: ordered stages (with per-stage weight that must total 100%), ordered sections per stage, ordered questions per section, carried into the opening on approval.
- **Five evaluation/decision records, never merged**: AI Evaluation (immutable, headline score + seven areas), Evaluator Evaluation (immutable, per-interviewer), HR Decision, HR Comment, and Hiring Manager Comment.
- **Stage configuration lock/revision**: once an applicant enters a stage, its questions/sections/order/weightage/criteria are locked or revisioned, preserving the configuration used at evaluation time.
- **Blind-review visibility rule**: résumé and "distance traveled" context are hidden from all reviewers until a stage's scores lock (an evaluation-visibility control, not a new table).
- **Interviewer assignment**: interviewers are assigned at job-opening creation via a job-opening-scoped `organization_member_roles` row (`role = INTERVIEWER`); invitation state is carried on the role/verification request, with no separate participant table (sections BP–BQ).
- **Live Interview configuration**: enabled flag, required interviewer count, and evaluation sections with weightage (no preset questions), carried from the requisition into the opening; the interviewers themselves are named at opening creation (sections BR–BS).
- **Live Interview records**: scheduling is stored on the applicant's stage result (`stage_results.scheduled_date`/`scheduled_time`/`status`); per-interviewer evaluations live in a generalized `stage_interviewers` table (overall comment, submitted status, submitted timestamp) with per-section scores reusing `interview_results`/`interview_criteria`, and `UNIQUE (application_id, stage_id, interviewer_membership_id)` enforces one submission per interviewer (sections CI, CL).
- **Canonical identity and cross-realm linking — resolved**: `canonical_people` owns verified unique email and each realm-local `users` identity links to it; the same person may be Applicant in one organisation and staff in another.
- **Multi-stage Live Interview assignment — resolved**: `stage_interviewers` provides stage-scoped panel assignment, attempt number, and one immutable submission per interviewer/attempt.
- **Final selection and onboarding — resolved**: `application_selections` records the decision-maker/timestamp and reservation; `onboarding_records` records HR status updates from the external process; `position_fills` is created only at `ONBOARDING_COMPLETED`.

## AX. Hiring Manager role model and scope

A Hiring Manager is a job-opening-specific role, not a permanent organisation role. Before an opening exists, the person acts through the separate organisation-level `REQUISITION_CREATOR` capability. An organisation can have multiple requisition creators/Hiring Managers, and one user can hold multiple opening assignments. For example:

| User | Organisation | Job Opening | Role |
|---|---|---|---|
| Arun | Pente | 001 | Hiring Manager |
| Arun | Pente | 002 | Hiring Manager |
| Arun | Pente | 001 | Interviewer |
| Arun | ABC Corp | 015 | Applicant |

The UI must therefore distinguish organisation/workspace context from role context. Each assignment is a separate `organization_member_roles` row (or, for the Applicant row, a `job_applications` row) — never a separate user record.

## AY. Hiring Manager registration and login

Registration follows the same path as the HR Reviewer: register in the known organisation workspace → submit proof/comment → Organization Admin verifies → membership created → `REQUISITION_CREATOR` authority assigned → login. The user becomes the job-opening-specific Hiring Manager only when they create an opening from their own HR-approved requisition; one user can therefore have multiple Hiring Manager assignments.

## AZ. Role / workspace switcher

After login, the user sees only the role contexts available inside the currently authenticated organisation realm:

~~~text
Pente realm
 ├── Hiring Manager — JOB-001
 └── Interviewer — JOB-002
~~~

Switching from Pente Hiring Manager → Pente Interviewer changes the sidebar and workflow for that role without leaving the Pente realm. Cross-organisation switching is not offered as an authenticated role toggle. To move from Pente staff access to an Applicant account in ABC Corp, the person logs out of Pente, opens the ABC Corp workspace URL, and authenticates separately against the ABC Corp realm. The verified email may be the same, but the realm-local Keycloak user ID and password may be different; there is no cross-realm SSO.

## BA. Hiring Manager dashboard

When operating as a Hiring Manager, the dashboard shows the Hiring Manager profile, the organisation/workspace, the active role/context, requisitions, job openings, recruitment progress, and pending actions. Summary cards include:

- My Requisitions
- Pending Approval
- Active Job Openings
- Total Applicants
- Applicants in Progress
- Positions Remaining

## BB. Create requisition

Creating a requisition is the Hiring Manager's primary responsibility. The requisition carries the job requirements: Requisition ID, job title/role, description, number of positions, expected salary/salary range, expected experience, required skills, qualifications/requirements, job responsibilities, and the configured recruitment stages, stage sections, questions, and stage scoring/weightage (sections BC–BD).

## BC. Configure stage → section → question

The Hiring Manager configures the stages required for that position; the sequence is never hard-coded. For example:

~~~text
1. Pre-Screening
2. Online Assessment
3. Live Interview
4. HR Interview
~~~

or, for another job, Pre-Screening → Live Interview → HR Interview. The stage types are predefined while presence, ordering, sections, questions, and scoring are job-specific.

The hierarchy is:

~~~text
Job Requisition
    ↓
Stages
    ↓
Sections
    ↓
Questions
~~~

Example:

~~~text
Online Assessment
├── Self Introduction        → Record a 2-minute introduction
├── Reading                  → Read the given passage
└── Technical Questions
    ├── Question 1
    ├── Question 2
    ├── Question 3
    └── Question 4
~~~

## BD. Configure stage scoring / weightage

Different jobs prioritise different stages. For example:

| Stage | Developer weight | Sales Executive weight |
|---|---|---|
| Pre-screening | 20% | 20% |
| Online Assessment | 25% | 15% |
| Live Interview | 40% | 25% |
| HR Interview | 15% | 40% |

The Hiring Manager configures each stage's weight; the system validates the agreed rule (for percentages, the total must equal 100%). Weights rank and guide prioritisation only — they are never combined into a single hire score and never trigger an automatic advance, hold, or rejection.

Each scored stage—including a custom stage—may also define total marks and a passing mark. These values label the result as meeting/not meeting the configured benchmark, but they never change application state automatically; HR owns progression.

## BE. Submit requisition → HR review → return / resubmit

~~~text
Draft → Pending HR Review → HR Reviewer
~~~

The Hiring Manager submits; the HR Reviewer controls approval. The Hiring Manager does not directly create the Job Opening. If HR returns/rejects it:

~~~text
Pending → Returned / Rejected → Hiring Manager sees HR comment → Edit → Resubmit → HR Review
~~~

The requisition keeps its revision history rather than overwriting the previous submission, and a resubmitted requisition appears above newly submitted requisitions in the HR review queue.

## BF. Hiring Manager requisition list

| Requisition | Role | Positions | Status | Last Updated | Action |
|---|---|---|---|---|---|
| REQ-001 | React Developer | 2 | Pending | Oct 8 | View |
| REQ-002 | Backend Developer | 1 | Approved | Oct 7 | View |
| REQ-003 | QA Engineer | 2 | Returned | Oct 6 | Edit |
| REQ-004 | UI Designer | 1 | Withdrawn | Oct 4 | View |

Requisition states are `DRAFT → PENDING_HR_REVIEW → RETURNED → RESUBMITTED → APPROVED → WITHDRAWN`. There is no Delete. Only the Hiring Manager who created it can execute withdrawal; HR may raise an in-product withdrawal request, which the Hiring Manager accepts or rejects with an audited reason and notification. Suspension/reactivation belongs to the Job Opening lifecycle, not the requisition lifecycle.

## BG. Job openings (Hiring Manager view)

Once HR approves a requisition, the Hiring Manager creates the draft Job Opening from that approved version, enters the stage-specific panel names/emails, submits it for HR opening/panel approval, and publishes it after approval. The Hiring Manager's Job Openings screen lists only the openings they are authorised to manage, for example:

~~~text
JOB-001 · Senior React Developer · Positions 2 · Filled 1 · Remaining 1 · Applicants 24 · Active
~~~

## BH. View job opening

**Job information** — Job Opening ID, title, description, positions requested/filled/remaining, salary range, experience, skills, requirements, Hiring Manager, creation/approval dates, current status.

**Recruitment configuration** — the approved stages, stage order, sections, questions, stage weights, custom-stage execution rules, and stage-specific interviewer nominations.

Actions depend on state: **Edit Draft** | **Submit Opening/Panel for HR Approval** | **Publish** (only when `APPROVED_TO_PUBLISH`) | **View Stages** | **View Applicants** | **Suspend/Reactivate/Close** where authorised.

## BI. View stages

~~~text
Job Opening: JOB-001
1. Pre-Screening        20%
2. Online Assessment    25%
3. Live Interview       40%
4. HR Interview         15%
~~~

Selecting a stage opens its configuration and applicant progress.

## BJ. View applicants and applicant detail

The applicants table shows:

| Applicant | Current Stage | Status | Overall Score | Action |
|---|---|---|---|---|
| Applicant 001 | Online Assessment | In Progress | 78 | View |
| Applicant 002 | Live Interview | Completed | 86 | View |
| Applicant 003 | Pre-Screening | Rejected | 42 | View |

The applicant detail view shows Applicant ID, name, email, college, CGPA, résumé, job opening, current stage, application status, stage history, AI evaluation, stage scores, evaluator scores, applicant responses, questions presented, and (where permitted) human evaluator comments.

**Read-only rule:** the Hiring Manager cannot modify evaluation results — AI score, evaluator score, AI recommendation, and applicant responses are all read-only. The Hiring Manager may add a **Hiring Manager Comment** (for example "Strong Java and backend experience; good fit for our current development requirements") but cannot change the AI score (82), the evaluator score (88), the AI recommendation, or the applicant response.

The AI evaluation shape is a single headline 0–100 score plus seven per-area 1–4 scores (Task, Thinking, Follow-through, Learning, Judgment, People, Integrity) with reasoning.

## BK. Stage-specific applicant review

For each stage the Hiring Manager inspects the relevant evidence but cannot alter AI or HR evaluation:

- **Pre-screening** — résumé, AI screening result/score/reasoning, eligibility, skills matching, HR decision and comment, plus the Hiring Manager's own comment.
- **Online Assessment** — questions, applicant responses, AI evaluation and score, consolidated result, HR decision, plus the Hiring Manager comment.
- **Live Interview** — interview information, questions, responses, interviewer/evaluator score and evaluation, HR decision, plus the Hiring Manager comment.
- **HR Interview** — the evaluation and the Hiring Manager's own observation, without altering the evaluator's score.

## BL. HR Reviewer vs Hiring Manager capability boundary

| Capability | HR Reviewer | Hiring Manager |
|---|---|---|
| Create requisition | No | Yes |
| Edit own returned requisition | No | Yes |
| Approve requisition | Yes | No |
| Reject / return requisition | Yes | No |
| Configure stages | Review | Yes |
| Configure sections / questions | Review | Yes |
| Configure stage weightage | Review | Yes |
| View applicants | Yes | Yes |
| View AI evaluation | Yes | Yes |
| Edit AI evaluation | No | No |
| View applicant responses | Depending on stage/business rule | Yes |
| View evaluator scores | Yes | Yes |
| Edit evaluator scores | No | No |
| Add own comment | Yes | Yes |
| Make applicant stage decision | Yes | No |
| Request requisition withdrawal | Yes | No |
| Execute own requisition withdrawal | No | Yes |
| Create draft Job Opening from approved requisition | No | Yes |
| Approve opening + interviewer package for publication | Yes | No |
| Publish approved Job Opening | No | Yes |
| Suspend/reactivate/close Job Opening | Request | Yes (creator only) |

## BM. Stage configuration lock after applicants enter

Before applicants enter a stage the configuration can be edited. After the stage has started, its questions, sections, stage order, scoring/weightage, and evaluation criteria are locked or moved to a new revision, so historical evaluations stay consistent. The system preserves the configuration actually used when each applicant was evaluated.

## BN. Hiring Manager comments vs official decisions

The following records are distinct and never merged into one generic comment field: AI Evaluation, Evaluator Evaluation, HR Decision, HR Comment, and Hiring Manager Comment. A Hiring Manager observation ("candidate has strong React skills, looks suitable for our team") is never mistaken for an official workflow decision ("approved for next stage"), which is owned by the HR Reviewer.

## BO. Hiring Manager lifecycle (end to end)

~~~text
Registration
→ Organization Admin Verification
→ Hiring Manager Role Assignment
→ Login
→ Select Organization + Hiring Manager Context
→ Hiring Manager Dashboard
    ├── My Requisitions
    │    → Create Requisition → Configure Stages → Configure Sections
    │      → Configure Questions → Configure Stage Weightage → Submit
    │      → HR Review
    │         ├── Returned → Edit → Resubmit
    │         └── Approved → Create Draft Job Opening
    │              → Enter interviewer names/emails → Submit opening/panel package
    │              → HR approves/returns
    │              → Approved to Publish → Hiring Manager publishes
    └── My Job Openings
         → View Job Opening
            ├── View Stages (configuration, sections, questions, weightage)
            └── View Applicants → Applicant Detail
                 ├── Resume / Profile
                 ├── Stage History
                 ├── AI Evaluations       [READ ONLY]
                 ├── Applicant Responses  [READ ONLY]
                 ├── Evaluator Scores     [READ ONLY]
                 ├── HR Decisions         [READ ONLY]
                 └── Hiring Manager Comment
~~~

## BP. Interviewer role model and scope

An Interviewer is a **participant assigned to a specific job opening** (at opening creation, after HR approval) — never a permanent organisation-wide role, and never a separate user identity. Being an interviewer for one requisition must NOT automatically grant access to other requisitions, other applicants, other organisations, or other interview processes. Access is scoped to the specific organisation and requisition/job opening for which the interviewer is assigned, and specifically to the **Live Interview** stage.

The Interviewer is represented by the existing `users` (Keycloak) identity plus an `organization_memberships` row and a job-opening-scoped `organization_member_roles` row with `role = INTERVIEWER`. No `InterviewerUser` / `InterviewerAccount` / `InterviewerOrganization` table is introduced. One user can hold many assignments across openings and organisations (for example Interviewer on JOB-001 and Hiring Manager on JOB-002 within Pente, plus an Applicant application in another organisation); each is a distinct role row, never a distinct user.

## BQ. Interviewer assignment model

The Hiring Manager nominates interviewers at **job-opening creation** (not on the requisition) by entering the required names and email addresses for each Live Interview stage. Nominees may be existing realm users or new invitees. The draft opening and panel package is submitted to HR; only HR approval creates/activates the assignments and sends invitations. The data model is:

~~~text
Organization
    ↓
Job Opening (created from the approved requisition)
    ↓
organization_member_roles (role = INTERVIEWER, job_opening_id)
    ↓
Interviewers
~~~

Before an invitee has an organisation-realm account, the nomination is stored as workflow data keyed by opening, Live Interview stage, name, and normalized email; it is not yet a membership or role. For an existing realm user, HR approval links the nomination to the existing membership and creates the job-opening-specific `organization_member_roles` row. For a new invitee, HR approval sends the one-time Keycloak action link; successful activation creates/links the realm user and membership, creates the job-opening role, and links it back to the nomination. No placeholder user, membership, or role row is created merely from an email address.

This pending nomination/invitation record is not a separate Interviewer account and not a requisition participant/evaluation table. The invitation state is `NOMINATED`, `HR_APPROVED`, `INVITED`, `ACTIVATED`, `EXPIRED`, `REPLACED`, or `REVOKED` and is audited with its 24-hour deadline. Applicant-specific evaluator assignment still belongs to `stage_interviewers` only after a valid membership exists.

## BR. Hiring Manager — Live Interview configuration (on the requisition)

While creating the requisition, the Hiring Manager configures the Live Interview stage (section BC lists it among the predefined stage types). The configuration is stored as requisition data and reviewed by HR before approval. The Hiring Manager sets:

- **Live Interview enabled** — e.g. `Live Interview: Enabled`.
- **Required number of interviewers** — e.g. `Required Interviewers: 2`.

Interviewers themselves are **not** named on the requisition. The requisition defines the required count and criteria. Names/emails are entered later by the Hiring Manager on the draft opening and invitations are sent only after HR approves the opening/panel package.

## BS. Live Interview scoring configuration

The Hiring Manager configures the **scoring framework** for the Live Interview while creating the requisition. There are **no fixed interview questions** for the Live Interview — preset questions belong only to the Online Assessment/Online Interview stage. For the Live Interview, the manager configures **evaluation/scoring sections** instead:

~~~text
Live Interview Evaluation

Technical Knowledge       - 40%
Communication Skills      - 20%
Problem Solving           - 20%
Role Fit                  - 20%

Total                     - 100%
~~~

The exact structure follows the existing HireScope scoring model. If that model already supports criteria, sections, weightage, maximum score, and a rating scale, it is reused rather than duplicated. The interviewer must be able to see the configured scoring framework before conducting the interview.

## BT. Interviewer invitation and Hiring Manager publication

Interviewers are nominated when the draft opening is created. HR reviews the nominated names/emails against the required count and stage configuration. HR approval changes the opening to `APPROVED_TO_PUBLISH`, authorises the panel assignments, and sends each nominee an invitation. Existing members can be linked immediately; a new nominee receives the role only after activating their realm identity/membership. HR does not publish; the Hiring Manager who created the opening performs the final Publish action, consistent with the approved mockup.

The invitation is sent from the verified organisation sender with the HR Reviewer as `Reply-To`. It contains a single-use, expiring Keycloak action link and a 24-hour activation deadline, but no password and no applicant-specific information. Example:

~~~text
A new job opening has been created for [Role Name].
You have been listed as an interviewer for the Live Interview stage.

Job Opening: [Job Title]
Job Description: [Description]
Organization: [Organization Name]

You have been approved as an interviewer for this job opening.
Use the secure link below within 24 hours. If you already have an account in this
organisation, sign in. Otherwise, complete registration and create your password.
You will be notified when an applicant becomes eligible for the Live Interview.
~~~

If the invitee has not activated access within 24 hours, the invitation becomes `EXPIRED`, HR and the Hiring Manager are notified, and the Hiring Manager must resend or nominate a replacement for HR approval. Expiry does not silently grant access. The final Publish button is available only in `APPROVED_TO_PUBLISH`; publication itself is always performed by the Hiring Manager.

## BU. Interviewer registration / login

The interviewer authenticates through the existing Keycloak / `users` / organisation-membership / role architecture — no separate authentication mechanism is created.

- **Not yet registered:** Opening/panel package approved → one-time invitation sent → interviewer registers and creates a realm password within 24 hours → assignment becomes active.
- **Already has an account in this realm:** Opening/panel package approved → one-time invitation sent → existing interviewer authenticates within 24 hours → assignment becomes active.

The system preserves the user's existing organisation/member identity throughout.

## BV. Interviewer dashboard

After login, the interviewer sees the requisitions/interviews for which they are assigned:

~~~text
Interviewer Dashboard — My Interviews

Software Engineer — Organization: ABC Technologies
Live Interview · Required Interviewers: 2
Applicants: 1. Candidate A Scheduled · 2. Candidate B Completed · 3. Candidate C Pending
~~~

The interviewer must NOT see unrelated job openings, other organisations, or other applicants.

## BW. Interviewer requisition view

Opening an assigned requisition displays:

- **Job information** — job title, job description, organisation, relevant job requirements, requisition details.
- **Live Interview configuration** — enabled, required number of interviewers, assigned interviewers, evaluation sections, score weightage, rating/score scale, and evaluation instructions.

~~~text
Live Interview — Required Interviewers: 2
Interviewers: ✓ John Smith · ✓ Priya Kumar
Evaluation Criteria: Technical Knowledge 40% · Communication 20% · Problem Solving 20% · Role Fit 20%
~~~

This information is available to the interviewer **before** evaluating applicants.

## BX. Applicant becomes eligible for Live Interview

The applicant does not automatically enter the Live Interview stage — the **HR Reviewer** makes this decision:

~~~text
Applicant → Pre-screening → Online Assessment → HR reviews applicant
  → HR reviews previous-stage information → HR enters HR comment
  → HR selects "Eligible for Live Interview"
~~~

Only the HR Reviewer marks an applicant eligible for the Live Interview.

## BY. HR Live Interview scheduling

When HR selects **Eligible for Live Interview**, the system displays the configured Live Interview information and tells HR the required panel:

~~~text
This Live Interview requires 2 interviewers.
Assigned Interviewers: 1. John Smith · 2. Priya Kumar
~~~

HR then enters the **Interview Date** and **Interview Time** (e.g. `9 October 2026` · `1:00 PM`), reviews a confirmation message, and creates the scheduling record. Scheduling is refused unless: Live Interview is configured for the requisition, the required interviewers exist, the applicant is eligible, a valid date/time is provided, and the applicant has not already completed the Live Interview. The scheduling decision belongs to the HR Reviewer.

## BZ. Applicant notification

After HR submits the schedule, an email is sent to the applicant from the configured identity, generated from the confirmed scheduling information:

~~~text
You have been selected to attend a Live Interview for [Job Title].
Interview Date: 9 October 2026
Interview Time: 1:00 PM
The interview will be conducted by the assigned interview panel.
Please log in to HireScope for further details.
~~~

## CA. Interviewer notification

After HR schedules the applicant's Live Interview, every assigned interviewer receives a notification/email:

~~~text
You have been assigned to conduct a Live Interview.
Job: Software Engineer · Applicant: [Applicant Name]
Interview Date: 9 October 2026 · Interview Time: 1:00 PM
Required Interviewers: 2 · You are one of the assigned interviewers.
Please log in to HireScope to review the evaluation criteria and submit your assessment.
~~~

The notification is sent to **every** assigned interviewer.

## CB. Interviewer applicant access

When an interviewer opens a scheduled applicant, only authorised information is shown:

- **Applicant information** required for the interview.
- **Previous-stage HR comments** (e.g. the pre-screening HR comment and the online-assessment HR comment), plus the current stage (`Eligible for Live Interview`).
- **Interview information** — date, time, and the assigned interviewer list.
- **Live Interview scoring criteria** (sections + weightage).

## CC. Strict information visibility

The interviewer must NOT see any of the following from previous evaluation stages: AI-generated scores, AI recommendations, AI evaluation results, AI confidence scores, AI ranking, AI classification, internal ATS decision data, another interviewer's evaluation/score/comments.

The interviewer MAY see: applicant information required for the interview, job information, the Live Interview configuration, previous-stage HR comments, the Live Interview schedule, other interviewers' **names**, and other interviewers' **completion status**. This separation is critical and enforced server-side (never only in the UI).

## CD. Interviewer evaluation

The interviewer conducts the Live Interview (in or outside the platform, per the existing product design), then opens the applicant's Live Interview evaluation form. The form is generated dynamically from the scoring configuration created by the Hiring Manager:

~~~text
Live Interview Evaluation
Technical Knowledge  Weight 40%  Score: [   ]
Communication Skills Weight 20%  Score: [   ]
Problem Solving      Weight 20%  Score: [   ]
Role Fit             Weight 20%  Score: [   ]
Overall Comment: [............]
[Submit Evaluation]
~~~

The interviewer cannot modify scoring criteria, weightage, interviewer assignment, interview schedule, or applicant eligibility — those are controlled by the appropriate roles.

## CE. Score validation and duplicate prevention

Before accepting an evaluation, the system validates: every required scoring section is scored, scores are within the configured range, required comments are present (where the specification requires them), the interviewer is actually assigned to this requisition/interview, the interviewer has not already submitted, and the Live Interview belongs to the applicant/requisition being accessed. Duplicate submission is rejected (the record enforces one final evaluation per interviewer per live interview).

## CF. Interviewer submission

On submit the interviewer sees `Evaluation submitted successfully.` The evaluation becomes immutable unless the product explicitly supports an authorised edit/reopen mechanism. The interviewer then sees `Your Evaluation: Completed` and cannot edit it.

## CG. Multiple interviewers

The system supports multiple interviewers and tracks each one's status:

~~~text
Required Interviewers: 2
John Smith — Completed · Priya Kumar — Pending
Overall Interview Status: Pending
~~~

Priya must NOT see John's score, rating, or comments — only that John's evaluation is completed.

## CH. When all interviewers submit

Once all required interviewers have submitted, the Live Interview status becomes `Completed`, and the applicant becomes eligible for the next HR review/decision stage. HR is notified that the applicant is ready for review and can then review the collected interviewer evaluations and continue the recruitment decision.

## CI. Interviewer and Live Interview status model

- Individual interviewer evaluation: `PENDING`, `IN_PROGRESS`, `COMPLETED`.
- Overall Live Interview: `SCHEDULED`, `IN_PROGRESS`, `PENDING_EVALUATIONS`, `COMPLETED`, `CANCELLED`.

Existing project status names (e.g. `INTERVIEW_COMPLETED`) are preserved where already defined.

## CJ. Authorization rules (server-side)

An interviewer can access an applicant evaluation only when:

~~~text
Authenticated User AND User is assigned as interviewer
  AND Interviewer belongs to the correct organisation context
  AND Applicant belongs to the assigned requisition
  AND Applicant is eligible/scheduled for Live Interview
~~~

An interviewer receives `403 Forbidden` for an unauthorized applicant or requisition. Unauthorized data is never exposed through API responses. Authorization is enforced on the backend, not only through frontend route protection.

## CK. API requirements

The existing backend conventions are extended (not duplicated) to cover:

- **Requisition / opening** — create/update Live Interview requirements and scoring criteria on the requisition; nominate names/emails per stage on the draft opening; submit the opening/panel package; HR approve/return; Hiring Manager publish.
- **Interviewer** — get assigned requisitions, get assigned applicants/interviews, get interview configuration, get authorized applicant information, get previous HR comments, get evaluation form, submit evaluation, get own evaluation status.
- **HR** — approve/return the opening/panel package, send one-time invitations through the organisation notification service, mark applicant eligible for Live Interview, schedule Live Interview, view interviewer completion status, view submitted interviewer evaluations.

## CL. Database requirements and constraints

The required model is `Job Requisition → Live Interview Configuration → Required Interviewer Count`, `Draft Job Opening → stage-scoped panel nominations (name/email/invitation state) → activated organization_member_roles (INTERVIEWER)`, and `Applicant → Application → stage_results (schedule) → stage_interviewers → interview_results (scores)`. An unregistered nominee cannot be represented by `organization_member_roles`; the pending nomination exists until Keycloak activation creates/links the membership and role. There is **no separate Live Interview table** and no separate Interviewer account. Evaluator assignment/submission state lives in generalized `stage_interviewers`, while per-section scores reuse `interview_results` keyed to `interview_criteria`.

For a specific Live Interview, one interviewer has **at most one final evaluation**: `UNIQUE (application_id, stage_id, interviewer_membership_id)` on `stage_interviewers`. Only interviewers assigned to the opening (via `organization_member_roles`) can create an evaluation.

## CM. Score aggregation

Individual interviewer scores are never exposed to other interviewers. After all required interviewers complete their evaluations, the collected evaluations are made available to the HR Reviewer. If the product already defines an aggregation formula it is used; otherwise no new hiring-decision formula is invented — individual interviewer scores are stored separately and reviewed by HR. Any aggregate, where required, is calculated from the configured scoring model while preserving the individual evaluator records.

## CN. Notification rules and audit

Notifications are emitted for: interviewer assignment (notify interviewer); applicant eligible + interview scheduled (notify applicant and all assigned interviewers); interviewer evaluation submitted (optionally notify the HR workflow where supported); all interviewer evaluations completed (notify HR that the applicant is ready for review). No interviewer is ever notified with another interviewer's score. Where the notification architecture supports it, each notification records recipient, notification type, related requisition/applicant/live interview, sent status, timestamp, and failure status/reason. All important actions are auditable using the existing audit framework: interviewer assigned, invitation sent, applicant marked eligible, Live Interview scheduled/rescheduled, interviewer opened evaluation, interviewer submitted evaluation, Live Interview completed.

## CO. Edge cases

- **Interviewer has not registered** → show the interviewer registration/login flow.
- **Interviewer already has an account** → attach the interviewer assignment to the existing user.
- **Assigned but applicant not yet eligible** → interviewer sees the assigned requisition but cannot access applicant evaluation until the applicant reaches the Live Interview stage.
- **Applicant rejected before Live Interview** → no interviewer evaluation is allowed.
- **Interview cancelled** → evaluation submission disabled.
- **Interview rescheduled** → schedule updated and relevant parties notified.
- **One interviewer submits, another does not** → HR records the valid reason, replaces the missing required interviewer, schedules one HR-approved retake, and requires the replacement evaluation. The system writes the internal retake remark, alerts the Hiring Manager, exposes the event to HR, adds it to the Organisation Admin SLA report, and begins the escalation procedure.
- **One interviewer submits twice** → duplicate submission rejected.
- **Interviewer accesses another interviewer's score** → unauthorized/forbidden; data not exposed.
- **Interviewer accesses an applicant from another requisition** → rejected.
- **Interviewer removed from requisition** → future access revoked; historical evaluations (if already submitted) are never deleted.
- **Removed after assignment but before evaluation** → further evaluation access prevented unless reassigned.
- **Required interviewer count changes after applicants are scheduled** → already-scheduled interviews are not silently invalidated; a transition is defined and enforced using the existing requisition-edit rules.

## CP. Interviewer lifecycle (end to end)

~~~text
Hiring Manager creates requisition
  → Live Interview configured (enabled, required count, evaluation sections + weightage)
  → HR approves requisition → Hiring Manager creates the Job Opening (carries the configuration) and names the interviewers
  → HR approves the opening/panel package → one-time interviewer invitations sent
  → Hiring Manager publishes the approved opening
  → Interviewer registers/logs in (existing Keycloak identity)
  → Interviewer dashboard shows assigned requisitions
  → Applicant progresses (pre-screening → online assessment) → HR marks Eligible for Live Interview
  → HR schedules Live Interview (date/time) → applicant + all interviewers notified
  → Interviewer reviews job info + Live Interview configuration + authorised applicant info + prior HR comments
  → Interviewer submits evaluation (dynamic sections from HM configuration)
  → Interviewer submission becomes immutable → own status Completed
  → All required interviewers submit → Live Interview COMPLETED → HR notified
  → HR reviews the collected evaluations → next-stage decision
~~~

## CQ. Interviewer guardrails (what the interviewer must not do)

The interviewer is an evaluator, not the final decision-maker, and cannot: approve/reject an applicant, mark an applicant selected, move an applicant to a final hiring stage, change applicant eligibility, change HR decisions, change job requisition configuration, modify scoring criteria/weightage, modify interviewer assignment, modify the interview schedule, or view another interviewer's scores/comments or any AI evaluation result. After submission the interviewer's responsibility is complete; HR reviews the consolidated evaluation and makes the next-stage decision.

## CR. Capability boundary (HR Reviewer vs Hiring Manager vs Interviewer)

| Capability | HR Reviewer | Hiring Manager | Interviewer |
|---|---|---|---|
| Create / configure Live Interview on requisition | Review | Yes | No |
| Nominate interviewer names/emails on draft opening | Review | Yes | No |
| Approve/return opening and panel package | Yes | No | No |
| Publish approved opening | No | Yes | No |
| Configure evaluation sections + weightage | Review | Yes | No |
| Mark applicant eligible for Live Interview | Yes | No | No |
| Schedule / reschedule Live Interview | Yes | No | No |
| View interviewer completion status | Yes | Yes (assigned opening) | Assigned interview (status only) |
| View interviewer evaluations | Yes | Yes (read-only) | Own only |
| Submit own interview evaluation | No | No | Assigned interview |
| Edit any evaluation | No | No | No |
| Make applicant stage decision | Yes | No | No |

## CS. Testing requirements

- **Backend unit tests** — interviewer authorization; interviewer assignment; scoring configuration; HR eligibility transition; scheduling; evaluation submission; duplicate-submission prevention; score validation; multi-interviewer completion; unauthorized score access; cross-organisation access; cross-requisition access; interviewer removal; cancelled-interview handling.
- **API/integration tests** — the complete flow: requisition creation → Live Interview configured → HR approves requisition → opening created + interviewers named → HR approves opening → invitation sent → applicant passes previous stages → HR marks eligible → HR schedules → applicant notified → interviewers notified → interviewer logs in → sees scoring framework → sees authorised HR comments → submits → second interviewer submits → Live Interview COMPLETED → HR reviews.
- **Security tests** — interviewer A cannot view interviewer B's score; cannot access an applicant from another requisition/organisation; cannot change scoring weightage; cannot mark an applicant eligible; cannot approve/reject an applicant; cannot submit for an applicant to whom they are not assigned.

## CT. Organization-scoped workspace entry and login (multi-organisation)

There is **no public directory of organisations**. Entry to HireScope is always workspace-scoped: the workspace URL selects the organisation, and login/registration are scoped to that organisation.

~~~text
Applicant opens pente.hirescope.com
→ backend resolves organisation from the workspace identifier (server-side, never from the client)
→ Keycloak redirects to the pente realm
→ applicant authenticates in the pente realm
→ backend resolves the identity's applications/roles scoped to Pente
→ Applicant dashboard (Pente-scoped)
~~~

Rules:

- The workspace URL (subdomain/workspace identifier) selects the organisation. There is no "browse all organisations" landing page and no organisation directory.
- Each organisation authenticates in its **own Keycloak realm**. Logging into `pente` does **not** authenticate the user in `careplus`. The person must log out of the current workspace, open the other workspace URL, and authenticate in that realm. The verified email may be the same, but the realm-local Keycloak user ID and password may differ; there is no cross-realm SSO or cross-organisation role toggle.
- The `organization_id` is derived server-side from the workspace identifier. A client-supplied organisation ID is **never trusted**.
- Login and registration are organisation-scoped: registering at `pente.hirescope.com` registers the user against Pente only.

## CU. Applicant registration and canonical identity

- A person may register as an Applicant at any organisation's workspace; there is no global applicant directory.
- **Canonical identity is the verified email address.** When a verified email is already known to the platform (possibly in another organisation's realm), the backend links the new realm identity to the same canonical person instead of creating a duplicate `users` record, where the identity model supports linking; where linking is not yet supported, that decision is recorded in AW.2 rather than silently assumed.
- One person can hold different roles across organisations (Applicant in Pente, Hiring Manager in CarePlus) without becoming a duplicate user. Each membership/application is a separate relationship over the same canonical identity.
- Applying for a job **never creates an `organization_memberships` row**. Membership is granted only through the verification/assignment flows (sections D, N, O).

## CV. Applicant dashboard — active openings

After login the Applicant dashboard shows:

- **Available openings** — the organisation's published, currently-appliable openings (title, location/work mode, employment type, closing date where configured, View/Apply action).
- **My Applications** — the applicant's applications with current stage, status, and any pending action.
- **Profile / account** — contact details and, where applicable, the applicant's résumé.
- Logout (global).

The dashboard is organisation-scoped: it shows only the organisation whose workspace the applicant is in.

## CW. Job details and publication eligibility

An opening is visible/appliable only when all of the following hold:

- Its requisition is approved and the opening is **published/active** (not draft, suspended, or closed).
- The application window is open (`opening date ≤ now ≤ closing date`, where a window is configured).
- A vacancy remains (if headcount-limited — see DH).
- The applicant has not already submitted an active application for that opening (duplicate prevention).

A closed, expired, withdrawn, or suspended opening renders a terminal "no longer accepting applications" state and never shows an Apply action.

## CX. Apply — configurable pre-screening form

Applying is the **Pre-screening stage's intake**. The form is generated dynamically from the requisition's configured Pre-screening stage → sections → questions — it is never hardcoded. It captures personal/professional identity, education, experience, eligibility questions, and the résumé upload exactly as configured for that requisition.

~~~text
Applicant → Apply → register interest (INTEREST_REGISTERED; application created)
→ APPLICATION_DETAILS_REQUIRED (configured form presented)
→ applicant completes configured sections/questions and uploads résumé
→ RESUME_SUBMITTED
~~~

Rules:

- No organisation membership is created by applying.
- The organisation is derived from the opening, never from a client-supplied value.
- Duplicate application for the same opening is prevented server-side.

## CY. Application submission and confirmation

On submission the applicant sees a confirmation ("application received") and the application enters the HR pre-screen review queue. The confirmation states the next step (HR review) and that the applicant will be notified of the outcome. A confirmation/notification email is sent (see DI). The submission does not advance any stage — the HR Reviewer's pre-screen decision does.

## CZ. Pre-screening pending and HR decision (applicant view)

After submission the dashboard shows the application in pre-screen review. AI pre-screening runs and produces an advisory, immutable result; the HR Reviewer reviews it and decides:

- `QUALIFIED_FOR_ASSESSMENT` — proceed to the Online Assessment (if configured).
- `PRE_SCREEN_PENDING` — awaiting HR decision; all required pre-screening information was collected at submission and the applicant has no further action.
- `NOT_CURRENTLY_QUALIFIED` — not proceeding.

The applicant sees the outcome and, where provided, the HR Reviewer's communication. A rejected applicant can still log in and view status but can never access future stages (section AM).

### CZ.1 Bulk pre-screen status decision (HR Reviewer)

The HR candidate queue supports an explicit bulk-status action for reviewed résumés. HR selects named applications using row checkboxes; filtering or changing pages never silently selects hidden applications. A bulk selection may contain only applications from the same organisation, job opening, opening/configuration version, and Pre-screening stage that are currently `PRE_SCREEN_ANALYSIS_READY` or `UNDER_HR_PRE_SCREEN_REVIEW`. Processing failures, grievances, unresolved discrepancy/attention flags, stale versions, already-decided applications, and applications in another stage are disabled for bulk action and retain their individual review path.

The HR Reviewer chooses `QUALIFIED_FOR_ASSESSMENT`, `PRE_SCREEN_PENDING`, or `NOT_CURRENTLY_QUALIFIED`, enters the required confidential internal note and applicant-visible status message, and confirms that every selected résumé and advisory result was reviewed. Qualifying in bulk also requires a scheduled assessment start, deadline, and permitted duration. Before confirmation, the screen shows the exact application IDs, candidate count, opening/version, target status, and notification effect; changing the selection invalidates the confirmation.

Bulk action is a convenience, not a group decision. On confirmation the backend rechecks role, tenant, opening/version, stage, current status, attention flags, prior decisions, and timing for every explicit application ID. If any item fails, the whole request is rejected before mutation and the screen identifies the applications requiring individual review. If validation succeeds, the system atomically creates a separate attributable HR decision, confidential note, applicant-visible message, status transition, audit event, and notification-outbox item for each applicant, linked by one batch/correlation ID. Each qualified applicant receives a distinct secure assessment invitation. Retries use the batch idempotency key and cannot duplicate decisions or messages; a rolled-back batch sends no notification.

No AI score, recommendation, search result, filter, sort order, or "select all matching" behavior may perform or imply bulk approval. The HR Reviewer remains accountable for every application included in the batch, and each applicant sees only their own resulting status and communication.

## DA. Online Assessment readiness and schedule

Before sending an assessment invitation, HR must set `scheduled_start_at`, `deadline_at`, and `permitted_duration` on that applicant's attempt. The invitation (`ASSESSMENT_INVITED`) displays all three. The timer begins only after readiness succeeds and the applicant actively starts; the effective end is the earlier of the permitted-duration expiry and the fixed deadline. An invitation cannot be sent with any of these values missing.

~~~text
ASSESSMENT_INVITED → READINESS_PENDING → READINESS_CONFIRMED → ASSESSMENT_STARTED
~~~

## DB. Assessment sections, questions, and submission

The applicant completes the configured sections (for example Self Introduction, Reading, Technical Questions) with per-section instructions. On completion the applicant submits (`ASSESSMENT_SUBMITTED`). The submitted responses are preserved together with the assessment version/configuration actually used at the time (section BM).

## DC. Assessment lifecycle — interruption, grievance, AI processing

- **Browser close / network interruption** — the in-progress attempt is preserved; the applicant resumes, or, where configured, submits a grievance (`ASSESSMENT_GRIEVANCE_SUBMITTED`).
- **Deadline expiry** (where a schedule is configured) — the attempt is handled per the configured rule; the applicant is never silently advanced.
- **AI processing** — after submission, `AI_PROCESSING` runs, then `ANALYSIS_READY` or `AI_PROCESSING_FAILED`. A processing failure is retried; the applicant is never falsely failed by an infrastructure error.

## DD. AI evaluation and HR review (applicant view)

The applicant's AI result is advisory and immutable; the HR Reviewer reviews it and decides. The applicant sees the outcome — `LIVE_INTERVIEW_RECOMMENDED`, `LIVE_INTERVIEW_HOLD`, or `NOT_PROCEEDING` — plus any permitted HR communication. The applicant never sees raw AI scores, reviewer comments, or another candidate's data.

## DE. Live Interview stages (one or more) — applicant view

Multiple Live Interview stages are supported. Each Live Interview stage has its own assigned panel, evaluation criteria, and completion/advancement conditions; "Live Interview" is **never** assumed to be a single stage. Example: **Live Interview 1** with interviewers A + B, then **Live Interview 2** with interviewers C + D. A later Live Interview stage becomes available only after the applicant satisfies the previous stage's advancement conditions (an HR decision).

Applicant view per Live Interview stage: stage name, schedule (date/time), joining instructions (the organisation's external meeting process), and the assigned interviewers' names. The applicant never sees interviewer scores/comments or any AI score.

Per-stage states: `LIVE_INTERVIEW_RECOMMENDED → INTERVIEW_ASSIGNED → INTERVIEW_COMPLETED`; after the final configured stage → `ADVANCED` / `HOLD` / `REJECTED`.

## DF. HR Interview stage — applicant view

HR Interview is a **distinct configurable stage**, never merged with the HR review/decision activity that follows any stage. It is scheduled like any human-evaluated stage; the applicant sees its schedule and instructions, then the outcome after evaluation. The stage produces an evaluation, and a separate HR decision subsequently advances or rejects the applicant (section AQ).

## DG. Final selection and onboarding

After all configured interview/evaluation stages are complete and the applicant is `ADVANCED`, the authorised HR decision-maker records **final selection**:

~~~text
ADVANCED → final selection → SELECTED → ONBOARDING_PENDING → ONBOARDING_COMPLETED → POSITION_FILLED
ADVANCED → final selection → NOT_SELECTED / REJECTED
~~~

On selection the backend **transactionally** records the decision-maker, timestamp, and vacancy reservation; it does not increment filled positions. HR records the external offer outcome and updates `ONBOARDING_PENDING`/completion from the external onboarding process. `ONBOARDING_COMPLETED` creates `POSITION_FILLED`; offer decline, onboarding failure, or cancellation releases the reservation and closes the application. Onboarding is a status/workflow trigger, **not another interview stage**.

## DH. Headcount and vacancy tracking

Track headcount as **target, reserved, onboarding-pending, filled, and available to select**. `available_to_select = target − reserved`, where `reserved` is committed capacity: every selected/onboarding applicant plus completed fills. `filled` increases only on `ONBOARDING_COMPLETED`; it is reported separately as the completed-employment metric. Selection is refused transactionally when `available_to_select = 0`; offer decline, withdrawal-equivalent suspension, onboarding failure, or cancellation releases an unfilled reservation. Each transition is separately recorded and auditable.

## DI. Applicant status and notification rules

The applicant-visible statuses follow the finalized lifecycle vocabulary (`INTEREST_REGISTERED → APPLICATION_DETAILS_REQUIRED → RESUME_SUBMITTED → AI_PRE_SCREENING → PRE_SCREEN_ANALYSIS_READY → UNDER_HR_PRE_SCREEN_REVIEW → QUALIFIED_FOR_ASSESSMENT / NOT_CURRENTLY_QUALIFIED / PRE_SCREEN_PENDING → ASSESSMENT_INVITED → READINESS_PENDING → READINESS_CONFIRMED → ASSESSMENT_STARTED → ASSESSMENT_SUBMITTED / ASSESSMENT_GRIEVANCE_SUBMITTED → AI_PROCESSING → ANALYSIS_READY / AI_PROCESSING_FAILED → UNDER_HR_REVIEW → LIVE_INTERVIEW_RECOMMENDED / LIVE_INTERVIEW_HOLD / NOT_PROCEEDING → INTERVIEW_ASSIGNED → INTERVIEW_COMPLETED → ADVANCED / HOLD / REJECTED → SELECTED → ONBOARDING_PENDING → ONBOARDING_COMPLETED → POSITION_FILLED`).

A notification is emitted at every applicant-visible persisted transition: application submitted, pre-screen decision, assessment invitation with its required schedule, interview scheduled, final selection, rejection, and automatic no-response suspension. Notifications are sent from the verified organisation address with the responsible HR Reviewer as `Reply-To`; a rolled-back transition must not send a contradictory email, and any failed send is retried without corrupting application state.

The named statuses above are specialised labels for the standard stages. Each configured `Other / Custom Stage` repeats the generic stage-instance lifecycle defined in AR and records its own stage ID, display name, mode, conductor/evaluator, result, passing-mark comparison, and HR decision. A custom-stage passing mark never advances the application automatically.

## DJ. Negative use cases and validation

- **Cross-realm login** — logging into one organisation's realm must not authenticate the user in another organisation's realm.
- **Client-supplied organisation ID** — rejected; the organisation is derived server-side from the workspace.
- **Apply ≠ membership** — applying for a job never creates an `organization_memberships` row.
- **Duplicate application** — prevented server-side for the same applicant and opening.
- **Expired/closed opening** — not appliable; the UI shows a terminal state and no Apply action.
- **Assessment deadline passed** — handled per the configured rule; the applicant is never silently advanced.
- **Rejected applicant** — future assessment/interview stages are inaccessible.
- **Headcount full** — final selection is refused transactionally.
- **Cross-tenant/IDOR access** — all applicant endpoints resolve `organization_id`, `job_opening_id`, and `application_id` from the authenticated session, never from the frontend.

## DK. Data integrity and authorization

The relationship chain is: **canonical person (verified email) → realm identity (`keycloak_user_id`) → `job_applications` → stage history / responses / evaluations / decisions**. `organization_memberships` is a **separate** relationship from applications: a person can be an applicant without being a member, and a member can hold many roles across organisations. The backend derives `organization_id`, `job_opening_id`, and `application_id` from the authenticated session and the opening graph; IDOR and cross-tenant access are blocked and audited.

Schema gaps that the Applicant lifecycle requires (and that must be recorded for the DBML pass, **not** silently added here) are listed in AW.2: applicant profile vs application separation, notification/audit history, multi-stage Live Interview per-stage interviewer assignment, final-selection/onboarding records, and assessment-version preservation.

## DL. Applicant lifecycle (end to end)

~~~text
Workspace URL → organisation resolved → Keycloak realm → login/register
→ Dashboard → active openings → job details → Apply
→ register interest (INTEREST_REGISTERED) → configurable pre-screening form
→ RESUME_SUBMITTED → AI pre-screening → HR pre-screen review
→ QUALIFIED_FOR_ASSESSMENT | NOT_CURRENTLY_QUALIFIED | PRE_SCREEN_PENDING
→ (qualified) ASSESSMENT_INVITED → readiness → ASSESSMENT_STARTED
→ ASSESSMENT_SUBMITTED → AI_PROCESSING → ANALYSIS_READY
→ UNDER_HR_REVIEW → LIVE_INTERVIEW_RECOMMENDED | HOLD | NOT_PROCEEDING
→ INTERVIEW_ASSIGNED → INTERVIEW_COMPLETED  (Live Interview 1 … Live Interview N)
→ HR Interview (if configured) → ADVANCED | HOLD | REJECTED
→ final selection → SELECTED → ONBOARDING_PENDING → ONBOARDING_COMPLETED → POSITION_FILLED | NOT_SELECTED / REJECTED
~~~

## DM. Testing requirements (applicant)

- **Backend unit tests** — workspace→organisation resolution; cross-realm login isolation; canonical-identity linking (no duplicate user on a known verified email); apply-creates-no-membership; duplicate-application prevention; dynamic pre-screen form generation from requisition config; bulk pre-screen eligibility, same-opening/version validation, idempotency, and stale-state rejection; assessment attempt preservation on interruption; AI-processing-failure retry; headcount reservation/over-fill refusal; final-selection transaction (decision-maker, timestamp, onboarding trigger); withdrawal releases reservation; server-side ID/IDOR rejection.
- **API/integration tests** — the complete flow: workspace entry → registration → job discovery → apply (configurable form) → submission/confirmation → individual or controlled bulk pre-screen decision → assessment invitation/readiness/submission → AI processing → HR review → Live Interview stage(s) (one or more) → HR Interview → final selection → onboarding initiated; each transition emits its notification. A valid bulk request creates individual decisions/outbox records with one batch ID; an invalid member rejects the entire batch without mutation or notification.
- **Security tests** — an applicant cannot reach another applicant's data; cannot submit an application against a client-supplied organisation ID; cannot access a rejected/closed stage; cannot self-advance a stage; cannot see AI scores or reviewer comments; cannot be granted membership by applying. Bulk requests cannot cross tenant/opening/version boundaries, include flagged/failed/grievance/already-decided applications, or use filter results as an implicit selection.

## DN. Applicant decisions adopted after completeness audit (2026-10-09)

This section supersedes earlier applicant wording where it conflicts. It is the binding decision register for implementation, UI, APIs, and the DBML.

### DN.1 Identity, workspace, and account rules

- A verified email identifies one **canonical person**. That person may have a different realm-local Keycloak user ID in each organisation realm. The same email may register in another organisation, but cannot register twice in the same organisation.
- Applicant access is application-scoped and never blocks staff membership in another organisation. A staff-role request must be evaluated only in its requested organisation; multi-organisation Applicant access is not a denial condition.
- Invalid, suspended, inactive, and unauthorised workspace URLs show the generic **Workspace unavailable** page without disclosing whether the organisation exists. The internal reason is audit-only.
- HR may permanently deactivate an Applicant only in HR's own organisation. It blocks that organisation's login/action access, preserves its history, and has no effect on the person's other realms or organisations.

### DN.2 Application, opening, and pre-screening rules

- A canonical person may have one active application per job opening. Rejection, closure, expiry, reactivation of the opening, or ordinary terminal states never permit a second application. The sole exception is a reasoned `VALID_WITHDRAWAL_OR_INCOMPLETE`: after six months, the service may create a linked retest/reapplication record for an eligible open position; it never reopens or overwrites the original application.
- Pre-screening collects every required field before submission. Submitted pre-screen responses are immutable. `PRE_SCREEN_PENDING` means only **awaiting HR decision**; it never asks the applicant for more information.
- HR may record a controlled bulk pre-screen status for explicitly selected, individually reviewed applications from one opening/version. Bulk execution still creates one human-owned decision, note, applicant message, audit event, and notification per application; flagged, failed, grievance, stale, previously decided, and cross-opening records require individual handling.
- Public application closing date blocks only new applications. Existing applicants continue through authorised stages.
- **Withdraw requisition** — applies before an opening exists. HR may raise an in-product request; only the requisition's creator executes or rejects it with a reason. A withdrawn requisition cannot create an opening.
- **Suspend opening** — temporarily stops new applications and further scheduling. HR raises an in-product request; only the Hiring Manager who created the opening executes it. The action records the request, decision, reason, timestamp, and notification. Active applications close with non-performance reason `APPLICATION_CLOSED_OPENING_SUSPENDED`.
- **Reactivate opening** — only that Hiring Manager executes reactivation against an in-product HR request/agreement and records the reason. Earlier closed applications remain closed; only people who never applied may apply after reactivation.
- **Close opening** — only that Hiring Manager executes manual closure, normally following an in-product HR request. Automatic closure still occurs when the last vacancy is filled. Closure is terminal and cannot use the Reactivate action.
- **No-response application suspension** — there is no applicant self-withdrawal control. Every applicant-actionable stage has a configured response deadline. If the applicant does not respond by that deadline, the service sets `APPLICATION_SUSPENDED_NO_RESPONSE`, releases any reservation, prevents later-stage access, and emails the applicant from the verified organisation sender with HR as `Reply-To`. HR may request reactivation; only the opening's creating Hiring Manager executes or rejects that request with an audited reason.
- When final onboarding completion fills the last vacancy, the system automatically closes the opening and closes remaining active applications with reason `OPENING_FILLED` and HR-originated notifications.

### DN.3 Assessment and grievance rules

- HR sets `scheduled_start_at`, `deadline_at`, and `permitted_duration` on the applicant's assessment attempt before invitation; the invitation states all three. The timer begins only after the applicant completes readiness and actively starts. The effective end is the earlier of duration expiry and deadline.
- The assessment requires a desktop/laptop using Google Chrome and a working built-in microphone or connected headset unless HR records an approved accommodation. The applicant's full accommodation detail is HR-restricted; an interviewer receives only the minimum operational instruction required to conduct the assigned stage, never the underlying personal detail. The accommodation must not change scores or progression automatically.
- At deadline, saved work is automatically submitted. An applicant who never starts, suffers interruption, or reaches deadline may submit an in-product grievance with a reason within 24 hours after that stage deadline, only when the opening's configured reattempt allowance remains. Readiness displays that allowance, including zero.
- HR approves or rejects grievance requests. Approval creates a separately auditable reattempt with HR-set timing and the latest HR-approved assessment configuration. A refusal or expired request closes the application with its specific reason and sends the applicant an email.
- An attempt already started remains pinned to the version active at start. An invited but not-yet-started applicant receives the latest HR-approved version at start. Later assessment/pre-screen configuration revisions apply prospectively and require HR approval.
- AI-processing failure keeps the application pending; HR cannot bypass, reject, or advance until successful AI evaluation is persisted.

### DN.4 Interview rules

- Each Live Interview stage owns its panel, required evaluator count, scoring criteria/weightage, schedule, completion condition, and advancement decision. A panel of two interviewers in one session remains one stage.
- HR cancellation/rescheduling keeps the applicant active, invalidates the cancelled session for evaluation, and sends replacement schedule/instructions from HR.
- An applicant no-show opens a 24-hour grievance window. Without a grievance, the application closes automatically. HR decides a grievance and notifies the applicant.
- After a Live Interview, a missing required interviewer evaluation keeps the stage pending for 24 hours. At hour 23 HR sends a reminder. At hour 24, or earlier where HR records a valid reason, HR may replace that interviewer and schedule one HR-approved retake with a new required evaluation. The system stores the confidential internal remark: **“This user retook the interview approved by HR because the previous interview could not be evaluated for a valid reason.”** It alerts the assigned Hiring Manager, is visible to HR, creates an SLA-report event for Organization Admin, and follows the configured escalation procedure. Only the retake evaluation set determines progression; original submissions remain immutable audit history.
- HR conducts/evaluates HR Interview and may record the subsequent progression/final decision. The HR evaluation and decision remain separate records.
- Interviewers may read prior applicant-visible HR messages for their authorised applicant/stage, but never AI results/scores or another interviewer's scores/comments.

### DN.5 Selection, onboarding, notifications, and retention

- `ADVANCED → SELECTED → ONBOARDING_PENDING → ONBOARDING_COMPLETED → POSITION_FILLED`. Selection transactionally reserves a vacancy; it does not increment filled count. HR records external offer acceptance/decline. Decline, onboarding failure, or onboarding cancellation closes the application and releases the reservation. Only HR updates the recruitment-side onboarding status.
- Every applicant-visible persisted transition emits a non-confidential email from the responsible verified HR Reviewer address. Failed delivery retries without rolling back state; terminal failure remains audited and can be followed up through the Organization Admin support/ticket route.
- Every HR decision stores exactly two separate communications: one confidential internal HR note and one applicant-visible status message used in the notification. The applicant-visible message contains status only; it never exposes the confidential note, AI reasoning, interviewer evaluations, or internal remarks.
- Closed applicants can log in during retention to see their final status and permitted closure reason, but cannot resume a stage. During retention/legal hold, authorised HR has full historical evidence; the assigned Hiring Manager has read-only historical assessment, immutable AI result, interviewer evaluations, and applicant-visible HR decisions for the opening. Interviewers have no historical applicant evidence, AI, peer-evaluation, or HR-decision access.
- Non-selected applications and all related recruitment evidence are permanently deleted after one year. Recruitment/onboarding records for an employee are deleted when employment terminates because ongoing employee records belong to an external system. Scheduled deletion retries on failure and blocks normal access until resolution.
- An HR Reviewer may place/release an organisation-scoped legal hold. An active hold prevents deletion and restricts access to authorised HR/legal personnel until release.
- Assessment terminal outcomes are `COMPLETED_SUCCESSFUL`, `COMPLETED_UNSUCCESSFUL`, and `TEST_STARTED_NOT_COMPLETED`. HR may classify a valid withdrawal/incomplete attempt only with a recorded reason. A valid withdrawal/incomplete outcome does not resume the old application; it permits an eligible retest/reapplication only after six months, subject to a then-open position and duplicate checks.

## DO. Cross-role decisions adopted 2026-10-12

This section supersedes earlier wording where it conflicts and is binding for screen design, APIs, authorization, and the next architecture/DBML reconciliation.

### DO.1 Requisition, opening approval, and publication

1. The person known in the business process as the Hiring Manager creates and submits requisitions using the organisation-level `REQUISITION_CREATOR` claim. That claim grants no applicant or unrelated-opening access; it is distinct from the job-opening `HIRING_MANAGER` claim.
2. The Hiring Manager configures stages, sections, questions/criteria, required interviewer counts, scoring, and weightage on the requisition before HR review.
3. HR approval locks an approved requisition version and authorises its creator to create one draft Job Opening carrying that version.
4. On the draft opening, the creator receives the opening-specific `HIRING_MANAGER` assignment and enters the required interviewer names/emails for each Live Interview stage.
5. HR approves or returns the opening/panel package. Approval changes it to `APPROVED_TO_PUBLISH` and sends the interviewer invitations.
6. HR never performs the Publish action. The Hiring Manager who created the opening publishes it, matching the director-provided mockup in `hirescope-mockup-4.html`.

### DO.2 Interviewer invitation

- A nominee may already be an organisation-realm user or may be new.
- Until a new nominee activates, the system stores a pending opening/stage/email nomination rather than inventing a user, membership, or role.
- HR approval sends a single-use Keycloak action link from the verified organisation address with the HR Reviewer as `Reply-To`; no password is emailed or stored.
- The nominee must authenticate/register within 24 hours. On activation the service links or creates the realm identity/membership and adds the opening-specific Interviewer role. The HR opening screen shows only `Invited`, `Activated`, or `Expired`; expiry notifies HR and the Hiring Manager, who can re-invite the same nominee or replace the nominee through the normal panel approval flow.

### DO.3 Realm and role switching

- A role switcher shows only assignments within the currently authenticated organisation realm.
- The same verified email may identify the canonical person in several realms, but each realm has a different Keycloak user ID and may have a different password.
- To change organisations, the user logs out, opens the other organisation's workspace URL, and authenticates there. There is no cross-realm SSO.

### DO.4 Blind-review release points

- Before a stage's AI/evaluator result locks, résumé and distance-travelled context are hidden from HR, Hiring Manager, Interviewer, Organization Admin, and Platform Admin.
- After the AI pre-screen result locks, HR may see that context for the HR decision.
- After the HR decision locks, the assigned Hiring Manager may see it read-only.
- Interviewers never see résumé/distance-travelled context unless a later explicit stage configuration and privacy decision permits it; they never see AI or peer-evaluator results.

### DO.5 Assessment and custom stages

- `scheduled_start_at`, `deadline_at`, and `permitted_duration` are mandatory before HR can send an assessment invitation.
- `Other / Custom Stage` is supported and configures stage name, mode, conductor, evaluator/count, sections/questions/criteria, total marks, passing mark, weightage, timing/submission needs, and completion condition.
- A passing mark is advisory decision support. It never advances, holds, or rejects an applicant automatically; HR records the decision.

### DO.6 Controlled withdrawal, suspension, closure, and reactivation

- HR raises in-product requests concerning a requisition or opening. Applicants have no self-withdrawal action; non-response is handled by the configured automatic suspension rule.
- Only the Hiring Manager who created the requisition/opening executes or rejects the applicable withdrawal, suspension, reactivation, or closure request, with an audited reason and notification. The Hiring Manager has an in-product request inbox; HR never directly closes an opening.
- The resulting email is service-owned, sent from the verified organisation address, and uses the responsible HR Reviewer as `Reply-To`.

### DO.7 Headcount semantics

- `available_to_select = target − reserved`; selection atomically creates a reservation and is refused when this value is zero.
- `filled` counts only completed onboarding. It is a completed-employment metric, not an available-capacity calculation.
- Decline, automatic no-response suspension, onboarding failure, or cancellation releases an unfilled reservation. The UI shows both available-to-select and filled so they cannot be confused.

### DO.8 Super Admin replacement authority

- Superseded by section DP. Super Admin is the organization-scoped buyer/owner who initiates Organization Admin replacement only for an owned organization; it is not a platform-level HR Reviewer replacement authority.
- The owner cannot manage requisitions, applicants, evaluations, openings, interviews, or ordinary role assignments.

### DO.9 Accommodation and grievance privacy

- HR sees the full applicant accommodation request and records the operational adjustment. Assigned interviewers see only that operational instruction.
- An applicant submits an in-product grievance with a reason no later than 24 hours after the relevant configured stage deadline. HR records the decision and any reattempt; an expired grievance window is rejected with notification.

### DO.10 Manual closure retention visibility

- When the creating Hiring Manager manually closes an application, the closed record remains read-only for retention/legal-hold purposes.
- HR may view the full historical assessment, immutable AI evaluation, interviewer evaluations, HR decision, and confidential HR note. The assigned Hiring Manager may view the same historical assessment/evaluation evidence and the applicant-visible HR decision, but never the confidential HR note.
- Interviewers do not receive historical assessment, AI, peer-evaluation, or HR-decision access after closure. Organization Admin receives SLA/audit metadata only, not recruitment evidence.

### DO.11 HR decision communications

- Each HR decision stores exactly one confidential internal HR note and exactly one applicant-visible status message.
- The notification uses only the applicant-visible status message. The internal note is never returned by applicant, Hiring Manager, interviewer, Organization Admin, or Platform/Super Admin APIs.

### DO.12 Missing interviewer evaluation, retake, and SLA

- If a required interviewer does not evaluate, HR may replace that interviewer and require a new evaluation through one HR-approved retake.
- The confidential internal retake remark is: **“This user retook the interview approved by HR because the previous interview could not be evaluated for a valid reason.”**
- The event alerts the assigned Hiring Manager, is visible to HR, enters the Organization Admin SLA report, and starts the configured escalation procedure. The replacement evaluation governs progression; previous submitted evaluations remain immutable audit history.

### DO.13 Assessment completion and six-month retest

- Assessment outcomes are `COMPLETED_SUCCESSFUL`, `COMPLETED_UNSUCCESSFUL`, or `TEST_STARTED_NOT_COMPLETED`.
- HR may record `VALID_WITHDRAWAL_OR_INCOMPLETE` only with an internal reason. It ends the current application attempt and creates an eligibility date six months later; it never reopens the old application. Any retest/reapplication after that date must be for an eligible open position and pass normal duplicate checks.

## DP. Super Admin ownership, onboarding, and Organization Admin continuity

This section is the controlling interpretation of the earlier onboarding and recovery wording in sections B, I, J, and DO.8. It extends those established workflows without changing recruitment-role assignment, review, or decision ownership.

### DP.1 Scope, authentication, and access boundary

- The **Super Admin** is the buyer/business owner for one or more explicitly authorized organizations; it is not Platform Admin and is not a tenant-wide operational role.
- Before a workspace exists, the Super Admin authenticates in the existing Keycloak **onboarding** realm. After provisioning, that same control-plane identity continues to use the owner dashboard; it is not silently moved into the organization realm and does not need a tenant realm role.
- PostgreSQL authorizes every owner action from a verified `organization_ownership` relationship. The UI shows only owned organizations; a client-supplied organization ID, hidden control, or another realm session never authorizes access.
- A Super Admin can initiate onboarding, read its owned organization’s appointment/audit metadata, and initiate Organization Admin replacement. It cannot provision realms, approve its request, assign Hiring Manager/HR Reviewer/Interviewer roles, view applicant evidence, or make recruitment decisions. Organization Admin cannot edit, remove, demote, or replace Super Admin ownership.

### DP.2 Super Admin screens and navigation

~~~text
Super Admin
├─ Dashboard (SA-1)
├─ Organization onboarding (SA-2)
│  └─ Submission / provisioning status (SA-3)
├─ Organization Admin management (SA-4)
│  └─ Replace Organization Admin (SA-5)
├─ Read-only audit history (SA-6)
└─ Logout
~~~

`SA-1` shows organization identifier/name, onboarding/provisioning status, current Organization Admin, appointment/replacement status, recent ownership changes, and pending owner actions. Values in the mockup are labelled static demo data. `SA-2` captures organization details and the **nomination** of the initial admin. `SA-3` exposes `PENDING_REVIEW`, `RETURNED_FOR_CORRECTION`, `REJECTED`, `PROVISIONING`, `REALM_READY`, `AWAITING_ADMIN_ACTIVATION`, `ACTIVE`, and `PROVISIONING_RECOVERY` without implying a live backend. `SA-4` and `SA-5` show current appointment, history, candidate status, controlled transition, and recoverable error states. `SA-6` is read-only and contains no applicant/recruitment evidence.

### DP.3 Onboarding: Super Admin initiates; Platform Admin approves and provisions

1. Super Admin logs in through the onboarding realm and opens `SA-2`.
2. Super Admin submits organization details, required proof/reason, and the initial Organization Admin candidate’s name and email. The nomination alone creates no membership, tenant role, or administrative privilege.
3. The backend records `organization_onboarding_requests = PENDING_REVIEW`, linked to the Super Admin identity but with no `organization_id` yet. `SA-3` shows the pending state.
4. Platform Admin reviews the request. It may return it for correction or reject it with a recorded reason; the Super Admin is notified and may correct/resubmit when policy allows.
5. On approval, Platform Admin (through the backend service) creates the organization workspace and its dedicated Keycloak realm, records organization ownership, and starts the initial appointment. The browser never calls Keycloak Admin APIs.
6. The nominated candidate is either an existing verified identity eligible for the new realm or receives the existing Keycloak invitation/registration path. An identity in another organization realm is not transferred automatically.
7. The appointment remains `AWAITING_REGISTRATION`, `INVITED`, `EMAIL_VERIFICATION_PENDING`, or `ACCEPTANCE_PENDING` until required verification and acceptance succeed. Only then does the backend create/activate the organization membership and `ORGANIZATION_ADMIN` authority in the new tenant realm.
8. The backend records `ACTIVE`, notifies the Super Admin and Organization Admin, and the established Organization Admin workflows begin. Platform Admin remains the approver/provisioner, never the buyer or routine role assigner.

### DP.4 Replace Organization Admin (controlled transition)

1. Super Admin opens `SA-4`, selects an owned organization, reviews the active administrator and appointment history, then chooses **Replace Organization Admin** (`SA-5`).
2. The backend checks ownership, organization lifecycle, candidate eligibility, and that the request is not a duplicate/in-progress replacement. A current Organization Admin attempt to remove the Super Admin is denied and audited.
3. The candidate is classified explicitly: `EXISTING_ELIGIBLE_IDENTITY`, `AWAITING_REGISTRATION`, `ANOTHER_REALM_IDENTITY_REQUIRES_LINKING`, `VALIDATION_FAILED`, or `INELIGIBLE`. No cross-realm transfer is implicit.
4. A replacement record is created as `INVITED` / `EMAIL_VERIFICATION_PENDING` / `ACCEPTANCE_PENDING`. Expiry or rejection becomes `EXPIRED` / `REJECTED`; the outgoing admin stays active and the Super Admin may resend, cancel, or nominate another candidate.
5. After the candidate completes required identity verification and acceptance, the system performs a guarded activation transaction: revalidate both accounts and the ownership record; activate the incoming tenant membership and `ORGANIZATION_ADMIN` authority; record the appointment; revoke the outgoing **Organization Admin** authority; then mark the replacement `COMPLETED` and notify all appropriate parties.
6. Existing organization data, job openings, applicant records, participant assignments, interviews, evaluations, and audit history remain intact. An outgoing administrator may retain any separately authorized Hiring Manager, HR Reviewer, or Interviewer assignment; that assignment is neither silently revoked nor inherited by the incoming administrator.
7. If activation fails before commit, the incoming appointment is `ACTIVATION_FAILED` / `RECOVERY_REQUIRED`, the outgoing administrator remains active, and a reconciler/Platform Admin recovery operation resolves Keycloak and database state. The Super Admin sees the recoverable outcome at `SA-5`.
8. If the outgoing administrator must be removed immediately for a security incident, an authorized emergency revocation creates `NO_ACTIVE_ADMIN_RECOVERY` after revocation, alerts Platform Admin, and restricts normal organization administration until a verified incoming administrator is activated. This is distinct from ordinary replacement.
9. A later replacement uses the same flow. An organization with no active Organization Admin shows an explicit recovery state; it never silently promotes a member.

### DP.5 Appointment, assignment, and audit history

Every role assignment retains `organization_id`, recipient identity/membership, scope (including job opening where applicable), current status, `created_by_user_id`, actor authority snapshot, `created_at`, later modifier/revoker identity, timestamps, and required reason. Organization Admin appointment history additionally retains outgoing/incoming identities, initiator, transition status, invitation/verification evidence references, effective time, failure/recovery reason, and correlation ID.

Replacing Organization Admin does not rewrite attribution. For example, a Hiring Manager assignment created by Organization Admin 001 remains assigned by that actor after Organization Admin 002 becomes active; it remains valid unless an authorized, independent lifecycle action changes it. Super Admin audit views show this metadata only and do not grant operational assignment controls.

### DP.6 Server-side authorization checks

- `owner.organization.read`, `owner.onboarding.create`, `owner.admin-replacement.create`, and `owner.audit.read` require the authenticated Super Admin identity plus an active ownership record for the target organization.
- Platform Admin-only approval/provisioning endpoints require a `master`-realm Platform Admin identity. Tenant authorization continues to require tenant realm identity, active membership, scoped role, and server-resolved organization.
- UI visibility mirrors these checks but is not a security boundary. All denied cross-owner, Organization Admin-to-Super-Admin, and stale/replayed transition attempts are recorded in audit history.
