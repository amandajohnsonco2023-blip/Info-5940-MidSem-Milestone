# Margins: Technical Environment Document

> **Note:** Everything below is the team's current plan. Tools, models, and constraints may change if something proves infeasible during development, and any such change will go through the AI-DLC workflow.

## 1. Language and Dependency Management

- **Python 3.12** or newer
- **uv** for dependencies, virtual env, and scripts
- `pyproject.toml` as the single config file
- `uv.lock` checked in so all teammates get identical versions

## 2. Application Framework

- **FastAPI** + Pydantic v2 models for input validation
- **Jinja2** server-side templates + **HTMX** for dynamic updates (no frontend build)
- **SQLModel** + **SQLite** for app data and stored embeddings

## 3. Hosting and LLM Provider

- **Local only**: started with `uv run fastapi dev app/main.py`, served at `127.0.0.1:8000`; nothing hosted or deployed
- **Google Gemini API** as the only external service, via the official `google-genai` Python SDK
  - **`gemini-2.5-flash`**: study-guide agent and cited chat (supports function calling)
  - **`gemini-embedding-001`**: embeddings for semantic search

## 4. Test Tooling

- **pytest** as the test runner
- **pytest-cov**, target ≥ 80% on business logic
- **ruff** for lint and style checks

## 5. Libraries and Services We Are Not Using

| Not Using | Why | Using Instead |
| --- | --- | --- |
| LangChain, LlamaIndex | Wrap the agent loop in abstractions the team would struggle to debug and explain | A function-calling loop written by the team on top of `google-genai` |
| Other LLM providers (OpenAI, Anthropic, Ollama) | One provider keeps prompts, tool schemas, and testing consistent | Gemini API via `google-genai` |
| ChromaDB, PostgreSQL | One more datastore or server to set up and keep in sync, with no benefit at our data size | SQLite, with similarity computed in numpy |
| Flask, Django, Streamlit | FastAPI is the chosen framework; Streamlit handles logins, roles, and multi-page navigation poorly | FastAPI with Jinja2 and HTMX |
| React, npm-based build tooling | Introduces a second language and a build pipeline we don't have time for | Jinja2 templates with HTMX |
| `eval()`, `exec()`, `pickle` on user or model content | Opens the door to running arbitrary code | Explicit parsing or JSON |
| Jinja2's `\|safe` filter on notes or model output | Allows injected HTML/JavaScript (XSS) | Render Markdown via markdown-it-py with raw HTML turned off, and leave autoescaping on |
| pip, poetry, conda | Mixing installers leads to inconsistent environments | uv |

## 6. Security Essentials

- **Authentication**
  - Local email/password sign-up
  - Passwords stored as bcrypt hashes only
  - Signed session cookie for logins
  - Student vs. instructor role checks in FastAPI dependencies
- **Validating input**
  - Pydantic type and length limit on every field
  - Uploads: `.md`, `.txt`, `.pdf` only, max 5 MB
  - Course-site fetching: public http/https URLs only, with timeouts and size caps
  - Note and website text passed to the model as content, never as instructions
- **Secrets**
  - `GEMINI_API_KEY` and `SESSION_SECRET` in a git-ignored `.env`
  - Only `.env.example` with placeholders committed
  - Secrets never logged

## 7. Reference Code Snippets

**Route handler pattern** (uploading a note to a course):

```python
from fastapi import APIRouter, Depends, Form, HTTPException, Request, UploadFile, status
from fastapi.responses import HTMLResponse
from sqlmodel import Session

from app.auth import require_member
from app.db import get_session
from app.models import User
from app.services.files import UnsupportedFileError
from app.services.notes import NoteService
from app.templating import templates

router = APIRouter(prefix="/courses/{course_id}/notes")

MAX_UPLOAD_BYTES = 5 * 1024 * 1024


@router.post("", response_class=HTMLResponse, status_code=status.HTTP_201_CREATED)
def upload_note(
    request: Request,
    course_id: int,
    file: UploadFile,
    title: str = Form(..., min_length=1, max_length=200),
    user: User = Depends(require_member),
    session: Session = Depends(get_session),
) -> HTMLResponse:
    data = file.file.read(MAX_UPLOAD_BYTES + 1)
    if len(data) > MAX_UPLOAD_BYTES:
        raise HTTPException(status.HTTP_413_REQUEST_ENTITY_TOO_LARGE, "Notes must be 5 MB or smaller.")
    service = NoteService(session)
    try:
        note = service.create_note(course_id, user, title, file.filename or "", data)
    except UnsupportedFileError as e:
        raise HTTPException(status.HTTP_415_UNSUPPORTED_MEDIA_TYPE, str(e)) from e
    suggestion = service.suggest_lecture(note)  # LLM-backed; None if unsure
    return templates.TemplateResponse(
        request, "notes/_confirm_lecture.html", {"note": note, "suggestion": suggestion}, status_code=201
    )
```

**Business-logic function pattern** (finding lectures with missing or thin note coverage):

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class CoverageGap:
    lecture_id: int
    title: str
    note_count: int
    status: str  # "missing" or "thin"


def find_coverage_gaps(
    lectures: list[tuple[int, str]],
    note_counts: dict[int, int],
    thin_threshold: int = 2,
) -> list[CoverageGap]:
    """Return lectures with no notes ("missing") or fewer than thin_threshold notes ("thin")."""
    if thin_threshold < 1:
        raise ValueError("thin_threshold must be at least 1")
    gaps = []
    for lecture_id, title in lectures:
        count = note_counts.get(lecture_id, 0)
        if count == 0:
            gaps.append(CoverageGap(lecture_id, title, 0, "missing"))
        elif count < thin_threshold:
            gaps.append(CoverageGap(lecture_id, title, count, "thin"))
    return gaps
```

**Unit test pattern:**

```python
import pytest

from app.services.coverage import find_coverage_gaps


def test_reports_missing_and_thin_lectures() -> None:
    lectures = [(1, "Intro"), (2, "Regression"), (3, "Trees")]
    gaps = find_coverage_gaps(lectures, note_counts={1: 4, 2: 1})
    assert [(g.lecture_id, g.status) for g in gaps] == [(2, "thin"), (3, "missing")]


def test_invalid_threshold_raises() -> None:
    with pytest.raises(ValueError):
        find_coverage_gaps([], {}, thin_threshold=0)
```
