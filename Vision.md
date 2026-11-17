# Margins: Vision Document

## 1. What we are building and for whom

### Summary

Margins is a locally run web platform where students in the same course pool their lecture notes, and an AI agent turns that shared pool into a study guide grounded in the course itself. The agent reads every set of notes uploaded to a class, fetches the public course website to compare them against the syllabus and schedule, and produces a study guide that merges overlapping notes, flags disagreements between notes, and points out lectures that nobody has covered. Professors and TAs can endorse notes, and the agent weights endorsed and highly rated notes more heavily, both when building guides and when answering questions in a chat that cites its sources.

### The problem

Every student in a lecture leaves with different notes. Before an exam, students trade screenshots and shared documents and then reconcile them by hand: deciding which version of a definition is right, discovering that nobody has notes for a missed lecture, and guessing whether any of it matches what the syllabus says will be tested. Existing note-sharing tools store files but do not do this reconciliation, and generic AI chat tools answer from general knowledge rather than from this course's lectures, so they can contradict what the professor actually taught.

### Why an LLM

The inputs are unstructured and written in many voices. Merging three paraphrases of the same concept, detecting that two notes state a formula differently, and mapping a note titled "Tuesday stuff" to the correct lecture on the schedule are semantic tasks that keyword search and templates cannot do reliably. The product's value depends on grounding: every claim the agent makes must trace to a specific uploaded note, so students can verify it and instructors can trust it.

### Target users

| User | Description | What they need |
|------|-------------|----------------|
| **Student contributor** | Enrolled student who takes notes and uploads them | An easy way to upload notes, assign them to lectures, and see how their notes compare to the class's |
| **Student studier** | Student preparing for an exam, possibly with missing lectures | A single, trustworthy study guide that fills gaps in their own notes and shows where sources disagree |
| **Instructor (professor or TA)** | Teaches the course and wants students to study accurate material | A way to endorse good notes, see which lectures lack coverage, and see where students' notes conflict |

The initial audience is students at a university taking lecture-based courses (the team will use real course structures such as their own syllabi as test data). Margins is designed around one course at a time.

### What success looks like

- A student can upload their notes, generate a study guide for a set of lectures, and trust it because every statement links back to the notes it came from.
- The guide clearly shows what is agreed on, what is disputed, and what is missing, instead of presenting a blended answer that hides conflicts.
- An instructor's endorsement visibly changes what the agent prioritizes.

## 2. MVP features in scope

Each feature must work end to end the way a real user would need it, not as a stub.

1. **Accounts and roles.** Simple local accounts. A user can create or join a course and has a role of student or instructor. Role determines who can endorse notes.
2. **Course structure from the public course website.** An instructor or student enters the URL of the course's public website. The agent fetches and parses the site (syllabus, schedule, and linked lecture materials) and extracts a lecture-by-lecture outline (number, date, title, topics), which the user can review and correct. The site is fetched again only when a user asks to refresh it.
3. **Notes upload and assignment.** Students upload typed notes (Markdown, plain text, or text-based PDF). The system proposes which lecture each note covers; the uploader can confirm or change it.
4. **Ratings and endorsements.** Students can rate other students' notes. Instructors can endorse a note. Endorsements and ratings combine into a quality weight stored per note.
5. **Weighted retrieval.** Notes are split into passages and indexed with embeddings. Retrieval combines semantic relevance with the note's quality weight, so endorsed and highly rated notes rank higher.
6. **Study guide generation (agentic).** For a selected course and lecture range, the agent produces a guide that:
   - merges overlapping content from multiple notes into one section per topic,
   - flags disagreements, showing each version with its source,
   - reports lectures on the schedule that have no notes or thin coverage,
   - cites the source note for every claim.
7. **Cited chat (agentic).** A chat where students ask questions about the course. The agent searches the pooled notes and the fetched course site, answers with citations, and says so plainly when the notes do not support an answer.
8. **Instructor coverage view.** A simple view showing, per lecture, how many notes exist, which are endorsed, and where conflicts were flagged.

### Agentic behavior

The study guide and chat features use an agent that chooses its own steps and calls tools, rather than a single prompt. Planned tools include: `fetch_course_site`, `parse_schedule`, `list_notes_for_lecture`, `retrieve_passages`, `compare_passages`, `find_coverage_gaps`, and `cite_source`. The agent decides which tools to call, in what order, and when it has enough evidence.

### Planned models

The product runs locally. We plan to use Ollama with an open instruction-tuned model that supports tool calling (candidates: Qwen3 or Llama 3.1, final choice after testing) for the agents, and `nomic-embed-text` for embeddings. The model layer sits behind an interface so a hosted provider can be substituted if needed.

## 3. Features explicitly out of scope

These are good ideas that we are deliberately leaving for later so the MVP stays achievable.

- University single sign-on or production-grade authentication and authorization
- OCR of handwritten notes, photographed whiteboards, or scanned PDFs
- Audio or video lecture transcription
- Course sites that require login, such as Canvas or other learning management systems (we demo with one or two courses whose websites are public)
- Continuous or scheduled syncing of a course website (the site is fetched when added and when a user asks for a refresh)
- Multiple courses in a single study guide, or cross-course search
- Quizzes, flashcards, and spaced-repetition features generated from the guide
- Real-time collaborative editing of notes
- Notifications, email, and mobile apps
- Cloud deployment and hosting; the product only runs locally on team laptops
- Plagiarism or academic-integrity checks on uploaded notes
- Fine-tuning or training any model

## 4. Open questions

These are questions the team wants to settle, and some may be answered during AI-DLC Inception.

1. **Weighting formula.** How exactly should instructor endorsement and student ratings combine into a weight? Should an endorsement dominate, or just boost? How do we avoid a few early ratings skewing results?
2. **Conflict detection.** How do we decide that two passages "disagree" rather than simply phrase things differently, and how do we avoid false alarms from a small local model?
3. **Local model quality.** Can a local model reliably call tools and produce well-cited output? If not, do we fall back to a hosted provider, and what changes in the technical environment document?
4. **Lecture assignment.** How should the system handle notes that span several lectures or are not tied to a lecture at all?
5. **Citation granularity.** Should citations point to a note, a passage, or a line range, and how do we display them so students can verify quickly?
6. **Instructor involvement.** In practice, instructors may not endorse anything. What does the product do when no endorsements exist, and should students be able to flag notes as "instructor-verified" through some other route?
7. **Privacy and sharing.** Should uploaded notes be visible to all students in the course by default, and should uploaders be able to withhold their name?
8. **Evaluation.** How will we know the guide is good? We plan a small test set (real notes from a course, with known gaps and conflicts planted) to check that the agent finds them.
9. **Course site variety.** Public course websites differ a lot in structure. How should the agent handle a site where it cannot find a clear schedule, and how much should the user have to correct by hand?
