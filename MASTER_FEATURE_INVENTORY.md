# HireScope Master Feature Inventory

**Updated:** 2026-09-21  
**Purpose:** Current documentation-only feature boundary before architecture/data-model/build work. The latest product requirement supersedes conflicting older source statements.

## Phase plan

| Phase | Outcome | Story files |
|---|---|---|
| Phase 1 | Keycloak/role/organisation access, requisition approval, organisation-owned opening, per-opening configuration, and immutable publication | `hirescope_auth_phase1_stories.md`, `hirescope_identity_phase1_stories.md`, `hirescope_requisition_opening_phase1_stories.md` |
| Phase 2 | Candidate/résumé collection, advisory AI pre-screen, HR invitation decision, readiness, configured assessment, recording versions, interruption/grievance | `hirescope_auth_phase2_stories.md`, `hirescope_candidate_intake_phase2_stories.md`, `hirescope_assessment_phase2_stories.md` |
| Phase 3 | Immutable assessment AI, HR live-progression review, multiple-interviewer evidence, final HR decision, notification/audit/headcount closure | `hirescope_ai_hr_review_phase3_stories.md`, `hirescope_interview_decision_phase3_stories.md`, `hirescope_operations_phase3_stories.md` |

## Inventory

| Domain | Feature | Role | Source | Main behavior | Dependencies | Phase | Story |
|---|---|---|---|---|---|---:|---|
| Identity | Organisation-scoped role/record access | All | New requirement; Flow/QA | Enforce role + organisation + ownership/application/assignment; audit denials | Identity provider, organisation membership | 1 | IAM-03, AUTH-03 |
| Requisition | Draft/submit/approve/return/resubmit | Hiring Manager, HR | Flow/QA; new requirement | Organisation-owned versioned draft; authorised HR approve/return; resubmission loop; email outcome notification; headcount gate | Identity/organisation | 1 | REQ-01/02 |
| Opening | Organisation-owned FairPack | Hiring Manager, HR | HireScope.md; new requirement | Draft seven-area instrument; HR approve/return; package resubmission; decision email; isolate organisation/opening | Approved requisition | 1 | OPN-01 |
| Opening | Version/change | Hiring Manager, HR | HireScope.md; Flow/QA; new requirement | Immutable active versions; in-flight pinning; config cannot bypass approval; change resubmission; target-opening-only changes | OPN-01 | 1 | OPN-02 |
| Opening | External source/application link | Hiring Manager, HR | Flow/QA | Record source; public link grants intake only | Active opening | 1 | OPN-03 |
| Opening | Pre-screen criteria | Hiring Manager, HR | New requirement | Opening-specific candidate/résumé criteria and mandatory HR review procedure | OPN-01 | 1 | OPN-04 |
| Opening | Assessment delivery configuration | Organisation user, HR | New requirement | Per-opening audio/sections/questions/timing/re-record max/interviewer count; video planned | OPN-01/02 | 1 | OPN-05 |
| Identity | Secure assessment invitation | Applicant, HR/System | Flow/QA; new requirement | Unique bound invitation only after HR qualification | CAN-03 | 1–2 | IAM-02, AUTH-06 |
| Candidate Intake | Candidate details/résumé collection | Applicant | New requirement | Collect complete application before analysis/rejection; protected/audited | Active opening/criteria | 2 | CAN-01 |
| Candidate Intake | AI résumé pre-screen/discrepancy | AI, HR | New requirement | Extract/compare/rank when configured; cautious discrepancy evidence; immutable versions | CAN-01, OPN-04 | 2 | CAN-02 |
| Candidate Intake | HR pre-screen decision/test link | HR | New requirement | Review complete evidence; accept/disregard AI; decide; link only after HR qualification | CAN-02 | 2 | CAN-03 |
| Candidate Intake | Applicant status/outcome email | Applicant | New requirement | Own state; reason/strengths/weaknesses; no exact/internal scores | HR decisions | 2 | CAN-04 |
| Assessment | Configured versioned assessment | Applicant | New requirement; HireScope.md | Load/enforce correct opening configuration; autosave; final lock | OPN-05, HR invitation | 2 | ASM-01 |
| Assessment | Readiness and consent | Applicant | New requirement | General/time/network acknowledgement; audio checks when enabled; no speed/video check | ASM-01 | 2 | ASM-02 |
| Assessment | Versioned audio/re-record | Applicant | New requirement | Enforce configured max; retain versions; latest submitted used | OPN-05, ASM-02 | 2 | ASM-03 |
| Assessment | Typed accommodation | Applicant, HR | HireScope.md | Human-approved equivalent route; voice N/A | Accommodation decision | 2 | ASM-04 |
| Assessment | Interruption/grievance/second attempt | Applicant, HR | New requirement | Submit Test or Submit Grievance; HR authorises/denies linked retry | Active attempt | 2 | ASM-05 |
| AI Review | Seven-area assessment AI | AI, HR | HireScope.md; new requirement | Latest active answers; immutable 1–4 evidence; no composite/decision | Submitted packet | 3 | REV-01 |
| AI Review | Failure/reprocessing | HR/Ops | Flow/QA | Preserve source; versioned retry; selected result version | REV-01 | 3 | REV-02 |
| AI Review | HR live-progression review | HR | New requirement | View relevant candidate/resume/prior evidence; human decision; immutable AI | Analysis ready | 3 | REV-03 |
| AI Review | Voice integrity | HR | HireScope.md | Cautious within-speaker flag pinned to active versions; no automatic effect | Voice versions/consent | 3 | REV-04 |
| Live Interview | Multiple participant assignment/brief | HR, Interviewer | New requirement | Pinned required count; assignment-scoped relevant read-only evidence | HR recommendation, OPN-05 | 3 | INT-01 |
| Live Interview | Individual ten-area scores/comments | Interviewer | Flow/QA; new requirement | Existing 1–5 rubric; per-interviewer provenance; optional approved aggregation | INT-01 | 3 | INT-02 |
| Live Interview | Draft/submit/completion | Interviewer/System | New requirement | Owner draft; immutable individual result; all-required factual completion | INT-02 | 3 | INT-03 |
| Decision | Final HR disposition/email | HR | Flow/QA; new requirement | Separate evidence review; `ADVANCED`/`HOLD`/`REJECTED`; strengths/weaknesses email | `INTERVIEW_COMPLETED` | 3 | DEC-01 |
| Operations | Organisation-scoped funnel | Hiring Manager, HR | Flow/QA; new requirement | Role/organisation-specific aggregates/drill-down | Workflow states | 3 | OPS-01 |
| Operations | Notifications/failure queues | HR/Ops/System | Flow/QA; new requirement | HR-gated invitation, grievance/result notices, safe strengths/weaknesses | Domain events | 3 | OPS-02 |
| Operations | Audit | HR/Auditor | HireScope.md; new requirement | Append-only provenance/access across all new and retained workflows | All domains | 3 | OPS-03 |
| Operations | Headcount/closure | HR/Owner | Flow/QA | Separate fills; deliberate read-only closure | Final decisions | 3 | OPS-04 |

