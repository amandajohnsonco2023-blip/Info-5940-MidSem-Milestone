# Unit of Work Plan: Margins

**Role**: Software architect
**Inputs**: `requirements.md`, `stories.md` (39 stories, E1-E10), `application-design/` (approved: components C1-C16, dependency matrix, build order), `execution-plan.md`
**Outputs**: `aidlc-docs/inception/application-design/unit-of-work.md`, `unit-of-work-dependency.md`, `unit-of-work-story-map.md`

**Context.** Margins is one FastAPI application (a modular monolith). Here a **unit of work** is a group of components and stories that one teammate can own and build in Construction, inside the shared app, not a separately deployed service. The team has five members: Nancy, Amanda, Ram, Namra and Anushka. Construction starts after the mid-semester milestone.

---

## Part A: Questions

Fill in each `[Answer]:` tag with a letter. If no option fits, choose the last option (Other) and describe what you want after the tag. Where I have a recommendation it is marked **(Recommended)** with a one-line reason.

### Candidate 5-unit split (background for Question 1)

| Unit | Components | Epics and stories | Load |
|---|---|---|---|
| **U1 Platform and Access** | C1 Web Shell, C2 Accounts, C3 Courses, C15 Jobs | E1, E2, US-3.5 | Medium; must land first |
| **U2 Content Ingestion** | C5 Sources, C6 Ingestion, C4 Site Import | E4, US-3.1 to US-3.4 | Medium-high (parsing, safe fetching) |
| **U3 Trust and Retrieval** | C7 Trust and Weighting, C8 Retrieval, C11 Conflicts | E5, E6, E8 | Medium; most pure logic and property tests |
| **U4 Agent Core and Study Guides** | C9 Model Client, C10 Agent Core, C12 Guides | E7 | High (agent loop, citations) |
| **U5 Chat, Coverage and Evaluation** | C14 Chat, C13 Coverage, C16 Evaluation | E9, E10 | Medium; depends on U4's loop |

---

### Story Grouping and Business Domain

## Question 1
How should the system be divided into units of work?

A) **Five domain units, one per teammate, as in the table above (Recommended)**. *Reason: matches the five-person team and the approved component boundaries, and puts each epic in exactly one unit except the website epic, which splits between U1 (browsing) and U2 (import).*

B) **Four units**: merge U3 and U5 (trust, retrieval, conflicts, coverage, chat), with two people pairing on the biggest unit.

C) **Vertical slices by journey**: (1) key journey (sign up to cited chat), (2) instructor setup, (3) study guides, (4) trust and conflicts, (5) coverage. Each slice touches several components.

D) **One unit per epic** (10 units), assigned round-robin.

E) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
Where should Conflicts (C11: compare, decide majority, resolve) live?

A) **With Trust and Retrieval in U3 (Recommended)**. *Reason: conflicts write agreement records that feed weights, and the majority and precedence rules are pure logic next to the weight formula.*

B) **With Agent Core and Study Guides in U4**, because `compare_passages` is an agent tool.

C) **With Coverage in U5**, because instructors resolve conflicts in the coverage view.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
How should the Should and Could stories (13 Should, 3 Could) be handled?

A) **Assign every story to a unit, keep its priority label, and build Must stories first within each unit (Recommended)**. *Reason: nothing is lost, and each owner knows what to cut if time runs short.*

B) Assign only Must stories to units; keep Should and Could in a separate backlog.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

### Dependencies and Integration

## Question 4
How should units that depend on each other be built in parallel?

A) **Contracts first, then parallel (Recommended)**: U1 lands the app skeleton (auth dependencies, database setup, job runner) and U4 lands the `ModelClient` protocol with `FakeModelClient` first. Each unit publishes its service interfaces (from `component-methods.md`) as stubs early, so others build against them. *Reason: lets all five work at once without waiting for finished code.*

B) **Strict sequence**: follow the build order (U1, then U2 and U3, then U4, then U5); later units wait for earlier ones.

C) **Fully parallel from day one**, each unit mocking whatever it needs, integrating at the end.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
Should the team build a thin end-to-end version of the key journey before finishing units?

A) **Yes, a walking skeleton first (Recommended)**: an early milestone where sign-up, join, upload and a basic cited chat answer work end to end (each unit contributes its piece), then units fill in the rest. *Reason: the key journey is the demo, and integration problems show up early.*

B) No; each unit completes its stories, then everything is integrated.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
Who owns the SQLModel table definitions?

A) **Each unit owns the tables of its components (Recommended)**, per the entity sketch in `components.md` section 4; other units only read them through services or as leaf modules. *Reason: clear ownership, no single person blocking everyone.*

