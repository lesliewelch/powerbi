---
applyTo: '**/definition/tables/*.tmdl'
description: 'Power Query (M) partition review'
---

# Power Query review

The `partition <table> = m` block holds the M query. Read the whole step chain before commenting — order is the substance here.

## Query folding

Folding is the engine translating your steps into a single native query the source executes. When it breaks, everything downstream is pulled into memory and processed locally. **Folding stops at the first step that cannot be translated, and never resumes.**

Common breakers — flag with the step name — [STATIC]:

- `Table.Buffer` — an absolute stop. Usually added to fix a symptom, and it forces a full local load every refresh.
- `Table.AddIndexColumn` — no SQL equivalent.
- Custom columns using M-only functions (`Text.Proper`, `Text.Clean`, most `Splitter.*`, custom lambdas).
- Merges against a non-foldable query, or against a different source entirely.
- Steps added *after* any of the above — they inherit the breakage.

**Step order matters more than step count.** Filters and column removal belong **first**, so the source does the work. A filter as the last step means every row was fetched and then discarded locally. — [STATIC]

**You cannot confirm folding from code.** You can identify a construct that breaks it; whether it broke *here*, and what it cost, is **[RUNTIME]** — verified by right-clicking the last step and checking whether **View Native Query** is available. Always recommend that check rather than asserting.

## Data fabrication — the highest-severity finding

These steps produce plausible, wrong data silently. Treat them as more serious than any performance issue — [RUNTIME]:

- **`Table.FillDown`** — propagates the previous row's value into nulls. Correct for genuinely report-shaped sources; catastrophic on a normalised table, where it invents attributes. Ask why the nulls exist before accepting it.
- **`Table.ReplaceErrorValues`** — patches over a failed type conversion instead of fixing the type. The errors were telling you something.
- **Replacing nulls with zero** on a measure input, turning "unknown" into a real value that averages.

Say clearly: the M is valid and the output is fiction. No static analysis catches this — a human has to compare against the source.

## Maintainability

- **Consolidate `Table.TransformColumnTypes`.** A trail of `Changed Type1`, `Changed Type2`… is the fingerprint of clicking the ribbon column by column. One typing step per table. — [STATIC]
- **Parameterise connections.** A hardcoded server, database, or file path repeated across queries means an environment change is an edit in every one. — [STATIC]
- **`Table.SelectColumns` over `Table.RemoveColumns`.** Choosing what to keep survives a new source column; removing named ones silently admits it. — [STATIC]
- **Reference, not Duplicate.** A duplicated query copies the entire upstream chain, and the two will drift. — [STATIC]
- **Staging queries should not load.** An intermediate query with load enabled materialises a second copy in the model for nothing. — [STATIC]
- **Hardcoded exclusion lists.** `each [x] <> "a" and [x] <> "b" and ...` comes from unticking checkboxes and breaks the moment a new value appears. Invert to an inclusion list, or drive it from data. — [STATIC]
- **Round trips.** Merge → expand every column → remove most later; split by delimiter → rename → merge back; a cosmetic `Table.ReorderColumns`. Each pins column names and buys nothing. — [STATIC]
- **Generated `if/then` chains** from Column From Examples — unreadable, unmaintainable, and often a lookup table hardcoded into an expression. If it encodes a codebook, the real fix is joining the lookup. — [STATIC] → [SOURCE]

## Rename steps

Auto-generated names (`Changed Type3`, `Added Custom1`) make a chain unreadable. Suggest intent-revealing names — but note it is cosmetic, and never renumber steps without updating every downstream reference.
