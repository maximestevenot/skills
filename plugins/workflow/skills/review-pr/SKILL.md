---
name: review-pr
description: Review a PR or branch diff — runs the built-in code-review for bugs, then adds the contract (acceptance criteria, scope, tests) and the tech debt the change introduces (architecture, abstractions, duplication, consistency) — and returns triaged findings without writing to the PR. Use when the user asks to review a PR, branch, or diff, or when implement-issue or ship-project needs an independent review.
argument-hint: "<PR number/URL or branch> [Linear issue id] [--range <sha>..<sha>]"
---

# Review PR

Review **the diff**, not the codebase. Judge what this change adds or worsens; pre-existing debt is
context, not a finding. Read-only: never comment on, approve, or push to the PR — findings go back
to whoever asked (the user or an orchestrator).

Best run in a fresh context (a subagent), not by the agent that wrote the code.

## 1. Gather context

- The diff: `gh pr diff <pr>` or `git diff origin/<default>...<branch>`. Read touched files in full where the diff alone doesn't show the impact.
- `--range` given (re-review after fixes): review only `git diff <range>`, and check that each previously applied finding the caller lists is actually fixed.
- The contract: the linked Linear issue (from the argument, the PR body's `Fixes <ID>`, or the branch name) — its acceptance criteria and recorded decisions.
- The conventions: `CLAUDE.md` / `AGENTS.md` (root and touched directories), and how neighboring code solves the same problems.

## 2. Bug pass — delegate to `/code-review`

Run the built-in `code-review` skill at level `high` on the PR or branch (`/code-review high <target>`) — always pass the level, never inherit it. It owns the correctness review; don't redo it.

- Never pass `--comment` or `--fix`, and never use a review command that posts to the PR (e.g. the `code-review:code-review` plugin).
- If it's unavailable or can't run in this context, do the bug pass yourself: logic errors, edge cases, broken callers, security, history (does the change undo a past fix?), invariants stated in code comments.

## 3. Contract

- **Acceptance criteria** — each one met? Anything missing, anything beyond scope (creep)?
- **Tests** — do they exercise the new behavior, including failure paths? Would they fail if the change were reverted? Skipped or weakened existing tests are a finding.

## 4. Debt introduced by the diff

- **Architecture** — layering violations (UI reaching into data access, domain depending on framework), new circular imports, a file that becomes a god file or mixes concerns (routes + validation + queries + formatting).
- **Abstraction** — missing (same logic spread across call sites, concrete module imported where an interface is the established pattern) *and* premature (indirection, options, or generics with one caller and no second in sight).
- **Duplication** — reimplements a utility, hook, component, or validator that already exists; copy-pasted blocks within the diff.
- **Consistency** — naming, error handling, response shapes, folder placement that diverge from the established pattern without a reason.
- **Configuration** — hardcoded URLs, ports, keys, thresholds; feature flags with no removal path.
- **Complexity** — deep nesting, long functions, many parameters, where a simpler shape is obvious.
- **Docs** — docs or comments the change makes wrong; new TODOs without an issue reference.

## 5. Merge, verify, rank

Fold the `/code-review` findings in with yours, deduplicated. Confirm each of your own against the code — no speculative findings; fewer, solid findings beat a noisy list. Drop pre-existing issues (→ out-of-scope list), what the quality gate already catches, clearly intentional behavior changes, and convention findings you can't tie to a `CLAUDE.md` rule or an established pattern.

- `blocker` — bug, unmet acceptance criterion, security issue, or debt that will clearly bite soon. Must be fixed before merge.
- `should-fix` — real debt or inconsistency; cheap to fix now, costly later.
- `nit` — style or taste; never blocks.

## 6. Report

```markdown
## Review — <PR/branch> (<ISSUE-ID>)
**Verdict:** ready / needs fixes / needs a decision

### Findings
1. [blocker|should-fix|nit] [category] `path/file.ts:42` — problem. **Fix:** suggestion. (needs product decision: yes/no)

### Acceptance criteria
- [x] … / [ ] … — why

### Pre-existing debt noticed (out of scope)
- … (candidates for `/workflow:create-issue`)
```

A clean category needs no mention; an empty findings list with verdict "ready" is a valid review.
