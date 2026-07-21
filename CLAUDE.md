# skills — authoring conventions

Claude Code plugin-marketplace repo. One plugin: `workflow` (`plugins/workflow/`), skills under `plugins/workflow/skills/<name>/SKILL.md`.

## Rules

- **Lean and direct.** No "Iron Laws", no rationalization tables, no motivational framing. State a rule once.
- **SKILL.md stays under ~120 lines**; split reference material into sibling files only when needed.
- **Descriptions are triggers**: third person, first sentence = what it does, second = "Use when …" with concrete keywords. Max 1024 chars.
- **Generic across repos**: conditional behavior ("if the project has X, use it; otherwise Y") instead of hardcoded repo specifics. Anything tied to one repo belongs in that repo's `.claude/skills/`, not here.
- **Pipeline-aware**: when a skill has a natural next step, name it (`/workflow:<skill>`).
- **English** for all skill content.

## Before committing

1. `claude plugin validate .`
2. Bump `version` in `plugins/workflow/.claude-plugin/plugin.json` for any skill change.
3. Conventional commits (`feat:`, `fix:`, `docs:`, `chore:`), linear history — no merge commits.
