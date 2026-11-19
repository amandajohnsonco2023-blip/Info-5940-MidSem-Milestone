# Margins: User Stories

## How to Read This Document
- **Organization:** one epic per feature area, E1 to E10, matching FR-1 to FR-10 in `requirements.md` (plan Q1: A)
- **Personas:**
  - SC = Student contributor
  - SS = Student studier
  - IN = Instructor (professor)
  - TA = Teaching assistant
  - See `personas.md`.
- **Size:** one story per user goal (Q4: B)
- **Acceptance criteria:** Given/When/Then for the main and edge scenarios, plus a **Rules** checklist for simple constraints (Q5: C). Model-failure and grounding behavior is written inside the relevant stories (Q7: A).
- **Priority:** MoSCoW (Q6: A). Must = needed for the MVP loop; Should = expected in the MVP; Could = include if time allows; Won't = listed at the end.
- **[Key journey]** marks stories on the most important demo path (Q9: B). These are all Must. The path: a student signs up, joins an existing course, uploads notes, and studies with cited chat.

### Key Journey Path
1. US-1.1 Sign up
2. US-1.2 Log in and out
3. US-2.3 Join a course with a student code
4. US-3.5 Browse the course outline
5. US-4.1 Upload a note
6. US-4.2 Tag a note to lectures
7. US-4.3 Choose how my name appears
8. US-10.1 Ask a question and get a cited answer
9. US-10.2 Be told plainly when the notes don't cover my question

**Prerequisite for the demo (not a story, Q8: A):** a seeded instructor has already created the course (US-1.3 and US-2.1) and imported its website (US-3.1).

---

## E1: Accounts and Authentication (FR-1)

### US-1.1 Sign up [Key journey]
**Personas:** SC, SS, TA | **Priority:** Must | **Traces to:** FR-1.1, FR-1.2, NFR-2

As a student, I want to create an account with my email and a password, so that I can join my course on Margins.

**Scenarios**
- **Given** I am not signed in, **when** I submit a valid email, display name and password, **then** my account is created as a student account, I am signed in, and I land on "My courses".
- **Given** an account already exists for that email, **when** I submit the form, **then** I see "An account with this email already exists" and no second account is created.
- **Given** a field is empty or too long, **when** I submit, **then** I see an error next to that field and my other inputs are kept.

**Rules**
- [ ] Passwords have a minimum length (8 characters) and are stored only as bcrypt hashes
- [ ] Sign-up never grants the instructor flag
- [ ] Every field has a Pydantic type and length limit

### US-1.2 Log in and out [Key journey]
**Personas:** SC, SS, IN, TA | **Priority:** Must | **Traces to:** FR-1.2, FR-1.4, FR-2.4

As a Margins user, I want to log in, move between my courses, and log out, so that my courses and notes are only accessible to me.

**Scenarios**
- **Given** a valid email and password, **when** I log in, **then** a signed session cookie is set and I see "My courses", listing every course I belong to with my role in each.
- **Given** a wrong password or unknown email, **when** I log in, **then** I see one generic "Email or password is incorrect" message.
- **Given** I am signed in, **when** I click Log out, **then** my session ends and course pages redirect me to the login page.
- **Given** I am not signed in, **when** I open any course URL directly, **then** I am redirected to the login page.

**Rules**
- [ ] The session cookie is signed with `SESSION_SECRET` from `.env`
- [ ] Neither the session secret nor passwords are ever logged

### US-1.3 Use my seeded instructor account
**Personas:** IN | **Priority:** Must | **Traces to:** FR-1.3, section 2

As an instructor, I want to sign in with the account the Margins team set up for me, so that I can create and manage my course without a separate approval process.

**Scenarios**
- **Given** my account was created by the team's seeding command, **when** I log in, **then** I see a "Create course" action and the "Requested courses" list.
- **Given** a student account, **when** it logs in, **then** neither "Create course" nor "Requested courses" is visible, and their URLs return 403.

**Rules**
- [ ] The instructor flag can only be set by the seeding command, not through any page

---

## E2: Courses and Membership (FR-2)

### US-2.1 Create a course
**Personas:** IN | **Priority:** Must | **Traces to:** FR-2.1, FR-2.2

As an instructor, I want to create a course page with its name, code, term and public website URL, so that my students have one place to pool notes.

**Scenarios**
- **Given** I am an instructor, **when** I submit a course name, code and term, with an optional website URL, **then** the course is created, I am its course instructor, and student and instructor join codes are generated.
- **Given** I entered a website URL, **when** the course is created, **then** the website import (US-3.1) starts automatically.
- **Given** the URL is not a public http/https address, **when** I submit, **then** I see "Enter a public http or https URL" and the course is not created.

