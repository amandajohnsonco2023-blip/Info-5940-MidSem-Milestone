# Margins: Component Methods

**Stage**: INCEPTION - Application Design

Method signatures for each component, with one-line purposes and input/output types. These are interface contracts, not implementations. **Business rules** (weight constants, smoothing priors, thresholds, chunk sizes, step caps, prompt wording) **are defined later in Functional Design** for each unit.

**Conventions**
- Services take a SQLModel `Session` in their constructor (Q8: A): `NoteService(session)`.
- `viewer: User` is the signed-in user. Services re-check the specific permission and raise `PermissionDenied`, `NotFound` or `ValidationFailed`; C1 maps these to HTTP 403, 404 and 422.
- `PublicSource` is the anonymity-safe view from C5. Templates and agent tools never receive a raw `Source` with the uploader's name.
- Types in `CamelCase` without a prefix are Pydantic models or SQLModel tables to be detailed in Functional Design.

---

## C1: Web Shell and Auth (`app/web/`)

```python
def get_current_user(request: Request, session: Session) -> User | None
    # Read the signed session cookie and load the user.
def require_user(user: User | None = Depends(get_current_user)) -> User
    # Redirect to /login when not signed in (FR-1.4).
def require_instructor_flag(user: User = Depends(require_user)) -> User
    # 403 unless the account has the seeded instructor flag.
def require_member(course_id: int, user: User = Depends(require_user), session: Session = ...) -> CourseContext
    # 403 unless the user belongs to the course; returns CourseContext(course, user, role).
def require_course_instructor(ctx: CourseContext = Depends(require_member)) -> CourseContext
    # 403 unless role is course instructor (professor or TA).
def login_user(response: Response, user: User) -> None
def logout_user(response: Response) -> None
def render_markdown(text: str) -> Markup
    # markdown-it-py with raw HTML disabled; the result is safe because HTML is escaped.
```

## C2: Accounts (`app/accounts/`)

```python
class AccountService:
    def sign_up(self, form: SignUpForm) -> User                      # student account; raises EmailTaken
    def authenticate(self, email: str, password: str) -> User | None # None on any failure (one generic message)
    def get(self, user_id: int) -> User

def hash_password(password: str) -> str
def verify_password(password: str, password_hash: str) -> bool
def seed_instructors(session: Session, list_file: Path) -> SeedReport  # CLI: create/flag instructor accounts
```

## C3: Courses and Membership (`app/courses/`)

```python
class CourseService:
    def create_course(self, creator: User, form: CourseForm) -> Course          # instructor flag required
    def get_course(self, course_id: int) -> Course
    def list_my_courses(self, user: User) -> list[CourseWithRole]
    def regenerate_join_code(self, ctx: CourseContext, kind: JoinCodeKind) -> str  # "student" | "instructor"

class MembershipService:
    def join_by_code(self, user: User, code: str) -> Membership                 # role decided by which code matched
    def get_role(self, user: User, course_id: int) -> CourseRole | None

class CourseRequestService:
    def submit(self, user: User, form: CourseRequestForm) -> CourseRequest
    def list_open(self, viewer: User) -> list[CourseRequest]                    # instructor flag required
    def create_from_request(self, viewer: User, request_id: int, form: CourseForm) -> Course
    def dismiss(self, viewer: User, request_id: int) -> None

class CourseStructureService:
    def get_outline(self, course_id: int) -> CourseOutline                     # lectures, exams, has_schedule
    def replace_structure(self, course_id: int, outline: ExtractedOutline | None) -> None  # None = no schedule
    def lectures_for_scope(self, course_id: int, scope: GuideScope) -> list[Lecture]
        # exam -> its lecture range; custom range; whole course -> [] with topic mode

def select_exam_lectures(lectures: list[Lecture], exam: Exam) -> list[Lecture]   # pure, property-testable
```

## C4: Course Site Import (`app/ingestion/site/`)

```python
class SiteImportService:
    def start_import(self, ctx: CourseContext, background: BackgroundTasks) -> Job   # first add or Refresh
    def import_site(self, course_id: int, job_id: int) -> ImportResult             # runs inside the job

class SafeFetcher:
    def fetch(self, url: str) -> FetchedDocument          # raises UnsafeUrl, FetchTooLarge, FetchTimeout

def validate_public_url(url: str, resolve: Callable[[str], list[IPAddress]]) -> ValidatedUrl
    # pure given the resolver; http/https only; refuses private, loopback, link-local
def same_site_links(base_url: str, html: str, max_pages: int) -> list[str]      # pure
def extract_html_text(html: str) -> str
def extract_pdf_text(data: bytes) -> str

class CourseStructureExtractor:
    def extract(self, pages: list[FetchedDocument]) -> ExtractedOutline | None
        # model.generate_structured -> ExtractedOutline(lectures, exams); None when no schedule found
```

## C5: Sources (`app/sources/`)

