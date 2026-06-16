# AI Instructions (Prep data for AI)

Paste the block below into Prep data for AI, AI instructions tab. It tells Copilot how to reason about the model so answers are reliable and explainable by hand.

---

This is a sales model for Contoso. Sales is the fact table at one row per order line. Product, Customer, Store, and Date are dimension tables. Analyze sales by filtering and grouping through these dimensions. Prefer the predefined measures over raw columns, and prefer descriptive fields over key or ID columns when grouping or filtering.

Revenue is Quantity times Net Price, summed across order lines. Net Price is the actual selling price per unit after discount, so use Net Price for revenue, not List Price. List Price is the catalog price before any discount. Cost is Quantity times Unit Cost, where Unit Cost is the actual cost recorded on the sale. Gross Margin is Revenue minus Cost. Gross Margin % is Gross Margin divided by Revenue. Use Gross Margin % for any question about profitability or margin.

The Product table contains Product Standard Cost and Product List Price. These are reference catalog values and are hidden. Do not use them for actual revenue, cost, or margin. Use Net Price and Unit Cost from Sales for anything based on what actually sold.

Orders counts distinct orders, and one order can contain several lines. Lines counts order lines. Total Quantity sums units sold. Use Orders for order frequency, Total Quantity for unit volume, and Lines for the most granular transactional count.

Use the Date table for any time based analysis. Order Date is the primary date for sales and trends. Delivery Date is available for delivery timing questions. Months and days of week are already sorted in calendar order.

Refer to stores by Store Name. Store 999999 is the online channel and accounts for roughly half of revenue. Treat it as a normal store unless the question is specifically about online versus physical stores.

Report figures in the single reporting currency. The currency exchange table is a hidden support table. Do not use it for analysis unless a question is explicitly about currency conversion.

Gross Margin % normally runs in the mid fifties as a percentage of revenue. A sudden drop below about 50 percent, or a negative gross margin, usually points to a cost data quality issue rather than a pricing or sales decision. When margin looks unusual, compare it across stores, products, and months to find where the change comes from, and check cost before concluding it is a real business change.
