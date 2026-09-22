# HireScope Requirements Traceability Matrix

**Updated:** 2026-09-21  
**Rule:** Each row maps requirement → source → domain/story → phase → acceptance criteria → test cases. “New requirement” means the product requirement supplied in this documentation task. `HireScope.md` and `hirescope-mockup-4.html` remain protected historical sources.

## New and changed product requirements

| Trace ID | Requirement | Source | Domain / Story | Phase | Acceptance Criteria | Test Cases |
|---|---|---|---|---:|---|---|
| RT-N01 | Multi-organisation ownership/isolation | New requirement | Identity/Opening — IAM-03, AUTH-03, OPN-01 | 1 | IAM-03 AC-06; AUTH-03 AC-07; OPN-01 AC-01/05 | IAM-03-TC-06; AUTH-03-TC-07; OPN-01-TC-01/05 |
| RT-N02 | Organisation-specific access | New requirement | Identity — IAM-03, AUTH-03 | 1 | IAM-03 AC-01/05/06; AUTH-03 AC-01/06/07 | IAM-03-TC-01/05/06; AUTH-03-TC-01/06/07 |
| RT-N03 | Opening-owned configuration; no cross-opening effect | New requirement | Opening — OPN-02, OPN-05 | 1 | OPN-02 AC-03/05; OPN-05 AC-01/06/07 | OPN-02-TC-03/05; OPN-05-TC-01/06/07 |
| RT-N04 | Configure supported modalities/sections/questions/timing | New requirement | Opening — OPN-05 | 1 | OPN-05 AC-02/04/05 | OPN-05-TC-02/04/05 |
| RT-N05 | Audio current; video planned only | New requirement clarification | Opening/Assessment — OPN-05, ASM-02 | 1–2 | OPN-05 AC-04/05; ASM-02 AC-03 | OPN-05-TC-04/05; ASM-02-TC-03 |
| RT-N06 | Configuration persistence/retrieval/enforcement | New requirement | Opening/Assessment — OPN-05, ASM-01 | 1–2 | OPN-05 AC-01/06/07; ASM-01 AC-01/02/04 | OPN-05-TC-01/06/07; ASM-01-TC-01/02/04 |
| RT-N07 | Collect candidate information/résumé before evaluation | New requirement | Candidate Intake — CAN-01 | 2 | CAN-01 AC-01–05 | CAN-01-TC-01–05 |
| RT-N08 | AI résumé fact extraction/criteria comparison | New requirement | Candidate Intake — CAN-02 | 2 | CAN-02 AC-01/02/05 | CAN-02-TC-01/02/05 |
| RT-N09 | Multiple résumés; advisory ranking/shortlist | New requirement | Candidate Intake — CAN-02 | 2 | CAN-02 AC-03/05 | CAN-02-TC-03/05 |
| RT-N10 | Résumé discrepancy detection with cautious evidence | New requirement | Candidate Intake — CAN-02 | 2 | CAN-02 AC-04/05 | CAN-02-TC-04/05 |
| RT-N11 | HR reviews complete pre-screen evidence | New requirement | Candidate Intake — CAN-03 | 2 | CAN-03 AC-01/02 | CAN-03-TC-01/02 |
| RT-N12 | HR authoritative pre-screen decision/audit | New requirement | Candidate Intake — CAN-03 | 2 | CAN-03 AC-02/03/05 | CAN-03-TC-02/03/05 |
| RT-N13 | Test link only after HR qualification | New requirement | Candidate Intake/Identity — CAN-03, IAM-02 | 1–2 | CAN-03 AC-04/05; IAM-02 AC-01 | CAN-03-TC-04/05; IAM-02-TC-01 |
| RT-N14 | Dedicated environment/time readiness page | New requirement | Assessment — ASM-02 | 2 | ASM-02 AC-01/05/06 | ASM-02-TC-01/05/06 |
| RT-N15 | Audio-specific readiness/blocked microphone | New requirement | Assessment — ASM-02 | 2 | ASM-02 AC-02/03/05 | ASM-02-TC-02/03/05 |
| RT-N16 | Network acknowledgement; no speed measurement | New requirement clarification | Assessment — ASM-02 | 2 | ASM-02 AC-01/04 | ASM-02-TC-01/04 |
| RT-N17 | Interruption provides Submit Test | New requirement | Assessment — ASM-05 | 2 | ASM-05 AC-01/02 | ASM-05-TC-01/02 |
| RT-N18 | Grievance reason/status/HR review | New requirement | Assessment — ASM-05 | 2 | ASM-05 AC-03/05/07 | ASM-05-TC-03/05/07 |
| RT-N19 | HR authorises/denies second attempt; applicant notice | New requirement | Assessment/Operations — ASM-05, OPS-02 | 2–3 | ASM-05 AC-05/06/07; OPS-02 AC-04 | ASM-05-TC-05/06/07; OPS-02-TC-04 |
| RT-N20 | Opening-configured re-record allowance/maximum | New requirement | Opening/Assessment — OPN-05, ASM-03 | 1–2 | OPN-05 AC-03; ASM-03 AC-03 | OPN-05-TC-03; ASM-03-TC-03 |
| RT-N21 | Preserve recording versions and active/latest | New requirement | Assessment — ASM-03 | 2 | ASM-03 AC-02/04 | ASM-03-TC-02/04 |
| RT-N22 | AI analyses latest submitted active recording | New requirement | Assessment/AI Review — ASM-03, REV-01 | 2–3 | ASM-03 AC-05; REV-01 AC-02/05 | ASM-03-TC-05; REV-01-TC-02/05 |
| RT-N23 | HR sees relevant candidate info before live progression | New requirement | AI/HR Review — REV-03 | 3 | REV-03 AC-01/02/06 | REV-03-TC-01/02/06 |
| RT-N24 | HR decides live progression; AI advisory | New requirement | AI/HR Review — REV-03 | 3 | REV-03 AC-03/04 | REV-03-TC-03/04 |
| RT-N25 | One or more interviewers | New requirement | Live Interview — INT-01 | 3 | INT-01 AC-01/02/03 | INT-01-TC-01/02/03 |
| RT-N26 | Individual interviewer scores/comments/provenance | New requirement | Live Interview — INT-02/03 | 3 | INT-02 AC-02/03; INT-03 AC-01–03 | INT-02-TC-02/03; INT-03-TC-01–03 |
| RT-N27 | AI and human evidence remain distinguishable | New requirement | AI Review/Live Interview — REV-03, INT-02/03 | 3 | REV-03 AC-02/03; INT-02 AC-04/05; INT-03 AC-03 | REV-03-TC-02/03; INT-02-TC-04/05; INT-03-TC-03 |
| RT-N28 | Configurable score aggregation without overwrite | New requirement | Opening/Live Interview — OPN-05, INT-02/03 | 1/3 | OPN-05 AC-04/05; INT-02 AC-05; INT-03 AC-03 | OPN-05-TC-04/05; INT-02-TC-05; INT-03-TC-03 |
| RT-N29 | HR final decision/audit | New + existing Flow/QA | Decision — DEC-01 | 3 | DEC-01 AC-01–03 | DEC-01-TC-01–03 |
| RT-N30 | Applicant email: strengths/weaknesses, no exact scores | New requirement | Candidate/Review/Decision/Ops — CAN-04, REV-03, DEC-01, OPS-02 | 2–3 | CAN-04 AC-02/03; REV-03 AC-05; DEC-01 AC-04; OPS-02 AC-04 | CAN-04-TC-02/03; REV-03-TC-05; DEC-01-TC-04; OPS-02-TC-04 |
| RT-N31 | Requisition return → revise → resubmit → re-review version loop | New requirement | Requisition — REQ-01/02 | 1 | REQ-01 AC-04; REQ-02 AC-05 | REQ-01-TC-04; REQ-02-TC-05 |
| RT-N32 | Email notification of requisition approval/rejection outcome to the Hiring Manager | New requirement | Requisition/Operations — REQ-02, OPS-02 | 1/3 | REQ-02 AC-04; OPS-02 AC-01 | REQ-02-TC-04; OPS-02-TC-01 |
| RT-N33 | Opening package return → revision → resubmit; email decision notification | New requirement | Opening — OPN-01/02 | 1 | OPN-01 AC-06/07; OPN-02 AC-06/07 | OPN-01-TC-06/07; OPN-02-TC-06/07 |
| RT-N34 | Opening-level configuration cannot bypass HR approval | New requirement | Opening — OPN-02, OPN-05 | 1 | OPN-02 AC-06; OPN-05 AC-08 | OPN-02-TC-06; OPN-05-TC-08 |

