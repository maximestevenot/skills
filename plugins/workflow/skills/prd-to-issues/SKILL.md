---
name: prd-to-issues
description: Break a PRD or plan into independently-implementable Linear issues using vertical tracer-bullet slices. Use when the user wants to convert a PRD or plan into issues, create the implementation issues for a project, or break down work.
argument-hint: "<Linear project, PRD document, or issue reference>"
---

# PRD → Issues

Break a PRD into thin **vertical slices** (tracer bullets), each an independently-grabbable Linear issue.

## 1. Gather context

Work from the conversation if the PRD was just written. Otherwise fetch it: the Linear project's documents (`list_documents` / `get_document`) or the reference the user passed. Read it fully.

## 2. Explore the codebase (if not already done)

Issue titles and descriptions should use the project's domain vocabulary and respect recorded decisions (ADRs) in the area.

## 3. Draft vertical slices

Each issue is a thin slice that cuts through **all** layers end-to-end (schema → API → UI → tests) — never a horizontal slice of one layer.

- Each slice delivers a narrow but **complete** path; done means demoable or verifiable on its own.
- Prefer many thin slices over few thick ones.
- The first slice is the tracer bullet: the smallest path that proves the architecture end-to-end.
- Flag slices that need a human decision (design review, architectural choice) so they aren't picked up blindly by an autonomous agent.

Slices are usually implemented by parallel agents via `/workflow:ship-project`. Split for that:

- **Wide over deep.** Independent slices run in parallel; a dependency chain runs serially. After the tracer bullet, aim for as many slices as possible blocked only by it.
- **Split finer than for a human team** when it buys parallelism or a smaller PR. Don't split when the pieces would just queue up one after another on the same files — that adds rebases, not speed.
- **Mind shared hotspots** (migrations, route/registry files, shared types, i18n): slices that all touch one are merge conflicts waiting. Note it in each slice, or sequence them explicitly.
- **Self-contained issues.** The implementing agent never sees this conversation — every decision it needs goes in the issue.

## 4. Quiz the user

Present the breakdown as a numbered list — for each slice: **title**, **blocked by**, **user stories covered**, and whether it **needs a human decision**. Then show the resulting waves (which slices can run in parallel). Ask: granularity right? dependencies correct? merge or split anything? Iterate until approved.

## 5. Publish to Linear

Create issues in dependency order (blockers first, so "Blocked by" can reference real identifiers). Attach each to the project, and use Linear issue relations for blocking links where possible. Issue body template:

```markdown
## Parent

<project or parent issue reference>

## What to build

End-to-end behavior of this slice — not layer-by-layer implementation. No file paths or code
snippets (they rot), except a snippet that encodes a decision better than prose (schema, state
machine, type shape).

## Acceptance criteria

- [ ] ...

## Blocked by

<issue reference> — or "None, can start immediately"
```

Do not close or modify the parent PRD or project.

## Next step

Each issue gets implemented via `/workflow:implement-issue`, or the whole project/milestone at once via `/workflow:ship-project`.
