# HireScope

**Work first. Paper second. A human shortlists.**

Compiled product spec, fairness model, voice interview, and user stories.

Related files:

- Interactive mockup: `hirescope-mockup.html`
- Screenshots: `hirescope-screens/`
- Stories-only Word file: `HireScope-User-Stories.docx`

---

## 1. How scoring works

HireScope is a hiring assessment product. The mechanics:

- **Horizontal review** — score one criterion across all applicants, not one person at a time.
- **Shared, versioned rubrics** — the same anchors for everyone; later edits do not silently rewrite earlier scores.
- **Evidence on the work** — every score points at a specific part of the artifact or answer.
- **Human owns the score** — the model drafts; the reviewer confirms.

What it is *not*: a résumé parser, a personality test, or an auto-reject engine.

---

## 2. Product objective

Give every applicant the same job-like pack.

Score **seven work behaviors** — not school, years, or brand.

Hide the résumé until a human locks those scores.

Then shortlist from evidence.

- **Grit** is follow-through on an incomplete task, not a quiz.
- **Distance traveled** (gaps, bootcamp, military, caregiving, no brand) is context after lock, never points.
- **AI** drafts requirements, tests, and scores with pinned evidence.
- **A human** edits the pack, confirms or overrides scores, and shortlists.

---

## 3. Anti-goals — do not build

- Grit quiz or “passion consistency” scale
- Adversity points / scoring distance traveled
- Culture-fit score
- Résumé-first ranking or keyword sift as a gate
- Auto-advance or auto-reject on an AI score
- School, years, or employer brand as scored criteria
- Accent, confidence, warmth, or “executive presence” from voice
- “Cheated with an LLM 87%” meter
- Face / gaze / emotion scoring
- Training foundation models on applicant audio by default

---

## 4. Personas

| Persona | Owns |
|---|---|
| Hiring Manager | Creates the opening, edits the AI-drafted pack, publishes a locked version |
| Applicant | Completes the same pack: work sample, voice baselines, spoken interview |
| HR Reviewer | Confirms or overrides scores, reviews voice flags, shortlists |

**System actor:** AI drafts and scores. AI never decides.

---

## 5. Expected flow

1. Hiring manager creates the job opening from title, team, and 90-day ownership.
2. AI drafts job requirements (the seven scored areas) and the tests (work sample + structured interview + reading passage).
3. Hiring manager edits anything. Pedigree gates stay off unless replaced with something job-related.
4. Manager publishes. That pack version is now frozen. Every applicant gets the same instrument.
5. Applicant logs in and completes:
   - Written work sample (one necessary figure missing on purpose)
   - Spoken intro (natural-voice baseline)
   - Reading sample (read-aloud baseline)
   - Four structured questions answered out loud
   - Optional context (gap / military / bootcamp / caregiving) — sealed until scores lock
6. AI grades the seven areas with pinned evidence and may raise a voice integrity *flag*.
7. HR reviews horizontally (one area across people), résumé still hidden.
8. HR locks scores. Résumé and distance-traveled context appear.
9. HR shortlists for the next live interview. AI may suggest. AI may not invite.

---

## 6. What to analyze to be fair

Résumés encode school, brand-name employers, consecutive years, and tidy titles. Those are among the weakest predictors of later performance. Years of experience sit around 0.07–0.18. Years of education sit around 0.10. Structured interviews, job-knowledge tests, and work samples are several times stronger.

That is why companies keep discovering people the paper would have killed: the military spouse with a gap who can run an ops board, the bootcamp engineer who ships, the career-switcher who already did the hard part under a different title.

“Grit” is the right instinct and the wrong label. On tests it overlaps conscientiousness. Perseverance of effort is the useful piece. Do not add a grit quiz. Measure follow-through on work.

### Seven scored areas

