---
name: dynamic-workflows-for-analytics
description: Structures analytical work as parallel agent workflows. Use when a task decomposes naturally along independent dimensions (SKU × region × time), when output has a clear correctness criterion (data reconciles, schema validates, tests pass), and when running it sequentially would take hours. Not for open-ended interpretation — only for computation with a checkable result.
---

# Dynamic Workflows for Analytics

## Overview

Map analytical work onto parallel agent workflows when the task splits cleanly along independent dimensions. Each agent owns one cell of the decomposition; reviewer agents validate the output; a final agent aggregates. The workflow cycles until all cells pass their acceptance criteria.

This is not a general analytics pattern — it only applies when the work is computation-heavy and the result is verifiable without human judgment.

## When to Use

- Dataset is large and splits naturally: SKU × region × period, store × category, brand × channel
- Acceptance criterion is mechanical: data reconciles with source, schema matches, SQL returns expected row count, tests pass
- Sequential execution would take hours and adds no value over parallel
- You're migrating or rebuilding something (ETL, reports, models) rather than interpreting something

**When NOT to use:** Questions that require business context ("why did sales drop?"), first-time exploration of a new dataset, or when correctness can only be judged by a domain expert.

## FMCG-Specific Decompositions

| Task | Decomposition axis | Acceptance criterion |
|---|---|---|
| Promotional post-analysis | One agent per promotion event | Revenue lift ± matches control; margin delta computes without error |
| ETL for new retailer feed | One agent per retailer format | Row counts match source; no nulls in key columns; schema validates |
| Weekly category reports | One agent per category | Report renders; no division-by-zero; period-over-period delta computes |
| Price elasticity models | One agent per category × region | Model converges; R² above threshold; coefficients are sign-consistent |
| Data quality audit (Nielsen/IRI) | One agent per data dimension | Coverage % meets SLA; no duplicate keys; date spine complete |

## Process

### Step 1: Define the decomposition

Split the task along the axis that makes cells independent. A cell is independent when it can be processed without knowing the result of any other cell.

Write the decomposition as a list before spawning agents:
```
[category=snacks, region=south]
[category=snacks, region=north]
[category=dairy, region=south]
...
```

### Step 2: Define acceptance criteria per cell

Each cell must have a binary pass/fail test the reviewer agent can run without human input:
- Row counts match source system
- No nulls in required fields
- Aggregates reconcile to control totals
- Unit tests pass

If you cannot write the acceptance criterion before starting, the task is not ready for a workflow.

### Step 3: Spawn agents

Enable in Claude Code: `/config → Dynamic workflows`, then `/effort ultracode`.

Either let Claude decide automatically, or trigger explicitly: `"Create a workflow to [task] decomposed by [axis], with acceptance criteria: [criterion]"`.

### Step 4: Review and aggregate

Two reviewer agents per cell is the minimum. Final aggregation agent runs after all cells pass. If any cell fails, the workflow cycles — fix and rerun that cell only.

## Token Budget

Dynamic workflows consume significantly more tokens than a sequential session. Estimate before starting:

- Small decomposition (10–20 cells): acceptable for one-off migrations
- Medium (50–100 cells): use for recurring high-value work, not ad-hoc queries
- Large (500+ cells): only justified when manual equivalent takes days

Start with a 5-cell pilot on the hardest cells. If those pass cleanly, scale to the full decomposition.

## Red Flags

- You find yourself writing acceptance criteria that say "looks reasonable" — stop, this is not a workflow task
- Cells are not truly independent (cell B needs cell A's output) — restructure as sequential phases, parallelize within each phase
- The decomposition has only 2–3 cells — just run them sequentially, the orchestration overhead isn't worth it
- You're using a workflow to explore unfamiliar data — exploration requires human judgment at each step

## Reference

- See `planning-and-task-breakdown` for how to structure the dependency graph before decomposing
- See `context-engineering` for how to pass only the relevant slice of context to each agent cell
- [sennin.ai](https://sennin.ai) — example of context-scoped agent harness (passes only task-relevant context, not the full session)
