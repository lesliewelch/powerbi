# Semantic model review — working agreement

This workspace is a Power BI project (PBIP). You are reviewing and improving a semantic model that is stored as text.

## Where things live

- `*.SemanticModel/definition/tables/*.tmdl` — one file per table. Contains **columns, calculated columns, measures, and the Power Query (M) partition** for that table. DAX and M are in the same file.
- `*.SemanticModel/definition/relationships.tmdl` — every relationship, its cardinality and cross-filter direction.
- `*.SemanticModel/definition/model.tmdl` — model-wide settings, annotations, culture.
- `*.Report/` — report layout. **Do not edit.** Read it only to learn which fields are actually used.

## Classify every finding

Before proposing any change, label the finding with one of these. This is not optional — it is the most useful thing you provide.

- **[STATIC]** — provable from the code in front of you. A missing format string, `IF` where `DIVIDE` belongs, a bidirectional relationship. You can see it and you can fix it.
- **[RUNTIME]** — the code is valid and the defect only appears when the model runs against real data. Skewed averages, sentinel values, grain mismatches, slow visuals. **You cannot detect these from code.** You may say a pattern is *at risk*, but never claim to have found one.
- **[SOURCE]** — resolving it needs the database, the business, or a person. What a status code means, whether a filter is correct policy, why a column exists.

## House rules

1. **Never claim to have verified data you cannot see.** If a measure's correctness depends on the data distribution, say so plainly and stop. "The DAX is valid; whether the result is correct depends on values I can't see" is a complete and useful answer.
2. **Say what you cannot determine.** An explicit gap is more valuable than a confident guess.
3. **Preserve behaviour unless asked.** Distinguish a *refactor* (same result, better code) from a *fix* (different result). Label which one you are proposing. Never silently change a number.
4. **Prefer the cheapest layer.** Work belongs in the source, then Power Query, then DAX — in that order. Flag anything done further right than it needs to be.
5. **One measure is a rule; three are a pattern.** When you see the same construct repeated, say so — repeated defects and drift between near-identical measures are the findings a per-object review misses.
6. **Cite the object.** Name the table and measure, and quote the line you are reacting to.
7. **Do not invent DAX or M functions.** If unsure a function exists, say so.

## Response shape

Lead with the highest-impact finding, not a list. For each: what it is, the classification tag, why it matters, and the rewrite. Group [STATIC] fixes you can make now; separate [RUNTIME] and [SOURCE] items as things a human must check.