| # | Area | What it means | How it shows up |
|---|---|---|---|
| 1 | Job-task performance | Can they do a slice of this job? | Work sample artifact |
| 2 | Thinking on this material | Messy job data, not a brainteaser | Work sample + assumptions |
| 3 | Follow-through | Finish the ugly middle; recover from a wrong turn | Incomplete-data behavior + month-four answer |
| 4 | Learning velocity | Notice a missing input and keep moving | Designed hole in the export |
| 5 | Judgment under constraint | What they cut; what they will not ship | Spec + “would not ship” answer |
| 6 | Working with people when messy | Not culture fit | Same conflict scenario for everyone |
| 7 | Integrity and ownership | Names limits; no fake certainty | What they admit they don’t know |

### Context only — never a score

**Distance traveled** — gaps, bootcamp, military, caregiving, no brand.

Shown after scores lock. Explains the paper. Does not add or subtract points. Do not award adversity points.

### Résumé rule

The résumé is hidden from AI grading and from HR until all seven scores on that packet are confirmed or overridden. After lock, paper is context for the next conversation — not the reason they passed.

Post-lock note when a strong packet would have died in an ATS sift:

> ATS-style résumé screen would likely have rejected this packet. The work sample would not.

---

## 7. The assessment pack

### Work sample

A 90-minute slice of week-1 work. Example for Senior PM:

> Activation fell 18% after checkout redesign. Diagnose, pick one bet, write v1.

**Designed hole:** Q2 returning-trial rate is missing from the export.

- Stall with no mention of the hole → low follow-through / learning
- Name the hole and still ship a usable spec → high follow-through / learning
- Invent the missing number without saying so → integrity flag

### Structured interview (spoken by default)

Same four questions for every applicant on the pack version:

| Q | Prompt | Maps to |
|---|---|---|
| 1 | A goal that failed for months — what did you do in month four? | Follow-through |
| 2 | A tool you had to teach yourself. First week vs week four. | Learning |
| 3 | Sales VP wants the redesign rolled back Friday. Next 48 hours? | People + Judgment |
| 4 | What would you not ship this quarter, and why? | Judgment + Integrity |

Typed path remains a human-granted accommodation. It is not a self-serve “hide from integrity flags” toggle.

---

## 8. Voice interview

### Sequence

0. Consent + mic check. No recording without explicit agree.
1. Spoken intro (~45s) — **natural speaking baseline**. Prompt is work-style (“kind of problems you like to work on”), not school or employers, so pedigree does not leak before lock.
2. Reading sample (~30s) — **read-aloud baseline**. Same passage for everyone. Not a literacy or accent test.
3. The four structured questions answered out loud, one at a time. Optional think time, then record.

### Why two baselines

Read-aloud and spontaneous speech differ in pace, repairs, pitch range, and filled pauses. An answer read off a generated script often shifts toward **that person’s reading voice**. That is a within-speaker comparison, stronger than a generic “sounds like ChatGPT” model.

It is a useful signal. It is not proof. Notes, rehearsal, anxiety, stutter, and additional-language speakers can look the same.

### What AI may say (integrity flag)

Per answer, compare delivery to *this person’s* intro vs *this person’s* reading sample.

**Allowed labels only:**

- Closer to their reading sample than their intro
- Long pause, then paragraph fluency
- Possible different speaker / TTS vs intro

**Forbidden labels:**

- Cheated with ChatGPT
- LLM probability 87%
- Dishonest

A flag pins three clips: intro, reading sample, this answer. It does not change the seven scores. It cannot auto-reject. Dismiss or confirm requires a reason. Confounders are listed on the card.

### What voice is not

Voice is a **channel** for the same questions. The rubric does not become “sounds confident.”

“Spoken communication” is an optional extra area only if the hiring manager turns it on because the job needs it (support, sales, teaching). Off by default.

Voice files are purpose-limited to this opening, retained on a stated schedule, and not used to train foundation models unless separately consented.

---

## 9. Scoring model

- Each of the seven areas: **1–4** with a pinned quote or clip.
- No overall hire %.
- No composite that hides a weak area.
- AI drafts. Human confirms or overrides (reason required on override).
- Horizontal review: one criterion across applicants before a whole-person view.
- Pack version is stamped on every submission. In-flight candidates stay on the version they started.

### Follow-through anchors (example)

