# HireScope System Architecture

**Status:** Cross-role architecture reconciled through 2026-10-12; DBML migration remains required for opening approval, pending interviewer nominations, controlled-action requests, and automatic no-response suspension.  
**Updated:** 2026-10-12  
**Related references:** [website-user-flow.md](./website-user-flow.md), [pente_db_diagram.dbml](./pente_db_diagram.dbml)

## 1. Authority, identity, and tenancy

HireScope is a multi-tenant recruitment platform. Keycloak is the authentication and credential authority; PostgreSQL is the application/business-data authority. PostgreSQL never stores a Platform Admin password.

There is one pre-provisioned Platform Admin. The account exists in the platform Keycloak realm before deployment is handed over. First login may trigger Keycloak's required password-update action; password recovery and later password changes remain Keycloak flows. Super Admin is a separately auditable, organization-scoped business ownership authority; it can initiate onboarding and Organization Admin replacement only for organizations explicitly linked to that owner. It is not a Platform Admin, tenant operational role, or HR Reviewer replacement authority.

### Keycloak realm decision

~~~text
Keycloak
├─ master
│  └─ Platform Admin
├─ onboarding
│  └─ pending registrations / unassigned users
├─ pente
│  └─ Pente users
└─ careplus
   └─ CarePlus users
~~~

Initial public registration authenticates in the **onboarding** realm. The backend creates the existing `users` row with `is_assigned = false`; there is no separate registration-user table. A user becomes assigned only when an organisation membership exists, at which point `users.is_assigned` is set to `true` in the same business transition.

A verified registrant who selected **I am not registering as an applicant** enters through a known workspace URL and requests `REQUISITION_CREATOR` or HR Reviewer access. There is no public organisation directory. The backend creates/links that identity in the selected tenant realm, creates a `PENDING_VERIFICATION` membership, sets `users.is_assigned = true`, and records the role-verification request. This is not active staff authority: only Organization Admin approval changes the membership to `ACTIVE` and creates the requested organisation-level authority; rejection leaves the membership non-active and preserves the request history.

The onboarding realm is required because a pending registrant has no organisation yet and therefore has no organisation realm to authenticate in. The registrant remains in the onboarding realm until Platform Admin approval creates the organisation realm, then the identity transition links the user into that realm.

Because Keycloak user IDs are realm-local, approval must use an explicit **identity transition**: create/link the user in the approved organisation realm through the Keycloak Admin API, then transactionally replace the application's external Keycloak reference (including dependent foreign keys) with the organisation-realm identity before creating the membership. The old onboarding identity is disabled or removed only after that transition succeeds. This avoids an application-owned mapping table while preserving the existing `keycloak_user_id` data model.

| Scope | Authority or participant | Representation |
|---|---|---|
| Platform | Platform Admin | Pre-provisioned Keycloak identity in `master`; optionally a `users` row for application auditing. No organisation membership or public registration. |
| Organisation | Organization Admin, HR Reviewer, Requisition Creator | `organization_memberships` + organisation-level authority. `ORGANIZATION_ADMIN`, `HR_REVIEWER`, and `REQUISITION_CREATOR` have `job_opening_id = NULL`. |
| Job opening | Hiring Manager | A job-opening-specific `HIRING_MANAGER` assignment is created when an authorised requisition creator creates an opening from their own HR-approved requisition. |
| Job opening / Live Interview stage | Interviewer | Pending panel nomination keyed by opening + stage + normalized email; after account activation, `organization_member_roles` for an active membership and opening plus stage-scoped assignments. |
| Application | Applicant | `job_applications` joins a Keycloak-backed user to a job opening. |

`REQUISITION_CREATOR` is the limited organisation-level authority required before an opening exists; it grants no applicant, interview, or unrelated-opening access. Hiring Manager and Interviewer operational assignments are job-opening-scoped, and an organisation can have many of each. One person may have realm-local identities in multiple organisations, but each authenticated session belongs to one realm only. The backend enforces that every job-specific assignment belongs to the membership's organisation. `ORGANIZATION_ADMIN` and `HR_REVIEWER` remain the one-per-organisation authorities.

### Multi-tenancy and role architecture

~~~text
Platform → Organizations → Organization Keycloak realm → Organization members → Organization-scoped data
~~~

An Organization Admin authenticating in the `pente` realm is scoped to Pente data only and must never reach `careplus` data. The backend enforces tenant scope on every request; cross-tenant access is denied and audited.

Roles are layered by scope, not flattened into organisation-wide titles:

~~~text
Platform Admin     → platform level
Organization Admin → organisation level (job_opening_id = NULL)
HR Reviewer        → organisation level (job_opening_id = NULL)
Requisition Creator→ organisation level (job_opening_id = NULL; requisitions only)
Hiring Manager     → job-opening level
Interviewer        → job-opening level
Applicant          → job-application level
~~~

Hiring Manager and Interviewer always require a job opening. Requisition creation before an opening uses the separate `REQUISITION_CREATOR` authority. Organization Admin and HR Reviewer are organisation-level (`job_opening_id = NULL`); each organisation has at most one of each.

Within one authenticated realm, a user can expose several role contexts—for example Pente → Hiring Manager and Pente → Interviewer. The workspace UI renders these in a same-realm **role switcher**. A role in another organisation is not included in that authenticated switcher: the person logs out, opens the other workspace URL, and authenticates in its realm. The verified email may be the same, but realm-local Keycloak user IDs and passwords may differ; there is no cross-realm SSO.

### Multi-organisation identity, cross-realm authentication, and canonical identity

There is **no public directory of organisations**. Entry is workspace-scoped: the workspace URL (for example `pente.hirescope.com`) selects the organisation, and the `organization_id` is resolved **server-side** from the workspace identifier — a client-supplied organisation ID is never trusted.

Because each organisation has its **own Keycloak realm**, authentication is realm-scoped and there is **no cross-realm SSO**: a session in `pente` does not authenticate the user in `careplus`, and switching organisations requires a fresh login in that organisation's realm. The application must never assume that a login in one realm authenticates the user in another.

A single person can hold different roles across organisations (Applicant in one, Hiring Manager in another). **Canonical identity is the verified email address.** `canonical_people` owns that unique verified email, while `users` stores each realm-local `keycloak_user_id` linked to the canonical person. Applying for a job **never** creates an `organization_memberships` row — membership is separate from application (section 5).

## 2. System context

~~~text
Landing page
  ├─ Login ───────────────► Keycloak authentication (correct realm for the user)
  └─ Organization / user registration ─► onboarding realm ─► Core backend
                                           │
                                           ▼
                                     PostgreSQL

Platform Admin (platform realm)
  └─ Frontend ─► Core backend
                   ├─ PostgreSQL: onboarding, tenants, memberships, hiring data
                   ├─ Keycloak Admin API: realm and initial-admin provisioning
                   ├─ Email service: approval/rejection notices
                   └─ Object storage: proof documents (stored as URLs in business records)

Organization user (organisation realm) ─► Keycloak ─► Organisation workspace ─► Core backend
Applicant ─► Job opening ─► job_applications
~~~

The platform frontend and API authorize every request from the validated Keycloak token and application relationships. A Platform Admin token permits platform monitoring/onboarding endpoints only; it does not confer access to organisation recruitment operations.

## 3. Platform Admin login and session

1. The user selects **Login** on the landing page.
2. The application redirects to Keycloak.
3. Keycloak validates the pre-provisioned Platform Admin credentials.
4. If required, Keycloak performs the password-update action and returns the authenticated user to the application.
5. The backend validates token claims/identity and determines the Platform Admin authority.
6. The Platform Admin dashboard is returned.

Failed authentication remains at the Keycloak/login experience with an appropriate error. Logout terminates the application session, invokes Keycloak logout/end-session, and redirects to Login or the landing page.

## 4. Organisation onboarding architecture

`role_verification_requests` cannot safely represent organisation onboarding: it requires an existing `organization_id`, is tied to an organisation membership, and represents an in-workspace role decision. Onboarding occurs before all of those records exist. The dedicated `organization_onboarding_requests` table is therefore required.

~~~text
Requester registration + onboarding-realm identity (users.is_assigned = false)
  └─ submit organisation details, proof and reason
       └─ organization_onboarding_requests = PENDING
            └─ Platform Admin reviews read-only submission
                 ├─ Reject
                 │    └─ REJECTED + rejection reason + email from configured Platform Admin sender
                 └─ Approve (backend transaction / compensating workflow)
                      ├─ create organizations record and organisation identifier
                      ├─ Keycloak Admin API creates/configures tenant realm
                      ├─ create/link requester in tenant realm; retire onboarding identity after success
                      ├─ update application Keycloak reference and store organizations.keycloak_realm
                      ├─ create/link initial Organization Admin identity
                      ├─ create active membership + ORGANIZATION_ADMIN role (no job opening); set is_assigned = true
                      ├─ set request APPROVED + approved_organization_id
                      ├─ activate workspace (status = ACTIVE; may later be set to SUSPENDED)
                      └─ send approval/onboarding email
~~~

Realm provisioning is a service operation, not a browser operation. The backend must use idempotency, persist or compensate failures, and only mark the request approved after the business state is coherent. The approval email is sent from the configured Platform Admin email identity; a rejection never requires manual follow-up.

### Organization onboarding table reference