**Rules**
- [ ] Only instructor-flagged accounts can create courses
- [ ] Course code plus term must be unique

### US-2.2 Share and regenerate join codes
**Personas:** IN, TA | **Priority:** Must | **Traces to:** FR-2.2

As a course instructor, I want to see and regenerate the course's student and instructor join codes, so that I control who joins and with what role.

**Scenarios**
- **Given** I am a course instructor, **when** I open course settings, **then** I see both join codes, each with a copy button and a short explanation of the role it grants.
- **Given** a code leaked, **when** I click Regenerate for that code, **then** a new code replaces it, and the old code no longer works.

**Rules**
- [ ] Students never see either join code in any page or response

### US-2.3 Join a course with a student code [Key journey]
**Personas:** SC, SS | **Priority:** Must | **Traces to:** FR-2.2, FR-2.4

As a student, I want to enter the join code my instructor shared, so that I can see and contribute to my class's notes.

**Scenarios**
- **Given** a valid student join code, **when** I enter it, **then** I become a student member and I land on the course page.
- **Given** I am already a member, **when** I enter the code again, **then** I see "You're already in this course" and go to the course page.
- **Given** an invalid or regenerated code, **when** I enter it, **then** I see "That code doesn't match any course" and nothing changes.

**Rules**
- [ ] Course pages and data are visible only to members (NFR-7)

### US-2.4 Join as a TA with the instructor code
**Personas:** TA | **Priority:** Should | **Traces to:** FR-2.2, section 2

As a TA, I want to join my course using the instructor join code, so that I can endorse notes and manage conflicts without using the professor's account.

**Scenarios**
- **Given** a valid instructor join code, **when** I enter it, **then** I become a course instructor of that course only, and "My courses" shows my role as "Course instructor".
- **Given** I am already a student member, **when** I enter the instructor code, **then** my role in that course is upgraded to course instructor.
- **Given** I am a course instructor in course A, **when** I open course B where I am a student, **then** I have only student permissions there.

**Rules**
- [ ] Joining as a TA does not grant the global instructor flag (no course creation)

### US-2.5 Request a course that isn't on Margins
**Personas:** SC, SS | **Priority:** Could | **Traces to:** FR-2.5

As a student, I want to request a course page by giving the course name, website and instructor email, so that an instructor can set it up.

**Scenarios**
- **Given** I am signed in, **when** I submit the request form with the course name, website URL and instructor email, **then** I see "Request sent to Margins instructors" and the request appears on the Requested courses list.
- **Given** a field is missing or invalid, **when** I submit, **then** I see field-level errors.

**Rules**
- [ ] No email or notification is sent
- [ ] Students can see the status of their own requests (open, created, dismissed)

### US-2.6 Act on course requests
**Personas:** IN | **Priority:** Could | **Traces to:** FR-2.5

As an instructor, I want to see requested courses and create or dismiss them, so that students' requests are acted on.

**Scenarios**
- **Given** open requests exist, **when** I open Requested courses, **then** I see each request's course name, URL, instructor email and date.
- **Given** an open request, **when** I click Create course, **then** the create form opens prefilled from the request, and saving marks the request "created".
- **Given** an open request, **when** I click Dismiss, **then** it is marked "dismissed" and leaves the open list.

---

## E3: Course Structure from the Website (FR-3)

### US-3.1 Import the course outline from the public website
**Personas:** IN, TA | **Priority:** Must | **Traces to:** FR-3.1, FR-3.2, FR-3.3, FR-3.7, NFR-2, NFR-4

As a course instructor, I want Margins to read our public course website and build the lecture outline, so that I don't have to type the schedule in.

**Scenarios**
- **Given** a public course website with a schedule, **when** the import runs, **then**:
  - I see step-by-step progress, for example "Fetching schedule page...", "Reading linked PDFs (3 of 8)..."
  - the course shows a lecture outline with number, date, title and topics for each lecture
- **Given** the site links to same-site pages and PDFs, **when** the import runs, **then** pages one level deep and linked PDFs are fetched up to the page cap, and their text becomes searchable "Course website" sources.
- **Given** a link points to a private, loopback or link-local address, or to a non-http(s) scheme, **when** the import runs, **then** that link is skipped and logged, and the import continues.
- **Given** the site times out or is unreachable, **when** the import runs, **then** I see "We couldn't reach the course website. Try Refresh later." and the course stays usable.
- **Given** the Gemini API fails during extraction, **when** the import runs, **then** I see a retryable error and the previous outline (if any) is kept.

