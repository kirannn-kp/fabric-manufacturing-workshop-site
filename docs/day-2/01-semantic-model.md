# Improve the semantic model

**Goal:** Review relationships and make fields and measures easier to use. **Time:** 70 minutes plus a 15-minute review. **Start:** `Manufacturing Operations - <TeamName>` in your team workspace. Do not edit the baseline.

1. In **Workspaces**, open your team workspace. On the semantic model row, select **...** > **Open semantic model**. In the top-right dropdown, switch from **Viewing** to **Editing**. Edits save automatically.
2. Read the diagram. `plants[PlantId]` is the one side and `production_risk`, `purchase_orders`, `machine_events`, and `quality_inspections` are the many sides. `suppliers[SupplierId]` (one) filters `purchase_orders` (many). Check these in **Manage relationships**. Filters should flow from lookup to fact only. Supplier Context is a single display value in the risk snapshot, not a relationship.
3. In the **Data** pane, expand `quality_inspections`. Select `DefectCategory`; in **Properties**, change **Name** to `Defect category`, and set **Description** to `Type of defect recorded for this inspection`. Do not rename fields used by the report recipe.
4. On `purchase_orders[SupplierId]`, use **...** > **Hide in report view** (or **Is hidden** in **Properties**). Do the same for technical `PlantId` fields on facts if you have time. Leave the production order ID visible; supervisors need it for follow-up. Hiding fields does not change permissions or relationships.
5. In **Data**, expand each measure's home table and find the required measure. If it is already present, review it and add a plain-language description; don't create another measure with that name. If missing, select its home table, select **New measure**, paste one formula at a time from below, and commit using the checkmark. In **Properties**, set supplier and quality rates to percentage, one decimal; format counts/quantity as whole numbers. Keep the report's filter context in mind.

## Measure recipes

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

[Next: Business definitions](02-business-definitions.md)