```python
class SourceService:
    def upload(self, ctx: CourseContext, upload: UploadedFile, form: UploadForm,
               kind: SourceKind, background: BackgroundTasks) -> tuple[Source, Job]
        # kind: student_note | professor_material (instructor only); validates type and 5 MB cap
    def add_website_source(self, course_id: int, doc: FetchedDocument) -> Source     # called by C4
    def remove_website_sources(self, course_id: int) -> None                          # on refresh
    def set_lecture_tags(self, ctx: CourseContext, source_id: int, lecture_ids: list[int]) -> None  # [] = untagged
    def set_anonymity(self, ctx: CourseContext, source_id: int, anonymous: bool) -> None
    def replace_file(self, ctx: CourseContext, source_id: int, upload: UploadedFile,
                     background: BackgroundTasks) -> Job                               # keeps ID, ratings, endorsement
    def delete(self, ctx: CourseContext, source_id: int) -> None                       # uploader only; cascades
    def list_sources(self, ctx: CourseContext, filters: SourceFilters) -> list[PublicSource]
    def get_source(self, ctx: CourseContext, source_id: int) -> PublicSourceDetail

def present_source(source: Source, viewer_role: CourseRole) -> PublicSource
    # the ONLY place uploader names are exposed; "Anonymous classmate" unless viewer is a course instructor
```

## C6: Ingestion Pipeline (`app/ingestion/`)

```python
class IngestionService:
    def process_source(self, source_id: int, job_id: int) -> ProcessingResult
        # parse -> chunk -> embed -> store passages -> lecture suggestion (student notes only)
    def suggest_lectures(self, source_id: int) -> list[int]       # [] when the model is unsure

def parse_file(filename: str, data: bytes) -> str                 # raises UnsupportedFile, NoExtractableText
def chunk_text(text: str, max_chars: int, overlap: int) -> list[Chunk]
    # pure; property targets: chunks cover the whole text, round-trip, size bounds
```

## C7: Trust and Weighting (`app/trust/`)

```python
class TrustService:
    def toggle_upvote(self, ctx: CourseContext, source_id: int) -> bool          # not on own note; returns new state
    def toggle_slides_flag(self, ctx: CourseContext, source_id: int) -> bool
    def set_endorsement(self, ctx: CourseContext, source_id: int, endorsed: bool) -> None  # course instructor
    def record_agreement(self, conflict_id: int, outcomes: list[AgreementOutcome]) -> None # called by C11
    def course_has_endorsements(self, course_id: int) -> bool

class WeightService:
    def recompute(self, source_id: int) -> float                 # called after every input change
    def get_weights(self, source_ids: list[int]) -> dict[int, SourceWeight]
    def ranking_explanation(self) -> RankingExplanation           # for the "How sources are ranked" page

# weighting.py (pure, property-testable)
def smoothed_score(positives: int, total: int, prior_mean: float, prior_strength: float) -> float
def compute_weight(inputs: WeightInputs, config: WeightConfig) -> float
    # property targets: bounded, monotonic in each signal, professor material at the top
def precedence_rank(kind: SourceKind, endorsed: bool, in_majority: bool,
                    slides_flagged: bool, upvote_score: float) -> tuple
    # sort key implementing FR-6.1 order
```

## C8: Retrieval (`app/retrieval/`)

```python
class RetrievalService:
    def search(self, course_id: int, query: str, viewer_role: CourseRole,
               lecture_ids: list[int] | None = None, kinds: set[SourceKind] | None = None,
               top_k: int = 12) -> list[RetrievedPassage]
        # RetrievedPassage(passage_id, text, score, public_source)

def cosine_similarities(query: np.ndarray, matrix: np.ndarray) -> np.ndarray     # pure
def combine_scores(similarity: np.ndarray, weights: np.ndarray, alpha: float) -> np.ndarray  # pure
```

## C9: Model Client (`app/llm/`)

```python
class ModelClient(Protocol):
    def generate(self, messages: list[Message], tools: list[ToolDeclaration] | None = None,
                 system: str | None = None) -> ModelTurn
        # ModelTurn(text: str | None, tool_calls: list[ToolCall])
    def generate_structured(self, messages: list[Message], schema: type[T],
                            system: str | None = None) -> T
    def embed(self, texts: list[str]) -> list[list[float]]

class GeminiClient(ModelClient): ...      # google-genai; timeouts; retries; raises ModelUnavailableError
class FakeModelClient(ModelClient): ...   # scripted turns, structured outputs and deterministic embeddings
```

## C10: Agent Core (`app/agent/`)

