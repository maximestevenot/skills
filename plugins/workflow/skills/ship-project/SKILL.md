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

Record each answer as a comment on the Linear issue (in English) so the decision lives with it.

Size each issue: **trivial** (copy, config, a small well-bounded fix with an obvious approach) or **standard**. An issue with a large scope may ship as several stacked PRs — propose it, the user decides.

Set the **stakes** for the run: **production**, or **spike/prototype** (code that exists to produce evidence, not to last). It sets the review depth in §5.

Present the global plan — waves, dependencies, shared hotspots likely to conflict, each issue's size, the stakes — with the questions, and wait until the plan and decisions are settled with the user. No issue starts before that.

## 3. Implement (parallel subagents)

Launch one background subagent per issue in the current wave, with `model: opus` and `effort: medium` — or `model: sonnet` for a trivial issue. The issue and PRD already carry the hard thinking, and review catches the rest. Each prompt is self-contained:

- the issue id, and the instruction to follow `/workflow:implement-issue` in **delegated mode**;
- the clarifications gathered for that issue;
- for an issue stacked on an unmerged dependency: the base branch to start from, and that its PR targets that branch;
- the worktree rule: `git worktree add .claude/worktrees/<short-name> -b <branch> <base>`, absolute paths, never `EnterWorktree`/`ExitWorktree` — in a parallel run it moves the shared session's directory and locks the other agents out of their worktrees;
- the return format: PR URL, branch, worktree path, acceptance-criteria checklist, quality-gate result, open questions or blockers.

If an agent moved the session anyway, `ExitWorktree({action: "keep"})` from your session restores the repo root without removing anything.

When an agent returns with a question, decide it yourself if the issue/PRD/codebase answers it; otherwise ask the user, then continue the same agent (`SendMessage`) with the answer. Start the next wave as soon as its dependencies have PRs.

## 4. Review (separate agent per PR)

For each opened PR, run both reviews yourself — they launch their own fresh agents, never the implementer, and subagents can't launch agents. Both are read-only and write nothing to the PR. One full review per PR, never more.

- `/code-review high <PR>` — bugs and `CLAUDE.md` compliance. Never pass `--comment` or `--fix`.
- `/workflow:review-acceptance <PR> <ISSUE-ID>` — criteria met and proven by tests, scope, decisions.

## 5. Triage findings and fix

You own the triage. First gather what is already decided: the issue's comments, and the Review section of this PR and of its parent PRs. A finding already decided or deferred is skipped with a pointer to that decision — never re-escalated. Hold the rest to this bar — don't polish. On a **spike/prototype**, *apply* narrows to what breaks the spike's results or loses data (a lost conclusion, a wrong citation, a leaked boundary, a flaky test); everything else is follow-up or skip:

- **apply** — a bug with a concrete failure scenario, or a criterion missing/partial.
- **follow-up** — real but out of scope or not urgent: listed in the PR body's Review section. A deferral to another issue is also a comment on that issue (finding, source PR, reason), so its implementer sees it.
- **skip** (one-line reason) — nits, style, commit layout, cosmetic tests or docs.
- **escalate** to the user — product decisions, scope changes.

Send the findings to apply to the original implementer in one batch (`SendMessage`; if it is gone, a new agent working in the same worktree, same model and effort), with the decisions to add to the PR's Review section. The fixes must pass the quality gate before the agent pushes. Don't rewrite history during review — fix commits get squashed at merge.

On a spike/prototype, stop there — no verification. On production, verify at most once, and only if a fix changed non-trivial behavior: `/workflow:review-acceptance <PR> <ISSUE-ID> --range <sha-before-fixes>..HEAD` with the list of applied findings. Never re-run a full `/code-review` on the PR: a single pass catches only part of the bugs, so each fresh pass finds new ones and the loop never ends — recall comes from the first round's effort, not from more rounds. What the verification still raises goes to follow-up, or to the user if it is a real blocker. There is no third round.

## 6. Report and wait for the go

When every PR is reviewed and fixed, report one table: issue · PR · CI status · review rounds · review outcome (applied / follow-up / skipped with reason) · open points. Below it, the **merge plan**: order and method per PR (see §7).

Stop here. Merging starts only on the user's explicit go — which may cover all PRs or a subset.

## 7. Merge (after the go)

Order: dependency order first, then the PRs least likely to conflict. For each PR, one at a time, delegate steps 1–4 to a `model: sonnet` agent working in the PR's worktree. It returns either "pushed, CI green" or the blocker — it never resolves a non-mechanical conflict or a red CI on its own.

1. `git fetch origin`; rebase onto the up-to-date default branch.
   A stacked PR, once its parent is merged: a rebase- or squash-merge rewrote the parent's commits, so re-parent with `git rebase --onto origin/<default> <old-parent-head> <branch>` (the tree should be unchanged: `git diff <old-head> <branch> --stat` is empty). Retarget with `gh pr edit --base <default>` *before* pushing — retargeting alone doesn't trigger CI, the push does. Follow the actual base chain, not branch names or phase numbers.
2. Mechanical conflicts (imports, lockfiles, adjacent edits) → resolve preserving both sides. Anything else → return it. You then delegate it to an `opus` agent with both issues as context, or ask the user if it is really a product choice.
3. Run the quality gate, then `git push --force-with-lease`.
4. Wait for CI green (`gh pr checks --watch`). Red → return the failure; you delegate the root-cause fix, never bypass.

Then you:

5. Merge with the planned method and `--delete-branch`. **Never a merge commit.**
   - `--rebase` when every commit is a clean, self-contained conventional commit worth keeping;
   - `--squash` when the history has fixups, review fixes, or WIP noise;
   - respect what the repo allows (`gh repo view --json rebaseMergeAllowed,squashMergeAllowed`).
6. Remove the worktree; move on to the next PR.

On any failure you can't resolve, stop and report — don't skip ahead to later PRs that depend on it.

## 8. Final report

Merged PRs in order, with method; anything left open (skipped findings, unresolved PRs, follow-up findings worth an issue).
Review metrics, to calibrate the pipeline: per reviewer (`/code-review`, `review-acceptance`), findings applied / follow-up / skipped, and review rounds per PR. A reviewer whose findings are mostly skipped is noise; a bug found after merge in reviewed code is a recall miss.

If the conversation gets long mid-run, `/workflow:handoff` with the status table.

Next step, if a release is going out → `/workflow:changelog`.
