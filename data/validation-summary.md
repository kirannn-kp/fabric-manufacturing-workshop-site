# Synthetic data and validation

All rows are invented. The plants, cities, suppliers, machine IDs, products, and transactions are fictional training examples. Download the eight source files without opening and resaving them in a spreadsheet; that can alter dates or IDs.

| Download | Rows, not including header | Grain or key |
|---|---:|---|
| [plants.csv](plants.csv) | 3 | One row per `PlantId`. |
| [materials.csv](materials.csv) | 15 | One row per `MaterialId`. |
| [suppliers.csv](suppliers.csv) | 6 | One row per `SupplierId`. |
| [purchase_orders.csv](purchase_orders.csv) | 84 | One row per `PurchaseOrderId`; references supplier, material, and plant. |
| [production_orders.csv](production_orders.csv) | 80 | One row per `ProductionOrderId`; references plant and material. |
| [inventory.csv](inventory.csv) | 180 | One row per `InventoryDate` + `PlantId` + `MaterialId`. |
| [quality_inspections.csv](quality_inspections.csv) | 105 | One row per `InspectionId`; references plant. |
| [machine_events.csv](machine_events.csv) | 500 | One event per `EventTime` + `PlantId` + `MachineId`; references plant. |

The separate facilitator-only production risk snapshot contains **45 rows**, one per eligible production order. It is not a ninth source for the participant upload exercise.

## Rules

- Open production orders starting **11–17 November 2026** use the latest inventory snapshot at or before **10 November 2026** for their plant and material. `AvailableQuantity` is gross of reservations. Usable stock is the greater of zero and available minus reserved. Shortage is the greater of zero and required minus usable. Missing inventory means unknown, not zero. Each eligible order in this sample has its own plant/material pair; this is not an allocation model. Safety stock is not deducted.
- Supplier context is the supplier on the most recently placed purchase order for the same plant and material on or before the snapshot date. Break order-date ties by highest `PurchaseOrderId`. This includes both open and delivered purchase orders and says nothing about assignment or cause. No match means `Unknown`.
- Supplier lateness uses actual delivery dates **1–10 November**, only for `Delivered` orders with actual dates. Late means actual date is later than promised. Two or more such orders make a supplier repeatedly late.
- Machine events use UTC, **1 November inclusive to 11 November exclusive**. Three or more `Fault` events per plant/machine means repeated faults; `Severity = High` is a separate filter.
- Quality covers **1–10 November**. Rejection rate is total rejected divided by total inspected. A rate of at least 10% is high. All sample materials use `Each`.

## Deliberate findings and example records

| Finding | Evidence |
|---|---|
| 12 orders have a shortage, 4 per plant, totaling 420 units. | PR001 needs 100 units. P01/M01 has 75 gross and 10 reserved, so 65 usable and 35 short. |
| Supplier A and B have repeated late deliveries. | PO001 was promised 1 November and delivered 3 November. A: 8/12 late; B: 6/12. Each other supplier: 0/12. The 12 open purchase orders are not part of the denominator. |
| Two machines repeatedly fault, and high severity is visible. | P01 / MC-N01 and P02 / MC-S01 have 8 faults each. Six High events are all on MC-N01, including the first at `2026-11-01T00:00:00Z`. |
| Drive Unit A has a high rejection rate. | QI001 records 15 rejects out of 100. Drive Unit A totals 525 of 3,500 (15.0%). Pump Unit B is 3.0%; Valve Unit C is 1.0%. |

CSV headers, row counts, keys and joins, and the findings above were checked locally. Lakehouse type inference and live query/report results still need the rehearsal in the organizer's Fabric workspace.
