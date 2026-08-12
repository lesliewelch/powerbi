# Model-Layer Plant Plan & Measure Spec

Turns the two-page wireframe into buildable objects, with the deliberate flaws specified. Reconciled against the **as-built PBIP** (verified from `orders.tmdl`, `relationships.tmdl`, `model.tmdl`). Two audiences:

- **For Claude Code:** implement these calculated columns and measures in the PBIP TMDL **exactly as written, flaws included.** Do not fix, optimize, refactor, add variables, add format strings, add display folders, or "improve" naming. Use the real column names below. The badness is the requirement.
- **For you (Leslie):** this is the plant plan. Section D maps every defect to how visible it is (code / runtime / source) and therefore which Copilot is positioned to catch it — the backbone for the tool-by-tool demo.

## What's already true in the built model (don't re-do these)

- **Date columns imported as `dateTime`** (`General Date`), not text — pandas parsed them on load. This *is* the antipattern: datetime with a time component is high-cardinality and expensive, and it triggers the next point.
- **Auto date/time is ON** — `__PBI_TimeIntelligenceEnabled = 1`, with **nine hidden `LocalDateTable_*` tables** (one per date column across orders/order_items/order_reviews) plus a template. Live bloat, very visible in metadata. Leave it on.
- **Relationships are already auto-detected into the messy state we want** (see A2). `relationships.tmdl` needs **no changes**.
- **Injections landed:** `order_status` is `int64` (summarizeBy: sum — a bonus key-summarizing antipattern), audit columns present (`created_at`/`updated_at`/`created_by` as `string`), sentinel in `order_delivered_customer_date`.
- Audit `created_at`/`updated_at` are the only **text** datetimes left (unused) — the last trace of the text-date mess, latent because nothing references them.

## Column name reference (as built, `datacopilotdemo.dbo`)

- **orders:** order_id, customer_id, order_status *(int64)*, order_purchase_timestamp *(dateTime)*, order_approved_at *(dateTime, null)*, order_delivered_carrier_date *(dateTime, null)*, order_delivered_customer_date *(dateTime, null, 9999 sentinel)*, order_estimated_delivery_date *(dateTime)*, created_at/updated_at/created_by *(string)*
- **order_items:** order_id, order_item_id, product_id, seller_id, shipping_limit_date *(dateTime)*, price *(dec)*, freight_value *(dec)*, +audit
- **order_reviews:** review_id, order_id, review_score *(int)*, review_comment_title, review_comment_message, review_creation_date, review_answer_timestamp
- **customers:** customer_id, customer_unique_id, customer_zip_code_prefix *(text)*, customer_city, customer_state
- **sellers:** seller_id, seller_zip_code_prefix, seller_city, seller_state
- **products:** product_id, product_category_name *(pt, null)*, product_name_lenght, product_description_lenght, product_photos_qty, product_weight_g, product_length_cm, product_height_cm, product_width_cm
- **category_translation:** product_category_name, product_category_name_english *(related to products, but report uses the pt column)*
- **geolocation:** ~1M rows, imported, **unrelated, unused** — the bloat lesson
- **status_code:** *(NOT imported — the discoverability beat)*

Visibility tags: **[CODE]** legible from TMDL/M by static analysis · **[RUNTIME]** valid code, wrong only when you see the number · **[SOURCE]** needs the database or domain knowledge, not in the model.

---

## A. Base model state

**A1 — Storage / date antipatterns (already present, no build).** Datetime columns with time component (expensive), auto date/time on with nine hidden tables, geolocation 1M rows unused, audit columns unused. All metadata-legible. **[CODE]** presence / **[RUNTIME]** size.

