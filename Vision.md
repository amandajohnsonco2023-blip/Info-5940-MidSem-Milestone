# Margins: Vision Document

> **Alignment note:** This document was updated on November 23, 2026 to match the decisions made during AI-DLC Inception, recorded in `aidlc-docs/inception/requirements/requirements.md` and `aidlc-docs/inception/user-stories/stories.md`. Where the two differ, `requirements.md` wins. The **Ref.** column in the MVP feature table gives the IDs that `requirements.md` uses: **V-n** for the original MVP feature n, and **FR-n** for its functional requirements.

## Executive Summary

Margins is a locally run web platform where undergraduate students in the same lecture-based course pool their notes, and an AI agent turns that shared pool into a study guide grounded in the course itself. The agent reads every set of notes uploaded to a course page, along with any slides and handouts the instructor uploads, fetches the course's public website to compare the notes against the syllabus and schedule, and writes a study guide that merges overlapping notes, flags disagreements, cites its sources, and lists lectures nobody has covered. When sources disagree, Margins trusts them in a fixed order: instructor material first, then instructor-endorsed notes, then what most notes say, then notes flagged as matching the slides, then upvotes. The same rules apply in a grounded chat that answers questions with citations. The expected outcome is that a student can go from a scattered pile of classmates' notes to one trustworthy, cited study guide in minutes instead of hours of manual reconciliation.

---

## Business Context

### Problem Statement

Students in the same course take notes on the same lectures, but each set is incomplete in its own way: one student misses a definition, another skips a lecture, a third writes down an example incorrectly. As exams approach, students trade notes in group chats and end up with a pile of overlapping, inconsistently formatted, and sometimes contradictory documents. They then reconcile them by hand, deciding which version of a definition is right, discovering that nobody has notes for a missed lecture, and guessing whether any of it matches what the syllabus says will be tested.

Existing tools do not solve this. Shared drives and note-sharing sites store files but do not reconcile them. General-purpose chatbots summarize whatever one student uploads, answer from general knowledge rather than from this course's lectures, and do not know what the rest of the class wrote or what the course actually covers. The result is that students either spend hours merging notes or study from material that may contradict what the professor taught.

### Business Drivers

- **Exam-season pressure.** The pain is concentrated before midterms and finals, when students have the least time and the most need for a complete, accurate picture of the course.
- **Group chats already exist.** Students already share notes informally; Margins replaces an existing habit rather than asking them to start a new one.
- **LLMs can now do the reconciliation.** Matching "the same idea" across notes written in different vocabulary, spotting conflicting statements of a formula, and mapping a note titled "Tuesday stuff" to the right lecture are semantic tasks that rule-based software handles poorly and current LLMs handle well, provided their output is grounded in and cited back to the source notes.
- **Instructors want students studying accurate material.** Uploading their own slides, endorsing notes, and resolving conflicts give professors and TAs lightweight, visible ways to steer students toward accurate material without writing a study guide themselves.

### Target Users and Stakeholders

| User Type | Description | Primary Need |
|-----------|-------------|--------------|
| Student contributor | Undergraduate enrolled in a lecture-based course who takes notes and uploads them to the course page | Upload notes quickly, tag them to the right lectures, choose whether their name is shown, and see how they compare with the class's notes |
| Student studier | Student preparing for an exam, often with gaps from missed or poorly noted lectures | One trustworthy study guide for an exam that fills gaps, shows where sources disagree, and lets them check every claim |
| Professor | Teaches the course; has an instructor account that the team sets up, and creates the course page | Upload authoritative course material, endorse accurate notes, resolve conflicts, and see which lectures lack coverage |
| TA | Helps teach the course; joins its course page with the instructor join code | The same instructor permissions as the professor, within that course only |
| Project team (stakeholder) | INFO 5940 team building Margins | A meaningful, demo-ready MVP by the final project deadline |
| Course staff (stakeholder) | INFO 5940 instructors and graders | Evidence that the product uses LLMs meaningfully and was built through AI-DLC |