| Score | Label | Meaning |
|---|---|---|
| 1 | Stalled | Stopped when data or the first bet failed |
| 2 | Performed effort | Busy, same bet, no new tactic |
| 3 | Kept moving | Finished a usable v1 despite a hole |
| 4 | Recovered + cut | Named the hole, changed the bet, kill criterion |

---

## 10. Example packet

**Opening:** Senior Product Manager · Growth · REQ-1842  
**90-day work:** Own activation from signup to first value. Find why trial-to-paid dropped after checkout redesign and ship one bet.

**C-07** (the person a résumé sift would have dropped)

- Scores: Task 3 · Think 4 · Follow-through 4 · Learn 4 · Judge 4 · People 3 · Integrity 3
- Voice: delivery closer to intro than to reading. No integrity flag.
- After lock: bootcamp 2022, 2 years B2B PM at a 40-person company, 14-month caregiving gap, no FAANG, no CS degree.
- Context chip: distance traveled — do not downscore.

**C-11**

- Low follow-through (stalled on the missing number).
- Voice flag: Q1 closer to their reading sample. HR listens. Flag does not auto-fail.

**C-14**

- Strong packet, linear pedigree. Pedigree is not a bonus on top of the scores.

---

## 11. User stories

Every story has an explicit **objective**. AI is a system actor, not a persona.

### Epic 1 — Create the opening

#### US-01 Create a job from 90-day work

As a hiring manager, I want to create an opening from title, team, and what they will own in 90 days, so that the assessment is built from the job.

**Objective:** Start from the work, not from a résumé template.

**Acceptance criteria**

- Required: title, team, 90-day ownership. Optional: location, seniority.
- Saved as Draft REQ. No applicant access until publish.
- “Do not use as a gate” stores items such as FAANG, degree, years, no-gaps as excluded filters, not scored areas.

#### US-02 Preview what AI will generate

As a hiring manager, I want to see the 7 areas, work sample, interview, reading passage, and context-only rule before generate runs.

**Objective:** The manager stays owner of the pack.

**Acceptance criteria**

- Preview is explicit. Generate does not run until confirm.
- Copy states: no grit slider, no pedigree gate.

---

### Epic 2 — AI drafts the fair pack

#### US-03 Draft the seven scored areas

As a hiring manager, I want AI to propose seven job-related areas from my brief.

**Objective:** Replace pedigree gates with the same seven areas for every candidate.

**Acceptance criteria**

- Always emits: Job-task, Thinking, Follow-through, Learning, Judgment, Working with people, Integrity.
- Each has a “why this is this job” note.
- School / years / brand are unchecked.
- Distance traveled is labeled context only.

#### US-04 Draft an incomplete work sample + mapped interview

As a hiring manager, I want a 90-minute week-1 task missing one necessary input, plus four structured questions.

**Objective:** Grit is observed on the task. Soft-looking skills use the same questions, not vibe.

**Acceptance criteria**

- One figure is omitted on purpose (follow-through / learning probe).
- Default Qs: month-four failure; tool you taught yourself; 48-hour stakeholder conflict; what you would not ship.
- No “are you gritty?” prompt.
- Same pack for every applicant on this version.

#### US-05 Mark distance traveled as context only

As a hiring manager, I want gaps, bootcamp, military, caregiving, and no-brand paths to be un-scorable.

**Objective:** Never create an adversity-points contest.

**Acceptance criteria**

- Cannot be checked as a scored area.
- Chip: “Do not award adversity points. Reveal after lock.”

---

### Epic 3 — Edit and publish

#### US-06 Edit or drop any drafted item

As a hiring manager, I want to rewrite, uncheck, or replace anything AI drafted.

**Objective:** Nothing reaches a candidate unless a human believes it is this job.

**Acceptance criteria**

- Edits persist. Cannot publish with zero scored areas.
- Pedigree stays dropped unless re-added as job-related criteria.

#### US-07 Publish and lock pack version

As a hiring manager, I want every applicant on this REQ to get the identical sample, questions, passage, and anchors.

**Objective:** Fairness is “same test,” not good intent.

**Acceptance criteria**

