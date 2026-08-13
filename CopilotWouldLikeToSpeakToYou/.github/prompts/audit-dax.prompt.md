---
mode: agent
description: Review all DAX measures and calculated columns, including cross-measure patterns
---

# DAX audit

Review every `measure` and calculated `column` across all `definition/tables/*.tmdl` files.

Read them **all** before reporting. The most valuable findings are patterns across measures, and those are invisible one file at a time.

## Per measure

- Correctness risks: unguarded division, filters that don't do what the name implies, a base or denominator that includes rows the label excludes.
- Readability: nested `IF` where `SWITCH` belongs, repeated subexpressions that want a `VAR`, no format string, no description.
- Efficiency: `FILTER` over a table where a boolean predicate would serve — **but not when the condition compares two columns of the same table, where `FILTER` is required.** Check before recommending; a broken rewrite is worse than the original.
- Hardcoded literals: years, status codes, thresholds. If a code's meaning isn't in the model, that's **[SOURCE]**, and the fix may be a lookup table rather than a DAX edit.

## Per calculated column

Ask first whether it should exist at all. The cost ladder is **source → Power Query → DAX calculated column**, cheapest first, and a calculated column is stored and refreshed every time.

Flag as wrong-layer: string cleanup, date parts on a fact table, banding with nested `IF`, concatenated keys, `LOOKUPVALUE` across tables. Before recommending removal of a `LOOKUPVALUE` column, check `relationships.tmdl` — if the relationship already exists, the column is redundant as well as misplaced, which is a stronger finding.

**But not every calculated column is wrong.** Relationship keys, `sortByColumn` targets, and slicer fields must be columns; a measure cannot serve those. Say so rather than reflexively recommending a measure.

## Cross-measure patterns — the section that matters most

- **Copy-paste families.** Near-identical measures that have drifted apart: one uses `DIVIDE`, its twin uses `IF`; one filters a status, its twin forgot. Report the family together with one consistent rewrite, and say which variant is right.
- **Inconsistent conventions** — naming, filter approach, blank handling — applied differently in different places.
- **Model-wide gaps with counts**: how many measures have no format string, how many have no description, how many sit on fact tables rather than a measures table.
- **Measures that exist to work around the model** rather than use it. These point at a relationship or grain problem, and fixing the model deletes the measure entirely — the most valuable finding you can produce.

## Tagging

- **[STATIC]** — the defect is in the text.
- **[RUNTIME]** — valid DAX whose correctness depends on data you can't see. Sentinel values, unexpected blanks, grain mismatches, and mislabelled averages all live here. State the hypothesis and how to check it. **Never claim you fixed one.**
- **[SOURCE]** — needs the upstream database or domain knowledge.

Performance claims are **[RUNTIME]** without exception. You cannot tell from DAX how long a query takes; the engine fuses and caches in ways the text doesn't reveal. Point at Performance Analyzer — Start, interact with a slicer to force a cache miss, Stop, read the DAX query line — rather than asserting a speedup.
