---
name: write-prd
description: Write a PRD for a project-scale feature and publish it to the Linear project. Use when a feature spans multiple issues, after a brainstorm session, or when the user asks for a PRD, spec, or feature document at project level. For a single-issue feature sheet, use create-issue instead.
argument-hint: "<feature name or Linear project>"
---

# Write PRD

Turn accumulated context (usually a brainstorm session) into a PRD published on the Linear project.

## 1. Gather what's missing

Work from the conversation first — never re-ask what has already been decided. If starting cold, gather context conversationally, most important questions first (not a questionnaire dump):

- **Problem** — what problem, who experiences it, how often
- **Users** — which segment(s)
- **Success** — how we will know it worked
- **Constraints** — technical, timeline, dependencies
- **Prior art** — earlier attempts, existing partial solutions

Explore the codebase for the technical grounding: sketch the modules to build or modify. Prefer *deep modules* — lots of functionality behind a small, stable, testable interface. Confirm the module sketch with the user before writing.

## 2. Write the PRD

Sections, in order (skip one only when genuinely empty):

1. **Problem statement** — 2–3 sentences from the user's perspective, grounded in evidence (support data, metrics, feedback).
2. **Goals** — 3–5 measurable outcomes, not outputs ("cut time-to-first-claim in half", not "build the claim wizard").
3. **Non-goals** — 3–5 explicit exclusions, each with a one-line reason. This is scope-creep insurance; it matters as much as Goals.
4. **User stories** — "As a <actor>, I want <capability>, so that <benefit>". Numbered, grouped by persona, including edge, error, and empty states.
5. **Requirements** — Must-have (P0) / Nice-to-have (P1) / Future (P2), each with acceptance criteria. Be ruthless: if everything is P0, nothing is.
6. **Implementation decisions** — modules and their interfaces, schema changes, API contracts, architectural choices, resolved trade-offs. No file paths or code snippets (they rot) — except a snippet that encodes a decision better than prose can (state machine, schema, type shape).
7. **Success metrics** — targets with a number and a time window, plus how each is measured.
8. **Open questions** — genuinely unresolved items, each with an owner and marked blocking / non-blocking.

Keep it scannable: a busy reader should get the story from headers and bold text alone.

## 3. Publish to Linear

1. Find or create the Linear project (`list_projects`, `save_project`).
2. Review the draft with the user.
3. Publish as a project document (`save_document` attached to the project), or as the project description if the user prefers.

## Next step

Once published → `/workflow:prd-to-issues` to break it into implementable issues.