- Publish creates v1 + invite link.
- Later edits become v2.
- In-flight applicants stay on the version they started.

---

### Epic 4 — Applicant completes the same pack

#### US-08 Start the same assessment

As an applicant, I want to open one assessment every other applicant also gets.

**Objective:** Equal instrument. Résumé not required to begin.

**Acceptance criteria**

- Work sample + voice steps + timer.
- Banner: “Same pack as every other candidate.”
- No school / employer / years required to start.

#### US-09 Submit the work sample

As an applicant, I want time to diagnose, pick one bet, write a usable v1, and say what I will not ship — including when a number is missing.

**Objective:** Produce the written artifact that will be scored.

**Acceptance criteria**

- Draft save works. Submit locks the artifact.
- Bound to pack version. Identity hidden from reviewers until lock.

#### US-10 Optional context stays sealed until lock

As an applicant, I want to optionally explain a gap / military / bootcamp / caregiving path that AI and HR cannot see yet.

**Objective:** A non-linear path can be explained without biasing the first pass.

**Acceptance criteria**

- Field optional. Hidden from grading and review until lock.
- Never a scored criterion.

#### US-11 Request a different way to show the same skills

As an applicant, I want to request an alternate format (extra time, typed instead of voice) without that request entering the grade.

**Objective:** Access needs go through a human path, not the model.

**Acceptance criteria**

- Request routed to HR / hiring manager.
- Alternate format still maps to the same seven areas and pack version.

---

### Epic 5 — AI grades with evidence

#### US-12 Score each of the seven areas with pinned evidence

As an HR reviewer, I want a 1–4 and a quoted snippet per area.

**Objective:** Review evidence, not a hire-meter or a grit scale.

**Acceptance criteria**

- One score + pin per area. No overall hire %.
- Model cannot use school, brand, years, gaps, or sealed context.
- Follow-through / learning must address the omitted input (stall vs recover).
- Inventing data without saying so flags Integrity.
- Auto-advance and auto-reject are off.

#### US-13 Flag low-confidence scores

As an HR reviewer, I want unsure items marked.

**Objective:** Human time goes to the risky drafts first.

**Acceptance criteria**

- Flag is visible and is not itself a fail.

---

### Epic 6 — Human review, résumé hidden

#### US-14 Inbox of completed packs — seven areas only

As an HR reviewer, I want a queue with the seven scores and no pedigree columns.

**Objective:** Compare people on the work.

**Acceptance criteria**

- Columns are the seven areas.
- No school / employer / years.
- No composite “AI total” as the decision.

#### US-15 Review one area across applicants

As an HR reviewer, I want to grade Follow-through (or any one area) across packets before I look at a whole person.

**Objective:** Same standard on the same behavior (one criterion across people).

**Acceptance criteria**

- Criterion tabs, visible 1–4 anchors, anonymous packet IDs.

#### US-16 Confirm or override each AI score

As an HR reviewer, I want to keep or change each 1–4, with a reason on override.

**Objective:** No candidate moves on a model score alone.

**Acceptance criteria**

- Override requires a new score + reason.
- Original AI score stays in the log.

#### US-17 Keep the résumé hidden until all seven scores lock

As an HR reviewer, I must not see school, brand, years, or sealed context until I have confirmed all seven areas.

**Objective:** Pedigree cannot leak into the grade.

**Acceptance criteria**

- Unlock is disabled until seven scores are set.
- Early open is blocked and logged.
- No LinkedIn / filename side channel on the review UI.
- Intro audio that names a school or employer stays sealed until lock.

---

### Epic 7 — Reveal and shortlist

#### US-18 Lock scores, then reveal résumé + distance traveled

As an HR reviewer, I want lock to reveal paper and optional context.

**Objective:** Context after evidence. A gap may be interpreted; it may not change locked scores.

**Acceptance criteria**

- Chip reads “Do not downscore. Do not add adversity points.”
- Unlocking scores later is audited.

#### US-19 Show when the paper would have killed a strong packet

As an HR reviewer, I want a post-lock note when 3s/4s sit next to no degree, a gap, or a non-brand last title.

