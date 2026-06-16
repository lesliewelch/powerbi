# AI Data Schema (Copilot Field Visibility)

This defines which fields and measures Copilot should consider. Keys, sort columns, and low-value technical fields are hidden. Everything else is exposed.

Apply in Power BI Desktop via Prep data for AI, AI data schema tab, or in the Power BI service on the semantic model page. Display names are shown with the raw column in parentheses.

---

## EXPOSED to Copilot

### Sales
- Order Date (OrderDate)
- Delivery Date (DeliveryDate)
- Quantity (Quantity)
- List Price (UnitPrice)
- Net Price (NetPrice)
- Unit Cost (UnitCost)
- Currency (CurrencyCode)
- Exchange Rate (ExchangeRate)

### Product
- Product Code (ProductCode)
- Product Name (ProductName)
- Manufacturer (Manufacturer)
- Brand (Brand)
- Color (Color)
- Weight (Weight)
- Weight Unit (WeightUnit)
- Category (CategoryName)
- Subcategory (SubCategoryName)

### Store
- Store Code (StoreCode)
- Store Name (Description)
- Store Country (CountryName)
- Store State (State)
- Store Open Date (OpenDate)
- Store Close Date (CloseDate)
- Store Size (sq m) (SquareMeters)
- Store Status (Status)

### Date
- Date (Date)
- Year (Year)
- Quarter (Quarter)
- Year-Month (YearMonth)
- Month (Month)
- Day of Week (DayofWeek)
- Working Day (WorkingDay)

### Customer
- Customer Continent (Continent)
- Customer Country (CountryFull)
- Customer State (StateFull)
- Customer City (City)
- Customer Zip (ZipCode)
- Customer Gender (Gender)
- Customer Age (Age)
- Customer Birthday (Birthday)
- Customer Occupation (Occupation)
- Customer Company (Company)
- First Name (GivenName)
- Last Name (Surname)

### _Measures
- Revenue
- Cost
- Gross Margin
- Gross Margin %
- Orders
- Lines
- Total Quantity
- Average Order Value
- Revenue YTD
- Gross Margin % MoM Change

---

## HIDDEN from Copilot

### Sales
- OrderKey (key)
- LineNumber (sort)
- CustomerKey (key)
- StoreKey (key)
- ProductKey (key)

### Product
- ProductKey (key)
- CategoryKey (key)
- SubCategoryKey (key)
- Product Standard Cost (Cost) — reference catalog value, use Sales Unit Cost instead
- Product List Price (Price) — reference catalog value, use Sales Net Price instead

### Store
- StoreKey (key)
- GeoAreaKey (key)
- CountryCode (key)

### Date
- DateKey (key)
- YearQuarter (covered by Quarter)
- YearQuarterNumber (sort)
- YearMonthShort (covered by Year-Month)
- YearMonthNumber (sort)
- MonthShort (covered by Month)
- MonthNumber (sort)
- DayofWeekShort (covered by Day of Week)
- DayofWeekNumber (sort)
- WorkingDayNumber (sort)

### Customer
- CustomerKey (key)
- GeoAreaKey (key)
- StartDT, EndDT (technical)
- Title, MiddleInitial (low value)
- StreetAddress (low value for this demo)
- Vehicle (low value)
- Latitude, Longitude (low value)
- Country, State (code versions, the full-name versions are exposed)

### currencyexchange
- Entire table hidden (support table for currency conversion, not needed for the demo)

---

## Application Steps

Power BI Desktop:
1. Open the PBIP model.
2. Home ribbon, Prep data for AI, AI data schema tab.
3. Check the EXPOSED fields, uncheck the HIDDEN ones.
4. Apply, then commit to Git.

Power BI service (paid Fabric workspace, not trial):
1. Open the semantic model.
2. Semantic model page ribbon, Prep data for AI, AI data schema tab.
3. Check and uncheck as above, Apply.
4. Allow a few minutes for Copilot to reindex.

---

## Quick Test After Applying

- "Show gross margin percent by month for 2025." Should use Date and Gross Margin %, not any key column.
- "Show revenue by store for September 2025." Should use Store Name and Revenue, not StoreKey.

If Copilot pulls a key or a hidden field, the schema has not reindexed yet (wait a few minutes) or that field was left checked.
