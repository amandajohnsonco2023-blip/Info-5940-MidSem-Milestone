# Margins: Personas

Four short personas, per story-generation-plan.md (Q2: B, Q3: A). Permissions follow requirements.md section 2. The two student personas are one account type (student). They are split by goal, not by permission.

| ID | Persona | Account type | Course role |
|---|---|---|---|
| **SC** | Student contributor | Student (self sign-up) | Student member |
| **SS** | Student studier | Student (self sign-up) | Student member |
| **IN** | Instructor (professor) | Instructor flag (seeded by the team) | Course instructor (creator) |
| **TA** | Teaching assistant | Student account | Course instructor, through the instructor join code |

---

## SC: Student Contributor

**Role.** An enrolled student who takes typed notes in most lectures and is willing to share them with the class.

**Goals**
- Upload notes quickly after a lecture and attach them to the right lecture
- See that their notes help classmates (upvotes, endorsements, citations in guides)
- Control whether their name appears next to their notes

**Frustrations**
- Sharing notes in group chats where they get lost or reposted without credit
- Worrying that a mistake in their notes will mislead classmates
- Tools that make uploading take longer than writing the notes did

**Permissions in Margins**
- Join a course with the student join code; request a course that doesn't exist yet
- Upload `.md`, `.txt` or text-based `.pdf` notes (5 MB maximum); tag lectures; choose anonymity
- Delete or replace their own notes
- Upvote and flag "matches slides" on other members' notes, never their own
- Generate guides, view saved guides, and use cited chat (same as SS)

---

## SS: Student Studier

**Role.** A student preparing for an exam who may have missed lectures or has thin notes. They may never upload anything.

**Goals**
- Get one study guide for the next exam that they can trust
- See quickly what the class agrees on, what is disputed, and what nobody covered
- Ask specific questions and get answers that come from this course, not the internet

**Frustrations**
- Piles of overlapping, contradictory notes from friends
- Generic AI answers that contradict what the professor actually taught
- Not knowing which version of a definition is correct

**Permissions in Margins**
- Same as SC. Uploading is optional.
- Sees all course notes, with anonymous uploaders shown as "Anonymous classmate"
- Sees every saved guide in the course, but only their own chat history

---

## IN: Instructor (Professor)

**Role.** Teaches a lecture course. Their account was created by the Margins team (no self-declared instructor role). Busy, and will use Margins in short sessions.

**Goals**
- Set up the course page from the existing public course website with minimal effort
- Make sure students study accurate material, mainly by uploading slides or handouts and endorsing good notes
- See which lectures lack notes and where students disagree, and correct mistakes

**Frustrations**
- Students studying from inaccurate shared notes they never see
- Tools that require hours of setup or manual data entry
- Being flooded with notifications or email

**Permissions in Margins**
- Create courses (needs the instructor flag); see and regenerate both join codes; act on course requests
- Refresh the course website import
- Upload professor material (authoritative)
- Endorse notes and remove endorsements
- See real names behind anonymous notes
- Use the coverage view, run "Analyze course", and resolve conflicts
- Everything a student can do

---

## TA: Teaching Assistant

**Role.** A graduate or advanced undergraduate TA for one course. Has an ordinary account and gets instructor powers in that course by joining with the instructor join code.

**Goals**
- Help the professor keep course material accurate without needing the professor's account
- Triage conflicts and endorse notes, especially before exams
- Upload section handouts or worked examples

**Frustrations**
- Being unable to help because only the professor has access
- Answering the same student confusion repeatedly

**Permissions in Margins**
- **Within the joined course only:** the same as IN, except creating courses and acting on course requests (those need the global instructor flag)
- In other courses: student permissions only

---

## Persona-to-Epic Map

✔ = primary persona for stories in that epic; ○ = can also use it.

| Epic | SC | SS | IN | TA |
|---|---|---|---|---|
| E1 Accounts and authentication | ✔ | ✔ | ✔ | ✔ |
| E2 Courses and membership | ✔ | ✔ | ✔ | ✔ |
| E3 Course structure from the website | ○ | ✔ | ✔ | ✔ |
| E4 Notes and professor material | ✔ | ✔ | ✔ | ✔ |
| E5 Upvotes, flags and endorsements | ✔ | ✔ | ✔ | ✔ |
| E6 Ranking transparency | ✔ | ✔ | ○ | ○ |
| E7 Study guides | ○ | ✔ | ○ | ○ |
| E8 Conflicts | ○ | ✔ | ✔ | ✔ |
| E9 Coverage view and course analysis | | | ✔ | ✔ |
| E10 Cited chat | ○ | ✔ | ○ | ○ |

A per-story persona list is in `stories.md`, section "Persona-to-Story Mapping".