### Business Constraints

- **Timeline:** Inception complete by November 24, 2026; working MVP, demo video, and presentation by December 16, 2026.
- **Team:** Five students working part-time alongside other courses.
- **Deployment:** The product runs locally on team laptops only. No hosting or cloud deployment.
- **Course content and sharing:** Some courses restrict sharing of materials. Course pages are opt-in, and instructors have a visible role in what is shared through uploads, endorsements, and conflict resolution.
- **Data sources:** Only public course websites can be used; sites behind a login (Canvas and other learning management systems) cannot be fetched.
- **LLM provider:** Google Gemini API only, through one team member's API key (models are listed in the Technical Environment Document).
- **Budget:** No paid services beyond what the Gemini API offers within its free or low-cost usage.

### Success Metrics

Metrics are measured on a team-built evaluation set: one or two real courses with public websites, about 10 lectures each, and about 30 uploaded notes, with coverage gaps, conflicts between notes, and a contradiction between instructor material and student notes deliberately planted. An on-demand evaluation script runs the live agent on this set and reports which planted items it found.

| Metric | Current State | Target State | Measurement Method |
|--------|---------------|--------------|-------------------|
| Time to a usable study guide for 10 lectures | 1–3 hours of manual merging | About 2 minutes from clicking "Generate", with step-by-step progress shown | Timed runs on the evaluation set |
| Chat response time | Not applicable | Answers normally return within about 20 seconds | Timed runs of the scripted question set |
| Coverage gaps reported | Discovered by accident, often the night before the exam | 100% of planted missing or thin lectures reported | Evaluation script compares the coverage report with the planted gaps |
| Conflicts flagged | Not detected | At least 4 of 5 planted conflicts flagged, with every version shown and the majority version labeled | Evaluation script compares flagged conflicts with the planted conflicts |
| Citation validity | No citations | 100% of guide sections and chat answers cite at least one source; every citation links to the note or course website page it names | Automated check that each citation resolves to a stored source |
| Grounded refusals | Chatbots answer from general knowledge | At least 4 of 5 questions the class materials do not cover are answered with an explicit statement that they are not covered | Scripted question set |
| Lecture outline accuracy | Typed in by hand | At least 80% of lectures extracted correctly (date and title) from each demo course site | Compare the extracted outline with the real schedule |
| Source precedence | No signal of quality | When sources conflict, instructor material is presented as authoritative, and an endorsed note is presented ahead of unendorsed notes, in 100% of cases | Scripted test cases on the evaluation set |

---

## Full Scope Vision

### Product Vision Statement

Every lecture-based course has a living, class-built study companion: students contribute notes as the term goes on, instructors add their own material and endorse what is accurate, and anyone in the class can get a complete, cited picture of what the course covered, where understanding differs, and what is still missing.

### Feature Areas

#### Feature Area 1: Courses and Membership

- **Description:** Course pages that students and instructors join, with roles.
- **Key Capabilities:**
  - Instructors create an opt-in course page for their course
  - Students join with a student join code; TAs join with an instructor join code
  - Student and course-instructor (professor or TA) roles, with role-based permissions
  - Students can request a course that is not on Margins yet
  - University single sign-on, verified instructor identity, and roster import (full vision)
  - Integration with learning management systems such as Canvas (full vision)
- **User Value:** Notes are shared only with the right class, and instructors have a recognized role.

#### Feature Area 2: Course Structure

- **Description:** A lecture-by-lecture outline of the course that notes are mapped to, taken from the course's public website.
- **Key Capabilities:**
  - Fetch the course's public website, the same-site pages it links to, and linked PDFs such as slides and handouts
  - Extract lectures (number, date, title, topics) and, when the site lists them, exams (name, date, lecture range)
  - Make the fetched website text searchable as a source for guides and chat
  - Refresh on request; the website is the only source of the outline
  - Run in topic-only mode when no schedule can be found
  - Automatic syncing when the site changes, and support for login-protected sites (full vision)
