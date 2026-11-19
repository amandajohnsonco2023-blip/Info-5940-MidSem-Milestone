# Margins: Requirements Clarification Questions

Thank you for answering `requirement-verification-questions.md`. Most answers are clear. The ones below either don't answer the question that was asked, contradict another answer, or add a new feature whose shape is unclear. Each section quotes your original answer.

Answer the same way as before: put a letter after `[Answer]:`, or choose the last option (Other) and describe what you want. Notes after the letter are welcome.

---

## Ambiguity 1: LLM provider is still undecided (Q1)
Q1 asked which document is the source of truth for the **LLM provider** (Gemini API in TechnicalEnvironment.md vs. local Ollama in README.md and Vision.md). Your answer was: "D Primary source is professor-uploaded documents, secondary (fallback) to student-uploaded documents." That answer is about **which course content the agent trusts**, which is a separate, useful requirement (see Ambiguity 2). The provider question still needs an answer, because it decides which SDK, models, API key and test setup Construction uses.

### Clarification Question 1
Which LLM provider should Margins use?

A) Gemini API via `google-genai` (`gemini-2.5-flash` and `gemini-embedding-001`), as in TechnicalEnvironment.md; README.md and Vision.md get updated to match

B) Ollama with local models, as in README.md and Vision.md; TechnicalEnvironment.md gets updated to match

C) Gemini as the primary provider, behind a small model interface so a local Ollama model could be substituted later (only one provider active at a time)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Ambiguity 2: Professor-uploaded documents as the primary source (Q1)
Your Q1 answer adds a feature: professors upload their own documents, and the agent treats them as the primary source, with student notes as the fallback. Vision.md only has instructors *endorsing* student notes, not uploading their own material.

### Clarification Question 2
What are "professor-uploaded documents", and how do they relate to student notes?

A) Instructors can upload course materials (slides, handouts, their own notes) through the same upload flow; these are marked "instructor material" and the agent treats them as authoritative over any student note. Material linked from the course website (Q5: C) counts the same way.

B) As A, but only instructor uploads count as authoritative; PDFs fetched from the course website are treated like ordinary sources

C) No separate upload: "professor documents" means the lecture PDFs and slides fetched from the public course website (Q5: C); these are authoritative, and student notes fill gaps

D) Other (please describe after [Answer]: tag below)

[Answer]: B

### Clarification Question 3
When a student note disagrees with professor material, what should the study guide do?

A) Use the professor material's version and silently ignore the student version

B) Use the professor material's version, and show the student version as a flagged disagreement so students see the correction

C) Treat it like any other disagreement: show both versions, but mark the professor version as authoritative

D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Contradiction 1: How instructors get their role (Q2 vs. Q4)
Q2: "B" says the course creator is the instructor and shares an instructor join code with TAs.
Q4: "B, students can request courses to instructors, instructor roles are assigned via public posted professor job titles online for all currently offered courses."

These conflict on how someone becomes an instructor. Q4 also describes verifying instructors against public faculty listings online. That is a form of identity verification, and Vision.md puts university SSO and production-grade auth out of scope. Automatically matching an account to a public faculty page is unreliable: anyone can type a professor's name. It is also hard to build in the time available.

### Clarification Question 4
How does a user get the instructor role?

A) Self-declared at sign-up (honor system, acceptable for a local demo); only instructor-role users can create courses; TAs join a course with the instructor join code from Q2

B) Seeded by the team: instructor accounts are created by a setup script or admin command from a small list (for example names and emails copied from public course pages for the demo courses); everyone else signs up as a student

C) Self-declared at sign-up, but the course page shows the instructor names listed on the public course website next to the course owner, so students can check them by eye (no automatic verification)

D) Other (please describe after [Answer]: tag below)

[Answer]: B

### Clarification Question 5
Q4 also says "students can request courses to instructors." What should that look like?

A) Out of scope for the MVP: students can only join courses an instructor has already created

B) A student fills in a short form (course name, course website URL, instructor email). It shows up on a "requested courses" list that any instructor can see and act on by creating the course. No email is sent, since notifications are out of scope.

C) A student can create a "pending" course page and start uploading notes. An instructor can later claim it, becoming its owner and getting the join codes.

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Ambiguity 3: Missing schedule on the course site (Q6)
Your answer: "No schedule if notes don't offer, describe limitation." My reading: if the course website has no clear schedule, the system tries to infer lectures from the uploaded notes. If it still can't, the course runs without a schedule, and the guide and coverage views say that lecture-level coverage isn't available. Is that right, and can users still correct the outline by hand? Vision.md feature 2 says the outline can be reviewed and corrected.

### Clarification Question 6
What happens when the site has no clear schedule?

