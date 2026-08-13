---
mode: agent
description: Trace query folding through every partition and find where it breaks
---

# Query folding audit

Examine the Power Query `source = let … in …` expression in every `definition/tables/*.tmdl` partition and determine where folding is likely lost.

## For each table

1. **List the steps in order**, with the M function each uses.
2. **Identify the first step the source probably cannot express.** Common breakers: `Table.AddIndexColumn`, `Table.Buffer`, `Table.AddColumn` with a custom expression, `Table.Sort`, a merge against a non-foldable query, most `Text.*` and `Date.*` work in a custom column.
3. **Say what runs locally as a result** — every step after the break, against the full table, on every refresh.
4. **Check what comes after the break.** Filters and column removals sitting downstream of a breaker are the expensive mistake: they can no longer reduce what the source sends.
5. **Propose a reordered chain.** Filters and column selection first; unavoidable non-foldable work last. Give the complete rewritten `let` expression.

## Report as

A table: table name · step count · first suspected breaker · what's stranded after it · whether reordering would help.

Then the rewrites, longest-pole first. Note the source type (SQL, file, API, Direct Lake) since it determines what can fold at all — a CSV folds nothing and reordering won't change that.

## Constraints — read these before writing anything

**You cannot determine folding status from code.** You are reasoning about what a source *probably* supports. Every finding here is **[RUNTIME]** until confirmed in Power Query Editor: click the last step, right-click, and check whether **View Native Query** is available. Say this explicitly in the report. Do not write "folding is broken" — write "folding is likely lost at step N; confirm with View Native Query."

**`Table.Buffer` may be deliberate.** It legitimately prevents repeated re-evaluation in some merge and nested-join patterns. Ask what it's for rather than removing it. Removing a purposeful buffer can make refresh slower.

**Reordering can change results.** Moving a filter earlier is safe; moving it across a step that adds or renames columns is not. Flag any reordering that isn't provably result-neutral.

**Preserve the column set.** If the model's TMDL declares a column, the M must still return it. A rewrite that drops a declared column breaks refresh.

**Preserve TMDL dialect:** tab indentation, CRLF, UTF-8, and the existing indentation of the `source =` block. Power BI Desktop must be closed while these files are edited.
