# HireScope — Configured Assessment, Readiness, and Voice — Phase 2 User Stories

## Scope and rules

An applicant receives the immutable assessment configuration pinned to their organisation/opening/version. Audio is currently supported; video is planned only. Disabled modalities and sections do not appear. The applicant completes a dedicated readiness page before the assessment begins. Recording versions are retained and only the latest submitted version is active for analysis. Interruption offers **Submit Test** or **Submit Grievance**; only HR can authorise a second attempt.

# US-ASM-01: Load and complete the configured versioned assessment

## User Story

As an applicant, I want the assessment for my specific opening and version, so that I complete only the sections, questions, modality, and timing that the organisation approved for that opening.

## Description

Assessment entry loads the configuration bound to the applicant's application. The work sample uses the frozen FairPack and designed missing input where enabled. Drafts autosave and final submission locks the packet. A configuration from another opening or organisation must never leak into this experience.

## Key Capabilities

- Retrieve and enforce pinned opening configuration.
- Display enabled sections/questions and expected duration.
- Autosave/reconnect and immutable final submission.
- Keep optional distance-travelled context non-scoring.

## Acceptance Criteria

- AC-01: Entry retrieves the application-bound organisation, opening, opening version, FairPack, and configuration and refuses a missing/mismatched/unpublished configuration.
- AC-02: Only enabled supported modalities, sections, configured question count/content, expected duration, and timing rules are presented and enforced.
- AC-03: Work drafts autosave and reconnect restores only permitted state under the pinned timer/configuration rules.
- AC-04: Final submission locks the artifact/packet and retains the application, opening, configuration, and FairPack version references.
- AC-05: Optional distance-travelled context remains non-scoring and excluded from seven-area AI analysis.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| ASM-01-TC-01 | Correct configuration retrieval | Two openings/configurations exist | Enter each applicant assessment; tamper version/opening | Correct version loads; mismatch is refused (AC-01) |
| ASM-01-TC-02 | Enabled-only experience | Sections/modalities differ | Compare assessments | Only configured supported content appears (AC-02) |
| ASM-01-TC-03 | Draft/reconnect | Work in progress | Edit, disconnect, reconnect | Permitted state restores under same configuration (AC-03) |
| ASM-01-TC-04 | Final lock/version pin | Draft exists | Submit then edit/inspect references | Packet is immutable and fully version-linked (AC-04) |
| ASM-01-TC-05 | Context exclusion | Optional context supplied | Inspect AI input/result | Context is absent and non-scoring (AC-05) |

# US-ASM-02: Complete opening-specific environment readiness before starting

## User Story

As an applicant, I want a clear readiness page tailored to my opening's enabled modality and expected duration, so that I can prepare my environment and explicitly confirm I am ready before the test begins.

## Description

The dedicated pre-interview page appears before `ASSESSMENT_STARTED`. It explains quiet/private space, interruptions, device readiness, connectivity risk, and time commitment. Audio openings include microphone permission/function/playback checks and noise guidance. Video instructions/checks are not shown because video is planned only. Network readiness is a declaration, not a speed measurement.

## Key Capabilities

- General environment and time-commitment instructions.
- Modality-specific preparation/checks.
- Explicit network-readiness acknowledgement without speed test.
- Consent and final readiness confirmation gate.

## Acceptance Criteria

