# Application Design Plan: Margins

**Role**: Software architect
**Inputs**: `aidlc-docs/inception/requirements/requirements.md` (approved), `aidlc-docs/inception/user-stories/stories.md` and `personas.md` (approved), `aidlc-docs/inception/plans/execution-plan.md` (approved), `TechnicalEnvironment.md`
**Outputs**: `aidlc-docs/inception/application-design/` (`components.md`, `component-methods.md`, `services.md`, `component-dependency.md`, `application-design.md`)

**Scope reminder.** Application Design identifies components, their responsibilities and interfaces, the service layer, and how components depend on each other. Detailed business rules (weight constants, smoothing, thresholds, exact prompts) are left to Functional Design in Construction.

---

## Part A: Questions

Fill in each `[Answer]:` tag with a letter. If no option fits, choose the last option (Other) and describe what you want after the tag. All questions must be answered before I generate the design. Your answers stay in this file as part of the audit trail.

Where I have a recommendation, it is marked **(Recommended)** with a one-line reason. It is only a suggestion.

---

### Component Identification

## Question 1
How should the application code be organized into packages?

A) **By layer**: `app/routes/`, `app/services/`, `app/agent/`, `app/llm/`, `app/ingestion/`, `app/models/`, `app/templates/`. Each layer holds every feature's code for that layer.

B) **By feature**: `app/accounts/`, `app/courses/`, `app/notes/`, `app/trust/`, `app/guides/`, `app/chat/`, each containing its own routes, service, models and templates, plus shared `app/agent/` and `app/llm/` packages.

C) **Hybrid (Recommended)**: feature packages for the web-facing domains (as B), plus separate shared core packages for cross-cutting machinery used by several features: `app/llm/` (model client), `app/agent/` (loop and tool registry), `app/ingestion/` (parsing, chunking, embedding, site fetching), `app/retrieval/`. *Reason: maps cleanly onto units that different teammates can own, while keeping the agent and retrieval in one place.*

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 2
Should the study-guide agent, the cited-chat agent and "Analyze course" share one agent component?

A) **One generic agent loop (Recommended)**: a single function-calling loop and tool registry. Each feature (guide, chat, analyze course) is a thin "agent profile": its own system prompt, allowed tool subset, step cap and output schema. *Reason: one loop to test and debug; FR-7.2's step caps and NFR-4's error handling are written once.*

B) **Separate agents**: each feature has its own loop implementation tailored to it.

C) **Agent only for guide and chat**: "Analyze course" is a fixed, non-agentic pipeline (for each lecture: retrieve, compare, store conflicts) that calls the same `compare_passages` logic directly.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
How should the course-website import (FR-3) work?

A) **Fixed pipeline (Recommended)**: deterministic code fetches the page, its same-site links one level deep and linked PDFs; then one or two structured-output Gemini calls extract the lecture outline and exams into Pydantic models. No agent loop. *Reason: the steps are always the same, so a pipeline is faster, cheaper and easier to test.*

B) **Agentic**: the agent decides which links to follow and when it has found the schedule, using `fetch_page` and `parse_schedule` tools.

C) **Fully deterministic**: HTML table and list parsing with no LLM; if parsing fails, treat it as "no schedule found".

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

### Component Methods and Interface Contracts

## Question 4
How should citations be produced and checked (FR-7.3, FR-10.2, NFR-4 "no citations = failure")?

A) **Passage IDs in the output, validated by code (Recommended)**: the model writes inline markers such as `[P123]` that refer to retrieved passage IDs. Deterministic code checks that every marker is a passage the agent actually retrieved, maps it to a note-level citation (title plus uploader or "Anonymous classmate"), and rejects output where sections have no valid citation. *Reason: grounding is verified by code rather than trusted to the model.*

B) **`cite_source` tool**: the model calls a `cite_source` tool for each claim, and the final text is assembled from those calls.

C) **Structured output**: the model returns JSON (sections, each with claims and source IDs), and the server renders it into Markdown.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
How should conflicts be decided and stored (FR-8.1, FR-8.2, assumption A3)?

A) **LLM groups, code counts (Recommended)**: the `compare_passages` tool asks the model to group passages into versions of a claim and say whether the difference is factual. Deterministic code then counts distinct sources per version, picks the majority (or a tie), applies the professor-material label, and writes conflict and agreement records. *Reason: majority and precedence stay deterministic and property-testable; only the semantic judgment uses the LLM.*

