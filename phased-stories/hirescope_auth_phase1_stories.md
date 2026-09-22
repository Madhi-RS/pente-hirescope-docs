# HireScope Auth Module — Phase 1: Keycloak, Guards, Sign-In, and Session Refresh

## Realm setup, JWT validation, role authorization, and secure session endpoints

> **Source requirements:** `PROJECT_WORKFLOW_QA.md` §13 and `website-user-flow.md` §2–3, §15.  
> **Architecture input:** User-confirmed authentication provider: **Keycloak**.  
> **Depends on:** foundational workspace, PostgreSQL user-reference model, and Keycloak service being available.  
> **Build order:** AUTH-01 → AUTH-02 → AUTH-03 → AUTH-04 → AUTH-05.

---

## Resolved decisions

- HireScope uses an application-owned sign-in page; users do not need to use the Keycloak UI directly.
- The Next.js backend is a confidential Keycloak client and performs the direct credential exchange in the same manner as the Pente reference. Browser code never holds a Keycloak client secret.
- The four top-level Keycloak roles are `HIRING_MANAGER`, `HR_REVIEWER`, `APPLICANT`, and `INTERVIEWER`.
- A JWT carries `sub`, verified email, `hirescope_role`, and group membership. It must not carry résumé, score, interview, or candidate-decision data.
- The access token is returned to the client and held in browser memory only—never local storage or a cookie. The refresh token is stored only in a `HttpOnly`, `Secure`, `SameSite=Strict` cookie scoped to the HireScope auth endpoints.
- Backend APIs validate token signature, issuer, audience, and expiry using Keycloak JWKS with caching. Domain ownership and interview-assignment checks remain API/service checks; a role claim alone never grants access to another candidate's record.

## AUTH-01 — Keycloak realm, confidential client, and four role groups

**As a** platform architect,  
**I want** a Keycloak realm and a confidential HireScope backend client with the four product roles,  
**so that** HireScope can authenticate users and pass minimal trusted identity claims to its APIs.

**Depends on:** none

### Acceptance Criteria

1. A Keycloak realm named `hirescope` is defined as a version-controlled realm configuration.
2. A confidential client `hirescope-web-backend` is configured for the Next.js backend. Its secret is stored only in server-side deployment configuration.
3. The realm contains groups/roles for `HIRING_MANAGER`, `HR_REVIEWER`, `APPLICANT`, and `INTERVIEWER`; a user may have only the role combinations approved by HireScope governance.
4. Tokens issued for HireScope contain `sub`, verified email, `hirescope_role`, and group membership through supported Keycloak role/group mapping.
5. Test users for each role exist only in development/test configuration; no production password or user secret is committed to source control.
6. Realm configuration can be imported/reapplied without duplicating groups, clients, or role mappings.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-01-TC-01 | Import the `hirescope` realm configuration | Import succeeds and creates/updates the intended realm (AC-01, AC-06) |
| AUTH-01-TC-02 | Inspect `hirescope-web-backend` | Client is confidential; its secret is not exposed to browser configuration (AC-02) |
| AUTH-01-TC-03 | Inspect realm role/group catalog | All four required HireScope roles/groups exist (AC-03) |
| AUTH-01-TC-04 | Authenticate one test user per role | JWT contains only the required identity/role/group claims (AC-04) |
| AUTH-01-TC-05 | Inspect repository and production configuration templates | No committed production credentials or production seeded users exist (AC-05) |
| AUTH-01-TC-06 | Reapply realm configuration | No duplicate client, group, or mapper is created (AC-06) |

### Done Criteria

- [ ] Realm configuration, client configuration, and role/group mapping are reviewed by security and platform owners.
- [ ] A test can obtain and validate a token for each of the four roles.
- [ ] No sensitive client secret or production test credential is committed.

---

## AUTH-02 — Validate Keycloak JWTs on every protected API

**As a** backend developer,  
**I want** protected HireScope APIs to validate Keycloak JWTs before executing business logic,  
**so that** unauthenticated or forged requests cannot access recruitment data.

**Depends on:** AUTH-01

### Acceptance Criteria