- **User Value:** Study guides and coverage reports are organized around what the course actually teaches and the exams it actually has.

#### Feature Area 3: Notes, Instructor Material, and Quality Signals

- **Description:** Contributing, organizing, and judging the class's sources.
- **Key Capabilities:**
  - Upload typed notes, with the system suggesting which lecture they cover
  - Instructors upload their own slides, handouts, and notes as authoritative material
  - Tag notes to one or more lectures, or leave them untagged
  - Show the uploader's name or appear as "Anonymous classmate"
  - Delete or replace your own notes
  - Upvote classmates' notes, flag notes as matching the posted slides, and have instructors endorse notes
  - Combine these signals, plus how often a note agrees with the rest of the class, into a quality weight per note
  - Handwritten and photographed notes via handwriting recognition; lecture audio transcription (full vision)
  - Comments and discussion on notes (full vision)
- **User Value:** The class's sources are in one place, and more trustworthy sources carry more weight.

#### Feature Area 4: Study Guide Agent

- **Description:** An agent that plans its own steps and calls tools to build a guide.
- **Key Capabilities:**
  - Scope a guide to an exam, a range of lectures, or the whole course
  - Organize the guide by lecture and topic from the course outline
  - Merge overlapping notes into one section per topic
  - Flag factual disagreements, show each version with its sources, and label the majority and authoritative versions
  - Report lectures with missing or thin coverage
  - Cite the source behind every claim
  - Save every guide, with version history and comparison between versions
  - Generated practice questions, flashcards, and spaced repetition (full vision)
- **User Value:** One guide that is complete where the class's sources are complete, honest about conflicts, and clear about gaps.

#### Feature Area 5: Grounded Chat

- **Description:** Question answering over the class's notes, instructor material, and course site.
- **Key Capabilities:**
  - Answer questions using the course's sources, following the same trust order as the guide
  - Cite the notes and pages used
  - Say explicitly when the class materials do not cover a question
  - Keep each user's chat history private to them
- **User Value:** Quick answers that match this course, not the internet's version of the topic.

#### Feature Area 6: Instructor Insight

- **Description:** A view of the class's collective understanding for professors and TAs.
- **Key Capabilities:**
  - Per-lecture note counts, endorsements, instructor material, open and resolved conflicts, and missing or thin status
  - Run a course-wide conflict analysis without generating a guide
  - Resolve a conflict by choosing the correct version, so later guides use it
  - See highly weighted notes that have not been endorsed yet
  - Trends in which topics students disagree on most (full vision)
- **User Value:** Instructors see where students are confused or under-served and can respond.

### Integration Points

- **Public course websites:** source of the syllabus, schedule, exams, and linked lecture materials.
- **Google Gemini API:** reading, reconciling, writing, and embeddings (models are specified in the Technical Environment Document).
- **University single sign-on and learning management systems (full vision):** login, rosters, verified instructor identity, and access to protected course pages.

### User Journeys (Full Vision)

#### Journey 1: Contributing through the term

1. A student signs in with university credentials; their enrolled courses' pages are already listed.
2. After each lecture, they upload typed or photographed notes; the system assigns them to the right lecture.
3. Classmates upvote the notes and the TA endorses them.

**Outcome:** The student's notes help the whole class, and the class's best notes rise to the top.

#### Journey 2: Studying for a final

1. A student opens the course page a week before the final and selects the exam.
2. Margins generates a cited study guide with conflicts and gaps flagged, plus practice questions.
3. The student asks follow-up questions in chat and reviews flashcards in the days before the exam.

**Outcome:** The student studies from a complete, verifiable guide aligned with the course.

#### Journey 3: Instructor check-in

