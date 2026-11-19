# Margins: Services and Orchestration

**Stage**: INCEPTION - Application Design

This document describes the service layer: what each service is responsible for, and how services work together in the main flows. Margins is one local FastAPI application (a modular monolith), so "service" here means a Python service class inside the app, not a separately deployed service.

---

## 1. Service Layer Rules (Q8: A, Q13: A)

- **Routes** parse HTTP input into Pydantic forms, apply auth dependencies (C1), call one service method, and choose a template or HTMX partial. Routes contain no business logic.
- **Services** hold all business logic, take a SQLModel `Session`, re-check the specific permission, and commit their own changes.
- **Pure modules** (`weighting.py`, `decisions.py`, `gaps.py`, `chunk_text`, `validate_public_url`, `select_exam_lectures`, `cosine_similarities`) hold deterministic rules with no database or LLM access. These are the targets for property-based tests.
- **Only C9** talks to Gemini. **Only C10** runs an agent loop. **Only C5's presenter** exposes uploader names.
- **Background work** always goes through C15 (`JobRunner`), which opens its own session. Request-scoped sessions are never shared with background tasks.

---

## 2. Service Catalogue

| Service | Component | Responsibility | Called by |
|---|---|---|---|
| `AccountService` | C2 | Sign-up, authentication | Account routes, C1 |
| `CourseService` | C3 | Courses, join codes, "My courses" | Course routes, `CourseRequestService` |
| `MembershipService` | C3 | Join by code, role lookup | Course routes, C1 dependencies |
| `CourseRequestService` | C3 | Course requests lifecycle | Course-request routes |
| `CourseStructureService` | C3 | Outline, exams, scope-to-lectures | C4, C10 tools, C12, C13 |
| `SiteImportService` | C4 | Fetch, crawl, extract, store outline and website sources | Course routes (via job) |
| `SourceService` | C5 | Upload, tag, anonymity, delete, replace, listing | Source routes, C4 |
| `IngestionService` | C6 | Parse, chunk, embed, lecture suggestion | `SourceService` (via job), C4 |
| `TrustService` | C7 | Upvotes, flags, endorsements, agreement records | Trust routes, C11 |
| `WeightService` | C7 | Recompute and read weights | `TrustService`, C8, C10 tools, C13 |
| `RetrievalService` | C8 | Ranked passage search | C10 tools |
| `ModelClient` | C9 | Gemini calls | C4, C6, C8, C10, C11 |
| `AgentRunner` | C10 | Agent loop for guide, chat, analyze | C12, C13, C14 |
| `ConflictService` | C11 | Compare, decide, store, resolve | C10 `compare_passages` tool, conflict routes |
| `GuideService` | C12 | Generate, save, version guides | Guide routes (via job) |
| `CoverageService` | C13 | Coverage view, analysis | Coverage routes (via job) |
| `ChatService` | C14 | Threads and answers | Chat routes |
| `JobService`, `JobRunner` | C15 | Background jobs and progress | C4, C5, C12, C13; job-status route |

---

## 3. Orchestration Flows

### Flow A: Key journey (sign up, join, upload, chat)

```
Student -> AccountService.sign_up -> login_user (C1)
        -> MembershipService.join_by_code (student code)
        -> CourseStructureService.get_outline (browse outline, US-3.5)
        -> SourceService.upload
             -> JobRunner.start(source_processing)
                  -> IngestionService.process_source
                       parse_file -> chunk_text -> ModelClient.embed -> store Passages
                       -> suggest_lectures (ModelClient.generate_structured)
             (page polls job status; confirm form appears when done)
        -> SourceService.set_lecture_tags / set_anonymity
        -> ChatService.ask
             -> AgentRunner.run(profile="chat")
                  -> tools: get_course_outline, retrieve_passages (RetrievalService), get_resolved_conflicts
             -> CitationValidator.validate
             -> cited answer, or "the course materials don't cover this"
```

### Flow B: Course website import (Q3: A, fixed pipeline)

```
Course instructor adds URL / clicks Refresh
  -> SiteImportService.start_import -> JobRunner.start(site_import)
       -> SafeFetcher + same_site_links (one level, page cap) + PDF download
       -> extract_html_text / extract_pdf_text
       -> CourseStructureExtractor.extract (ModelClient.generate_structured)
       -> CourseStructureService.replace_structure (outline + exams, or "no schedule")
       -> SourceService.remove_website_sources, then add_website_source per page/PDF
       -> IngestionService.process_source for each website source
       -> JobService.add_event at each step; succeed or fail
```

### Flow C: Study guide generation (Q2: A, Q4: A, Q9: A)

```
Member picks scope (exam | lecture range | whole course)
  -> GuideService.start_generation -> JobRunner.start(guide_generation)
       -> CourseStructureService.lectures_for_scope
       -> AgentRunner.run(profile="guide", on_progress=JobService.add_event)
            loop (capped):
              get_course_outline
              list_sources_for_lecture / retrieve_passages     (C8, ranked by weight)
              get_source_weights                               (C7)
              compare_passages -> ConflictService.compare       (C11: LLM groups, code decides)
                                -> TrustService.record_agreement -> WeightService.recompute
              get_resolved_conflicts                           (C11)
              find_coverage_gaps                               (C13 pure, skipped without a schedule)
            final Markdown with [P<id>] markers
       -> CitationValidator.validate (reject ungrounded sections)
       -> add "no instructor endorsements" note when TrustService.course_has_endorsements is false
       -> save GuideVersion + GuideCitations
  Browser polls job status (HTMX) -> shows the new version when done
```

### Flow D: Analyze course (FR-9.2)

```
Course instructor clicks "Analyze course"
  -> CoverageService.start_analysis -> JobRunner.start(course_analysis)
       -> for each lecture: AgentRunner.run(profile="analyze_course")
            tools: list_sources_for_lecture, retrieve_passages, compare_passages
       -> conflicts and agreement records stored (as in Flow C)
  -> CoverageService.coverage_view shows refreshed conflict counts
```

### Flow E: Trust signals change

```
Upvote / slides flag / endorsement / conflict resolution with endorse
  -> TrustService (or ConflictService.resolve)
  -> WeightService.recompute(source_id)    (Q10: A, stored)
  -> next retrieval and next guide reflect the new weight
```

### Flow F: Delete or replace a note

```
Delete:  SourceService.delete -> cascade Passages, Upvotes, SlidesFlags, Endorsement
         -> GuideCitation.source_id set to null (shown as "(removed note)")
Replace: SourceService.replace_file -> JobRunner.start(source_processing)
         -> IngestionService deletes old Passages and indexes the new file; ID and trust signals kept
```

---

## 4. Cross-Cutting Patterns

| Concern | Pattern | Where |
|---|---|---|
| Authorization | Route dependencies plus service re-checks | C1, every service |
| Anonymity | Single presenter `present_source` | C5; used by C8, C10, templates |
| LLM failure | Retries in `GeminiClient`; `ModelUnavailableError` becomes a retryable message | C9, C15, C14 |
| Agent safety | Step and tool-call caps, validated tool args, source text as quoted content | C10 |
| Grounding | Passage-ID markers checked by code; uncited output is a failure | C10 |
| Progress | Job table plus HTMX polling | C15 |
| Safe rendering | markdown-it-py with raw HTML off, autoescape on, no `\|safe` | C1 |
| Safe fetching | URL validation after DNS resolution, redirects re-checked, caps and timeouts | C4 |
| Testability | `FakeModelClient`; pure modules with Hypothesis tests | C9, C7, C11, C13, C6, C4 |
