# Margins: Requirements

## 1. Intent Analysis

| Item | Value |
|---|---|
| **User request** | "Using AI-DLC, start the Inception phase for our product Margins. Read README.md, Vision.md and our tech environment doc first. They are our input documents." |
| **Request type** | New project (greenfield) |
| **Scope** | System-wide: web app, accounts and roles, course-site ingestion, notes ingestion, weighting, retrieval, two LLM agents, instructor views |
| **Complexity** | Complex (multiple personas, agentic LLM features, LLM output that must stay grounded) |
| **Requirements depth** | Comprehensive (traceability to inputs and answers) |
| **Inputs** | `README.md`, `Vision.md`, `TechnicalEnvironment.md`, `requirement-verification-questions.md` (Q1-Q23), `requirement-clarification-questions.md` (CQ1-CQ12) |

**Summary.** Margins is a locally run web app where students in one course pool their typed lecture notes. Instructors also upload authoritative course materials. A Gemini-backed agent builds cited study guides and answers cited chat questions from that pool, and checks it against the public course website. The agent flags disagreements between sources and reports lectures with no coverage. Source precedence is: professor material, then endorsed notes, then the majority of notes, then student "matches slides" flags, then upvotes.

### Traceability Key
- **V-n**: Vision.md section 2, MVP feature n
- **Qn**: answer to requirement-verification-questions.md
- **CQn**: answer to requirement-clarification-questions.md
- **TE-n**: TechnicalEnvironment.md section n

Where a clarification answer replaces an earlier answer, the clarification wins (for example CQ6 replaces Q6).

---

## 2. Users and Roles

| Role | How the role is obtained | Key permissions |
|---|---|---|
| **Student** | Self sign-up (email and password) | Join courses by code, upload notes, upvote, flag "matches slides", request courses, generate guides, chat |
| **Instructor (professor)** | Account **seeded by the team** through a setup script or admin command from a small list, for example names copied from the demo courses' public pages (CQ4) | Everything a student can do, plus: create courses, upload professor material, endorse notes, view coverage, run "analyze course", resolve conflicts, act on course requests |
| **TA** | Joins a course with the course's **instructor join code** (Q2) | Instructor permissions **within that course only** |

**Note.** "Instructor" is a global account flag (seeded) that allows creating courses. "Course instructor" is a per-course membership role that instructor-flagged accounts and TAs hold. Both are checked in FastAPI dependencies (TE-6).

---

## 3. Functional Requirements

### FR-1 Accounts and Authentication (V-1, TE-6)
- **FR-1.1** Users sign up with email, display name and password. Sign-up always creates a student-level account.
- **FR-1.2** Passwords are stored only as bcrypt hashes. A login sets a signed session cookie, and users can log out.
- **FR-1.3** An admin CLI command or setup script creates instructor-flagged accounts from a list file. There is no UI path to gain the instructor flag (CQ4).
- **FR-1.4** Unauthenticated users can only reach the sign-up and login pages.

### FR-2 Courses and Membership (V-1, Q2, Q3, Q4, CQ5)
- **FR-2.1** Only instructor-flagged users can create a course. A course has a name, a code (for example "INFO 5940"), a term, and an optional public course website URL. Its creator becomes a course instructor.
- **FR-2.2** Each course has two join codes, both visible to course instructors:
  - a **student join code**, which gives student membership (Q3)
  - an **instructor join code**, which gives TA/course-instructor membership (Q2)
- **FR-2.3** Course instructors can regenerate either join code.
- **FR-2.4** A user can belong to several courses. Every feature works inside one course at a time; nothing searches across courses.
- **FR-2.5** **Course requests (CQ5):**
  - Any signed-in user can submit a request with course name, course website URL and instructor email.
  - Requests appear on a "Requested courses" list visible to all instructor-flagged users.
  - An instructor can create a course from a request (fields prefilled) or dismiss it. Either action marks the request resolved.
  - No email or notification is sent.