## Cross-cutting invariants

1. Organisation membership never substitutes for role/record checks, and no cross-organisation data operation is allowed.
2. Opening configuration is versioned per opening, never organisation-global, requires HR approval to create or change, and an applicant is pinned to one version.
3. Candidate details/résumé precede AI pre-screening; AI never makes the HR decision or sends the assessment link.
4. Résumé pre-screen AI and seven-area assessment AI are different evidence sources. Résumé data cannot alter the seven work-assessment scores.
5. Readiness is required before start. Audio checks are conditional; video is planned only; network speed is not measured.
6. Recording history is immutable; the latest submitted active version alone is analysed.
7. Interruption offers Submit Test and Submit Grievance; only HR can authorise a linked second attempt.
8. Authorised HR can review relevant candidate/prior evidence before live progression, but cannot edit AI output.
9. Each interviewer's result remains separate. Aggregation, if used, is derived under an approved versioned opening policy and never overwrites sources.
10. Applicant email may state approved strengths/weaknesses but never exact scores, rankings, internal comments, or confidential rationale.

## Decision-gated inventory

| Item | Current status | Register |
|---|---|---|
| Video modality | Planned only; cannot activate | DEC-01 |
| General text modality | Accommodation-only | DEC-02 |
| Section duration/preparation time | Not defined | DEC-03 |
| Per-opening AI disablement | Not defined | DEC-04 |
| Live-stage optionality | Not defined | DEC-05 |
| Interviewer aggregation formulas | Not defined | DEC-06 |
| Score-2/4 live anchors | Pending HR approval | DEC-07 |
| Exact pre-screen/evaluation/discrepancy catalogs | Not defined | DEC-08/09 |
| Data/recording retention | Not defined | DEC-10/11 |
| Organisation administration | Not defined | DEC-12 |
| Secure pre-qualification intake identity mechanism | Not defined | DEC-13 |
| Repeat retest policy | Not defined | DEC-14 |