A) Show a clear "no schedule found" message and run the course without lectures. Guides are organized by topic only, and coverage reporting is disabled. No inference from notes, no manual editing.

B) As A, but first try to infer a topic list from the uploaded notes. Users cannot edit the outline by hand.

C) As B, and the course owner can still add, edit or delete lectures by hand in a form (this applies to every course, not only ones without a schedule)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Ambiguity 4: Notes without lecture tags (Q7) vs. per-lecture coverage
Your answer: "D, doesn't have to tag by lecture." Several features count notes per lecture: coverage gaps in the guide (Vision feature 6), the instructor coverage view (feature 8), and guides for a selected lecture range. If notes aren't tied to lectures, these features need another way to know what a note covers.

### Clarification Question 7
How should notes relate to lectures?

A) Lecture tagging is optional. The system suggests one, and the uploader can accept it, pick one or more lectures, or leave it untagged. Coverage and lecture-range guides use tagged notes plus untagged passages the system matches to a lecture by topic similarity.

B) No lecture tagging at all. The system matches each passage to the schedule's lectures or topics by semantic similarity, and coverage is computed from those matches.

C) Lecture tagging is optional, and untagged notes only appear as general sources (retrieved by relevance); they don't count toward per-lecture coverage

D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Ambiguity 5: "Commonly found information is more likely correct" (Q11)
Your answer: "B plus LLM reference on cross-reference with other uploaded content, with commonly found information being more likely to be correct."

Option B gives each **note** a quality weight in deterministic code: an upvote score smoothed with a Bayesian average (Q10: C), plus an endorsement bonus. "Commonly found information" is a different kind of signal. It applies to individual **claims** (how many sources agree) and is judged by the LLM at the time a guide is written.

### Clarification Question 8
How should cross-source agreement be used?

A) Keep it separate from the stored note weight. The note weight (upvotes, endorsement, "matches slides" flag) stays deterministic code. While writing a guide or answering in chat, the agent is told how many independent notes support each version of a claim, and prefers the version with more support, unless professor material or an endorsed note says otherwise.

B) Also feed it back into the note weight: notes whose content often agrees with the majority get a higher stored weight over time

C) Other (please describe after [Answer]: tag below)

[Answer]: B

### Clarification Question 9
What is the order of precedence when sources disagree? (This combines Q1, Q11, Q12 and Q13.)

A) Professor material > instructor-endorsed note > "matches slides" note > majority of other notes > upvotes

B) Professor material > instructor-endorsed note > majority of notes > "matches slides" note > upvotes

C) Professor material > majority of notes > instructor-endorsed note > upvotes ("matches slides" is shown only as a badge)

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Ambiguity 6: "Flagged to instructor" (Q13)
Your answer: "Combination of majority and LLM, option A, flagged to instructor." Notifications and email are out of scope in Vision.md, so I want to confirm what "flagged to instructor" means.

### Clarification Question 10
How should a detected conflict reach the instructor?

A) It appears in the instructor coverage view (per lecture, with both versions and their sources). The guide shows the conflict to students too, marking the majority version as "more commonly reported". No notifications.

B) As A, and the instructor can resolve it from that view by choosing the correct version or endorsing a note; the resolution is used in later guides

C) It appears only to instructors, in the coverage view. Students see just the majority version in the guide until an instructor resolves it.

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Ambiguity 7: Guides "tied to exam schedule if available" (Q15)
Your answer: "C (tied to exam schedule if available)." It's unclear whether this changes how a guide's scope is chosen or only how saved guides are labeled.

### Clarification Question 11
What does "tied to exam schedule" mean?

A) When the course site lists exams (for example "Prelim 1: Lectures 1-10"), the parser extracts them. Users can then generate a guide for an exam, which selects that exam's lecture range automatically. Saved guides (with version history) are grouped by exam. Users can still pick a custom lecture range.

B) Only labeling: saved guides show the next upcoming exam date from the site, but the scope is always a lecture range chosen by hand

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Confirmation 1: Security baseline opted out (Q21)
You chose "B" (skip the security extension rules). TechnicalEnvironment.md section 6 already lists security essentials, and I plan to keep those as normal requirements:
- bcrypt password hashes and a signed session cookie
- role checks
- Pydantic length limits
- upload type and size limits
- public-URL-only fetching with timeouts and size caps
- treating notes and website text as content, never as instructions
- no `|safe`
- secrets kept in `.env`

The extension's extra blocking rules won't be applied.

### Clarification Question 12
Is that right?

A) Yes. Keep the TechnicalEnvironment.md security essentials as requirements; don't enforce the security extension rules.

B) No. Drop the security essentials as well (not recommended: the app handles passwords and fetches arbitrary URLs).

C) Other (please describe after [Answer]: tag below)

[Answer]: A
