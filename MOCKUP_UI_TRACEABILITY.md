# HireScope Mockup UI Traceability

**Prototype reviewed:** `hirescope-mockup-4.html` (protected; not edited)  
**Interpretation rule:** The prototype demonstrates the original work-assessment concept. The 2026-09-21 product requirement and current phased stories govern conflicts.

| Mockup surface | Current story ownership | Status | Required implication |
|---|---|---|---|
| Create opening | REQ-01/02, OPN-01/04/05 | Partial/conflicts | Add organisation ownership, requisition approval, per-opening criteria/configuration, and video-planned state. |
| FairPack draft | OPN-01/02/05 | Partial | Retain seven-area instrument; sections/questions/timing/re-record/interviewer settings come from opening configuration. |
| Publish/invite | OPN-02/03, CAN-03, IAM-02 | Conflicts | Public link is intake only. Assessment link follows résumé AI analysis and explicit HR qualification. |
| Candidate intake/résumé pre-screen | CAN-01–03 | Missing | Add candidate/resume collection, AI extraction/matching/ranking/discrepancies, HR review/decision. |
| Applicant readiness | ASM-02 | Missing | Add general/time/network acknowledgement and audio checks before start; no speed/video checks. |
| Work sample | ASM-01 | Partial | Load the application-pinned opening configuration/version. |
| Voice intro/reading/questions | ASM-03 | Conflicts | Replace fixed single retry with opening-configured maximum; show version history and latest active selection. |
| Interruption/grievance | ASM-05 | Missing | Add Submit Test and Submit Grievance plus HR second-attempt decision/notification. |
| Submitted screen | CAN-04, ASM-03 | Mostly aligned | Keep no exact scores; later email may provide approved strengths/weaknesses. |
| AI grades/review | REV-01–03 | Partial/conflicts | AI result immutable; HR cannot override score. Authorised HR may review relevant candidate/resume/prior evidence before progression. |
| Voice integrity | REV-04 | Partial | Pin active recording versions and retain cautious/non-decisional labels. |
| Shortlist | REV-03 | Partial | HR-only live-progression decision with reason/version references. |
| Multi-interviewer live stage | INT-01–03 | Missing | Add configured participant assignment, individual ten-area results, completion, provenance, and approved aggregation display. |
| Final HR outcome/email | DEC-01, OPS-02 | Missing | Add human final decision and strengths/weaknesses email without exact scores. |
| Organisation isolation/audit | IAM-03, AUTH-03, OPS-03 | Missing | Enforce organisation/role/record scope server-side and audit denied access. |

## Prototype-only or superseded controls

- **Override to 3** is superseded: HR may comment, request reprocessing, select an immutable version, and decide progression, but cannot edit AI scores.
- **Publish pack & open candidate preview** does not issue a real assessment credential before HR pre-screen qualification.
- Fixed **one retry** copy is superseded by the opening's re-record setting and maximum.
- The anonymous/blind HR review presentation no longer prevents authorised HR from accessing relevant candidate information before live-interview progression.
- **Write back to Greenhouse** remains prototype-only; no external ATS contract is approved.
- Any apparent video capability is not current scope; video is planned only.