**Objective:** Stop reviewers from re-applying the old filter after lock.

**Acceptance criteria**

- Shown only after lock.
- Copy: “ATS-style résumé screen would likely have rejected this packet. The work sample would not.”
- No points added or subtracted.

#### US-20 Shortlist for the next-round interview

As an HR reviewer, I want to choose who advances after scores are locked.

**Objective:** Shortlist is a human click. AI may suggest; AI may not invite.

**Acceptance criteria**

- Checkboxes are human.
- A packet cannot enter the next round without an advance action.
- Can write back to the ATS.

#### US-21 Reject or hold on the same rubric

As an HR reviewer, I want a no to be about the seven areas.

**Objective:** A rejection is explainable from the same instrument used to advance someone.

**Acceptance criteria**

- Requires a low criterion or a written reason.
- Reason required if rejecting a mostly-3+ packet.
- Pack version stored.

---

### Epic 8 — Fairness, versioning, audit

#### US-22 Version the pack when the job or rubric changes

As a hiring manager, I want a published edit to become v2.

**Objective:** People already in flight stay on the instrument they took.

**Acceptance criteria**

- v1 stays on v1 anchors.
- Queues do not mix versions.

#### US-23 Audit the decision and refuse auto-decisions

As an HR reviewer, I want every AI score, override, lock, reveal, shortlist, reject, and voice-flag action logged — and I want the system to refuse to move anyone without a human action.

**Objective:** We can explain a decision later. The model is never the hiring decision.

**Acceptance criteria**

- Log is append-only.
- Reason required on override and on reject-after-high-score.
- No auto-advance / auto-reject setting.
- Publishing a scored gate on school, years, brand, culture fit, or a grit scale does not save.

#### US-24 See where people drop out

As an HR reviewer, I want counts by stage: invited → submitted → each of the seven areas at 3+ → shortlisted.

**Objective:** Find whether a scored area — not a hidden résumé screen — is where people disappear.

**Acceptance criteria**

- Stage counts for the REQ.
- No demographic inference in v1 unless separately specified and lawful.
- Dropout before lock cannot be attributed to “had a gap.”

---

### Epic 9 — Spoken interview + reading baseline

#### US-25 Consent and mic check before any recording

As an applicant, I want to know why my voice is recorded and to test my mic before I start.

**Objective:** Lawful recording and a usable file. No hidden voiceprint.

**Acceptance criteria**

- Consent names: audio + transcript; within-speaker compare of intro / reading / answers; retention window; not used to score accent, confidence, or personality; not used to train foundation models unless separately opted in.
- Decline → typed path or stop. Cannot continue voice without consent.
- Mic check plays back a few seconds. Fail the check = retry, not a flag.

#### US-26 Spoken introduction (natural baseline)

As an applicant, I want about 45 seconds to introduce how I work.

**Objective:** Enroll this person’s spontaneous voice. Not a résumé recitation. Not a scored “presence” trait.

**Acceptance criteria**

- Same prompt for every applicant on this pack version.
- Prompt is work-style, not school or employers.
- One take, optional single re-record. Stored as baseline A (natural).
- Voice quality is never a 1–4.

#### US-27 Reading sample (read-aloud baseline)

As an applicant, I want to read a short provided passage aloud.

**Objective:** Enroll this person’s reading voice. Not a literacy, fluency, or accent test.

**Acceptance criteria**

- Same passage for every applicant on this pack version (~30 seconds, neutral, not the interview answers).
- UI discloses that it is a reading sample, not scored.
- One take, optional single re-record. Stored as baseline B (read).
- Cannot be a scored competency.

#### US-28 Answer the structured questions out loud

As an applicant, I want to answer the four structured questions verbally, one at a time.

**Objective:** Same four questions as the written interview, spoken like a real interview.

**Acceptance criteria**

- Same four Qs, same order, one question visible at a time.
- Optional think time, then record. One retry per question.
- Submit locks the clip. Transcript + audio stored on the pack version.

#### US-29 Transcribe and score content from speech

As an HR reviewer, I want a transcript plus the audio clip, scored on the same seven areas.

