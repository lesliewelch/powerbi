# Project Context: Contoso Copilot Demo Semantic Model

Single source of truth for building this Power BI semantic model in TMDL. Read this file and the spec files in `docs/` before making changes.

> Naming note: Claude Code automatically loads a file named `CLAUDE.md` at the repo root as project memory. Keep this file named `CLAUDE.md` so it is picked up without being asked. A copy named `DEMO_CONTEXT.md` is fine too, but `CLAUDE.md` is the one that auto-loads.

---

## 1. What we are building

A Power BI semantic model (PBIP, TMDL, Git source control) for a client demo. The client is migrating from a search-driven natural-language BI tool to Power BI and Fabric. The demo must reassure a specific audience that they keep, and improve on, two things they value:

1. Answering data questions fast and confidently in meetings, so they retain their reputation as the operational data experts.
2. Proactively monitoring data quality and catching anomalies same day, which is a point of professional pride for them.

The audience is mid-level, self-taught manufacturing analysts, not a central analytics team. The domain in the demo is sales (Contoso), not manufacturing. The demo shows that the behaviors transfer, not that the data matches their world.

Everything we build should make Copilot reliable and the model transparent enough that a user can explain any answer by hand. No black boxes.

---

## 2. Where we are now (you are here)

- The PBIP skeleton already exists, with Power Query pulling from a folder of Contoso CSVs.
- The `sales` table now points to `sales_demo.csv` (the version with the seeded anomaly). Do not repoint it to the original `sales.csv`.
- Nothing else has been done yet: no measures, no descriptions, no synonyms, no relationships beyond whatever auto-detected, no AI schema or instructions.

The job is to take this data-only skeleton to a clean, Copilot-ready model.

---

## 3. Environment and workflow

- Authoring happens in VS Code with Claude Code, editing TMDL directly. Avoid manual UI steps where TMDL can do the job.
- Two-machine flow: author and commit on the cloud-account laptop, then pull on a second laptop that has Power BI service access to publish and test.
- Copilot testing runs in the Power BI service on a paid F2 Fabric capacity (Copilot does not run on trial capacity).
- Commit in logical chunks with clear messages so the second machine pulls a clean state.

---

## 4. Data model

Star schema. `sales` is the fact. Grains and relationships below.

**Tables and grain**
- `sales` (or `Sales`): one row per order line. Fact table, points to `sales_demo.csv`.
- `product`: one row per product.
- `customer`: one row per customer.
- `store`: one row per store. Store 999999 is the online channel, roughly half of revenue.
- `date`: one row per day. Mark as the model date table on `date[Date]`.
- `currencyexchange`: currency support table. Keep but hide from Copilot for this demo.

**Relationships** (all single direction, dimension one-side to fact many-side)
- `product[ProductKey]` 1 to many `sales[ProductKey]`
- `customer[CustomerKey]` 1 to many `sales[CustomerKey]`
- `store[StoreKey]` 1 to many `sales[StoreKey]`
- `date[Date]` 1 to many `sales[OrderDate]`  (active)
- `date[Date]` 1 to many `sales[DeliveryDate]`  (inactive; optional, for delivery analysis via USERELATIONSHIP)
- `currencyexchange`: leave unrelated for the demo.

Mark `date` as the date table using the `date[Date]` column.

---

## 5. Build tasks, in order

Two phases. Phase A is pure TMDL (do this with Claude Code). Phase B is the Prep data for AI features that are authored in the Desktop UI today, then serialized into the model and committed.

### Phase A: TMDL (Claude Code)
1. Confirm relationships above exist and are correct; add the inactive DeliveryDate relationship.
2. Mark `date` as the date table.
3. Create the physical measures table. It is a single-column, single-row "enter data" stub. Hide the dummy column, leave the table visible.
4. Add all measures from `docs/measure_catalog.md`, with DAX, format strings, display folders, and descriptions.
5. Apply display names and descriptions to all user-facing fields from `docs/model_names_descriptions_synonyms.md`.
6. Hide all keys and sort-order columns listed as hidden in `docs/ai_data_schema.md`.
7. Add synonyms. These live in the culture or linguistic metadata in TMDL. If editing the linguistic schema by hand is messy, flag it and we will do synonyms in the service UI during testing instead. Do not block Phase A on synonyms.
8. Validate the model opens in Desktop after each major step, and commit.

