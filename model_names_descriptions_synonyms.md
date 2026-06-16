# Semantic Model: Names, Descriptions, and Synonyms (Copilot-ready)

How to use this: set the display name and description on each field in TMDL or the Model view, and enter the synonyms in the synonyms pane (or Prep data for AI). Descriptions are written to read well as a business tooltip and to give Copilot the context it needs to pick the right field. Synonyms include natural phrasing plus database-style variants (raw column name, snake_case, camelCase, abbreviations) so users who know the underlying fields still get matched. Keys and technical sort columns are marked hidden.

---

## sales  (grain: one row per order line)

| Raw column | Display name | Description | Synonyms (incl. database-style) |
|---|---|---|---|
| OrderKey | Order ID | Identifier for the order. One order can contain multiple lines. Hidden from view; used to count distinct orders. | order id, order number, order, OrderKey, order_key, order_id, order_no |
| LineNumber | (hidden) | Technical line sequence within an order. | LineNumber, line_number, line_no |
| OrderDate | Order Date | Date the order was placed. This is the primary date for sales and trend analysis. | order date, purchase date, sale date, transaction date, date ordered, when ordered, OrderDate, order_date, order_dt |
| DeliveryDate | Delivery Date | Date the order was delivered to the customer. | delivery date, ship date, delivered date, date delivered, DeliveryDate, delivery_date, ship_dt |
| CustomerKey | (hidden) | Technical key linking to the customer. | CustomerKey, customer_key, cust_id |
| StoreKey | (hidden) | Technical key linking to the store. Store 999999 is the online channel. | StoreKey, store_key, store_id |
| ProductKey | (hidden) | Technical key linking to the product. | ProductKey, product_key, prod_id |
| Quantity | Quantity | Number of units sold on the line. | quantity, units, units sold, number sold, how many, qty, Quantity, qty_sold, units_qty |
| UnitPrice | List Price | Catalog list price per unit before any discount. For the actual price paid, use Net Price. | list price, catalog price, sticker price, gross price, price before discount, UnitPrice, unit_price, list_price |
| NetPrice | Net Price | Actual selling price per unit after discount. Line revenue equals Quantity times Net Price. | net price, selling price, sale price, price after discount, net unit price, revenue per unit, NetPrice, net_price, net_amt, sell_price |
| UnitCost | Unit Cost | Cost to the company per unit. Line cost equals Quantity times Unit Cost, and gross margin is revenue minus cost. | unit cost, cost, cost per unit, cost of goods, cogs per unit, landed cost, UnitCost, unit_cost, cost_per_unit, cogs |
| CurrencyCode | Currency | Currency the order was transacted in. | currency, currency code, transaction currency, iso currency, CurrencyCode, currency_code, ccy, curr |
| ExchangeRate | Exchange Rate | Rate applied to convert the order currency to the reporting currency. | exchange rate, fx rate, conversion rate, currency rate, ExchangeRate, exchange_rate, fx_rate, rate |

---

## product  (grain: one row per product)

| Raw column | Display name | Description | Synonyms (incl. database-style) |
|---|---|---|---|
| ProductKey | (hidden) | Technical product key. | ProductKey, product_key, prod_id |
| ProductCode | Product Code | Business code or SKU that identifies the product. | product code, sku, item code, article number, part number, ProductCode, product_code, sku_id, item_no |
| ProductName | Product Name | Descriptive name of the product. | product, product name, item, item name, model, ProductName, product_name, prod_name, item_name |
| Manufacturer | Manufacturer | Company that makes the product. | manufacturer, maker, vendor, oem, Manufacturer, mfr, manufacturer_name |
| Brand | Brand | Brand the product is sold under. | brand, label, make, Brand, brand_name |
| Color | Color | Product color. | color, colour, finish, Color, color_name |
| Weight | Weight | Product weight, in the unit shown by Weight Unit. | weight, mass, item weight, Weight, weight_value |
| WeightUnit | Weight Unit | Unit of measure for product weight, such as ounces or pounds. | weight unit, unit of weight, uom, WeightUnit, weight_unit, weight_uom |
| Cost | Product Standard Cost | Reference standard unit cost from the product catalog. Actual cost on a sale comes from Unit Cost in Sales. | standard cost, catalog cost, reference cost, product cost, Cost, std_cost, product_cost |
| Price | Product List Price | Reference catalog list price from the product record. Actual selling price comes from Net Price in Sales. | catalog price, reference price, msrp, product price, Price, list_price, product_price, msrp |
| CategoryKey | (hidden) | Technical category key. | CategoryKey, category_key, cat_id |
| CategoryName | Category | High-level product category. | category, product category, department, CategoryName, category_name, prod_category, cat |
| SubCategoryKey | (hidden) | Technical subcategory key. | SubCategoryKey, subcategory_key, subcat_id |
| SubCategoryName | Subcategory | More specific product grouping within a category. | subcategory, sub category, product subcategory, SubCategoryName, subcategory_name, subcat |

---

## store  (grain: one row per store)

