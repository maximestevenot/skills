---
name: review-acceptance
description: Reviews whether a PR or branch delivers what its Linear issue asks — each acceptance criterion met and proven by a test, no scope creep, recorded decisions respected — with every finding checked by an independent refuter, and returns a report without writing to the PR. Use when the user asks for an acceptance review, asks whether a PR does what the issue says, or when implement-issue or ship-project reviews a PR alongside /code-review.
argument-hint: "<PR number/URL or branch> [Linear issue id] [--range <sha>..<sha>]"
---

# Review Acceptance

Judge the PR against **its issue**, not the code's general quality. Bugs and `CLAUDE.md` compliance
belong to `/code-review` — don't redo them. Read-only: never comment on, approve, or push to the PR;
the report goes back to whoever asked (the user or an orchestrator).

Run from the main session: it launches subagents, and subagents can't launch their own.

## 1. Resolve the contract

- The issue: from the argument, the PR body's `Fixes <ID>`, or the branch name. Fetch it (`get_issue`) and its discussion (`list_comments`).
- Extract the acceptance criteria and the decisions recorded in the issue or its comments.
- No linked issue, or an issue without acceptance criteria → stop and say so; point to `/code-review` for the bug review.

## 2. Check (one fresh subagent)

Launch a subagent that didn't write the code. Give it the target, the diff command (`gh pr diff <pr>`, `git diff origin/<default>...<branch>`, or `git diff <range>` with `--range`), the criteria and decisions verbatim, and these checks:

- **Criteria** — per criterion: `met` / `partial` / `missing`, with the `path:line` where it's implemented. A binary verdict and one line of evidence — no essay.
- **Proof** — per met criterion: the test that would fail if the change were reverted, or `none`. Skipped, deleted, or weakened existing tests are findings.
- **Scope** — behavior the diff adds that no criterion or decision asks for.
- **Decisions** — a recorded decision the implementation contradicts.
- **`--range`** (re-review after fixes) — only the fix commits; for each previously applied finding the caller lists, `fixed` / `not fixed`.

It reads touched files in full where the diff doesn't show the behavior, and returns the list of findings: everything not `met`-and-proven.

## 3. Refute (one fresh subagent per finding, in parallel)

Each refuter gets one finding plus the criterion text, and tries to prove it **wrong**: the criterion is met elsewhere in the diff, a test does cover it, the "extra" behavior is required by a criterion or decision. It answers `refuted` or `confirmed` with the evidence.

Drop refuted findings. A finding that survives is reported; one the refuter can't settle (ambiguous criterion) becomes a question.

## 4. Rank

- `blocker` — criterion `missing` or `partial`, decision contradicted, existing test weakened or removed. Must be fixed before merge.
- `should-fix` — criterion met but not proven by a test.
- `needs a decision` — scope creep, or a criterion too ambiguous to judge. The caller escalates it; it never blocks silently.

## 5. Report

```markdown
## Acceptance review — <PR/branch> (<ISSUE-ID>)
**Verdict:** accepted / needs fixes / needs a decision

### Criteria
- [x] <criterion> — `path:line`, proven by `test/file.test.ts` "<test name>"
- [ ] <criterion> — partial: what's missing

### Findings
1. [blocker|should-fix|needs a decision] `path:line` — problem. **Fix:** suggestion.
```

An empty findings list with every criterion met and proven is a valid review: verdict "accepted".