**Objective:** Voice is a channel. The rubric does not become “sounds confident.”

**Acceptance criteria**

- Transcript shown with playback. Reviewer can correct ASR errors.
- Seven areas scored from what they said.
- No column for warmth, fluency, pace, or accent.
- “Spoken communication” is optional and off by default; only if the job needs it.

#### US-30 Flag possible scripted delivery — never auto-fail

As an HR reviewer, I want a per-answer flag when delivery is closer to this applicant’s reading sample than to their intro.

**Objective:** Detect likely read-aloud of a script (including an LLM script) as a review flag, not a cheat verdict.

**Acceptance criteria**

- Compare each answer to this person’s baseline A vs baseline B.
- Allowed labels only: “Closer to their reading sample,” “Possible different speaker / TTS vs intro,” “Long pause then paragraph fluency.”
- Forbidden labels: “Cheated with ChatGPT,” “LLM probability 87%,” “Dishonest.”
- Flag pins answer + intro + reading clips.
- Does not change the seven scores. Cannot auto-reject.
- Dismiss / confirm requires a reason.
- Card lists confounders: notes, rehearsal, anxiety, stutter, additional language.

#### US-31 Typed interview remains an accommodation

As an applicant, I want a human-granted typed path if I cannot record voice.

**Objective:** Voice is default, not a gate against people who cannot speak on a mic.

**Acceptance criteria**

- Request goes to HR, not the model.
- Typed answers still score the seven areas. Voice integrity flag is N/A.

#### US-32 Purpose-limit and delete voice files

As an applicant, I want voice used only for this opening’s review and then deleted on the stated schedule.

**Objective:** A hiring pack must not become a biometric database.

**Acceptance criteria**

- Retention shown at consent.
- No default training on candidate audio.
- Delete on request / on schedule.
- Audit may keep “flag existed / dismissed” without keeping the waveform past retention.

---

## 12. Build order

| Intent | Stories |
|---|---|
| Pack exists and is human-owned | US-01, US-02, US-03, US-06, US-07 |
| Assessment can be taken | US-04, US-08, US-09, US-10, US-11 |
| Scoring is explainable | US-05, US-12–US-17 |
| Decision is human | US-18–US-21, US-23 |
| Spoken interview | US-25–US-32 |
| Defensible over time | US-22, US-24 |

If the first slice has to be small, ship these five first: **US-04** (incomplete task), **US-17** (résumé hidden), **US-19** (paper vs work), **US-20** (human shortlist), **US-23** (no auto-decision + audit). Voice can follow as US-25–US-30.

---

## 13. Product rules that apply to every story

1. Same pack version for every applicant in the same review queue.
2. The seven areas are the only scored columns (unless the manager explicitly adds a job-related eighth, such as spoken communication).
3. Follow-through is scored from the work and the month-four answer — never from a personality quiz.
4. Distance traveled is context after lock — never points.
5. Voice integrity is a listen-flag — never a cheat verdict and never an auto-reject.
6. AI drafts and pins evidence. A human confirms, overrides, locks, and shortlists.

---

## 14. Legal and fairness notes

- Structured, job-related, consistently applied assessments are more defensible than unstructured interviews and résumé pedigree filters.
- Voice can be biometric in some jurisdictions (e.g. BIPA-class notice). Consent must name recording, transcript, within-speaker style compare, and retention.
- Automated speech recognition has documented error gaps by accent and dialect. Transcripts must be reviewable and correctable. Accent is not a score.
- A voice flag that auto-rejects is an automated employment decision. HireScope does not allow that setting.
- All-party consent may apply to recording depending on state. Do not record before agree.

This document is a product spec, not legal advice.

---

## 15. Artifacts

| File | What it is |
|---|---|
| `HireScope.md` | This compiled spec |
| `hirescope-mockup.html` | Clickable prototype (Hiring mgr / Applicant / HR) |
| `hirescope-screens/` | Screen captures of pack, review, voice |
| `hirescope-user-stories.md` | Earlier stories-only draft (IDs superseded by this file) |
| `HireScope-User-Stories.docx` | Word export of the earlier stories draft |
