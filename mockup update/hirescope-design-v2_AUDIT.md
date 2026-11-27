# HireScope prototype audit — `hirescope-design-v2.html`

**Audited artefact:** `pente-hirescope-docs/mockup update/hirescope-design-v2.html`
**Sources of truth reviewed in full:** `HireScope_Master_Feature_Matrix.tsv` (13 phases), `Tables.tsv` (56 tables)
**Passes:**
- Pass 1 — 2026-11-27, initial audit.
- Pass 2 — 2026-11-27, re-audit after first Codex revision.
- Pass 3 (this file) — 2026-11-27, re-audit after second Codex revision. File is now 5,334 lines / ~588 KB.

**Method:** full static read of the HTML (markup, CSS, all inline JS), targeted grep verification of every prior finding, and verification of the newly introduced decision-control code. Screenshots (1697)–(1700) are present (valid 1920×1200 PNGs); visual comparison is limited to palette extraction because this session's model has no image input — all four are dark near-black (`#131619`) + white monochrome, consistent with the prototype's "Pente Sites" monochrome system.

---

## Change log (all three passes)

| Finding | Pass 1 | After Codex #1 | After Codex #2 (now) |
|---|---|---|---|
| Reviewer / Approver / HR separation of duties | 🔴 collapsed into one "HR Reviewer" | 🟡 visible only (panels + copy) | ✅ **functionally enforced** (see §3.1) |
| Stale 55-table catalogue (legacy DBML names) | 🔴 wrong schema | ✅ rebased to `Tables.tsv` 56 tables | ✅ still correct |
| `registerApplicant()` null-reference throw | 🔴 broken | ✅ fixed | ✅ still fixed |
| `installWorkflowAtlas()` never called | 🟠 dead | ✅ wired | ✅ still wired |
| Duplicate `renderMembershipDirectory` / `updatePortalUserRole` | 🟠 duplicated | ❌ still duplicated | ✅ **commented out** (first set retired) |
| Legacy vs canonical atlas renderer duplicates | 🟠 duplicated | ❌ still duplicated | ❌ **still duplicated** (§5.2) |
| `SUSPENDED` requisition status | 🔴 contradicts model | ✅ `WITHDRAWN` | ✅ still fixed |
| Opening statuses vs `Tables.tsv` | ⚠️ drift | ✅ `IN_REVIEW` | ✅ still aligned |
| Role-model drift (Keycloak / Super Admin / Platform Admin / Requisition Creator) | 🟠 drift | ❌ still present | ❌ **still present** (§6.2) |
| `.grid3` CSS missing | n/a (introduced in #1) | 🟡 nit | ✅ **added** (142, 215) |
| Reverse table→workflow traceability stubbed | n/a | ⚠️ stubbed to `pl-6` | ✅ **stub removed** (feature retired, no dead stub) |
| Missing verification & duplicate screens | ⚠️ mentioned only | 🟡 console-only | ✅ still console-only |

---

## 1. Navigation model coverage

All nine required areas remain present and unchanged: dashboards, requisitions/approvals, openings/panel, candidate pipeline, review/decision queue, interviews, offers/verification/onboarding, organisation guardrails, reporting/audit. Role-scoped routing with `VIEW_ROLE` isolation works.

| Area | Status |
|---|---|
| Dashboard / overview | ✅ |
| Requisitions & approvals | ✅ |
| Openings & panel setup | ✅ |
| Candidate pipeline | ✅ (search/filter/sort/pagination + bulk) |
| Review & decision queue | ✅ |
| Interviews | ✅ |
| Offers, verification, onboarding | ✅ (verification in console) |
| Organisation guardrails | ✅ |
| Reporting & audit | ⚠️ static counters only (no funnel/SLA chart) |

---

## 2. Lifecycle visibility coverage

- **Requisition:** DRAFT → IN_REVIEW/PENDING_APPROVAL → RETURNED_FOR_CHANGES → APPROVED → WITHDRAWN ✅ (no invented `SUSPENDED`).
- **Opening:** created → panel nominated → package approved → interviewer activation (24 h) → published → suspended/reactivated/closed ✅ (`IN_REVIEW/APPROVED/PUBLISHED/SUSPENDED/CLOSED`).
- **Application:** submitted → pre-screen → assessment invite/attempt/grievance → evidence review → interview → selection/non-proceed → offer → verification → onboarding → employee record / position fill ✅.
- **Terminal & exception paths:** withdrawal, non-proceed, technical interruption + grievance, retake/replacement, offer declined/expired/cancelled, AI failure, closure, legal hold ✅.
  - **Duplicate resolution** (`mwDuplicates`) and **post-acceptance verification** (`mwVerification`, SpringVerify) are demonstrable — 🟡 console-only, not the main `hr-*` body.

---

## 3. Governance conformance

### 3.1 Separation of duties — ✅ FIXED (functionally enforced)
The primary flow now has a real three-step gated handoff with distinct actors:

- **1 · Hiring Reviewer** (`recordPrimaryRecommendation`, actor "N. Rao · Hiring Reviewer") records a recommendation with rationale + correlation ID.
- **2 · Hiring Approver** (`recordHrPrescreenDecision`, `recordHrFinalDecision`, `saveHrRequisitionStatus`, `saveHrOpeningStatus`, actor "M. Chen · Hiring Approver") is **blocked** (`if (!record.reviewerRecommendation) return`) until a recommendation exists.
- **3 · HR** (`communicatePrimaryOutcome`, actor "A. Patel · HR") is **blocked** until an approver decision exists, and delivers only the approved applicant-visible status.

`setPrimaryDecisionControls` (3786) disables the approver button until a recommendation is present, and the communicate button until a decision is present. In-view buttons exist for all three steps in requisition (1771), opening (1826), pre-screen (2020), and final decision (2166). Conflict rules ("same person cannot recommend and decide; creators cannot self-approve; declared conflict blocks evaluation") are stated in the injected governance panel. **This resolves the Pass-1 P0 governance gap.**

*Residual nuance (minor):* the pre-screen decision state lives in a standalone `primaryDecisionRecords.prescreen` object, while requisition/opening/final store the recommendation/decision on their existing record objects — a slight state-model inconsistency, not a functional bug.

### 3.2 AI advisory labelling — ✅ strong (unchanged)
AI consistently labelled advisory and immutable; "never auto-advance/reject/select/send" stated throughout.

### 3.3 HR communication-only boundary — ✅ now structural
`communicatePrimaryOutcome` enforces that HR only sends the approved status and that reviewer/approver rationale stays confidential. Good.

### 3.4 Decision evidence/actor/timestamp/status/audit — ✅ substantially met
Recommendation and decision now each carry actor, rationale, timestamp, and correlation ID (`REC-…`, `DEC-…`, `NTF-…`), with the recommendation/decision/communication stored separately.

### 3.5 Blind review — ✅ (unchanged)

---

## 4. Interactions present (verified)

Working view routing, tabs, search/filter/sort/pagination, detail views, lifecycle steppers, decision panels with rationale, and confirmation/feedback states — all present. Happy path (requisition → … → employee record → position fill) and exception paths (grievance, AI failure, retake, cross-tenant denied, non-proceed, closure/legal hold) remain demonstrable.

---

## 5. Technical / functional defects (remaining)

### 5.1 ❌ Legacy vs canonical atlas renderers still duplicated
Two full sets of renderer declarations remain in the same scope; the later ("canonical") set silently overrides the earlier ("legacy") set, so the legacy set is dead code and still references the superseded DBML names (`organizations`, `canonical_people`, `hr_decisions`, `interview_results`, `opening_stage_interviewer_nominations`, …):

- Legacy: `renderOrgAdminStart` 5168, `renderOwnerOverview` 5172, `renderOwnerReview` 5173, `renderHrSchedule` 5180, `renderHrSelection` 5182, `renderApplicantChoose` 5189, `renderApplicantDecision` 5193, `renderInterviewerAssignment` 5194, `renderPipelineRail` 5198.
- Canonical: `renderOwnerOverview` 5206, `renderOwnerReview` 5207, `renderOrgAdminStart` 5211, `renderHrSchedule` 5215, `renderHrSelection` 5217, `renderApplicantChoose` 5220, `renderApplicantDecision` 5226, `renderInterviewerAssignment` 5227, `renderPipelineRail` 5232.

Legal JS (no console error), but the legacy block should be removed — it is confusing and contradicts the rebased 56-table catalogue. (The `renderMembershipDirectory`/`updatePortalUserRole` duplicates *were* cleaned this pass via comment-out at 3159–3210.)

### 5.2 ✅ Fixed — registration flow
No null-reference remains in `registerApplicant`/`toggleStaffRequest`.

### 5.3 ✅ Fixed — `.grid3`
`.grid3` is now defined (142) with a responsive rule (215); the governance panel renders three columns.

### 5.4 ✅ Fixed — `WF_TABLE_WORKFLOWS` stub removed
The `[table → ['pl-6']]` stub is gone. The trade-off is that the reverse table→workflow-step view is now retired (the catalogue shows table scope/purpose only), which is cleaner than a misleading stub.

---

## 6. Data-model conformance (Tables.tsv)

| Area | Conformance |
|---|---|
| 56-table catalogue (`WF_TABLES`) | ✅ matches `Tables.tsv` names and count |
| Organisation/workspace + tenant isolation | ✅ |
| People/roles | ❌ role model still drifted (§6.2) |
| Requisition / version / review | ✅ statuses aligned; review-vs-decision now enforced |
| Opening / version / stage / panel | ✅ aligned |
| Application / duplicate / resume facts | 🟡 duplicate review in console; stage-level statuses still not the `job_applications.status` enum |
| Attempts / invitation / accommodation / grievance | ✅ strong |
| Responses / media / AI advisories / integrity flags | ✅ strong |
| Recommendations vs decisions | ✅ enforced (distinct actors + gating) |
| Live interview / assignments / evaluations | ✅ strong |
| Selection / reservation / offer / verification | ✅ offer strong; verification in console |
| Onboarding / tasks / account activation | ✅ strong |
| Employment profile / history / documents | ✅ strong |
| Capacity ledger / position fills / closure | ✅ |
| Retention / legal hold / deletion / audit / outbox | 🟡 retention/legal hold static; hold surfaced in console |
| Integration operations | 🟡 AI failure/retry surfaced; calendar/provider ops not a screen |

### 6.2 Role-model drift — **still open (top remaining item)**
`Tables.tsv` roles: `ORGANISATION_OWNER, ORGANISATION_ADMIN, HR, HIRING_MANAGER, HIRING_REVIEWER, HIRING_APPROVER, INTERVIEWER`.

Prototype roles actually used: `hm`, `hr`, `in`, `oa` (nav "Org Owner" but renders "Organization Admin" copy), `sa` ("Org Admin"/"Super Admin"), `rc` ("Requisition Creator" — not a `Tables.tsv` role), `ap`, and a retired `pa` (Platform Admin).

The source HTML still contains ~40 occurrences of "Keycloak" (415, 417, 431, 442, 512, 519, 526, 541, 796, 860, 868, 950, 969, 1040, 1165, 2357, 2386, 2980–2983, 3017–3020, 3253, 3405, 3491, 3654, 5170, …), "Super Admin" (927, 942, 999, 1002, 1009, 3531), "Platform Admin" (920, 927, 942, 963, 968, 970, 3398, 3491, 3505), "Requisition Creator" (2512, 3141, 3561, 3569, 3622, 3648, 3651, 3690, 3718, 4131), `superadmin@pente.ai` (2897), and the `rc`/`sa` role labels. The only mitigation is still a runtime text-replacement hack (5322: "Keycloak"→"Steward", "Super Admin"→"Organization Owner", "Platform Admin"→"Organization provisioning service") plus removal of `#nav-pa` / `[id^="pa-"]` / `[id^="ap-role"]` at load.

Partial progress: some Org Admin copy was updated to the correct role names (696, 764 now say "Hiring Manager, Hiring Reviewer, Hiring Approver" / "Organisation Owner / Organisation Admin"). But the role system, demo accounts, and `RVR` role-request flow still carry the old labels.

---

## 7. Prioritised issue list (current)

1. **P2 — Role-model cleanup at source.** Replace "Keycloak"→"Steward" in the HTML/JS, drop "Super Admin"/"Platform Admin"/"Requisition Creator" in favour of `ORGANISATION_OWNER`/`ORGANISATION_ADMIN`/`HIRING_MANAGER`, align `oa`/`sa`, and remove the load-time text-masking hack once strings are corrected. This is now the last substantive conformance gap.
2. **P2 — Remove the duplicate legacy atlas renderer block** (5168–5198), which is dead code referencing a superseded schema.
3. **P3 — (optional) reporting depth** — a funnel/SLA view would round out phase 13.
4. **P3 — (optional) promote `mwVerification` and `mwDuplicates`** from the console into the primary `hr-*` body for full lifecycle parity.
5. **P3 — (nit) unify the pre-screen decision state** (`primaryDecisionRecords.prescreen`) with the requisition/opening/final state model.

---

## 8. What is strong (preserve)

- AI-advisory discipline, immutability, "no composite score / weights rank only".
- Blind-review reveal gate + pre-screen exemption.
- Independent, immutable interviewer evaluations with no peer/AI-score leakage.
- Offer → onboarding → employee-record → position-fill chain with correct capacity accounting.
- Tenant isolation + explicit cross-organisation access-denied demo.
- Honest "simulated delivery / no real backend" labels.
- **New this pass:** functionally gated Reviewer → Approver → HR handoff with distinct actors, rationale, timestamp, and correlation IDs; three-step in-view controls.

---

## 9. Appendix — key line references (current file)

- SoD controls: `setPrimaryDecisionControls` 3786, `recordPrimaryRecommendation` 3795, `communicatePrimaryOutcome` 3805, `recordHrPrescreenDecision` 3814, `recordHrFinalDecision` 4792, `saveHrRequisitionStatus` 4985, `saveHrOpeningStatus` 5028; buttons 1771, 1826, 2020, 2166.
- `.grid3`: 142 (definition), 215 (responsive).
- Commented-out duplicates: 3159–3210; active `renderMembershipDirectory` 3220, `updatePortalUserRole` 3233.
- Legacy vs canonical renderers: legacy 5168–5198, canonical 5206–5232.
- 56-table catalogue: `WF_TABLES` (entries at 5145, 5150); "56 database tables" labels near 5177/5184/5191.
- Role drift + masking: Keycloak/Super Admin/Platform Admin throughout; `superadmin` 2897; partial OA alignment 696, 764; masking 5322.
