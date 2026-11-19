# Story Generation Plan: Margins

**Role**: Product owner
**Inputs**: `aidlc-docs/inception/requirements/requirements.md` (approved), `Vision.md`, `user-stories-assessment.md`
**Outputs**: `aidlc-docs/inception/user-stories/personas.md`, `aidlc-docs/inception/user-stories/stories.md`

---

## Part A: Questions

Fill in each `[Answer]:` tag with a letter. If no option fits, choose the last option (Other) and describe what you want. All questions must be answered before I generate the stories. Your answers are kept in this file as part of the audit trail.

### Breakdown approaches (background for Question 1)

| Approach | How stories are grouped | Strength | Weakness for Margins |
|---|---|---|---|
| **User Journey-Based** | By end-to-end flows (join a course, then upload, then study) | Shows how features connect; good for the demo | Shared features (retrieval, weights) are scattered across journeys |
| **Feature-Based** | By FR area (Accounts, Courses, Notes, Guide, Chat...) | Maps 1:1 to requirements and probably to code modules | Hides the cross-feature user experience |
| **Persona-Based** | By Student, Instructor, TA | Makes permissions obvious | Many stories are shared by all personas |
| **Domain-Based** | By business domain (Course structure, Content, Trust, AI study tools) | Clean boundaries for units of work | A bit abstract for a demo audience |
| **Epic-Based** | Epics with child stories | Easy to scope and to cut if time runs short | Needs a second organizing principle for the epics themselves |

## Question 1
How should the stories be organized?

A) Epic-based, with one epic per feature area (FR-1 to FR-10), and stories inside each epic. Each story is tagged with its persona(s).

B) User-journey-based: a handful of end-to-end journeys (for example "Instructor sets up a course", "Student studies for Prelim 1"), with stories as journey steps

C) Hybrid: epics per feature area (as A), plus a short "Key journeys" section at the top that links the stories each demo journey uses

D) Persona-based: one section per persona

E) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
Which personas should be written up?

A) Three: Student (contributor and studier combined), Instructor (professor), TA

B) Four: Student contributor, Student studier, Instructor (professor), TA (matches Vision.md's split of students into two personas)

C) As B, plus a "Team admin" persona for the developer who seeds instructor accounts and runs the evaluation script

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 3
How detailed should each persona be?

A) Short: role, goals, frustrations, permissions in Margins (about half a page each)

B) Richer: as A, plus a named example person, a typical week, technical comfort, and quotes (about one page each)

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
How large should stories be?

A) Fine-grained: one story per user action (for example "upvote a note" and "remove my upvote" are separate). About 50-70 stories.

B) Medium: one story per user goal, with related actions as acceptance criteria (for example "Rate a note's helpfulness" covers upvoting and removing it). About 30-40 stories.

C) Coarse: one story per feature area, with detailed acceptance criteria. About 10-15 stories.

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 5
What format should acceptance criteria use?

A) Given / When / Then scenarios

B) Bulleted checklist of observable outcomes

C) Given / When / Then for main and edge scenarios, plus a short checklist for simple rules (limits, labels, permissions)

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 6
Should stories be prioritized?

A) Yes, MoSCoW (Must / Should / Could / Won't for MVP), so the team can cut scope if needed

B) Yes, a simple ordering by demo importance: P1 (core loop), P2 (important), P3 (nice to have)

C) No; everything in requirements.md is in scope, so stories are not prioritized

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
How should LLM-specific and failure behavior be covered?

A) As acceptance criteria inside the relevant feature stories (for example "Given the Gemini API times out, then I see a retryable error")

B) As separate stories in a "Trust and reliability" epic (grounding, refusal to answer without sources, error handling, prompt-injection resistance)

C) Both: main failure criteria inside feature stories, plus a small cross-cutting epic for grounding and safety behavior that applies to both agents

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8
Should the stories also cover team- and developer-facing work that is not used through the web UI?

A) No; only end-user stories. Seeding instructors and the evaluation script stay as requirements only.

B) Yes, include a few "Team admin" stories: seed instructor accounts (FR-1.3), run the evaluation script on the planted test set (NFR-5), start the app locally with a `.env` (NFR-1)

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
For the demo, what is the single most important end-to-end journey? Its stories will be marked so the team builds and tests that path first.