1. A professor opens the instructor view mid-term.
2. They see that two lectures have almost no notes and that students disagree about one key definition.
3. They resolve the conflict, endorse the most accurate note, and clarify the definition in the next lecture.

**Outcome:** The instructor corrects misunderstandings before the exam.

### Scalability and Growth

At maturity, Margins would support many courses per university, many universities, and larger classes, with LMS integration replacing manual setup. For this project, it is designed and tested at demo scale: one or two courses, about 10–30 users, about 25 lectures, and up to about 200 notes per course. Every feature works inside one course at a time.

### Long-Term Roadmap

| Phase | Focus | Timeframe |
|-------|-------|-----------|
| MVP | Course pages and join codes, course-site outline and exams, notes and instructor material upload, upvotes and endorsements, source weighting, cited study guides, coverage report, conflict flagging and resolution, grounded chat, instructor coverage view | December 16, 2026 |
| Phase 2 | Handwriting recognition, comments, practice questions and flashcards, scheduled site syncing | TBD |
| Phase 3 | University SSO, LMS integration, multi-course and multi-university support, hosting | TBD |

---

## MVP Scope

### MVP Objective

Prove that an agent can turn a class's pooled notes, its instructor's material, and its public course website into a study guide that students can trust, because every claim is cited, conflicts are surfaced rather than hidden, and missing lectures are reported.

### MVP Success Criteria

- [ ] A student can sign up, join a course, upload notes, and get a cited study guide and cited chat answers in one session, without help from the team.
- [ ] The guide's coverage report finds every planted missing or thin lecture in the evaluation set.
- [ ] The guide flags at least 4 of 5 planted conflicts and shows every version with its sources.
- [ ] Every statement in a guide links to a source, and every citation opens the note or page it names.
- [ ] Chat explicitly declines at least 4 of 5 questions the class materials do not cover.
- [ ] Instructor material, an endorsement, or a resolved conflict visibly changes what the next guide says or presents first.

### Features In Scope (MVP)

Priorities follow the approved user stories: **Must** is needed for the MVP loop, **Should** is expected in the MVP, and **Could** is included if time allows.

