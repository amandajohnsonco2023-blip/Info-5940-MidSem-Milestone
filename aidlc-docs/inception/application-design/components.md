# Margins: Components

**Stage**: INCEPTION - Application Design
**Inputs**: `requirements.md`, `stories.md`, `application-design-plan.md` (answers Q1-Q14)

This document lists each component, its purpose, its responsibilities and its interface. Method signatures are in `component-methods.md`. Detailed business rules (weight constants, smoothing, thresholds, prompts) are left to Functional Design.

---

## 1. Code Organization (Q1: C, hybrid)

Feature packages hold the web-facing domains: each has its own routes, service(s), SQLModel tables and templates. Shared core packages hold machinery that several features use.

```
app/
  main.py              # FastAPI app, router registration, startup
  config.py            # Settings from .env (GEMINI_API_KEY, SESSION_SECRET, caps)
  db.py                # engine, get_session dependency, background session factory
  templating.py        # Jinja2 environment, Markdown filter (raw HTML off)
  web/                 # C1: auth dependencies, session cookie, error pages, layout
  accounts/            # C2: users, sign-up, login, seed command
  courses/             # C3: courses, memberships, join codes, course requests,
                       #     lectures and exams (C4 owns their import)
  sources/             # C5: notes, professor material, website sources, tags
  trust/               # C7: upvotes, flags, endorsements, agreement, weights
  conflicts/           # C11: conflicts and resolutions
  guides/              # C12: study guides and versions
  coverage/            # C13: coverage view and course analysis
  chat/                # C14: chat threads and messages
  jobs/                # C15: background jobs and progress events
  llm/                 # C9: model client protocol, Gemini client, fake client   (shared core)
  agent/               # C10: agent loop, tool registry, tools, profiles, citations (shared core)
  ingestion/           # C6 + C4 pipeline: file parsing, chunking, embedding,
                       #     safe fetching, site crawling, structure extraction   (shared core)
  retrieval/           # C8: similarity search and ranking                       (shared core)
scripts/
  evaluate.py          # C16: on-demand evaluation on the planted test set
tests/                 # mirrors app/ (unit, property-based, integration with fakes)
```

**Layering inside a feature package (Q8: A):** routes handle HTTP only (form parsing, auth dependencies, choosing templates). Services hold all business logic and permission re-checks and receive a SQLModel `Session`. There is no repository layer.

---

## 2. Component Summary

| ID | Component | Package | Kind | Main stories |
|---|---|---|---|---|
| C1 | Web Shell and Auth | `app/web/`, `main.py`, `templating.py` | Infrastructure | US-1.2, all pages |
| C2 | Accounts | `app/accounts/` | Feature | US-1.1, US-1.2, US-1.3 |
| C3 | Courses and Membership | `app/courses/` | Feature | US-2.1 to US-2.6, US-3.5 |
| C4 | Course Site Import | `app/ingestion/site/`, `app/courses/` | Pipeline | US-3.1 to US-3.4 |
| C5 | Sources | `app/sources/` | Feature | US-4.1 to US-4.7 |
| C6 | Ingestion Pipeline | `app/ingestion/` | Shared core | US-4.1, US-4.6, US-4.7, US-3.1 |
| C7 | Trust and Weighting | `app/trust/` | Feature + pure logic | US-5.1 to US-5.3, US-6.1, US-6.2 |
| C8 | Retrieval | `app/retrieval/` | Shared core | US-7.1, US-10.1 |
| C9 | Model Client | `app/llm/` | Shared core | all LLM features |
| C10 | Agent Core | `app/agent/` | Shared core | US-7.1, US-7.2, US-9.2, US-10.1, US-10.2 |
| C11 | Conflicts | `app/conflicts/` | Feature + pure logic | US-8.1, US-8.2, US-6.2 |
| C12 | Study Guides | `app/guides/` | Feature | US-7.1 to US-7.5 |
| C13 | Coverage and Course Analysis | `app/coverage/` | Feature + pure logic | US-9.1 to US-9.3, US-7.3 |
| C14 | Cited Chat | `app/chat/` | Feature | US-10.1 to US-10.3 |
| C15 | Jobs and Progress | `app/jobs/` | Infrastructure | US-3.1, US-4.1, US-7.1, US-9.2 |
| C16 | Evaluation Harness | `scripts/evaluate.py` | Tooling | NFR-5 evaluation script |