| Column | Purpose/Definition | PK | FK | Data Type | Nullable | Notes |
|---|---|---|---|---|---|---|
| `onboarding_request_id` | Immutable identifier for the pre-organisation request. | Yes | No | `uuid` | No | Request/audit reference shown to Platform Admin. |
| `requester_keycloak_user_id` | Keycloak-backed requester and proposed initial Organization Admin. | No | `users.keycloak_user_id` | `uuid` | No | Name, email, and phone are read from `users`; no credential duplication. |
| `organization_name` | Submitted legal/operating organisation name. | No | No | `varchar(150)` | No | Becomes the `organizations.organization_name` value if approved. |
| `document_url` | Location of required organisation/admin proof. | No | No | `text` | No | Object-storage reference; not authentication data. |
| `comment` | Requester reason/context. | No | No | `text` | No | Visible to Platform Admin during review. |
| `request_status` | Lifecycle state. | No | No | `varchar(20)` | No | Allowed values: `PENDING`, `APPROVED`, `REJECTED`. |
| `reviewed_by_keycloak_user_id` | Platform Admin who recorded the decision. | No | `users.keycloak_user_id` | `uuid` | Yes | Null while pending. |
| `reviewed_at` | Decision timestamp. | No | No | `timestamptz` | Yes | Null while pending. |
| `rejection_reason` | Reason included in rejection notification. | No | No | `text` | Yes | Required for `REJECTED`; null otherwise. |
| `approved_organization_id` | Eventual created organisation. | No | `organizations.organization_id` | `uuid` | Yes | Null for pending/rejected; unique when set. |
| `created_at` | Request submission timestamp. | No | No | `timestamptz` | No | Indexed with status for the review queue. |
| `updated_at` | Last lifecycle update timestamp. | No | No | `timestamptz` | No | Updated on decision/provisioning outcome. |

Cardinality is `users (requester) 1 — * onboarding requests` and `onboarding request 0..1 — 1 organization`. The request has no organisation relationship until approval; it must never use an `organization_id` to represent a pending organisation. DBML indexes `request_status, created_at`, `requester_keycloak_user_id, created_at`, and unique `approved_organization_id` support its review and final-link lifecycle.

The existing `users.is_assigned` flag is `false` for onboarding-realm registration and `true` only after its organisation membership has been created. It is not a replacement for `organization_memberships`: membership remains the authoritative tenant relationship and the flag is synchronized as part of the same transition.

## 5. Membership, roles, and applications

`organization_memberships` says that a user belongs to an organisation. It can be created before a role-verification decision. `organization_member_roles` holds scoped authority:

- `ORGANIZATION_ADMIN` has `job_opening_id = NULL`.
- `HR_REVIEWER`, `ORGANIZATION_ADMIN`, and `REQUISITION_CREATOR` have `job_opening_id = NULL`. `HIRING_MANAGER` and `INTERVIEWER` are job-opening-scoped.
- A member may participate in more than one job opening as permitted by the organisation's business rules.
- A single membership may hold multiple job-specific roles across one or more job openings in addition to an organisation-level role. The workspace UI switches only among contexts in the current realm; changing organisation requires logout and authentication in the other realm.

The approved Hiring Manager first raises a requisition, configuring its job requirements and its stage → section → question hierarchy together with per-stage scoring/weightage (sections 16–18 and 27). Once the HR Reviewer approves the requisition, that same Hiring Manager creates the Job Opening, which carries the approved stage configuration; the service then creates the job-specific Hiring Manager assignment for that opening. Organization Admins only view openings and their aggregate applicant/position information in this phase.

A public registrant may select **I am not registering as an applicant** from a known organisation workspace and request either `REQUISITION_CREATOR` or HR Reviewer access, then provide proof and a comment. Email verification creates a `PENDING_VERIFICATION` membership and a pending verification request; both authorities have no Job Opening. Interviewer access never originates from public self-selection: the Hiring Manager nominates a name/email on a draft opening and HR approves the opening/panel package. The Organization Admin approves/rejects the submitted organisation-level authority without changing its scope.

Applicants are associated to a job opening through `job_applications`; organisation scope is derived from that opening. Existing stage, response, advisory-AI, human-decision, and grievance records remain organisation-scoped through the application/opening graph.

## 6. Platform Admin capabilities and boundaries

Platform Admin can view onboarded workspaces, pending/approved/rejected onboarding requests, requester proof/comments, workspace membership status, Organization Admin status, job-specific assignment summaries, and role-verification requests. Platform Admin can approve/reject an onboarding request and thereby initiate realm/workspace provisioning, and can activate/suspend a workspace.

Platform Admin cannot create job openings, manage candidates or interviews, make recruitment decisions, manage recruitment workflows, or assign Organization Admin/job-specific roles in ordinary operation. Workspace screens are monitoring-only and visually/API read-only.

## 7. Organization Admin authentication and onboarding email

After Platform Admin approves the onboarding request, the notification service sends the Organization Admin an onboarding email containing the Login ID, a single-use expiring Keycloak action link, organisation/workspace information, and access instructions. No plaintext/default password is sent or stored by HireScope.

~~~text
Platform Admin approval
  → organisation created
  → organisation Keycloak realm created
  → Organization Admin associated
  → onboarding email sent
  → Organization Admin logs in
~~~

First-login flow:

1. The Organization Admin selects **Login** on the landing page.
2. The application redirects to the organisation's Keycloak realm (for example `pente`).
3. A new realm user follows the single-use action link and establishes a password; an existing realm user authenticates normally.
4. Keycloak completes any other required first-login actions.
5. Authentication succeeds and the Organization Admin enters the organisation workspace.

This is distinct from Platform Admin login in `master`. After the workspace exists, the Organization Admin authenticates in the organisation realm, never the onboarding realm.

## 8. Organization Admin workspace and tenancy

After login, the Organization Admin enters the tenant-scoped workspace for their organisation. The workspace shows the organisation name, the Organization Admin identity, organisation context, workspace lifecycle status (ACTIVE/SUSPENDED), and organisation-administration navigation.

All organisation context is tenant-scoped. The backend derives tenant scope from the authenticated organisation realm and rejects any request whose organisation does not match; an Organization Admin in `pente` sees only Pente data and must never reach `careplus` data.

The dashboard provides two member views:

- **Pending / unverified users** — registered users whose organisation/job-specific assignment has not been confirmed, showing user details, requested role, requested job opening (if applicable), supporting document, comment, and verification status.
- **Confirmed users** — users whose assignment has been verified, showing membership status, assigned role, job opening (if applicable), and assignment date.

Pending users are the pending `role_verification_requests` described in section 5, viewed from the Organization Admin's side.

## 9. Organization Admin user verification and role assignment

The Organization Admin is the decision-maker for in-workspace user verification:

1. Open the pending user.
2. Review user details.
3. Review the submitted document.
4. Review the user comment.
5. Review the requested organisation-level authority.
6. Decide to approve or reject.
7. On approval, assign the appropriate role and confirm the user; on rejection, record the rejection reason.

On approval the user moves from the pending/unverified view to the confirmed users view; the confirmed user detail page shows user details, organisation details, workflow-created job assignments, and supporting verification. On rejection the request becomes `REJECTED` (reason required). The notification service sends from the verified organisation address with the responsible Organization Admin as `Reply-To`; individual mailbox credentials are never stored. The rejected request is retained in history.

There are two one-per-organisation authorities: **Organization Admin** and **HR Reviewer**. An approved `REQUISITION_CREATOR` has requisition authority without a Job Opening; creating an opening from their approved requisition creates job-specific Hiring Manager scope. An Interviewer always requires an HR-approved opening/stage nomination. The Organization Admin must approve/reject the requested organisation-level authority as submitted, not substitute another scope.

No new table or column is introduced. Pending/confirmed state reuses `users.is_assigned`, `organization_memberships.membership_status`, and `role_verification_requests.verification_status`.

### Workflow-created job-specific assignments

Organization Admin may approve `REQUISITION_CREATOR` authority and monitor/deactivate assignments, but cannot bypass the recruitment workflow with a manual Add Role action. Creating an opening from the user's own approved requisition creates its `HIRING_MANAGER` assignment. HR approval of the Hiring Manager's stage-specific panel nominations authorises `INTERVIEWER` assignments; new invitees receive the assignment only after Keycloak activation creates/links their membership.

## 10. Organization Admin job opening and applicant management

The Organization Admin monitors the organisation's job openings:

- View job openings.
- View applicant counts, filled positions, remaining positions, and assigned participants.
- View assigned Hiring Managers and Interviewers, and the organisation's HR Reviewer.
- View applicant/application summaries for the organisation's job openings.

Applicants are not organisation members; they are associated to a job opening through `job_applications` and never appear in the confirmed-members list. The Organization Admin sees applicant information through the relevant job opening, not as organisation membership.

## 11. Organization Admin capabilities and boundaries

The Organization Admin can:

- View and manage organisation users.
- Review pending user requests and supporting documents/comments.
- Approve organisation-level `REQUISITION_CREATOR`/HR Reviewer access and manage membership status.
- Monitor or deactivate workflow-created Hiring Manager and Interviewer assignments; never create them through a manual Add Role bypass.
- View confirmed users and user details.
- View job opening assignments.
- Monitor job openings and their assignment/status summaries.
- View applicants/applications for organisation job openings.
- Manage organisation-level configuration where already supported.
- Logout.

The Organization Admin must not:

- Create other organisations or approve organisation onboarding.
- Create or configure Keycloak realms.
- Manage the Platform Admin or platform-level resources.
- Access another organisation's data.

Organization Admin authority is organisation-scoped and excludes all platform-level responsibilities.

## 12. APIs and service boundaries

| Area | Application API | Service responsibility |
|---|---|---|
| Authentication | `GET /auth/login`, `POST /auth/logout`, password recovery/change redirects | Delegates sign-in, logout, recovery, and required password update to Keycloak. |
| Onboarding request | `POST /organization-onboarding-requests`, `GET /...`, `GET /.../{id}` | Creates/list/reads PENDING, APPROVED, REJECTED business records and proof links. |
| Onboarding decision | `POST /.../{id}/approve`, `POST /.../{id}/reject` | Platform-admin authorization; approval orchestrates organisation/realm/admin creation; rejection records reason and triggers mail. |
| Workspace monitoring | `GET /organizations`, `GET /organizations/{id}`, `GET /organizations/{id}/users` | Platform-level read models; no recruitment mutation endpoints exposed here. |
| User monitoring | `GET /organizations/{id}/users/{userId}`, `GET /role-verification-requests` | Returns membership, Organization Admin flag, job assignments, and verification detail. |
| Role management | Organisation-scoped endpoints only | Organization Admin approves/rejects role requests; approval creates the assignment, rejection records the reason and triggers the decision email. |
| Keycloak administration | Internal service adapter, not public browser API | Create/configure realm, create/link initial Organization Admin, configure required actions; secrets stay server-side. |