**Rules**
- [ ] Timeouts and per-response size caps apply to every fetch
- [ ] Website text is passed to the model as content, never as instructions
- [ ] Website sources rank like ordinary notes, not like professor material

### US-3.2 See exams from the course website
**Personas:** SS, IN, TA | **Priority:** Should | **Traces to:** FR-3.3, FR-7.1

As a student, I want the course page to list the exams and which lectures they cover, so that I can study for the right range.

**Scenarios**
- **Given** the site lists exams such as "Prelim 1: Lectures 1-10", **when** the import finishes, **then** the course page lists each exam's name, date and lecture range.
- **Given** an exam's range can't be determined, **when** it is listed, **then** it shows "Lecture range unknown" and cannot be used as a guide scope.
- **Given** the site lists no exams, **when** the import finishes, **then** no exam section is shown.

### US-3.3 Refresh the course website import
**Personas:** IN, TA | **Priority:** Should | **Traces to:** FR-3.4

As a course instructor, I want to re-import the website when the schedule changes, so that the outline and exams stay current.

**Scenarios**
- **Given** I am a course instructor, **when** I click Refresh, **then** the import runs again and replaces the outline, exams and website sources.
- **Given** notes were tagged to a lecture that still exists after refresh (same number), **when** the refresh finishes, **then** the tags are kept.
- **Given** a tagged lecture no longer exists, **when** the refresh finishes, **then** the affected notes become untagged and their uploaders see a "Re-tag needed" badge.

**Rules**
- [ ] Students cannot trigger Refresh
- [ ] The site is never re-fetched automatically

### US-3.4 Understand when no schedule was found
**Personas:** SC, SS, IN, TA | **Priority:** Must | **Traces to:** FR-3.5, FR-3.6

As a course member, I want a clear message when the website has no usable schedule, so that I understand why lecture features are missing.

**Scenarios**
- **Given** the import found no lecture schedule, **when** I open the course page, **then** I see "No lecture schedule was found on the course website", with an explanation that guides will be organized by topic and coverage reporting is unavailable.
- **Given** the course has no schedule, **when** I upload a note, **then** no lecture-tag step is shown.

**Rules**
- [ ] The outline cannot be edited by hand; Refresh is the only way to change it
- [ ] The system does not try to infer a schedule from notes

### US-3.5 Browse the course outline [Key journey]
**Personas:** SC, SS, IN, TA | **Priority:** Must | **Traces to:** FR-3.3, FR-4.3

As a student, I want to see the lecture list with the notes tagged to each lecture, so that I can find material for the lecture I missed.

**Scenarios**
- **Given** the course has an outline, **when** I open the course page, **then** I see each lecture's number, date, title and topics, and the count of notes tagged to it.
- **Given** I click a lecture, **when** it opens, **then** I see its tagged notes and professor material, with badges (Instructor material, Instructor-endorsed, Matches slides) and upvote counts.
- **Given** untagged notes exist, **when** I view the course, **then** they are listed under "Not tied to a lecture".

---

## E4: Notes and Professor Material (FR-4)

### US-4.1 Upload a note [Key journey]
**Personas:** SC, SS, TA | **Priority:** Must | **Traces to:** FR-4.1, FR-4.7, NFR-2

As a student contributor, I want to upload my typed notes with a title, so that the class and the study tools can use them.

**Scenarios**
- **Given** a `.md`, `.txt` or text-based `.pdf` file of 5 MB or less and a title, **when** I upload it, **then** the note is saved, split into passages and indexed, and I go to the lecture-tag step (US-4.2).
- **Given** a file over 5 MB, **when** I upload it, **then** I see "Notes must be 5 MB or smaller." and nothing is saved.
- **Given** an unsupported type, or a PDF with no extractable text (for example a scan), **when** I upload it, **then** I see a clear message naming the problem and nothing is saved.
- **Given** the embedding call to Gemini fails, **when** I upload, **then** the note is saved as "Indexing pending", I see a retryable message, and the note is excluded from search until indexing succeeds.

**Rules**
- [ ] The title is 1 to 200 characters
- [ ] Note text is rendered as escaped Markdown (raw HTML off)
- [ ] Note content is passed to the model as content, never as instructions

### US-4.2 Tag a note to lectures [Key journey]
**Personas:** SC, TA, IN | **Priority:** Must | **Traces to:** FR-4.3, FR-4.4

