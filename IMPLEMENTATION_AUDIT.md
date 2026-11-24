# HireScope implementation audit and delivery backlog

**Audited:** 2026-11-24  
**Implementation status:** authentication shell; organization roles are now recognized from Steward, but the documented workflows are not implemented.  
**Authoritative functional flow:** `website-user-flow.md` (including its later controlling sections).  
**Authoritative data catalogue:** `mockup update/pente_db_diagram.dbml` (55 tables).

## What exists today

| Area | Evidence | Status |
|---|---|---|
| Browser session | Steward PKCE callback, opaque cookie session, session/logout endpoints | Implemented; Steward is the approved authentication baseline |
| Local data model | Six Prisma models: Organization, Identity, Membership, MembershipRole, AuthSession, AuthSignInAttempt | Implemented only for the existing Steward sign-in shell |
| Frontend shell | Landing, login, callback, session guard, shared workspace shell; role-aware navigation for Organization Owner, Organization Admin, HR, Hiring Manager, and Interviewer | Implemented, workflow screens still pending |
| Hiring Manager UI | Static dashboard values; create action disabled pending API | Demo only |
| Admin, HR, Applicant, Interviewer UIs | Placeholder panels | Not implemented |
| Product workflow APIs | No requisition, opening, application, assessment, interview, decision, notification, audit, or retention routes | Not implemented |

## Journey coverage

| # | Journey | Status | Missing implementation work |
|---:|---|---|---|
| 1 | Organization ownership: onboarding, provisioning status, Organization Owner appointment/replacement | Not implemented | Ownership schema, onboarding/status and appointment lifecycle, owner-only authorization, provisioning integration, audit trail, screens, tests |
| 2 | Organization Admin: access, members, role requests, read-only opening monitoring | Not implemented | Admin authorization, member/role-request APIs, read-only monitoring projections, screens, tests |
| 3 | Hiring Manager: requisitions, opening versions, HR return/resubmission, panels, publication, monitoring | Demo only | Versioned requisition/opening domain, approval gates, panel nomination, publication rules, read-only applicant projections, UI/API/tests |
| 4 | HR: approvals, controlled pre-screen, operations, selection/headcount/onboarding/closure | Not implemented | Approval queues, HR decision boundary, assessments/interviews, transactional capacity, onboarding handoff, closure and retention controls |
| 5 | Applicant: intake, resume/pre-screen, readiness, assessment, accommodation/grievance, interview/outcome/onboarding | Not implemented | Public opening discovery, application/profile API, pinned forms/versions, invitation/readiness, applicant-safe status/messages, accommodation/grievance UI/API |
| 6 | Interviewer: invitation, stage/brief, immutable evaluation, completion/retake | Not implemented | Panel invitations, assignment-scoped access, brief projection, immutable scorecard, replacement/retake rules |
| 7 | System workflow: AI evidence, notifications, gates, suspension, audit/retention | Not implemented | Worker/integration boundaries, immutable evidence records, outbox delivery, workflow engine/jobs, suspension, legal holds, audit and retention processing |
| 8 | 55-table catalogue | Not implemented | Replace/extend the six-model schema with the approved catalogue, tenant/version invariants, migrations, repositories, and migration tests |

## Blocking decisions that must be resolved before changing authentication or data migrations

1. **Identity provider baseline — decided 2026-11-24.** Steward remains the authentication provider. Keycloak-specific language in the older flow is not an implementation requirement. HireScope synchronizes registered Steward module roles into tenant-scoped application roles and continues to keep its own authorization, workflow, and audit records.
2. **Organization-role boundary — decided 2026-11-24.** The Organization Owner is responsible for organization onboarding. The Organization Admin maintains members. The older onboarding/appointment wording must be reconciled before its screens and state machine are implemented.
3. **Schema enforcement.** The replacement schema document requires PostgreSQL cross-table tenant and version checks. The exact constraints/triggers must be designed before the first migration, not left to UI or ORM checks.

## Delivery order

Work is deliberately sliced so every later role rests on enforced tenancy, immutable history, and auditability.

1. **Foundation migration design** — map all 55 DBML tables to Prisma/migration ownership; design tenant/version constraints, audit append-only policy, retention policy, and test fixtures.
2. **Identity, tenancy, and authorization** — extend the Steward role boundary with organization, membership, scoped roles, policy middleware, and denied-access auditing. The first sub-slice (Steward Organization Owner, Organization Admin, and HR role synchronization) is complete.
3. **Organization onboarding and continuity** — onboarding submission/status, controlled Owner appointment/replacement/recovery, read-only admin monitoring.
4. **Requisition and opening workflow** — drafts, immutable versioning, HR review/return/resubmit, opening/package/panel approval, publication and read-only monitoring.
5. **Applicant intake and controlled pre-screen** — public opening view, version-pinned form/resume, AI evidence stored as advisory, HR-only progression decision.
6. **Assessment and applicant support** — readiness, configured attempt/response capture, accommodation and grievance paths, controlled invitations/no-response suspension.
7. **Interviews and final decision** — assignment/invitation, immutable evaluation, retake/replacement, HR selection, transactionally protected headcount, onboarding handoff and closure.
8. **Operations hardening** — outbox worker, notification templates, retention/legal holds, AI processing worker, audit exports, end-to-end role/security tests and accessibility QA.

## First implementation slice: acceptance checklist

The next code change is **Foundation migration design**. It is complete only when:

- every one of the 55 catalogue tables is mapped to a migration and repository boundary;
- tenant/version constraints are specified as database-enforced constraints or triggers;
- no existing authentication behavior is silently broken;
- migration and authorization test plans exist before domain endpoints are added.

No user journey is marked complete until its API authorization, persistence, UI state, audit behavior, and acceptance tests all pass.
