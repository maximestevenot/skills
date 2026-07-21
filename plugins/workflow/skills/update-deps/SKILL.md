---
name: update-deps
description: Bump all npm dependencies (including majors) to latest, fix mechanical breakage, and land one atomic chore(deps) commit when green. Use for periodic dependency maintenance, "update deps", or when outdated majors pile up. Complements Dependabot-style minor/patch automation.
argument-hint: "[workspace] [--dry-run]"
---

# Update Dependencies

Bump everything to latest, auto-fix mechanical breakage, land one atomic commit. If the current repo ships its own dependency-update skill (e.g. `/update-dependencies` in backwise), **use that instead** — it knows the repo's pins and quirks.

## Step 0 — Detect the setup

- Package manager from the lockfile: `bun.lock` → bun, `pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `package-lock.json` → npm.
- Workspaces from the root `package.json` (or `pnpm-workspace.yaml`). An optional argument narrows scope to one workspace.
- Announce scope and dry-run flag before proceeding.

## Step 1 — Pre-flight (stop with a clear message on failure)

1. Working tree clean (`git status --porcelain` empty) — otherwise: commit or stash first.
2. Not on the default branch — otherwise: check out a branch (e.g. `chore/update-deps`) first, or offer to create it.
3. Lockfile present at the root — otherwise this isn't the repo root.

## Step 2 — Snapshot the bump plan

Run the manager's outdated command (`bun outdated` / `pnpm outdated` / `npm outdated`) per workspace in scope and render one combined table: Workspace | Package | Current | Target | Type, majors in **bold**. Nothing outdated → stop, all green. `--dry-run` → stop here.

## Step 3 — Review overrides

For each workspace with an `overrides`/`resolutions` block, walk each entry with `AskUserQuestion` (**Keep** / **Bump to latest** / **Drop**), showing the stored rationale from a sibling `DEPENDENCIES.md` if present. Persist every decision back to `DEPENDENCIES.md` (create it if needed):

```markdown
## <package> <range>
- **Set**: YYYY-MM-DD  - **Last re-evaluated**: YYYY-MM-DD
- **Reason**: <one line>  - **Status**: active | bumped-on-<date> | dropped-on-<date>
```

## Step 4 — Apply bumps

Run the manager's latest-bump command per workspace (`bun update --latest`, `pnpm update --latest`, `npx npm-check-updates -u && npm install`, …). Then restore what must not change:

- **Exact pins** (entries without `^`/`~`) — re-pin to the exact new version; don't introduce carets.
- **Non-semver specifiers** (URLs, `git:`, `file:`, `workspace:*`, `catalog:`) — restore verbatim if touched.

## Step 5 — Refresh the lockfile

One install at the repo root so the lockfile regenerates consistently across workspaces.

## Step 6 — Quality gate

Run the project's check skill if one exists (e.g. `/check-the-app full`); otherwise the repo's lint / typecheck / build / test scripts in that order.

## Step 7 — Fix loop (max 3 iterations)

On red: parse the errors and attempt **mechanical** fixes only — renamed/moved exports, signature shifts, removed APIs with a documented replacement, config-key changes per the new major's release notes, official codemods. Re-run the gate. Stop early if an iteration makes no progress. Still red after 3 → **stop, do not commit**: print the bump diff, remaining errors, fixes attempted, and recommend reverting the troublesome package(s) or fixing manually.

## Step 8 — Commit

Single atomic commit, no push, no PR:

```
chore(deps): bump all dependencies to latest

Notable bumps:
- <pkg>: <old> → <new> (major)

Override decisions:
- <pkg>: kept @ <range> — <reason>

Fixes applied:
- <one line per mechanical fix>
```

## Guardrails

- Never run on the default branch; never commit red; never silently drop overrides.
- Only touch dependency-related `package.json` keys — never `name`, `version`, `scripts`.
- Don't loosen exact pins unless explicitly asked, per package.
