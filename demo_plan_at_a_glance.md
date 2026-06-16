# Demo Plan: At a Glance

## What we are demoing

**Audience:** mid-level manufacturing self-taught analysts (the people to win), with a supportive stakeholder reviewing content first.

**The frame:** this uses sales data (Contoso), not manufacturing data. The behaviors transfer. State that line out loud so no one gets lost in the domain.

**The two behaviors they care about:**
1. In-meeting credibility: ask a question in plain language, get an answer fast, stay the expert.
2. Proactive data-quality monitoring: watch a board, catch an anomaly, trace it to the source, hand it off to fix same day.

**The four doors (capabilities shown):**
1. Copilot in the service and Desktop: natural-language answers in the moment.
2. Self-built monitoring reports in the service: their proactive watch, all in one tool.
3. Analyze in Excel: shown for completeness, as a way to hand light exploration to people who ask them for data.
4. Paginated reports: precise handoff of the exact problem rows to the data team.

**The seeded anomaly (the centerpiece):** Store 500 (Contoso Store Kansas), September 2025, Unit Cost loaded 10x (a decimal-shift cost feed error), 53 lines. Volume looks normal; only margin breaks. Company September gross margin dips 55.97% to 51.52%; store 500 goes +51.6% to -383.5%. Gross Margin % is the only measure that exposes it.

**The reveal path:** notice (margin % by month dips in September) to isolate (margin % by store, store 500 red) to confirm it is a feed issue not a product issue (all categories in store 500 negative) to bound it in time (starts September 1) to root cause (cost feed decimal error) to handoff (paginated list of the 53 lines).

**The judgment beat:** contrast April (low revenue, normal margin = seasonal, uniform across stores) with September (normal revenue, broken margin = data problem, one store). Shows the discipline they pride themselves on.

---

## What needs to be prepped

### Data
- [x] sales_demo.csv seeded and loaded as the Sales table (originals untouched)

### Semantic model (PBIP / TMDL via Claude Code)
- [x] Clean star schema: Orders and OrderRows removed, Sales fact plus Product, Customer, Store, Date
- [x] Date relationships: OrderDate active, DeliveryDate inactive; Date marked as the date table
- [ ] _Measures stub table plus the 10 measures (verify Gross Margin % reads ~56% unfiltered)
- [ ] Display names and descriptions on all user-facing fields
- [ ] Hide keys and sort columns
- [ ] Synonyms, including database-style variants (linguistic schema step)

### Prep data for AI (Desktop; needs Q&A enabled to activate tabs)
- [ ] AI data schema: keep and hide fields per ai_data_schema.md (Simplify data schema tab)
- [ ] AI instructions: paste the business-rules block
- [ ] Verified answers: pin the core prompts to visuals (after the report exists)

### Reports
- [ ] Monitoring report: Gross Margin % by month, by store, by category (surfaces the anomaly)
- [ ] Paginated report: the 53 affected lines for the handoff moment

### Capacity and publish
- [ ] F2 live, Copilot tenant setting on, US region, provisioned at least 24 hours ahead
- [ ] Do not pause the F2 before the demo; turn off scheduled refresh on the demo model
- [ ] Publish the model to the F2 workspace for live Copilot testing

### Testing (tomorrow)
- [ ] Test each Copilot prompt in the service; confirm the anomaly surfaces cleanly
- [ ] Iterate on descriptions, synonyms, and AI instructions where prompts miss
- [ ] Confirm each TMDL change loads in Desktop before committing

### Supporting material
- [x] 10-slide deck (self-service story, the four doors, the model as the foundation)
- [x] Citizen-developer learning path (closing slide)
- [x] Power BI Desktop unblocked (GAC Newtonsoft fix)

---

## The one critical dependency

Everything in the reveal rides on three things being true at once: the F2 has Copilot enabled, the model is published to it, and Gross Margin % is correct and well-described. If any one of those is off, the in-meeting and monitoring moments fall flat. Test that trio first tomorrow.

## Core Copilot prompts to verify
- "Show gross margin percent by month for 2025."
- "Show gross margin percent by store for September 2025."
- "Show gross margin percent by category for store 500 in September 2025."
- "What is the gross margin percent for 2025?"
