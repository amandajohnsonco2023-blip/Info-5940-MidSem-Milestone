# Margins: Units of Work

**Stage**: INCEPTION - Units Generation
**Inputs**: `unit-of-work-plan.md` (answers Q1-Q11, approved), `application-design.md` and the four detailed design documents (approved), `stories.md`
**Related**: [`unit-of-work-dependency.md`](unit-of-work-dependency.md), [`unit-of-work-story-map.md`](unit-of-work-story-map.md)

Margins is one FastAPI application (a modular monolith). A **unit of work** here is a set of components, tables and stories that one teammate owns and builds in Construction. Units are **modules inside one app**, not separately deployed services (Q10: A).

---

## 1. Units at a Glance (Q1: A, Q2: A)

| Unit | Components | Stories (Must / Should / Could) | Owner | NFR stages in Construction (Q11: A) |
|---|---|---|---|---|
| **U1 Platform and Access** | C1 Web Shell and Auth, C2 Accounts, C3 Courses and Membership, C15 Jobs and Progress | 10 (7 / 1 / 2) | TBD | Yes (auth, sessions) |
| **U2 Content Ingestion** | C5 Sources, C6 Ingestion Pipeline, C4 Course Site Import | 11 (7 / 4 / 0) | TBD | Yes (uploads, safe fetching) |
| **U3 Trust and Retrieval** | C7 Trust and Weighting, C8 Retrieval, C11 Conflicts | 7 (3 / 4 / 0) | TBD | No |
| **U4 Agent Core and Study Guides** | C9 Model Client, C10 Agent Core, C12 Study Guides | 5 (3 / 2 / 0) | TBD | Yes (LLM, agent) |
| **U5 Chat, Coverage and Evaluation** | C14 Cited Chat, C13 Coverage and Course Analysis, C16 Evaluation Harness | 6 (3 / 2 / 1) | TBD | Yes (chat uses the agent) |
| **Total** | 16 components | **39 (23 / 13 / 3)** | | |

**Owners (Q7: B):** to be assigned by the team at the next meeting. Record each owner in the table above when decided.

Every component belongs to exactly one unit. Every story has exactly one primary unit (see the story map). Within each unit, **Must stories are built first**, then Should, then Could (Q3: A).

---

## 2. Unit Definitions

### U1: Platform and Access
**Goal:** the application frame everyone builds on, plus accounts, courses and background jobs.

**Components:** C1 Web Shell and Auth, C2 Accounts, C3 Courses and Membership, C15 Jobs and Progress
**Packages:** `app/main.py`, `app/config.py`, `app/db.py`, `app/templating.py`, `app/web/`, `app/accounts/`, `app/courses/`, `app/jobs/`
**Owns tables (Q6: A):** `User`, `Course`, `Membership`, `CourseRequest`, `Lecture`, `Exam`, `Job`, `JobEvent`

**Responsibilities**
- App startup, router registration, database setup, settings from `.env`, `.env.example`
- Session cookie, auth dependencies (`require_user`, `require_instructor_flag`, `require_member`, `require_course_instructor`), error mapping
- Layout templates, navigation, safe Markdown filter (raw HTML off)
- Sign-up, login, logout, bcrypt, instructor seeding command
- Courses, join codes, joining, "My courses", course requests
- Course outline and exams storage, `lectures_for_scope`, `select_exam_lectures`; outline browsing page (US-3.5)
- Job table, `JobRunner`, job-status HTMX partial

**Contracts to publish first (M0):** app skeleton that runs; auth dependencies and `CourseContext`; `get_session` and background session factory; `JobService`/`JobRunner`; `CourseStructureService.get_outline` and `replace_structure`; table models above.

**Build order inside the unit**
1. Must: skeleton and auth dependencies (M0), US-1.1, US-1.2, US-1.3, US-2.1, US-2.2, US-2.3, US-3.5
2. Should: US-2.4
3. Could: US-2.5, US-2.6

**Testing focus:** permission rules (property tests over roles), `select_exam_lectures` (property tests), password hashing, join-code flows.

---

### U2: Content Ingestion
**Goal:** get notes, professor material and course websites into the system as searchable passages.

