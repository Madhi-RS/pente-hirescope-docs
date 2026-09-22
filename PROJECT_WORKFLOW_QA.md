# HireScope Project Questions, Decisions, and Structured Workflow

**Updated:** 2026-09-21  
**Protected baseline sources:** [HireScope.md](../HireScope.md), [hirescope-mockup-4.html](../hirescope-mockup-4.html)  
**Current website flow:** [website-user-flow.md](../website-user-flow.md)

## 1. Requirement precedence

The product requirements supplied on 2026-09-21 are the latest authority where they change the earlier work-first baseline or the 2026-09-18 workflow. Unaffected fairness, consent, immutable evidence, human-decision, versioning, and audit rules remain.

| Earlier active position | Current position |
|---|---|
| No résumé parsing/analysis in the initial release | Candidate details and résumé are collected first; AI extracts facts, compares them with opening criteria, may rank/recommend when configured, and flags potential discrepancies. |
| Manual pre-screen rules could qualify/reject before résumé collection | No candidate is rejected before required data/résumé collection. AI is advisory; HR makes every pre-screen progression decision. |
| A qualified rule outcome automatically created an assessment invitation | Only an HR-set `QUALIFIED_FOR_ASSESSMENT` decision creates/sends the link. |
| HR assessment review hid identity/résumé until a review lock | Authorised HR may see relevant candidate/resume/pre-screen/assessment evidence before live-interview progression. Seven-area AI scoring still excludes résumé/identity. |
| Audio re-record counts were fixed at one | Each opening configures whether re-recording is allowed and the required maximum; all versions are retained and only latest submitted is analysed. |
| Interruption provided grievance only | Interruption provides **Submit Test** and **Submit Grievance**; HR alone decides a second chance. |
| Live interview could be interpreted as a single result | One or more interviewers submit separate identity-linked scores/comments; no result overwrites another or AI. |
| Applicant outcomes omitted all evaluation detail | Email may communicate approved strengths/weaknesses, but never exact scores, rankings, internal comments, or confidential rationale. |

## 2. Confirmed product decisions

- Multi-organisation support is limited to organisation ownership, isolation, and authorised access needed for this workflow.
- Configuration is opening-level, not one global organisation setting.
- Audio is supported now. Video is planned only and cannot be enabled in an active opening.
- Network readiness is an applicant acknowledgement. HireScope performs no network-speed measurement.
- AI outputs are advisory and immutable; HR may comment, accept/disregard a recommendation, request reprocessing, and select versions, but cannot edit AI output.
- The seven work-assessment scores stay separate from résumé pre-screening.
- Text remains a human-approved accommodation unless later approved as a general opening modality.
- The existing live-interview ten categories and 1–5 scale apply; score-2 and score-4 anchor wording still needs approval.

## 3. Core records and identifiers

| Record | Purpose |
|---|---|
| Organisation | Ownership/isolation boundary for workforce users and recruitment data |
| Requisition | Organisation-owned headcount/business case with versioned draft/submitted/returned/resubmitted submissions |
| Opening/version | Organisation-owned job opening and immutable version history |
| Opening configuration | Per-opening/version modalities, sections, timing, questions, re-record maximum, interviewer count, and approved aggregation reference |
| Pre-screen criteria version | Opening-specific candidate/résumé requirements and review instructions |
| Application/packet | One applicant for one opening/version; never keyed only by global user ID |
| Candidate information/résumé | Protected intake evidence collected before pre-screen analysis |
| Résumé AI result version | Extracted facts, matches, recommendation/rank if configured, discrepancies, reasons, and provenance |
| HR pre-screen decision | Authoritative qualify/non-qualify/pending result linked to selected AI version |
| Assessment invitation | Unique expiring credential issued only after HR qualification |
| Readiness record | Configuration-pinned checks/acknowledgements/consent/final confirmation |
| Recording version | Application + section/question + attempt/version + time + active/latest status |
| Assessment packet/result | Submitted work/latest responses and immutable seven-area AI result version |
| Grievance/second-attempt decision | Interruption reason/status and HR-authorised retry/non-progression |
| Interview event/participant result | Multi-participant assignment and separate immutable scores/comments |
| Final HR decision | Human `ADVANCED`, `HOLD`, or `REJECTED` with evidence references |

