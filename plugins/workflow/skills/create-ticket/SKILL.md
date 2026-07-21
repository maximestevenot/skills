---
name: create-ticket
description: Create a well-structured Linear issue (feature sheet) with codebase analysis, a design section, and acceptance criteria. Use when the user wants a ticket for a single deliverable — a new screen, endpoint, or behavior. For a multi-issue feature use write-prd. The deliverable is the ticket itself; no implementation.
argument-hint: "<what the ticket is about>"
---

# Create Ticket

Produce one well-structured Linear issue. The ticket is the deliverable — write no code.

## Process

1. **Understand the request.** What is the business goal, what problem does it solve, who benefits? If unclear, ask: "When you say X, do you mean A or B?"
2. **Analyze the codebase.** Which files/modules are affected, which existing patterns and utilities apply, integration points, risks, which packages are touched. Use the Explore agent for breadth.
3. **Resolve gaps.** Edge cases, competing approaches, constraints (performance, compatibility), error scenarios. Use `AskUserQuestion` for real ambiguities — don't ask what the codebase can answer.
4. **Propose the ticket.** Present your understanding (goal, key changes, affected areas) and get a nod before creating.
5. **Create it in Linear.** `list_teams` / `list_projects` to place it (attach it to its project when it belongs to one), `list_issue_labels` for labels, then `save_issue`.

## Ticket structure

### Title
Under 80 characters, business-oriented, action verb first: "Add", "Replace", "Fix", "Remove".

### Overview
**What** we aim to achieve and **why** — readable by non-technical stakeholders. Include the problem or opportunity, the expected outcome, and the high-level scope (what's in, what's out).

### Design
**How** it will be implemented — detailed principles, no code. As relevant: data model changes, API changes, UI/UX changes, shared schema changes, business logic, migration strategy. Use tables for enum values, field mappings, and state transitions.

### Acceptance criteria
A checklist of verifiable conditions. Each criterion is **specific**, **testable**, and **independent** of unstated assumptions.

## Notes

- Quality over speed: a well-written ticket pays for itself at implementation time.
- Next step, when the time comes → `/workflow:implement-ticket`.