As a student contributor, I want Margins to suggest which lecture my note covers and let me confirm, change or skip it, so that my note counts toward the right lectures.

**Scenarios**
- **Given** the course has an outline, **when** the upload finishes, **then** I see a suggested lecture (or "No suggestion" if the system is unsure), plus options to accept it, choose one or more lectures, or leave the note untagged.
- **Given** I leave the note untagged, **when** I save, **then** the note is still searchable and usable in guides and chat, but it does not count toward any lecture's coverage.
- **Given** the suggestion call fails or times out, **when** the upload finishes, **then** I see "No suggestion" and can tag by hand.
- **Given** I own a note, **when** I edit its tags later, **then** coverage counts update.

### US-4.3 Choose how my name appears [Key journey]
**Personas:** SC, TA | **Priority:** Must | **Traces to:** FR-4.5, NFR-7

As a student contributor, I want to choose between showing my name and appearing as "Anonymous classmate", so that I can share notes without exposing myself.

**Scenarios**
- **Given** I am uploading, **when** I choose "Anonymous classmate", **then** students see that label on the note, in guide citations and in chat citations.
- **Given** an anonymous note, **when** a course instructor views it, **then** they see my real display name.
- **Given** I own a note, **when** I change the setting later, **then** all displays use the new choice, including saved guides when they are viewed again.

**Rules**
- [ ] The real name of an anonymous uploader is never sent to student-facing pages or responses

### US-4.4 Browse and read class notes
**Personas:** SS, SC | **Priority:** Must | **Traces to:** FR-4.5, FR-4.6

As a student studier, I want to browse and open any note in the course, so that I can read sources directly and check citations.

**Scenarios**
- **Given** I am a member, **when** I open a note, **then** I see its title, uploader (or "Anonymous classmate"), source type, lectures, badges, upvote count, "Last updated" time and rendered content.
- **Given** I am not a member of the course, **when** I open a note URL, **then** I get a 403 or 404.

### US-4.5 Delete my note
**Personas:** SC, TA, IN | **Priority:** Should | **Traces to:** FR-4.6

As a student contributor, I want to delete a note I uploaded, so that I can withdraw material I no longer want shared.

**Scenarios**
- **Given** I own a note, **when** I confirm Delete, **then** the note, its passages, upvotes, flags, endorsement and agreement records are removed.
- **Given** saved guide versions cited the note, **when** they are viewed, **then** those citations show "(removed note)".
- **Given** I don't own the note, **when** I try to delete it, **then** the action is not offered and the endpoint returns 403.

### US-4.6 Replace my note with a new version
**Personas:** SC, TA, IN | **Priority:** Should | **Traces to:** FR-4.6

As a student contributor, I want to upload a corrected file for an existing note, so that classmates see my fix without losing its upvotes and endorsement.

**Scenarios**
- **Given** I own a note, **when** I upload a replacement file that passes the same checks as US-4.1, **then** the note keeps its ID, upvotes, flags and endorsement, its passages are re-indexed, and "Last updated" shows the new time.
- **Given** the replacement fails validation, **when** I upload it, **then** the old version stays unchanged.

### US-4.7 Upload professor material
**Personas:** IN, TA | **Priority:** Must | **Traces to:** FR-4.2, FR-6.1

As a course instructor, I want to upload slides, handouts or my own notes as instructor material, so that the study tools treat them as authoritative.

**Scenarios**
- **Given** I am a course instructor, **when** I upload a file through the note flow, **then** it is saved with type "Instructor material", shows that badge everywhere, and can be tagged to lectures.
- **Given** a student uploads, **when** the upload form is shown, **then** there is no option to mark the file as instructor material.

**Rules**
- [ ] The same file-type and size rules as US-4.1 apply
- [ ] If an instructor uploads the same document as a website PDF, it counts once, as instructor material (assumption A4)

---

## E5: Upvotes, Flags and Endorsements (FR-5)

### US-5.1 Upvote a helpful note
**Personas:** SC, SS, TA, IN | **Priority:** Must | **Traces to:** FR-5.1, FR-6.2

As a student studier, I want to upvote notes that helped me, so that better notes rank higher for everyone.

**Scenarios**
- **Given** a note by another member, **when** I click Upvote, **then** the count increases by one, and the button shows I upvoted it.
- **Given** I already upvoted it, **when** I click again, **then** my upvote is removed.
- **Given** my own note, **when** I view it, **then** the Upvote button is disabled.

**Rules**
- [ ] At most one upvote per user per note; there are no downvotes

### US-5.2 Flag a note as matching the slides
**Personas:** SC, SS | **Priority:** Should | **Traces to:** FR-5.2, FR-6.1