## 13. Access-control matrix

| Function | Platform Admin | Organization Admin | Hiring Manager | HR Reviewer | Interviewer | Applicant |
|---|---|---|---|---|---|---|
| View platform dashboard | Yes | No | No | No | No | No |
| Manage organisation onboarding | Yes | No | No | No | No | No |
| Create realm/workspace | Yes / system service | No | No | No | No | No |
| View workspace users | Read-only, all workspaces | Own org | Limited assignment scope | Own org | Limited assignment scope | Own data |
| Assign Organization Admin / sole HR Reviewer | Initial provisioning only; read-only appointment monitoring | Cannot replace Super Admin; ordinary role flows only | No | No | No | No |
| Create job-specific assignments | No | Monitor/deactivate | HM assignment by opening creation; nominate panel | Approve panel | No | No |
| View role verification | Read-only | Own org | No | No | No | Own request |
| Manage jobs | No | Monitor | Create/publish/manage own opening | Approve/return opening package | No | No |
| Create / edit requisition | No | As permitted | Yes (own) | No | No | No |
| Configure stages / sections / questions / weights | No | Review | Yes (own requisition) | Review | No | No |
| Request / execute requisition withdrawal | No | No | Execute own | Request | No | No |
| Approve / return requisition | No | No | No | Yes | No | No |
| View applicants/applications | No | Own org | Assigned opening | Own org | Assigned interview | Own application |
| Apply to job | No | No | No | No | No | Yes |
| Review applicants & evaluations | No | Own org | Assigned opening (read-only) | Own org | Assigned interview | Own application |
| View applicant responses | No | Own org | Assigned opening | Own org (per rule) | Assigned interview | Own application |
| Edit AI / evaluator evaluation | No | No | No | No | No | No |
| Add own comment | No | Yes | Yes (Hiring Manager comment) | Yes (HR comment) | Yes | No |
| Decide applicant progression | No | No | No | Own org | No | No |
| Mark applicant eligible / schedule Live Interview | No | No | No | Own org | No | No |
| View interviewer completion status | No | Own org | Assigned opening | Own org | Assigned interview (status only) | No |
| Submit interview evaluation | No | No | No | No | Assigned interview | No |
| Suspend/reactivate/close opening | No | Monitor | Execute own | Request | No | No |
| Record final selection / initiate onboarding | No | As permitted | No | Yes (own org) | No | No |
| View own onboarding status | No | No | No | No | No | Own application |
| No-response application suspension / reactivation | No | Monitor | Execute approved reactivation for own opening | Request reactivation | No | No self-withdrawal; submit grievance only |
| Switch own role context | No | Same realm | Same realm | Same realm | Same realm | Same realm only |
| Logout | Yes | Yes | Yes | Yes | Yes | Yes |

In the business UI, the person creating a requisition may be called the Hiring Manager. The authorization check for that action is nevertheless `REQUISITION_CREATOR`, not `HIRING_MANAGER`; the latter claim is created only with the user's own opening.

## 14. Data constraints and cardinality

- One organisation has many memberships; each membership has one user and can have many scoped role rows.
- One organisation has one active Organization Admin and one HR Reviewer under the current rule; enforce each with a partial unique constraint/transactional service rule (both are `job_opening_id = NULL`).
- One job opening has many job-specific participant assignments; a membership may have many job-opening assignments.
- A role verification request has one requesting membership and optional job opening; its job opening is mandatory only for job-specific role values (`HIRING_MANAGER`, `INTERVIEWER`) and is null for `ORGANIZATION_ADMIN`, `HR_REVIEWER`, and `REQUISITION_CREATOR`.
- An onboarding request has one requester and zero or one eventual organisation. Pending/rejected records have no organisation FK.
- An organisation has a lifecycle status (`ACTIVE`/`SUSPENDED`); a suspended workspace denies member login at authentication, enforced server-side (the toggle only records the state).
- A job opening has many applications; each application belongs to one applicant identity and opening.
- A requisition has one raising Hiring Manager and one owning organization; it carries an ordered stage list, each stage carrying ordered sections and questions plus a per-stage weight; submitted versions are immutable and resubmission creates a new version while preserving append-only review history.
- A job opening is created by the raising Hiring Manager after HR approves its requisition; it retains the requisition reference and carries the approved stage → section → question configuration and per-stage weightage.
- Capacity uses `available_to_select = target − reserved`, where `reserved` is committed capacity: selected/onboarding reservations plus completed fills. `filled` counts only `ONBOARDING_COMPLETED` records and is displayed separately as the completed-employment metric. Both values are transactionally maintained, non-negative, and auditable; a closed opening is retained read-only.
- An application's current stage is derived from the opening's configured stage sequence and the applicant's stage history; the sequence is data, never hardcoded.
- AI evaluation rows are immutable; the HR Reviewer's decision/comment is a separate record and never mutates the AI result.
- A canonical person (verified email) may have many realm-local identities; one realm identity maps to at most one canonical person; membership and application are **separate** relationships over the identity.
- Applying for a job creates a `job_applications` row but **never** an `organization_memberships` row.
- One applicant may hold at most one active application per job opening (duplicate-application prevention).
- Multiple Live Interview stages are distinct `requisition_stages` / `job_opening_stages` rows; each stage has its own assigned interviewers, evaluation criteria, and completion/advancement conditions (stage-scoped interviewer assignment — see section 25).
- Final selection reserves a headcount slot; the reservation is released on withdrawal/decline; over-fill is refused transactionally.

Required cross-row enforcement includes tenant matching for memberships/jobs/requests, `NULLS NOT DISTINCT` duplicate-role prevention where supported, and status transitions. Keycloak identities are referenced with `keycloak_user_id`; credentials and tokens are never persisted in this schema.

## 15. HR Reviewer authentication, workspace, and lifecycle

An HR Reviewer is not a separate identity or account. The lifecycle is:

~~~text
Registration (organization workspace)
→ Default Applicant / unverified state
→ Organization Admin verification (role_verification_requests)
→ Organization-level HR Reviewer assignment (organization_member_roles, job_opening_id = NULL)
→ Active HR Reviewer
~~~

Authentication is Keycloak-backed and organization-scoped: an HR Reviewer for Pente authenticates through the Pente organization realm, never another organization's realm. PostgreSQL never stores the initial or subsequent password; Keycloak handles first-login password setup/change.

After authentication the backend resolves the user's organization membership and role assignments to build the HR Reviewer workspace context:

