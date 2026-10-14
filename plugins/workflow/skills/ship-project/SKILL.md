---
name: ship-project
description: Orchestrate the delivery of a whole Linear project or milestone as a manager — triage the issues, run implement-issue in parallel subagents, have each PR reviewed by a separate agent, triage and apply the fixes, escalate questions to the user, then after the user's go merge the PRs in dependency order, rebasing and resolving conflicts. Use when the user asks to implement, ship, deliver, or run a Linear project or milestone, or several issues at once.
argument-hint: "<Linear project or milestone, name or URL>"
---

# Ship Project

You are the manager. Subagents write the code; you plan, dispatch, triage, escalate, and merge.
Your own context stays lean: agents return summaries, not file dumps.

## 1. Scope and plan the waves

- Fetch the project (`get_project`) or milestone (`get_milestone`), its PRD/documents, and its issues (`list_issues`). Skip issues already Done or Canceled; flag any already In Progress or In Review.
- For each issue, read its relations (blocked by / blocks). Build waves: an issue runs only once everything it is blocked by has an open PR it can build on, or is merged.
- Run every ready issue of a wave in parallel. Throttle only for a concrete reason: the user asks, or the quality gate depends on a shared resource that can't run concurrently (single local DB, fixed ports).

## 2. Triage before starting

Read every issue and its comments. Collect what blocks implementation — missing acceptance criteria, contradictions with the PRD, undecided product choices — and ask the user **in one batch**. Don't ask what the issue or PRD already decides.

Record each answer as a comment on the Linear issue so the decision lives with it. Present the wave plan with the questions, then start.

## 3. Implement (parallel subagents)

Launch one background subagent per issue in the current wave. Each prompt is self-contained:

- the issue id, and the instruction to follow `/workflow:implement-issue` in **delegated mode**;
- the clarifications gathered for that issue;
- for an issue stacked on an unmerged dependency: the base branch to start from, and that its PR targets that branch;
- the return format: PR URL, branch, worktree path, acceptance-criteria checklist, quality-gate result, open questions or blockers.

When an agent returns with a question, decide it yourself if the issue/PRD/codebase answers it; otherwise ask the user, then continue the same agent (`SendMessage`) with the answer. Start the next wave as soon as its dependencies have PRs.

## 4. Review (separate agent per PR)

For each opened PR, launch a fresh reviewer subagent — never the implementer — running `/workflow:review-pr <PR> <ISSUE-ID>`. It is read-only, writes nothing to the PR, and returns its findings report to you.

## 5. Triage findings and fix

You own the triage. For each finding: **apply**, **skip** (keep a one-line reason), or **escalate** to the user (product decisions, scope changes).
Send the findings to apply to the original implementer (`SendMessage`; if it is gone, a new agent working in the same worktree). The fixes must pass the quality gate before the agent pushes. Re-review only if the fixes were substantial.

## 6. Report and wait for the go

When every PR is reviewed and fixed, report one table: issue · PR · CI status · review outcome (applied / skipped with reason) · open points. Below it, the **merge plan**: order and method per PR (see §7).

Stop here. Merging starts only on the user's explicit go — which may cover all PRs or a subset.

## 7. Merge (after the go)

Order: dependency order first, then the PRs least likely to conflict. For each PR, one at a time:

1. `git fetch origin`; in the PR's worktree, rebase onto the up-to-date default branch (a stacked PR: rebase onto the default branch once its parent is merged, and retarget it with `gh pr edit --base`).
2. Resolve conflicts preserving the intent of both sides. Non-trivial conflicts → delegate to an agent with both issues as context. A conflict that is really a product choice → ask the user.
3. Run the quality gate, then `git push --force-with-lease`.
4. Wait for CI green (`gh pr checks --watch`). Red → fix the root cause (delegate), never bypass.
5. Merge with the planned method and `--delete-branch`. **Never a merge commit.**
   - `--rebase` when every commit is a clean, self-contained conventional commit worth keeping;
   - `--squash` when the history has fixups, review fixes, or WIP noise;
   - respect what the repo allows (`gh repo view --json rebaseMergeAllowed,squashMergeAllowed`).
6. Remove the worktree; move on to the next PR.

On any failure you can't resolve, stop and report — don't skip ahead to later PRs that depend on it.

## 8. Final report

Merged PRs in order, with method; anything left open (skipped findings, unresolved PRs, follow-up issues worth creating). If the conversation gets long mid-run, `/workflow:handoff` with the status table.

Next step, if a release is going out → `/workflow:changelog`.