As a student, I want to mark a note as "matches the posted lecture slides", so that classmates know it lines up with official material.

**Scenarios**
- **Given** another member's note, **when** I click "Matches slides", **then** my flag is recorded, and the note shows the badge with a count.
- **Given** I flagged it, **when** I click again, **then** my flag is removed.

**Rules**
- [ ] At most one flag per user per note; not allowed on your own note
- [ ] Ranks below endorsement and majority agreement in precedence (FR-6.1)

### US-5.3 Endorse a note
**Personas:** IN, TA | **Priority:** Must | **Traces to:** FR-5.3, FR-5.4, FR-6.1

As a course instructor, I want to endorse accurate notes, so that students and the study tools prioritize them.

**Scenarios**
- **Given** I am a course instructor, **when** I click Endorse on a note, **then** it shows an "Instructor-endorsed" badge everywhere and its weight increases.
- **Given** an endorsed note, **when** I click Remove endorsement, **then** the badge and the bonus are removed.
- **Given** I am a student, **when** I view a note, **then** no Endorse control is shown, and the endorse endpoint returns 403.

---

## E6: Ranking Transparency (FR-6)

### US-6.1 Understand how sources are ranked
**Personas:** SS, SC | **Priority:** Should | **Traces to:** FR-6.1, FR-6.4

As a student studier, I want to read a plain explanation of how Margins ranks sources, so that I can judge how much to trust a guide.

**Scenarios**
- **Given** I am in a course, **when** I open "How sources are ranked", **then** I see:
  - the precedence order: instructor material > instructor-endorsed > majority of notes > matches slides > upvotes
  - a plain-language description of how upvotes, flags, endorsement and agreement affect a note's weight
- **Given** I open a note, **when** I look at its badges, **then** each badge links to this explanation.

**Rules**
- [ ] The explanation matches the constants in the single weight module (FR-6.4)

### US-6.2 Notes that agree with the class gain trust over time
**Personas:** SC, SS | **Priority:** Should | **Traces to:** FR-6.2, FR-6.3, FR-6.5, FR-8.6

As a student studier, I want notes that consistently agree with the rest of the class to rank higher, so that one-off mistakes don't drive guides and chat answers.

**Scenarios**
- **Given** conflict analysis has recorded comparisons for a note, **when** its weight is computed, **then** the weight includes a smoothed agreement score based on how often it matched the majority version.
- **Given** a note with very few comparisons, **when** its weight is computed, **then** the agreement score stays near neutral, so one comparison can't swing it.
- **Given** two passages are equally relevant, **when** retrieval ranks them, **then** the passage from the higher-weight source ranks first.

**Rules**
- [ ] The weight is computed by deterministic code from stored counts, not by the LLM

---

## E7: Study Guides (FR-7)

### US-7.1 Generate a study guide for an exam or lecture range
**Personas:** SS, SC | **Priority:** Must | **Traces to:** FR-7.1, FR-7.2, FR-7.5, NFR-3, NFR-4

As a student studier, I want to generate a study guide for an upcoming exam or a range of lectures, so that I can study from one document built from the whole class's notes.

**Scenarios**
- **Given** the course has exams with known ranges, **when** I choose "Prelim 1" and click Generate, **then** the scope is set to that exam's lectures.
- **Given** I choose a custom lecture range, **when** I click Generate, **then** the guide covers exactly those lectures.
- **Given** generation is running, **when** I wait, **then** I see step-by-step progress (for example "Retrieving notes for Lecture 3...") and the guide normally appears within about 2 minutes at demo scale.
- **Given** the Gemini API errors, times out or hits a rate limit, **when** generation fails, **then** I see "Guide generation failed; try again" and no partial guide is saved.
- **Given** the agent reaches its step limit before finishing, **when** generation stops, **then** I see a failure message, not a half-finished guide presented as complete.

**Rules**
- [ ] The agent loop has a hard cap on steps and tool calls
- [ ] Malformed tool arguments are returned to the model as tool errors, not crashes

### US-7.2 Verify every statement in the guide
**Personas:** SS | **Priority:** Must | **Traces to:** FR-7.3, FR-7.7, FR-4.5, NFR-4, NFR-6

As a student studier, I want every claim in the guide to cite the note it came from, so that I can trust it and check it.

**Scenarios**
- **Given** a generated guide, **when** I read it, **then**:
  - overlapping content is merged into one section per topic (grouped by lecture when the course has a schedule)
  - every claim cites its source at note level: the note title, plus the uploader name or "Anonymous classmate"
  - "Course website: <page>" citations are used for website sources