**A2 — Relationships (already auto-detected — leave `relationships.tmdl` alone).**
- Present and correct-for-mess: order_items→orders, order_payments→orders, order_reviews→orders, order_items→products, order_items→sellers.
- **customers→orders is `bothDirections`** — autodetect made it bidirectional (customer_id is 1:1-ish here because of the per-order grain). Authentic accident; keep it. **[CODE]**
- **products→category_translation IS related** — but visuals use `products[product_category_name]` (Portuguese), never `category_translation[product_category_name_english]`. The English name is one hop away and nobody switched to it. **[CODE]** (a whole-file tool sees the related table and the unused English column).
- **geolocation is not related to anything** — stays that way. 1M-row unused table = bloat. **[CODE]**
- Do **not** import/relate `status_code`.

**A3 — Calculated columns on the fact (wrong layer; columns are already dateTime, so no conversion needed).** These belong in Power Query/source; done in DAX because that's where the person was. **[CODE]** wrong layer; Delivery Days also **[RUNTIME]** via the sentinel. Guards keep nullable-date columns from producing garbage (a blank date reads as serial 0 → huge negatives); the guards do **not** touch the 9999 sentinel — authentic.
```dax
orders[Order Year]  = YEAR ( orders[order_purchase_timestamp] )
orders[Order Month] = FORMAT ( orders[order_purchase_timestamp], "mmm" )   -- text; sorts Apr, Aug, Dec… no sort-by column

orders[Delivery Days] =
IF (
    NOT ISBLANK ( orders[order_delivered_customer_date] ),
    DATEDIFF ( orders[order_purchase_timestamp], orders[order_delivered_customer_date], DAY )   -- sentinel rows → ~2.9M days
)

orders[Ship Days] =
IF (
    NOT ISBLANK ( orders[order_delivered_carrier_date] ),
    DATEDIFF ( orders[order_purchase_timestamp], orders[order_delivered_carrier_date], DAY )
)

orders[Approve Days] =
IF (
    NOT ISBLANK ( orders[order_approved_at] ),
    DATEDIFF ( orders[order_purchase_timestamp], orders[order_approved_at], DAY )
)

orders[Carrier Days] =
IF (
    NOT ISBLANK ( orders[order_delivered_carrier_date] ),
    DATEDIFF ( orders[order_approved_at], orders[order_delivered_carrier_date], DAY )
)

orders[Lateness Bucket] =                                       -- nested IF where SWITCH belongs
IF ( orders[Delivery Days] <= 5, "0-5 days",
    IF ( orders[Delivery Days] <= 10, "6-10 days",
        IF ( orders[Delivery Days] <= 20, "11-20 days", "20+ days" ) ) )
```
The trend axis uses `orders[Order Year]` + `orders[Order Month]` (so the alphabetical month sort shows), not the auto date/time hierarchy.

---

## B. Measures — Page 1 (Delivery Performance)

**Scatter measures across tables on purpose** (some on orders, some on order_items, review measure on order_reviews). No measures table.

**Total Orders** · orders · base for rates · **[RUNTIME]** (base includes canceled/undelivered)
```dax
Total Orders = COUNTROWS ( orders )
```

**Orders Delivered** · orders · KPI · hardcoded status code, no codebook · **[SOURCE]**
```dax
Orders Delivered = CALCULATE ( COUNTROWS ( orders ), orders[order_status] = 4 )   -- 4 = delivered, but nothing in the model says so
```

**Avg Delivery Days** · orders · KPI + trend + by-state + worst-sellers · **[RUNTIME]**
```dax
Avg Delivery Days = AVERAGE ( orders[Delivery Days] )   -- includes 9999 sentinel rows → detonates; no filter to delivered
```

**On Time Orders** · orders · FILTER-over-table inside CALCULATE · **[CODE]** + **[RUNTIME]**
```dax
On Time Orders =
CALCULATE (
    COUNTROWS ( orders ),
    FILTER (
        orders,
        NOT ISBLANK ( orders[order_delivered_customer_date] )
            && orders[order_delivered_customer_date] <= orders[order_estimated_delivery_date]
    )
)   -- sentinel rows never <= estimated → counted not-on-time
```

