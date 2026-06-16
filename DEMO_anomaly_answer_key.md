# Demo Anomaly: Answer Key (internal, do not show the audience)

This is your ground truth for the staged data quality issue. The audience should discover it; you should know it cold.

## What was seeded

A single, realistic cost feed error. For **Contoso Store Kansas (StoreKey 500)**, every sales line in **September 2025** had its `UnitCost` multiplied by 10, consistent with a feed that loaded the cost one decimal place to the right (dollars written where dimes were expected, or a misplaced decimal on load). Nothing else was touched. Quantity, price, and revenue are all original, so volume looks completely normal and only margin breaks.

Source file is the untouched `sales.csv`. The demo file `sales_demo.csv` is identical except for those 53 cost cells.

## Ground truth

| Item | Value |
|---|---|
| Store | 500, Contoso Store Kansas, United States |
| Window | September 2025 (full month) |
| Lines affected | 53 |
| Change | `UnitCost` x10 (decimal shift) |
| Store revenue | $24,854 (unchanged) |
| Store gross margin % | +51.6% -> -383.5% |
| Company Sept-2025 gross margin % | 55.97% -> 51.52% (a 4.45 point dip) |
| OrderKey range | 3533009 to 3561032 |

Company gross margin is a steady ~56% every other month in 2025, so September is the only month off-band. That stability is what makes the dip noticeable without being loud.

## Why it shows but the source is not obvious

- The dip is visible at the company level (about 4.5 margin points) but small enough that it does not scream which store.
- Revenue and volume for store 500 are completely normal, so anything built on sales volume sees nothing wrong. You only catch it on a margin measure.
- The online store (StoreKey 999999) is roughly half of all revenue and is perfectly healthy. It dominates any revenue-sorted view and pulls the eye away from a small Kansas store.
- To find it you have to look at margin percent by store, not revenue, which is exactly the disciplined habit this audience prides itself on.

## The investigation path (how to run the reveal)

1. **Notice (monitoring board, self-built in the service).** Gross margin % by month shows September below the steady ~56% line. Revenue for September looks normal, which is the first clue: this is a cost problem, not a sales problem.
2. **Isolate (still in Power BI).** Margin % by store surfaces one store deep in the red while every other store sits near 55 to 59%. Sorting by margin %, not revenue, is the move.
3. **Confirm it is a feed issue, not a product issue.** Break store 500 down by category. Every category is negative, so it is not one bad product, it is the whole store for that period. That points to a load or feed problem.
4. **Bound it in time.** Margin % by day for store 500 shows the break begins and ends inside September. A dated window is the signature of a one-time bad load.
5. **Root cause.** Costs are exactly 10x expected, a decimal shift in the cost feed for that store and month.
6. **Hand off (paginated).** The exact 53 affected lines are in `affected_lines_store500_sep2025.csv`, including the loaded cost and the original cost, ready to format as a paginated report for the data team to fix same day.

## In-meeting Copilot prompts that surface it

- "Show gross margin percent by month for 2025."
- "Show gross margin percent by store for September 2025." (store 500 stands out)
- "For store 500 in September 2025, show gross margin percent by product category." (all negative)
- "List the orders for store 500 in September 2025 with unit cost and net price."

These also double as your credibility beat: each answer is something you could explain by hand if asked, because the measures are defined in the model. Not a black box.

## The seasonal contrast beat (optional, high value)

Use April 2025 as a deliberate counter-example to show judgment, not just tooling.

- **April 2025:** revenue is low (~$975K vs a ~$2.5M norm) but gross margin is a normal 55.5%. Low volume, healthy margin. This is **seasonal**, and the dip is uniform across categories and stores. Not a problem.
- **September 2025 (seeded):** revenue is normal but margin breaks in one store. Healthy volume, broken margin, concentrated in one source. This is a **data problem**.

The one-liner: low revenue with healthy margin is seasonality, healthy revenue with broken margin is a data issue. That distinction is the expertise they are worried about losing, shown to be stronger here, not weaker.

## Files and how to use them

- `sales_demo.csv` — load this as your `sales` table for the demo (or rename to `sales.csv` if your model is wired to that name). Same schema and row count as the original.
- `affected_lines_store500_sep2025.csv` — the 53 seeded lines for the paginated handoff moment.
- `DEMO_anomaly_answer_key.md` — this file. Keep it to yourself.

If you want the dip louder or quieter, the only knob is the multiplier. 10x gives the ~4.5 point company dip above. Lower it toward 3x to make it subtler, raise it to make it blunter.
