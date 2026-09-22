# HireScope Auth Module — Phase 2: Invitation Activation, Logout, and Frontend Session Experience

## Candidate account activation, session termination, frontend auth state, and route protection

> **Source requirements:** `PROJECT_WORKFLOW_QA.md` §7–8 and §13–14; `website-user-flow.md` §7–8 and §17.  
> **Depends on:** `hirescope_auth_phase1_stories.md` completed.  
> **Build order:** AUTH-06 → AUTH-07 → AUTH-08 → AUTH-09 → AUTH-10.

---

## Resolved decisions

- Applicant self-service account activation is invitation-bound. A public opening/source link creates interest only; it never creates assessment access.
- Only an applicant qualified by an authorised HR decision receives an activation/assessment invitation bound to verified email, organisation, application, opening, opening-configuration version, and FairPack version. Candidate details and résumé have already been collected through the secure application-intake path.
- Hiring Managers, HR Reviewers, and Interviewers are provisioned by the approved organizational process; they do not self-register from a public page.
- The frontend maintains access token and decoded minimal identity in memory. On application load it attempts a cookie-backed refresh; it never persists an access token to local storage.
- Password reset, SSO, MFA, account-deprovisioning, and exact session lifetimes remain governed by GAP-01 decisions. This phase does not invent them.

## AUTH-06 — Activate a qualified applicant account from a secure invitation

**As a** qualified applicant,  
**I want** to activate or link my HireScope account through my secure invitation,  
**so that** only I can continue to the selected opening's readiness and assessment flow.

**Depends on:** AUTH-04, qualified application/invitation workflow

### Acceptance Criteria

1. The activation endpoint is public only for a valid invitation token and requires creation/linking of an applicant account to the token's verified email.
2. The token is unique, time-limited, and bound to one verified email, organisation, application, opening, opening-configuration version, and FairPack version; it cannot be redeemed by another authenticated account.
3. Valid activation creates or links only the `APPLICANT` role, records activation/redeem history, establishes the secure session policy from Phase 1, and directs the applicant to the configured readiness step.
4. Invalid, expired, revoked, previously redeemed, or identity-mismatched tokens create no account/application change and display safe applicant-facing guidance without exposing internal data.
5. Invitation issue, delivery, expiry, resend, revocation, redemption, and identity mismatch are auditable; resend/recovery is rate-limited and authorized.
6. Activation does not bypass the completed intake, HR qualification, or readiness requirements and grants no access to other organisations, applications, opening versions, or internal evaluation data.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-06-TC-01 | Redeem a valid HR-qualified applicant token | Applicant account is created/linked, bound to correct organisation/application/version, session is established, readiness step opens (AC-01, AC-03) |
| AUTH-06-TC-02 | Redeem token from another authenticated account | Attempt is refused and identity mismatch is audited (AC-02, AC-04) |
| AUTH-06-TC-03 | Redeem expired, revoked, invalid, or used token | No account/application mutation; safe guidance appears (AC-04) |
| AUTH-06-TC-04 | Resend/recovery lifecycle | Actions are controlled/rate-limited and all invitation events are audited (AC-05) |
| AUTH-06-TC-05 | Activated applicant requests another organisation/application or internal result | Access remains scoped; intake, HR-decision, and readiness prerequisites still apply (AC-06) |

### Done Criteria

- [ ] One valid token can activate/link only its intended applicant and application.
- [ ] All invalid token paths are safe, non-enumerating, and non-mutating.
- [ ] Activation is integrated with the secure invitation audit trail.

---

## AUTH-07 — End the session and revoke browser access safely

**As a** signed-in HireScope user,  
**I want** to log out,  
**so that** my current browser session can no longer access HireScope.

**Depends on:** AUTH-04, AUTH-05

### Acceptance Criteria

1. Logout requires a valid authenticated request and clears the refresh cookie using the same security attributes/path with an immediate expiry.
2. Logout requests Keycloak session revocation according to the approved Keycloak administrative capability; local cookie clearing still occurs if revocation is temporarily unavailable.
3. Logout clears in-memory frontend identity/access-token state and routes to the sign-in page.
4. The action is idempotent: repeating logout does not leak session state or create an error that prevents local session clearing.
5. Logout and revocation outcome are audited without recording token values.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-07-TC-01 | Authenticated user logs out | Refresh cookie clears, Keycloak revocation is requested, memory state clears, sign-in route opens (AC-01–03) |
| AUTH-07-TC-02 | Keycloak revocation service is unavailable | Local cookie/state still clear; operational outcome is recorded (AC-02) |
| AUTH-07-TC-03 | Repeat logout request | Safe idempotent result; no active local session remains (AC-04) |
| AUTH-07-TC-04 | Try refresh after logout | Refresh fails and no new access token is issued (AC-01–03) |
| AUTH-07-TC-05 | Inspect logout audit event | Actor/result/time are retained without token value (AC-05) |

### Done Criteria

- [ ] Logout clears local browser capability regardless of upstream revocation availability.
- [ ] Repeated logout and post-logout refresh tests pass.

---

## AUTH-08 — Maintain frontend auth state and protected-route behavior

**As a** frontend developer,  
**I want** one in-memory authentication state and role-aware route protection,  
**so that** HireScope users receive an appropriate experience without client-side authorization replacing server checks.

**Depends on:** AUTH-04, AUTH-05, AUTH-07

### Acceptance Criteria

