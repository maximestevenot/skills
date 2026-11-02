---
name: create-issue
description: Create a well-structured Linear issue (feature sheet) with codebase analysis, a design section, and acceptance criteria. Use when the user wants an issue for a single deliverable — a new screen, endpoint, or behavior. For a multi-issue feature use write-prd. The deliverable is the issue itself; no implementation.
argument-hint: "<what the issue is about>"
---

# Create Issue

Produce one well-structured Linear issue. The issue is the deliverable — write no code.

## Process

1. **Understand the request.** What is the business goal, what problem does it solve, who benefits? If unclear, ask: "When you say X, do you mean A or B?"
2. **Analyze the codebase.** Which files/modules are affected, which existing patterns and utilities apply, integration points, risks, which packages are touched. Use the Explore agent for breadth.
3. **Resolve gaps.** Edge cases, competing approaches, constraints (performance, compatibility), error scenarios. Use `AskUserQuestion` for real ambiguities — don't ask what the codebase can answer.
4. **Propose the issue.** Present your understanding (goal, key changes, affected areas) and get a nod before creating.
5. **Create it in Linear.** `list_teams` / `list_projects` to place it (attach it to its project when it belongs to one), `list_issue_labels` for labels, then `save_issue`.

## Issue structure

Short: an implementer reads it in a minute. Written in English, even when the conversation is in another language.

### Title
Under 80 characters, business-oriented, action verb first: "Add", "Replace", "Fix", "Remove".

### Overview
2–3 sentences: **what** we aim to achieve and **why**, readable by non-technical stakeholders, with what's in and out of scope.

### Design
**How**, as a few bullets of decisions — no code, no step-by-step plan. Only what applies: data model, API, UI/UX, shared schema, business rules, migration. A table only for enum values, field mappings or state transitions.

### Acceptance criteria
2–3 verifiable conditions, each **specific** and **testable** — the contract the PR is reviewed against.

## Next step

When the time comes → `/workflow:implement-issue`.
