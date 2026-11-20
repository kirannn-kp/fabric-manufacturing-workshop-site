# OneLake and Lakehouse

**Goal:** Upload eight source files, turn them into tables, and inspect them. **Time:** 65 minutes plus a separate 20-minute verification. **Start:** Team workspace, edit access, and the eight files from the [data inventory](../../data/validation-summary.md).

1. Sign in at [Microsoft Fabric](https://app.fabric.microsoft.com/). In the left menu, select **Workspaces**, then your workshop workspace.
2. Select **New item**. Search for **Lakehouse** and select it. Enter `ManufacturingOperationsLH_<TeamName>`, replacing `<TeamName>` with your assigned name without angle brackets. Leave the schema option at the facilitator's preset/default. Select **Create**.
3. In the Lakehouse **Explorer**, hover over **Files** and select **...** > **Upload** > **Upload files**. Browse to the eight CSVs and upload them. Do not upload the validation Markdown or facilitator CSV in this exercise.
4. Open **Files**. Select `inventory.csv`. Look for `PlantId`, `MaterialId`, `AvailableQuantity`, and `ReservedQuantity`. **Check:** All eight CSVs appear under Files. This is the file area of your Lakehouse in OneLake.
5. On each CSV row, select **...** > **Load to tables** > **New table**. Use the filename without `.csv` for the table name. If asked, use the default schema, CSV format, comma separator, and **Column header** on. Select **Load**. Repeat once per file; do not append on a retry.
6. Expand **Tables** (and `dbo` if shown). Open `plants`, then `inventory`. Check that IDs and columns have the right names. Compare the three visible plants with the [expected counts](../../data/validation-summary.md).
7. For an exact count beyond the table preview, open **SQL analytics endpoint** from the top-right Lakehouse selector. Refresh Explorer if needed. If your endpoint offers **New visual query**, add the `inventory` table, select **Transform** > **Count rows**, and run the visual query. Repeat for `production_orders` or another assigned table. If unavailable, open the CSV in the browser or a plain-text viewer that shows line numbers and subtract the one header line; compare with the facilitator's verified table count. A source line count alone does not verify a loaded table.
8. Pair with a teammate: point to a raw file and its queryable Delta table, name one difference, and report any count discrepancy.

**Check:** `plants` has 3 rows, `inventory` has 180, and `production_orders` has 80. The model later uses prepared tables, not your personal practice copy.

**If you don't see it:** Missing **Lakehouse** or **Upload** usually means the wrong workspace or insufficient edit access. Ask the facilitator to check capacity and role. If creation or load is blocked, inspect the backup Lakehouse with a teammate. If columns are named `_c0` or counts include a header, ask before deleting anything; reload as a new team table with **Column header** on.

Screenshot placeholder: `01-lakehouse-files.png`.

[Next: Eventhouse](02-eventhouse.md)
