# Info-5940-MidSem-Milestone

# Margins

Margins is a web platform where students in the same course pool their lecture notes, and an AI agent turns that shared pool into a study guide grounded in the course itself.

## Team

Nancy Khosla, nk549
| Amanda Johnson | alj73 |
| [Full name] | [netid] |
| [Full name] | [netid] |

**Coding agent:** [Claude Code] <!-- TODO: list every agent your team uses -->
**AI-DLC version:** v1.0.1

## Problem and Why an LLM Fits It

Students in the same course take notes on the same lectures, but each set of notes is incomplete in its own way: one student misses a definition, another skips a lecture, a third writes down an example incorrectly. When exams approach, students try to fill these gaps by trading notes in group chats, which leaves them with a pile of overlapping, inconsistently formatted, and sometimes contradictory documents and no clear picture of what the course actually expects them to know. Existing tools do not solve this. Shared drives store notes but do not reconcile them, and general-purpose chatbots summarize whatever one student uploads without knowing what the rest of the class wrote or what the course covers. Margins is for undergraduate students in lecture-based courses who want one trustworthy study guide built from the class's collective notes, and for professors and TAs who want a lightweight way to point students toward accurate material.

An LLM is the right tool because the core work is reading and reconciling unstructured text written by many different people. Notes vary in vocabulary, structure, and detail, so matching "the same idea" across them requires language understanding rather than keyword search. The system also has to judge which version of a conflicting explanation is more reliable, explain those judgments in plain language, and connect topics in the notes to topics listed on a course website. These are tasks rule-based software handles poorly and LLMs handle well, especially when their output is grounded in and cited back to the source notes.

## Planned LLM Integration

The central component is a study-guide agent. Rather than making a single summarization call, the agent decides which tools to use and in what order: it retrieves relevant passages from the class's uploaded notes, fetches and parses the public course website (syllabus, schedule, and linked lecture materials), compares the topics in the course schedule with the topics covered by the notes, and checks endorsement and rating data to decide which sources to trust. It then writes a study guide organized by course topic that merges overlapping notes, flags places where notes disagree, cites the specific notes behind each section, and lists lectures that no student has covered. A second feature, grounded chat, lets students ask questions about the class's material; the agent searches the notes and course site, answers with citations, and says explicitly when the materials do not cover a question.

The planned tools are: a course-site fetcher and parser, a document parser for uploaded notes (PDF and Markdown), a semantic search tool over the class's notes using embeddings, a coverage-comparison tool, and a database lookup for endorsements and ratings. Deterministic logic such as permission checks and endorsement scores stays in ordinary code; the LLM is used for reading, reconciling, and writing. We plan to run open models locally through Ollama: a tool-capable chat model for the agent and an embedding model for search, with the exact models to be chosen after testing in Construction. [TODO: confirm model and provider.] If local models prove too slow or unreliable at tool calling, we will switch to a hosted provider's API through the same OpenAI-compatible interface.

## Scope and Feasibility

By the final project deadline, Margins will let a user create an account, join a class page, upload notes to that class, and rate notes. Users with a professor or TA role can endorse notes. For any class, a student can generate a cited study guide from the pooled notes, see a coverage report comparing the notes against a public course website, and ask questions in grounded chat. Together these features form a complete loop, from contributing notes to studying from the class's combined knowledge, so the project is substantially more than a single LLM feature.

To keep the project achievable, we are deliberately leaving out features that add web-development work without strengthening the core product: comments and discussion threads, following users, notifications, real-time co-editing, handwriting recognition, mobile apps, and integration with Canvas or other systems that require login. We will demo with one or two courses whose websites are public. The product runs locally on our laptops, so we avoid cloud deployment work entirely. We also recognize that some courses restrict sharing materials; class pages are opt-in, and professor endorsement gives instructors a visible role in what is shared.

## Collaboration Plan

