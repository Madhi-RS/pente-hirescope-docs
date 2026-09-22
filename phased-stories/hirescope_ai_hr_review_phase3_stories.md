# HireScope — AI Assessment and HR Review — Phase 3 User Stories

## Scope and rules

This domain owns AI analysis of the submitted work assessment, not the earlier résumé pre-screening in CAN-02. Assessment AI receives only the frozen FairPack/anchors, work artifact, latest active submitted response versions, and permitted delivery evidence. It produces immutable evidence-backed results and never decides progression. The latest product requirement removes the former restriction that prevented authorised HR from viewing relevant candidate information before the live-interview progression decision. Résumé/candidate information still cannot alter the seven AI assessment scores.

# US-REV-01: Produce immutable, evidence-backed seven-area AI results

## User Story

As an HR Reviewer, I want one evidence-backed immutable AI result for each of the seven assessment areas, so that I can review job-related work without a hidden composite decision.

## Description

Each area is scored 1–4 with pinned evidence, rationale, confidence, model/configuration metadata, and time. For an audio item with multiple versions, only its latest submitted active version is analysed; recording-version provenance is retained.

## Key Capabilities

- Strict assessment-input boundary.
- Latest-active response selection with version provenance.
- Per-area results, not a hire percentage/threshold.
- Immutable, versioned output.

## Acceptance Criteria

- AC-01: AI assessment input excludes résumé pre-screen output, résumé, source, salary, optional context, identity, school, employer brand, and years; those records remain available separately to authorised HR where permitted.
- AC-02: For each response/section, analysis uses only the latest submitted active recording/text version and records its application/question/version reference.
- AC-03: Each result version contains one immutable 1–4 score for each of the seven areas with anchor, pinned evidence, rationale, confidence, model/configuration metadata, and timestamp.
- AC-04: No composite hire score, auto-advance, auto-reject, automatic invitation, or silent threshold decision is produced.
- AC-05: Designed-hole evidence and permitted response content may inform a score; résumé facts/discrepancies and older recording versions cannot.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| REV-01-TC-01 | Input isolation | Packet plus prohibited records exist | Inspect processing payload | Only permitted assessment evidence is present (AC-01) |
| REV-01-TC-02 | Latest active version | Three answer recordings exist; v3 active | Process answer | v3 alone is analysed and provenance identifies it (AC-02) |
| REV-01-TC-03 | Complete immutable result | Assessment submitted | Process/inspect result | Seven 1–4 evidence-backed immutable entries and metadata exist (AC-03) |
| REV-01-TC-04 | No automated decision | Result exists | Attempt automatic progression | No assessment AI action changes candidate state (AC-04) |
| REV-01-TC-05 | Evidence boundary | Résumé discrepancy and old recording exist | Inspect score evidence | Neither influences the seven scores (AC-05) |

# US-REV-02: Recover safely from AI processing failure or reprocessing

## User Story

As HR or operations staff, I want controlled recovery from an AI-processing problem, so that technical failures do not harm an applicant or erase evidence.

## Description

Failure preserves the packet and safe diagnostics. Authorised retry/reprocessing creates a new immutable result version. HR may comment and select the applicable result version but cannot edit an AI score.

## Key Capabilities

- `AI_PROCESSING_FAILED` state.
- Authorised, idempotent, audited retry.
- Immutable result history and selected version.

## Acceptance Criteria

- AC-01: Failure sets `AI_PROCESSING_FAILED`, preserves the packet, and records failed step, attempt, and safe error reference.
- AC-02: Authorised HR/operations can perform an idempotent audited retry; technical failure never becomes candidate failure.
- AC-03: HR reprocessing requires a reason and creates a new immutable result version while preserving earlier versions.
- AC-04: The decision record identifies the selected AI result version; no user can alter a result value or provenance.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| REV-02-TC-01 | Failure preservation | Simulated error | Process packet | Safe failure state exists; packet remains intact (AC-01) |
| REV-02-TC-02 | Authorised retry | Failed packet | Retry as authorised/unauthorised user | Only authorised idempotent retry succeeds; no rejection occurs (AC-02) |
| REV-02-TC-03 | Versioned reprocessing | Analysis ready | Supply reason and reprocess | New result exists; original remains (AC-03) |
| REV-02-TC-04 | Immutable selected version | Multiple versions exist | Select then attempt edit | Version reference is stored; mutation fails (AC-04) |

# US-REV-03: Review assessment and relevant candidate information and decide live-interview progression

## User Story