| Raw column | Display name | Description | Synonyms (incl. database-style) |
|---|---|---|---|
| StoreKey | (hidden) | Technical store key. Store 999999 is the online channel. | StoreKey, store_key, store_id |
| StoreCode | Store Code | Business code for the store. | store code, location code, shop code, StoreCode, store_code |
| Description | Store Name | Display name of the store, such as Contoso Store Kansas. | store, store name, location, branch, shop, outlet, Description, store_name, store_desc |
| GeoAreaKey | (hidden) | Technical geography key. | GeoAreaKey, geo_area_key, geo_id |
| CountryCode | (hidden) | Technical country code for the store. | CountryCode, country_code |
| CountryName | Store Country | Country where the store operates. | store country, country, location country, CountryName, country_name, store_country |
| State | Store State | State or region where the store operates. | store state, state, region, province, State, store_state, state_name |
| OpenDate | Store Open Date | Date the store opened. | open date, opening date, store open date, OpenDate, open_date, opened_dt |
| CloseDate | Store Close Date | Date the store closed. Blank if the store is still open. | close date, closing date, store close date, CloseDate, close_date, closed_dt |
| Description (sq) | Store Size (sq m) | Selling floor area of the store in square meters. | store size, square meters, floor area, area, size, sqm, SquareMeters, square_meters, store_area |
| Status | Store Status | Operational status of the store, such as Open, Closed, or Restructured. | store status, status, Status, store_status |

Note: the size column above is the raw `SquareMeters` field. Listed with a clear label so it is not confused with the store name.

---

## date  (grain: one row per day; mark as the model date table)

| Raw column | Display name | Description | Synonyms (incl. database-style) |
|---|---|---|---|
| Date | Date | Calendar date. | date, day, calendar date, Date, calendar_date |
| Year | Year | Calendar year. | year, yr, Year, cal_year |
| Quarter | Quarter | Calendar quarter, such as Q3. | quarter, qtr, q, Quarter, cal_quarter |
| YearMonth | Year-Month | Year and month label, such as 2025-Sep. | year month, month year, period, YearMonth, year_month, yyyymm |
| Month | Month | Month name. | month, month name, Month, month_name |
| MonthNumber | (hidden, sort) | Month number used to sort Month. | MonthNumber, month_number, month_no |
| DayofWeek | Day of Week | Day name, such as Monday. | day of week, weekday, day name, DayofWeek, day_of_week, weekday_name |
| WorkingDay | Working Day | Indicates whether the date is a working day. | working day, business day, workday, WorkingDay, working_day, is_workday |

Note: the remaining date columns (DateKey, YearQuarter, YearQuarterNumber, YearMonthShort, YearMonthNumber, MonthShort, DayofWeekShort, DayofWeekNumber, WorkingDayNumber) are technical or sort-order columns. Keep them but hide them from Copilot.

---

## customer  (grain: one row per customer; useful subset)

| Raw column | Display name | Description | Synonyms (incl. database-style) |
|---|---|---|---|
| CustomerKey | (hidden) | Technical customer key. | CustomerKey, customer_key, cust_id |
| Continent | Customer Continent | Continent where the customer is located. | continent, customer continent, region, Continent, continent_name |
| CountryFull | Customer Country | Country where the customer is located. | customer country, country, CountryFull, Country, country_full, customer_country |
| StateFull | Customer State | State or region where the customer is located. | customer state, state, region, StateFull, State, state_full, customer_state |
| City | Customer City | City where the customer is located. | city, customer city, town, City, city_name, customer_city |
| ZipCode | Customer Zip | Postal code for the customer. | zip, zip code, postal code, postcode, ZipCode, zip_code, postal |
| Gender | Customer Gender | Customer gender. | gender, sex, Gender, customer_gender |
| Age | Customer Age | Customer age in years. | age, customer age, Age, customer_age |
| Birthday | Customer Birthday | Customer date of birth. | birthday, birth date, dob, date of birth, Birthday, birth_date |
| Occupation | Customer Occupation | Customer occupation or job. | occupation, job, profession, role, Occupation, occupation_name, job_title |
| Company | Customer Company | Customer employer or company. | company, employer, organization, Company, company_name, employer |
| GivenName | First Name | Customer first name. | first name, given name, forename, GivenName, given_name, first_name, fname |
| Surname | Last Name | Customer last name. | last name, surname, family name, Surname, last_name, lname |

Note: lower-priority customer columns (Title, MiddleInitial, StreetAddress, Country code, State code, StartDT, EndDT, Company Vehicle, Latitude, Longitude) can stay in the model but hidden from Copilot unless a use case needs them.

---

## currencyexchange  (support table)

| Raw column | Display name | Description | Synonyms (incl. database-style) |
|---|---|---|---|
| Date | Rate Date | Date the exchange rate applies to. | rate date, fx date, Date, rate_date |
| FromCurrency | From Currency | Source currency being converted from. | from currency, source currency, base currency, FromCurrency, from_currency, ccy_from |
| ToCurrency | To Currency | Target currency being converted to. | to currency, target currency, ToCurrency, to_currency, ccy_to |
| Exchange | Exchange Rate | Conversion rate from the source to the target currency. | exchange rate, fx rate, conversion rate, Exchange, exchange_rate, fx_rate |

Note: this is a support table for currency conversion. Hide it from Copilot unless you plan to demo multi-currency analysis.

---

## Quick application checklist

1. Apply display names and descriptions in TMDL or the Model view, commit to Git.
2. Hide all keys and sort-order columns from Copilot via the AI data schema.
3. Enter synonyms in the synonyms pane or Prep data for AI.
4. Confirm `date` is marked as the model date table.
5. Re-publish to the F2 workspace and test a few prompts in the Copilot pane.
