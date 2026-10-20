---
name: tech-debt
description: >
  Scans a TypeScript/JavaScript codebase for tech debt and produces a prioritized report — architecture
  and design-principle violations (hexagonal layering, SOLID, DRY, KISS, YAGNI, consistency, least
  surprise), duplication, dependency, testing, API, configuration, code quality, and documentation debt.
  Use when the user asks to audit a codebase for tech debt, find code smells, review architecture
  health or design principles, check for dead code, assess code quality, or says "what should we
  clean up", "find things to refactor", "run a code audit". Supports scoping by debt category or by
  codebase area, or a full review covering everything.
argument-hint: "[category and/or area to scope the scan]"
---

# Tech Debt Scanner

Scan a TypeScript/JavaScript codebase and produce a prioritized tech debt report. Cheap models collect evidence; expensive judgment is spent only on candidates.

## Scoping

The user can narrow the scan by **debt category** ("just architecture debt"), by **codebase area** ("focus on src/payments"), or both. No scope → full review. Never silently expand the scope.

## Scan categories

1. **Architecture & design** — the project's declared architecture enforced (hexagonal or otherwise), circular dependencies, god files (~500+ lines or 10+ exports), mixed concerns, and violations of the design principles in [principles.md](principles.md).
2. **Code duplication** — copy-pasted blocks >10 lines, near-identical functions that should be one shared utility — but only duplication of *knowledge* (see DRY in principles.md).
3. **Dependency debt** — redundant packages serving the same purpose, packages listed but never imported. Not outdated versions — that's `/workflow:update-deps`.
4. **Testing debt** — untested critical paths (domain/services with no test), skipped tests, missing e2e for key workflows, tests coupled to internals.
5. **API & contract debt** — inconsistent naming/error handling/response shapes, `@deprecated` code still called, OpenAPI/JSDoc drifted from the code.
6. **Configuration debt** — hardcoded URLs/ports/keys/thresholds, dead feature flags.
7. **Code quality** — cyclomatic complexity ~10+, nesting >4, 5+ parameters, misleading names, stale `TODO|FIXME|HACK|XXX` (older than ~3 months via `git blame`).
8. **Documentation rot** — docs referencing files/commands/patterns that no longer exist; comments contradicted by the code.

## Running the scan

### 1. Map (you, inline)

Read the top-level layout, `package.json`, `tsconfig`, lint config, and any architecture source of truth (`CLAUDE.md`, ADRs, `docs/architecture*`, dependency-cruiser/eslint boundary rules). Write down in a few lines: the layers and the allowed dependency direction, the conventions, the in-scope paths. Every agent receives this map — no agent re-discovers the structure.

Measure size: `git ls-files '*.ts' '*.tsx' '*.js' '*.jsx' | xargs wc -l | tail -1` (restricted to scope).

### 2. Pick the route

- **Small scope** (one or two categories, or under ~10k lines) → skip the fan-out: run step 3's commands yourself and judge inline. Subagents would cost more than they save.
- **Otherwise** → steps 3–4 with subagents.

### 3. Inventory (one `model: haiku` agent, `effort: low`)

Mechanical only — no judgment, no file reading beyond what a command prints. It runs, within scope:

- `wc -l` sorted (god files), export counts per file;
- cycles: `npx madge --circular --extensions ts,tsx <src>` if `madge` or `dependency-cruiser` is in the project; otherwise skip and say so;
- duplication: `jscpd` if installed; otherwise skip;
- unused files/exports/deps: `knip` if installed; otherwise grep each `package.json` dependency for an import;
- cross-layer imports: grep imports from inner-layer paths toward outer-layer paths using the map;
- `it.skip|xit|test.skip|describe.skip`, `TODO|FIXME|HACK|XXX` with `git blame` dates, `@deprecated`, URL/port/secret-looking literals, source files with no sibling test.

Returns a **candidate list** grouped by category: `path:line — signal` one per line, capped at ~30 per category, highest signal first. No code excerpts.

### 4. Judge (parallel, background)

One agent per category group, each receiving the map, its candidates, and the instruction to read only the lines needed to confirm or reject each candidate:

| Group | Categories | Model |
|---|---|---|
| Design | 1 Architecture & design, 2 Duplication | `opus`, `effort: medium` — reads [principles.md](principles.md) |
| Hygiene | 4 Testing, 7 Code quality | `sonnet` |
| Surface | 3 Dependencies, 5 API, 6 Configuration, 8 Docs | `sonnet` |

Drop groups outside the scope. Each agent returns **at most 8 confirmed findings** in the report's finding format, plus a one-line count of rejected candidates. Findings without a `path:line` and a concrete consequence are discarded.

### 5. Stitch and verify (you)

Merge sections, deduplicate (one root cause = one finding, listing its locations), and for every **critical/high** finding open the cited lines yourself to confirm. Downgrade or drop what doesn't hold.

## Report format

```markdown
# Tech Debt Report — [Project]
> Scanned [date] | Scope: [full / categories / area] | Architecture reference: [source or "inferred"]

## Summary
[2–3 sentences + table: Category | Findings | Critical | High | Medium | Low]

## [Category]
### [Finding] — [critical/high/medium/low]
**Location:** `path/file.ts:L42`
**What:** …  **Principle:** [e.g. Dependency rule, SRP, DRY — when applicable]
**Why it matters:** …  **Suggested fix:** …  **Effort:** small/medium/large  **When:** now / boy-scout

## Recommended Action Plan
[Ordered by priority × effort, quick wins grouped; "boy-scout" items listed separately as fix-when-touched]
```

Priorities: **critical** = causing bugs/security risk/blocking work, or breaking the declared architecture; **high** = significant burden, worsens over time; **medium** = real friction, sprint-cleanup material; **low** = nice-to-have.

Fewer, well-evidenced findings beat an exhaustive noisy list. A clean category gets one line saying so.

Next step: turn the action plan into issues with `/workflow:create-issue`, or a multi-issue cleanup project with `/workflow:write-prd`.