**On Time Rate** · orders · KPI + by-region bar · IF instead of DIVIDE, wrong base · **[CODE]** + **[RUNTIME]**
```dax
On Time Rate = IF ( [Total Orders] = 0, BLANK ( ), [On Time Orders] / [Total Orders] )   -- DIVIDE; base = all orders incl canceled
```
*PENDING (SLA file): when the SharePoint SLA table exists, add `On Time vs SLA` comparing delivery days to per-region promised days — carrying the region-name and monthly-grain mismatch. Wireframe marker 3.*

**Not Delivered or Late %** · orders · KPI · copy-paste of On Time Rate, same base flaw · **[CODE]** + **[RUNTIME]**
```dax
Not Delivered or Late % =
IF ( [Total Orders] = 0, BLANK ( ), ( [Total Orders] - [On Time Orders] ) / [Total Orders] )
```

*By-state map (marker 4) uses `customers[customer_state]` directly with **Avg Delivery Days** — the sentinel does the damage. (Geolocation is not involved; it's the unused-bloat lesson, separate.)*

---

## B. Measures — Page 2 (Fulfillment & Satisfaction)

**Avg Review Score** · order_reviews · KPI + vs-lateness · logic fine, **no format string** · **[CODE]** (hygiene)
```dax
Avg Review Score = AVERAGE ( order_reviews[review_score] )
```

**Late Order %** · orders · KPI · copy-paste family — note it uses DIVIDE while On Time Rate uses IF (the drift copy-paste leaves) · **[CODE]**
```dax
Late Order % =
IF ( [Total Orders] = 0, BLANK ( ), DIVIDE ( [Total Orders] - [On Time Orders], [Total Orders] ) )
```

**Total Freight** · order_items · seller matrix · item grain · **[RUNTIME]**
```dax
Total Freight = SUM ( order_items[freight_value] )
```

**Avg Freight per Order** · order_items · KPI · **grain trap** · **[RUNTIME]**
```dax
Avg Freight per Order = AVERAGE ( order_items[freight_value] )   -- averages per ITEM; labeled per order; multi-item orders skew it
```

**Avg Days To Ship** · orders · KPI · reuse of Ship Days · **[CODE/RUNTIME]**
```dax
Avg Days To Ship = AVERAGE ( orders[Ship Days] )
```

**Pipeline stage measures** · orders · funnel (marker 7) · copy-paste family · **[CODE]**
```dax
Avg Purchase to Approved = AVERAGE ( orders[Approve Days] )
Avg Approved to Carrier  = AVERAGE ( orders[Carrier Days] )
Avg Carrier to Delivered = AVERAGE ( orders[Delivery Days] ) - AVERAGE ( orders[Carrier Days] )   -- crude; also carries the sentinel
```

*Review-vs-lateness (marker 2) plots **Avg Review Score** by `orders[Lateness Bucket]` — sentinel rows all pile into "20+ days." Category bar (marker 5) uses **Total Orders**/**Total Freight** by `products[product_category_name]` — Portuguese, because the report never switched to the related English column. Seller matrix (marker 8) puts **Avg Delivery Days** (order grain), **Avg Review Score** (review grain), and **Total Freight** (item grain) in one visual — the grain tension, visible.*

---

## C. Optional extra plants (add if a segment needs more material)

- **DAX date table** (redundant on top of auto date/time; wrong layer; month alpha-sort). Adds the "calculated table should be Power Query" lesson. Fragile in raw TMDL — build via Desktop TMDL view if added. **[CODE]**
```dax
Date =
ADDCOLUMNS ( CALENDAR ( DATE(2016,1,1), DATE(2018,12,31) ),
    "Year", YEAR ( [Date] ), "Month", FORMAT ( [Date], "mmm" ), "Quarter", "Q" & FORMAT ( [Date], "Q" ) )
```
- **Hardcoded year YoY** · **[CODE]**: `Orders 2018 = CALCULATE ( [Total Orders], orders[Order Year] = 2018 )`
- **ALL vs REMOVEFILTERS** · **[CODE]**: `% of Total Orders = DIVIDE ( [Total Orders], CALCULATE ( [Total Orders], ALL ( orders ) ) )`

---

## D. Defect → visibility → which Copilot is positioned to help

The demo backbone. "Positioned to help" = whether the defect is even in that tool's field of view.

| Defect | Visibility | Copilot in Power BI (in-canvas, local) | GitHub Copilot (whole PBIP as text) | M365 Copilot (surfaced files, narrow window/no memory) |
|---|---|---|---|---|
| IF instead of DIVIDE | CODE | ✅ per-measure rewrite | ✅ across all | ◐ if the one measure is in the window |
| FILTER-in-CALCULATE → boolean | CODE | ✅ | ✅ | ◐ |
| Nested IF → SWITCH (bucket) | CODE | ✅ | ✅ | ◐ |
| Copy-paste rate family + IF/DIVIDE drift | CODE, **systematic** | ✗ sees one at a time | ✅ spots the pattern | ✗ can't hold all 3 |
| Missing format strings (model-wide) | CODE, **systematic** | ◐ one measure | ✅ all at once | ✗ fragments |
| Calc columns that should be measures/PQ | CODE | ◐ flags the one | ✅ the pattern | ◐ |
| Month sorts alphabetically | CODE | ✅ (add sort-by) | ✅ | ◐ |
| Key summarizing (order_status sum) | CODE | ✅ | ✅ | ◐ |
| Bidirectional customers→orders | CODE | ◐ | ✅ | ◐ |
| **Datetime type + auto date/time (9 hidden tables)** | **CODE** presence / **RUNTIME** size | ◐ can flag auto date/time | ✅ sees the annotation + all LocalDateTables | ◐ |
| **Geolocation: 1M rows, unrelated, unused** | **CODE** | ◐ | ✅ sees the orphan table | ◐ |
| **Sentinel wrecks the average** | **RUNTIME** | ✗ | ✗ (can't run it) | ✗ — **needs a human who sees the number** |
| **Undelivered in the denominator** | **RUNTIME** | ✗ | ✗ | ✗ |
| **Freight grain double/skew** | **RUNTIME** | ✗ | ✗ | ✗ |
| **`order_status = 4` meaning** | **SOURCE** | ✗ | ✗ (status_code not in model) | ✗ unless DB surfaced |
| **English category column unused (translation related)** | CODE→SOURCE | ◐ may suggest the field swap | ✅ sees related table + unused English col | ◐ |
| **status_code discoverability** | **SOURCE** | ✗ | ◐ only if DB schema is in context | ◐ |

Reading for the talk: **CODE** rows are where the Copilots earn their keep, and GitHub Copilot's whole-file view wins the *systematic* ones (copy-paste family, format strings everywhere, the orphan table, the hidden date tables) that the in-canvas tool literally can't all see. **RUNTIME** rows are the honesty floor — no tool catches them from static analysis; you catch them because a package doesn't take 2.9 million days. **SOURCE** rows are the discoverability edge — only a tool that can see the database even has a shot, and the best case is a *suggestion*. That's the shape of "even if M365 disappointed you, here's what the other two genuinely do, and what none of them replace."

---

## Build order (Claude Code)

1. **Calculated columns (A3)** into `tables/orders.tmdl`: Order Year, Order Month, Delivery Days, Ship Days, Approve Days, Carrier Days, Lateness Bucket. (Columns are already dateTime — no DATEVALUE.)
2. **Page 1 measures**, then **Page 2 measures** — scattered across orders / order_items / order_reviews. No format strings, no display folders.
3. Leave `relationships.tmdl`, `model.tmdl`, all `partition = m` blocks, and the entire Report folder untouched.
4. (Optional, only if asked) Section C extras.

Then you build the visuals by hand from the wireframe and confirm each marker shows a wrong-but-plausible number.
