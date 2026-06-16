# Semantic Model: Measure Catalog (DAX)

All measures live in a single physical `_Measures` table (one hidden dummy column, table itself stays visible). Adjust the table names in the DAX below to match your actual model if your loaded tables are named differently (for example `sales` vs `Sales`).

Format strings use the Power BI format language. Display folders group the measures in the field pane.

---

## Revenue and Cost

### Revenue
```
Revenue = SUMX(Sales, Sales[Quantity] * Sales[NetPrice])
```
- Description: Total revenue from all sales lines, calculated as the sum of Quantity times Net Price. Net Price is the actual selling price after discount, not the list price. This is the numerator for margin.
- Format string: `$#,##0.00`
- Display folder: Revenue and Margin
- Synonyms: total revenue, sales, net revenue, total sales, sales revenue, gross revenue, Revenue, revenue, total_revenue, rev

### Cost
```
Cost = SUMX(Sales, Sales[Quantity] * Sales[UnitCost])
```
- Description: Total cost of goods sold across all sales lines, calculated as the sum of Quantity times Unit Cost. This is the landed cost on the sale, not the reference catalog cost from the product table.
- Format string: `$#,##0.00`
- Display folder: Revenue and Margin
- Synonyms: total cost, cogs, cost of goods sold, landed cost, Cost, cost, total_cost, cost_of_goods

### Gross Margin
```
Gross Margin = [Revenue] - [Cost]
```
- Description: Absolute gross profit in dollars, calculated as Revenue minus Cost. A negative value means the company is selling below cost, which signals a cost data problem rather than a pricing decision.
- Format string: `$#,##0.00`
- Display folder: Revenue and Margin
- Synonyms: gross profit, profit, margin dollars, gross margin dollars, Gross Margin, gross_margin, gp_dollars

### Gross Margin %
```
Gross Margin % = DIVIDE([Revenue] - [Cost], [Revenue])
```
- Description: Gross margin as a percentage of revenue. Normal range is about 55 to 57 percent. A dip below 50 percent or a negative value points to a cost feed or data quality issue, not a sales issue. This is the primary measure for spotting anomalies.
- Format string: `0.0%`
- Display folder: Revenue and Margin
- Synonyms: margin percent, margin %, gm%, gross margin percent, profit margin, margin percentage, Gross Margin %, gross_margin_pct, gm_pct, margin_pct

---

## Orders and Volume

### Orders
```
Orders = DISTINCTCOUNT(Sales[OrderKey])
```
- Description: Number of distinct orders. One order can contain multiple lines. Use this for order frequency, and use Quantity for unit volume.
- Format string: `#,##0`
- Display folder: Orders and Volume
- Synonyms: order count, number of orders, distinct orders, how many orders, Orders, orders, order_count, num_orders

### Lines
```
Lines = COUNTROWS(Sales)
```
- Description: Number of order lines, where each product within each order counts as one line. The most granular transactional count.
- Format string: `#,##0`
- Display folder: Orders and Volume
- Synonyms: line count, number of lines, item count, items, Lines, lines, line_count, num_lines

### Total Quantity
```
Total Quantity = SUM(Sales[Quantity])
```
- Description: Total units sold across all orders.
- Format string: `#,##0`
- Display folder: Orders and Volume
- Synonyms: total quantity, units, units sold, total units, volume, qty, Total Quantity, total_qty, units_sold

### Average Order Value
```
Average Order Value = DIVIDE([Revenue], [Orders])
```
- Description: Average revenue per order, calculated as Revenue divided by Orders. Useful for spotting pricing or volume shifts that revenue alone can hide.
- Format string: `$#,##0.00`
- Display folder: Orders and Volume
- Synonyms: avg order value, average per order, aov, order size, Average Order Value, avg_order_value, avg_sale

---

## Time Intelligence

### Revenue YTD
```
Revenue YTD = CALCULATE([Revenue], DATESYTD('Date'[Date]))
```
- Description: Year to date revenue from January 1 to the current date in filter context. Resets each calendar year.
- Format string: `$#,##0.00`
- Display folder: Time Intelligence
- Synonyms: ytd revenue, revenue year to date, year to date sales, ytd sales, Revenue YTD, revenue_ytd, ytd

