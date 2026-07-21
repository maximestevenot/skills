---
name: tech-debt
description: >
  Comprehensive tech debt scanner for TypeScript/JavaScript codebases. Identifies code duplication,
  architecture debt, dependency issues, testing gaps, API inconsistencies, configuration problems,
  code quality smells, and documentation rot. Use when the user asks to audit a codebase for tech
  debt, find code smells, review architecture health, check for dead code, assess code quality,
  or says "what should we clean up", "find things to refactor", "run a code audit". Supports
  scoping by debt category or by codebase area, or a full review covering everything.
argument-hint: "[category and/or area to scope the scan]"
---

# Tech Debt Scanner

Scan a TypeScript/JavaScript codebase and produce a prioritized tech debt report.

## Scoping

The user can narrow the scan by **debt category** ("just architecture debt"), by **codebase area** ("focus on src/payments"), or both. If neither is specified, run a full review. Same report structure either way, restricted to the requested slice.

## Scan categories

1. **Code duplication** — duplicate functions (>80% structural similarity), copy-pasted blocks >10 lines, near-identical implementations that could be one shared utility. Hotspots: utilities, validation, data transformation.
2. **Architecture debt** — circular dependencies (including indirect cycles), god files (~500+ lines or 10+ exports), layering violations (e.g. a UI component importing an ORM client directly), missing abstractions (one concrete module imported everywhere that would benefit from an interface), monolithic files mixing concerns (routes + validation + queries + formatting).
3. **Dependency debt** — redundant packages serving the same purpose (two date libs, two HTTP clients), packages listed but never imported. Focus on consolidation and dead weight, not outdated versions (that's `update-deps`' job).
4. **Testing debt** — untested critical paths (services/domain with no test file), skipped tests (`it.skip`, `xit`, `test.skip`, commented-out blocks), missing e2e for key workflows, tests coupled to internals that break on refactor.
5. **API & contract debt** — inconsistent naming/error handling/response shapes across routes, `@deprecated` code still called, docs (OpenAPI, JSDoc, README) drifted from the implementation.
6. **Configuration debt** — hardcoded values that should be env vars (URLs, ports, keys, thresholds), dead feature flags (always on/off, or never toggled).
7. **Code quality** — high cyclomatic complexity (~10+ decision points), nesting >4 levels, 5+ parameters, stale `TODO|FIXME|HACK|XXX` (use `git blame`; flag anything older than ~3 months).
8. **Documentation rot** — project docs referencing files/commands/patterns that no longer exist; comments contradicted by the code below them.

## Running the scan

1. **Understand the structure first** — read the top-level layout, `package.json`, and configs. You can't spot a layering violation without knowing the layers.
2. **Respect the scope** — never silently expand it.
3. **Parallelize** — for a full review, split categories across parallel subagents and stitch their sections together.
4. **Use tools efficiently** — grep/glob targeted patterns rather than reading every file: `wc -l` sorted for god files, grep for skipped tests, TODO markers, URL/port literals.
5. **Verify before reporting** — read enough context to confirm each finding is real. False positives erode trust.

## Report format

```markdown
# Tech Debt Report — [Project]
> Scanned [date] | Scope: [full / categories / area]

## Summary
[2–3 sentences + table: Category | Findings | Critical | High | Medium | Low]

## [Category]
### [Finding] — [critical/high/medium/low]
**Location:** `path/file.ts:L42`
**What:** …  **Why it matters:** …  **Suggested fix:** …  **Effort:** small/medium/large

## Recommended Action Plan
[Ordered by priority × effort, quick wins grouped]
```

Priorities: **critical** = causing bugs/security risk/blocking work; **high** = significant burden, worsens over time; **medium** = real friction, sprint-cleanup material; **low** = nice-to-have.

Lean toward fewer, well-evidenced findings over an exhaustive noisy list — 10 solid findings beat 50 with noise. If a category comes up clean, say so briefly; that's information too.