1. On initial application load, the frontend performs one cookie-backed refresh attempt; success hydrates minimal identity/role and access token in memory, while an unauthenticated result directs only protected routes to sign-in.
2. The frontend never saves access tokens, refresh tokens, or Keycloak client secrets in local storage, session storage, URL fragments, or application logs.
3. Client route guards provide role-appropriate navigation: Hiring Manager, HR Reviewer, Applicant, and Interviewer see only their permitted entry routes; server APIs remain the authority for every data/action decision.
4. The frontend has explicit loading, unauthenticated, and authorization-denied states and does not briefly render protected data before validation completes.
5. Role assignment, opening ownership, applicant ownership, and interview assignment changes take effect on the next authorized API evaluation and do not rely on stale client claims alone.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-08-TC-01 | Load app with valid refresh cookie | Minimal identity/access token hydrates in memory and permitted home view opens (AC-01) |
| AUTH-08-TC-02 | Load protected route with no valid session | User is routed to sign-in without protected-data flash (AC-01, AC-04) |
| AUTH-08-TC-03 | Inspect browser storage/logging after sign-in/refresh | No token or client secret persists outside approved cookie/memory locations (AC-02) |
| AUTH-08-TC-04 | Open each role's permitted and unpermitted navigation route | Client navigation is role-aware; server still rejects unauthorized data/actions (AC-03) |
| AUTH-08-TC-05 | Change interview assignment/authorization scope then retry access | Subsequent server authorization determines access; stale UI does not grant capability (AC-05) |

### Done Criteria

- [ ] Frontend state has safe startup, loading, sign-in, sign-out, and forbidden behavior.
- [ ] Browser-storage inspection verifies the token policy.
- [ ] Route guards improve UX but all authorization tests run against APIs too.

---

## AUTH-09 — Provide accessible, safe sign-in and applicant activation pages

**As a** HireScope user,  
**I want** clear sign-in and applicant activation pages,  
**so that** I can access my authorized workflow without seeing internal authentication details.

**Depends on:** AUTH-04, AUTH-06, AUTH-08

### Acceptance Criteria

1. The sign-in page validates email/password input, supports password-manager use, presents loading/error states, and uses generic invalid-credential wording.
2. An already authenticated user who opens sign-in is routed to their permitted home view without rendering a sign-in form unnecessarily.
3. Public applicant activation is available only through a valid invitation token; there is no generic public "create account" path that bypasses qualification.
4. Expired, invalid, or already-used applicant invitation pages provide safe guidance and do not reveal an applicant's identity, opening details, assessment status, or internal rule result.
5. The screens meet approved accessibility standards, including keyboard operation, visible labels, focus order, and error announcements.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-09-TC-01 | Submit valid and invalid sign-in form values | Valid values call sign-in; invalid values show accessible field errors before request (AC-01) |
| AUTH-09-TC-02 | Submit incorrect credentials | Generic error appears; form does not disclose whether email exists (AC-01) |
| AUTH-09-TC-03 | Visit sign-in with active session | Role-aware redirect occurs with no unnecessary form flash (AC-02) |
| AUTH-09-TC-04 | Visit activation with valid and invalid/expired/used token | Valid token reaches activation; all invalid paths are safe and non-disclosing (AC-03, AC-04) |
| AUTH-09-TC-05 | Navigate screens using keyboard/screen-reader checks | Labels, focus, loading, and errors are accessible (AC-05) |

### Done Criteria

- [ ] Sign-in and activation pages have safe error states and no enumeration leaks.
- [ ] Public registration cannot bypass qualification/invitation.
- [ ] Accessibility checks cover keyboard and error-announcement behavior.

---

## AUTH-10 — Refresh on one API 401 and fail safely

**As a** signed-in user,  
**I want** the application to refresh a legitimate expired access token once and retry my request,  
**so that** normal access-token expiry is not disruptive while invalid sessions end safely.

**Depends on:** AUTH-05, AUTH-07, AUTH-08

### Acceptance Criteria

1. The API client sends the current in-memory access token in the bearer header and includes credentials only for the approved same-site authentication/session flow.
2. On one authentication failure from a protected API, the client performs one refresh attempt, updates memory with a new access token on success, and retries the original request exactly once.
3. If refresh fails or the retried request remains unauthenticated, the client clears local state and routes to sign-in without an infinite retry loop.
4. Concurrent authentication failures share one in-flight refresh operation; queued requests retry with the new token or fail consistently when refresh fails.
5. Non-authentication errors such as authorization-denied or validation failures do not trigger refresh.

### Test Cases

| ID | Scenario | Expected |
|---|---|---|
| AUTH-10-TC-01 | Protected API request with valid in-memory token | Bearer header is sent and no refresh occurs (AC-01) |
| AUTH-10-TC-02 | First protected request receives authentication failure then refresh succeeds | One refresh occurs; original request retries once with new token (AC-02) |
| AUTH-10-TC-03 | Refresh or retry fails authentication | State clears, sign-in route opens, and no loop occurs (AC-03) |
| AUTH-10-TC-04 | Multiple concurrent requests receive authentication failure | One refresh operation is shared; all queued outcomes are consistent (AC-04) |
| AUTH-10-TC-05 | Protected request receives 403 or validation error | Error passes through; no refresh is attempted (AC-05) |

### Done Criteria

- [ ] Refresh/retry happens exactly once per failed request.
- [ ] Concurrent failure test proves there is no refresh stampede.
- [ ] 403 and validation errors never trigger token refresh.
