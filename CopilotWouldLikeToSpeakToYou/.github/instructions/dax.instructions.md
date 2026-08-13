---
applyTo: '**/definition/tables/*.tmdl'
description: 'DAX measure and calculated column review'
---

# DAX review

## Correctness and safety

- **`DIVIDE` over `/`.** `DIVIDE(a, b)` handles division by zero and blank denominators. `IF([x] = 0, BLANK(), [a]/[x])` is the long way round and often guards the wrong thing. — [STATIC]
- **Boolean predicates over `FILTER` where possible.** `CALCULATE(COUNTROWS(t), t[col] = "x")` pushes to the storage engine; `CALCULATE(COUNTROWS(t), FILTER(t, t[col] = "x"))` materialises the table first. **Important exception:** a predicate comparing two columns (`t[a] <= t[b]`) *requires* `FILTER` — a boolean filter argument only accepts column-vs-constant. Do not "fix" those. — [STATIC]
- **`SELECTEDVALUE` over `IF(HASONEVALUE(...), VALUES(...))`.** Same result, one function. — [STATIC]
- **`REMOVEFILTERS` over `ALL` when the intent is filter removal.** `ALL` used as a table function inside `CALCULATE` removes filters from every column of the table, which is usually more than intended and silently wrong in subtotals. — [STATIC]
- **No hardcoded years, IDs, or codes.** `CALCULATE([Sales], 'Date'[Year] = 2024)` breaks the following year. A status or category code compared to a bare integer means the codebook lives in someone's head. Flag it as [SOURCE] and ask what the code means rather than guessing.
- **Nested `IF` beyond two branches → `SWITCH(TRUE(), ...)`.** Readability, not speed. — [STATIC]
- **Blank vs zero.** `COUNTROWS` returns blank, not zero, on an empty filter context. If the visual needs a zero, say so explicitly rather than letting blank rows disappear.

## Performance

- **`VAR` for any expression evaluated more than once.** Repeated subexpressions are recomputed each time; a variable is evaluated once. — [STATIC]
- **Watch iterators with context transition.** `AVERAGEX(VALUES(t[high_cardinality_col]), CALCULATE(...))` forces per-row work the engine may not fold. Flag it as a candidate, but classify actual slowness as **[RUNTIME]** — you cannot tell from the text whether the engine optimises it away on this data. Recommend Performance Analyzer, do not predict milliseconds.
- **Never claim a rewrite is faster.** Say it *should* reduce work, and that the query plan is the only proof.

## Wrong layer

Cost ladder, cheapest first: **source → Power Query → DAX**. A calculated column is the most expensive place to put anything, because it is materialised in the model and cannot be folded.

Flag any calculated column doing work that belongs upstream — [STATIC]:

- String cleanup: `TRIM`, `UPPER`, `SUBSTITUTE`, `LEFT`/`RIGHT` parsing.
- Banding or bucketing with nested `IF` — a conditional column in Power Query, or a lookup in the source.
- Concatenating a composite key to enable a relationship — build the key upstream, or fix the schema so it isn't needed.
- `LOOKUPVALUE` pulling one attribute across tables — that is a merge, or a relationship that should already exist. **If the relationship does exist, the column is both wrong-layer and redundant. Say both.**
- Date parts (`YEAR`, `FORMAT(...,"mmm")`) on a fact table — these belong on a date dimension.
- A "cleaned" column kept alongside the dirty original, doubling storage for one usable field.

Calculated columns that are genuinely fine: ones needing filter context or measure references that Power Query cannot express.

## Hygiene

Check every measure for — [STATIC]:

- A **format string**. Percentages, currency, and decimal places should not be left to the visual.
- A **description**, especially where the name is ambiguous or the logic is non-obvious.
- A **display folder** once the table has more than a handful of measures.
- A **consistent home table.** Measures scattered across fact tables by accident of where the author was standing make a model hard to navigate. Note the pattern; do not move them without asking.
- **Naming drift** — `Rate` vs `%` vs `Pct` for the same idea, or near-identical measures whose logic has diverged. Comparing several measures at once is exactly the review a per-object tool cannot do.

## Two things you cannot see

State these plainly rather than glossing them:

- **Whether the number is right.** Valid DAX over sentinel dates, unfiltered cancelled rows, or the wrong grain produces a confident, wrong answer. — [RUNTIME]
- **Whether the aggregation matches the grain.** `AVERAGE(order_items[freight])` labelled "per order" averages per *item*. You can flag that the name and the table disagree; you cannot confirm the intent. — [RUNTIME] + [SOURCE]
