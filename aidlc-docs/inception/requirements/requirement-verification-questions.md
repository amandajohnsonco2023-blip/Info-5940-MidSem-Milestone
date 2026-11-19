# Margins: Requirements Verification Questions

These questions come from reading `README.md`, `Vision.md` and `TechnicalEnvironment.md`. Several of them correspond directly to the open questions in Vision.md section 4. One addresses a conflict between the input documents (Question 1).

Answer each question by putting the letter of your choice after the `[Answer]:` tag. If no option fits, choose the last option (Other) and describe what you want after the tag. You can add a short note after any letter (for example `B - but cap it at 3x`).

---

## Section A: Input-Document Conflict

## Question 1
README.md and Vision.md plan to run open models locally through Ollama (Qwen3 or Llama 3.1 plus `nomic-embed-text`). TechnicalEnvironment.md names the Google Gemini API (`gemini-2.5-flash` and `gemini-embedding-001`) as the only external service and lists Ollama under "Not Using". Which is the source of truth for the LLM provider?

A) Gemini API via `google-genai`, as in TechnicalEnvironment.md (README.md and Vision.md will be updated to match)

B) Ollama with local models, as in README.md and Vision.md (TechnicalEnvironment.md will be updated to match)

C) Gemini as the primary provider, behind a model interface so a local Ollama model could be swapped in later (still one provider at a time)

D) Other (please describe after [Answer]: tag below)

[Answer]: D Primary source is professor-uploaded documents, secondary (fallback) to student-uploaded documents.

---

## Section B: Accounts, Courses and Roles

## Question 2
How does a user become an instructor for a course? (Role determines who can endorse notes.)

A) Self-declared: any user can pick "instructor" when they create or join a course (acceptable for a local demo)

B) The user who creates a course is its instructor; they share a separate instructor join code with TAs, and everyone else joins as a student

C) Instructor role is assigned by an existing instructor of the course from a member list

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 3
How does a student join an existing course?

A) Open: any signed-in user can browse the course list and join any course

B) Join code: the course creator gets a short code to share; users enter it to join

C) Request and approve: users request to join and an instructor approves

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 4
Can a student create a course page, or only instructors? (Vision.md says "an instructor or student enters the URL".)

A) Anyone can create a course; the creator becomes its owner and can edit the lecture outline

B) Only users with the instructor role can create a course

C) Anyone can create a course, but a student-created course has no instructor until one joins with the instructor code

D) Other (please describe after [Answer]: tag below)

[Answer]: B, students can request courses to instructors, instructor roles are assigned via public posted professor job titles online for all currently offered courses.

---

## Section C: Course Structure from the Website

## Question 5
How far should the course-site fetcher go when parsing the public course website?

A) Only the single URL the user enters (the syllabus or schedule page)

B) The entered URL plus same-site pages it links to, one level deep (for example a separate schedule page or lecture pages), with a page-count cap

C) Option B, and also download linked lecture PDFs or slides so their text is available to the agent

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 6
What should happen when the agent cannot find a clear schedule on the course site (Vision.md open question 9)?

A) Show what was extracted (possibly nothing) and let the user add, edit and delete lectures by hand in a form

B) Let the user paste the schedule text directly, then re-run extraction on the pasted text, with hand editing afterward

C) Both A and B

D) Other (please describe after [Answer]: tag below)

[Answer]: No schedule if notes don't offer, describe limitation.

---

## Section D: Notes Upload and Assignment

## Question 7
How should the system handle notes that span several lectures or are not tied to any lecture (Vision.md open question 4)?

A) One note maps to exactly one lecture or to "General / unassigned"; multi-lecture notes must be split by the uploader before uploading

B) One note can be assigned to several lectures; all its passages are tagged with every assigned lecture

C) Assignment happens per passage: the system proposes a lecture for each section of the note, and the uploader confirms or changes them

D) Other (please describe after [Answer]: tag below)

[Answer]: D, doesn't have to tag by lecture

## Question 8
Who can see uploaded notes, and can uploaders hide their name (Vision.md open question 7)?

A) All course members see all notes and the uploader's name

B) All course members see all notes; the uploader can choose to show their name or appear as "Anonymous classmate" (instructors still see the real name)

C) All course members see all notes; uploader names are always hidden from students and visible only to instructors

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 9
What can an uploader do with a note after uploading it?

A) Nothing; uploads are permanent for the MVP

B) Delete their own note (its passages, ratings and endorsements are removed, and guides that cited it show the citation as "removed")

C) Delete their own note, or replace its file with a new version

D) Other (please describe after [Answer]: tag below)

[Answer]: C, with a "last updated tab"

---

## Section E: Ratings, Endorsements and Weighting

## Question 10
What rating scale should students use on other students' notes?

A) 1-5 stars

B) Thumbs up / thumbs down

C) A single "helpful" upvote (no downvotes)

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 11
How should endorsements and ratings combine into a note's quality weight (Vision.md open question 1)?

A) Endorsement dominates: any endorsed note always ranks above any unendorsed note on equal relevance; ratings only order notes within each group

B) Endorsement boosts: weight = base + bounded rating score (smoothed so a few early ratings cannot swing it, for example a Bayesian average) + a fixed endorsement bonus; relevance still matters most