- **Given** a citation, **when** I click it, **then** the full note opens.
- **Given** the model produces a section with no citations, **when** the guide is assembled, **then** that section is not shown as grounded content. Either generation fails, or the section is dropped with a visible note.
- **Given** a note contains text like "ignore previous instructions", **when** the guide is generated, **then** that text is treated as note content, and the agent's behavior does not change.
- **Given** the course has no endorsements, **when** the guide is shown, **then** it states that no instructor has endorsed any source.

**Rules**
- [ ] Guide Markdown is rendered with raw HTML off and autoescaping on; `|safe` is never used

### US-7.3 See what nobody covered
**Personas:** SS, SC | **Priority:** Must | **Traces to:** FR-7.3, FR-9.1

As a student studier, I want the guide to list lectures with missing or thin notes, so that I know where to look for other sources.

**Scenarios**
- **Given** lectures in scope have no tagged notes, **when** the guide is generated, **then** a "Coverage gaps" section lists them as "missing".
- **Given** lectures have fewer tagged notes than the thin threshold, **when** the guide is generated, **then** they are listed as "thin", with their note count.
- **Given** the course has no schedule, **when** the guide is generated, **then** the gaps section says coverage reporting is unavailable for this course.

**Rules**
- [ ] Untagged notes do not count toward coverage
- [ ] Gap computation is deterministic code (find_coverage_gaps)

### US-7.4 Generate a topic guide for a course without a schedule
**Personas:** SS | **Priority:** Should | **Traces to:** FR-3.5, FR-7.1

As a student studier in a course whose website had no schedule, I want to generate a whole-course guide organized by topic, so that I still get a merged, cited guide.

**Scenarios**
- **Given** the course has no schedule, **when** I open Generate guide, **then** the only scope offered is "Whole course (by topic)".
- **Given** I generate it, **when** it finishes, **then** sections are organized by topic, with citations and disagreements as in US-7.2 and US-8.1.

### US-7.5 Revisit saved guides and compare versions
**Personas:** SS, SC, IN, TA | **Priority:** Should | **Traces to:** FR-7.6

As a student studier, I want to open guides my classmates already generated and see how a regenerated guide changed, so that I don't have to wait for a new one and can see what changed after new notes arrived.

**Scenarios**
- **Given** guides exist, **when** I open the Guides page, **then** I see them grouped by exam (or "Custom range" or "Whole course"), each showing its latest version date.
- **Given** a guide has several versions, **when** I regenerate it, **then** a new version is added, and earlier versions stay viewable.
- **Given** two versions, **when** I choose Compare, **then** I see added and removed sections highlighted. *(Comparing is Could within this Should story.)*

---

## E8: Conflicts (FR-8)

### US-8.1 See where sources disagree
**Personas:** SS, IN, TA | **Priority:** Must | **Traces to:** FR-7.4, FR-8.1, FR-8.2, FR-8.3, A3

As a student studier, I want the guide to show conflicting versions side by side with their sources, so that I know what is disputed instead of getting a blended answer.

**Scenarios**
- **Given** two or more sources state a definition, formula, number or claim differently in substance, **when** the guide is generated, **then** a "Disagreement" block shows each version, its sources, and how many sources support it.
- **Given** one version has more supporting sources than any other, **when** it is shown, **then** it is labeled "More commonly reported".
- **Given** the versions are tied, **when** they are shown, **then** no majority label is used and the block says "Sources are split".
- **Given** instructor material is one of the versions, **when** it is shown, **then** it is labeled "Authoritative (instructor material)", and the student version is still shown.
- **Given** sources only differ in wording, **when** the guide is generated, **then** no disagreement is flagged.

**Rules**
- [ ] Each flagged conflict includes the differing text from each source
- [ ] Every conflict is stored for the coverage view (US-9.1) and feeds agreement scores (US-6.2)

### US-8.2 Resolve a conflict
**Personas:** IN, TA | **Priority:** Should | **Traces to:** FR-8.4, FR-8.5

As a course instructor, I want to pick the correct version of a flagged conflict, so that future guides show the right answer.

**Scenarios**
- **Given** an open conflict in the coverage view, **when** I choose the correct version and save, **then** the conflict is marked resolved, recording who resolved it and when.
- **Given** I also tick "Endorse this note", **when** I save, **then** the note behind the chosen version is endorsed (US-5.3).
- **Given** a resolved conflict, **when** a later guide covers that topic, **then** the chosen version is shown as "Resolved by course staff", and other versions are listed beneath it.

