---
published: true
type: workshop
title: Manufacturing Operations with Microsoft Fabric
short_title: Fabric Manufacturing Workshop
description: A beginner-friendly, click-by-click workshop that uses Microsoft Fabric to identify manufacturing production risks.
level: beginner
authors:
  - Kiran Panchal
contacts:
  - "@kirannn-kp"
duration_minutes: 720
tags: microsoft-fabric, manufacturing, onelake, lakehouse, real-time-intelligence, power-bi, data-agent
---

# Start here

The labs run in a Microsoft Fabric workspace prepared by your workshop organizer.

## Prerequisites

### Microsoft Fabric

You need access to the workshop Fabric workspace.
You must be able to create and edit items.
Before the workshop, check that you can:

- Open the workshop workspace.
- Select New item.
- Create or edit a Lakehouse.
- Create or edit an Eventhouse, if enabled.
- Open a semantic model.
- Create or edit a Fabric Data Agent, if enabled.

### Laptop and browser

Bring your laptop and charger.
Use Microsoft Edge or Google Chrome.
Check that you can sign in to:

- Microsoft Fabric
- Microsoft Teams
- GitHub

### Workshop data

You do not need to prepare data.
Microsoft will provide the datasets and starter material.

### GitHub

GitHub is used to open or download the workshop instructions.
GitHub Copilot is not required.

### Microsoft Foundry

Microsoft Foundry access is not required for the two Fabric workshop days.
Foundry will be used only for an optional facilitator demonstration.

### Team access

At least one person in each group must have working edit access to the Fabric workspace.
We recommend edit access for every hands-on participant.

### Access problem

If access does not work, share:

- Product name
- Screenshot
- Complete error message
- Time of the error

Share these details privately and redact sensitive information. See [support](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/SUPPORT.md).

## Morning production risk review

You work with three fictional factories: Plant North, Plant South, and Plant East. The shift lead has time to call only a few teams before the next shift. Your task is to identify open production orders with material shortages and recommend what someone should verify next.

Use inventory as of **10 November 2026** and planned starts from **11 to 17 November 2026**, inclusive. Net usable stock is available stock minus reservations, floored at zero. An order is at risk if it needs more than that amount. This is a fixed training example, not a forecast or a shared-stock allocation model.

Look at supplier deliveries, machine faults, and quality separately. Those signals can guide a follow-up call. They do not prove why a particular order has a shortage. All data is invented and machine events are a stored batch, not a live feed.

At the end, give your shift lead one priority, one piece of evidence, and one limitation. Start with [Day 1](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-1/README.md) or inspect the [dataset definitions](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/data/validation-summary.md).

## Synthetic data and validation

All rows are invented. The plants, cities, suppliers, machine IDs, products, and transactions are fictional training examples. Download the eight source files without opening and resaving them in a spreadsheet; that can alter dates or IDs.

| Download | Rows, not including header | Grain or key |
|---|---:|---|
| [plants.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/plants.csv) | 3 | One row per `PlantId`. |
| [materials.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/materials.csv) | 15 | One row per `MaterialId`. |
| [suppliers.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/suppliers.csv) | 6 | One row per `SupplierId`. |
| [purchase_orders.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/purchase_orders.csv) | 84 | One row per `PurchaseOrderId`; references supplier, material, and plant. |
| [production_orders.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/production_orders.csv) | 80 | One row per `ProductionOrderId`; references plant and material. |
| [inventory.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/inventory.csv) | 180 | One row per `InventoryDate` + `PlantId` + `MaterialId`. |
| [quality_inspections.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/quality_inspections.csv) | 105 | One row per `InspectionId`; references plant. |
| [machine_events.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/machine_events.csv) | 500 | One event per `EventTime` + `PlantId` + `MachineId`; references plant. |

The separate facilitator-only production risk snapshot contains **45 rows**, one per eligible production order. It is not a ninth source for the participant upload exercise.

### Rules

- Open production orders starting **11–17 November 2026** use the latest inventory snapshot at or before **10 November 2026** for their plant and material. `AvailableQuantity` is gross of reservations. Usable stock is the greater of zero and available minus reserved. Shortage is the greater of zero and required minus usable. Missing inventory means unknown, not zero. Each eligible order in this sample has its own plant/material pair; this is not an allocation model. Safety stock is not deducted.
- Supplier context is the supplier on the most recently placed purchase order for the same plant and material on or before the snapshot date. Break order-date ties by highest `PurchaseOrderId`. This includes both open and delivered purchase orders and says nothing about assignment or cause. No match means `Unknown`.
- Supplier lateness uses actual delivery dates **1–10 November**, only for `Delivered` orders with actual dates. Late means actual date is later than promised. Two or more such orders make a supplier repeatedly late.
- Machine events use UTC, **1 November inclusive to 11 November exclusive**. Three or more `Fault` events per plant/machine means repeated faults; `Severity = High` is a separate filter.
- Quality covers **1–10 November**. Rejection rate is total rejected divided by total inspected. A rate of at least 10% is high. All sample materials use `Each`.