- AC-01: The page states quiet/private location, avoidance of other people/interruptions/background noise, required device readiness, unstable-connectivity risk, expected duration, approximate portal time, and uninterrupted-session expectation.
- AC-02: When audio is enabled, the page checks microphone availability, permission, basic functionality/playback, and applicant-confirmed audio readiness; failure is retryable and never an integrity flag.
- AC-03: When audio is disabled, audio controls/instructions are absent. Video/camera/lighting/framing checks are absent while video remains planned and unavailable; they become required only in a future approved video configuration.
- AC-04: Network readiness requires an explicit applicant acknowledgement; no network-speed measurement or pass/fail threshold is performed.
- AC-05: No recording starts before required consent, and the assessment cannot enter `ASSESSMENT_STARTED` until required checks/acknowledgements and final readiness confirmation are complete.
- AC-06: Consent/readiness record stores application/opening/configuration version, completed requirements, actor, and time.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| ASM-02-TC-01 | General/time instructions | Assessment invited | Open readiness | All required environment/time/connectivity guidance appears (AC-01) |
| ASM-02-TC-02 | Audio readiness retry | Audio opening; microphone denied/fails | Attempt check, grant/fix, retry | Start is blocked until pass/confirmation; no integrity flag (AC-02) |
| ASM-02-TC-03 | Modality-specific omission | Audio-disabled opening | Open readiness and inspect for audio/video | Irrelevant audio and planned-video controls are absent (AC-03) |
| ASM-02-TC-04 | Network acknowledgement only | Readiness page open | Inspect traffic/UI and acknowledge | Acknowledgement is stored; no speed measurement/threshold exists (AC-04) |
| ASM-02-TC-05 | Start gate and consent | Requirements incomplete then complete | Attempt start both times | Start fails first and succeeds only after consent/readiness confirmation (AC-05) |
| ASM-02-TC-06 | Readiness audit | Applicant confirms | Inspect record | Correct version, completed requirements, actor/time exist (AC-06) |

# US-ASM-03: Capture and version configured audio responses

## User Story

As an applicant, I want to record and, where configured, re-record enabled audio sections within the opening's limit, so that my latest submitted answer is analysed while prior versions remain preserved.

## Description

The configured sequence may include self-introduction, read-a-passage, and question answers. Every recording is a distinct version associated with application, section/question, attempt number, timestamp, and active/latest status. A new version does not delete an earlier one. Re-record is unavailable when disabled and capped by the opening configuration when enabled.

## Key Capabilities

- Enabled-only audio sections and configured questions.
- Configurable, enforced re-record maximum.
- Immutable recording history with active/latest selection.
- Latest submitted recording used for transcript/AI analysis.

## Acceptance Criteria

- AC-01: Only enabled audio sections/questions from the pinned configuration appear, in their configured order; disabled sections cannot create recordings.
- AC-02: The initial recording and every permitted re-record receive a monotonically increasing version/attempt number, timestamp, section/question link, and active/latest status.
- AC-03: Re-record controls are absent when disabled; when enabled, attempts beyond the configured maximum are refused.
- AC-04: Submitting a newer version marks it active/latest without deleting or overwriting historical recordings; access/retention follows authorised product policy.
- AC-05: Transcription and AI response analysis use only the latest submitted active version while retaining provenance to prior versions.
- AC-06: Final packet submission locks all active selections and provides a receipt without scores.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| ASM-03-TC-01 | Enabled sections/questions | Configuration enables a subset | Progress through audio | Only configured content/order appears (AC-01) |
| ASM-03-TC-02 | Version metadata | Re-record enabled | Record three versions | Versions 1–3 have attempt/time/link/latest metadata (AC-02) |
| ASM-03-TC-03 | Limit enforcement | Maximum re-record count configured | Attempt one beyond maximum | Excess attempt is refused (AC-03) |
| ASM-03-TC-04 | History preservation | Three versions exist; v3 submitted | Inspect storage/view | v3 is active; v1/v2 remain immutable/authorised (AC-04) |
| ASM-03-TC-05 | Latest-only analysis | Several versions exist | Run transcription/analysis | Only latest submitted version is input; provenance is retained (AC-05) |
| ASM-03-TC-06 | Final packet lock | Active recordings selected | Submit final packet then modify | Selections lock; applicant sees no scores (AC-06) |

# US-ASM-04: Request and complete an approved typed accommodation

## User Story

As an applicant who cannot use the configured audio path, I want to request an accessible typed alternative, so that I can demonstrate the same job-related areas without penalty.

## Description

Typed assessment remains a human-approved accommodation, not a general opening modality unless product later approves that change. It maps to the same enabled assessment areas/version and records voice integrity as not applicable.

## Key Capabilities

- Separate accommodation request and authorised decision.
- Equivalent pinned assessment path.
- No voice flag or score effect from the request.

## Acceptance Criteria