1. Protected APIs require an `Authorization: Bearer <access-token>` header by default; only explicit public endpoints bypass this rule.
2. Validation uses the Keycloak JWKS endpoint and verifies signature, issuer, audience, expiry, and not-before time; a shared signing secret is not used.
3. JWKS keys are cached for a bounded approved period and refresh safely after key rotation.
4. The authenticated request context contains only normalized `userId` (`sub`), verified email, `role`, and groups needed for authorization.
5. Missing, malformed, expired, wrong-audience, wrong-issuer, or tampered tokens receive a generic authentication failure without leaking token internals.
6. Keycloak/JWKS unavailability fails closed and is observable to authorized operations staff.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-02-TC-01 | Valid JWT on protected API | Request is admitted and normalized identity context is available (AC-01, AC-04) |
| AUTH-02-TC-02 | No bearer token on protected API | Request is rejected before business handler execution (AC-01, AC-05) |
| AUTH-02-TC-03 | Expired, tampered, wrong issuer, or wrong audience token | Each is rejected with safe generic authentication response (AC-02, AC-05) |
| AUTH-02-TC-04 | Request a documented public health/auth endpoint without token | Only explicitly public endpoint succeeds without JWT (AC-01) |
| AUTH-02-TC-05 | Send repeated valid requests then rotate signing key | JWKS is cached within policy and validation recovers after rotation (AC-03) |
| AUTH-02-TC-06 | Make JWKS unavailable | Request fails closed and an operational signal is recorded (AC-06) |

### Done Criteria

- [ ] JWT validation is globally enforced for protected APIs.
- [ ] Token validation tests cover valid, missing, malformed, expired, tampered, wrong issuer, and wrong audience cases.
- [ ] Key rotation and Keycloak unavailability behavior are documented and tested.

---

## AUTH-03 — Enforce role, ownership, and assignment authorization

**As a** HireScope user,  
**I want** routes and data to be limited to my role and record scope,  
**so that** recruitment decisions and candidate data are visible only to authorized people.

**Depends on:** AUTH-02

### Acceptance Criteria

1. The API enforces the following top-level role permissions within explicit organisation scope: Hiring Manager manages their authorised organisation's requisitions/opening drafts; HR Reviewer performs authorised organisation approvals and review decisions; Applicant accesses only their own application; Interviewer accesses only assigned interview events.
2. A Hiring Manager cannot approve their own requisition/change, publish before required HR approval, access HR candidate-review evidence outside an explicit assigned permission, or make final HR decisions.
3. An HR Reviewer cannot alter immutable AI scores and can access only organizationally authorized openings/applications.
4. An Interviewer cannot access unrelated applications, edit/reprocess AI results, change candidate eligibility/final status, or submit on behalf of another participant.
5. An Applicant cannot access other applicants, AI scores, interviewer scores, reviewer comments, or internal rationale.
6. Every denied protected-data action is audited with organisation, actor, attempted action, target, and time; APIs enforce this even when a user knows a UI route or record identifier.
7. A role claim without matching organisation ownership/scope never grants access, and cross-organisation record association is rejected.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-03-TC-01 | Each role opens its own permitted landing/queue | Allowed role-scoped resources are available (AC-01) |
| AUTH-03-TC-02 | Hiring Manager attempts self-approval, early publish, unauthorised HR review, or final decision | Every prohibited action is denied (AC-02) |
| AUTH-03-TC-03 | HR Reviewer attempts AI score mutation or unauthorized-opening access | Both actions are denied (AC-03) |
| AUTH-03-TC-04 | Interviewer requests another event or attempts status/AI changes | Unassigned access and prohibited mutations are denied (AC-04) |
| AUTH-03-TC-05 | Applicant requests internal data or another application | Request is denied and no confidential fields are returned (AC-05) |
| AUTH-03-TC-06 | Authorized user manually requests a forbidden API route | Server denies request and writes a protected-access audit record (AC-06) |
| AUTH-03-TC-07 | Cross-organisation access/association | Same role exists in two organisations | Use one organisation's identity against the other's opening/configuration/application | Request is denied, no fields leak, and the denial is audited (AC-07) |

