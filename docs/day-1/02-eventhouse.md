# Eventhouse: find repeated machine faults

**Goal:** Find machines with repeated faults and inspect severe events. **Time:** 65 minutes plus a 15-minute comparison. **Start:** Team workspace with an enabled Eventhouse and `machine_events.csv` from the [data inventory](../../data/validation-summary.md). If blocked, use the fallback below.

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

## If Eventhouse is not enabled

1. Open the facilitator's `Manufacturing Operations - Reference` report > **Machine events**. It uses `machine_events.csv` through the prepared Lakehouse and model.
2. Set the date window to 1–10 November 2026 UTC (inclusive), or leave the all-data default if it already covers that range. Clear any plant or severity selection. Read the **Repeated faults** table, which shows only `Fault` counts of 3 or more per plant/machine.
3. Read the **High severity events** table. It filters **Severity = High** independently of the fault count. Check that you get the same two machines and six events as above.
4. Explain what you would ask maintenance to check first. These are batch records, not a live feed.

**If you don't see it:** Check database permissions and the header mapping; open the facilitator's prepared Eventhouse if only ingestion is blocked. If the event data or query still does not work, take the report path rather than retrying. If report access fails too, pair with a teammate or ask the facilitator to share the private view.

Screenshot placeholder: `02-eventhouse-query.png`.

[Next: Fabric Data Agent](03-data-agent.md)