- User context.
- Organization context.
- Active role context (defaulting to the first/primary assignment; switchable via the role switcher).
- Authorized Job Openings (all openings in the HR Reviewer's organization, by virtue of the organization-level role).
- Requisitions requiring HR Reviewer action.
- Applicant evaluations for the authorized openings.

Authorization is derived from organization membership plus job-specific role assignment, enforced per request. Cross-organization access is denied and audited.

## 16. Job requisition architecture

A requisition is an organization-owned, versioned business record that precedes the Job Opening:

~~~text
Hiring Manager → Create Requisition → Configure job requirements
   → Configure stages → sections → questions → stage weightage
   → Submit → HR Reviewer
HR Reviewer → Review → Approve
            OR
HR Reviewer → Return with comment → Hiring Manager edits → Resubmit → HR Review queue
~~~

Lifecycle states: `REQUISITION_DRAFT → REQUISITION_PENDING_APPROVAL → REQUISITION_APPROVED`, with `REQUISITION_RETURNED` for returns and `REQUISITION_WITHDRAWN` as its terminal withdrawal state. `ACTIVE → SUSPENDED → CLOSED` belongs to the Job Opening lifecycle, not the requisition. Each submission locks an immutable version; a returned requisition is revised and resubmitted as a new version.

Resubmitted requisitions take priority over newly submitted requisitions in the HR Review queue (previously-reviewed first, then new; within each group, most recent relevant date first).

The requisition carries the job requirements — role/title, position count, business justification, job description, expected experience, salary range, required skills, qualifications/requirements, job responsibilities, eligibility criteria, employment type, and location/work mode — plus the configured recruitment sequence. Delete and suspension are unsupported for requisitions. HR may raise an in-product withdrawal request; only the requisition creator executes or rejects it with an audited reason and notification.

## 17. Requisition stage → section → question configuration (data model)

The recruitment sequence is configurable data, never hardcoded. The hierarchy is:

~~~text
Job Requisition
    ↓
Stages (ordered, configurable)
    ↓
Sections (ordered, within a stage)
    ↓
Questions (ordered, within a section)
~~~

Stage types are predefined options — Pre-screening, Online Assessment, Live Interview, HR Interview, and Other (a custom name the Hiring Manager supplies) — but their presence and ordering are job-specific. A Developer requisition might be Pre-screening → Online Assessment → Live Interview → HR Interview, while a Sales requisition omits the Online Assessment. Each stage holds ordered sections; each section holds ordered questions. Example:

~~~text
Online Assessment
→ Self Introduction
→ Reading
→ Technical Questions
     ├── Question 1
     ├── Question 2
     ├── Question 3
     └── Question 4
~~~

The Hiring Manager configures this hierarchy while raising the requisition; the HR Reviewer reviews it before approving. The approved configuration is stored as data and drives the Job Opening's applicant pipeline (section 20).

For an `Other / Custom Stage`, configuration additionally stores the applicant-facing name/description, mode, conductor rule, evaluator role/count, total marks, passing mark, schedule/submission requirements, completion condition, and notification instructions. A passing mark is advisory decision support only; it never changes application state automatically. Each custom stage instance uses `NOT_STARTED → READY/SCHEDULED → IN_PROGRESS → SUBMITTED/AWAITING_EVALUATIONS → EVALUATION_LOCKED → UNDER_HR_REVIEW → ADVANCED/HOLD/REJECTED`.

## 18. Job Opening creation from an approved requisition

When HR approves a requisition, its authorised creator may create one draft Job Opening. Creation atomically adds that opening's `HIRING_MANAGER` assignment. The opening retains:

- Requisition reference.
- Job details (title, description).
- Position count (`required_candidates`).
- Salary range.
- Expected experience.
- Requirements / skills.
- The approved stage → section → question configuration and per-stage weightage, carried from the approved requisition (sections 17 and 27).
- Hiring Manager.
- Organization.

For each Live Interview stage, the Hiring Manager enters the required nominees' names and email addresses on the draft opening. The opening/panel package follows:

~~~text
DRAFT → PENDING_OPENING_APPROVAL → RETURNED / APPROVED_TO_PUBLISH
      → Hiring Manager Publish → PUBLISHED/ACTIVE → SUSPENDED / CLOSED
~~~

HR approves or returns the opening/panel package and approval sends interviewer invitations, but HR never publishes. Only the Hiring Manager who created the opening performs Publish. Suspension, reactivation, and manual closure are likewise executed only by that Hiring Manager against an in-product HR request. Applicants have no manual withdrawal action: missed configured response deadlines cause automatic application suspension; HR may request reactivation for the creating Hiring Manager to execute or reject.

The Job Opening is organization-owned and tenant-scoped. Organization Admins can monitor it but do not create, approve, publish, or configure it.

## 19. Position fulfillment and closure

Position fulfillment is tracked as:

~~~text
Requested = 2
Filled    = 1
Remaining = 1
~~~

Available capacity is derived/validated as `available_to_select = target − reserved`. `filled` counts completed onboarding only and is reported separately; it cannot be used to create another selection. The architecture rejects negative available capacity. Filling a position is a deliberate, separately recorded event (a `POSITION_FILLED` record linked to the application where applicable); advancing one candidate does not automatically fill or close a multi-position opening. A closed opening is retained read-only and never deleted.

## 20. Applicant pipeline and stage progression

The applicant pipeline is:

~~~text
Applicant → Job Opening → Current Stage → Stage History → Evaluation → HR Reviewer Decision → Next Stage / Rejected
~~~

The current stage and next stage are derived from the Job Opening's configured stage sequence (carried from the requisition), never from a hardcoded universal sequence. Each application records stage history; the HR Reviewer's per-stage decision advances the applicant to the next configured stage or rejects them (rejection makes future stages inaccessible).

## 21. AI and human evaluation architecture (immutability)

AI evaluation is immutable:

~~~text
Applicant submission → AI evaluation service → Persist AI result (immutable) → HR Reviewer reads result → HR Reviewer adds separate human comment → HR Reviewer approves/rejects
~~~

The HR Reviewer must not modify the original AI evaluation. AI-generated results and the human HR Reviewer decision/comment are stored separately (`ai_stage_evaluations` vs `hr_decisions`) and clearly distinguished in the UI.

For human-evaluated stages (for example Live Interview, HR Interview):

~~~text
Evaluator → Submit Evaluation → Persist Evaluation → HR Reviewer review → HR Reviewer decision
~~~

Live interviews are conducted through the organization's configured external meeting process (for example Google Meet); HireScope does not assume it hosts the meeting. The evaluator submits the interview result to HireScope, and the HR Reviewer reviews it. Multiple evaluators submit independent, attributable results that are not overwritten by one another or by AI.

## 22. Applicant notification architecture

Decisions trigger notifications through the application email service:

- Approve: HR Reviewer → decision → email service → Applicant (job opening, next stage, required action, instructions).
- Reject: HR Reviewer → decision/comment → email service → Applicant (decision/comment).

Notifications are sent from the verified organisation address with the responsible HR Reviewer as `Reply-To`. Individual mailbox credentials/passwords are never stored. The pre-screening qualification email carries the secure assessment link; a rejected/non-qualified applicant receives no assessment link.

## 23. Rejected applicant access

A rejected applicant can still log in and view their application status, but cannot access future assessment/interview stages. Once rejected, all future workflow actions (assessment entry, stage completion, progression) are inaccessible for that application. The applicant's view shows only their own application, its status, and the permitted HR Reviewer communication.

## 24. End-to-end data flow

~~~text
Requisition Creator → Requisition → HR Reviewer approval
    → Hiring Manager creates draft opening + panel nominations
    → HR opening/panel approval → Hiring Manager publishes → Applicant
    → AI / Human Evaluation → HR Reviewer → Applicant notification → Next Stage
    (…configured stages…) → Position fulfillment → Final outcome

Organization Admin → Member → approve organisation-level authority
Opening creation / HR panel approval → workflow-created job-specific assignments
    → same-realm HR Reviewer / Interviewer / Hiring Manager role switcher
~~~

## 25. Architectural decisions and remaining follow-up

The following product decisions are now resolved and override any older one-gate wording above: an approved Hiring Manager has limited requisition authority before a Job Opening exists; HR approval changes the requisition to approved but does not create an opening; the approved Hiring Manager then creates one opening, which carries the stage → section → question configuration and per-stage weightage the manager configured on the requisition. Organization Admins monitor openings and aggregate counts only. Items 1–3 below are retained as historical reconciliation notes and are no longer open decisions.

The following are retained as historical reconciliation notes. Applicant items 7–10 are resolved by section 43 and represented in the DBML:

1. **Resolved — requisition approval → opening approval → publication.** HR approves the requisition; its creator creates the draft opening and nominates the panel; HR approves/returns that package; only the Hiring Manager publishes after `APPROVED_TO_PUBLISH`.
2. **Resolved — where stages/sections/questions/weights are configured.** The Hiring Manager configures the full stage → section → question hierarchy and per-stage weightage on the requisition (sections 16–18 and 27). The opening carries the approved configuration; the opening refines assessment-delivery and pre-screen criteria without changing the approved sequence once applicants enter it (section 31). The DBML note that marks `requisition_stages`/`requisition_stage_sections` as "legacy/proposed — do not populate" is therefore reversed: requisition-level stage design is now authoritative.
3. **Resolved — requisition creator scope.** HR Reviewer and `REQUISITION_CREATOR` are organisation-scoped. `HIRING_MANAGER` is job-opening-scoped and is created when the authorised creator creates an opening from their approved requisition.
4. **"HR Interview" stage.** HR Interview remains a valid configurable stage (the Hiring Manager flow lists it among the predefined stage types); the final HR disposition (`ADVANCED`/`HOLD`/`REJECTED`) after `INTERVIEW_COMPLETED` remains the closing decision. No candidate-performed stage is assumed beyond what the manager configured.
5. **AI model/evaluation metadata.** The workflow and stories require model/configuration metadata on AI results; the supplied DBML (`ai_stage_evaluations`) omits a model-version column. The agreed evaluation shape is a single headline 0–100 score plus seven per-area 1–4 scores with reasoning (section 28).
6. **Resolved — notification sender identity.** The service sends from a verified organisation address and sets the responsible HR Reviewer/Organization Admin as `Reply-To`. Individual mailbox credentials are never stored.
7. **Resolved — canonical identity / cross-realm linking.** `canonical_people` owns the verified unique email; `users` holds realm-local Keycloak identities linked to that person. The same person may be Applicant in one organisation and staff in another.
8. **Resolved — multi-stage Live Interview assignment.** `stage_interviewers` provides stage-scoped panel assignment and immutable per-attempt submissions, permitting distinct Live Interview panels and reattempts.
9. **Resolved — final selection and onboarding.** `application_selections` reserves capacity at selection; `onboarding_records` tracks HR updates from the external onboarding process; `position_fills` is created only at onboarding completion.
10. **Resolved — notification and audit history.** `notification_outbox` provides transactional delivery/retry history and `audit_events` provides append-only transition history.

## 26. Hiring Manager role, workspace switcher, and dashboard

A Hiring Manager is a job-opening-scoped responsibility, not a permanent organisation role. Before an opening exists, the person uses the separate organisation-level `REQUISITION_CREATOR` capability. An organisation can have many requisition creators/Hiring Managers, and one user can hold many assignments inside that realm.

After login the backend resolves every context available inside the current realm and renders a **same-realm role switcher**:

~~~text
Pente realm
 ├── Requisition Creator / Hiring Manager
 └── Interviewer
~~~

Switching from Pente Hiring Manager to Pente Interviewer changes the sidebar and data scope without leaving the realm. Changing to an Applicant account in ABC Corp requires logout, opening the ABC workspace URL, and authenticating against the ABC realm. The verified email may be the same, but realm-local Keycloak user IDs and passwords may differ; there is no cross-realm SSO.

The Hiring Manager dashboard, when operating in a Hiring Manager context, shows the Hiring Manager profile, the organisation/workspace, the active role/context, requisitions, job openings, recruitment progress, and pending actions. Summary cards include My Requisitions, Pending Approval, Active Job Openings, Total Applicants, Applicants in Progress, and Positions Remaining.

## 27. Stage weightage and the no-composite rule

Different jobs prioritise different stages. The Hiring Manager configures a per-stage weight when raising the requisition; for example a Developer requisition may weight Pre-screening 20%, Online Assessment 25%, Live Interview 40%, and HR Interview 15%, while a Sales Executive requisition weights Pre-screening 20%, Online Assessment 15%, Live Interview 25%, and HR Interview 40%. The system validates the agreed rule (for percentage weights, the total must equal 100%).

Stage weights **rank and guide prioritisation only**. They are never combined into a single composite hire score, and no threshold derived from any aggregate may automatically advance, hold, or reject a candidate. Every progression decision is a human decision recorded against a specific stage (section 30). This preserves the mockup's "no composite hire score / no silent threshold" rule alongside the manager-configured weighting.

## 28. Fair-hiring principles

The following principles are binding on evaluation design and are carried through the requisition, opening, and applicant-review UIs:

- **Blind review.** Before the AI/evaluator result locks, résumé and "distance traveled" context are hidden from every reviewer. After the AI pre-screen result locks, HR may see them for its decision. Only after that HR decision locks may the assigned Hiring Manager see them read-only. Interviewers, Organization Admin, and Platform Admin do not receive them.
- **Seven job-task areas.** AI evaluation reports a single headline 0–100 score plus seven per-area 1–4 scores with reasoning: Task, Thinking, Follow-through, Learning, Judgment, People, and Integrity. Areas score evidence of doing the job, not school, years, or employer brand.
- **"Do not use as a gate."** The requisition/opening records an explicit anti-gate statement (for example FAANG, CS degree, 7+ years, or employment gaps must not be used as gates).
- **Distance traveled — context only.** Gaps, bootcamps, military or caregiving history, and the absence of brand-name employers are context shown after lock and never scored.
- **Voice integrity.** Spoken answers are compared against the applicant's own introduction (natural baseline) and reading (read-aloud baseline). A mismatch raises a flag for a human to listen to — never an automatic failure or score change. A typed interview is an explicit accommodation.
- **Human decision, no AI editing or auto-decide.** A reviewer records an attributable decision/comment separately and never edits an AI per-area score; AI never changes application state or makes a decision.

Visibility is enforced in the query/response layer, not by CSS masking:

| Event | HR Reviewer | Hiring Manager | Interviewer | Org/Platform Admin |
|---|---|---|---|---|
| Submission received; result not locked | Résumé/context hidden | Hidden | Hidden | Hidden |
| AI pre-screen result immutable/locked | Visible for HR decision | Hidden | Hidden | Hidden |
| HR pre-screen decision immutable/locked | Visible | Visible read-only for assigned opening | Hidden | Hidden |
| Application manually closed by creating HM | Full historical assessment, immutable AI result, evaluator evaluations, HR decision, and confidential HR note during retention/legal hold | Read-only historical assessment, immutable AI result, evaluator evaluations, and applicant-visible HR decision for the assigned opening; confidential HR note denied | No historical assessment, AI, peer-evaluation, or HR-decision access; own submission receipt only where policy requires | SLA/audit metadata only |

## 29. Hiring Manager applicant review and comment separation

After the relevant HR decision locks, the Hiring Manager can view, for applicants of their assigned openings: the released applicant identity/context, job opening, current stage, application status, stage history, AI evaluation, stage scores, evaluator scores, applicant responses, questions presented, and permitted evaluator comments. Before that release event the API omits résumé/distance-travelled fields. The Hiring Manager cannot modify any evaluation result.

The Hiring Manager can add one thing: a **Hiring Manager Comment** (an observation such as "Strong Java and backend experience — good fit for our current requirements"), distinct from the official workflow decision. This is a separate record and can never become an HR decision.

## 30. Evaluation and decision records (never merged)

Six records are kept distinct and never collapsed into one generic comment field:

1. **AI Evaluation** — immutable advisory result (headline score plus seven area scores and reasoning).
2. **Evaluator Evaluation** — immutable, attributable per-interviewer rubric.
3. **HR Decision** — the official progression decision, owned by the HR Reviewer (`QUALIFIED_FOR_ASSESSMENT`, `LIVE_INTERVIEW_RECOMMENDED`, `ADVANCED`, `REJECTED`, `HOLD`, etc.).
4. **Confidential HR Note** — exactly one HR-only explanatory/internal note attached to a decision.
5. **Applicant-visible Status Message** — exactly one status-only message attached to the HR decision and used in the applicant notification; it cannot contain the confidential note.
6. **Hiring Manager Comment** — the Hiring Manager's observation, read-only for evaluation purposes and never a decision.

A Hiring Manager observation ("strong React skills, suitable for our team") is never mistaken for an official decision ("approved for next stage"), which is owned exclusively by the HR Reviewer.

## 31. Stage configuration locking after applicants enter

Once any applicant has entered a stage, that stage's configuration — questions, sections, order, weightage, and evaluation criteria — is locked. Editing it after applicants have been evaluated against it would make historical evaluations inconsistent. The system therefore either locks the stage, or requires a new revision while preserving the configuration actually used when each applicant was evaluated. This applies especially to questions, sections, stage order, scoring/weightage, and evaluation criteria.

## 32. HR Reviewer vs Hiring Manager capability boundary

| Capability | HR Reviewer | Hiring Manager |
|---|---|---|
| Create requisition | No | Yes |
| Edit own returned requisition | No | Yes |
| Approve requisition | Yes | No |
| Reject / return requisition | Yes | No |
| Configure stages | Review | Yes |
| Configure sections / questions | Review | Yes |
| Configure stage weightage | Review | Yes |
| View applicants | Yes | Yes (assigned opening) |
| View AI evaluation | Yes | Yes |
| Edit AI evaluation | No | No |
| View applicant responses | Depending on stage / business rule | Yes (assigned opening) |
| View evaluator scores | Yes | Yes |
| Edit evaluator scores | No | No |
| Add own comment | Yes (HR comment) | Yes (Hiring Manager comment) |
| Make applicant stage decision | Yes | No |
| Request requisition withdrawal | Yes | No |
| Execute own requisition withdrawal | No | Yes |
| Create draft Job Opening from approved requisition | No | Yes |
| Approve/return opening + panel package | Yes | No |
| Publish approved Job Opening | No | Yes (creator only) |
| Request opening suspend/reactivate/close | Yes | No |
| Execute opening suspend/reactivate/close | No | Yes (creator only) |

This is the RBAC boundary preserved in the architecture.

## 33. Interviewer role, participant model, and tenancy

An Interviewer is a **job-opening-scoped participant**, not a permanent organisation role and not a separate identity. Being an interviewer for one requisition grants access only to that organisation and that requisition's Live Interview stage; it confers no access to other requisitions, applicants, organisations, or interview processes.

An activated Interviewer is represented by the existing `users` (Keycloak) identity plus `organization_memberships` and a job-opening-scoped `organization_member_roles` row with `role = INTERVIEWER`. There is no `InterviewerUser` / `InterviewerAccount` / `InterviewerOrganization` table. Before a new nominee has a realm account, only a pending opening/stage/email nomination exists; no placeholder user, membership, or role is created.

~~~text
Organization
    ↓
Job Opening (created from the approved requisition)
    ↓
organization_member_roles (role = INTERVIEWER, job_opening_id)
    ↓
Interviewers
~~~

The Hiring Manager nominates interviewers by name and email for each Live Interview stage on the draft opening. HR approves or returns the opening/panel package. Approval changes the opening to `APPROVED_TO_PUBLISH` and sends each nominee a one-time Keycloak action link from the verified organisation address with HR as `Reply-To`. Existing realm members link immediately; new invitees must authenticate/register within 24 hours, after which the service creates/links membership and the opening role. Invitation states are `NOMINATED`, `HR_APPROVED`, `INVITED`, `ACTIVATED`, `EXPIRED`, `REPLACED`, and `REVOKED`. Expiry notifies HR and the Hiring Manager for resend/replacement; it never silently grants access.

## 34. Live Interview configuration data model

The Hiring Manager configures the Live Interview on the requisition (carried into the opening on HR approval, sections 16–18):

- **Enabled** flag.
- **Required interviewer count** (e.g. `2`).
- **Interviewers are not named here** — the requisition carries only the enabled flag, required count, and evaluation sections. Names/emails are nominated per stage on the draft opening and approved by HR.
- **Evaluation sections + weightage** — e.g. Technical Knowledge 40%, Communication Skills 20%, Problem Solving 20%, Role Fit 20% (total 100%). There are **no preset interview questions** for the Live Interview; preset questions belong only to the Online Assessment stage.

The evaluation sections reuse the existing scoring model (criteria, sections, weightage, maximum score, rating scale) where one exists rather than duplicating it. The configured framework is visible to interviewers before they conduct the interview, and its configuration locks or revisions once an applicant enters the stage (section 31).

## 35. Live Interview scheduling and status model

Only the HR Reviewer marks an applicant **Eligible for Live Interview** and schedules the event. Scheduling requires: the Live Interview is configured, the required interviewers exist, the applicant is eligible, a valid date/time is provided, and the applicant has not already completed the Live Interview. On confirmation the system records the schedule on the applicant's stage result (`stage_results.scheduled_date`/`scheduled_time`/`status`) and notifies the applicant and every assigned interviewer.

Status models (preserving existing names such as `INTERVIEW_COMPLETED` where already defined):

- Individual interviewer evaluation: `PENDING`, `IN_PROGRESS`, `COMPLETED`.
- Overall Live Interview: `SCHEDULED`, `IN_PROGRESS`, `PENDING_EVALUATIONS`, `COMPLETED`, `CANCELLED`.

## 36. Interviewer authorization and information visibility

Authorization is enforced **server-side** (never only in the frontend). An interviewer can access an applicant evaluation only when:

~~~text
Authenticated User AND User is assigned as interviewer
  AND Interviewer belongs to the correct organisation context
  AND Applicant belongs to the assigned requisition
  AND Applicant is eligible/scheduled for Live Interview
~~~

An interviewer receives `403 Forbidden` for an unauthorized applicant or requisition, and unauthorized data is never exposed through API responses.

Information visibility is strict and enforced in the query/response layer:

- **NOT visible to interviewers** — AI-generated scores, recommendations, evaluation results, confidence scores, ranking, classification, internal ATS decision data, and any other interviewer's evaluation/score/comments.
- **Visible to interviewers** — applicant information required for the interview, job information, Live Interview configuration, previous-stage HR comments, the Live Interview schedule, other interviewers' names, and other interviewers' completion status (status only, never scores).

## 37. Interviewer evaluation, validation, and immutability

The evaluation form is generated dynamically from the Hiring Manager's configured sections (weightage + score fields + overall comment). The interviewer cannot modify scoring criteria, weightage, interviewer assignment, schedule, or applicant eligibility.

Before accepting an evaluation the backend validates: every required section is scored, scores are within range, required comments are present (where specified), the user is an assigned interviewer, the user has not already submitted, and the Live Interview belongs to the applicant/requisition being accessed. Duplicate submission is prevented by the `UNIQUE (application_id, stage_id, interviewer_membership_id)` constraint on `stage_interviewers`. After submission the evaluation is immutable (unless an authorised edit/reopen mechanism exists), and the interviewer sees `Your Evaluation: Completed`.

## 38. Completion, aggregation (no composite), and HR handoff

When all required interviewers submit, the Live Interview becomes `COMPLETED` and the applicant becomes eligible for the next HR review/decision stage; HR is notified that the applicant is ready for review. Individual interviewer scores are stored separately and are never exposed to other interviewers. If the product already defines an aggregation formula it is applied; otherwise no new hiring-decision formula is invented — HR reviews the individual evaluator records. Any aggregate is computed from the configured scoring model while preserving the individual evaluator records.

## 39. APIs and service boundaries (interviewer)

The existing API conventions are extended, not duplicated:

- **Requisition/opening** — configure Live Interview requirements on the requisition; nominate names/emails per stage on the draft opening; submit opening/panel package; HR approve/return; Hiring Manager publish.
- **Interviewer** — get assigned requisitions, get assigned applicants/interviews, get interview configuration, get authorized applicant information, get previous HR comments, get evaluation form, submit evaluation, get own evaluation status.
- **HR** — approve/return the opening/panel package, trigger one-time invitations, mark applicant eligible, schedule Live Interview, view completion status, and view submitted evaluations.

Every interviewer endpoint resolves `interviewer_id`, `organization_id`, `requisition_id`, and `application_id` from the authenticated session — never from frontend-supplied values — preventing IDOR, cross-organisation access, cross-requisition access, cross-applicant access, unauthorized score access, and unauthorized evaluation modification.

## 40. Notifications, audit, and data constraints

Notifications are emitted for HR panel approval/invitation, 24-hour expiry, resend/replacement, applicant eligibility + scheduling, interviewer evaluation submission, all-evaluations-completed, and required-evaluator replacement. They use the verified organisation sender with the responsible HR Reviewer as `Reply-To`; no individual mailbox credential or plaintext password is stored. No interviewer is notified with another interviewer's score.

The audit framework records: interviewer assigned, interviewer invitation sent, applicant marked eligible, Live Interview scheduled/rescheduled, interviewer opened evaluation, interviewer submitted evaluation, Live Interview completed, missing-evaluator detection, HR-approved evaluator replacement, retake scheduled, and SLA escalation state.

Data constraints require a pending nomination/invitation record keyed by opening + stage + normalized email because an unregistered nominee has no membership. Activation links it to the created/existing membership and opening role. Applicant-specific submissions remain in generalized `stage_interviewers`, with per-section scores in `interview_results`; only activated assigned interviewers can submit. Removing an interviewer revokes future access but never deletes an already-submitted evaluation.

## 41. Security and testing requirements

The backend must treat `interviewer_id`, `organization_id`, `requisition_id`, and `application_id` as untrusted, and prevent IDOR, cross-organisation, cross-requisition, cross-applicant, and unauthorized-score access and unauthorized evaluation modification.

Required tests:

- **Unit** — interviewer authorization; interviewer assignment; scoring configuration; HR eligibility transition; scheduling; evaluation submission; duplicate-submission prevention; score validation; multi-interviewer completion; unauthorized score access; cross-organisation access; cross-requisition access; interviewer removal; cancelled-interview handling.
- **Integration** — the complete flow from requisition creation → Live Interview configuration → interviewer assignment → notification → applicant progression → HR eligibility → scheduling → applicant/interviewer notification → interviewer login → scoring framework + authorized HR comments → submission → second interviewer submission → Live Interview COMPLETED → HR review.
- **Security** — interviewer A cannot view interviewer B's score; cannot access another requisition's/organisation's applicant; cannot change scoring weightage; cannot mark an applicant eligible; cannot approve/reject an applicant; cannot submit for an applicant to whom they are not assigned.

## 42. Applicant lifecycle architecture

This section finalises the Applicant lifecycle end to end — workspace entry through onboarding — and extends (without contradicting) the applicant pipeline, evaluation, notification, and rejected-access architecture in sections 20–24.

### 42.1 Workspace-scoped entry and canonical identity

Entry is workspace-scoped (section 1). The workspace URL selects the organisation; the backend resolves `organization_id` server-side and never trusts a client-supplied value. Each organisation authenticates in its own Keycloak realm, so there is **no cross-realm SSO**. Canonical identity is the verified email address; a known email is linked to its existing canonical person across realms rather than duplicated (representation is decision 7 in section 25).

### 42.2 Applicant profile vs application vs membership

Three distinct concepts, never collapsed:

- **Canonical person / realm identity** — who the person is (Keycloak).
- **Membership** (`organization_memberships`) — an organisation relationship granted only by verification/assignment; **never** created by applying.
- **Application** (`job_applications`) — a job-opening relationship; organisation scope is derived from the opening.

Applying creates only the application. An applicant is not an organisation member and never appears in the confirmed-members list.

### 42.3 Configurable pre-screening form (application intake)

Applying is the Pre-screening stage's intake. The form is generated dynamically from the requisition's configured stage → section → question hierarchy (section 17); nothing is hardcoded. Fields (identity, education, experience, eligibility questions, résumé upload) exist only as configured. The submitted responses are preserved with the configuration actually used (section 31).

### 42.4 Application submission, HR pre-screen queue, and duplicate prevention

Submission creates/updates the application through `INTEREST_REGISTERED → APPLICATION_DETAILS_REQUIRED → RESUME_SUBMITTED`, records a confirmation, and places the application in the HR pre-screen review queue. Duplicate application for the same applicant + opening is rejected server-side. AI pre-screening then runs (advisory, immutable); the HR Reviewer reviews and records `QUALIFIED_FOR_ASSESSMENT` / `PRE_SCREEN_PENDING` / `NOT_CURRENTLY_QUALIFIED`.

#### 42.4.1 Controlled bulk pre-screen decisions

An authorised HR Reviewer may apply one pre-screen status to several reviewed applications as a queue convenience. Bulk action is permitted only when every selected application belongs to the same organisation, job opening, opening/configuration version, and Pre-screening stage, and is currently `PRE_SCREEN_ANALYSIS_READY` or `UNDER_HR_PRE_SCREEN_REVIEW`. Applications with processing failures, grievances, unresolved discrepancy/attention flags, stale versions, an existing locked HR decision, or any other stage/status are excluded and must be reviewed individually. Selection is by explicit application ID; "all matching" or hidden-page selection is never inferred from a filter.

Bulk processing never creates a shared decision. The service first re-authorizes and validates the complete selection against current database state, then materializes one attributable `hr_decisions` record, one confidential HR note, one applicant-visible status message, one application transition, one audit event, and one notification-outbox record per application. A batch/correlation ID links those individual records for operational traceability. The acting HR Reviewer must attest that every selected résumé and advisory analysis was reviewed; AI recommendations, scores, filters, and ranking never select or approve applicants automatically.

The supported bulk dispositions are `QUALIFIED_FOR_ASSESSMENT`, `PRE_SCREEN_PENDING`, and `NOT_CURRENTLY_QUALIFIED`. The shared note/message entered in the bulk form is copied into each application's own immutable decision record. For `QUALIFIED_FOR_ASSESSMENT`, a valid scheduled start, deadline, and permitted duration are required and a distinct secure invitation is created for every application. The entire batch is rejected before mutation if selection scope, authorization, eligibility, required text, or invitation timing fails for any application. The accepted batch writes all individual transitions and outbox records atomically and is idempotent by batch key, so a retry cannot duplicate decisions or notifications. A rolled-back batch sends nothing; delivery retries remain per applicant.

`POST /applications/pre-screen-decisions:bulk` accepts explicit application IDs, the expected opening/configuration version and current status, disposition, note/message, optional assessment timing, and an idempotency key. Its response reports the batch ID and each application decision/notification identifier. Authorization and optimistic-concurrency checks occur server-side; frontend selection is never trusted. Bulk selection, validation failure, confirmation, completion, and every affected application ID are audited. Persistence must retain the correlation ID on the decision/audit relationship; the recruitment DBML requires a corresponding batch-link field or relation before this endpoint is claimed as implemented.

### 42.5 Online Assessment lifecycle and version preservation

Before `ASSESSMENT_INVITED`, HR must set `scheduled_start_at`, `deadline_at`, and `permitted_duration`; the API refuses invitation when any value is missing. The timer begins only after readiness succeeds and the applicant actively starts, and ends at the earlier of duration expiry and the fixed deadline. In-progress attempts survive interruption; submitted responses preserve the version used.

### 42.6 AI evaluation, HR review, and the no-auto-decide rule

After submission, `AI_PROCESSING` runs, then `ANALYSIS_READY` or `AI_PROCESSING_FAILED` (retried; never a false rejection). The AI result is immutable and advisory; the HR Reviewer reviews it and records `LIVE_INTERVIEW_RECOMMENDED` / `LIVE_INTERVIEW_HOLD` / `NOT_PROCEEDING`. AI never changes application state or makes a decision; every progression is a human decision (sections 21, 28, 30).

### 42.7 Multiple Live Interview stages

"Live Interview" is **not** assumed to be a single stage. A requisition may configure several Live Interview stages (for example Live Interview 1 with panel A + B, Live Interview 2 with panel C + D), each as its own `requisition_stages` / `job_opening_stages` row with its own enabled flag, required interviewer count, evaluation criteria, and completion/advancement conditions. Each stage reuses the Live Interview scheduling/status/evaluation model of sections 34–40; a later stage becomes available only after the applicant satisfies the previous stage's advancement conditions (an HR decision). Stage-scoped interviewer assignment (so panel A is on Live Interview 1 only) is decision 8 in section 25.

### 42.8 HR Interview as a distinct stage

HR Interview is a distinct configurable stage (sections 17, 30) — never merged with the HR review/decision activity that follows a stage. The stage produces an evaluation; a separate HR decision then advances or rejects. It is scheduled and evaluated like any human-evaluated stage.

### 42.9 Final selection and onboarding

After all configured stages are complete and the applicant is `ADVANCED`, the authorised HR decision-maker records final selection:

~~~text
ADVANCED → SELECTED → ONBOARDING_PENDING → ONBOARDING_COMPLETED → POSITION_FILLED
ADVANCED → NOT_SELECTED / REJECTED
~~~

Selection records the decision-maker and timestamp and reserves a headcount slot. HR records external offer acceptance/decline and updates recruitment-side onboarding status; only `ONBOARDING_COMPLETED` creates `POSITION_FILLED`. Decline, failure, or cancellation releases the reservation. Onboarding is a status/workflow trigger, not another interview stage.

### 42.10 Headcount and vacancy tracking

Headcount is tracked as target vs reserved/selected vs onboarding-pending vs filled vs remaining. Selection is refused transactionally when no vacancy remains; external-offer decline, onboarding failure, or cancellation releases the reservation. Each transition is separately recorded and auditable; filling a position is never an automatic consequence of advancing a candidate.

### 42.11 Notification architecture

A notification is emitted at every applicant-visible persisted transition. It is sent from the verified organisation address with the responsible HR Reviewer as `Reply-To`; individual mailbox credentials are never stored. A rolled-back transition must not send a contradictory email, and a failed send is retried without corrupting application state.

### 42.12 Data integrity, authorization, and tenant isolation

All applicant endpoints resolve `organization_id`, `job_opening_id`, and `application_id` from the authenticated session and the opening graph — never from the frontend. Cross-tenant access is denied and audited; IDOR is blocked in the query/response layer. The applicant sees only their own application: no other candidate's data, no AI scores, no reviewer comments. Applying never grants membership.

### 42.13 Error handling and retries

- **AI processing failure** — retried; the applicant is never falsely failed.
- **Assessment interruption** — attempt preserved; resume or grievance path.
- **Notification send failure** — retried without corrupting state.
- **Headcount over-fill** — selection refused transactionally.

### 42.14 Testing requirements

- **Unit** — workspace→organisation resolution; cross-realm isolation; canonical-identity linking; apply-creates-no-membership; duplicate-application prevention; dynamic pre-screen form generation; bulk pre-screen eligibility/same-opening-version validation; batch idempotency and optimistic concurrency; assessment attempt preservation; AI-processing retry; headcount reservation/over-fill refusal; final-selection transaction; withdrawal releases reservation.
- **Integration** — workspace entry → registration → job discovery → apply (configurable form) → confirmation → individual or controlled bulk pre-screen decision → assessment invitation/readiness/submission → AI processing → HR review → Live Interview stage(s) → HR Interview → final selection → onboarding initiated, with a notification at each transition. Bulk success creates one decision/outbox item per application; any pre-validation failure creates none.
- **Security** — applicant cannot reach another applicant's data, submit against a client-supplied organisation ID, access a rejected/closed stage, self-advance, see AI scores or reviewer comments, or be granted membership by applying. Bulk requests cannot cross tenant/opening/version boundaries, include ineligible application IDs, rely on filter-defined selection, or bypass per-application authorization and audit.

## 43. Adopted applicant architecture decisions (2026-10-09)

This section supersedes unresolved Applicant items in section 25 and is the implementation authority for the DBML and frontend.

### 43.1 Canonical person and realm identities

`canonical_people` owns globally unique verified email. `users` remains a realm-local Keycloak identity and references exactly one canonical person. A person may therefore have one user ID in Pente and another in CarePlus while retaining one person record. Duplicate registration is rejected only within the same organisation; Applicant access in one organisation must never block a staff membership/role in another.

### 43.2 Opening and application lifecycle

`job_applications` stores `canonical_person_id` and permits only one active application per applicant/opening. Public closing dates stop new applications but do not interrupt active ones. Applicants have no self-withdrawal action: each actionable stage has a configured response deadline and missed response creates `APPLICATION_SUSPENDED_NO_RESPONSE`, releases any reservation, and blocks later-stage access. HR may raise a reactivation request; only the creating Hiring Manager executes or rejects requisition withdrawal, opening suspension/reactivation/closure, and reactivation requests with an audited reason. Ordinary prior closed applications never reopen or permit a replacement. The sole exception is a reasoned `VALID_WITHDRAWAL_OR_INCOMPLETE`: once `retest_eligible_at` is reached, the service may create a linked replacement application for an eligible open position without overwriting its original. Final vacancy fill automatically closes the opening and remaining active applications.

### 43.3 Assessment execution and versioning

Assessment attempt timing belongs on `stage_attempts`: `scheduled_start_at`, `deadline_at`, and `permitted_duration_minutes`, not `updated_at`. The timer starts after readiness and active start. The attempt stores an immutable configuration version selected at start; invitations not yet started use the latest approved version. HR-approved reattempts are separate attempts with the latest configuration, a new schedule, and a configured reattempt allowance. AI failure holds the application pending until a successful advisory result exists.

The current client requirements are desktop/laptop Google Chrome plus successful microphone permission/playback, unless HR records an approved accommodation. The full accommodation request is HR-restricted. Interviewers receive only the minimum operational instruction required to conduct their assigned stage; the adjustment never changes a score or progression automatically.

Assessment attempts record one terminal outcome: `COMPLETED_SUCCESSFUL`, `COMPLETED_UNSUCCESSFUL`, or `TEST_STARTED_NOT_COMPLETED`. HR may additionally classify an incomplete attempt as `VALID_WITHDRAWAL_OR_INCOMPLETE` only with an internal reason. That classification ends the old application attempt and sets `retest_eligible_at = closed_at + 6 months`; it never reopens the old application. A later retest/reapplication must target an eligible open position and pass normal duplicate checks.

### 43.4 Interview attempts and HR Interview

`stage_interviewers` is the authoritative stage-scoped participant record. Its uniqueness is `(application_id, stage_id, interview_attempt_number, interviewer_membership_id)`. A missing evaluator triggers an hour-23 HR reminder and, at 24 hours, HR may create a reattempt with a replacement panel and applicant schedule. HR may act earlier when a valid reason is recorded. The replacement requires a new evaluation and writes the confidential retake note: “This user retook the interview approved by HR because the previous interview could not be evaluated for a valid reason.” The system alerts the assigned Hiring Manager, exposes the event to HR, adds it to the Organization Admin SLA report, and starts the configured escalation procedure. Original evaluations remain immutable audit history; only the retake evaluation set is eligible for stage progression. HR cancellation/reschedule retains applicant eligibility and invalidates the cancelled session. An applicant may submit an in-product grievance with a reason within 24 hours after the relevant configured stage deadline; after that the grievance window is rejected and the configured no-response/closure rule applies.

HR Reviewer conducts/evaluates HR Interview and may make its subsequent progression or final decision, but the evaluation and HR decision remain distinct records. Interviewers receive only authorised applicant information and applicant-visible previous HR messages; AI and peer-evaluator data remain server-side denied.

### 43.5 Selection, onboarding, notifications, retention, and audit

`application_selections` reserves capacity atomically at `SELECTED`; `position_fills` is created only when `onboarding_records.status = ONBOARDING_COMPLETED`, which preserves that committed capacity while incrementing the separately reported filled count. HR records external offer acceptance/decline and recruitment-side onboarding status. Decline, no-response suspension, onboarding failure, or cancellation releases an unfilled reservation and closes/suspends the application. Headcount queries calculate target, reserved, filled, and `available_to_select = target − reserved` under a row lock/transaction to prevent overfill.

Every applicant-visible transition writes an outbox record in the same transaction. The sender is the verified organisation address and the responsible HR Reviewer is `Reply-To`; retry/failure cannot undo the state transition. Every HR decision persists exactly one confidential HR note plus exactly one applicant-visible status message; only the latter is placed in the outbox payload. `audit_events` is append-only. On a manual HM closure, HR retains full historical evidence; the assigned HM retains read-only historical assessment/AI/evaluator evidence and applicant-visible decisions for that opening; interviewers receive no historical evidence or decision access; Organization Admin receives only SLA/audit metadata. Non-selected records are scheduled for deletion one year after closure, and selected recruitment records at employment termination; legal holds override deletion and restrict access while active.

### 43.6 Workspace availability and account scope

Invalid, suspended, inactive, and unauthorised workspace URLs return one generic **Workspace unavailable** response; details remain audit-only. HR may permanently deactivate an applicant only within HR's organisation. The action preserves history, blocks that organisation's access, and never affects other realm identities.

## 44. Cross-role architecture decisions adopted 2026-10-12

This section is binding where earlier architecture wording conflicts.

1. **Two HR checks; Hiring Manager publishes.** The business Hiring Manager uses the organisation-level `REQUISITION_CREATOR` claim to create the requisition. HR first approves the versioned requisition; its creator then creates the draft opening and stage-specific panel nominations. HR approves/returns that package. Approval sets `APPROVED_TO_PUBLISH` and sends invitations; only the opening's `HIRING_MANAGER` performs Publish.
2. **Separate authority scopes.** `REQUISITION_CREATOR` is organisation-scoped and can create/submit requisitions only. `HIRING_MANAGER` is a distinct job-opening-scoped claim created when that authorised person creates an opening from their approved requisition.
3. **Simple interviewer invitation status.** A pending record keyed by opening + stage + normalized email stores the name, HR decision, and 24-hour deadline. The HR screen shows only `Invited`, `Activated`, or `Expired`; it is not a user or membership. Keycloak activation creates/links the realm identity/membership and opening role. Expiry notifies HR and the Hiring Manager, who may re-invite or use the normal replacement approval flow.
4. **Same-realm role switching.** The role switcher never changes realm. Changing organisation requires logout and authentication in the other workspace realm; the same canonical email does not imply the same Keycloak ID or password.
5. **Blind-review release.** Résumé/distance-travelled context is omitted until the AI/evaluator result locks; HR receives it for decision, and the Hiring Manager only after the HR decision locks. Interviewers and administrative monitoring roles never receive it.
6. **Mandatory assessment timing.** HR must set start, deadline, and permitted duration before invitation. The timer begins after readiness and active start and ends at the earlier of deadline/duration expiry.
7. **Custom stages.** Stage name, mode, conductor, evaluator/count, sections/questions/criteria, total marks, passing mark, weightage, timing/submission needs, and completion condition are data. Passing marks never auto-progress an application.
8. **Controlled actions and no-response suspension.** HR raises requisition/opening requests. The creating Hiring Manager alone executes/rejects withdrawal, suspension, reactivation, and closure requests in an in-product inbox with audit and notification; HR never directly closes an opening. Applicants have no self-withdrawal action: after a configured stage-response deadline, the application becomes `APPLICATION_SUSPENDED_NO_RESPONSE` and releases any reservation. HR may request reactivation, which only the creating Hiring Manager executes/rejects.
9. **Service-owned mail and Keycloak links.** Mail uses the verified organisation address with the responsible reviewer/admin as `Reply-To`. Account invitations use single-use expiring Keycloak action links; plaintext/default passwords and individual mailbox credentials are never stored or emailed.
10. **Headcount capacity.** `available_to_select = target − reserved`; `filled` is only completed onboarding. Selection reserves capacity transactionally; decline, automatic no-response suspension, failure, or cancellation releases it.
11. **Super Admin ownership.** Superseded by section 45: Super Admin is the buyer/owner scoped by an explicit organization ownership record. It initiates Organization Admin appointment/replacement for that organization; it is neither Platform Admin nor a sole-HR-Reviewer recovery authority and has no recruitment-operation permissions.
12. **Accommodation and grievances.** HR sees full accommodation details and gives interviewers only the operational instruction required for their stage. Applicants may submit an in-product grievance with a reason within 24 hours after the relevant configured stage deadline; HR records the result and any reattempt.
13. **Manual-closure retention access.** During retention/legal hold after manual HM closure, HR retains full historical assessment, AI, evaluator, HR-decision, and confidential-note access. The assigned HM retains read-only historical assessment/AI/evaluator evidence and applicant-visible HR decisions for that opening. Interviewers receive none of those historical records; Organization Admin receives SLA/audit metadata only.
14. **Two HR-decision communications.** Every HR decision has exactly one confidential internal HR note and exactly one applicant-visible status message. Notifications use only the status message; all non-HR endpoints must deny the confidential note.
15. **Missing required interviewer.** HR may record a valid reason, replace the required evaluator, and require one new evaluation/retake. The confidential retake remark, HM alert, HR visibility, Organization Admin SLA event, and escalation state are mandatory. Only the retake evaluation set governs progression; earlier submissions remain immutable.
16. **Assessment outcomes and retest.** Attempts are `COMPLETED_SUCCESSFUL`, `COMPLETED_UNSUCCESSFUL`, or `TEST_STARTED_NOT_COMPLETED`. A reasoned `VALID_WITHDRAWAL_OR_INCOMPLETE` sets a six-month retest eligibility date; it never resumes the prior application.

Required DBML follow-up: introduce the opening approval/publication state machine, distinct `REQUISITION_CREATOR` and `HIRING_MANAGER` claims, pending stage-panel invitation data, 24-hour activation status, custom-stage execution fields, controlled-action request records, automatic no-response suspension, headcount reservation/available-capacity fields, Super Admin authority handover records, HR decision confidential-note/status-message fields, evaluator-replacement/SLA escalation records, manual-closure retention ACLs, and assessment outcome/retest eligibility fields plus a linked replacement-application relationship before implementation claims schema completeness.

## 45. Super Admin ownership and Organization Admin appointment architecture

This section supersedes the platform-level `SUPER_ADMIN` recovery wording in sections 1, 13, and 44.11. It is a proposed extension to the documented architecture and DBML; no new table, endpoint, worker, or Keycloak configuration is claimed as implemented merely by this document.

### 45.1 Authority and identity model

The Super Admin is the buyer/business owner of an explicitly authorized organization, not an identity-provider administrator. The owner authenticates through the existing **onboarding** Keycloak realm before a tenant exists and continues to use that control-plane identity after provisioning. This is intentionally separate from:

- **Platform Admin** — authenticates in `master`, approves onboarding, and invokes backend-controlled provisioning.
- **Organization Admin** — authenticates in the organization’s dedicated realm and runs existing tenant operations.
- **Keycloak administration** — performed only by the backend’s server-side Keycloak Admin API adapter.

`SUPER_ADMIN` need not be a realm-wide Keycloak role. The backend derives owner permissions from the authenticated onboarding-realm identity plus a PostgreSQL ownership relationship. An ownership relationship is scoped to exactly one organization; an owner may be linked to multiple organizations only through separate explicit records. Tenant IDs from the client, another tenant realm session, or hidden UI controls never establish ownership. Owners receive neither Platform Admin, Organization Admin, Hiring Manager, HR Reviewer, Interviewer, Applicant, nor cross-organization authority by inheritance.

### 45.2 Proposed records and audit fields

The following additions require a DBML/schema implementation pass before implementation:

| Proposed record | Minimum responsibility | Important constraints |
|---|---|---|
| `organization_ownership` | Links `organization_id` to the Super Admin’s onboarding-realm/canonical identity. | One active owner is a policy default; any ownership transfer is a separate, unapproved future decision. |
| `organization_admin_appointments` | Append-only initial/replacement appointment lifecycle. | One active appointment per organization; retains initiator, outgoing/incoming identity, effective timestamp, status, reason, correlation ID, and invitation/verification evidence references. |
| `admin_replacement_transitions` or equivalent state data | Tracks the guarded work across PostgreSQL, Keycloak, and notification delivery. | Idempotency key, retry state, error/recovery metadata; no duplicate simultaneous replacement. |
| extended `audit_events` / assignment audit | Preserves historical attribution for every authority change. | `organization_id`, recipient, actor, actor-authority snapshot, created/modified/revoked timestamps, status, reason, and immutable correlation ID. |

Existing role assignments must preserve `created_by_user_id` (or equivalent immutable actor reference), the actor’s authority at the time, the recipient, organization, job-opening scope where applicable, current status, later modifier/revoker, timestamps, and required reason. Revoking Organization Admin authority must never overwrite the original actor on prior Hiring Manager, HR Reviewer, Interviewer, or other assignments.

### 45.3 Onboarding sequence and appointment activation

~~~text
Super Admin (onboarding realm)
  -> submits organization details + initial-admin nomination
  -> PENDING_REVIEW (no organization ID / no tenant realm)
Platform Admin (master realm)
  -> approve or return/reject
  -> backend provisions organization + dedicated realm
  -> ownership record + initial appointment = INVITED
Nominated Organization Admin
  -> registration/linking + Keycloak verification + acceptance
  -> active tenant membership + ORGANIZATION_ADMIN role
  -> appointment = ACTIVE; notifications sent
~~~

The initial candidate is nominated by the Super Admin but appointed only after Platform Admin approval, realm provisioning, identity verification, and required acceptance succeed. The candidate may be an existing eligible identity or a new invitee. Realm-local Keycloak IDs are not transferable: an identity in another organization realm is linked/created through an explicit supported identity flow, never silently moved. The system must not create active tenant membership or administrative privilege merely because a form was submitted.

The provisioning orchestration is idempotent and records an operation/correlation ID before external effects. It writes `PROVISIONING`, `REALM_READY`, `AWAITING_ADMIN_ACTIVATION`, `ACTIVE`, or `PROVISIONING_RECOVERY` (and return/rejection states) truthfully. Database status becomes final only when the matching Keycloak realm/identity/membership state is confirmed. A partial failure retains repair metadata, retries safely, or runs a compensating action; the frontend has no privileged Keycloak Admin access.

### 45.4 Organization Admin replacement transaction and recovery

Ordinary replacement is a controlled transition, initiated only by the linked Super Admin:

1. Authorize owner + target organization; lock/reject concurrent replacement transitions; read the active appointment.
2. Validate candidate identity/eligibility and classify it as existing eligible identity, registration required, another-realm identity requiring explicit linking, or failed/ineligible. Create `INVITED`, `EMAIL_VERIFICATION_PENDING`, or `ACCEPTANCE_PENDING` state without changing the outgoing authority.
3. Expiry/rejection/validation failure ends as `EXPIRED`, `REJECTED`, or `VALIDATION_FAILED`; outgoing Organization Admin access remains active and the owner may nominate another candidate.
4. After verification and acceptance, revalidate ownership and current appointment, prepare the incoming realm identity/membership, and activate its `ORGANIZATION_ADMIN` role within the same guarded logical transition that ends the outgoing Organization Admin role.
5. Commit append-only appointment/audit events, then notify outgoing/incoming administrators and the Super Admin. Existing tenant data and role assignments remain intact.

The intended steady state has one active Organization Admin and no indefinite dual administrative access. If incoming activation fails before the transition commits, the outgoing administrator remains active and the transition becomes `ACTIVATION_FAILED` / `RECOVERY_REQUIRED`; reconciliation compares recorded operation state with Keycloak and repairs/compensates safely. If a security incident requires immediate outgoing suspension, use a distinct emergency-revocation action. It may produce `NO_ACTIVE_ADMIN_RECOVERY`, invokes authorized Platform Admin recovery, and blocks routine administration until a verified replacement becomes active. It never silently promotes another member.

Revoking Organization Admin does not revoke separately scoped Hiring Manager, HR Reviewer, or Interviewer assignments held by the outgoing person. Those remain governed by their existing lifecycles and must be independently changed through an authorized operation.

### 45.5 Authorization and API boundary (proposed)

Proposed owner endpoints are `POST /organization-onboarding-requests`, `GET /owner/organizations`, `GET /owner/organizations/{id}/admin-appointments`, `POST /owner/organizations/{id}/admin-replacements`, and read-only `GET /owner/organizations/{id}/audit`. They require a validated onboarding-realm owner identity and a server-side ownership lookup. Proposed Platform Admin endpoints remain responsible for review/provisioning; only the backend adapter calls Keycloak Admin APIs. Organization Admin must be denied any ownership mutation or appointment-replacement endpoint. UI hiding is informational; every request rechecks realm, ownership, membership, role scope, lifecycle state, and object tenant relationship server-side and audits denials.

### 45.6 Explicit open decisions

1. **Ownership transfer and loss/compromise of the Super Admin account** are not defined by the current artifacts. This update supports Organization Admin replacement, not ownership transfer. A platform-authorized recovery/ownership-transfer policy, evidence standard, quorum, and audit model need product/security approval before implementation.
2. **Multiple owners** are not approved. The proposed model permits multiple owned organizations per person through discrete records, but assumes one active owner per organization until the business policy says otherwise.
3. **Acceptance requirement** is represented as required for a normal appointment/replacement. Exact Keycloak required actions, expiry duration, resend limits, and emergency-approval authority must align with the deployed identity configuration.