**Components:** C5 Sources, C6 Ingestion Pipeline, C4 Course Site Import
**Packages:** `app/sources/`, `app/ingestion/` (including `app/ingestion/site/`)
**Owns tables:** `Source` (the `weight` column is written only by U3's `WeightService`), `SourceLectureTag`, `Passage`

**Responsibilities**
- Upload (type allowlist, 5 MB cap), professor material, delete, replace, anonymity, lecture tags
- `present_source`: the single anonymity presenter used by every other unit
- Parse, chunk, embed (through U4's `ModelClient`), lecture suggestion
- Safe fetcher, same-site crawl, HTML and PDF text extraction, outline and exam extraction, refresh, "no schedule" state
- Website sources

**Contracts to publish first (M0):** `Source`/`Passage` tables; `present_source` and `PublicSource`; `SourceService.list_sources`/`get_source` stubs; `IngestionService.process_source` stub.

**Build order inside the unit**
1. Must: US-4.1, US-4.2, US-4.3, US-4.4, US-4.7, US-3.1, US-3.4
2. Should: US-4.5, US-4.6, US-3.2, US-3.3

**Testing focus:** `chunk_text` (property tests: coverage of the text, size bounds), `validate_public_url` and `same_site_links` (property tests), anonymity presenter, upload limits; site extraction with `FakeModelClient`.

---

### U3: Trust and Retrieval
**Goal:** decide how much to trust each source, find the best passages, and decide conflicts.

**Components:** C7 Trust and Weighting, C8 Retrieval, C11 Conflicts
**Packages:** `app/trust/`, `app/retrieval/`, `app/conflicts/`
**Owns tables:** `Upvote`, `SlidesFlag`, `Endorsement`, `AgreementRecord`, `Conflict`, `ConflictVersion`, `ConflictResolution`; writes `Source.weight`

**Responsibilities**
- Upvotes, slides flags, endorsements; "How sources are ranked" page
- `weighting.py` (smoothing, weight formula, precedence); `WeightService` recompute-on-change
- `RetrievalService.search` (cosine similarity plus weight)
- `ConflictService.compare` (LLM grouping through U4's `ModelClient`, then `decisions.py`), storing conflicts, agreement records, instructor resolution

**Contracts to publish first (M0):** `RetrievalService.search` signature and `RetrievedPassage`; `WeightService.get_weights`; `ConflictService.compare` and `resolved_for_lectures` stubs; `TrustService.course_has_endorsements`.

**Build order inside the unit**
1. Must: retrieval (needed for the walking skeleton), US-5.1, US-5.3, US-8.1
2. Should: US-5.2, US-6.1, US-6.2, US-8.2

**Testing focus:** the most property-based testing of any unit: weight bounds and monotonicity, smoothing, precedence ordering, `decide_majority` (ties, counts), `cosine_similarities` and `combine_scores`.

**NFR stages:** skipped (Q11: A). Its one LLM call (conflict grouping) goes through U4's `ModelClient`, whose NFR decisions cover it.

---

### U4: Agent Core and Study Guides
**Goal:** the model client, the shared agent loop, and study guides.

**Components:** C9 Model Client, C10 Agent Core, C12 Study Guides
**Packages:** `app/llm/`, `app/agent/`, `app/guides/`
**Owns tables:** `Guide`, `GuideVersion`, `GuideCitation`

**Responsibilities**
- `ModelClient` protocol, `GeminiClient` (timeouts, retries), `FakeModelClient`
- Agent loop with caps, tool registry, all seven tools (as adapters over U1, U3 and U5 services and pure functions), agent profiles, citation validator, prompt safety
- Guide scopes, generation job, versions, compare versions, "no endorsements" note

**Contracts to publish first (M0):** `ModelClient` protocol and a working `FakeModelClient` (**everyone depends on this**); `AgentRunner.run` signature, `AgentProfile`, `ToolContext`; `CitationValidator.validate` signature.

**Build order inside the unit**
1. Must: model client and fake (M0), agent loop and `chat`-ready tools (M1), US-7.1, US-7.2, US-7.3
2. Should: US-7.4, US-7.5

**Testing focus:** agent loop with scripted fake turns (caps, invalid tool arguments, model errors), tool argument/result serialization (property tests), citation validator (property tests: only retrieved IDs survive).

---

### U5: Chat, Coverage and Evaluation
**Goal:** the student chat, the instructor coverage view and analysis, and proof that the whole system works.

**Components:** C14 Cited Chat, C13 Coverage and Course Analysis, C16 Evaluation Harness
**Packages:** `app/chat/`, `app/coverage/`, `scripts/evaluate.py`, `tests/integration/`
**Owns tables:** `ChatThread`, `ChatMessage`

**Responsibilities**
- Chat threads, `ask` using the `chat` profile, "not covered" answers, privacy
- `gaps.py` (`find_coverage_gaps`, also used by U4's tool), coverage view, endorsement candidates, "Analyze course" job
- **Cross-unit testing (Q9: A):** end-to-end integration tests with `FakeModelClient` (key journey, guide, analysis) and the live evaluation harness with the planted test set

**Contracts to publish first (M0):** `gaps.py` (pure, small: can be finished in M0); `ChatService.ask` signature.

**Build order inside the unit**
1. Must: `gaps.py` (M0), US-10.1, US-10.2 (M1 key journey), US-9.1
2. Should: US-10.3, US-9.2
3. Could: US-9.3
4. Throughout: integration tests; evaluation harness once guides and analysis work

**Testing focus:** `find_coverage_gaps` (property tests), chat privacy, integration tests, evaluation report.

---

## 3. Code Organization Strategy (greenfield; Q10: A)

**Deployment model:** one repository, one FastAPI application, one process, started with `uv run fastapi dev app/main.py`. One `pyproject.toml` and one `uv.lock` at the root.

```
Info-5940-MidSem-Milestone/
  pyproject.toml, uv.lock, .env.example        # U1 creates; anyone may add a dependency via PR
  app/
    main.py, config.py, db.py, templating.py    # U1
    web/                                        # U1  (C1)
    accounts/                                   # U1  (C2)
    courses/                                    # U1  (C3)
    jobs/                                       # U1  (C15)
    sources/                                    # U2  (C5)
    ingestion/                                  # U2  (C6)
      site/                                     # U2  (C4)
    trust/                                      # U3  (C7)
    retrieval/                                  # U3  (C8)
    conflicts/                                  # U3  (C11)
    llm/                                        # U4  (C9)
    agent/                                      # U4  (C10)
      tools/                                    # U4  (adapters over other units' services)
    guides/                                     # U4  (C12)
    chat/                                       # U5  (C14)
    coverage/                                   # U5  (C13)
    templates/<package>/                        # each unit owns the folder for its package
  scripts/
    evaluate.py                                 # U5  (C16)
    seed_instructors.py                         # U1  (or a CLI entry point)
  tests/
    <package>/                                  # each unit: unit and property-based tests
    integration/                                # U5: cross-unit tests with FakeModelClient
    fixtures/planted_set/                       # U5: evaluation test set
  aidlc-docs/                                   # documentation only; never application code
```

**Package rules**
- Inside each feature package: `routes.py`, `service.py` (or several services), `models.py` (SQLModel tables), pure modules such as `weighting.py`, and `schemas.py` (Pydantic forms and views).
- `models.py` and pure modules are **leaves**: they import nothing from other units' services, so any unit may import them.
- A unit calls another unit's code only through that unit's **service** methods, pure functions or presenter, as listed in `component-methods.md`. No reaching into another unit's tables to write.
- `app/agent/tools/` (U4) is the only place where agent tools adapt other units' services.
- Only `app/llm/` imports `google-genai`.
- The regular test suite never calls live Gemini; `scripts/evaluate.py` is the only live caller and is run by hand.

---

## 4. Readiness for Construction

| Unit | Functional Design | NFR Requirements | NFR Design | Infrastructure Design | Code Generation |
|---|---|---|---|---|---|
| U1 | Execute | Execute | Execute | Skip | Execute |
| U2 | Execute | Execute | Execute | Skip | Execute |
| U3 | Execute | Skip | Skip | Skip | Execute |
| U4 | Execute | Execute | Execute | Skip | Execute |
| U5 | Execute | Execute | Execute | Skip | Execute |

Functional Design runs for every unit because the Property-Based Testing extension (PBT-01) requires a "Testable Properties" section per unit. Construction starts after the mid-semester milestone.