### Done Criteria

- [ ] Role checks are supplemented by opening ownership, HR authorization scope, applicant ownership, and interview assignment checks.
- [ ] A role matrix test proves both allowed and denied behavior for all four roles.
- [ ] Denied confidential access is auditable.

---

## AUTH-04 — Sign in through HireScope and establish a safe session

**As a** HireScope user,  
**I want** to sign in through the HireScope application,  
**so that** I receive a secure session without exposing Keycloak credentials or token secrets to browser storage.

**Depends on:** AUTH-01, AUTH-02

### Acceptance Criteria

1. A public sign-in endpoint accepts validated email and password, exchanges credentials with Keycloak through the server-side confidential client, and returns only an access token and expiry to the browser.
2. On success, the refresh token is set only as an `HttpOnly`, `Secure`, `SameSite=Strict` cookie scoped to the HireScope authentication path; it is never returned in a response body.
3. The access token is never set as a cookie and the frontend stores it only in memory.
4. Invalid credentials, disabled accounts, and unknown emails receive the same generic failure message and do not create a session cookie.
5. Sign-in applies approved rate limiting and audit logging without recording passwords or full token values.
6. Successful sign-in redirects/routs the user to the appropriate role-aware HireScope home location; authorization still controls every later API request.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-04-TC-01 | Valid credentials for each of the four roles | Access token/expiry return; secure refresh cookie is set; correct role-aware home route is selected (AC-01, AC-02, AC-06) |
| AUTH-04-TC-02 | Inspect success response and browser storage | Refresh token is absent from body; access token is absent from cookie and persistent storage (AC-02, AC-03) |
| AUTH-04-TC-03 | Wrong password, unknown email, and disabled account | Same generic failure; no cookie/session is created (AC-04) |
| AUTH-04-TC-04 | Rapid repeated sign-in attempts | Approved rate limit is applied and attempt metadata is safely audited (AC-05) |
| AUTH-04-TC-05 | Inspect sign-in audit entry | Actor/result/time are present; password and token values are absent (AC-05) |

### Done Criteria

- [ ] Sign-in works for all four roles without exposing a client secret, password, or refresh token.
- [ ] Browser storage inspection proves access/refresh token handling matches the approved policy.
- [ ] Failed sign-in behavior is enumeration-safe and audited.

---

## AUTH-05 — Refresh an expired access token securely

**As a** signed-in user,  
**I want** my access token refreshed from my secure session cookie,  
**so that** normal token expiry does not interrupt permitted work.

**Depends on:** AUTH-04

### Acceptance Criteria

1. The refresh endpoint reads only the `HttpOnly` refresh cookie, exchanges it with Keycloak, returns a new access token/expiry, and rotates the refresh cookie where Keycloak supports rotation.
2. The endpoint does not accept refresh tokens in request body, query parameters, or browser-readable storage.
3. Missing, invalid, revoked, expired, or reused refresh tokens clear the cookie and produce a generic unauthenticated response.
4. Concurrent refresh handling permits at most one effective refresh operation for one browser session; queued requests either reuse the refreshed token or fail safely.
5. Refresh success/failure and session invalidation are auditable without retaining raw token values.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-05-TC-01 | Valid refresh cookie | New access token/expiry return and secure refresh cookie is renewed/rotated (AC-01) |
| AUTH-05-TC-02 | Supply refresh token in body or query | Value is ignored/rejected; only cookie can refresh (AC-02) |
| AUTH-05-TC-03 | Missing, expired, revoked, invalid, or reused cookie | Session is rejected and cookie is cleared (AC-03) |
| AUTH-05-TC-04 | Concurrent expired requests from one browser session | At most one effective refresh occurs; requests resolve safely (AC-04) |
| AUTH-05-TC-05 | Inspect refresh audit history | Result/session event is recorded without raw token data (AC-05) |

### Done Criteria

- [ ] Refresh handles valid, missing, expired, invalid, revoked, and reused tokens safely.
- [ ] Concurrent refresh behavior is tested.
- [ ] Cookie attributes match the sign-in policy on issue, rotation, and clearing.
