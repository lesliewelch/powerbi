---
mode: 'ask'
description: 'Review a single measure or calculated column in model context'
---

# Review this measure

Review the selected DAX. Do not stop at the expression — read its context.

1. **What it does.** Restate the logic plainly. If your reading and the object's name disagree, that gap is the finding.
2. **Correctness.** Filter context, grain, blank handling, division safety. Check the tables it touches: does the aggregation match the grain of the column being aggregated?
3. **Neighbours.** Look for other measures in this model doing nearly the same thing. Divergence between near-identical measures is a real defect and only visible by comparison — report it if you find it.
4. **Layer.** If it is a calculated column, could it be a measure, or better, work done in Power Query or the source?
5. **Rewrite.** Only if it improves correctness, readability, or reduces work. Say which. If it is already fine, say that instead — a clean bill is a valid result.

Classify every point as **[STATIC]** (provable here), **[RUNTIME]** (depends on data you cannot see), or **[SOURCE]** (needs the database or a person).

Be explicit about what the expression alone cannot tell you. If correctness depends on the values in the columns, say so and stop — do not guess at the data.
