# HireScope — Feature Finalization

**Updated:** 2026-09-21  
**Documentation stage:** Requirements/user stories only. This document does not authorise implementation.

## 1. Current product boundary

HireScope is a multi-organisation hiring-assessment platform in which each organisation owns its requisitions, job openings, applicants, and decisions. Configuration is versioned per job opening, not globally per organisation. The product collects candidate information and résumés, uses AI for advisory pre-screen analysis, requires HR to decide who receives an assessment link, delivers the configured assessment after readiness confirmation, uses AI to analyse submitted work, supports HR-owned live-interview progression, captures separate results from one or more interviewers, and requires HR to make the final decision.

AI never qualifies, rejects, invites, advances, or makes the final hiring decision.

The platform does not yet support video assessment; video is planned only. It does not measure network speed. It does not merge or overwrite AI/human evidence, expose exact scores to applicants, or include offer/onboarding/payroll functionality.

## 2. Non-negotiable rules

1. Every recruitment record belongs to one organisation; cross-organisation access and association are denied and audited.
2. Every application is pinned to one approved opening/configuration/FairPack version.
3. Two openings in the same organisation may have different configurations.
4. Candidate details and résumé are collected before AI pre-screening and before any HR non-qualification decision.
5. Résumé AI is advisory, explainable, immutable, and separate from the seven-area work-assessment AI.
6. HR is authoritative for pre-screen progression, live-interview progression, grievance second chance, and final disposition.
7. The assessment link is issued only after an HR-set qualified decision.
8. Readiness confirmation is required before assessment start. Network is acknowledged, not measured.
9. Audio is current; video is planned only; typed response is an approved accommodation unless later expanded.
10. Re-recording, when enabled, has an opening-configured maximum. All versions remain; latest submitted active version alone is analysed.
11. Interruption presents **Submit Test** and **Submit Grievance**. A grievance never automatically grants a retry.
12. Authorised HR may review relevant candidate/resume/prior evaluation information before live-interview progression.
13. One or more interviewers may participate. Each result remains individually attributable and immutable.
14. Any aggregation is a derived view under an approved versioned opening policy and never replaces individual scores.
15. Applicant emails may provide approved strengths/weaknesses but never exact scores, rankings, internal comments, or confidential rationale.

## 3. Roles and access

| Role | Owns | May decide | Must not do |
|---|---|---|---|
| Hiring Manager / Organisation User | Own organisation requisitions/opening drafts/configuration proposals | No candidate decision by default | Cross organisation boundary; self-approve; publish without approval |
| HR Reviewer | Approvals, pre-screen review, grievance decision, live progression, final decision | Candidate progression and final statuses | Edit AI output; access outside organisation scope |
| Applicant | Own application, résumé, readiness, assessment, grievance | Readiness/submission/grievance choices only | View internal/exact scores or other applicants |
| Interviewer | Assigned event and own rubric submission | Own evaluation only | Change candidate status; submit as peer; edit AI/other results |
| AI/System | Draft/extract/compare/analyse/score technically | None | Candidate state transition or unapproved data use |

## 4. Phase and story ownership

| Capability | Phase | Canonical stories |
|---|---:|---|
| Organisation/role isolation | 1 | IAM-03, AUTH-03 |
| Requisition (draft → submit → approve/return → resubmit; email outcome) and organisation-owned opening | 1 | REQ-01/02, OPN-01 |
| Version/publish/change (config cannot bypass approval) | 1 | OPN-02, OPN-05 |
| External source/intake link | 1 | OPN-03 |
| Opening-specific pre-screen criteria | 1 | OPN-04 |
| Opening assessment/interview configuration | 1 | OPN-05 |
| Candidate details/résumé collection | 2 | CAN-01 |
| Advisory AI résumé screening/discrepancy | 2 | CAN-02 |
| HR pre-screen decision/test-link gate | 2 | CAN-03, IAM-02, AUTH-06 |
| Applicant status/outcome communication | 2 | CAN-04 |
| Configured assessment retrieval/completion | 2 | ASM-01 |
| Environment readiness/consent | 2 | ASM-02 |
| Recording/re-record version history | 2 | ASM-03 |
| Typed accommodation | 2 | ASM-04 |
| Interruption/grievance/second attempt | 2 | ASM-05 |
| Seven-area assessment AI | 3 | REV-01/02 |
| HR live-progression review | 3 | REV-03 |
| Voice-integrity review | 3 | REV-04 |
| Multiple-interviewer assignment/results | 3 | INT-01–03 |
| Final HR disposition/email | 3 | DEC-01, OPS-02 |
| Reporting/audit/headcount/closure | 3 | OPS-01–04 |

No new domain or phase is required.

## 5. Opening configuration

The full category analysis is maintained under `US-OPN-05`. Current required opening-level settings include:

- audio enabled/disabled;
- enabled self-introduction, read-passage, and question-answer sections;
- question count and definitions;
- expected duration/time limit;
- recording and re-record allowance/maximum;
- pre-screen criteria version;
- required live-interviewer count; and
- aggregation/display policy reference when an approved policy is used.

Decision-gated settings include video, general text mode, section-specific timers, preparation time, AI disablement, live-stage optionality, and allowed aggregation formulas.

## 6. Candidate intake and AI résumé analysis