---

## 3. Component Definitions

### C1: Web Shell and Auth
**Purpose:** The FastAPI application frame: app startup, router registration, sessions, auth dependencies, layout templates and safe rendering.

**Responsibilities**
- Create the FastAPI app, register every feature router, and create tables on startup.
- Issue and read the signed session cookie (`SESSION_SECRET`).
- Provide auth dependencies: `require_user`, `require_instructor_flag`, `require_member(course_id)`, `require_course_instructor(course_id)` (FR-1.4, FR-2, Q13: A).
- Render Markdown with markdown-it-py, raw HTML disabled; keep Jinja2 autoescaping on; never use `|safe` on user or model content (FR-7.7, NFR-2).
- Shared layout, navigation ("My courses"), flash messages and error pages; convert service exceptions (not found, forbidden, validation) to HTTP responses.

**Interface:** FastAPI dependencies and a `render_markdown` Jinja2 filter, used by every feature router.

---

### C2: Accounts
**Purpose:** User accounts and authentication.

**Responsibilities**
- Sign-up (always a student-level account), login with one generic error message, logout (FR-1.1, FR-1.2).
- bcrypt password hashing and verification.
- Admin CLI command that seeds instructor-flagged accounts from a list file; no UI path to the instructor flag (FR-1.3).

**Interface:** `AccountService`; CLI entry point `seed-instructors`.
**Owns entities:** `User`.

---

### C3: Courses and Membership
**Purpose:** Courses, who belongs to them in which role, and the course structure (lectures, exams) that C4 imports.

**Responsibilities**
- Create courses (instructor flag only); creator becomes course instructor (FR-2.1).
- Generate, show and regenerate the student and instructor join codes (FR-2.2, FR-2.3).
- Join by code, with the role the code grants (student or TA/course instructor).
- List a user's courses and their role in each; resolve a user's role in a course for the auth dependencies.
- Course requests: submit, list for instructor-flagged users, create-from-request (prefilled), dismiss (FR-2.5).
- Store and serve the lecture outline and exam list; expose "has schedule" for features that depend on it (FR-3.5).
- Exam-range selection: turn an exam or custom range into a list of lectures (FR-7.1).

**Interface:** `CourseService`, `MembershipService`, `CourseRequestService`, `CourseStructureService`.
**Owns entities:** `Course`, `Membership`, `CourseRequest`, `Lecture`, `Exam`.

---

### C4: Course Site Import (Q3: A, fixed pipeline)
**Purpose:** Turn a public course website into a lecture outline, an exam list and searchable website sources.

**Responsibilities**
- Safe fetching: public http/https only; refuse private, loopback and link-local addresses (checked after DNS resolution and on redirects); timeouts; per-response size caps (FR-3.2, NFR-2).
- Crawl the entered page plus same-site links one level deep, with a page cap; download linked PDFs (FR-3.1).
- Extract readable text from HTML and PDFs.
- Run one or two `generate_structured` calls to extract the outline (number, date, title, topics) and exams (name, date, lecture range) into Pydantic models (FR-3.3).
- On refresh, replace the outline, exams and website sources (FR-3.4). If no schedule is found, record that state (FR-3.5).
- Hand fetched page and PDF text to C5/C6 as `website` sources (FR-3.7).
- Runs as a background job (C15) because it can take a while.

**Interface:** `SiteImportService.import_site(course_id, job_id)`; internal helpers `SafeFetcher`, `SiteCrawler`, `TextExtractor`, `CourseStructureExtractor`.
**Owns entities:** none of its own; writes `Lecture`/`Exam` through C3 and website `Source`s through C5.

---

### C5: Sources
**Purpose:** Everything that can be cited: student notes, professor material and website sources, plus lecture tagging and anonymity.

