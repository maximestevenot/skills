# skills

Personal Claude Code skills marketplace — one unified product & delivery workflow, installable in every repo.

These are **my personal skills**, not a general-purpose framework. Several started from ideas borrowed from other people's skillsets, then were rewritten to fit how I actually work: Linear as the source of truth, worktree-per-issue, PRD → issues → PR.

They are **opinionated on purpose**. Where a choice existed, one was made rather than parameterised. Fork and adapt them — don't expect them to match your process out of the box.

## The workflow

```
                    need / idea
                        │
                  ┌─ brainstorm ─┐          (grill one question at a time,
                  │              │           feasibility, impacts)
          project scale      issue scale
                  │              │
              write-prd    create-issue    (PRD on the Linear project /
                  │              │           feature sheet in a Linear issue)
            prd-to-issues ───────┤
                                 │
                        implement-issue    (worktree → plan → code → checks
                                 │           → review-pr → push → PR → Linear updated)
                                 ▼
                                PR

   ship-project: orchestrates a whole project/milestone — implement-issue
   in parallel agents → review agent → fixes → (your go) → ordered merges
```

**Maintenance** (periodic): `update-deps` · `changelog` · `tech-debt` · `debug`
**Meta**: `handoff` · `write-skill`

## Skills

### Delivery pipeline

| Skill | What it does |
| --- | --- |
| `brainstorm` | Interviews you one question at a time to stress-test an idea against the codebase, then routes to a PRD or an issue. |
| `write-prd` | Writes a PRD for a project-scale feature and publishes it to the Linear project. |
| `create-issue` | Writes a single-deliverable Linear issue: codebase analysis, design section, acceptance criteria. |
| `prd-to-issues` | Slices a PRD into independently-implementable issues (vertical tracer bullets). |
| `implement-issue` | Takes an issue to a PR: worktree, branch, plan, code, checks, push, Linear updated. |
| `review-pr` | Independent review of a PR's diff: built-in `/code-review` for bugs, plus acceptance criteria, tests, and the debt it introduces (architecture, abstraction, duplication, consistency). Never writes to the PR. |
| `ship-project` | Manages a whole project or milestone: parallel implement-issue agents, independent review, fix triage, questions escalated to you, then ordered merges after your go. |

### Maintenance

| Skill | What it does |
| --- | --- |
| `debug` | Disciplined loop for bugs and perf regressions — reproduce, hypothesise, instrument, fix, regression-test. |
| `tech-debt` | Audits a TS/JS codebase for duplication, architecture debt, testing gaps, dead code and doc rot. |
| `update-deps` | Bumps all npm deps to latest (majors included), fixes mechanical breakage, lands one `chore(deps)` commit. |
| `changelog` | Writes a product-voice changelog entry from merged commits and prepends it to `CHANGELOG.md`. |

### Meta

| Skill | What it does |
| --- | --- |
| `handoff` | Compacts the session into a handoff doc for a fresh session or another agent. |
| `write-skill` | Creates or edits a skill, and decides whether it belongs here or in a repo's `.claude/skills/`. |

## Install

```
/plugin marketplace add maximestevenot/skills
/plugin install workflow@maxime-skills
```

For live iteration on this local clone, add the marketplace by path instead:

```
/plugin marketplace add /Users/maxime.stevenot/Sources/skills
```

Skills auto-activate from their descriptions, or explicitly via `/workflow:<skill>` (e.g. `/workflow:implement-issue BW-42`).

## Iterate

1. Edit a skill under `plugins/workflow/skills/<name>/SKILL.md` (or add one with `/workflow:write-skill`).
2. Bump `version` in `plugins/workflow/.claude-plugin/plugin.json`.
3. `claude plugin validate .`
4. Commit, push, then `claude plugin marketplace update maxime-skills && claude plugin update workflow@maxime-skills` (restart the session to apply).

## Principles

- **One unified workflow, no imported dogma** — skills are lean, direct, and personal.
- **Generic across repos** — repo-specific behavior stays conditional ("if the project has a check skill, use it"). Repo-specific skills live in each repo's `.claude/skills/`.
- **Pipeline-aware** — each skill names its next step so sessions flow naturally.