Input is the candidate's complete configured details and original résumé for one opening/version. AI output may contain extracted qualifications/facts, criterion matches, advisory analysis, recommendation/ranking when configured, reasons/evidence, and potential discrepancies. Every output is versioned and application-attributable.

Discrepancy wording must identify inconsistent/conflicting information or information requiring verification. It must not assert dishonesty or invent external verification sources.

HR reviews the original and derived evidence, may accept or disregard the recommendation, adds a separate comment, and records the actual decision. AI output cannot be edited and cannot issue the assessment link.

## 7. Applicant assessment

Readiness is a required state between invitation and start. It covers environment, time commitment, connectivity acknowledgement, consent, and the enabled modality's checks. Audio-disabled openings do not show microphone steps. Video steps remain unavailable. No speed test occurs.

The assessment loads the application-pinned configuration. Disabled sections are absent. Drafts autosave. Final submission is immutable. An approved typed accommodation measures the same configured areas and records voice integrity as not applicable.

Recording attempts are immutable versions. Re-record control and maximum come from the opening. The latest submitted active version is selected for transcription/analysis; historical versions remain under access/retention policy.

## 8. Interruption/grievance

The existing five-minute inactivity trigger and one-minute resolution window remain. The page provides both required choices. Submit Test locks/submits the permitted current packet. Submit Grievance requires a reason and locks the attempt without AI scoring. Expiry locks without scoring and signs out.

HR either authorises one linked second attempt or does not authorise it. A repeat-attempt policy remains a product decision. The applicant receives the HR outcome.

## 9. AI work assessment and HR progression

Assessment AI excludes résumé/pre-screen/identity data and uses the frozen pack, work artifact, latest active responses, and permitted evidence. It produces seven immutable 1–4 results with pins/rationale/confidence and no composite or status action.

Authorised HR may review relevant candidate details, résumé, pre-screen evidence, work assessment, AI results, and separate comments before setting live-interview progression. This visibility change does not permit résumé data to alter the seven AI scores.

## 10. Live interview and final decision

The opening version defines the required interviewer count. Each assigned interviewer receives a read-only permitted brief and independently completes the existing ten categories on the existing 1–5 scale. Scores 2 and 4 remain blocked until anchor wording is approved. Each submission is identity-linked, immutable, and separate.

All required submissions create factual `INTERVIEW_COMPLETED`. HR then reviews separate evidence and records `ADVANCED`, `HOLD`, or `REJECTED`. The outcome email may provide strengths/weaknesses without exact/internal scores.

## 11. Lifecycle

```text
INTEREST_REGISTERED
→ APPLICATION_DETAILS_REQUIRED
→ RESUME_SUBMITTED
→ AI_PRE_SCREENING
→ PRE_SCREEN_ANALYSIS_READY
→ UNDER_HR_PRE_SCREEN_REVIEW
→ QUALIFIED_FOR_ASSESSMENT | NOT_CURRENTLY_QUALIFIED | PRE_SCREEN_PENDING

QUALIFIED_FOR_ASSESSMENT
→ ASSESSMENT_INVITED
→ READINESS_PENDING
→ READINESS_CONFIRMED
→ ASSESSMENT_STARTED
→ ASSESSMENT_SUBMITTED | ASSESSMENT_GRIEVANCE_SUBMITTED
→ AI_PROCESSING → ANALYSIS_READY | AI_PROCESSING_FAILED
→ UNDER_HR_REVIEW
→ LIVE_INTERVIEW_RECOMMENDED | LIVE_INTERVIEW_HOLD | NOT_PROCEEDING

LIVE_INTERVIEW_RECOMMENDED
→ INTERVIEW_ASSIGNED
→ INTERVIEW_COMPLETED
→ ADVANCED | HOLD | REJECTED
```

`SECOND_ATTEMPT_AUTHORISED` and `SECOND_ATTEMPT_NOT_AUTHORISED` branch from an assessment grievance. `POSITION_FILLED` is a separate downstream record.

## 12. Audit and reporting boundary

The audit is append-only and covers organisation scope, configuration lifecycle/retrieval, candidate/résumé access, both AI domains and versions, discrepancy evidence, HR decisions, invitations, readiness/consent, recording versions, interruption/grievance/retest, interviewer assignments/results, aggregation policy/use, final outcome/email, headcount/closure, and denied access.

Reporting is operational and organisation-scoped. It may show authorised lifecycle counts and separate evidence statuses. It does not expose unauthorised candidate data or create cross-organisation analytics.

## 13. Decisions required

See [REQUIREMENTS_GAPS.md](./REQUIREMENTS_GAPS.md) DEC-01 through DEC-14. The main blockers are video enablement, modality/timing/AI/live optionality, aggregation formulas, live score-2/4 anchors, exact pre-screen/AI criteria catalogs, retention schedules, organisation administration, secure pre-qualification intake identity, and repeat-retest limits.

## 14. Explicitly out of scope

- AI-only candidate progression or final decisions.
- Editing/overwriting AI or interviewer evidence.
- Active video assessment before approval.
- Network speed measurement.
- Unapproved combined/aggregate score.
- Applicant access to exact/internal scores or comments.
- Cross-organisation discovery, reports, or associations.
- Offer, onboarding, payroll, and general HRMS workflows.
- Claiming a discrepancy proves lying/dishonesty.
