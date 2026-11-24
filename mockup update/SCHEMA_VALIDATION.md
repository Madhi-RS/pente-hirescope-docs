# HireScope replacement schema validation

**Status:** Structurally validated on 2026-11-24; pending product-sign-off decisions below.

## Structural checks passed

- 55 table declarations.
- 145 foreign-key references.
- Every referenced table and column exists in `pente_db_diagram.dbml`.
- No Platform Admin, Portal Admin, or Requisition Creator role remains in the replacement DBML.
- Organization Owner is modelled separately from tenant membership; Applicants are application-scoped.

## Remaining product decisions before implementation migration

1. **Onboarding initiator:** `organization_onboarding_requests.requester_keycloak_user_id` is inherited from the supplied list, where it is described as the proposed Organization Admin. Confirm whether this identity is instead always the Organization Owner. If so, rename it to `owner_keycloak_user_id`.
2. **Staff self-service:** Retain `role_verification_requests` only if staff may request HR or Hiring Manager access. Otherwise, Organization Owner/Admin should create `organization_member_roles` directly and this table can be removed.
3. **Hiring Manager scope:** Confirm that an organisation-scoped `HIRING_MANAGER` role may create requisitions before an opening exists; Interviewer is opening-scoped. The schema permits this with a nullable `job_opening_id`.
4. **Database enforcement:** PostgreSQL migrations must add cross-table tenant/version checks. In particular, a nominated interviewer, application stage, or stage attempt must belong to the same opening and opening version as its parent record. These checks cannot be guaranteed by a single-column foreign key alone.
5. **Fairness/privacy:** `applicant_profiles` education fields and `job_applications.distance_traveled_context` must be optional, access-controlled context only. They must never be used as an automated score, rank, or decision input.

## Source of truth

The complete replacement schema is [pente_db_diagram.dbml](./pente_db_diagram.dbml). The original DBML outside this folder was intentionally left unchanged.
