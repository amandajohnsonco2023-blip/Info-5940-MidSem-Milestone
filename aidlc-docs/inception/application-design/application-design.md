# Margins: Application Design

**Stage**: INCEPTION - Application Design
**Inputs**: `requirements.md` (approved), `stories.md` and `personas.md` (approved), `execution-plan.md` (approved), `application-design-plan.md` (answers Q1-Q14)
**Detailed documents**: [`components.md`](components.md), [`component-methods.md`](component-methods.md), [`services.md`](services.md), [`component-dependency.md`](component-dependency.md)

This document summarizes the application design in one place. The four documents above hold the full detail.

---

## 1. Architecture at a Glance

Margins is a **modular monolith**: one local FastAPI application, server-rendered with Jinja2 and HTMX, storing everything (including embeddings) in SQLite through SQLModel, with Gemini as the only external service.

- **Feature packages** own the web-facing domains: accounts, courses, sources, trust, conflicts, guides, coverage, chat, jobs.
- **Shared core packages** hold the machinery used by several features: the model client, the agent loop, ingestion and retrieval.
- **Pure modules** hold the deterministic rules (weights, majority decisions, coverage gaps, chunking, URL validation, exam ranges), so they can be property-tested without the LLM.

```
Browser (HTMX)
   |
C1 Web Shell and Auth  (routes, auth dependencies, safe Markdown rendering)
   |
Feature services: C2 Accounts | C3 Courses | C4 Site Import | C5 Sources | C7 Trust
                  C11 Conflicts | C12 Guides | C13 Coverage | C14 Chat
   |
Shared core:      C6 Ingestion | C8 Retrieval | C10 Agent Core (loop, tools, citations)
   |
Foundations:      C9 Model Client -> Gemini      C15 Jobs and Progress      SQLite
```

---

## 2. Design Decisions

| # | Decision | Answer | Why it matters |
|---|---|---|---|
| AD-1 | Hybrid package layout: feature packages plus shared `llm/`, `agent/`, `ingestion/`, `retrieval/` | Q1: C | Packages map onto units that different teammates can own |
| AD-2 | One generic agent loop; guide, chat and analyze-course are **agent profiles** (prompt, tool subset, caps, output contract) | Q2: A | Caps, validation and error handling are written and tested once |
| AD-3 | Website import is a **fixed pipeline** with structured-output extraction | Q3: A | Faster, cheaper and easier to test than an agent for a fixed task |
| AD-4 | Citations are `[P<id>]` passage markers, **checked by code** and mapped to note-level citations | Q4: A | Grounding is verified, not trusted. **Replaces the `cite_source` tool listed in FR-7.2**; FR-7.3 and FR-10.2 behavior is unchanged |
| AD-5 | Conflicts: **LLM groups** passages into versions and judges "factual"; **code counts** sources, picks the majority or a tie, applies labels | Q5: A | Majority and precedence stay deterministic and property-testable |
| AD-6 | Tools are plain functions with Pydantic argument and result models in a registry that generates Gemini declarations; course ID and viewer are injected | Q6: A | Validated arguments (NFR-4); the model cannot reach another course |
| AD-7 | One `ModelClient` protocol (`generate`, `generate_structured`, `embed`) with `GeminiClient` and `FakeModelClient` | Q7: A | No live Gemini calls in the test suite (NFR-5) |
| AD-8 | Routes, then services (taking a `Session`); no repository layer | Q8: A | Matches TechnicalEnvironment.md; fewer layers |
| AD-9 | Long jobs: `BackgroundTasks` plus a `Job` table plus HTMX polling | Q9: A | Step-by-step progress (FR-7.5) that survives a reload, with no extra infrastructure |
| AD-10 | Note weights recomputed on every input change and stored | Q10: A | Fast retrieval; the stored value is easy to show and debug |
| AD-11 | Upload returns at once; parse, chunk, embed and lecture suggestion run as a background job | Q11: C | Keeps uploads under 1 second (NFR-3); reuses AD-9 |
| AD-12 | Embeddings loaded from SQLite per query | Q12: A | Nothing to keep in sync at demo scale |
| AD-13 | Permissions in route dependencies **and** services; one presenter enforces anonymity | Q13: A | Anonymity is enforced in one place, server-side (NFR-7) |
| AD-14 | Main entities sketched with an owning component; fields in Functional Design | Q14: A | Units Generation can assign table ownership |
| AD-15 | Chat runs within the request (no job), with an HTMX loading indicator | Derived from NFR-3 (about 20 s) | Simpler than a job for a short wait |
| AD-16 | Added agent tool `get_resolved_conflicts` | Derived from FR-8.5 | Later guides and chat must present instructor resolutions |

---

## 3. Components