| Feature | Description | Priority | Ref. | Rationale for Inclusion |
|---------|-------------|----------|------|------------------------|
| Accounts and roles | Local sign-up with email and password always creates a student account. The team sets up instructor accounts with a setup command; there is no way to become an instructor through the app. | Must | V-1, FR-1 | Endorsements and permissions depend on roles, and seeded instructor accounts keep students from granting themselves instructor powers. |
| Course pages and join codes | Instructors create a course page (name, code, term, optional website URL). Students join with a student join code. Notes, guides, and chat are scoped to one course. | Must | V-1, FR-2 | Notes must be shared only within the course they belong to. |
| TA membership | TAs join a course with its instructor join code and get instructor permissions in that course only. | Should | V-1, FR-2.2 | Lets TAs endorse and review without a seeded account. |
| Course requests | Any signed-in user can request a course that is not on Margins yet; instructors create it from the request or dismiss it. | Could | FR-2.5 | Gives students a path to a missing course without email or notifications. |
| Course outline from public website | The system fetches the course's public website, same-site pages one level deep, and linked PDFs, and extracts a lecture-by-lecture outline (number, date, title, topics). The outline cannot be edited by hand. If no schedule is found, the course page says so and the course runs in topic-only mode with no coverage report. Fetched text becomes searchable website sources. | Must | V-2, FR-3 | Guides and coverage reports are organized around the course; without the outline there is nothing to compare notes against. |
| Exams and refresh | When the site lists exams, they are extracted (name, date, lecture range) so guides can be scoped by exam. Course instructors can refresh the import, which replaces the outline and exam list. | Should | V-2, FR-3.3, FR-3.4 | Students study for specific exams, and sites change during the term. |
| Notes upload and lecture tagging | Course members upload typed notes (Markdown, plain text, or text-based PDF, up to 5 MB). The system suggests a lecture, or none if unsure; the uploader accepts it, picks one or more lectures, or leaves the note untagged. Uploaders choose to show their name or appear as "Anonymous classmate". | Must | V-3, FR-4 | Pooled notes are the core input. |
| Delete and replace notes | Uploaders can delete their own notes or replace the file with a new version that keeps its upvotes and endorsement. | Should | V-3, FR-4.6 | Students need to fix mistakes in what they shared. |
| Instructor material | Course instructors upload slides, handouts, and their own notes. These are labeled "Instructor material" and are the most trusted source. | Must | FR-4.2 | Gives the agent an authoritative source to check student notes against. |
| Upvotes and endorsements | Students upvote classmates' notes (no downvotes). Course instructors endorse notes, which then show an "Instructor-endorsed" badge. | Must | V-4, FR-5 | The product's trust model depends on quality signals. |
| "Matches the slides" flag | Students flag a note as matching the posted lecture slides; this is a weaker signal than an endorsement. | Should | V-4, FR-5.2 | Lets students vouch for accuracy, not just helpfulness. |
| Quality weight and source precedence | Ordinary code, not the LLM, computes each note's quality weight from smoothed upvotes, the slides flag, endorsement, and how often the note agrees with the majority in past conflict checks. When sources disagree, the order is: instructor material, endorsed note, majority of notes, slides-flagged note, upvotes. A help page explains the ranking in plain language. | Must (agreement score and help page: Should) | V-4, V-5, FR-6 | Trust must be predictable, explainable, and testable. |
| Weighted search | Notes are split into passages and searched by meaning. Results combine relevance with the source's quality weight. Website sources rank like ordinary student notes. | Must | V-5, FR-6.5 | Both the guide and chat depend on finding the right passages. |
| Study guide agent | For an exam, a lecture range, or the whole course, the agent decides which tools to use and in what order, and writes a guide that merges overlapping notes, cites the source note behind every claim, and states when no instructor has endorsed anything. Progress is shown step by step. | Must | V-6, FR-7 | This is the product's central value. |
| Saved guides and versions | Every guide is saved per course and scope and visible to all course members. Regenerating creates a new version, and users can compare versions. | Should | V-6, FR-7.6 | Students return to guides, and instructors can see what the class is studying from. |
| Coverage report | Lectures in scope with no tagged notes are reported as "missing" and those with fewer than a set number of notes (default 2) as "thin". Untagged notes do not count. | Must | V-6, FR-7.3 | Telling students what nobody covered is a core promise and is cheap to compute reliably. |
| Conflict flagging | The agent flags factual differences (definitions, formulas, numbers, claims), not rewording. Every version is shown with its sources; the majority version is labeled "More commonly reported" and instructor material "Authoritative (instructor material)". | Must | V-6, FR-8 | Hiding disagreements would make the guide untrustworthy. |
| Conflict resolution | A course instructor resolves a conflict by choosing the correct version, which can also endorse the note behind it. Later guides present it as resolved by the instructor. | Should | FR-8.5 | Gives instructors a direct way to correct misunderstandings. |
| Grounded chat | Course members ask questions. The agent searches the course's sources, follows the same precedence, answers with citations, and says explicitly when the materials do not cover the question. | Must | V-7, FR-10 | Students need targeted follow-up beyond the guide; this shows the same grounding in an interactive form. |
| Private chat history | Each user's conversations are saved per course and visible only to them. | Should | V-7, FR-10.3 | Students return to earlier answers without exposing their questions. |
| Instructor coverage view | Per lecture: number of tagged notes, how many are endorsed, whether instructor material exists, open and resolved conflicts, and missing or thin status. | Must | V-8, FR-9.1 | Gives instructors a reason to engage and endorse; built from data the guide already produces. |
| Analyze course | Course instructors run conflict detection across the course without generating a guide. | Should | V-8, FR-9.2 | Lets instructors find conflicts before students generate guides. |
| Endorsement candidates | The coverage view lists the highest-weighted notes that are not endorsed yet. | Could | V-8, FR-9.4 | Helps instructors decide what to review first. |

