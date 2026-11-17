# Technical Environment: Margins

Margins runs locally on team laptops: a FastAPI web app that calls open models through Ollama. This document tells AI-DLC which tools to use, which to avoid, and what our code should look like.

## Language and Package Manager

- **Python 3.12**
- **uv** for all package management (no pip, poetry, conda, or pipenv)
- `pyproject.toml` for all project and tool configuration
- `uv.lock` committed to Git
- Run everything with `uv run` (for example `uv run uvicorn margins.main:app --reload`)

## Web Framework and Libraries

| Purpose | Library | Notes |
| --- | --- | --- |
| Web framework | **FastAPI** with Pydantic v2 | Request and response validation through Pydantic models |
| Frontend | **Jinja2** server-rendered templates | Plain HTML and CSS, small amounts of vanilla JavaScript; no separate frontend build |
| Database | **SQLite** through **SQLAlchemy 2.0** | One file, `data/margins.db`, gitignored |
| Vector search | **Chroma** (local persistent client) | Stored in `data/chroma/`, gitignored |
| LLM client | **openai** Python SDK pointed at Ollama's OpenAI-compatible endpoint | Same client works for the hosted fallback by changing base URL and key |
| HTTP client | **httpx** | Used to fetch public course websites |
| HTML parsing | **BeautifulSoup4** | Extracts syllabus, schedule, and lecture links from course pages |
| PDF text | **pypdf** | Text-based PDFs only; no OCR |
| Password hashing | **argon2-cffi** | |
| Sessions | Starlette `SessionMiddleware` (signed cookies) | Secret key from `.env` |

## Models and Provider

- **Ollama**, running locally at `http://localhost:11434`, called through its OpenAI-compatible API (`/v1`)
- **Agent model:** a tool-calling instruction model, Qwen3 (8B) or Llama 3.1 (8B); final choice after testing tool-call reliability during Construction
- **Embedding model:** `nomic-embed-text`
- **Fallback:** if local models are too slow or unreliable at tool calling, switch to a hosted OpenAI-compatible provider by changing `LLM_BASE_URL`, `LLM_API_KEY`, and `LLM_MODEL` in `.env`. No code changes.
- All model access goes through one module, `margins/llm/client.py`, so nothing else imports the SDK directly.

## Cloud Provider and Deployment Model

- **No cloud provider. The product runs locally only.**
- Each team member runs the FastAPI app on their own laptop (`localhost:8000`) and Ollama on the same machine.
- Data lives in local files under `data/` (SQLite database, Chroma index, uploaded notes).
- The only outbound network calls are fetching public course websites and, if enabled, the hosted LLM fallback.
- No Docker, hosting, or infrastructure-as-code is required for the MVP.

## Project Structure

```text
margins/
  main.py            # FastAPI app factory, routers, middleware
  api/               # Routers: auth, courses, notes, guides, chat
  agent/             # Agent loop and tool definitions
  llm/client.py      # The only place that talks to Ollama or a hosted provider
  services/          # Business logic: retrieval, weighting, course-site parsing
  db/                # SQLAlchemy models and session
  templates/         # Jinja2 templates
tests/               # pytest tests mirroring the margins/ layout
data/                # Local runtime data (gitignored)
```

## Testing

- **pytest** for all tests, with FastAPI's `TestClient` for endpoints
- **pytest-cov**, target 80% line coverage on `margins/` excluding templates
- **ruff** for linting and formatting
- Unit tests never call a real model: the LLM client is replaced with a fake that returns scripted responses and tool calls
- A small evaluation set (real notes from one course with planted gaps and conflicts) checks the study-guide agent end to end; it runs manually against Ollama, not in the unit test suite

## Do NOT Use

| Prohibited | Reason | Use Instead |
| --- | --- | --- |
| LangChain, LlamaIndex | Heavy abstractions that hide the agent loop we need to understand and grade | A small hand-written agent loop using the OpenAI-compatible tool-calling API |
| `requests` | Blocks FastAPI's async event loop | `httpx` |
| `eval()`, `exec()`, or running code from model output | Arbitrary code execution | Structured tool calls validated with Pydantic |
| Flask, Django | Project uses FastAPI | FastAPI |
| React, Vue, or any frontend build step | Adds a second toolchain for a local MVP | Jinja2 templates |
| Pinecone, Weaviate Cloud, or other hosted vector databases | Product must run locally without accounts | Chroma (local) |
| PostgreSQL, MySQL | Needs a database server on every laptop | SQLite |
| Selenium, Playwright for scraping | Heavy, and course sites in scope are public static pages | `httpx` + BeautifulSoup4 |
| pip, poetry, pipenv, conda | Project uses uv exclusively | uv |
| black, flake8, isort | Replaced by ruff | ruff |
| passlib, plain or MD5/SHA password hashes | passlib is unmaintained; fast hashes are insecure | argon2-cffi |

