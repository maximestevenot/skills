---
name: brainstorm
description: Entry point for any new feature or change — interview the user one question at a time to stress-test the idea, explore feasibility and impacts against the codebase, and converge on decisions. Use when the user expresses a need, wants to brainstorm or stress-test a plan or design, or says "grill me". Ends by routing to write-prd (project-scale feature) or create-issue (single issue).
argument-hint: "<the need or idea to explore>"
---

# Brainstorm

Interview the user relentlessly about the idea until you reach shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one by one.

## Rules of the interview

- Ask questions **one at a time**; wait for the answer before the next one.
- For every question, **give your recommended answer** with a one-line rationale.
- If a question can be answered by exploring the codebase, **explore instead of asking**.
- Adapt the depth to the topic: product-level ideas get product questions (who is it for, what problem, what is the smallest useful version); technical plans get technical ones (data model, integration points, failure modes, migration path).
- Stress-test with concrete scenarios: invent edge cases that force precision ("what happens when X and Y occur at the same time?").
- Probe feasibility early: identify the riskiest assumption and attack it first — better to kill an idea in question 3 than in week 3.

## Ground in the codebase

Before and during the interview, explore the repo:

- Existing modules, patterns, and utilities the idea would touch or could reuse
- Behavior the idea would conflict with or change
- If the project keeps domain docs (`CONTEXT.md`, `CONTEXT-MAP.md`, `docs/adr/`), use their vocabulary, challenge terms that conflict with the glossary, and respect recorded decisions. Update those docs as decisions crystallise — but only in repos that already maintain them; don't introduce the structure elsewhere.

## Converge

Track decisions as they are made. When the branches are resolved (or the user says stop), summarize:

- **The need** — one paragraph
- **Decisions made** — bullet list
- **Open questions** — what is still unresolved and who owns it
- **Feasibility risks** — anything that could invalidate the plan

## Route to the next step

End by proposing the right artifact:

- **Project-scale feature** (several issues' worth, spans multiple areas) → `/workflow:write-prd` — a PRD on the Linear project.
- **Single issue** (one screen, one endpoint, one behavior) → `/workflow:create-issue` — a feature sheet in a Linear issue.
- **Not worth an artifact** (decided against, or trivial enough to just do) → leave the summary in the conversation and say so.

Don't produce the document inside this skill — hand off so the dedicated skill applies its structure to the full context you just built.