```python
class ToolRegistry:
    def register(self, name: str, args_model: type[BaseModel], result_model: type[BaseModel],
                 fn: Callable[[ToolContext, BaseModel], BaseModel], description: str) -> None
    def declarations(self, allowed: set[str]) -> list[ToolDeclaration]   # generated from args_model schema
    def call(self, name: str, raw_args: dict, ctx: ToolContext) -> ToolResult
        # validates raw_args; invalid args or exceptions become ToolResult(error=...) for the model

class AgentRunner:
    def run(self, profile: AgentProfile, ctx: ToolContext, user_message: str,
            on_progress: Callable[[str], None]) -> AgentOutput
        # loop until final text or caps; AgentOutput(text, retrieved_passage_ids, steps, stopped_reason)

class CitationValidator:
    def validate(self, text: str, retrieved: dict[int, RetrievedPassage]) -> CitedOutput
        # CitedOutput(markdown with markers replaced by note-level citations, citations, uncited_sections)
        # raises UngroundedOutput if a required section has no valid citation

@dataclass
class AgentProfile:            # "guide" | "chat" | "analyze_course"
    name: str; system_prompt: str; allowed_tools: set[str]; max_steps: int; max_tool_calls: int

@dataclass
class ToolContext:             # injected by the loop, never from the model
    course_id: int; viewer_role: CourseRole; session: Session; scope_lecture_ids: list[int] | None

# Tools: (args model) -> result model
get_course_outline(GetOutlineArgs) -> CourseOutlineResult
list_sources_for_lecture(ListSourcesArgs(lecture_id)) -> SourceListResult
retrieve_passages(RetrieveArgs(query, lecture_ids?, top_k?)) -> PassageListResult
compare_passages(CompareArgs(topic, passage_ids)) -> ComparisonResult
find_coverage_gaps(CoverageArgs(lecture_ids)) -> CoverageGapResult
get_source_weights(WeightsArgs(source_ids)) -> SourceWeightResult
get_resolved_conflicts(ResolvedArgs(lecture_ids?)) -> ResolvedConflictResult
```

## C11: Conflicts (`app/conflicts/`)

```python
class ConflictService:
    def compare(self, course_id: int, topic: str, passages: list[RetrievedPassage],
                lecture_id: int | None) -> ComparisonResult
        # model.generate_structured -> VersionGrouping; then decide_majority + label_versions; store + agreement
    def list_conflicts(self, ctx: CourseContext, lecture_id: int | None = None,
                       status: ConflictStatus | None = None) -> list[ConflictView]
    def resolve(self, ctx: CourseContext, conflict_id: int, version_id: int, endorse: bool) -> None
    def resolved_for_lectures(self, course_id: int, lecture_ids: list[int] | None) -> list[ResolvedConflict]

# decisions.py (pure, property-testable)
def decide_majority(versions: list[VersionSupport]) -> MajorityDecision
    # counts distinct sources; tie -> no majority label (assumption A3)
def label_versions(versions: list[VersionSupport], decision: MajorityDecision) -> list[LabeledVersion]
    # "Authoritative (instructor material)", "More commonly reported"
```

## C12: Study Guides (`app/guides/`)

```python
class GuideService:
    def start_generation(self, ctx: CourseContext, scope: GuideScope,
                         background: BackgroundTasks) -> Job
    def generate(self, course_id: int, scope: GuideScope, requester_id: int, job_id: int) -> GuideVersion
        # runs the "guide" profile, validates citations, saves a new version
    def list_guides(self, ctx: CourseContext) -> list[GuideSummary]       # grouped by exam
    def get_version(self, ctx: CourseContext, version_id: int) -> GuideVersionView
    def compare_versions(self, ctx: CourseContext, a_id: int, b_id: int) -> VersionDiff
```

## C13: Coverage and Course Analysis (`app/coverage/`)

```python
class CoverageService:
    def coverage_view(self, ctx: CourseContext) -> CoverageView          # course instructors; no-schedule state
    def endorsement_candidates(self, ctx: CourseContext, limit: int = 10) -> list[PublicSource]
    def start_analysis(self, ctx: CourseContext, background: BackgroundTasks) -> Job
    def analyze(self, course_id: int, requester_id: int, job_id: int) -> AnalysisSummary
        # runs the "analyze_course" profile per lecture

# gaps.py (pure, property-testable)
def find_coverage_gaps(lectures: list[tuple[int, str]], note_counts: dict[int, int],
                       thin_threshold: int = 2) -> list[CoverageGap]
```

## C14: Cited Chat (`app/chat/`)

```python
class ChatService:
    def list_threads(self, ctx: CourseContext) -> list[ChatThreadSummary]   # own threads only
    def new_thread(self, ctx: CourseContext) -> ChatThread
    def get_thread(self, ctx: CourseContext, thread_id: int) -> ChatThreadView  # 404 if not the owner
    def ask(self, ctx: CourseContext, thread_id: int, question: str) -> ChatMessage
        # runs the "chat" profile; ungrounded -> "the course materials don't cover this"
```

## C15: Jobs and Progress (`app/jobs/`)

```python
class JobService:
    def create(self, course_id: int, requester_id: int, kind: JobKind) -> Job
    def add_event(self, job_id: int, message: str) -> None
    def succeed(self, job_id: int, result_ref: str | None = None) -> None
    def fail(self, job_id: int, user_message: str) -> None
    def get_status(self, ctx: CourseContext, job_id: int) -> JobStatusView     # polled by HTMX

class JobRunner:
    def start(self, background: BackgroundTasks, job: Job,
              work: Callable[[Session, int], None]) -> None
        # opens its own Session, runs work, marks succeed/fail, never lets an exception escape
```

## C16: Evaluation Harness (`scripts/evaluate.py`)

```python
def load_planted_set(path: Path, session: Session) -> PlantedSet
def run_evaluation(planted: PlantedSet, client: ModelClient) -> EvaluationReport
def main() -> int     # prints found / missed per planted gap and conflict
```