| ID | Component | Package | Responsibility in one line |
|---|---|---|---|
| C1 | Web Shell and Auth | `app/web/` | App frame, sessions, auth dependencies, safe Markdown rendering |
| C2 | Accounts | `app/accounts/` | Sign-up, login, bcrypt, instructor seeding command |
| C3 | Courses and Membership | `app/courses/` | Courses, join codes, roles, course requests, outline and exams |
| C4 | Course Site Import | `app/ingestion/site/` | Safe fetch and crawl, text extraction, outline and exam extraction |
| C5 | Sources | `app/sources/` | Notes, professor material, website sources, tags, anonymity, delete/replace |
| C6 | Ingestion Pipeline | `app/ingestion/` | Parse, chunk, embed, lecture suggestion |
| C7 | Trust and Weighting | `app/trust/` | Upvotes, flags, endorsements, agreement records, weight formula |
| C8 | Retrieval | `app/retrieval/` | Cosine similarity plus weight ranking |
| C9 | Model Client | `app/llm/` | The only Gemini access; retries; fake for tests |
| C10 | Agent Core | `app/agent/` | Agent loop, tool registry, tools, profiles, citation validator |
| C11 | Conflicts | `app/conflicts/` | Compare, decide majority, store, resolve |
| C12 | Study Guides | `app/guides/` | Generate, save and compare guide versions |
| C13 | Coverage and Course Analysis | `app/coverage/` | Coverage gaps, instructor view, analyze course |
| C14 | Cited Chat | `app/chat/` | Private per-user threads with cited answers |
| C15 | Jobs and Progress | `app/jobs/` | Background jobs, progress events, polling partial |
| C16 | Evaluation Harness | `scripts/evaluate.py` | Live run on the planted test set |

Full responsibilities and interfaces: `components.md`. Method signatures: `component-methods.md`.

---

## 4. Agent Design

| Profile | Used by | Allowed tools | Output |
|---|---|---|---|
| `guide` | C12 | `get_course_outline`, `list_sources_for_lecture`, `retrieve_passages`, `get_source_weights`, `compare_passages`, `get_resolved_conflicts`, `find_coverage_gaps` | Markdown guide with `[P<id>]` markers |
| `chat` | C14 | `get_course_outline`, `retrieve_passages`, `get_source_weights`, `get_resolved_conflicts` | Cited answer, or "not covered" |
| `analyze_course` | C13 | `list_sources_for_lecture`, `retrieve_passages`, `compare_passages` | Conflicts and agreement records (no guide) |

Every profile has hard step and tool-call caps (values set in Functional Design). Source text is passed to the model only as quoted tool-result content, separated from system instructions.

---

## 5. Data Entities

`User`; `Course`, `Membership`, `CourseRequest`, `Lecture`, `Exam`; `Source`, `SourceLectureTag`; `Passage`; `Upvote`, `SlidesFlag`, `Endorsement`, `AgreementRecord`; `Conflict`, `ConflictVersion`, `ConflictResolution`; `Guide`, `GuideVersion`, `GuideCitation`; `ChatThread`, `ChatMessage`; `Job`, `JobEvent`. Owners and relationships are in `components.md` section 4.

---

## 6. Requirements Coverage

| Requirement | Components |
|---|---|
| FR-1 Accounts and authentication | C1, C2 |
| FR-2 Courses and membership | C3, C1 |
| FR-3 Course structure from the website | C4, C3, C5, C6 |
| FR-4 Notes and professor material | C5, C6 |
| FR-5 Ratings, flags, endorsements | C7 |
| FR-6 Quality weight and precedence | C7, C8, C11 |
| FR-7 Study guide generation | C12, C10, C8, C11, C13, C15 |
| FR-8 Conflicts | C11, C10, C7 |
| FR-9 Coverage view and analysis | C13, C10, C11, C15 |
| FR-10 Cited chat | C14, C10, C8 |
| NFR-1 Technology | all (FastAPI, SQLModel, Gemini only) |
| NFR-2 Security essentials | C1, C2, C4, C5, C10 |
| NFR-3 Performance | C8, C15, C14 |
| NFR-4 LLM reliability | C9, C10, C15 |
| NFR-5 Testability | C9 fakes, pure modules, C16 |
| NFR-6 Usability | C1 templates |
| NFR-7 Maintainability and privacy | layering rules, C5 presenter |

Every FR and NFR maps to at least one component.

---

## 7. Validation

- **Completeness:** all 10 functional and 7 non-functional requirements are covered (section 6); all 39 stories fall inside at least one component (main stories listed in `components.md` section 2).
- **Consistency:** the tool list, the model-client protocol and the retrieval contract match across `components.md`, `component-methods.md` and `services.md`.
- **Dependencies:** the service-level dependency matrix has no cycles; table modules and pure modules are leaves (`component-dependency.md` section 1).
- **Diagrams:** Mermaid diagrams use alphanumeric node IDs and quoted labels, and each has a text alternative.

---

## 8. Extension Compliance (Application Design stage)

| Extension | Status | Notes |
|---|---|---|
| Security Baseline | Disabled | Skipped. NFR-2 essentials are placed in C1, C2, C4, C5 and C10. |
| Resiliency Baseline | Disabled | Skipped. |
| Property-Based Testing | Enabled | **N/A at this stage** (PBT-01 applies in Functional Design; PBT-02 to PBT-08 and PBT-10 in Code Generation). The design isolates the deterministic logic PBT-01 will target into pure modules: `weighting.py` (C7), `decisions.py` (C11), `gaps.py` (C13), `chunk_text` (C6), `validate_public_url` and `same_site_links` (C4), `select_exam_lectures` (C3), `cosine_similarities` and `combine_scores` (C8), and the tool argument/result models (C10). PBT-09 remains compliant (Hypothesis). |