### Gross Margin % MoM Change
```
Gross Margin % MoM Change =
VAR ThisMonth = [Gross Margin %]
VAR PriorMonth = CALCULATE([Gross Margin %], DATEADD('Date'[Date], -1, MONTH))
RETURN IF(ISBLANK(PriorMonth), BLANK(), ThisMonth - PriorMonth)
```
- Description: Month over month change in gross margin percentage, in percentage points. Near zero every normal month. A large negative value flags a sudden margin break, which is the signature of the seeded anomaly in September 2025.
- Format string: `0.0%`
- Display folder: Time Intelligence
- Synonyms: margin change, month over month change, margin trend, margin shift, MoM change, mom_change, margin_change

---

## Notes

- `Gross Margin %` is the star measure. Revenue looks normal in September and order volume looks normal in store 500, but Gross Margin % is the only measure that exposes the seeded cost anomaly cleanly.
- `DIVIDE` is used instead of the `/` operator so a zero or blank denominator returns blank rather than an error.
- `DATESYTD` and `DATEADD` require the `Date` table to be marked as the model date table.
- Use `SUMX` row by row rather than precomputing a line revenue column, to keep the model lean. If you prefer calculated columns for performance on very large models, that is a fine alternative.

---

## Application Checklist

1. Create the `_Measures` table with one hidden dummy column.
2. Add each measure above, with its format string, display folder, and description.
3. Add synonyms for each measure.
4. Confirm DAX table and column names match your loaded model.
5. Commit to Git in TMDL.
6. Test on the service machine: "Show gross margin percent by month for 2025" and "Show gross margin percent by store for September 2025".

---

## Key Business Rules and Model Notes

Paste this block into the AI instructions field in Prep data for AI. It tells Copilot how to reason about the model so answers are reliable and explainable by hand.

**What this model covers.** This is a sales model for Contoso. Sales is the fact table at one row per order line. Product, Customer, Store, and Date are dimension tables. Analyze sales by filtering and grouping through these dimensions. Prefer the predefined measures over raw columns, and prefer descriptive fields over key or ID columns when grouping or filtering.

**Revenue, cost, and margin.** Revenue is Quantity times Net Price, summed across order lines. Net Price is the actual selling price per unit after discount, so use Net Price for revenue, not List Price. List Price is the catalog price before any discount. Cost is Quantity times Unit Cost, where Unit Cost is the actual cost recorded on the sale. Gross Margin is Revenue minus Cost. Gross Margin % is Gross Margin divided by Revenue. Use Gross Margin % for any question about profitability or margin.

**Reference values versus actuals.** The Product table contains Product Standard Cost and Product List Price. These are reference catalog values and are hidden. Do not use them for actual revenue, cost, or margin. Use Net Price and Unit Cost from Sales for anything based on what actually sold.

**Orders, lines, and quantity.** Orders counts distinct orders, and one order can contain several lines. Lines counts order lines. Total Quantity sums units sold. Use Orders for order frequency, Total Quantity for unit volume, and Lines for the most granular transactional count.

**Dates.** Use the Date table for any time based analysis. Order Date is the primary date for sales and trends. Delivery Date is available for delivery timing questions. Months and days of week are already sorted in calendar order.

**Stores and channel.** Refer to stores by Store Name. Store 999999 is the online channel and accounts for roughly half of revenue. Treat it as a normal store unless the question is specifically about online versus physical stores.

**Currency.** Report figures in the single reporting currency. The currency exchange table is a hidden support table. Do not use it for analysis unless a question is explicitly about currency conversion.

**Data quality expectation for margin.** Gross Margin % normally runs in the mid fifties as a percentage of revenue. A sudden drop below about 50 percent, or a negative gross margin, usually points to a cost data quality issue rather than a pricing or sales decision. When margin looks unusual, compare it across stores, products, and months to find where the change comes from, and check cost before concluding it is a real business change.