### Features Explicitly Out of Scope (MVP)

| Feature | Reason for Deferral | Target Phase |
|---------|---------------------|--------------|
| University single sign-on, production-grade authentication, and automatic verification of instructor identity | Requires university integration; local accounts and team-seeded instructor accounts are enough to demo roles | Phase 3 |
| Integration with Canvas or other systems that require login | Requires credentials and institutional approval; we demo with public course websites | Phase 3 |
| Handwriting recognition, photographed or scanned notes | Adds an unreliable extraction step before the core pipeline; typed notes prove the concept | Phase 2 |
| Audio or video lecture transcription | Large new input type unrelated to the core reconciliation problem | TBD |
| Continuous or scheduled syncing of course websites | Fetching on add and on request covers the demo | Phase 2 |
| Editing the lecture outline by hand | The course website is the single source of the outline; wrong outlines are fixed by refreshing | TBD |
| Inferring a lecture schedule from notes | Decided against in Requirements Analysis (CQ6); courses without a schedule run in topic-only mode instead | TBD |
| Comments and discussion threads | Adds web-development work without strengthening the core product | Phase 2 |
| Downvotes | Decided against in Requirements Analysis (Q10); upvotes, the slides flag, and endorsements give enough signal | TBD |
| Following users | Social feature unrelated to studying from notes | TBD |
| Notifications and email | Not needed for a local demo | TBD |
| Real-time co-editing of notes | High complexity; notes are uploaded as finished documents | TBD |
| Practice questions, flashcards, spaced repetition | Valuable, but builds on a trustworthy guide, which must come first | Phase 2 |
| Study guides spanning multiple courses, cross-course search | MVP is designed around one course at a time | Phase 3 |
| Mobile apps | Web app on laptops is enough for the demo | TBD |
| Hosting and cloud deployment | The product runs locally on team laptops | Phase 3 |
| Plagiarism or academic-integrity checks on uploads | Policy-heavy and outside the core problem | TBD |
| Training or fine-tuning models | Not needed; prompting and retrieval are sufficient | TBD |

### MVP User Journeys

The most important demo path is Journey 2 followed by Journey 4: a student signs up, joins an existing course, uploads notes, and studies with cited chat.

#### Journey 1: Set up a course

1. A professor signs in with the instructor account the team set up and creates a course page with the course's name, code, term, and public website URL.
2. Margins fetches the site and shows the extracted lecture outline and any exams. If no schedule is found, the page says so and the course runs in topic-only mode.
3. The professor shares the student join code with the class and the instructor join code with TAs, and uploads their slides as instructor material.

**Outcome:** The course page has a lecture outline and authoritative material, ready for notes.
**Limitation vs Full Vision:** Instructor accounts are set up by the team instead of university sign-in; no roster import; public sites only; refresh is manual; a wrong outline can only be fixed by refreshing.

#### Journey 2: Contribute notes

1. A student signs up and joins the course with the student join code.
2. They upload a Markdown or PDF file of their notes and choose whether their name is shown.
3. Margins suggests the lecture it covers; the student accepts, picks one or more lectures, or leaves it untagged.
4. Classmates upvote the note or flag it as matching the slides, and the TA endorses it.

**Outcome:** The note is in the class pool with a quality weight.
**Limitation vs Full Vision:** Typed notes only; no comments.

#### Journey 3: Generate a study guide

1. A student selects an upcoming exam (or a range of lectures) and clicks "Generate study guide".
2. Margins shows the agent's progress as it searches the notes, compares them against the outline and instructor material, and writes the guide.
3. The student reads merged sections, reviews flagged disagreements with every version and its label shown, sees which lectures are missing or thin, and clicks citations to open the source notes.
4. The guide is saved; regenerating it later creates a new version the student can compare.