## Retained baseline requirements

| Trace ID | Requirement | Source | Domain / Story | Phase | Acceptance Criteria | Test Cases |
|---|---|---|---|---:|---|---|
| RT-B01 | Requisition creation/approval | Flow/QA; new org scope | REQ-01/02 | 1 | REQ-01 AC-01–03; REQ-02 AC-01–04 | REQ-01-TC-01–03; REQ-02-TC-01–04 |
| RT-B02 | Immutable opening/FairPack versioning | HireScope.md; Flow/QA | OPN-01/02 | 1 | OPN-01 AC-02–04; OPN-02 AC-01–04 | OPN-01-TC-02–04; OPN-02-TC-01–04 |
| RT-B03 | Secure bound/expiring invitation | Flow/QA | IAM-02, CAN-03 | 1–2 | IAM-02 AC-01–05; CAN-03 AC-04 | IAM-02-TC-01–05; CAN-03-TC-04 |
| RT-B04 | Typed accommodation | HireScope.md | ASM-04 | 2 | ASM-04 AC-01–04 | ASM-04-TC-01–04 |
| RT-B05 | Seven-area immutable AI assessment; no composite | HireScope.md; Flow/QA | REV-01 | 3 | REV-01 AC-01/03/04 | REV-01-TC-01/03/04 |
| RT-B06 | AI failure/versioned retry | Flow/QA | REV-02 | 3 | REV-02 AC-01–04 | REV-02-TC-01–04 |
| RT-B07 | Voice-integrity guardrails | HireScope.md | REV-04 | 3 | REV-04 AC-01–04 | REV-04-TC-01–04 |
| RT-B08 | Existing ten-area live rubric/1–5 | Flow/QA | INT-02 | 3 | INT-02 AC-01/02 | INT-02-TC-01/02 |
| RT-B09 | Position-filled accounting/read-only closure | Flow/QA | OPS-04 | 3 | OPS-04 AC-01–04 | OPS-04-TC-01–04 |
| RT-B10 | Comprehensive audit | HireScope.md; Flow/QA; new requirement | OPS-03 | 3 | OPS-03 AC-01–04 | OPS-03-TC-01–04 |

