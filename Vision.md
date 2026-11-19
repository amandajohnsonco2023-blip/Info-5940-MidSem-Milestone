# Margins: Vision Document
 
## Executive Summary
 
Margins is a locally run web platform where undergraduate students in the same lecture-based course pool their notes, and an AI agent turns that shared pool into a study guide grounded in the course itself. The agent reads every set of notes uploaded to a class page, fetches the course's public website to compare the notes against the syllabus and schedule, and writes a study guide that merges overlapping notes, flags disagreements, cites its sources, and lists lectures nobody has covered. Professors and TAs can endorse notes, and the agent weights endorsed and highly rated notes more heavily, both in guides and in a grounded chat that answers questions with citations. The expected outcome is that a student can go from a scattered pile of classmates' notes to one trustworthy, cited study guide in minutes instead of hours of manual reconciliation.
 
---
 
## Business Context
 
### Problem Statement
 
Students in the same course take notes on the same lectures, but each set is incomplete in its own way: one student misses a definition, another skips a lecture, a third writes down an example incorrectly. As exams approach, students trade notes in group chats and end up with a pile of overlapping, inconsistently formatted, and sometimes contradictory documents. They then reconcile them by hand, deciding which version of a definition is right, discovering that nobody has notes for a missed lecture, and guessing whether any of it matches what the syllabus says will be tested.
 
Existing tools do not solve this. Shared drives and note-sharing sites store files but do not reconcile them. General-purpose chatbots summarize whatever one student uploads, answer from general knowledge rather than from this course's lectures, and do not know what the rest of the class wrote or what the course actually covers. The result is that students either spend hours merging notes or study from material that may contradict what the professor taught.
 
### Business Drivers
 
- **Exam-season pressure.** The pain is concentrated before midterms and finals, when students have the least time and the most need for a complete, accurate picture of the course.
- **Group chats already exist.** Students already share notes informally; Margins replaces an existing habit rather than asking them to start a new one.
- **LLMs can now do the reconciliation.** Matching "the same idea" across notes written in different vocabulary, spotting conflicting statements of a formula, and mapping a note titled "Tuesday stuff" to the right lecture are semantic tasks that rule-based software handles poorly and current LLMs handle well, provided their output is grounded in and cited back to the source notes.
- **Instructors want students studying accurate material.** Endorsement gives professors and TAs a lightweight, visible way to steer students toward good notes without writing a study guide themselves.
### Target Users and Stakeholders
 
| User Type | Description | Primary Need |
|-----------|-------------|--------------|
| Student contributor | Undergraduate enrolled in a lecture-based course who takes notes and uploads them to the class page | Upload notes quickly, assign them to the right lecture, and see how they compare with the class's notes |
| Student studier | Student preparing for an exam, often with gaps from missed or poorly noted lectures | One trustworthy study guide that fills gaps, shows where sources disagree, and lets them check every claim |
| Instructor (professor or TA) | Teaches the course and has an instructor role on its class page | Endorse accurate notes, see which lectures lack coverage, and see where students' notes conflict |
| Project team (stakeholder) | INFO 5940 team building Margins | A meaningful, demo-ready MVP by the final project deadline |
| Course staff (stakeholder) | INFO 5940 instructors and graders | Evidence that the product uses LLMs meaningfully and was built through AI-DLC |
 
### Business Constraints
 
- **Timeline:** Inception complete by November 24, 2026; working MVP, demo video, and presentation by December 16, 2026.
- **Team:** Four students working part-time alongside other courses.
- **Deployment:** The product runs locally on team laptops only. No hosting or cloud deployment.
- **Course content and sharing:** Some courses restrict sharing of materials. Class pages are opt-in, and professor endorsement gives instructors a visible role in what is shared.
- **Data sources:** Only public course websites can be used; sites behind a login (Canvas and other learning management systems) cannot be fetched.
- **Budget:** No paid services beyond what the team's chosen LLM provider offers within its free or low-cost usage.
### Success Metrics
 
Metrics are measured on a team-built evaluation set: one or two real courses with public websites, about 10 lectures each, and about 30 uploaded notes, with known coverage gaps and conflicts deliberately planted.
 
