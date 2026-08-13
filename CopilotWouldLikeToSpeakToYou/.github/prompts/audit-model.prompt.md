---
mode: 'agent'
description: 'Full semantic model audit, classified by what is provable from code'
---

# Full model audit

Review this semantic model end to end. Work from the files, not from assumptions.

## Read first

1. Every file in `*.SemanticModel/definition/tables/` — columns, calculated columns, measures, and the M partition in each.
2. `relationships.tmdl` — cardinality and cross-filter direction on every relationship.
3. `model.tmdl` — annotations and model-wide settings.
4. `*.Report/` — **read only**, to establish which fields are actually used.

## Report in this order

**1. Root causes.** Structural choices that generate multiple downstream symptoms — the schema shape, the storage decisions, the layer things were built in. One or two items. Lead here, not with a list of small fixes.

**2. Patterns across objects.** Findings only visible when several objects are compared: repeated constructs, near-identical measures whose logic has drifted, an inconsistency applied everywhere (missing format strings), a convention followed in most places and broken in a few. **This is the highest-value section — spend real effort here.** A reviewer looking at one object at a time cannot produce it.

**3. [STATIC] fixes.** Provable from code. Group by kind, not by file. Give the rewrite for each.

**4. [RUNTIME] risks.** Constructs that *may* be wrong or slow, where the code alone cannot tell you. Say what would confirm it — Performance Analyzer, a row count, comparing against the source. Do not present these as findings; present them as checks for a human.

**5. [SOURCE] questions.** Anything needing the database or the business: what a code means, whether a filter is correct policy, why a column exists. Phrase as questions, not recommendations.

## Rules

- Quote the line you are reacting to and name the object.
- Separate refactors (same result) from fixes (different result). Never change a number silently.
- If a section has nothing in it, say so. Do not manufacture findings to fill the shape.
- End with what you could **not** assess and why.