## Requirement ownership map

| Requirement | Existing Domain | Existing Story | Action |
|---|---|---|---|
| Multi-organisation platform | Identity / Opening | IAM-03, AUTH-03, OPN-01 | Updated |
| Opening configuration | Opening | OPN-01/02/04 | Added OPN-05; updated existing |
| Audio/video configuration | Opening / Assessment | OPN-05, ASM-02 | Updated; video decision-gated |
| Resume collection | Candidate Intake | CAN-01/CAN-03 (old) | Reworked CAN-01 |
| AI resume screening | Candidate Intake | CAN-01/03 (old boundary) | Reworked CAN-02 |
| Resume discrepancy analysis | Candidate Intake | None | Added within CAN-02 |
| HR pre-screening decision | Candidate Intake | CAN-02 (old routing) | Reworked CAN-03 |
| Test link generation | Identity / Candidate Intake | IAM-02, CAN-02 (old) | Updated CAN-03 gate |
| Environment readiness | Assessment | ASM-02 | Extended/reworked |
| Interruption handling | Assessment | ASM-01/05 | Reworked ASM-05 |
| Grievance | Assessment | ASM-05 | Extended |
| Second chance | Assessment | ASM-05 | Extended |
| Audio re-recording | Opening / Assessment | ASM-03 | Extended with OPN-05 |
| Recording versions | Assessment | ASM-03 | Extended |
| Live interview review | AI/HR Review | REV-03 | Reworked |
| Multiple interviewers | Live Interview | INT-01/03 | Extended |
| Individual interviewer scores | Live Interview | INT-02/03 | Extended |
| AI + human evaluation | AI/HR Review / Live Interview | REV-03, INT-02/03, DEC-01 | Extended |
| Requisition resubmission/version control | Requisition | REQ-01/02 | Extended |
| Requisition approval email notification | Requisition / Operations | REQ-02, OPS-02 | Extended |
| Opening package return/resubmit + decision notification | Opening | OPN-01/02 | Extended |
| Opening-level configuration approval gate | Opening | OPN-02, OPN-05 | Extended |