**Responsibilities**
- Upload with title, type allowlist (`.md`, `.txt`, `.pdf`) and 5 MB cap; store the file; start a note-processing job (Q11: C) (FR-4.1).
- Professor material uploads (course instructors only), labeled "Instructor material" (FR-4.2).
- Show and accept the lecture suggestion; tag to one or more lectures, or leave untagged; retag later (FR-4.3, FR-4.4).
- Anonymity choice at upload, changeable later (FR-4.5).
- Delete (cascade passages, upvotes, flags, endorsement; cited guides show "(removed note)") and replace (keep ID, ratings, endorsement; re-index) (FR-4.6).
- "Last updated" timestamps.
- **Public view presenter (Q13: A):** the only function that turns a source into what a viewer may see. It shows "Anonymous classmate" instead of the uploader's name unless the viewer is a course instructor. Templates and agent tools only receive this view (NFR-7).

**Interface:** `SourceService`, `present_source(source, viewer_role)`.
**Owns entities:** `Source` (type `student_note` | `professor_material` | `website`), `SourceLectureTag`.

---

### C6: Ingestion Pipeline
**Purpose:** Turn a source's file or text into indexed passages with embeddings.

**Responsibilities**
- Parse `.md`, `.txt` and text-based `.pdf` into plain text; reject PDFs with no extractable text with a clear message.
- Split text into passages (chunking).
- Embed passages through C9 (`gemini-embedding-001`) and store vectors in SQLite.
- Re-index on replace (delete old passages, create new ones).
- Ask C9 for a lecture suggestion for a new note given the course outline (returns none when unsure).

**Interface:** `IngestionService.process_source(source_id, job_id)`; pure helpers `parse_file`, `chunk_text`.
**Owns entities:** `Passage` (text, position, embedding).

---

### C7: Trust and Weighting
**Purpose:** The signals that say how much to trust a note, and the single place that turns them into a weight.

**Responsibilities**
- Upvotes (one per user per note, not on own notes, removable) (FR-5.1).
- "Matches the posted slides" flags (students, one per user per note, removable) (FR-5.2).
- Endorsements (course instructors, removable, badge) (FR-5.3).
- Agreement records: per compared claim, whether each source agreed with the majority (FR-6.3). Written by C11.
- **Weight formula** in one pure, configurable module: smoothed upvotes, flag bonus, endorsement bonus, smoothed agreement score, fixed top weight for professor material (FR-6.2, FR-6.4).
- Recompute and store a source's weight whenever an input changes (Q10: A).
- Precedence order for disagreements (FR-6.1), exposed as a pure function used by C11.
- Data for the "How sources are ranked" help page (US-6.1).

**Interface:** `TrustService`, `WeightService`; pure module `weighting.py` (`compute_weight`, `smoothed_score`, `precedence_rank`).
**Owns entities:** `Upvote`, `SlidesFlag`, `Endorsement`, `AgreementRecord`; the stored `weight` value on `Source`.

---

### C8: Retrieval
**Purpose:** Find the passages most relevant to a query, ranked by relevance and trust.

**Responsibilities**
- Embed the query through C9.
- Load the course's passage embeddings from SQLite per query (Q12: A), optionally filtered to a lecture set or source types.
- Cosine similarity in numpy; combine with each source's stored weight into one ranking score (FR-6.5).
- Return passages with their source's public view (from C5), never raw uploader names.

**Interface:** `RetrievalService.search(...)`; pure helpers `cosine_similarities`, `combine_scores`.
**Owns entities:** none.

---

### C9: Model Client (Q7: A)
**Purpose:** The only code that talks to Gemini.

**Responsibilities**
- One protocol with three methods: `generate` (chat turn with optional tools), `generate_structured` (Pydantic schema output) and `embed`.
- `GeminiClient` implements it with `google-genai` (`gemini-2.5-flash`, `gemini-embedding-001`), timeouts, retries with backoff on rate limits and transient errors, and one clear `ModelUnavailableError` for everything else (NFR-4).
- `FakeModelClient` with scripted responses for tests; the regular suite never calls live Gemini (NFR-5).
- Never logs the API key or full prompts containing secrets.