As an HR Reviewer, I want to review relevant candidate information and separate prior evaluations before deciding live-interview progression, so that the decision is human-owned and informed by the current workflow.

## Description

The earlier blanket restriction hiding candidate identity/personal information from HR before live-interview progression is superseded. Authorised HR may review candidate details, résumé, pre-screen information, work-assessment result, AI analysis/scores/evidence, recorded-response analysis, discrepancies, and separate comments needed for the decision. Access remains organisation-scoped and audited. HR cannot edit AI outputs.

## Key Capabilities

- Authorised combined review with clearly separated evidence sources.
- Separate HR comment/opinion; no AI score mutation.
- Human progression statuses and reason/audit.
- Applicant email with strengths/weaknesses and no exact scores.

## Acceptance Criteria

- AC-01: Only HR authorised for the owning organisation may view the relevant candidate details, résumé, pre-screen evidence, assessment artifacts/results, AI analysis/scores, discrepancies, and comments needed for progression review.
- AC-02: The UI labels résumé pre-screen AI, seven-area assessment AI, HR comments, and any human evidence separately and retains each source/version/provenance.
- AC-03: HR can comment, request reprocessing, compare/select versions, and accept/disregard recommendations but cannot edit, overwrite, or relabel AI output.
- AC-04: Only an HR human can set `LIVE_INTERVIEW_RECOMMENDED`, `LIVE_INTERVIEW_HOLD`, or `NOT_PROCEEDING`; the action records reviewer, evidence/version references, reason, and time.
- AC-05: Applicant notification for a progression/non-progression decision omits exact scores/internal comments and may provide an approved evidence-based strengths/weaknesses summary.
- AC-06: Cross-organisation, unrelated-candidate, or unauthorised-role access is denied and audited.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| REV-03-TC-01 | Relevant information review | Candidate evidence exists | Open as authorised HR | Permitted candidate/resume/prior evaluation evidence is available (AC-01) |
| REV-03-TC-02 | Provenance separation | Several evidence sources exist | Inspect review page | Every source/version is visibly separate (AC-02) |
| REV-03-TC-03 | Permitted actions/immutability | AI result exists | Comment/select/reprocess then edit score | Review actions work; score edit fails (AC-03) |
| REV-03-TC-04 | Human-only progression | AI recommendation exists | Attempt automatic then HR action | Only HR action sets status with full evidence record (AC-04) |
| REV-03-TC-05 | Applicant-safe summary | HR records decision | Inspect email | Strengths/weaknesses may appear; exact/internal data does not (AC-05) |
| REV-03-TC-06 | Organisation/role isolation | Other organisation/role exists | Open candidate review | Access is denied and audited (AC-06) |

# US-REV-04: Review voice-integrity flags without making a verdict

## User Story

As an HR Reviewer, I want to inspect a limited, explainable voice-integrity flag, so that I can investigate unusual delivery without treating voice as proof of misconduct or a scored trait.

## Description

The system compares the latest active answer only with that applicant's active natural and read-aloud baselines. It uses cautious allowed labels, pins exact recording versions, and cannot alter scores or progression.

## Key Capabilities

- Within-speaker, per-active-version comparison.
- Allowed/forbidden language guardrails.
- Purpose-limited voice retention/deletion.

## Acceptance Criteria

- AC-01: A flag is limited to approved cautious labels and pins the active answer, intro, and reading recording versions.
- AC-02: The interface never labels a candidate dishonest or gives a cheating probability; accent, fluency, pace, confidence, warmth, and personality are not scores.
- AC-03: A flag neither changes scores nor automatically rejects/advances; human dismiss/confirm requires a reason and shows confounders.
- AC-04: Voice use is opening-limited, retention is disclosed, no foundation-model training occurs by default, and deletion follows the approved schedule/request policy while preserving permitted audit facts.

## Test Cases

| Test ID | Scenario | Preconditions | Steps | Expected Result |
|---|---|---|---|---|
| REV-04-TC-01 | Allowed flag/version pins | Voice packet has signal | View flag | Only approved cautious label and exact active versions appear (AC-01) |
| REV-04-TC-02 | Forbidden language/traits | Review UI exists | Inspect fields/labels | No dishonesty/probability/proscribed trait score exists (AC-02) |
| REV-04-TC-03 | Non-decisive flag | Flag exists | Attempt automation; decide without/with reason | Automation fails; reason/confounders required (AC-03) |
| REV-04-TC-04 | Purpose/deletion | Retention trigger occurs | Execute/inspect workflow | Waveform deletion and permitted audit fact follow policy (AC-04) |
