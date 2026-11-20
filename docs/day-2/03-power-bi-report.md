# Build the morning risk report

**Goal:** Create five visuals in the browser. **Time:** 70 minutes plus a 10-minute share-back. **Start:** Editable team model with the measures from the [model lab](01-semantic-model.md).

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

[Next: Team challenge](04-team-challenge.md)
