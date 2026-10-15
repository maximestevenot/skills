---
name: implement-issue
description: Implement a Linear issue end-to-end — fetch the issue, isolate the work in a worktree and branch, plan, implement, verify, get an independent review, then push the branch and open the PR linked to the issue. Use when the user asks to implement, work on, or pick up a Linear issue or issue.
argument-hint: "<issue id or URL, e.g. BW-42>"
---

# Implement Issue

End-to-end: issue → code → green checks → independent review → pushed branch → open PR → Linear updated.

Running this skill is the user asking for the branch to be pushed and the PR opened — the deliverable
is a PR URL, not a local commit. Don't stop at "ready to push" to ask for confirmation.

**Delegated mode** — when run as a subagent by an orchestrator (e.g. `/workflow:ship-project`), you
can't talk to the user: no plan mode and no plan approval — plan in a few lines, then implement. Skip step 6 — the orchestrator runs the review.
If a real ambiguity blocks you, stop and return your questions instead of guessing. Your final message
is the step 8 report plus branch and worktree path.

## 1. Fetch and understand

- Fetch the issue (`get_issue`), its discussion (`list_comments`), and attached mockups (`extract_images`). No issue given → ask which one.
- Extract: goal, scope, design decisions already made, acceptance criteria. **The acceptance criteria are the contract.**
- Summarize your understanding in 3–5 bullets. Ask about real ambiguities only — don't re-litigate what the issue already decides.

## 2. Isolate the work

- `git fetch origin` so you branch from an up-to-date default branch.
- Create an isolated workspace: prefer the harness's native worktree tool when available; otherwise `git worktree add ../<repo>-<issue-id> -b <branch> origin/<default>`.
- Branch name: use Linear's suggested branch name from the issue when available; otherwise `<issue-id-lowercase>-<short-slug>`.
- Move the Linear issue to **In Progress** (`save_issue`).

## 3. Plan

- Explore the codebase. The project's `CLAUDE.md` / `AGENTS.md` tell you where things live and which conventions apply — follow them exactly.
- On a large codebase, if you can launch subagents, delegate the search to an `Explore` agent (`model: sonnet`): it returns the files to touch, the patterns to follow, and the utilities to reuse — not file dumps. Wait for its report before planning — a background agent's result arrives later as a notification.
- Produce a step-by-step plan (use plan mode if available) and get approval before writing code, unless the change is trivial.

## 4. Implement

- Reuse existing patterns and utilities; match the surrounding code's style.
- **No scope creep** — only what the issue asks for.
- Commit in coherent steps using the repo's commit style (check `git log --oneline -10`); include the issue id, e.g. `feat(web): add refund timeline (BW-42)`.
- Keep history linear: rebase on the default branch if it moved. Never create merge commits.

## 5. Verify

- Run the project's quality gate: its check skill if one exists (e.g. `/check-the-app`), otherwise the repo's lint / typecheck / build / test scripts. Fix root causes — never suppress — and re-run until green.
- If you can launch subagents, have a `model: haiku` agent run the gate and return only the failures, grouped and deduplicated; you do the fixing.
- Walk the acceptance criteria as a checklist, noting where each is satisfied. Unmet criteria must be explained, not hidden.

## 6. Review

- Launch a fresh subagent to run `/workflow:review-pr` on the branch with the issue id — a context that didn't write the code. Skip only for trivial changes or if the user says so.
- Triage each finding: **apply**, **skip** (one-line reason), or **ask the user** (product decisions, scope changes). Blockers are applied or escalated, never silently skipped.
- Apply the fixes as their own commits, then re-run the quality gate until green.

## 7. Push and open the PR

This step is the point of the skill — run it. If something blocks it (no remote, `gh` not
authenticated, protected branch), report the error; never end the run silently without a PR.

- `git push -u origin <branch>`
- `gh pr create` — title `<type>: <summary> (<ISSUE-ID>)`; body contains:
  - a short **what & why** summary,
  - the **acceptance criteria checklist** with status,
  - **test notes** (what the quality gate covered, any manual verification),
  - a **review summary** — findings applied, and skipped with the reason,
  - the Linear magic word — `Fixes <ISSUE-ID>` — so Linear links the PR and auto-closes on merge.
- Move the Linear issue to **In Review**. Confirm the PR is attached to the issue (the GitHub integration usually does this via the branch name); if not, add the PR link as an attachment or comment.

## 8. Report

Finish with: the PR URL, the acceptance-criteria checklist status, the quality-gate result, and anything intentionally left out of scope.
