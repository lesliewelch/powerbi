# CLAUDE.md — Model-layer build (deliberately flawed semantic model)

Project context for building the demo model for the "Copilot Would Like to Speak to You About Your Model" talk. Read this first, then `model-plant-plan.md` (the what) and `olist-source-schema.md` (source truth + exact names).

## What this project is

- A **Power BI Project (PBIP)** in **TMDL** format. The semantic model is plain text under `*.SemanticModel/definition/`.
- The source is **already imported** from Azure SQL (`datacopilotdemo.dbo`). Data is loaded, relationships are auto-detected, injections are applied. You work **only on the model-definition text files** — you do not connect to the database and do not touch the import queries.
- Purpose: build a **deliberately flawed** semantic model. **The flaws are the deliverable.** This model is the artifact the talk dissects.

## The one rule that matters most

Implement every measure and calculated column in `model-plant-plan.md` **exactly as written — flaws intact.**

**Do NOT** fix, optimize, refactor, simplify, add variables, add `DIVIDE`, collapse nested `IF` into `SWITCH`, add format strings, add display folders, rename for clarity, or otherwise "improve" the DAX or the model. If something looks wrong, that is the point — leave it. If a spec item looks like a mistake, it is intentional; do not correct it. When in doubt, be more faithful to the flaw, not less.

This will feel wrong to you. Do it anyway. Any "improvement" destroys a teaching moment.

## Scope — do / don't

**Do:**
- Add the calculated columns (plant plan A3) to `tables/orders.tmdl`.
- Add measures to the relevant table `.tmdl` files, **scattered on purpose** (some on `orders`, some on `order_items`, review measures on `order_reviews`). **No dedicated measures table.**

**Don't:**
- Don't modify `relationships.tmdl`. The relationships are already auto-detected into the intended messy state. In particular, **do not delete the bidirectional `customers → orders` relationship** and **do not delete the `products → category_translation` relationship** — both are intentional. (The report will just never use the English category column; that's the lesson, not a bug to fix.)
- Don't import, reference, or relate **`status_code`** — it stays out of the model (discoverability beat).
- Don't relate **`geolocation`** to anything — it stays an unused ~1M-row table (bloat lesson).
- Don't modify any `partition <table> = m` block — that's the loaded source query.
- Don't modify `model.tmdl` — leave auto date/time ON (`__PBI_TimeIntelligenceEnabled = 1`) and the nine `LocalDateTable_*` tables in place.
- Don't touch anything under `*.Report/`. **Never hand-author the report JSON (PBIR) visuals** — Leslie builds those by hand in Desktop from the wireframe.
- Don't add format strings or display folders anywhere.

## Key fact: dates are already `dateTime`

The date columns (`order_purchase_timestamp`, `order_delivered_customer_date`, etc.) imported as `dataType: dateTime`, not text. So the calc columns use them **directly — no `DATEVALUE`, no conversion.** `DATEDIFF ( orders[order_purchase_timestamp], orders[order_delivered_customer_date], DAY )`. The `9999-12-31` sentinel is a valid far-future datetime and still detonates the average — the flaw is intact.

## How to write TMDL

1. **Match the existing files exactly.** Open `tables/orders.tmdl` and copy its style: **tab indentation, CRLF line endings, UTF-8.** (This project uses CRLF, not LF — match it.)
2. **Measure:** `measure 'Name' = <DAX>` indented one tab under the table; multi-line expressions indent further. Measures are the safe, reliable edit.
3. **Calculated column:** `column 'Name' = <DAX>` with `dataType:` and `summarizeBy:` set to match neighboring columns. Calculated columns have **no** `sourceColumn`.
4. **Do not hand-write `lineageTag` or GUIDs** on new objects. Omit them — Power BI Desktop assigns them automatically on next open. (Existing objects have them; that's fine, leave those alone.)

Illustrative — match the real file, which wins if it differs (note: real files use CRLF + tabs):
```tmdl
table orders

	column 'Delivery Days' =
			IF (
				NOT ISBLANK ( orders[order_delivered_customer_date] ),
				DATEDIFF ( orders[order_purchase_timestamp], orders[order_delivered_customer_date], DAY )
			)
		dataType: int64
		summarizeBy: sum

	measure 'Avg Delivery Days' = AVERAGE ( orders[Delivery Days] )
```

## Workflow gotcha — read this

- **Power BI Desktop must be closed while you edit the TMDL, then reopened to load the changes.** Edits made outside Desktop don't appear until it restarts. The loop is: Leslie closes Desktop → you edit the `.tmdl` files → Leslie reopens Desktop → validates.
- **Measures and calculated columns in existing tables are the reliable case.** If the optional DAX `Date` table (plant plan Section C) is requested and a raw-TMDL calculated table doesn't materialize cleanly, don't fight the file — flag it, and Leslie will paste the DAX via Desktop's TMDL view / New Table. The DAX is correct; only the TMDL packaging is finicky.

## Task order

1. Calculated columns (A3) → `tables/orders.tmdl`: Order Year, Order Month, Delivery Days, Ship Days, Approve Days, Carrier Days, Lateness Bucket.
2. Page 1 measures, then Page 2 measures — scattered across orders / order_items / order_reviews.
3. (Optional, only if asked) Section C extras.

## Done when

- Every calculated column and measure in the plant plan exists, verbatim, flaws intact.
- `relationships.tmdl`, `model.tmdl`, all `partition = m` blocks, and the entire Report folder are untouched.
- `status_code` absent; `geolocation` unrelated; `category_translation` still related but the English column unused; no format strings; no display folders; auto date/time still on; measures scattered across tables.
- The project opens in Desktop without breaking (the NULL guards keep calc columns from producing garbage) and the flawed numbers compute — `Avg Delivery Days` is absurdly large from the sentinel; category visuals show Portuguese names.
