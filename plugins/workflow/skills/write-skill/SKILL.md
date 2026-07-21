---
name: write-skill
description: Create or edit an agent skill with proper structure and a trigger-rich description, and decide whether it belongs in the personal skillset or a project repo. Use when the user wants to create, write, improve, or relocate a skill.
argument-hint: "<what the skill should do>"
---

# Write Skill

## Where does it live?

Decide first — this determines the target directory:

- **Generic across repos** (workflow, methodology, tool usage) → the personal marketplace repo (`github.com/maximestevenot/skills`, local clone `~/Sources/skills`), under `plugins/workflow/skills/<name>/`. Bump the plugin version in `plugins/workflow/.claude-plugin/plugin.json`, commit, and run `claude plugin marketplace update maxime-skills && claude plugin update workflow@maxime-skills` to pick it up.
- **Tied to one repo's commands, paths, or conventions** → that repo's `.claude/skills/<name>/`.

## Process

1. **Gather requirements** — what task/domain, which concrete use cases, does it need scripts or just instructions, any reference material to bundle.
2. **Draft** — `SKILL.md` with concise instructions; split extra files (`REFERENCE.md`, formats, templates) only when SKILL.md would exceed ~120 lines; add utility scripts only for deterministic operations that would otherwise be regenerated each run.
3. **Review with the user** — does it cover the use cases, anything missing, right level of detail.

## House style

- Lean and direct. No "Iron Laws", no rationalization tables, no motivational framing — state the rule once and trust it.
- If the skill is part of a pipeline, name its neighbors ("next step → `/workflow:x`").
- Prefer conditional behavior over dogma: "if the project has X, use it; otherwise Y" travels across repos.

## The description is the trigger

The `description` frontmatter is the only thing the agent sees when deciding whether to load the skill. Max 1024 chars, third person. First sentence: what it does. Second: "Use when [specific triggers — keywords, contexts, file types]."

Good: `Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.`
Bad: `Helps with documents.`

## Review checklist

- [ ] Description includes concrete triggers ("Use when…")
- [ ] SKILL.md stays lean (~120 lines max; split otherwise)
- [ ] No time-sensitive info, no machine-specific paths in project skills
- [ ] Consistent terminology; concrete examples where they earn their place
- [ ] If in the marketplace repo: plugin version bumped, `claude plugin validate` passes