| Metric | Current State | Target State | Measurement Method |
|--------|---------------|--------------|-------------------|
| Time to a usable study guide for 10 lectures | 1–3 hours of manual merging | Under 3 minutes from clicking "Generate" | Timed runs on the evaluation set |
| Coverage gaps reported | Discovered by accident, often the night before the exam | 100% of planted missing or thin lectures reported | Compare the guide's coverage report with the planted gaps |
| Conflicts flagged | Not detected | At least 4 of 5 planted conflicts flagged, with both versions shown | Compare flagged disagreements with the planted conflicts |
| Citation validity | No citations | 100% of guide sections and chat answers cite at least one note; every citation opens the passage it refers to | Automated check that each citation resolves to a stored passage |
| Grounded refusals | Chatbots answer from general knowledge | At least 4 of 5 questions the notes do not cover are answered with an explicit "not covered in the class notes" | Scripted question set |
| Lecture outline accuracy | Typed in by hand | At least 80% of lectures extracted correctly (date and title) from each demo course site before user correction | Compare extracted outline with the real schedule |
| Endorsement effect | No signal of quality | When an endorsed note and an unendorsed note conflict, the endorsed one is presented first in 100% of cases | Scripted test cases on the evaluation set |
 
---
 
## Full Scope Vision
 
### Product Vision Statement
 
Every lecture-based course has a living, class-built study companion: students contribute notes as the term goes on, instructors endorse what is accurate, and anyone in the class can get a complete, cited picture of what the course covered, where understanding differs, and what is still missing.
 
### Feature Areas
 
#### Feature Area 1: Classes and Membership
- **Description:** Class pages that students and instructors join, with roles.
- **Key Capabilities:**
  - Create an opt-in class page for a course and join it
  - Student and instructor (professor or TA) roles, with role-based permissions
  - University single sign-on and roster import (full vision)
  - Integration with learning management systems such as Canvas (full vision)
- **User Value:** Notes are shared only with the right class, and instructors have a recognized role.
#### Feature Area 2: Course Structure
- **Description:** A lecture-by-lecture outline of the course that notes are mapped to.
- **Key Capabilities:**
  - Fetch the course's public website (syllabus, schedule, linked lecture materials) and extract lectures with number, date, title, and topics
  - Let users review and correct the extracted outline
  - Refresh on request
  - Automatic syncing when the site changes, and support for login-protected sites (full vision)
- **User Value:** Study guides and coverage reports are organized around what the course actually teaches.
#### Feature Area 3: Notes Pooling and Quality Signals
- **Description:** Contributing, organizing, and judging notes.
- **Key Capabilities:**
  - Upload typed notes and have the system suggest which lecture they cover
  - Rate other students' notes; instructors endorse notes
  - Combine endorsements and ratings into a quality weight per note
  - Handwritten and photographed notes via handwriting recognition; lecture audio transcription (full vision)
  - Comments and discussion on notes (full vision)
- **User Value:** The class's notes are in one place, and better notes carry more weight.
#### Feature Area 4: Study Guide Agent
- **Description:** An agent that plans its own steps and calls tools to build a guide.
- **Key Capabilities:**
  - Organize the guide by course topic from the lecture outline
  - Merge overlapping notes into one section per topic
  - Flag disagreements and show each version with its source
  - Report lectures with missing or thin coverage
  - Cite the specific notes behind every section
  - Generated practice questions, flashcards, and spaced repetition (full vision)
- **User Value:** One guide that is complete where the class's notes are complete, honest about conflicts, and clear about gaps.
#### Feature Area 5: Grounded Chat
- **Description:** Question answering over the class's notes and course site.
- **Key Capabilities:**
  - Answer questions using the pooled notes and the fetched course site, weighted by quality
  - Cite the notes and pages used
  - Say explicitly when the class materials do not cover a question
- **User Value:** Quick answers that match this course, not the internet's version of the topic.
#### Feature Area 6: Instructor Insight
- **Description:** A view of the class's collective understanding for professors and TAs.
- **Key Capabilities:**
  - Per-lecture note counts, endorsements, and flagged conflicts
  - Trends in which topics students disagree on most (full vision)
- **User Value:** Instructors see where students are confused or under-served and can respond.
### Integration Points
 
- **Public course websites:** source of the syllabus, schedule, and lecture materials.
- **Hosted LLM provider:** reading, reconciling, writing, and embeddings (provider and models are specified in the Technical Environment Document).
- **University single sign-on and learning management systems (full vision):** login, rosters, and access to protected course pages.
### User Journeys (Full Vision)
 
