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
- Create an isolated workspace where the project's `CLAUDE.md` says; by default `.claude/worktrees/<short-name>` inside the repo (gitignored). In an interactive session, prefer the harness's native worktree tool (`EnterWorktree`). In delegated mode, never call `EnterWorktree`/`ExitWorktree` — it moves the shared session's directory and locks parallel agents out of their worktrees; run `git worktree add .claude/worktrees/<short-name> -b <branch> origin/<default>` and work there with absolute paths.
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

One review round, not a loop. A single pass catches only part of the bugs — recall comes from the round's effort, not from more rounds. Run both reviews on the branch — each launches its own fresh agents, a context that didn't write the code. Skip only for trivial changes or if the user says so.

- `/code-review high <branch>` — bugs and `CLAUDE.md` compliance. Never pass `--comment` or `--fix`.
- `/workflow:review-acceptance <branch> <ISSUE-ID>` — criteria met and proven by tests, scope, decisions.

Before triaging, gather what is already decided: the issue's comments, and the Review section of any parent PR this branch stacks on. A finding already decided or deferred is skipped with a pointer to that decision — never re-raised. Triage the rest against this bar:

- **apply** — a bug with a concrete failure scenario, or a criterion missing/partial.
- **follow-up** — real but out of scope or not urgent: listed in the PR body.
- **skip** (one-line reason) — nits, style, commit layout, cosmetic tests or docs.
- **ask the user** — product decisions, scope changes. Blockers are applied or escalated, never silently skipped.

Match the fix round to the stakes. On a spike or prototype, apply only what breaks its results or loses data (a wrong conclusion, a leak, a flaky test) and skip the verification below; cleanups and refactors are skipped.

Apply the fixes as their own commits, then re-run the quality gate until green. Verify only if a fix changed non-trivial behavior: `/workflow:review-acceptance <branch> <ISSUE-ID> --range <sha-before-fixes>..HEAD` with the applied findings. Never re-run a full `/code-review`. What it still raises goes to follow-up or the user — there is no third round.

## 7. Push and open the PR

This step is the point of the skill — run it. If something blocks it (no remote, `gh` not
authenticated, protected branch), report the error; never end the run silently without a PR.

- `git push -u origin <branch>`
- `gh pr create` — title `<type>: <summary> (<ISSUE-ID>)`; body contains:
  - a short **what & why** summary,
  - the **acceptance criteria checklist** with status,
  - **test notes** (what the quality gate covered, any manual verification),
  - a **Review** section — findings applied, skipped with the reason, follow-ups, and deferrals to another issue. It is the decision record later reviews and stacked PRs triage against,
  - the Linear magic word — `Fixes <ISSUE-ID>` — so Linear links the PR and auto-closes on merge.
- Move the Linear issue to **In Review**. Confirm the PR is attached to the issue (the GitHub integration usually does this via the branch name); if not, add the PR link as an attachment or comment.

## 8. Report

Finish with: the PR URL, the acceptance-criteria checklist status, the quality-gate result, and anything intentionally left out of scope.