No new domain or phase file was required.

## Phase and dependency assignment

| Story | Prerequisite stories | Required data/entities | Required screen(s) | Conceptual API/service boundary | Phase rationale |
|---|---|---|---|---|---|
| IAM-03 / AUTH-03 updates | AUTH-01/02 identity/token validation | Organisation membership, role, ownership, assignment, access event | Role workspaces / access-denied state | Authentication + organisation/record authorisation | Phase 1 foundation for every later record |
| OPN-01/02 updates | REQ-01/02 | Organisation, requisition, opening, version, approval | Opening draft/change/version history | Opening ownership/version service | Phase 1 before candidate entry |
| OPN-04 update | OPN-01 | Criteria version, question, evidence rule, approval | Eligibility and Pre-screen configuration | Criteria validation/version service | Phase 1 input for Phase 2 résumé analysis |
| OPN-05 new | OPN-01/02/04 | Opening configuration, modality, section, question, timing, re-record policy, interviewer count, aggregation-policy reference | Create Job Opening configuration/preview | Configuration validation, persistence, version retrieval | Phase 1 because all applicant/interview flows consume it |
| CAN-01 rework | OPN-03/04; secure intake identity decision DEC-13 | Application, candidate fields, résumé, opening/version | Application details and résumé upload | Intake validation/protected résumé storage | Phase 2 after opening publication |
| CAN-02 rework | CAN-01, OPN-04 | Résumé AI result/version, extracted fact, criterion match, discrepancy, recommendation | Pre-screen processing/results | Résumé extraction/comparison AI + failure/retry | Phase 2 before HR qualification |
| CAN-03 rework | CAN-02, IAM-03 | HR review/comment/decision, selected result version, invitation | HR Pre-screen Review | Human decision + invitation issuance/notification | Phase 2; assessment access depends on it |
| ASM-01 update | CAN-03, IAM-02, OPN-05 | Pinned application/configuration/FairPack, drafts, packet | Assessment workspace | Configuration retrieval + assessment state service | Phase 2 after invitation |
| ASM-02 rework | ASM-01, OPN-05 | Readiness/consent/check/acknowledgement | Dedicated readiness page | Device-permission checks + readiness gate | Phase 2 immediately before assessment start |
| ASM-03 update | ASM-02, OPN-05 | Recording version/attempt/timestamp/active flag | Enabled audio sections and version controls | Media capture/version/selection service | Phase 2 assessment capture |
| ASM-05 update | ASM-01; OPS notification capability | Interruption event, grievance, HR decision, linked attempt | Interruption choices / grievance form / HR queue | Attempt state + grievance decision + notification | Phase 2 because it controls assessment attempts |
| REV-01 update | ASM-01/03 submission | Immutable assessment result/version and recording provenance | AI processing/result status | Assessment AI input isolation/version service | Phase 3 after submitted packet |
| REV-03 rework | CAN-03, REV-01/02 | Candidate/resume/pre-screen/assessment evidence, HR progression decision | HR progression review | Authorised evidence projection + human decision | Phase 3 after analysis |
| INT-01–03 updates | REV-03, OPN-05 | Interview, participant, individual category result, submission, aggregation-policy reference | Assignment/brief/rubric/results | Assignment authorisation + participant result service | Phase 3 after HR recommendation |
| DEC-01 update | INT-03 completion | Final decision, evidence references, communication summary | Final review/decision | Human decision + safe email rendering | Phase 3 terminal assessment workflow |
| OPS-01–03 updates | All contributing domains | Organisation-scoped counts, notifications, audit events | Summary/queues/audit lookup | Reporting, notification, append-only audit | Phase 3 cross-domain operation after source events exist |