## Security Basics

- **Auth:** local username and password accounts; passwords hashed with Argon2; signed session cookies. Roles are `student` and `instructor`; only instructors can endorse notes, checked in code, never by the LLM.
- **Secrets:** API keys and the session secret live in a gitignored `.env`, loaded with pydantic-settings. A committed `.env.example` lists the variable names with no values. No secrets in code or Git history.
- **Uploads:** only `.md`, `.txt`, and text-based `.pdf`; maximum 5 MB per file; files stored under generated names, never the user's filename.
- **Course-site fetching:** `http`/`https` URLs only; reject `localhost` and private IP ranges; 10-second timeout; 2 MB response limit; follow at most 3 redirects and stay on the course site's domain for linked pages.
- **Prompt injection:** note text and fetched web pages are untrusted data. They go into prompts inside clearly marked data sections, and the agent can only call the tools we define. Tool arguments are validated with Pydantic before running.
- **Input validation:** every request body is a Pydantic model; database access only through SQLAlchemy (no string-built SQL).
- **Privacy:** uploaded notes are visible only to members of that course; no notes leave the laptop unless the hosted fallback is turned on.

## Example Code Patterns

An endpoint should follow this structure:

```python
from fastapi import APIRouter, Depends, HTTPException, UploadFile
from sqlalchemy.orm import Session

from margins.api.deps import get_current_user, get_db
from margins.api.schemas import NoteOut
from margins.db.models import User
from margins.services.notes import NoteUploadError, save_note

router = APIRouter(prefix="/courses/{course_id}/notes", tags=["notes"])


@router.post("", response_model=NoteOut, status_code=201)
async def upload_note(
    course_id: int,
    file: UploadFile,
    lecture_number: int | None = None,
    user: User = Depends(get_current_user),
    db: Session = Depends(get_db),
) -> NoteOut:
    """Upload a typed note to a course. The service checks membership, type, and size."""
    try:
        note = await save_note(db, course_id=course_id, user=user, file=file, lecture_number=lecture_number)
    except NoteUploadError as e:
        raise HTTPException(status_code=e.status_code, detail=e.message) from e
    return NoteOut.model_validate(note)
```

An agent tool function should follow this structure:

```python
from pydantic import BaseModel, Field

from margins.services.vector_store import VectorStore


class RetrievePassagesArgs(BaseModel):
    """Arguments the agent sends when it calls retrieve_passages."""

    query: str = Field(min_length=1, max_length=500)
    lecture_number: int | None = None
    top_k: int = Field(default=8, ge=1, le=20)


class Passage(BaseModel):
    note_id: int
    passage_id: str
    text: str
    score: float


def retrieve_passages(store: VectorStore, course_id: int, args: RetrievePassagesArgs) -> list[Passage]:
    """Return the most relevant passages, ranked by similarity times the note's quality weight."""
    hits = store.search(course_id=course_id, query=args.query, lecture=args.lecture_number, limit=args.top_k * 2)
    ranked = sorted(hits, key=lambda h: h.similarity * h.quality_weight, reverse=True)
    return [
        Passage(note_id=h.note_id, passage_id=h.passage_id, text=h.text, score=h.similarity * h.quality_weight)
        for h in ranked[: args.top_k]
    ]
```

A test should follow this structure:

```python
from margins.agent.tools import RetrievePassagesArgs, retrieve_passages
from tests.fakes import FakeHit, FakeVectorStore


def test_endorsed_note_outranks_slightly_more_similar_note() -> None:
    store = FakeVectorStore(
        [
            FakeHit(note_id=1, passage_id="1-0", text="Bayes rule ...", similarity=0.82, quality_weight=1.0),
            FakeHit(note_id=2, passage_id="2-0", text="Bayes rule ...", similarity=0.78, quality_weight=1.5),
        ]
    )

    result = retrieve_passages(store, course_id=7, args=RetrievePassagesArgs(query="Bayes rule", top_k=2))

    assert [p.note_id for p in result] == [2, 1]


def test_top_k_limits_results() -> None:
    hits = [FakeHit(note_id=i, passage_id=f"{i}-0", text="x", similarity=0.5, quality_weight=1.0) for i in range(10)]

    result = retrieve_passages(FakeVectorStore(hits), course_id=7, args=RetrievePassagesArgs(query="x", top_k=3))

    assert len(result) == 3
```