### Phase B: Prep data for AI (Desktop UI, then commit serialized output)
9. AI data schema: expose and hide fields exactly as in `docs/ai_data_schema.md`.
10. AI instructions: paste the business rules block from `docs/measure_catalog.md` (the "Key Business Rules and Model Notes" section).
11. Verified answers: defer to the testing pass, once we know which prompts land. Do not build these yet.

Note on Phase B: authoring of the AI data schema and AI instructions is done in the Prep data for AI UI in Power BI Desktop. After applying, commit whatever the model serializes so it is versioned. Do not hand-author these blocks blind in TMDL unless we confirm the serialization format first.

---

## 6. Spec files in this repo

Place these in a `docs/` folder and treat them as the detailed specs:
- `docs/model_names_descriptions_synonyms.md` — display names, descriptions, and synonyms (natural and database-style) for every field.
- `docs/measure_catalog.md` — all DAX measures with descriptions, formats, folders, plus the business-rules block for AI instructions.
- `docs/ai_data_schema.md` — the expose and hide list for Copilot.

Internal, do not commit to any shared or client-facing location:
- `DEMO_anomaly_answer_key.md` — the ground truth for the seeded anomaly. Keep private.

---

## 7. Conventions

- No em dashes anywhere in descriptions or any drafted text. Use commas or periods. This is a hard rule for all client-facing content.
- Measures table is a physical single-row stub; hide the dummy column only, keep the table visible so measures surface in the field pane and Copilot.
- Format strings: `$#,##0.00` for currency, `0.0%` for percentages, `#,##0` for counts.
- Use display folders to group measures (Revenue and Margin, Orders and Volume, Time Intelligence).
- Keep all keys and technical sort columns hidden.
- Descriptions should read as a plain business tooltip and also disambiguate similar fields (for example, Net Price versus List Price, Unit Cost versus Product Standard Cost).

---

## 8. Guardrails

- Do not repoint or rewrite the Power Query source. `sales` must stay on `sales_demo.csv`. The other tables stay on their original CSVs.
- Do not "fix" the data. There is a deliberately seeded anomaly in `sales_demo.csv`. It is intentional and must remain. See the internal note below.
- Build `Gross Margin %` exactly as specified so negative margins surface cleanly. This measure is what makes the demo work.
- Validate the model still opens in Desktop after structural changes, before committing.
- If anything in a spec file conflicts with what is actually in the model, stop and flag it rather than guessing.

---

## 9. Internal: the seeded anomaly (do not expose to the audience)

For context only, so you understand why Gross Margin % matters and do not treat the anomaly as a bug to fix:

- Store 500 (Contoso Store Kansas, United States), September 2025, has `UnitCost` inflated 10x on 53 sales lines, simulating a cost feed loaded one decimal place off.
- Revenue and volume for that store are untouched and look normal. Only margin breaks.
- Company September gross margin drops from about 56% to about 51.5%. The store itself goes from about +52% to about -384%.
- It surfaces only on a margin measure, never on revenue or volume. That is the point: it rewards the data-quality vigilance the audience prides itself on.

This anomaly is the demo's discovery moment. Keep it intact and never reference it in any audience-facing artifact.

---

## 10. Done criteria

The model is ready for testing when:
- All relationships exist and `date` is the marked date table.
- All measures from the catalog exist, formatted, foldered, and described.
- All user-facing fields have display names and descriptions; all keys and sort columns are hidden.
- The model opens cleanly in Desktop and refreshes from the CSV folder.
- These prompts behave sensibly in the Copilot pane on the F2:
  - "Show gross margin percent by month for 2025." (steady near 56%, September dips)
  - "Show gross margin percent by store for September 2025." (store 500 stands out)
  - "Show revenue by store for September 2025." (looks normal, no red flag, by design)
  - "For store 500 in September 2025, show gross margin percent by product category." (all categories negative)

When those land consistently, we move to verified answers and the monitoring report.