- AC-01: The applicant can request accommodation without the request/details entering AI analysis or scores.
- AC-02: Only authorised HR approval enables the typed route and records permitted parameters.
- AC-03: The typed route maps to the same pinned opening/FairPack configuration and assessment areas.
- AC-04: Voice integrity is `N/A`; request details are visible only to authorised roles and necessary operational instructions omit confidential medical detail.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| ASM-04-TC-01 | Request isolation | Applicant at readiness | Request and inspect AI input | Request/details absent from scores (AC-01) |
| ASM-04-TC-02 | Human approval | Request pending | Self-enable then HR approves | Self-enable fails; authorised approval succeeds (AC-02) |
| ASM-04-TC-03 | Equivalent version | Typed route approved | Inspect prompts/areas/version | Pinned equivalent assessment is used (AC-03) |
| ASM-04-TC-04 | Privacy/voice N/A | Typed packet exists | Inspect projections | Voice is N/A and sensitive details are restricted (AC-04) |

# US-ASM-05: Resolve an interrupted attempt and submit a grievance for HR decision

## User Story

As an applicant whose assessment is interrupted, I want to submit my current test or submit a grievance with a reason, so that I can finish safely or ask HR for a second chance without receiving one automatically.

## Description

When the existing interruption/inactivity rule is triggered, HireScope preserves the latest permitted state and presents exactly two choices: **Submit Test** and **Submit Grievance**. Submit Test deliberately submits the permitted current packet for normal processing. Submit Grievance locks the interrupted attempt without AI scoring and routes the required reason to HR. HR decides whether to authorise one new attempt or end progression. No grievance automatically grants a retry.

## Key Capabilities

- Preserve state and present both required interruption options.
- Deliberate current-packet submission.
- Grievance reason/status and organisation-scoped HR queue.
- Human second-attempt authorisation or non-progression decision.
- Applicant notification and complete audit.

## Acceptance Criteria

- AC-01: On an interruption/inactivity event, the latest permitted state is preserved and the applicant sees **Submit Test** and **Submit Grievance**; the system does not silently choose either.
- AC-02: **Submit Test** validates/submits the current permitted packet, sets normal submission state, locks it, and makes it eligible for normal AI processing.
- AC-03: **Submit Grievance** requires a reason, creates `ASSESSMENT_GRIEVANCE_SUBMITTED`, locks the interrupted attempt without AI scoring, records applicant/application/opening/attempt/time, and queues authorised HR.
- AC-04: The existing one-minute interruption-resolution window remains; expiry without either action records `ASSESSMENT_INACTIVITY_ENDED`, locks the attempt without AI scoring, and safely signs the applicant out.
- AC-05: HR records `SECOND_ATTEMPT_AUTHORISED` or `SECOND_ATTEMPT_NOT_AUTHORISED` with reason, actor, and time; only authorisation creates a new attempt linked to the prior attempt/configuration.
- AC-06: Non-authorisation prevents progression from that attempt; both decisions send an appropriate applicant email without internal scores/comments.
- AC-07: No interruption, grievance, or technical failure automatically grants a pause, extension, offline continuation, or second attempt; all events and access are audited.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| ASM-05-TC-01 | Required choices | In-progress saved attempt | Trigger interruption | State preserved; exactly both options appear (AC-01) |
| ASM-05-TC-02 | Submit current test | Interrupted attempt has permitted content | Choose Submit Test | Packet locks and enters normal processing (AC-02) |
| ASM-05-TC-03 | Submit grievance | Interrupted attempt | Submit without/with reason | Reason required; grievance/HR queue created; no AI scoring (AC-03) |
| ASM-05-TC-04 | Window expiry | Interruption page open | Let one minute expire | Inactivity-ended state, locked unsent attempt, and sign-out occur (AC-04) |
| ASM-05-TC-05 | HR second-chance decisions | Grievance pending | Authorise one; do not authorise another | Only authorised case creates linked new attempt (AC-05) |
| ASM-05-TC-06 | Applicant notification | HR decides both cases | Inspect applicant status/email | Appropriate outcome sent without internal scores/comments (AC-06) |
| ASM-05-TC-07 | No automatic retry/audit | Interruption and grievance events exist | Inspect state/history | No automatic benefit; complete event/decision/access audit exists (AC-07) |