#### Journey 1: Contributing through the term
1. A student signs in with university credentials; their enrolled courses' class pages are already listed.
2. After each lecture, they upload typed or photographed notes; the system assigns them to the right lecture.
3. Classmates rate the notes and the TA endorses them.
**Outcome:** The student's notes help the whole class, and the class's best notes rise to the top.
#### Journey 2: Studying for a final
1. A student opens the class page a week before the final and selects the lectures on the exam.
2. Margins generates a cited study guide with conflicts and gaps flagged, plus practice questions.
3. The student asks follow-up questions in chat and reviews flashcards in the days before the exam.
**Outcome:** The student studies from a complete, verifiable guide aligned with the course.
#### Journey 3: Instructor check-in
1. A professor opens the instructor view mid-term.
2. They see that two lectures have almost no notes and that students disagree about one key definition.
3. They endorse the most accurate note and clarify the definition in the next lecture.
**Outcome:** The instructor corrects misunderstandings before the exam.
### Scalability and Growth
 
At maturity, Margins would support many courses per university, many universities, and larger classes, with LMS integration replacing manual setup. For this project, it is designed around one course at a time with dozens of notes.
 
### Long-Term Roadmap
 
| Phase | Focus | Timeframe |
|-------|-------|-----------|
| MVP | Class pages, course-site outline, notes upload, ratings and endorsements, cited study guide, coverage report, grounded chat, instructor coverage view | December 16, 2026 |
| Phase 2 | Handwriting recognition, comments, practice questions and flashcards, scheduled site syncing | TBD |
| Phase 3 | University SSO, LMS integration, multi-course and multi-university support, hosting | TBD |
 
---
 
## MVP Scope
 
### MVP Objective
 
Prove that an agent can turn a class's pooled notes and its public course website into a study guide that students can trust, because every claim is cited, conflicts are surfaced rather than hidden, and missing lectures are reported.
 
### MVP Success Criteria
 
- [ ] A student can go from uploading notes to a generated, cited study guide in one session, without help from the team.
- [ ] The guide's coverage report finds every planted missing or thin lecture in the evaluation set.
- [ ] The guide flags at least 4 of 5 planted conflicts and shows both versions with their sources.
- [ ] Every citation in a guide or chat answer opens the passage it refers to.
- [ ] Chat explicitly declines at least 4 of 5 questions the notes do not cover.
- [ ] An instructor's endorsement visibly changes which version the guide and chat present first.
### Features In Scope (MVP)
 
| Feature | Description | Priority | Rationale for Inclusion |
|---------|-------------|----------|------------------------|
| Accounts and roles | Local sign-up and sign-in. Each user is a student or an instructor (professor or TA) on a class page. Only instructors can endorse. | Must Have | Endorsements and permissions depend on roles. |
| Class pages | Create an opt-in class page for a course and join it. Notes, guides, and chat are scoped to one class. | Must Have | Notes must be shared only within the class they belong to. |
| Course outline from public website | A user enters the course's public website URL. The agent fetches and parses the syllabus, schedule, and linked lecture materials and extracts a lecture-by-lecture outline (number, date, title, topics), which users can review and correct. The site is fetched again only when a user asks to refresh. | Must Have | Guides and coverage reports are organized around the course; without the outline there is nothing to compare notes against. |
| Notes upload and lecture assignment | Students upload typed notes (Markdown, plain text, or text-based PDF, up to 5 MB). The system suggests which lecture a note covers, or none if unsure; the uploader confirms or changes it. | Must Have | Pooled notes are the core input. |
| Ratings and endorsements | Students rate classmates' notes. Instructors endorse notes. Endorsements and ratings combine into a quality weight per note, calculated in ordinary code rather than by the LLM. | Must Have | The product's trust model depends on quality signals. |
| Weighted search | Notes are split into passages and searched by meaning. Results combine relevance with the note's quality weight, so endorsed and highly rated notes rank higher. | Must Have | Both the guide and chat depend on finding the right passages. |
| Study guide agent | For a selected class and lecture range, the agent decides which tools to use and in what order (search notes, read the course outline, compare passages, check quality signals, find coverage gaps) and writes a guide organized by course topic that merges overlapping notes, flags disagreements with both versions and sources, cites the notes behind each section, and lists lectures with missing or thin coverage. | Must Have | This is the product's central value. |
| Coverage report | Lectures on the outline with no notes are reported as "missing" and those with fewer than a set number of notes (default 2) as "thin". | Must Have | Telling students what nobody covered is a core promise and is cheap to compute reliably. |
| Grounded chat | Students ask questions about the class. The agent searches the pooled notes and the fetched course site, answers with citations, and says explicitly when the materials do not cover the question. | Must Have | Students need targeted follow-up beyond the guide; this shows the same grounding in an interactive form. |
| Instructor coverage view | Per lecture: number of notes, which are endorsed, and where conflicts were flagged. | Should Have | Gives instructors a reason to engage and endorse; built from data the guide already produces. Can be cut first if time runs short. |
 