A) Instructor creates a course from a public website, then students join and upload notes, then a student generates an exam guide with citations, conflicts and gaps

B) Student joins an existing (seeded) course, uploads notes, and uses cited chat to study

C) Instructor reviews coverage, resolves a conflict, and the regenerated guide changes as a result

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Answer Summary (validated, no contradictions or ambiguities found)

| Q | Answer | Effect on generation |
|---|---|---|
| 1 | A | One epic per feature area, E1 to E10 (FR-1 to FR-10); each story tagged with its persona(s) |
| 2 | B | Four personas: Student contributor, Student studier, Instructor (professor), TA |
| 3 | A | Short personas: role, goals, frustrations, permissions |
| 4 | B | Medium stories, one per user goal: about 30-40 stories |
| 5 | C | Given/When/Then for main and edge scenarios, plus a checklist for simple rules |
| 6 | A | MoSCoW priority on every story |
| 7 | A | LLM failure and grounding behavior written as acceptance criteria inside the feature stories; no separate epic |
| 8 | A | End-user stories only; no Team admin stories (FR-1.3 seeding and the NFR-5 evaluation script stay as requirements) |
| 9 | B | Key journey: a student joins an existing course (created by a seeded instructor), uploads notes, and studies with cited chat. Stories on this path are marked **[Key journey]** and are Must. |

---

## Part B: Generation Checklist

I will run these steps after the answers are validated and you approve this plan. Each box is marked `[x]` as soon as its step is done.

### Step 1: Prepare
- [x] 1.1 Re-read requirements.md, Vision.md and the answers above
- [x] 1.2 Create the `aidlc-docs/inception/user-stories/` directory

### Step 2: Personas (`personas.md`)
- [x] 2.1 Write each persona chosen in Q2, at the detail level chosen in Q3
- [x] 2.2 For each persona, list its permissions, linked to requirements.md section 2
- [x] 2.3 Add a persona-to-epic map, filled in after Step 3

### Step 3: Stories (`stories.md`)
- [x] 3.1 Write the epic structure E1 to E10 (Q1: A), with the story ID scheme `US-<epic>.<n>`
- [x] 3.2 Accounts and authentication stories (FR-1)
- [x] 3.3 Course and membership stories, including course requests (FR-2)
- [x] 3.4 Course-website structure stories, including exams and the no-schedule case (FR-3)
- [x] 3.5 Notes and professor-material upload stories, including tagging, anonymity, and delete/replace (FR-4)
- [x] 3.6 Upvote, "matches slides" and endorsement stories (FR-5)
- [x] 3.7 Ranking-transparency story for the "How sources are ranked" page (FR-6)
- [x] 3.8 Study-guide stories, including scope, citations, gaps, versions and progress (FR-7)
- [x] 3.9 Conflict display and resolution stories (FR-8)
- [x] 3.10 Instructor coverage view and "Analyze course" stories (FR-9)
- [x] 3.11 Cited chat stories (FR-10)
- [x] 3.12 LLM failure and grounding coverage: check that the relevant stories in E7, E8, E9 and E10 (and the lecture suggestion in E4 and site parsing in E3) have failure criteria, per Q7: A
- [-] 3.13 Team admin stories: SKIPPED per Q8: A
- [x] 3.14 Apply prioritization (Q6) and mark the key-journey stories (Q9)

### Step 4: Quality Checks
- [x] 4.1 Every story uses "As a <persona>, I want <goal>, so that <benefit>"
- [x] 4.2 Every story has acceptance criteria in the format chosen in Q5
- [x] 4.3 INVEST check (Independent, Negotiable, Valuable, Estimable, Small, Testable); split or merge stories that fail it
- [x] 4.4 Traceability table: each story maps to FR/NFR IDs, and every FR-1 to FR-10 is covered by at least one story
- [x] 4.5 Persona-to-story mapping complete (personas.md 2.3)
- [x] 4.6 Validate markdown and any diagrams per content-validation.md

### Step 5: Wrap-Up
- [x] 5.1 Update aidlc-state.md and audit.md
- [x] 5.2 Present the User Stories completion message for approval