B) **LLM decides everything**: the model returns the versions, the majority and the labels, and code just stores them.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
How should each agent tool be defined so both Gemini and the tests can use it?

A) **Plain function plus Pydantic argument model (Recommended)**: each tool is a plain Python function with a Pydantic model for its arguments and its result, registered in a tool registry that produces the Gemini function declaration from the model's schema. The function receives the course ID and services by dependency injection, never from the model. *Reason: arguments are validated (NFR-4), schemas stay in sync, and tools can be unit-tested without an LLM.*

B) **Tool classes**: each tool is a class with `name`, `schema` and `run()`, with the schema written by hand.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
What should the model-client interface (the only code that talks to Gemini) expose?

A) **Small protocol with three methods (Recommended)**: `generate(messages, tools) -> ModelTurn`, `generate_structured(messages, schema) -> BaseModel` and `embed(texts) -> list[vector]`. A `GeminiClient` implements it with timeouts and retries, and a `FakeModelClient` with scripted responses is used in tests. *Reason: the regular test suite never calls live Gemini (NFR-5), and retries live in one place.*

B) **Two separate interfaces**: one for chat and generation, one for embeddings, so either could be swapped independently.

C) **No interface**: call `google-genai` directly in services and mock it in tests.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

### Service Layer Design

## Question 8
How should routes, services and database access be layered?

A) **Routes, then services, then the SQLModel session (Recommended)**: routes only handle HTTP (parsing forms, auth dependencies, choosing templates). All business logic and permission checks live in service classes that receive a `Session`. No separate repository layer. *Reason: matches the TechnicalEnvironment.md example (`NoteService(session)`) and keeps the layer count low for a student team.*

B) **Routes, then services, then repositories**: a separate repository class per table wraps all queries.

C) **Thin services**: routes query SQLModel directly for simple CRUD; services exist only for complex logic.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
How should long-running jobs (study guide, "Analyze course", website import: up to about 2 minutes, FR-7.5) run?

A) **Background task plus job table plus HTMX polling (Recommended)**: the request creates a `Job` row and starts the work with FastAPI `BackgroundTasks`. The agent writes progress events to the job row, and the page polls a small job-status partial with HTMX every second or two until it finishes. *Reason: no new infrastructure, it survives a page reload, and it is easy to test.*

B) **Server-sent events**: the request streams progress messages to the browser as they happen (HTMX SSE extension). Progress is lost if the page is reloaded.

C) **Synchronous**: the request blocks until the job finishes and shows a spinner; no step-by-step progress.

D) **Separate worker process** (for example a task queue) that picks up jobs.

E) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 10
When should a note's quality weight (FR-6.2) be computed?

A) **Recomputed on change and stored (Recommended)**: a `WeightService` recomputes and stores a note's weight whenever one of its inputs changes (upvote, flag, endorsement, new agreement records). Retrieval reads the stored value. *Reason: retrieval stays fast and simple, and the stored value is easy to show and debug.*

B) **Computed at retrieval time** from raw counts every time a search runs.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 11
When should the lecture suggestion for a newly uploaded note (FR-4.3, an LLM call) run?

A) **During the upload request**: the upload response waits for the suggestion (a few seconds) and shows it immediately, as in the TechnicalEnvironment.md example.

B) **After the upload, in the background**: the upload returns at once, and the "suggested lecture" box loads afterwards with HTMX. Parsing, chunking and embedding run in the same background job.

C) **Split (Recommended)**: the upload stores the file and returns at once; parsing, chunking, embedding and the lecture suggestion run as a background job (same mechanism as Question 9). The confirm-lecture form appears when the job finishes. *Reason: keeps uploads under the 1-second target (NFR-3) and reuses one job mechanism.*

D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

### Component Dependencies and Patterns

## Question 12
How should retrieval load passage embeddings for similarity search (FR-6.5)?

A) **Load per query (Recommended)**: each search loads the course's passage embeddings from SQLite into a numpy array and computes cosine similarity. At demo scale (about 200 notes per course) this takes milliseconds. *Reason: no cache to keep in sync when notes are added, replaced or deleted.*

