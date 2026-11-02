---
name: prd-to-issues
description: Break a PRD or plan into independently-implementable Linear issues — a coarse split into a few vertical slices. Use when the user wants to convert a PRD or plan into issues, create the implementation issues for a project, or break down work.
argument-hint: "<Linear project, PRD document, or issue reference>"
---

# PRD → Issues

Break a PRD into a few **vertical slices**, each an independently-grabbable Linear issue. Start coarse; split only when it pays.

## 1. Gather context

Work from the conversation if the PRD was just written. Otherwise fetch it: the Linear project's documents (`list_documents` / `get_document`) or the reference the user passed. Read it fully.

## 2. Explore the codebase (if not already done)

Issue titles and descriptions should use the project's domain vocabulary and respect recorded decisions (ADRs) in the area.

## 3. Draft the split

Default to **one or two issues per milestone**. Propose that coarse split first; go finer only when the user asks or a piece clearly earns its own issue.

- Each issue is a vertical slice through every layer it needs (schema → API → UI → tests) — never one horizontal layer. Done means demoable or verifiable on its own.
- Split further only when it buys real parallelism or a reviewable PR size. Pieces that would queue up one after another on the same files stay together — splitting adds rebases, not speed.
- **Mind shared hotspots** (migrations, route/registry files, shared types, i18n): issues that all touch one are merge conflicts waiting. Note it, or sequence them explicitly.
- Flag issues that need a human decision (design review, architectural choice) so an autonomous agent doesn't pick them up blindly.
- **Self-contained issues.** The implementing agent (often a parallel one via `/workflow:ship-project`) never sees this conversation — every decision it needs goes in the issue.

## 4. Quiz the user

Present the split as a numbered list — for each issue: **title**, **blocked by**, and whether it **needs a human decision** — plus which can run in parallel. Ask: merge or split anything? dependencies right? Iterate until approved.

## 5. Publish to Linear

Create issues in dependency order (blockers first, so "Blocked by" can reference real identifiers). Attach each to the project, and use Linear issue relations for blocking links where possible. Write them in English, even when the conversation is in another language. Keep bodies short — issue body template:

```markdown
## Parent

<project or parent issue reference>

## What to build

A few bullets: the end-to-end behavior and the decisions the implementer needs — not
layer-by-layer steps. No file paths or code snippets (they rot), except a snippet that encodes a
decision better than prose (schema, state machine, type shape).

## Acceptance criteria

2–3 verifiable criteria.

- [ ] ...

## Blocked by

<issue reference> — or "None, can start immediately"
```

Do not close or modify the parent PRD or project.

## Next step

Each issue gets implemented via `/workflow:implement-issue`, or the whole project/milestone at once via `/workflow:ship-project`.