**Rules**
- [ ] No notifications or email are sent

---

## E9: Coverage View and Course Analysis (FR-9)

### US-9.1 See coverage per lecture
**Personas:** IN, TA | **Priority:** Must | **Traces to:** FR-9.1, FR-9.3

As a course instructor, I want a per-lecture view of notes, endorsements and conflicts, so that I can see where students need help.

**Scenarios**
- **Given** the course has a schedule, **when** I open Coverage, **then** each lecture shows:
  - tagged note count
  - endorsed count
  - whether instructor material exists
  - open and resolved conflict counts
  - a missing or thin status
- **Given** I click a lecture's conflict count, **when** it opens, **then** I see each conflict's versions and sources, with a Resolve action (US-8.2).
- **Given** the course has no schedule, **when** I open Coverage, **then** I see the no-schedule limitation, not per-lecture rows.
- **Given** I am a student, **when** I request the Coverage URL, **then** I get 403.

### US-9.2 Analyze the course for conflicts
**Personas:** IN, TA | **Priority:** Should | **Traces to:** FR-9.2, FR-8.6

As a course instructor, I want to run conflict detection across lectures without generating a guide, so that I can review disagreements before students generate guides.

**Scenarios**
- **Given** I click "Analyze course", **when** it runs, **then** I see per-lecture progress, and when it finishes the Coverage view shows refreshed conflicts.
- **Given** the Gemini API fails midway, **when** analysis stops, **then** lectures already analyzed keep their new results, and I see which lectures were not analyzed and a retry option.

### US-9.3 Find notes worth endorsing
**Personas:** IN, TA | **Priority:** Could | **Traces to:** FR-9.4, FR-5.4

As a course instructor, I want a short list of the highest-weighted notes I haven't endorsed, so that I can endorse good material quickly.

**Scenarios**
- **Given** unendorsed notes exist, **when** I open Coverage, **then** I see up to 10 of them ordered by weight, each with Open and Endorse actions.

---

## E10: Cited Chat (FR-10)

### US-10.1 Ask a question and get a cited answer [Key journey]
**Personas:** SS, SC | **Priority:** Must | **Traces to:** FR-10.1, FR-10.2, FR-10.4, FR-6.1, NFR-3, NFR-4

As a student studier, I want to ask questions about the course and get answers that cite our notes, so that I can study specific points and trust the answers.

**Scenarios**
- **Given** relevant notes exist, **when** I ask a question, **then** within about 20 seconds I get an answer in which every claim cites its source at note level. Clicking a citation opens the note.
- **Given** sources disagree on the answer, **when** the agent answers, **then** it follows the precedence order, and mentions the disagreement with the other version's source.
- **Given** a note contains text that tries to instruct the model, **when** it is retrieved, **then** the agent treats it as content, and its behavior does not change.
- **Given** the Gemini API errors or times out, **when** I ask, **then** I see a retryable error, and my question stays in the input box.

**Rules**
- [ ] Chat Markdown is rendered with raw HTML off and autoescaping on
- [ ] Anonymous uploaders appear as "Anonymous classmate" in citations

### US-10.2 Be told plainly when the notes don't cover my question [Key journey]
**Personas:** SS | **Priority:** Must | **Traces to:** FR-10.2

As a student studier, I want the chat to say when our course materials don't support an answer, so that I'm not misled by general knowledge.

**Scenarios**
- **Given** no retrieved source supports an answer, **when** I ask, **then** the reply says the course notes and website don't cover this, and gives no uncited answer.
- **Given** sources only partly cover the question, **when** I ask, **then** the reply answers the supported part with citations, and says which part is not covered.

### US-10.3 Keep my chat history private
**Personas:** SS, SC | **Priority:** Should | **Traces to:** FR-10.3

As a student studier, I want my past conversations saved and private, so that I can come back to them without classmates seeing my questions.

**Scenarios**
- **Given** I chatted before in this course, **when** I open Chat, **then** I see my previous conversations and can open any of them.
- **Given** I click New conversation, **when** I ask a question, **then** it starts a separate thread.
- **Given** another user, including a course instructor, **when** they use the app, **then** they cannot see my conversations.

---

## Won't (Not in the MVP)
From requirements.md section 6:
- University SSO
- Automatic instructor verification
- Inferring a schedule from notes
- Editing the outline by hand
- OCR
- Transcription
- LMS sites
- Scheduled syncing
- Cross-course guides or search
- Quizzes and flashcards
- Co-editing
- Comments and follows
- Notifications and email
- Mobile apps
- Cloud deployment
- Plagiarism checks
- Fine-tuning
- Downvotes