C) Ratings only influence ranking after a minimum number of ratings (for example 3); endorsement is a fixed multiplier

D) Leave the exact formula to Functional Design, but it must be deterministic code, explainable to users, and configurable in one place

E) Other (please describe after [Answer]: tag below)

[Answer]: B plus LLM reference on cross-reference with other uploaded content, with commonly found information being more likely to be correct

## Question 12
What should the product do when a course has no endorsements (Vision.md open question 6)?

A) Nothing special: weight comes from ratings alone, and the guide notes that no instructor has endorsed any source

B) As A, and the instructor coverage view prompts instructors with a list of the highest-rated unendorsed notes to review

C) As A, and also let students mark a note as "matches the posted lecture slides", which acts as a weaker signal than an endorsement

D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Section F: Study Guide, Conflicts and Citations

## Question 13
How should "disagreement" between notes be decided (Vision.md open question 2)?

A) The LLM compares retrieved passages on the same topic and flags a conflict only for factual differences (definitions, formulas, numbers, claims), not wording; each flag shows both versions with sources

B) As A, but a conflict is flagged only when the model also gives a short stated reason and quotes the specific differing text from each source, to cut false alarms

C) As B, and users or instructors can dismiss a flagged conflict as "not a real disagreement"; dismissals are remembered

D) Other (please describe after [Answer]: tag below)

[Answer]: Combination of majority and LLM, option A, flagged to instructor

## Question 14
What should citations point to, and how are they displayed (Vision.md open question 5)?

A) Note level: the citation names the note and its uploader; clicking opens the whole note

B) Passage level: the citation links to the specific passage (chunk) of the note, which is shown highlighted in context

C) Line-range level: the citation names exact line numbers or page numbers inside the note

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 15
Should generated study guides be saved?

A) No; each guide is generated on request, shown, and discarded (the user can copy it)

B) Yes; guides are saved per course and lecture range, listed for all course members, and can be regenerated (latest version only)

C) Yes, with version history so users can compare a regenerated guide with earlier versions

D) Other (please describe after [Answer]: tag below)

[Answer]: C (tied to exam schedule if available)

## Question 16
Should grounded-chat conversations be saved?

A) No; chat lasts for the browser session only

B) Yes, each user's chat history per course is saved and visible only to that user

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 17
Where are conflicts shown in the instructor coverage view (MVP feature 8) coming from?

A) From the most recent study guide generated for that course (conflicts are only detected when a guide is generated)

B) From a separate "analyze course" action an instructor can run, which detects conflicts per lecture without producing a full guide

C) Both: saved from generated guides, and refreshable by an instructor-run analysis

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Section G: Non-Functional Requirements

## Question 18
What response time is acceptable for study guide generation on a range of lectures?

A) Under 30 seconds, with a progress indicator

B) Up to about 2 minutes, with step-by-step progress shown (for example "fetching notes for Lecture 3...")

C) Several minutes is fine; run as a background job and show the guide when ready

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 19
What data size should the MVP be designed and tested for?

A) Demo scale: 1-2 courses, about 10-30 users, about 25 lectures and up to about 200 notes per course

B) Small class scale: up to about 100 users and 500 notes per course

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 20
How will the team evaluate guide quality (Vision.md open question 8)?

A) A planted test set (real notes with known gaps and conflicts) checked manually during demos only

B) A planted test set checked by an automated evaluation script that reports whether known gaps and conflicts were found, run on demand (not in the regular pytest suite because it calls the live model)

C) As B, plus fixed recorded model responses so the agent loop itself is covered by normal pytest unit tests without calling the API

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Section H: Extension Opt-Ins

## Question 21: Security Extensions
Should security extension rules be enforced for this project?

Note: TechnicalEnvironment.md section 6 already lists security essentials (bcrypt, signed cookies, input limits, safe URL fetching, prompt-injection handling, no `|safe`). Those apply either way; this question is about enforcing the full extension rule set as blocking constraints.

A) Yes — enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)

B) No — skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)

X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 22: Resiliency Extensions
Should the resiliency baseline be applied to this project?

**What this extension is.** Enabling it applies a set of **directional, design-time best practices** for building resilient systems, derived from the **AWS Well-Architected Framework (Reliability Pillar)** and resilience-review guidance. It steers requirements, design, and code toward fault tolerance, high availability, observability, and recoverability — covering 15 practice areas across business goals, change management, observability, high availability, disaster recovery, and continuous improvement.

**What this extension is NOT.** Enabling it does **not** make your workload production-ready, nor does it certify or guarantee any availability, RTO, or RPO target. It is a **starting point** that scaffolds good resiliency decisions early — it is not a substitute for a formal **AWS Well-Architected Review** of the built system.

Treat the output as a well-grounded **first draft of your resiliency posture** to build on and validate — not a finished, production-certified result.

A) Yes — apply the resiliency baseline as directional best practices and design-time guidance (recommended for business-critical workloads, as an informed starting point that you can validate and harden before go-live)

B) No — skip the resiliency baseline (suitable for PoCs, prototypes, and experimental projects where rapid iteration matters more than reliability)

X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 23: Property-Based Testing Extension
Should property-based testing (PBT) rules be enforced for this project?

A) Yes — enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)

B) Partial — enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)

C) No — skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)

X) Other (please describe after [Answer]: tag below)

[Answer]: A
