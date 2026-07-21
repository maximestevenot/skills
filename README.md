# skills

Personal Claude Code skills marketplace — one unified product & delivery workflow, installable in every repo.

## The workflow

```
                    need / idea
                        │
                  ┌─ brainstorm ─┐          (grill one question at a time,
                  │              │           feasibility, impacts)
          project scale      issue scale
                  │              │
              write-prd    create-ticket    (PRD on the Linear project /
                  │              │           feature sheet in a Linear issue)
            prd-to-issues ───────┤
                                 │
                        implement-ticket    (worktree → plan → code → checks
                                 │           → push → PR → Linear updated)
                                 ▼
                                PR
```

**Maintenance** (periodic): `update-deps` · `changelog` · `tech-debt` · `debug`
**Meta**: `handoff` · `write-skill`

## Install

```
/plugin marketplace add maximestevenot/skills
/plugin install workflow@maxime-skills
```

For live iteration on this local clone, add the marketplace by path instead:

```
/plugin marketplace add /Users/maxime.stevenot/Sources/skills
```

Skills auto-activate from their descriptions, or explicitly via `/workflow:<skill>` (e.g. `/workflow:implement-ticket BW-42`).

## Iterate

1. Edit a skill under `plugins/workflow/skills/<name>/SKILL.md` (or add one with `/workflow:write-skill`).
2. Bump `version` in `plugins/workflow/.claude-plugin/plugin.json`.
3. `claude plugin validate .`
4. Commit, push, then `claude plugin update workflow` (restart the session to apply).

## Principles

- **One unified workflow, no imported dogma** — skills are lean, direct, and personal.
- **Generic across repos** — repo-specific behavior stays conditional ("if the project has a check skill, use it"). Repo-specific skills live in each repo's `.claude/skills/`.
- **Pipeline-aware** — each skill names its next step so sessions flow naturally.