### Features Explicitly Out of Scope (MVP)
 
| Feature | Reason for Deferral | Target Phase |
|---------|---------------------|--------------|
| University single sign-on and production-grade authentication | Requires university integration; simple local accounts are enough to demo roles | Phase 3 |
| Integration with Canvas or other systems that require login | Requires credentials and institutional approval; we demo with public course websites | Phase 3 |
| Handwriting recognition, photographed or scanned notes | Adds an unreliable extraction step before the core pipeline; typed notes prove the concept | Phase 2 |
| Audio or video lecture transcription | Large new input type unrelated to the core reconciliation problem | TBD |
| Continuous or scheduled syncing of course websites | Fetching on add and on request covers the demo | Phase 2 |
| Comments and discussion threads | Adds web-development work without strengthening the core product | Phase 2 |
| Following users | Social feature unrelated to studying from notes | TBD |
| Notifications and email | Not needed for a local demo | TBD |
| Real-time co-editing of notes | High complexity; notes are uploaded as finished documents | TBD |
| Practice questions, flashcards, spaced repetition | Valuable, but builds on a trustworthy guide, which must come first | Phase 2 |
| Study guides spanning multiple courses, cross-course search | MVP is designed around one class at a time | Phase 3 |
| Mobile apps | Web app on laptops is enough for the demo | TBD |
| Hosting and cloud deployment | The product runs locally on team laptops | Phase 3 |
| Plagiarism or academic-integrity checks on uploads | Policy-heavy and outside the core problem | TBD |
| Training or fine-tuning models | Not needed; prompting and retrieval are sufficient | TBD |
 
### MVP User Journeys
 
#### Journey 1: Set up a class
1. A student or instructor signs up, creates a class page for their course, and shares it with classmates, who join.
2. They enter the course's public website URL.
3. Margins fetches the site and shows the extracted lecture outline; the user corrects any wrong dates or titles.
**Outcome:** The class page has an accurate lecture outline ready for notes.
**Limitation vs Full Vision:** Local accounts instead of university sign-in; no roster import; public sites only; refresh is manual.
#### Journey 2: Contribute notes
1. A student uploads a Markdown or PDF file of their notes.
2. Margins suggests the lecture it covers; the student confirms or picks another.
3. Other students rate the note, and the TA endorses it.
**Outcome:** The note is in the class pool with a quality weight.
**Limitation vs Full Vision:** Typed notes only; no comments.
#### Journey 3: Generate a study guide
1. A student selects the lectures on an upcoming exam and clicks "Generate study guide".
2. The agent searches the notes, compares them against the outline, and writes the guide.
3. The student reads merged sections, reviews flagged disagreements with both versions shown, sees which lectures are missing or thin, and clicks citations to check the source passages.
**Outcome:** The student has one cited study guide for the exam.
**Limitation vs Full Vision:** No practice questions or flashcards.
#### Journey 4: Ask a question
1. A student asks in chat, "What's the difference between bias and variance in this course?"
2. The agent searches the notes and course site and answers with citations, preferring endorsed notes.
3. For a question the class materials do not cover, the agent says so instead of answering from general knowledge.
**Outcome:** The student gets a course-specific answer they can verify.
**Limitation vs Full Vision:** None significant.
#### Journey 5: Instructor check-in
1. A TA signs in with the instructor role and opens the coverage view.
2. They see note counts, endorsements, and flagged conflicts per lecture.
3. They endorse the most accurate note on a disputed topic.
**Outcome:** The next guide and chat answers prefer the endorsed version.
**Limitation vs Full Vision:** No trend analysis across the term.
### MVP Constraints and Assumptions
 