---

## Traceability: Requirements to Stories

| Requirement | Stories |
|---|---|
| FR-1 Accounts | US-1.1, US-1.2, US-1.3 |
| FR-2 Courses and membership | US-2.1, US-2.2, US-2.3, US-2.4, US-2.5, US-2.6, US-1.2 |
| FR-3 Course structure | US-3.1, US-3.2, US-3.3, US-3.4, US-3.5, US-2.1 |
| FR-4 Notes and professor material | US-4.1 to US-4.7 |
| FR-5 Upvotes, flags, endorsements | US-5.1, US-5.2, US-5.3 |
| FR-6 Weight and precedence | US-6.1, US-6.2, US-4.7, US-5.1 to US-5.3, US-10.1 |
| FR-7 Study guides | US-7.1 to US-7.5, US-8.1 |
| FR-8 Conflicts | US-8.1, US-8.2, US-6.2, US-9.2 |
| FR-9 Coverage and analysis | US-9.1, US-9.2, US-9.3, US-7.3 |
| FR-10 Cited chat | US-10.1, US-10.2, US-10.3 |
| NFR-2 Security essentials | US-1.1, US-1.2, US-2.2, US-3.1, US-4.1, US-4.3, US-7.2, US-10.1 |
| NFR-3 Performance | US-7.1, US-10.1, US-9.2 |
| NFR-4 LLM reliability | US-3.1, US-4.1, US-4.2, US-7.1, US-7.2, US-9.2, US-10.1 |
| NFR-6 Usability | US-7.2 (one-click citations), and all stories (field-level errors) |
| NFR-7 Privacy | US-2.3, US-4.3, US-10.3 |
| NFR-1, NFR-5 | Covered as requirements and in Build and Test, not as user stories (Q8: A) |

Every FR from FR-1 to FR-10 is covered by at least one story.

---

## Persona-to-Story Mapping

| Persona | Stories |
|---|---|
| **SC** Student contributor | 1.1, 1.2, 2.3, 2.5, 3.4, 3.5, 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 5.1, 5.2, 6.1, 6.2, 7.1, 7.3, 7.5, 10.1, 10.3 |
| **SS** Student studier | 1.1, 1.2, 2.3, 2.5, 3.2, 3.4, 3.5, 4.1, 4.4, 5.1, 5.2, 6.1, 6.2, 7.1, 7.2, 7.3, 7.4, 7.5, 8.1, 10.1, 10.2, 10.3 |
| **IN** Instructor | 1.2, 1.3, 2.1, 2.2, 2.6, 3.1, 3.2, 3.3, 3.4, 3.5, 4.2, 4.5, 4.6, 4.7, 5.1, 5.3, 7.5, 8.1, 8.2, 9.1, 9.2, 9.3 |
| **TA** Teaching assistant | 1.1, 1.2, 2.2, 2.4, 3.1, 3.2, 3.3, 3.4, 3.5, 4.1, 4.2, 4.3, 4.5, 4.6, 4.7, 5.1, 5.3, 7.5, 8.1, 8.2, 9.1, 9.2, 9.3 |

## Summary by Priority

| Priority | Count | Stories |
|---|---|---|
| Must | 23 | 1.1, 1.2, 1.3, 2.1, 2.2, 2.3, 3.1, 3.4, 3.5, 4.1, 4.2, 4.3, 4.4, 4.7, 5.1, 5.3, 7.1, 7.2, 7.3, 8.1, 9.1, 10.1, 10.2 |
| Should | 13 | 2.4, 3.2, 3.3, 4.5, 4.6, 5.2, 6.1, 6.2, 7.4, 7.5, 8.2, 9.2, 10.3 |
| Could | 3 | 2.5, 2.6, 9.3 |
| **Total** | **39** | |

## INVEST Check
- **Independent:** Stories depend only on shared foundations (an account, membership, an indexed note). No story requires another story's UI to be finished first, except the prerequisite chain in the key journey.
- **Negotiable:** Thresholds (thin-coverage count, page cap, weight constants) are left to Functional Design.
- **Valuable:** Each story names a persona benefit.
- **Estimable and Small:** Each covers one user goal. The largest stories (US-3.1, US-7.1, US-7.2) are bounded by explicit scenarios.
- **Testable:** Every story has Given/When/Then scenarios with observable outcomes.

US-7.5 merges viewing and comparing versions. Comparing is marked Could inside it, so it can be cut without splitting the story.