### Deliberate findings and example records

| Finding | Evidence |
|---|---|
| 12 orders have a shortage, 4 per plant, totaling 420 units. | PR001 needs 100 units. P01/M01 has 75 gross and 10 reserved, so 65 usable and 35 short. |
| Supplier A and B have repeated late deliveries. | PO001 was promised 1 November and delivered 3 November. A: 8/12 late; B: 6/12. Each other supplier: 0/12. The 12 open purchase orders are not part of the denominator. |
| Two machines repeatedly fault, and high severity is visible. | P01 / MC-N01 and P02 / MC-S01 have 8 faults each. Six High events are all on MC-N01, including the first at `2026-11-01T00:00:00Z`. |
| Drive Unit A has a high rejection rate. | QI001 records 15 rejects out of 100. Drive Unit A totals 525 of 3,500 (15.0%). Pump Unit B is 3.0%; Valve Unit C is 1.0%. |

CSV headers, row counts, keys and joins, and the findings above were checked locally. Lakehouse type inference and live query/report results still need the rehearsal in the organizer's Fabric workspace.

---

# Day 1: Load and explore

## Day 1: Load and explore

| Time | Activity |
|---|---|
| 09:30–09:50 | Welcome. Read the [scenario](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/01-business-scenario.md). Ask what evidence is needed to flag a production order. |
| 09:50–10:20 | Fabric and Ignite updates. Use the [official Fabric blog](https://blog.fabric.microsoft.com/) to pick at most two updates relevant to this tenant. Say whether each is available here, in preview, or plan-only. If none changes the labs, show the current UI and move on. |
| 10:20–10:50 | Explain the item map below. Identify the two optional branches. |
| 10:50–11:05 | Break. |
| 11:05–12:10 | [Lakehouse](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-1/01-onelake-lakehouse.md) hands-on. |
| 12:10–12:30 | Verify counts and discuss what stays in OneLake. |
| 12:30–13:15 | Lunch. |
| 13:15–14:20 | [Eventhouse or report fallback](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-1/02-eventhouse.md) hands-on. |
| 14:20–14:35 | Compare the fault and high-severity findings. |
| 14:35–14:50 | Break. |
| 14:50–15:50 | [Data Agent or manual answer checking](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-1/03-data-agent.md). |
| 15:50–16:00 | Checkpoint: show one Lakehouse table, name one repeated-fault machine, and explain one change that improved a question or answer. |

**Item map:** A workspace holds the items. OneLake stores the files and tables. The Lakehouse makes those files and tables easy to inspect. Eventhouse holds a separate copy of the machine events in a KQL database to practice event analysis. The semantic model names, relates, and measures the data. A Power BI report shows the results. A Fabric Data Agent can answer questions using the approved model. You use the prepared model and report today, then improve them tomorrow.

[Workshop home](https://microsoft.github.io/moaw/workshop/?src=gh:kirannn-kp/fabric-manufacturing-workshop-site/main/) · [Day 2](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/README.md)

## OneLake and Lakehouse

**Goal:** Upload eight source files, turn them into tables, and inspect them. **Time:** 65 minutes plus a separate 20-minute verification. **Start:** Team workspace, edit access, and the eight files from the [data inventory](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/data/validation-summary.md).

1. Sign in at [Microsoft Fabric](https://app.fabric.microsoft.com/). In the left menu, select **Workspaces**, then your workshop workspace.
2. Select **New item**. Search for **Lakehouse** and select it. Enter `ManufacturingOperationsLH_<TeamName>`, replacing `<TeamName>` with your assigned name without angle brackets. Leave the schema option at the facilitator's preset/default. Select **Create**.
3. In the Lakehouse **Explorer**, hover over **Files** and select **...** > **Upload** > **Upload files**. Browse to the eight CSVs and upload them. Do not upload the validation Markdown or facilitator CSV in this exercise.
4. Open **Files**. Select `inventory.csv`. Look for `PlantId`, `MaterialId`, `AvailableQuantity`, and `ReservedQuantity`. **Check:** All eight CSVs appear under Files. This is the file area of your Lakehouse in OneLake.
5. On each CSV row, select **...** > **Load to tables** > **New table**. Use the filename without `.csv` for the table name. If asked, use the default schema, CSV format, comma separator, and **Column header** on. Select **Load**. Repeat once per file; do not append on a retry.
6. Expand **Tables** (and `dbo` if shown). Open `plants`, then `inventory`. Check that IDs and columns have the right names. Compare the three visible plants with the [expected counts](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/data/validation-summary.md).
7. For an exact count beyond the table preview, open **SQL analytics endpoint** from the top-right Lakehouse selector. Refresh Explorer if needed. If your endpoint offers **New visual query**, add the `inventory` table, select **Transform** > **Count rows**, and run the visual query. Repeat for `production_orders` or another assigned table. If unavailable, open the CSV in the browser or a plain-text viewer that shows line numbers and subtract the one header line; compare with the facilitator's verified table count. A source line count alone does not verify a loaded table.
8. Pair with a teammate: point to a raw file and its queryable Delta table, name one difference, and report any count discrepancy.

**Check:** `plants` has 3 rows, `inventory` has 180, and `production_orders` has 80. The model later uses prepared tables, not your personal practice copy.

**If you don't see it:** Missing **Lakehouse** or **Upload** usually means the wrong workspace or insufficient edit access. Ask the facilitator to check capacity and role. If creation or load is blocked, inspect the backup Lakehouse with a teammate. If columns are named `_c0` or counts include a header, ask before deleting anything; reload as a new team table with **Column header** on.

Screenshot placeholder: `01-lakehouse-files.png`.

[Next: Eventhouse](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-1/02-eventhouse.md)

## Eventhouse: find repeated machine faults

**Goal:** Find machines with repeated faults and inspect severe events. **Time:** 65 minutes plus a 15-minute comparison. **Start:** Team workspace with an enabled Eventhouse and `machine_events.csv` from the [data inventory](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/data/validation-summary.md). If blocked, use the fallback below.

1. In the workspace, select **New item** > **Eventhouse**. Name it `ManufacturingEvents_<TeamName>` and select **Create**, or open the one assigned to your team.
2. Open the KQL database created with the Eventhouse. If it is not shown and you have permission, create one from the Eventhouse using **+ KQL database**. In the database, select **Get data** > **Local file**.
3. Choose **+ New table**, name it `machine_events`, and browse to `machine_events.csv`. In the preview, choose CSV format, use/skip the header as a data row, and check the seven named columns. Ensure `EventTime` is `datetime`, the next four fields are strings, and the last two are numeric. Select **Finish** to ingest and wait for confirmation. Do not reingest a successful file.
4. From the database select **Query with code** or open the associated **KQL Queryset**. Check the database in the connection selector. Paste one block at a time into a query tab and select **Run**.

```kusto
machine_events
| count
```

Line 1 chooses the loaded table. Line 2 counts its rows. **Check:** 500.

```kusto
machine_events
| where EventTime >= datetime(2026-11-01) and EventTime < datetime(2026-11-11)
| where EventType == "Fault"
| summarize FaultCount = count() by PlantId, MachineId
| where FaultCount >= 3
| order by FaultCount desc, MachineId asc
```

Line 1 chooses the table. Line 2 keeps 1–10 November UTC. Line 3 keeps faults only. Line 4 counts faults for each plant and machine. Line 5 defines repeated as at least three. Line 6 sorts by count, then machine. **Check:** `P01 / MC-N01` and `P02 / MC-S01`, 8 each.

```kusto
machine_events
| where EventTime >= datetime(2026-11-01) and EventTime < datetime(2026-11-11)
| where Severity == "High"
| project EventTime, PlantId, MachineId, EventType, TemperatureC, DurationMinutes
| order by EventTime asc
```

Line 1 chooses the table. Line 2 keeps the same UTC window. Line 3 keeps High severity. Line 4 shows the evidence fields. Line 5 sorts from earliest to latest. **Check:** 6 events, all on `MC-N01`; first at `2026-11-01T00:00:00Z`.

5. If dashboards are enabled, select **Save to dashboard** from the repeated-fault query results. Choose an existing team Real-Time Dashboard or create one if offered. Give the tile a name and confirm. If that entry point is missing, open the team Real-Time Dashboard in editing mode, select **Add tile**, choose the KQL database, paste the repeated-fault query, and apply it as a table. **Check:** The tile shows both machines. Otherwise leave the results in the query tab.

### If Eventhouse is not enabled

1. Open the facilitator's `Manufacturing Operations - Reference` report > **Machine events**. It uses `machine_events.csv` through the prepared Lakehouse and model.
2. Set the date window to 1–10 November 2026 UTC (inclusive), or leave the all-data default if it already covers that range. Clear any plant or severity selection. Read the **Repeated faults** table, which shows only `Fault` counts of 3 or more per plant/machine.
3. Read the **High severity events** table. It filters **Severity = High** independently of the fault count. Check that you get the same two machines and six events as above.
4. Explain what you would ask maintenance to check first. These are batch records, not a live feed.

**If you don't see it:** Check database permissions and the header mapping; open the facilitator's prepared Eventhouse if only ingestion is blocked. If the event data or query still does not work, take the report path rather than retrying. If report access fails too, pair with a teammate or ask the facilitator to share the private view.

Screenshot placeholder: `02-eventhouse-query.png`.

[Next: Fabric Data Agent](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-1/03-data-agent.md)

## Fabric Data Agent: check the answer

**Goal:** Test answers against a prepared report and improve one answer. **Time:** 60 minutes. **Start:** Team workspace, approved baseline semantic model and reference report, and the [five questions](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/business-questions.md). Do not use any other source.

1. In your workspace, select **New item** > **Fabric data agent**. Name it `ManufacturingRiskAgent_<TeamName>`.
2. In the OneLake catalog, select `Manufacturing Operations - Baseline` and then **Add**. In **Explorer**, select only `production_risk`, `plants`, `purchase_orders`, `suppliers`, `machine_events`, and `quality_inspections`. If model-level selection is controlled by its AI data schema in your tenant, use the facilitator's preset equivalent instead. No external data or extra sources are needed.
3. Select **Data agent instructions** (look under **Setup** if needed). Paste the text below. Save if prompted.

   > Answer only from the workshop's synthetic model. State the filter and measure used. For production risk, use the 10 November 2026 snapshot and open orders starting 11–17 November. At risk means a positive material shortage. The late percentage uses only delivered purchase orders. Machine faults count only Fault events and event time is UTC. Quality rejection rate is rejected units divided by inspected units. Supplier context is not cause. If evidence is missing, say so.

4. Append an **Example questions** heading and copy all five numbered questions from the [question sheet](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/business-questions.md) into the agent instructions as plain text. Power BI semantic models do not support the agent's **Example queries** question/query-pair feature. Use the five items as test prompts, not verified answers. Do not change the shared model.
5. In the test chat, ask all five questions, one at a time. Compare with the report. Record the value, filters, and evidence. Expand the execution steps if shown. Do not publish the agent.
6. Try an ambiguous prompt: “Which supplier is late?” Find a weak or underspecified answer. Amend the team agent instruction to say: “For supplier comparisons, report the late count and delivered count and exclude open orders. Use 1–10 November 2026 by actual delivery date unless asked otherwise.” If the team has the editable model and time, add the same definition as a measure description instead.
7. Clear the chat so the first reply does not influence the second. Retest the troublesome question and the precise version from the sheet. Label the answer correct, incomplete, or wrong. An answer can be correct even if a different turn was weak; do not invent a failure.

**Check:** You can point to the source visual for each answer and explain whether the change improved the answer.

### If Fabric Data Agent is not enabled

1. Open `Manufacturing Operations - Reference`. Use the [same five questions](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/business-questions.md).
2. For each, record the filters, evidence visual, and answer. For “Which supplier is late?”, first ask a teammate to answer without a shared definition.
3. Add a plain-language definition to your editable team model, or write it on the worksheet if you cannot edit. Clarify the date range and denominator. Ask the revised question and check the new answer against the same report. Compare what changed.

**If you don't see it:** Check the capacity and tenant setting with the facilitator. A model missing from the catalog may need Read permission or region alignment. If the agent times out or cannot answer, switch to manual checking. A plausible reply is not proof.

Screenshot placeholder: `03-agent-test.png`.

[Day 2](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/README.md)

## Business question sheet

Use the prepared report or your verified team report. Clear unrelated filters before each question. Record the first and revised answer, the measure or definition, the filter, and the evidence. All dates are in 2026.

| # | Sample question | Report evidence |
|---|---|---|
| 1 | How many open production orders planned for 11–17 November are at risk based on the 10 November snapshot? | **Risk review** > at-risk card. |
| 2 | For PR001, how many units are short and what reason is recorded? | **Risk review** > production order table; filter `PR001`. |
| 3 | Of orders delivered 1–10 November, which supplier has the highest late-delivery percentage? Include late and delivered counts. | **Risk review** > supplier bar and tooltips. |
| 4 | From 1–10 November UTC, which machines had at least three faults and how many did each have? | **Machine events** > repeated faults table. |
| 5 | From 1–10 November, which product had the highest rejection rate? What was that rate? | **Quality** > product rejection table. |

| Question # | Filters and measure | Evidence | First answer | Revised answer | Correct / incomplete / wrong |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

**Before:** “Which supplier is late?” What is unclear?

**After:** Use the date range, the definition of “late,” and the delivered-only denominator. What changed in your answer, and why?

---

# Day 2: Model and report

## Day 2: Model and report

| Time | Activity |
|---|---|
| 09:30–09:50 | Review Day 1. Show one order, one machine finding, and one clearer question. |
| 09:50–11:00 | [Semantic model](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/01-semantic-model.md) hands-on. |
| 11:00–11:15 | Review the relationship diagram and measures together. |
| 11:15–11:30 | Break. |
| 11:30–12:40 | [Business definitions and question quality](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/02-business-definitions.md). |
| 12:40–13:25 | Lunch. |
| 13:25–14:35 | [Build the report](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/03-power-bi-report.md) in the browser. |
| 14:35–14:45 | Share one visual and check its result. |
| 14:45–15:00 | Break. |
| 15:00–15:40 | [Team challenge](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/04-team-challenge.md). |
| 15:40–15:50 | Team use-case checkpoint. Capture one priority use case, current PoC status, main blocker, and support needed. Keep this in the organizer's private notes, not the repository. |
| 15:50–16:00 | Checkpoint: confirm the at-risk number, one reliable relationship, one report result, and one next action. |

[Workshop home](https://microsoft.github.io/moaw/workshop/?src=gh:kirannn-kp/fabric-manufacturing-workshop-site/main/) · [Day 3](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-3/README.md)

## Improve the semantic model

**Goal:** Review relationships and make fields and measures easier to use. **Time:** 70 minutes plus a 15-minute review. **Start:** `Manufacturing Operations - <TeamName>` in your team workspace. Do not edit the baseline.

1. In **Workspaces**, open your team workspace. On the semantic model row, select **...** > **Open semantic model**. In the top-right dropdown, switch from **Viewing** to **Editing**. Edits save automatically.
2. Read the diagram. `plants[PlantId]` is the one side and `production_risk`, `purchase_orders`, `machine_events`, and `quality_inspections` are the many sides. `suppliers[SupplierId]` (one) filters `purchase_orders` (many). Check these in **Manage relationships**. Filters should flow from lookup to fact only. Supplier Context is a single display value in the risk snapshot, not a relationship.
3. In the **Data** pane, expand `quality_inspections`. Select `DefectCategory`; in **Properties**, change **Name** to `Defect category`, and set **Description** to `Type of defect recorded for this inspection`. Do not rename fields used by the report recipe.
4. On `purchase_orders[SupplierId]`, use **...** > **Hide in report view** (or **Is hidden** in **Properties**). Do the same for technical `PlantId` fields on facts if you have time. Leave the production order ID visible; supervisors need it for follow-up. Hiding fields does not change permissions or relationships.
5. In **Data**, expand each measure's home table and find the required measure. If it is already present, review it and add a plain-language description; don't create another measure with that name. If missing, select its home table, select **New measure**, paste one formula at a time from below, and commit using the checkmark. In **Properties**, set supplier and quality rates to percentage, one decimal; format counts/quantity as whole numbers. Keep the report's filter context in mind.

### Measure recipes

**Home table: `production_risk`.** `CALCULATE` applies the positive-shortage filter before `DISTINCTCOUNT` counts orders. `COALESCE` displays zero if there are no matching orders. Each source row represents one eligible order; all quantities are in `Each`. `SUM` adds shortage quantities in the current selection. Do not sum mixed units if you replace this dataset.

```dax
At-risk production orders =
COALESCE(
    CALCULATE(
        DISTINCTCOUNT(production_risk[ProductionOrderId]),
        KEEPFILTERS(production_risk[ShortageQuantity] > 0)
    ),
    0
)
```

```dax
Material shortage quantity = SUM(production_risk[ShortageQuantity])
```

**Home table: `purchase_orders`.** The delivered count excludes open orders and blank actual dates. `FILTER` keeps only delivered orders whose actual date is after the promise for the numerator. The row set respects report filters, such as supplier and delivery date. `DIVIDE` returns blank when the denominator is zero. Add the first two helper measures before the percentage.

```dax
Delivered purchase orders =
COUNTROWS(
    FILTER(
        purchase_orders,
        purchase_orders[DeliveryStatus] = "Delivered"
            && NOT(ISBLANK(purchase_orders[ActualDeliveryDate]))
    )
)
```

```dax
Late delivered purchase orders =
COALESCE(
    COUNTROWS(
        FILTER(
            purchase_orders,
            purchase_orders[DeliveryStatus] = "Delivered"
                && NOT(ISBLANK(purchase_orders[ActualDeliveryDate]))
                && purchase_orders[ActualDeliveryDate] > purchase_orders[PromisedDate]
        )
    ),
    0
)
```

```dax
Supplier late-delivery percentage =
DIVIDE([Late delivered purchase orders], [Delivered purchase orders])
```

**Home table: `machine_events`.** `COUNTROWS` counts events, not machines. `KEEPFILTERS` narrows the current result to faults without discarding other filters.

```dax
Machine fault count =
CALCULATE(
    COUNTROWS(machine_events),
    KEEPFILTERS(machine_events[EventType] = "Fault")
)
```

**Home table: `quality_inspections`.** Sum the rejected and inspected counts separately, then divide. This is a weighted rate, not an average of row percentages.

```dax
Quality rejection rate =
DIVIDE(
    SUM(quality_inspections[RejectedQuantity]),
    SUM(quality_inspections[InspectedQuantity])
)
```

**Check:** The five measures appear in their home tables (plus two supplier helpers), and `Defect category` has its new description. Your team model is not the shared baseline.

**If you don't see it:** Missing **Editing** or **New measure** may be a tenant, permission, or model limitation. Do the review in Viewing mode with the facilitator and record proposed changes. Use the completed model for the report. If the editor marks a formula as invalid, compare table/column names, measure dependencies, and your tenant's DAX separators.

Screenshot placeholder: `04-model-relationships.png`.

[Next: Business definitions](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/02-business-definitions.md)

## Define the question before answering it

**Goal:** Make five business terms precise. **Time:** 70 minutes. **Start:** Team model or worksheet, reference report, and the [business questions](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/business-questions.md).

| Term | Plain-language definition |
|---|---|
| At risk | An open order starting 11–17 November with a positive shortage in the 10 November snapshot. Missing stock is unknown, not zero. |
| Shortage quantity | How many units the order still needs after removing reserved stock. All sample materials use `Each`. |
| Supplier late-delivery percentage | Late delivered orders divided by all delivered orders in the chosen actual-delivery period. Late means actual date is later than promised. |
| Machine fault count | Count of events marked `Fault` in the selected UTC interval. Repeated means three or more for the same plant and machine. |
| Quality rejection rate | Total units rejected divided by total units inspected in the selection. |

1. In the reference report, open **Risk review**. Read the supplier bar without changing filters. Ask two people what “late” means and compare their answers.
2. In your team semantic model, enter **Editing**. Select the supplier percentage measure. In **Properties** > **Description**, paste the supplier definition above. Repeat for the risk measure. If permitted, use **Prep data for AI** to expose only the needed fields and measures. If you cannot edit, write the same definitions in the worksheet.
3. Rewrite “Which supplier is late?” using the actual-delivery time window and counts from question 3. Use the reference bar and tooltips or a working agent to test it. If using the agent, clear chat before retesting.
4. Pick another question. Record the date, denominator or counting unit, and evidence. Ask a teammate to answer using only your written definition and the report.
5. Compare the answers and name one limitation. Example: the supplier displayed beside an order is context, not proof of cause.

**Check:** Two people using your definition and filters can reproduce the same answer and name its evidence.

**If you don't see it:** Report calculations do not require **Prep data for AI**. Keep the model description or worksheet route and compare answers manually.

[Next: Power BI report](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/03-power-bi-report.md)

## Build the morning risk report

**Goal:** Create five visuals in the browser. **Time:** 70 minutes plus a 10-minute share-back. **Start:** Editable team model with the measures from the [model lab](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/01-semantic-model.md).

1. From the semantic model, select **New report** in the ribbon. If you are in the workspace list, open the model first. When the report canvas opens, rename `Page 1` to `Risk review`.
2. Click blank canvas, then select the visual type in **Visualizations** (or use **Build a visual** if on-object editing is enabled). Drag the named fields or measures from **Data** into the wells below. Repeat with blank canvas for each visual. For **visual-level** filters, select the visual and drop the field in **Filters on this visual**.
3. Set each title in **Format visual** > **General** > **Title** where available (for a card, the measure label may already display the title). When choosing dates below, use the field itself, not a date hierarchy. For the supplier bar, add both helper measures under **Tooltips**.

| Visual | Fields and wells | Filters and expected output | Title |
|---|---|---|---|
| Card | **Values/Data:** `At-risk production orders` | No extra filter: the snapshot already keeps open orders starting 11–17 Nov. Expect 12. | At-risk production orders |
| Clustered bar chart | **Y-axis:** `plants[PlantName]`; **X-axis:** `At-risk production orders` | No extra filter. Expect 4 per plant. | At-risk orders by plant |
| Clustered bar chart | **Y-axis:** `suppliers[SupplierName]`; **X-axis:** `Supplier late-delivery percentage`; **Tooltips:** `Late delivered purchase orders`, `Delivered purchase orders` | `purchase_orders[ActualDeliveryDate]` on/after 1 Nov and before 11 Nov 2026. Blank dates are excluded. Supplier A leads at 66.7% (8/12); B is 50.0% (6/12). | Late deliveries by supplier |
| Line chart | **X-axis:** `machine_events[EventTime]` (raw field); **Y-axis:** `Machine fault count` | `EventType = Fault`, UTC 1 Nov inclusive to 11 Nov exclusive. The timeline plots 16 fault timestamps, not daily totals; all 16 timestamps have count 1. A flat trace at 1 is expected for event-level data. | Machine faults over time (UTC) |
| Table | **Columns:** `production_risk[ProductionOrderId]`, `[PlantName]`, `[MaterialName]`, `[ShortageQuantity]`, `[SupplierContext]`, `[RiskReason]`; use **Don't summarize** for the quantity. | `ShortageQuantity` greater than 0. Expect 12 rows, including PR001 with 35 units short. | Orders to review |

4. Clear any selection from a bar or table. Read the five visuals, then select Plant North and observe that the risk card filters to 4. Click the selection again or click blank canvas to clear it. A plant selection can also filter the supplier deliveries by plant; clear it before the overall supplier question.
5. Select **File** > **Save** (or **Save** on the toolbar). Use `Manufacturing Risk - <TeamName>` and your team workspace. Reopen it from the workspace to verify it was saved.

**Check:** Without cross-filters, the card shows 12, the table is one row per order, and Supplier A's tooltip shows 8 late of 12 delivered.

**If you don't see it:** If **New report** or **Save** is blocked, ask the facilitator to check model Build permission and workspace roles. Continue by matching the five visuals in the reference report. If totals differ, clear cross-filters, check date and numeric types, and check that you have the team model, not the raw Lakehouse. If the line is blank, check the UTC filters and use raw `EventTime`; a timestamp table is a backup if datetime remains text.

Screenshot placeholder: `05-risk-report.png`.

[Next: Team challenge](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/04-team-challenge.md)

## Team challenge

**Goal:** Make the existing report more useful for one decision. **Time:** 40 minutes. **Start:** Your team report or the facilitator's private working copy.

| Choose one | Suggested small change |
|---|---|
| Production-risk assistant | Add a plant slicer, then show how the order list changes. Use the question sheet if the agent is unavailable. |
| Supplier-performance view | Sort the supplier visual by late percentage and keep both counts in the tooltip. Explain why open orders are absent. |
| Machine-fault monitoring | Add a table of faults by plant and machine. Use the same UTC window and limit to at least three. |
| Quality issue analysis | Add a product table with rejection rate and total inspected units. Compare the sample's three products. |

1. In your team's existing report, select **Edit**. Add or improve one visual. Do not change the baseline report or create a new solution.
2. Add two business questions to your worksheet. Include filters and a visual that supports each answer.
3. Name one limitation, such as a fixed snapshot, no shared-stock allocation, or no causal evidence.
4. Save the report. Prepare a three-minute demo: 30 seconds for the decision, 90 seconds to show the visual and two questions, and one minute for the limitation and what you would check next.

**Check:** A teammate can open the report and reproduce a finding. If many teams are present, run demos in parallel pairs.

**If you don't see it:** If nobody on your team can edit, mark up a visual from the reference report using the worksheet. Present the proposed change and verify the answers from the original report.

[Day 2 checkpoint](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-2/README.md) · [Day 3](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-3/README.md)

---

# Day 3: Decide what to pilot

## Day 3: Choose a proof of concept

No participant Foundry account is needed. The facilitator can omit the demo without affecting the planning.

| Time | Activity |
|---|---|
| 09:30–09:45 | Recap Day 2 and the submitted team use cases. |
| 09:45–10:15 | [Optional Foundry demonstration](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-3/01-optional-foundry-demo.md) or Fabric-only walkthrough. |
| 10:15–11:15 | Team conversations: decision, users, business value, current status, and blockers. |
| 11:15–11:30 | Break. |
| 11:30–12:30 | [PoC planning](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-3/02-poc-planning.md): scope, data, architecture, access, and success criteria. |
| 12:30–13:00 | Break. |
| 13:00–13:45 | Review dependencies and risks; agree on owners and next actions. |
| 13:45–14:00 | Read back decisions and confirm the next private checkpoint. |

[Workshop home](https://microsoft.github.io/moaw/workshop/?src=gh:kirannn-kp/fabric-manufacturing-workshop-site/main/)

## Optional facilitator demonstration

**Goal:** Show what an additional conversational entry point can do, without making it part of the Fabric solution. **Time:** 30 minutes. **Start:** A pre-approved, rehearsed demo using the synthetic Fabric Data Agent only. Nothing new is provisioned during the workshop.

1. Before the event, follow the [official portal setup instructions](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/fabric) in the approved environment if the organizer has chosen this branch. Confirm the same tenant, region alignment for Fabric sources, eligible capacity, signed-in user permissions, and source access. Hide tenant URLs and IDs when sharing your screen.
2. Show the Fabric report first. Ask “How many production orders are at risk?” and point to the 12-order card and the fixed snapshot.
3. In the prepared Foundry demo, ask question 1 from the [question sheet](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/business-questions.md). Compare the reply with the same report. Follow with “Why did Supplier A cause the shortage?” A good answer says the available data does not establish cause.
4. Ask the room whether an additional interface would help the intended users. Discuss identity, access, evidence and evaluation before considering a PoC.

**Check:** Participants can distinguish the Fabric data source and the optional conversational entry point, and know where to verify the result.

**If you don't see it:** Skip Foundry rather than changing tenant settings or provisioning items during the session. Show the same questions in the Fabric Data Agent if ready, or compare the worksheet to the report. Continue with the same PoC discussion.

[Next: PoC planning](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-3/02-poc-planning.md)

## Proof-of-concept planning

**Goal:** Leave with one scoped experiment and named owners. **Time:** 60 minutes to draft, then a 45-minute separate action review. **Start:** The Day 2 use-case checkpoint and a private team document. Do not copy real data or internal identifiers into the workshop repository.

1. Describe one user, one recurring decision, and what they do today.
2. Choose one plant or process, one time window, and the minimum data needed. Write one item that is out of scope.
3. Sketch the smallest Fabric-first flow: approved source → Lakehouse → semantic model → report. Add Eventhouse only if event latency matters. Add a Data Agent or Foundry only if the team has a specific question-led need and the access path is approved.
4. Choose how to verify the outcome. Example: two shift leads can reproduce the answer on ten known scenarios using the same definitions and filters.
5. Assign owners and book the next checkpoint. If a dependency is not approved, mark it blocked rather than assuming it will be available.

| Field | Team notes |
|---|---|
| Priority use case, user, and decision | |
| Current PoC status | |
| Main blocker and support needed | |
| In scope / out of scope | |
| Architecture and why each item is needed | |
| Required data, grain, freshness, and data owner | |
| Security, residency, access, retention, review owner | |
| Dependencies, capacity, and time needed | |
| Evidence and success criteria | |
| Owner, date, and next action | |

**Check:** The business owner can repeat the decision and success criteria in one sentence, and every next action has an owner and date.

**If information is missing:** Record the assumption, who will resolve it, and by when. Use synthetic data for early discussion until real data is approved.

---

# Help and references

## Workshop support

Check [troubleshooting](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/troubleshooting.md) first. During the workshop, send the facilitator a private Teams message with the lab step, product name, time and time zone, exact error text after redaction, and a redacted screenshot if helpful. Tell them if another teammate can continue.

The organizer supplies the workshop Teams channel, the facilitator contact, and the tenant access contact in the private event invitation. Do not post error evidence or tenant details publicly.

## Troubleshooting

Work in your assigned workshop workspace, not **My workspace**. Do not change tenant settings to make a lab work.

| Symptom | Check or recovery |
|---|---|
| Can't sign in or workspace not listed | Confirm the invited account and tenant. Ask the facilitator to check group membership, guest access, and assigned workspace. Pair with a teammate while waiting. |
| **New item**, **Lakehouse**, or **Upload** is missing | Check capacity state and workspace edit access. Use the facilitator Lakehouse for inspection if creation or upload stays blocked. |
| Lakehouse columns are wrong or the file appears twice | Check **Column header** and comma separator. Do not append a second load. Ask the facilitator to repair a team-owned table or use the prepared copy. |
| Tables are absent in the SQL analytics endpoint | Wait for sync and refresh Explorer. Check that you opened the right endpoint and that Tables, not just Files, exist. |
| Eventhouse or Queryset unavailable | Check that the KQL database is selected and your account can query it. If blocked, use the [machine-event report path](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-1/02-eventhouse.md#if-eventhouse-is-not-enabled). |
| KQL count is not 500 | Verify the typed columns and header. Look for an accidental second ingest or a failed ingestion. Switch to the backup before deleting anything. |
| Cannot edit model or save a report | Check the team model, Editing mode, workspace role, tenant setting, and model Build permissions. Use the read-only reference and record the authoring gap. |
| Risk card or supplier bar differs | Clear cross-highlights, check visual filters, use the correct snapshot, and compare the delivered-only counts. Do not sum shortage over mixed units in another dataset. |
| Data Agent source or item missing | Check capacity, region, tenant flags, and model Read access. Do not change tenant settings. Use the [manual fallback](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/docs/day-1/03-data-agent.md#if-fabric-data-agent-is-not-enabled). |
| Agent answer sounds right but cannot be verified | Compare filters and measures to the report, record the mismatch, and ask a more precise question in a clean chat. |
| Foundry demonstration fails | Use the Fabric-only version. No participant setup changes are needed. |

Follow [support](https://github.com/kirannn-kp/fabric-manufacturing-workshop-site/blob/main/SUPPORT.md) when escalating. Include the step, product, complete redacted error, time and time zone, and a redacted screenshot in a private message.

## References

These are authoring and facilitator references, not participant setup dependencies. The cited Microsoft Learn flows were consulted on 19 November 2026; the workshop tenant has not been rehearsed.

| Source | Used for |
|---|---|
| [Microsoft MOAW template](https://github.com/microsoft/moaw/blob/main/template/workshop/workshop.md) | Private workshop front matter, comma-separated tags, and Markdown navigation. |
| [Microsoft Fabric Data Agent Workshop](https://github.com/microsoft/fabric-data-agent-workshop) | Adapt the baseline/weak-answer/metadata/retest pattern; replace notebooks and automated evaluation with browser activities and a manual answer sheet. |
| [Microsoft Skills for Fabric](https://github.com/microsoft/skills-for-fabric) | Builder reference only; no participant setup. |
| [Load CSV files to Lakehouse tables](https://learn.microsoft.com/en-us/fabric/data-engineering/load-to-tables) | File-to-Delta loading. |
| [Ingest a local file into a KQL database](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/get-data-local-file) | Local CSV ingestion and schema preview. |
| [Create a Real-Time Dashboard](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/dashboard-real-time-create) | Optional tile from a KQL queryset. |
| [Visual query editor](https://learn.microsoft.com/en-us/fabric/data-warehouse/visual-query-editor) | Optional no-code row checks; supported operators vary. |
| [Lakehouse model and report tutorial](https://learn.microsoft.com/en-us/fabric/data-engineering/tutorial-lakehouse-build-report) | Create a Direct Lake model and start a web report. |
| [Edit a model in the Power BI service](https://learn.microsoft.com/en-us/power-bi/transform-model/service-edit-data-models) | Editing mode, relationships, and measures. |
| [Create a Fabric Data Agent](https://learn.microsoft.com/en-us/fabric/data-science/how-to-create-data-agent) | Capacity, source access, and browser authoring. |
| [Data Agent tenant settings](https://learn.microsoft.com/en-us/fabric/data-science/data-agent-tenant-settings) | Review tenant controls and regional restrictions. |
| [Use the Fabric Data Agent with Foundry agents](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/fabric) | Optional facilitator demonstration only. |