**Outcome:** The student has one cited study guide for the exam.
**Limitation vs Full Vision:** No practice questions or flashcards.

#### Journey 4: Ask a question

1. A student asks in chat, "What's the difference between bias and variance in this course?"
2. The agent searches the course's sources and answers with citations, preferring instructor material and endorsed notes.
3. For a question the class materials do not cover, the agent says so instead of answering from general knowledge.

**Outcome:** The student gets a course-specific answer they can verify, saved in a chat history only they can see.
**Limitation vs Full Vision:** None significant.

#### Journey 5: Instructor check-in

1. A TA joins with the instructor join code and opens the coverage view.
2. They see note counts, endorsements, instructor material, open conflicts, and missing or thin lectures, and run "Analyze course" to refresh the conflicts.
3. They resolve a disputed topic by choosing the correct version and endorsing the note behind it.

**Outcome:** The next guide presents the chosen version as resolved by the instructor, and chat prefers the endorsed note.
**Limitation vs Full Vision:** No trend analysis across the term.

### MVP Constraints and Assumptions

- **Assumption:** Students already take typed notes or can export them as Markdown, text, or text-based PDF. **Risk if wrong:** Few notes can be uploaded and guides are thin.
- **Assumption:** The demo courses' public websites are server-rendered and list a schedule clearly enough to extract lectures. **Risk if wrong:** The course runs in topic-only mode, with no per-lecture structure or coverage report.
- **Assumption:** `gemini-2.5-flash` can reliably call tools and follow citation instructions. **Risk if wrong:** Guides lose citations or skip steps; the agent loop needs more guardrails.
- **Assumption:** One team member's Gemini API key, within its free or low-cost limits, is enough for demo-scale use. **Risk if wrong:** Development or the demo stalls on rate limits.
- **Assumption:** Students are willing to share notes with their class. **Risk if wrong:** The pool is too small to be useful.
- **Accepted Limitation:** One course at a time, with up to about 200 notes, not thousands.
- **Accepted Limitation:** Accounts are local and suitable for a demo, not for real-world deployment; instructor accounts are seeded by the team and their identity is not verified.
- **Accepted Limitation:** The agent can mistake rewording for disagreement; flagged conflicts show every version so the student can judge.
- **Accepted Limitation:** The majority of notes can be wrong; instructor material, endorsements, and resolved conflicts outrank it.
- **Accepted Limitation:** Notes, instructor material, and course-site text are sent to the Google Gemini API for processing.

### MVP Definition of Done

- [ ] All "Must" user stories implemented, with their acceptance criteria passing
- [ ] Success criteria met on the team's evaluation set, checked with the evaluation script
- [ ] Automated tests pass, with at least 80% coverage on business logic and ruff checks clean
- [ ] Property-based tests written with Hypothesis, as required by the Property-Based Testing extension
- [ ] The regular test suite makes no live Gemini API calls
- [ ] Every teammate can start the app locally from the README instructions, including the instructor setup command
- [ ] No API keys or other secrets committed to the repository
- [ ] Demo video and recorded presentation completed for December 16, 2026
- [ ] All features built through the AI-DLC Construction phase, with the record in `aidlc-docs/`

---

## Risks and Dependencies

