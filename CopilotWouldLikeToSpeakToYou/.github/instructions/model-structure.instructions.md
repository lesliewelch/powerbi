---
applyTo: '**/definition/{relationships,model}.tmdl'
description: 'Relationships, storage and model-wide settings'
---

# Model structure review

## Shape

A semantic model wants a **star schema**: narrow fact tables surrounded by dimensions, one relationship deep. Two shapes to flag — [STATIC]:

- **Source schema imported as-is.** An OLTP structure carried into the model unchanged is the parent defect: it produces the snowflakes, the grain traps, and the bidirectional filters that follow. Name it as the root cause rather than reporting each symptom separately.
- **Snowflaked dimensions** — dimension → dimension → fact. Usually collapsible into one table in Power Query.

## Relationships

- **Single direction by default.** Bidirectional cross-filtering creates ambiguity, can produce circular paths, and costs performance. It is occasionally correct — for a genuine many-to-many bridge — but it is far more often an accident of autodetect or a workaround for a measure that should use `CROSSFILTER`. Flag every one and ask for justification. — [STATIC]
- **Many-to-many.** Legitimate only with a deliberate bridge table. A m2m arising from a non-unique key on the "one" side is a modelling error. — [STATIC]
- **Key types.** Relationships on long text or hash keys cost far more memory and are slower to join than integer surrogates. Flag them, and note the fix is upstream. — [STATIC]
- **Inactive relationships** with no `USERELATIONSHIP` anywhere are dead weight — check whether any measure activates them.
- **Missing relationships.** If two tables share an obvious key and aren't related, say so — but classify as [SOURCE], since whether they *should* join is a business question.

## Storage and size

- **Auto date/time.** `__PBI_TimeIntelligenceEnabled = 1` generates a hidden date table for **every date column in the model**, each with its own hierarchy. On a model with several date columns this is significant, invisible bloat. Recommend turning it off and using one marked date table. — [STATIC] to detect, [RUNTIME] to size.
- **Datetime columns.** A datetime with a time component has near-row-level cardinality and compresses poorly. If the time is not used, split date from time upstream. — [STATIC]
- **Unused tables and columns.** A table imported and never related or referenced is pure cost. Cross-check against `*.Report/` and the measure definitions before calling a column unused — absence from the report is evidence, not proof, since it may be used in RLS or by a downstream consumer. Report it as a candidate.
- **`summarizeBy` on keys.** ID columns default to summing. A visual can silently show the sum of an order ID. Set `summarizeBy: none` on every key. — [STATIC]
- **Hide keys and technical columns** from report view.

## Date dimension

- One **marked date table**, contiguous, covering the full fact range, sourced from Power Query or the source — not a DAX calculated table.
- Month names need a **sort-by column**, or they sort alphabetically. This is one of the most common defects in real models, and it is trivially visible in metadata. — [STATIC]