B) One person writes all the table models up front in a single `models` module, and everyone builds on it.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

### Team Alignment

## Question 7
Should this plan name an owner for each unit now?

A) **Yes, record owners now** (tell me who owns which unit after the [Answer]: tag).

B) **No, write "Owner: TBD" and let the team assign units at the next team meeting (Recommended)**. *Reason: ownership affects all five teammates, and they haven't seen the units yet.*

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 8
How should code from different units be integrated?

A) **Short-lived feature branches per story or small group of stories, merged to `main` through PRs reviewed by another teammate, with `main` always runnable and tests green (Recommended)**. *Reason: matches the README collaboration plan and avoids a painful big merge.*

B) One long-lived branch per unit, merged when the unit is done.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
Who owns cross-unit testing (integration tests across units and the evaluation harness C16)?

A) **The U5 owner owns the evaluation harness and the end-to-end integration tests; every unit owns its own unit and property-based tests (Recommended)**. *Reason: U5 sits at the end of the dependency chain and sees the whole flow.*

B) Everyone shares integration testing; no single owner.

C) A rotating "integration owner" each week.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

### Technical Considerations and Code Organization

## Question 10
What is the deployment model and code layout for the units?

A) **One repository, one FastAPI app, one process; each unit owns packages inside `app/` following the approved hybrid layout, with tests mirrored under `tests/` (Recommended)**. *Reason: matches the local-only requirement (NFR-1) and the application design.*

B) Separate Python packages per unit inside the repo, each with its own `pyproject.toml`.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 11
Which units go through the NFR Requirements and NFR Design stages in Construction? (The execution plan says: units with LLM, fetching, auth or upload work.)

A) **U1 (auth), U2 (uploads and fetching), U4 (LLM and agent) and U5 (chat uses the agent); U3 skips them (Recommended)**. *Reason: U3 is pure logic and data with no LLM calls of its own except conflict grouping, which runs through U4's model client.*

B) All five units.

C) Only U2 and U4.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Answer Summary and Analysis

Answers were given interactively in a chat session with the AI on 2026-11-19 and recorded in the `[Answer]:` tags above.

| Q | Answer | Decision |
|---|---|---|
| 1 | A | Five domain units: U1 Platform and Access, U2 Content Ingestion, U3 Trust and Retrieval, U4 Agent Core and Study Guides, U5 Chat, Coverage and Evaluation |
| 2 | A | Conflicts (C11) belong to U3 |
| 3 | A | All 39 stories assigned with priority labels; Must stories first within each unit |
| 4 | A | Contracts first (U1 skeleton, U4 model client and fake, service stubs), then parallel |
| 5 | A | Walking-skeleton milestone for the key journey before units are finished |
| 6 | A | Each unit owns the tables of its components |
| 7 | B | Owners TBD; assigned by the team at the next meeting |
| 8 | A | Short-lived feature branches, PRs reviewed by a teammate, `main` always runnable |
| 9 | A | U5 owns end-to-end integration tests and the evaluation harness; each unit owns its unit and property tests |
| 10 | A | One repo, one FastAPI app and process; units own packages inside `app/`; tests mirrored under `tests/` |
| 11 | A | NFR Requirements and NFR Design run for U1, U2, U4, U5; U3 skips them |

**Analysis (Step 7).** No vague, combined or contradictory answers. Q11 is consistent with the execution plan: U3's only LLM use (conflict grouping in C11) goes through U4's model client, whose NFR decisions (timeouts, retries) cover it. Q4 and Q5 fit together: the walking skeleton is the first milestone that uses the published contracts.

**Follow-up questions:** none needed.

---

## Part B: Generation Checklist

Executed after all answers are in and this plan is approved.

- [x] 1. Apply answers Q1 to Q11 to define the units
- [x] 2. Generate `aidlc-docs/inception/application-design/unit-of-work.md`: unit definitions, components, responsibilities, owned tables, priority order within each unit, owner field
- [x] 3. Greenfield: document the code organization strategy in `unit-of-work.md` (directory layout per unit, shared core rules, test layout)
- [x] 4. Generate `aidlc-docs/inception/application-design/unit-of-work-dependency.md`: dependency matrix between units, contracts each unit publishes, integration sequence and milestones
- [x] 5. Generate `aidlc-docs/inception/application-design/unit-of-work-story-map.md`: every one of the 39 stories assigned to exactly one primary unit (plus contributing units where a story spans several)
- [x] 6. Validate unit boundaries and dependencies (no cycles; each component in exactly one unit)
- [x] 7. Verify all 39 stories are assigned