- **Assumption:** Students already take typed notes or can export them as Markdown, text, or text-based PDF. **Risk if wrong:** Few notes can be uploaded and guides are thin.
- **Assumption:** The demo courses' public websites list a schedule clearly enough to extract lectures. **Risk if wrong:** Users must type or correct much of the outline by hand.
- **Assumption:** The chosen LLM can reliably call tools and follow citation instructions. **Risk if wrong:** Guides lose citations or skip steps; the agent loop needs more guardrails.
- **Assumption:** Students are willing to share notes with their class. **Risk if wrong:** The pool is too small to be useful.
- **Accepted Limitation:** One class at a time, with dozens of notes, not hundreds.
- **Accepted Limitation:** Accounts are local and suitable for a demo, not for real-world deployment.
- **Accepted Limitation:** The agent can mistake rewording for disagreement; flagged conflicts show both versions so the student can judge.
- **Accepted Limitation:** Notes and course-site text are sent to a hosted LLM provider for processing.
### MVP Definition of Done
 
- [ ] All "Must Have" features implemented and tested
- [ ] Success criteria met on the team's evaluation set
- [ ] Automated tests pass, with the coverage target set in the Technical Environment Document
- [ ] Every teammate can start the app locally from the README instructions
- [ ] No API keys or other secrets committed to the repository
- [ ] Demo video and recorded presentation completed for December 16, 2026
- [ ] All features built through the AI-DLC Construction phase, with the record in `aidlc-docs/`
---
 
## Risks and Dependencies
 
### Key Risks
 
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| The agent misses steps, calls tools incorrectly, or drops citations | Medium | High | Keep the tool set small and well described; validate every citation against stored passages in code; cap the number of agent steps |
| False conflicts: rewording flagged as disagreement | High | Medium | Show both versions so students can judge; tune on planted examples in the evaluation set |
| Course websites vary widely in structure | High | Medium | Let users review and correct the outline; demo with one or two well-structured public sites |
| LLM provider rate limits or quota reached during development or the demo | Medium | High | Cache embeddings and fetched sites; keep the evaluation set small; test the full demo early |
| Not enough real notes to show conflicts and gaps | Medium | Medium | Build an evaluation set from team members' own notes, with conflicts and gaps planted deliberately |
| Prompt injection: note or website text tries to instruct the agent | Low | Medium | Treat note and site text as content, never as instructions; keep permission checks in ordinary code |
| Course materials that may not be shared | Low | Medium | Class pages are opt-in; demo only with courses whose materials are public |
| Scope too large for the time available | Medium | High | Instructor coverage view is "Should Have" and is cut first; out-of-scope list is enforced |
 
### External Dependencies
 
- Hosted LLM provider API (chat with tool calling, and embeddings) - provider - required; specified in the Technical Environment Document
- Public course websites for the demo courses - course instructors - to be chosen by the team
- Team members' own lecture notes for the evaluation set - project team - to be collected before Construction
### Open Questions
 
- [ ] **Weighting formula.** How should instructor endorsements and student ratings combine into a quality weight? Should an endorsement dominate or just boost? How do we keep a few early ratings from skewing results?
- [ ] **Conflict detection.** How does the agent decide that two passages disagree rather than simply phrase things differently, and how much do we tolerate false alarms?
- [ ] **Model reliability and limits.** Can the chosen hosted model call tools reliably and produce well-cited output within its free or low-cost usage limits? What do we do if we hit rate limits during the demo?
- [ ] **Sending notes to a provider.** Students' notes and course-site text are sent to an external LLM API. Do users need to be told, or asked to agree, when they join a class page?
- [ ] **Lecture assignment.** How should the system handle notes that span several lectures or are not tied to any lecture?
- [ ] **Citation granularity.** Should citations point to a note, a passage, or a line range, and how should they be displayed so students can verify quickly?
- [ ] **No endorsements.** In practice, instructors may not endorse anything. How should the product behave when no endorsements exist?
- [ ] **Privacy and sharing.** Should uploaded notes be visible to every member of the class page by default, and can uploaders withhold their name?
- [ ] **Course site variety.** How should the agent handle a site where it cannot find a clear schedule, and how much should users have to correct by hand?
- [ ] **Course site vs. notes in chat.** When the course website and students' notes disagree, which does chat present first, and how is the disagreement shown?
- [ ] **Evaluation.** Is the planned evaluation set (one or two courses, about 30 notes, planted gaps and conflicts) large enough to judge guide quality?
 