B) **In-memory cache per course**, invalidated whenever a note in that course changes.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 13
How should permission and anonymity rules be enforced across components (FR-1.4, FR-4.5, NFR-7)?

A) **Both layers (Recommended)**: FastAPI dependencies enforce sign-in and course membership/role on every course route; services re-check the specific rule (for example "only the uploader can delete"); and a single presenter function turns a note into its public view, replacing the uploader's name with "Anonymous classmate" unless the viewer is a course instructor. Templates and the agent only ever receive that public view. *Reason: anonymity is enforced in one place, server-side, as NFR-7 requires.*

B) **Route dependencies only**: services trust the caller, and each template decides how to show names.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 14
Should Application Design also sketch the main data entities (tables and their relationships), or leave the data model entirely to Functional Design?

A) **Sketch entities here (Recommended)**: list the main entities and which component owns each (names and relationships only, no columns). Functional Design fills in the fields per unit. *Reason: Units Generation needs to know which unit owns which tables so teammates don't collide.*

B) **Leave it to Functional Design**: components only, no entity list.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Answer Summary and Analysis

Answers were given interactively in a chat session with the AI on 2026-11-19 and recorded in the `[Answer]:` tags above.

| Q | Answer | Decision |
|---|---|---|
| 1 | C | Hybrid layout: feature packages plus shared `llm/`, `agent/`, `ingestion/`, `retrieval/` |
| 2 | A | One generic agent loop; guide, chat and analyze-course are agent profiles |
| 3 | A | Website import is a fixed pipeline with structured-output extraction, not an agent |
| 4 | A | Passage-ID citation markers, validated and mapped by deterministic code |
| 5 | A | LLM groups versions and judges "factual"; code counts sources, picks majority, applies labels |
| 6 | A | Tools are plain functions with Pydantic argument/result models in a registry |
| 7 | A | One model-client protocol: `generate`, `generate_structured`, `embed`; Gemini and fake implementations |
| 8 | A | Routes, then services (taking a `Session`), no repository layer |
| 9 | A | Long jobs: `BackgroundTasks` plus a `Job` table plus HTMX polling |
| 10 | A | Note weight recomputed on change and stored |
| 11 | C | Upload returns at once; parse, chunk, embed and lecture suggestion run as a background job |
| 12 | A | Embeddings loaded from SQLite per query |
| 13 | A | Permissions checked in route dependencies and services; one presenter enforces anonymity |
| 14 | A | Main entities sketched here with an owning component; fields left to Functional Design |

**Analysis (Step 8).** No vague, combined or contradictory answers. Q9 and Q11 share one job mechanism, and Q2, Q5 and Q6 fit together (analyze-course is an agent profile whose `compare_passages` tool returns LLM groupings that code then counts).

**Noted departure from requirements.** Q4 replaces the `cite_source` tool listed in FR-7.2 with passage-ID markers checked by a citation validator. The FR-7.3 and FR-10.2 behavior (note-level citations, uncited output treated as failure) is unchanged. This is recorded in `application-design.md` under design decisions.

**Follow-up questions:** none needed.

---

## Part B: Generation Checklist

Executed after all answers are in and the plan is approved.

- [x] 1. Analyze requirements (FR-1 to FR-10, NFR-1 to NFR-7) and the 39 stories to identify business capabilities
- [x] 2. Apply answers Q1 to Q14 to the component boundaries, interfaces and patterns
- [x] 3. Generate `components.md`: each component's name, purpose, responsibilities and interface
- [x] 4. Generate `component-methods.md`: method signatures with input/output types and one-line purposes (business rules deferred to Functional Design)
- [x] 5. Generate `services.md`: service definitions, responsibilities and orchestration flows (key journey, guide generation, chat, website import, analyze course)
- [x] 6. Generate `component-dependency.md`: dependency matrix, communication patterns, data-flow diagrams (Mermaid, validated, with text alternatives)
- [x] 7. Generate `application-design.md` consolidating the four documents above
- [x] 8. Validate completeness and consistency: every FR is covered by at least one component; no circular dependencies; the agent, model client and retrieval contracts match each other
- [x] 9. Check extension compliance (Property-Based Testing: note which components hold the deterministic logic that PBT-01 will target in Functional Design)