### FR-3 Course Structure from the Public Website (V-2, Q5, CQ6, CQ11, TE-6)
- **FR-3.1** When a course has a website URL, the system fetches the entered page plus same-site pages it links to, one level deep, with a cap on page count. It also downloads linked PDFs (lecture slides, handouts) and extracts their text (Q5).
- **FR-3.2** Fetching accepts only public http/https URLs, uses timeouts and per-response size caps, and refuses private, loopback and link-local addresses (TE-6).
- **FR-3.3** The agent extracts a **lecture outline** (number, date, title, topics) and, if the site lists them, **exams** (name, date, lecture range) (CQ11).
- **FR-3.4** The site is fetched when the URL is first added and again only when a course instructor clicks **Refresh** (V-2). Refresh replaces the outline and exam list.
- **FR-3.5** **No schedule found (CQ6: A):**
  - The course page shows a clear "No lecture schedule was found on the course website" message with a short explanation.
  - The course runs without lectures. Guides are organized by topic only, and coverage reporting is disabled.
  - The system does not infer a schedule from notes.
- **FR-3.6** The outline cannot be edited by hand. The only way to change it is to refresh from the website (CQ6: A; this deliberately departs from Vision.md feature 2's "review and correct").
- **FR-3.7** The text of fetched website pages and PDFs becomes searchable **website sources**. These rank like ordinary student notes, not as authoritative material (CQ2: B). The guide cites them as "Course website: <page or file>".

### FR-4 Notes and Professor Material Upload (V-3, Q7, Q8, Q9, CQ2, CQ7, TE-6)
- **FR-4.1** Course members can upload typed notes as `.md`, `.txt` or text-based `.pdf`, up to 5 MB each, with a title. Other types and oversize files are rejected with a clear message (TE-6).
- **FR-4.2** **Professor material (CQ2: B):**
  - Course instructors can upload course materials (slides, handouts, their own notes) through the same flow.
  - These are stored with source type `professor_material`, labeled "Instructor material" in the UI, and are authoritative (see FR-6).
- **FR-4.3** **Lecture tagging is optional (CQ7: C):**
  - After upload, the system suggests a lecture (LLM-backed; no suggestion if unsure).
  - The uploader can accept it, choose one or more lectures, or leave the note untagged.
  - Tags can be changed later by the uploader.
- **FR-4.4** Untagged notes are still used as general sources, retrieved by relevance. They **do not count** toward per-lecture coverage (CQ7: C).
- **FR-4.5** **Visibility and anonymity (Q8: B):**
  - All course members can see all notes.
  - At upload the uploader chooses to show their display name or appear as "Anonymous classmate". They can change this choice later.
  - Course instructors always see the real name.
- **FR-4.6** **Delete and replace (Q9: C):**
  - An uploader can delete their own note. This removes its passages, upvotes, flags and endorsement. Saved guide versions that cited it show the citation as "(removed note)".
  - An uploader can replace the file with a new version. The note keeps its ID, ratings and endorsement; its passages are re-indexed.
  - Each note shows a "Last updated" timestamp (Q9).
- **FR-4.7** Notes are split into passages and each passage is embedded with `gemini-embedding-001`. Embeddings are stored in SQLite (TE-2, TE-3).

### FR-5 Ratings, Flags and Endorsements (V-4, Q10, Q12)
- **FR-5.1** A course member can **upvote** another member's note at most once and can remove the upvote. Users cannot upvote their own notes. There are no downvotes (Q10: C).
- **FR-5.2** A student can flag another member's note as **"matches the posted lecture slides"** at most once, and can remove the flag. It is a weaker signal than an endorsement (Q12: C).
- **FR-5.3** A course instructor can **endorse** a note and remove the endorsement. Endorsed notes show an "Instructor-endorsed" badge.
- **FR-5.4** If a course has no endorsements, the product works on the other signals. Guides note that no instructor has endorsed any source (Q12).

### FR-6 Quality Weight and Source Precedence (V-4, V-5, Q11, CQ8, CQ9)
- **FR-6.1** **Precedence** when sources disagree, highest first (CQ9: B):
  1. Professor material
  2. Instructor-endorsed note
  3. Majority of notes (cross-source agreement)
  4. "Matches slides" flagged note
  5. Upvotes
- **FR-6.2** Each note's **quality weight** is computed by **deterministic code** in one configurable module (Q11: B). Its inputs are:
  - a smoothed upvote score (Bayesian average, so a few early votes cannot swing it)
  - a "matches slides" bonus
  - an endorsement bonus
  - an **agreement score** (CQ8: B; see FR-6.3)
  - a fixed top weight for professor material
- **FR-6.3** **Agreement score (CQ8: B):**
  - Whenever conflict analysis runs (guide generation or "analyze course"), it records, per compared claim, which notes agreed with the majority version and which did not.
  - A note's agreement score is a smoothed ratio of majority-agreements to comparisons, computed deterministically from these stored records.
  - It changes over time as more analyses run.
- **FR-6.4** The weight formula and its constants live in one place and are documented in plain language on a "How sources are ranked" help page (Q11: D spirit). The exact constants are set in Functional Design.
- **FR-6.5** Retrieval ranks passages by a combination of semantic relevance (cosine similarity computed in numpy) and the source's quality weight (V-5).

### FR-7 Study Guide Generation, Agentic (V-6, Q13, Q14, Q15, Q18, CQ3, CQ10, CQ11)
- **FR-7.1** A course member chooses a scope:
  - an **exam**, when exams were extracted, which selects that exam's lecture range
  - a **custom lecture range**
  - **whole course / by topic**, when there is no schedule
- **FR-7.2** The agent is a function-calling loop written by the team on `google-genai` with `gemini-2.5-flash`, with no LangChain or LlamaIndex (TE-3, TE-5).
  - It chooses which tools to call, in what order, and when it has enough evidence.
  - Planned tools: `get_course_outline`, `list_sources_for_lecture`, `retrieve_passages`, `compare_passages`, `find_coverage_gaps`, `get_source_weights`, `cite_source`.
  - The loop has a hard cap on steps and tool calls.
- **FR-7.3** The guide is organized by topic, or by lecture then topic when a schedule exists. It:
  - **merges** overlapping content into one section per topic
  - **cites** the source of every claim at **note level** (note title and uploader display or "Anonymous classmate"). The citation links to the full note (Q14: A).
  - **flags disagreements** (FR-8)
  - **reports coverage gaps**: lectures in scope with no tagged notes ("missing") or fewer than a threshold ("thin"). This is skipped when the course has no schedule (FR-3.5).
  - states when no endorsements exist (FR-5.4)
- **FR-7.4** **Professor material vs. student notes (CQ3: C):** when they differ, the guide shows both versions as a disagreement and marks the professor version **"Authoritative (instructor material)"**.
- **FR-7.5** Generation shows step-by-step progress (for example "Retrieving notes for Lecture 3...") and should finish within about 2 minutes at demo scale (Q18: B).
- **FR-7.6** **Saved guides with version history (Q15: C, CQ11: A):**
  - Every generated guide is saved per course and scope, and is visible to all course members.
  - Guides are grouped by exam when they have an exam scope.
  - Regenerating creates a new version. Users can view earlier versions and compare two versions.
- **FR-7.7** Guides render Markdown with markdown-it-py with raw HTML disabled, and Jinja2 autoescaping stays on. `|safe` is never applied to notes or model output (TE-5).

### FR-8 Conflict Detection and Resolution (Q13, Q17, CQ10)
- **FR-8.1** The agent compares retrieved passages on the same topic. It flags a conflict only for **factual** differences (definitions, formulas, numbers, claims), not wording (Q13: A).
- **FR-8.2** A conflict records:
  - the topic and lecture (if any)
  - each differing version with its source(s)
  - the number of sources supporting each version
  - which version is the **majority** (Q13: "combination of majority and LLM")
- **FR-8.3** In the guide, students see every version of a conflict. The majority version is labeled "More commonly reported", and professor material is labeled per FR-7.4 (CQ10: B).
- **FR-8.4** Conflicts are stored and shown to course instructors in the coverage view (FR-9) (CQ10: B). There are no notifications or email.
- **FR-8.5** **Resolution (CQ10: B):**
  - A course instructor can resolve a conflict by choosing the correct version, which can also endorse the note behind it.
  - Resolved conflicts are used in later guide generations: the chosen version is presented as resolved by the instructor.
- **FR-8.6** Conflict records feed the agreement score (FR-6.3).

### FR-9 Instructor Coverage View and Course Analysis (V-8, Q17)
- **FR-9.1** Course instructors see, per lecture:
  - the number of tagged notes
  - how many are endorsed
  - whether professor material exists
  - the number of open and resolved conflicts
  - missing or thin status
- **FR-9.2** An **"Analyze course"** action (instructor only) runs conflict detection per lecture without producing a guide. It refreshes the conflicts in the view (Q17: B). It shows progress the same way as FR-7.5.
- **FR-9.3** When the course has no schedule, the view shows the "no schedule" limitation instead of per-lecture rows (FR-3.5).
- **FR-9.4** The view lists the highest-weighted unendorsed notes as candidates for review.

### FR-10 Cited Chat, Agentic (V-7, Q16)
- **FR-10.1** Course members can ask questions in a per-course chat. The agent can call retrieval and course-outline tools, applying the precedence rules in FR-6.1.
- **FR-10.2** Every answer cites its sources at note level (website sources as "Course website: ..."). When the sources do not support an answer, the agent says so plainly and does not answer from general knowledge.
- **FR-10.3** Chat history is saved per user per course and is visible only to that user (Q16: B). Users can start a new conversation.
- **FR-10.4** The rendering rules in FR-7.7 apply.

---

## 4. Non-Functional Requirements

### NFR-1 Technology Constraints (TE-1 to TE-5, CQ1)
- Python 3.12+, managed with uv, configured in `pyproject.toml`, with `uv.lock` committed
- FastAPI and Pydantic v2; Jinja2 and HTMX; SQLModel and SQLite (no other datastore)
- **LLM provider: Google Gemini only** (`gemini-2.5-flash`, `gemini-embedding-001`) through `google-genai` (CQ1: A)
- Runs locally only, with `uv run fastapi dev app/main.py` on `127.0.0.1:8000`
- Not used: LangChain/LlamaIndex, other LLM providers, ChromaDB/Postgres, Flask/Django/Streamlit, React/npm, `eval`/`exec`/`pickle` on user or model content

### NFR-2 Security Essentials (TE-6, CQ12: A)
The Security Baseline extension is **not** enforced (Q21: B). These essentials are still **required**:
- bcrypt password hashes, a signed session cookie, and role checks in FastAPI dependencies
- Pydantic type and length limits on every input field
- an upload allowlist (`.md`, `.txt`, `.pdf`) and a 5 MB cap
- safe fetching: public http/https only, timeouts, size caps, no private or loopback addresses
- note and website text passed to the model as **content, never as instructions**; prompts separate the system instructions from the quoted source text
- no `|safe` on user or model content; Markdown rendered with raw HTML disabled
- `GEMINI_API_KEY` and `SESSION_SECRET` kept in a git-ignored `.env`, with only `.env.example` committed and secrets never logged

### NFR-3 Performance and Scale (Q18, Q19)
- Designed and tested for **demo scale**: 1-2 courses, about 10-30 users, about 25 lectures and up to about 200 notes per course (Q19: A).
- Study guide generation and "Analyze course" should finish within about 2 minutes, with streamed or polled step-by-step progress (Q18: B).
- Chat answers should normally return within about 20 seconds.
- Ordinary page loads (lists, uploads without the lecture suggestion) should respond in under 1 second locally.

### NFR-4 Reliability of LLM Features
- Gemini errors, timeouts and rate limits are caught and shown to the user as a clear retryable message; the app does not crash.
- The agent loop has a maximum number of steps and tool calls.
- Malformed tool-call arguments from the model are validated with Pydantic and returned to the model as tool errors.
- Guides and chat answers with no citations are treated as failures, not shown as grounded output.
- The Resiliency Baseline extension is **not** enforced (Q22: B).

### NFR-5 Testability and Quality (TE-4, Q20, Q23)
- pytest, with pytest-cov at **80% or more on business logic**, and ruff lint checks clean
- Deterministic logic (weights, agreement score, coverage gaps, permissions, chunking, URL validation, exam-range selection) stays in plain functions that can be tested without the LLM.
- **Property-based testing is enforced in full (PBT-01 to PBT-10)** (Q23: A). This adds **Hypothesis** to the dev dependencies, which TechnicalEnvironment.md does not list yet. Expected property targets:
  - weight monotonicity and bounds
  - chunking round-trip and coverage of the source text
  - coverage-gap invariants
  - smoothing behavior
  - permission rules
  - serialization of agent tool arguments and results
- The regular pytest suite does **not** call the live Gemini API. The model client sits behind a small interface so tests can use fakes.
- **Evaluation script (Q20: B):**
  - an on-demand script, outside the regular pytest suite, runs the live agent on a planted test set: real notes with known gaps, conflicts, and a professor-material contradiction
  - it reports whether each planted item was found

### NFR-6 Usability
- Server-rendered pages with HTMX, usable on a laptop browser
- Basic accessibility: semantic HTML, labeled form fields, keyboard-reachable controls, and badges that don't rely on color alone
- Citations and conflict versions are visibly linked to their sources so a student can check a claim in one click.

### NFR-7 Maintainability and Privacy
- Clear separation between routes, services (business logic), the agent and its tools, and the model client
- Anonymity is enforced server-side: student-facing templates never receive the real name of an anonymous uploader.
- Course data is visible only to that course's members.

---

## 5. Decisions That Change the Input Documents

| # | Decision | Source | Documents to update |
|---|---|---|---|
| D1 | The LLM provider is Gemini, not Ollama | CQ1: A | README.md (Planned LLM Integration), Vision.md (Planned models, open question 3) |
| D2 | Instructors can upload authoritative professor material | Q1, CQ2: B | Vision.md features 3 and 4 |
| D3 | Instructor accounts are seeded by the team; no self-declared instructor role | CQ4: B | Vision.md feature 1 |
| D4 | The lecture outline cannot be edited by hand; if no schedule is found, there is no lecture structure | CQ6: A | Vision.md feature 2 |
| D5 | A course-request list is added | Q4, CQ5: B | Vision.md section 2 |
| D6 | Cross-source agreement feeds the stored note weight | CQ8: B | Vision.md feature 4 |
| D7 | Exams are extracted and guides can be scoped by exam | CQ11: A | Vision.md features 2 and 6 |
| D8 | Instructors can resolve conflicts | CQ10: B | Vision.md feature 8 |
| D9 | Hypothesis is added for property-based testing | Q23: A | TechnicalEnvironment.md section 4 |

---

## 6. Out of Scope (confirmed from Vision.md section 3, plus decisions)
- University SSO, production-grade auth, and automatic verification of instructor identity (CQ4)
- OCR of handwritten or scanned notes; audio or video transcription
- Login-protected course sites (Canvas and other LMSs)
- Scheduled or continuous site syncing; inferring a schedule from notes (CQ6)
- Editing the lecture outline by hand (CQ6)
- Guides across courses or search across courses
- Quizzes, flashcards, spaced repetition
- Real-time co-editing; comments and threads; following users
- Notifications, email, mobile apps
- Cloud deployment
- Plagiarism and academic-integrity checks
- Fine-tuning models
- Downvotes (Q10)

---

## 7. Assumptions
- **A1.** Demo course websites are public and server-rendered. Pages that need JavaScript to show their content may yield no schedule (handled by FR-3.5).
- **A2.** One team member's Gemini API key is used for local runs; the free or paid tier limits are acceptable for demo scale.
- **A3.** "Majority" means more distinct sources support one version than any other. A tie is shown as a tie with no majority label.
- **A4.** A website PDF that is also uploaded by an instructor counts once, as professor material.

---

## 8. Success Criteria (from Vision.md section 1)
- A student uploads notes, generates a guide for an exam or lecture range, and every statement links to a source note.
- The guide clearly separates agreed, disputed (with majority and authoritative labels) and missing content.
- An instructor's endorsement, uploaded material or conflict resolution visibly changes what the next guide says or prioritizes.
- The evaluation script finds the planted gaps and conflicts in the test set.

---

## 9. Extension Compliance Summary (Requirements Analysis stage)

| Extension | Status | Notes |
|---|---|---|
| Security Baseline | Disabled (Q21: B) | Skipped and logged. TE-6 essentials are kept as NFR-2 (CQ12: A). |
| Resiliency Baseline | Disabled (Q22: B) | Skipped and logged. |
| Property-Based Testing | Enabled, full | **PBT-01 to PBT-08 and PBT-10: N/A at this stage.** They apply in Functional Design and Code Generation; candidate properties are recorded in NFR-5. **PBT-09 (framework selection): compliant.** Hypothesis is chosen for Python (NFR-5, D9). |
