---
name: write-prd
description: Write a PRD for a project-scale feature and publish it to the Linear project. Use when a feature spans multiple issues, after a brainstorm session, or when the user asks for a PRD, spec, or feature document at project level. For a single-issue feature sheet, use create-issue instead.
argument-hint: "<feature name or Linear project>"
---

# Write PRD

Turn accumulated context (usually a brainstorm session) into a PRD published on the Linear project.

## 1. Gather what's missing

Work from the conversation first — never re-ask what has already been decided. If starting cold, ask the few questions that matter, most important first (not a questionnaire dump): the problem and who has it, constraints, prior attempts.

Explore the codebase for the technical grounding: sketch the modules to build or modify. Prefer *deep modules* — lots of functionality behind a small, stable, testable interface. Confirm the module sketch with the user before writing.

## 2. Write the PRD

About one page. A busy reader gets the story from headers and bold text alone. Sections, in order (skip one when empty):

1. **Problem** — 2–3 sentences: what breaks or is missing, for whom, the evidence.
2. **Rules and decisions** — the behavior to build and the choices already made: modules and their interfaces, API contracts, resolved trade-offs. Short bullets, no file paths or code (they rot).
3. **Schema** — only if the data model changes; a snippet is fine when it encodes the decision better than prose (schema, state machine, type shape).
4. **Out of scope** — explicit exclusions, one line each with the reason.
5. **Open questions** — only genuinely unresolved ones, marked blocking / non-blocking.

No user-story lists, no P0/P1/P2 tables, no success-metrics section — unless the user asks for them.

Write it in English, even when the conversation is in another language.

## 3. Publish to Linear

1. Find or create the Linear project (`list_projects`, `save_project`).
2. Review the draft with the user.
3. Publish as a project document (`save_document` attached to the project), or as the project description if the user prefers.

## Next step

Once published → `/workflow:prd-to-issues` to break it into implementable issues.