## 4. Organisation and access boundary

Every requisition, opening, configuration, application, résumé, analysis, assessment, interview, decision, notification, and audit lookup is organisation-owned. Workforce identity claims are insufficient by themselves: APIs/services check organisation scope plus role, ownership, application, and assignment relationships. Cross-organisation discovery, read, mutation, association, aggregate, and export are denied and audited.

Applicants see only their own application. Interviewers see only assigned events and the event-authorised brief. HR and Hiring Managers are constrained to their organisation and their role authority.

## 5. Opening configuration boundary

The canonical configuration matrix is maintained in `US-OPN-05` in [hirescope_requisition_opening_phase1_stories.md](./phased-stories/hirescope_requisition_opening_phase1_stories.md).

Required current configuration covers audio, enabled assessment sections, question count/content, expected duration/time limit, recording/re-record allowance and maximum, pre-screen criteria, and required interviewer count. Video, general text mode, section timers/preparation time, disabling core AI analysis, live-stage optionality, and allowed aggregation formulas remain decision-gated as stated there.

Configuration is approved/versioned with the opening, pinned to each application, retrieved at assessment/interview entry, and isolated from every other opening.

## 6. Candidate intake and résumé AI pre-screening

1. Applicant chooses an active opening and submits its required information and résumé.
2. Validation failures do not create candidate rejection.
3. AI processes only after collection is complete and uses the pinned opening criteria.
4. For one or many résumés, AI may extract qualifications/facts, compare criteria, produce advisory match/recommendation/rank output, and explain evidence.
5. Cross-field checks may flag potential discrepancies when submitted facts support comparison.
6. Flags never assert lying/dishonesty and do not use invented verification sources.
7. Each result is immutable/versioned and cannot change status.
8. Authorised HR reviews the original résumé, extracted facts, criteria, analysis/recommendation/reasons, discrepancies, and result version.
9. HR accepts/disregards the recommendation, adds a separate comment, and records the authoritative decision/reason.
10. Only HR qualification issues the assessment link; non-qualification sends the approved outcome/reason email.

## 7. Applicant readiness and assessment

Before start, a dedicated page presents quiet/private environment, interruptions, background noise, device, connectivity risk, expected duration/portal time, consent, and final readiness confirmation. Audio-enabled openings also require microphone availability/permission/function/playback and applicant readiness. Audio-disabled openings omit those steps. Planned video steps are absent. Network requires acknowledgement only.

`ASSESSMENT_STARTED` cannot occur before all configured prerequisites and final confirmation.

The applicant then receives only enabled sections/questions/timing from their pinned opening configuration. Drafts autosave; final submission locks the packet.

## 8. Recording and re-recording

- Re-record control appears only when enabled.
- Enabling it requires a configured maximum.
- Initial and re-recorded clips are separate immutable versions.
- Version/attempt, timestamp, application, section/question, and active/latest flag are retained.
- Submitting a new version changes active/latest selection but does not delete history.
- Transcription, voice comparison, and seven-area response analysis use only latest submitted active versions.
- Historical access and deletion follow the approved retention policy.

## 9. Interruption, grievance, and second attempt

The existing five-minute inactivity trigger and one-minute resolution window remain unless product later changes them. On trigger, state is preserved and two options appear:

- **Submit Test:** validate and submit current permitted packet for normal AI processing.
- **Submit Grievance:** require reason, create `ASSESSMENT_GRIEVANCE_SUBMITTED`, lock without AI processing, and queue HR.

Window expiry records `ASSESSMENT_INACTIVITY_ENDED`, locks without scoring, and signs out. HR records `SECOND_ATTEMPT_AUTHORISED` or `SECOND_ATTEMPT_NOT_AUTHORISED` with reason. Only the former creates one linked new attempt. No technical event or grievance automatically grants a retry/pause/extension/offline completion.

## 10. AI assessment and HR live-progression review

Seven-area assessment AI input excludes résumé, résumé AI output, pre-screen result, source, salary, optional context, and identity. It uses the frozen FairPack, work artifact, latest active submitted answers, and permitted behavior evidence. It produces seven immutable 1–4 results with evidence/rationale/confidence and no composite/decision.

