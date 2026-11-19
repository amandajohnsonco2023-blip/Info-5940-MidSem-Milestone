# Margins: Unit of Work Story Map

**Stage**: INCEPTION - Units Generation

Every one of the 39 stories in `stories.md` is assigned to exactly **one primary unit** (the unit that owns delivering it and its acceptance criteria). **Contributing units** provide a contract or service the story needs. Priorities are kept from `stories.md`; Must stories are built first in each unit (Q3: A). **M1** marks stories that are part of the walking skeleton (Q5: A).

Units: **U1** Platform and Access, **U2** Content Ingestion, **U3** Trust and Retrieval, **U4** Agent Core and Study Guides, **U5** Chat, Coverage and Evaluation.

---

## 1. Story Map

| Story | Title | Priority | Primary unit | Contributing units | Key journey | M1 |
|---|---|---|---|---|---|---|
| US-1.1 | Sign up | Must | U1 | | Yes | Yes |
| US-1.2 | Log in and out | Must | U1 | | Yes | Yes |
| US-1.3 | Use my seeded instructor account | Must | U1 | | | Prerequisite |
| US-2.1 | Create a course | Must | U1 | | | Prerequisite |
| US-2.2 | Share and regenerate join codes | Must | U1 | | | |
| US-2.3 | Join a course with a student code | Must | U1 | | Yes | Yes |
| US-2.4 | Join as a TA with the instructor code | Should | U1 | | | |
| US-2.5 | Request a course that isn't on Margins | Could | U1 | | | |
| US-2.6 | Act on course requests | Could | U1 | | | |
| US-3.1 | Import the course outline from the public website | Must | U2 | U1 (outline storage, jobs), U4 (model client) | | Prerequisite |
| US-3.2 | See exams from the course website | Should | U2 | U1 (exam storage) | | |
| US-3.3 | Refresh the course website import | Should | U2 | U1 (jobs) | | |
| US-3.4 | Understand when no schedule was found | Must | U2 | U1 (outline page) | | |
| US-3.5 | Browse the course outline | Must | U1 | U2 (imported data) | Yes | Yes |
| US-4.1 | Upload a note | Must | U2 | U1 (jobs), U4 (embeddings) | Yes | Yes |
| US-4.2 | Tag a note to lectures | Must | U2 | U1 (lectures) | Yes | |
| US-4.3 | Choose how my name appears | Must | U2 | | Yes | |
| US-4.4 | Browse and read class notes | Must | U2 | U3 (badges, upvote counts) | | |
| US-4.5 | Delete my note | Should | U2 | U3 (trust rows), U4 ("(removed note)" citations) | | |
| US-4.6 | Replace my note with a new version | Should | U2 | U3 (weights kept) | | |
| US-4.7 | Upload professor material | Must | U2 | U3 (top weight) | | |
| US-5.1 | Upvote a helpful note | Must | U3 | U2 (sources) | | |
| US-5.2 | Flag a note as matching the slides | Should | U3 | U2 (sources) | | |
| US-5.3 | Endorse a note | Must | U3 | U2 (sources) | | |
| US-6.1 | Understand how sources are ranked | Should | U3 | | | |
| US-6.2 | Notes that agree with the class gain trust over time | Should | U3 | U4 (agent runs `compare_passages`) | | |
| US-7.1 | Generate a study guide for an exam or lecture range | Must | U4 | U1 (scopes, jobs), U3 (retrieval, conflicts) | | |
| US-7.2 | Verify every statement in the guide | Must | U4 | U2 (presenter, note pages) | | |
| US-7.3 | See what nobody covered | Must | U4 | U5 (`gaps.py`), U1 (outline) | | |
| US-7.4 | Generate a topic guide for a course without a schedule | Should | U4 | U1 (no-schedule state) | | |
| US-7.5 | Revisit saved guides and compare versions | Should | U4 | | | |
| US-8.1 | See where sources disagree | Must | U3 | U4 (guide displays conflicts) | | |
| US-8.2 | Resolve a conflict | Should | U3 | U5 (coverage view entry point) | | |
| US-9.1 | See coverage per lecture | Must | U5 | U1 (outline), U2 (tags), U3 (endorsements, conflicts) | | |
| US-9.2 | Analyze the course for conflicts | Should | U5 | U4 (`analyze_course` profile), U3 (conflicts), U1 (jobs) | | |
| US-9.3 | Find notes worth endorsing | Could | U5 | U3 (weights) | | |
| US-10.1 | Ask a question and get a cited answer | Must | U5 | U4 (agent, citations), U3 (retrieval) | Yes | Yes |
| US-10.2 | Be told plainly when the notes don't cover my question | Must | U5 | U4 (citation validator) | Yes | |
| US-10.3 | Keep my chat history private | Should | U5 | U1 (auth) | | |

"Prerequisite" in the M1 column means the story's setup (seeded instructor, created course, imported or minimal outline) is needed for the walking skeleton, as `stories.md` notes for the demo path. For M1 it may be satisfied by seed data rather than the finished story.

---

## 2. Totals by Unit

| Unit | Must | Should | Could | Total | Stories |
|---|---|---|---|---|---|
| U1 Platform and Access | 7 | 1 | 2 | **10** | 1.1, 1.2, 1.3, 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 3.5 |
| U2 Content Ingestion | 7 | 4 | 0 | **11** | 3.1, 3.2, 3.3, 3.4, 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7 |
| U3 Trust and Retrieval | 3 | 4 | 0 | **7** | 5.1, 5.2, 5.3, 6.1, 6.2, 8.1, 8.2 |
| U4 Agent Core and Study Guides | 3 | 2 | 0 | **5** | 7.1, 7.2, 7.3, 7.4, 7.5 |
| U5 Chat, Coverage and Evaluation | 3 | 2 | 1 | **6** | 9.1, 9.2, 9.3, 10.1, 10.2, 10.3 |
| **Total** | **23** | **13** | **3** | **39** | |

Totals match `stories.md` (23 Must, 13 Should, 3 Could).

**Load note.** Story counts are uneven on purpose: U3 and U4 have fewer stories but carry the hardest non-story work (the weight formula and property tests for U3; the model client, agent loop, tools and citation validator for U4). U5 also owns the integration tests and the evaluation harness. U1 and U2 have more stories, but most are standard web and file handling.

---

## 3. Validation

- **All 39 stories assigned:** yes, each to exactly one primary unit (section 2 totals = 39).
- **Each component in exactly one unit:** C1, C2, C3, C15 → U1; C4, C5, C6 → U2; C7, C8, C11 → U3; C9, C10, C12 → U4; C13, C14, C16 → U5 (16 components).
- **Epics:** each epic sits in one primary unit, except E3 (website), which splits between U2 (import, US-3.1 to US-3.4) and U1 (browsing, US-3.5), and E8 (conflicts), which sits in U3 while its display in guides is a U4 contribution.
- **Key journey:** all 9 key-journey stories are Must and are covered by U1, U2 and U5, with U3 and U4 contributing retrieval and the agent; the M1 walking skeleton exercises the path end to end.
- **Dependencies:** no blocking cycles between units (see `unit-of-work-dependency.md` section 1).