**Interface:** `ModelClient` protocol; `ModelTurn`, `ToolCall`, `Message` types.
**Owns entities:** none.

---

### C10: Agent Core (Q2: A, Q4: A, Q6: A)
**Purpose:** One function-calling agent loop used by every agentic feature.

**Responsibilities**
- **Agent loop:** send messages and tool declarations to C9, run the tool calls the model requests, feed results back, and stop at a final answer or at the profile's hard step and tool-call caps (FR-7.2, NFR-4).
- **Tool registry:** each tool is a plain function with Pydantic argument and result models; the registry generates the Gemini function declaration from the argument schema. Arguments are validated, and invalid ones go back to the model as tool errors (NFR-4). The course ID, the viewer and services are injected by the loop, never taken from the model.
- **Tools:** `get_course_outline`, `list_sources_for_lecture`, `retrieve_passages`, `compare_passages`, `find_coverage_gaps`, `get_source_weights`, `get_resolved_conflicts` (added for FR-8.5). `cite_source` is replaced by citation markers (Q4: A).
- **Agent profiles:** `guide`, `chat`, `analyze_course`. Each has a system prompt, an allowed tool subset, step caps and an output contract.
- **Prompt safety:** source text is always passed as quoted content in tool results, separated from system instructions, never as instructions (NFR-2).
- **Citation validator:** parse `[P<id>]` markers, check each refers to a passage retrieved during this run, map to note-level citations (title plus public uploader name, or "Course website: ..."), and reject sections with no valid citation (FR-7.3, FR-10.2, NFR-4).
- Emit progress events through a callback (C15) at each tool call.

**Interface:** `AgentRunner.run(profile, context, user_message, on_progress)`; `ToolRegistry`; `CitationValidator`.
**Owns entities:** none (results are stored by C11, C12, C14).

---

### C11: Conflicts (Q5: A)
**Purpose:** Decide, store and resolve factual disagreements between sources.

**Responsibilities**
- Back the `compare_passages` tool: ask C9 (`generate_structured`) to group passages into versions of one claim and say whether the difference is factual (FR-8.1).
- **Deterministic decision:** count distinct sources per version, pick the majority or record a tie (assumption A3), apply the "Authoritative (instructor material)" label using C7's precedence (FR-7.4, FR-8.2, FR-8.3).
- Store conflicts with versions and source counts; write agreement records to C7, which triggers weight recomputation (FR-6.3, FR-8.6).
- Instructor resolution: choose the correct version, optionally endorsing its note (FR-8.5). Serve resolved conflicts to later runs.

**Interface:** `ConflictService`; pure module `decisions.py` (`decide_majority`, `label_versions`).
**Owns entities:** `Conflict`, `ConflictVersion`, `ConflictResolution`.

---

### C12: Study Guides
**Purpose:** Generate, save and version study guides.

**Responsibilities**
- Accept a scope: exam, custom lecture range, or whole course by topic when there is no schedule (FR-7.1, US-7.4).
- Start a guide-generation job (C15) that runs the `guide` agent profile (C10).
- Save each result as a new version per course and scope; group by exam; list, view and compare two versions (FR-7.6).
- Store citations so a deleted note shows as "(removed note)" (FR-4.6).
- Note in the guide when no source in the course is endorsed (FR-5.4).

**Interface:** `GuideService`.
**Owns entities:** `Guide`, `GuideVersion`, `GuideCitation`.

---

### C13: Coverage and Course Analysis
**Purpose:** Coverage reporting for guides and the instructor view.

**Responsibilities**
- Pure `find_coverage_gaps(lectures, note_counts, thin_threshold)` ("missing" and "thin"), counting only tagged notes (FR-4.4, FR-7.3). This also backs the agent's `find_coverage_gaps` tool.
- Instructor coverage view per lecture: tagged notes, endorsed count, professor material present, open and resolved conflicts, missing/thin status (FR-9.1); the "no schedule" state instead of rows (FR-9.3).
- "Analyze course": start a job that runs the `analyze_course` profile per lecture and refreshes conflicts (FR-9.2).
- Highest-weighted unendorsed notes as candidates to review (FR-9.4).