Authorised HR may then review relevant candidate information, résumé, pre-screen evidence, assessment artifacts/results, AI scores/analysis, and reviewer comments before setting `LIVE_INTERVIEW_RECOMMENDED`, `LIVE_INTERVIEW_HOLD`, or `NOT_PROCEEDING`. All sources stay clearly labelled and immutable.

## 11. Multiple-interviewer live stage

The event requires the participant count/roles from the pinned opening configuration. Each interviewer sees only their assigned, authorised read-only brief and completes the ten existing categories on the 1–5 scale with evidence comments.

Every score/comment records candidate/application, interview, interviewer, category, submission status, and time. Draft ownership and immutable submission are participant-specific. All required submissions create factual `INTERVIEW_COMPLETED`; they do not create an outcome.

AI pre-screen, AI assessment, each interviewer, reviewer comment, optional derived aggregation, and final HR decision remain distinct. Aggregation is permitted only under a selected approved versioned policy; individual results are never overwritten. Allowed aggregation methods are a pending product decision.

## 12. Final HR decision and communication

After `INTERVIEW_COMPLETED`, HR alone records `ADVANCED`, `HOLD`, or `REJECTED`, with organisation, actor, time, reason, evidence/result versions, and aggregation policy/version if used.

Applicant emails may contain approved evidence-based strengths and weaknesses. Exact AI/interviewer/aggregate scores, ranking, internal comments, and confidential rationale are prohibited.

## 13. Notifications and failure behavior

| Event | Required behavior |
|---|---|
| Approval requested/decided | Notify authorised HR/Hiring Manager by email with permitted content |
| HR pre-screen qualified | Send unique bound assessment invitation |
| HR pre-screen not qualified | Send approved outcome/reason; no link |
| AI processing failure | Preserve source, alert authorised HR/operations, allow versioned retry |
| Interruption/grievance HR decision | Notify applicant of second chance/non-progression |
| Interview assignment/reassignment | Notify every affected participant |
| Missing required participant result | Show incomplete; do not set `INTERVIEW_COMPLETED` |
| Applicant outcome | Email outcome and approved strengths/weaknesses without exact/internal scores |

## 14. Audit requirements

Audit organisation ownership/access, requisition/opening/configuration versions, pre-screen criteria, candidate/résumé collection/access, AI résumé extraction/matches/ranking/discrepancies/result versions, HR pre-screen decisions, invitation lifecycle, readiness/consent, saves/submission, every recording version/active selection, interruption/grievance/second-attempt decision, assessment AI versions/retry, HR live progression, interviewer assignments/access/drafts/submissions, aggregation policy/use, final HR decision, email delivery, position-fill/closure, and all denied confidential access.

## 15. Decisions still required before implementation completion

1. Video support date and full requirements; current status is planned only.
2. Whether general text modality is supported beyond accommodation.
3. Section-specific duration and applicant preparation time.
4. Whether résumé pre-screen AI or core assessment AI can be disabled per opening.
5. Whether the live-interview stage can be disabled for an opening.
6. Approved interviewer-score aggregation methods and rounding/missing-submission behavior.
7. Formal approval of live-interview score-2 and score-4 anchors.
8. Exact role-family pre-screen criteria, AI evaluation criteria, and discrepancy-check catalog.
9. Résumé, audio/version, assessment, interview, candidate-data, and audit retention schedules.
10. Organisation membership/delegation lifecycle and approved administrators.
11. Definition/correction authority for `POSITION_FILLED` and early closure.

## 16. Final workflow boundary

HireScope includes organisation isolation; organisation-owned, opening-configured recruitment; candidate/résumé-first advisory AI pre-screening; authoritative HR invitation decisions; configuration-specific readiness/assessment; versioned audio attempts; human-reviewed interruption grievances; immutable assessment AI; HR live-progression review with relevant candidate information; multi-interviewer individual evidence; human final decisions; safe strength/weakness emails; audit; headcount; and closure.

It still does not allow AI to make candidate decisions, expose exact scores to applicants, silently overwrite AI/human evidence, activate video before approval, measure network speed, merge interviewer evidence without approved policy, or include a full offer/onboarding/HRMS workflow.
