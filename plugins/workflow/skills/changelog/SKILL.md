---
name: changelog
description: Write a product-voice changelog entry from merged commits — release notes since the last tag or entry, or a periodic (weekly) recap — and prepend it to CHANGELOG.md. Use when the user wants a changelog, release notes, a weekly summary, or asks "what shipped".
argument-hint: "[--release [tag]] [--week YYYY-Www] [--since <date> --until <date>] [--dry-run]"
---

# Changelog

Turn merged commits into a **short, product/impact-oriented** entry. The audience is product, ops, and leadership — it reads like release notes, not a commit log.

## Two modes

- **Period mode** (default): window = current ISO week, Monday 00:00 → now, local time. `--week YYYY-Www` targets a past week; `--since/--until` override.
- **Release mode** (`--release`, or the user mentions a release): window = last git tag (or last `CHANGELOG.md` entry if no tags) → now. Also print the entry standalone so it can be pasted as the GitHub Release body — offer `gh release create` only if the user asks.

Headings are always the ISO week, in both modes: `## <YYYY-Www> — <Mon> to <Fri>` (e.g. `## 2026-W41 — Oct 5 to Oct 9`), never a version — the version belongs to the release, not the changelog. Compute bounds with `date`, never hardcode. If the repo's `CHANGELOG.md` already uses another heading convention, follow it.

## Voice — the part that matters

- **Lead with user impact, not engineering phases.** Good: "Shopify integration is live — merchants install in one click and every claim is auto-enriched." Bad: "Phase 1 introduces a hexagonal EcommerceAdapter port…"
- **Collapse aggressively.** Multiple commits for one effort → one bullet; follow-ups, hot-fixes and polish belong to the feature's bullet.
- **Rewrite subjects into plain prose.** Strip `type(scope):`, expand jargon, use the product name.
- **A handful of bullets per bucket**, skimmable in under 30 seconds — not paragraphs recapping each phase. No commit hashes (provenance lives in git), no issue-description dumps — a trailing issue link is enough.

## Procedure

1. `git fetch origin` and read from `origin/<default-branch>`, `--no-merges`, with bodies (issue references live there).
2. **Classify** each commit into buckets, reading subject *and* body:
   - **New features** — net-new user-visible capability
   - **Product improvements** — user-visible enhancement to something existing
   - **Technical improvements** — internal only; include only substantial work (architecture, migrations, perf); skip minor refactors
   - **Fixes** — bugs a user would notice
   Exclude `chore`, `docs`, `style`, `test`, `ci`, `build`, deps bumps, and internal tooling unless clearly user-visible.
3. **Enrich issues lightly.** Detect the tracker prefix from commit messages (`[A-Z]+-\d+`). One `get_issue` per unique issue to confirm title and URL; retry once on transient errors, then keep the bare ID. Never invent issues or titles.
4. **Write bullets**: `- **<Punchy headline>.** <One or two sentences of user impact.> ([XX-42](<url>))`. Empty bucket → `_None this week._` (period mode) or omit (release mode). Nothing user-facing at all → one line saying so, and stop.
5. **Prepend** the entry to `CHANGELOG.md` at the repo root (create with `# Changelog` if missing), newest first, directly under the header. Never rewrite older entries. `--dry-run` → print only.

## Guardrails

- Only commits from the default branch's remote.
- Do not commit `CHANGELOG.md` automatically — leave it for the user to review.
- Never amend or rewrite existing changelog history; prepend only.
- Asked to publish the recap to a Linear cycle: the Linear MCP attaches documents to a project or issue only (`save_document`), not a cycle. Say so and skip the publish rather than post a doc that only looks attached.