### Key Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| The agent misses steps, calls tools incorrectly, or drops citations | Medium | High | Keep the tool set small and well described; validate tool arguments and citations in code; treat uncited output as a failure; cap the number of agent steps |
| False conflicts: rewording flagged as disagreement | High | Medium | Flag only factual differences; show every version so students can judge; tune on planted examples in the evaluation set |
| The majority of notes is wrong | Medium | Medium | Instructor material, endorsements, and resolved conflicts outrank the majority; every version stays visible |
| Course websites vary widely in structure | High | Medium | Show a clear message and run in topic-only mode when no schedule is found; demo with one or two well-structured public sites |
| Gemini rate limits or quota reached during development or the demo | Medium | High | Cache embeddings and fetched sites; show clear, retryable errors; keep the evaluation set small; test the full demo early |
| Not enough real notes to show conflicts and gaps | Medium | Medium | Build an evaluation set from team members' own notes, with conflicts and gaps planted deliberately |
| Instructors never endorse anything | Medium | Low | The product works on the other signals, and guides say that no instructor has endorsed any source |
| Prompt injection: note or website text tries to instruct the agent | Low | Medium | Treat note and site text as content, never as instructions; keep permission checks in ordinary code |
| Course materials that may not be shared | Low | Medium | Course pages are opt-in; demo only with courses whose materials are public |
| Scope too large for the time available | Medium | High | Features are prioritized Must, Should, and Could; Could features are cut first, then Should; the key demo journey is protected; the out-of-scope list is enforced |

### External Dependencies

- Google Gemini API (`gemini-2.5-flash` for the agent and chat, `gemini-embedding-001` for embeddings) through the `google-genai` SDK - Google - required; one team member's API key
- Public course websites for the demo courses - course instructors - to be chosen by the team
- Team members' own lecture notes and sample instructor material for the evaluation set - project team - to be collected before Construction

### Resolved Questions

These were open when the vision was first written and were settled during Requirements Analysis.

| Question | Decision | Reference |
|----------|----------|-----------|
| **Weighting formula.** How should endorsements and ratings combine? | Deterministic code combines smoothed upvotes (so a few early votes cannot swing it), a slides-flag bonus, an endorsement bonus, and an agreement score; instructor material has a fixed top weight. Precedence when sources disagree: instructor material, endorsed note, majority, slides flag, upvotes. | FR-6, Q11, CQ8, CQ9 |
| **Conflict detection.** Disagreement or rewording? | Only factual differences (definitions, formulas, numbers, claims) are flagged. The majority is the version supported by the most distinct sources; a tie has no majority label. Every version is shown. | FR-8, Q13, A3 |
| **Lecture assignment.** Notes spanning several lectures or none? | Tagging is optional, and a note can be tagged to several lectures. Untagged notes are still searchable but do not count toward coverage. | FR-4.3, FR-4.4, CQ7 |
| **Citation granularity.** Note, passage, or line range? | Note level: each citation shows the note title and uploader (or "Anonymous classmate") and links to the full note. Website sources are cited as "Course website: page or file". | FR-7.3, FR-3.7, Q14 |
| **No endorsements.** What if instructors never endorse? | The product works on the other signals, and guides state that no instructor has endorsed any source. | FR-5.4, Q12 |
| **Privacy and sharing.** Visible to all? Anonymous uploads? | All course members see all notes. Uploaders can appear as "Anonymous classmate"; course instructors always see the real name. | FR-4.5, Q8 |
| **Course site variety.** No clear schedule? | The course page says no schedule was found, and the course runs in topic-only mode with no coverage report. The outline is never edited by hand or inferred from notes. | FR-3.5, FR-3.6, CQ6 |
| **Course site vs. notes in chat.** Which comes first? | Website text ranks like an ordinary student note. Material the instructor uploads is authoritative and is shown as such when it differs from notes. | FR-3.7, FR-7.4, CQ2, CQ3 |

### Open Questions

- [ ] **Sending notes to a provider.** Students' notes and course-site text are sent to the Google Gemini API. Do users need to be told, or asked to agree, when they join a course page?
- [ ] **Model reliability and limits.** Can `gemini-2.5-flash` call tools reliably and produce well-cited output within its free or low-cost limits? Errors and rate limits are handled with clear, retryable messages (NFR-4), but the model's reliability still has to be tested in Construction.
- [ ] **Weight constants and thresholds.** The exact weight constants and the "thin" coverage threshold are to be set in Functional Design.
- [ ] **Evaluation.** Is the planned evaluation set (one or two courses, about 30 notes, planted gaps and conflicts) large enough to judge guide quality?
