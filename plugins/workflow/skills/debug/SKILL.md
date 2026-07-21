---
name: debug
description: Disciplined debugging loop for bugs and performance regressions — build a feedback loop, reproduce, hypothesise, instrument, fix, regression-test. Use when something is broken, throwing, failing, flaky, or slower than before, before proposing any fix.
argument-hint: "<symptom or bug description>"
---

# Debug

A discipline for bugs that resist the first obvious fix. For trivial bugs (clear stack trace pointing at a one-line fix), just fix them — but the moment a "trivial" fix doesn't work, enter the loop.

## Phase 0 — Read everything first

- Read the **full** error message and stack trace, not the first line.
- Check recent changes: `git log --oneline -15`, `git diff` — bugs usually live in what moved last.

## Phase 1 — Build a feedback loop

**This is the skill.** Get a fast, deterministic, agent-runnable pass/fail signal for the bug; every later phase just consumes that signal. Ways to construct one, in rough order of preference:

1. Failing test at whatever seam reaches the bug (unit, integration, e2e)
2. Curl/HTTP script against a running dev server
3. CLI invocation with a fixture input, diffing output against known-good
4. Headless browser script asserting on DOM/console/network
5. Replay a captured trace (real payload/event log) through the code path in isolation
6. Throwaway harness: minimal subset of the system exercising the bug path
7. Property/fuzz loop for "sometimes wrong output" bugs
8. Bisection harness so `git bisect run` can do the work
9. Differential run: same input through old vs new, diff the outputs
10. Last resort: a structured checklist driving the human, whose captured output feeds back to you

Then iterate on the loop itself: make it **faster** (skip unrelated init), **sharper** (assert the exact symptom, not "didn't crash"), **more deterministic** (pin time, seed RNG, freeze network). For non-deterministic bugs, the goal is a **higher reproduction rate** — loop the trigger 100×, add stress, narrow timing — until it's debuggable.

If you genuinely cannot build a loop: stop, list what you tried, and ask for a repro environment or a captured artifact (HAR, log dump, recording). Do **not** hypothesise without a loop.

## Phase 2 — Reproduce

Run the loop; confirm it shows the **user's exact symptom** (not a nearby different failure), repeatably. Wrong bug = wrong fix.

## Phase 3 — Hypothesise

Generate **3–5 ranked falsifiable hypotheses** before testing any — a single hypothesis anchors you on the first plausible idea. Each must state its prediction: "if X is the cause, then changing Y makes the bug disappear." Can't state the prediction → it's a vibe; discard or sharpen. Show the ranked list to the user — they often re-rank instantly — but don't block on it.

## Phase 4 — Instrument

One variable at a time; every probe maps to one prediction. Prefer debugger/REPL inspection over logs; otherwise targeted logs at the boundaries that distinguish hypotheses — never "log everything and grep". Tag every debug log with a unique prefix (e.g. `[DEBUG-a4f2]`) so cleanup is a single grep. For performance: measure a baseline first, then bisect — never guess.

## Phase 5 — Fix + regression test

Write the regression test **before** the fix, at a seam that exercises the real bug pattern as it occurs at the call site. If no correct seam exists, that is itself a finding — record it (issue) instead of writing a false-confidence test. Then: watch the test fail → apply the fix → watch it pass → re-run the Phase 1 loop against the original, un-minimised scenario.

**Three failed fixes → stop.** The problem is your model of the system, not the code. Re-question the assumptions and the architecture with the user instead of attempting fix #4.

## Phase 6 — Cleanup

- [ ] Original repro no longer reproduces (loop re-run) and the regression test passes
- [ ] All `[DEBUG-...]` instrumentation removed; throwaway harnesses deleted
- [ ] The winning hypothesis is stated in the commit/PR message so the next debugger learns
- [ ] If the root cause was architectural (no test seam, hidden coupling): open a ticket for it now, while you have the information