**Interface:** `CoverageService`; pure module `gaps.py`.
**Owns entities:** none.

---

### C14: Cited Chat
**Purpose:** Per-user, per-course grounded chat.

**Responsibilities**
- Threads per user per course, visible only to that user; start a new thread (FR-10.3).
- Run the `chat` agent profile (C10) for each question; return a cited answer, or a plain "the course materials don't cover this" when citations can't be produced (FR-10.2).
- Runs within the request (about 20 seconds, NFR-3), using an HTMX loading indicator rather than a job.

**Interface:** `ChatService`.
**Owns entities:** `ChatThread`, `ChatMessage`.

---

### C15: Jobs and Progress (Q9: A)
**Purpose:** Run long work in the background and let the page show step-by-step progress.

**Responsibilities**
- Create a `Job` row (kind, course, requester, status) and start the work with FastAPI `BackgroundTasks`, using its own database session.
- Record progress events ("Retrieving notes for Lecture 3...") and final status: succeeded with a result reference, or failed with a user-facing retryable message (NFR-4).
- Serve a small job-status partial that the page polls with HTMX every 1-2 seconds until the job finishes.
- Job kinds: `site_import`, `source_processing`, `guide_generation`, `course_analysis`.

**Interface:** `JobService`, `JobRunner.start(background_tasks, job, work_fn)`.
**Owns entities:** `Job`, `JobEvent`.

---

### C16: Evaluation Harness
**Purpose:** On-demand check that the live agent finds planted problems (NFR-5).

**Responsibilities**
- Load a planted test set (real notes with known gaps, conflicts and a professor-material contradiction) into a scratch database.
- Run guide generation and course analysis with the live `GeminiClient`.
- Report which planted items were found. Not part of the regular pytest run.

**Interface:** `uv run python scripts/evaluate.py`.
**Owns entities:** none (uses a separate scratch database).

---

## 4. Data Entities (Q14: A)

Names and relationships only; fields are set in Functional Design per unit.

| Entity | Owner | Main relationships |
|---|---|---|
| `User` | C2 | has many `Membership` |
| `Course` | C3 | has many `Membership`, `Lecture`, `Exam`, `Source`, `Guide`; holds both join codes |
| `Membership` | C3 | links `User` to `Course` with a role (student, course instructor) |
| `CourseRequest` | C3 | submitted by a `User`; resolved into a `Course` or dismissed |
| `Lecture` | C3 (written by C4) | belongs to `Course` |
| `Exam` | C3 (written by C4) | belongs to `Course`; spans a range of `Lecture`s |
| `Source` | C5 | belongs to `Course`; uploaded by a `User` (none for website); has many `Passage`; stored `weight` written by C7 |
| `SourceLectureTag` | C5 | links `Source` to `Lecture` (many to many) |
| `Passage` | C6 | belongs to `Source`; holds text and embedding |
| `Upvote`, `SlidesFlag` | C7 | link `User` to `Source` |
| `Endorsement` | C7 | links a course instructor to `Source` |
| `AgreementRecord` | C7 (written by C11) | links `Source` to a `Conflict` claim; agreed with majority or not |
| `Conflict` | C11 | belongs to `Course`, optional `Lecture`; has many `ConflictVersion` |
| `ConflictVersion` | C11 | belongs to `Conflict`; supported by one or more `Source`s |
| `ConflictResolution` | C11 | resolves a `Conflict` by choosing a `ConflictVersion` |
| `Guide` | C12 | belongs to `Course`; a scope (exam, lecture range or whole course); has many `GuideVersion` |
| `GuideVersion` | C12 | belongs to `Guide`; Markdown body; has many `GuideCitation` |
| `GuideCitation` | C12 | links `GuideVersion` to `Source` (nullable after delete, shown as "(removed note)") |
| `ChatThread` | C14 | belongs to `User` and `Course`; has many `ChatMessage` |
| `ChatMessage` | C14 | belongs to `ChatThread`; role, text, citations |
| `Job` | C15 | belongs to `Course`; requested by `User`; has many `JobEvent` |
| `JobEvent` | C15 | belongs to `Job`; progress message and time |
