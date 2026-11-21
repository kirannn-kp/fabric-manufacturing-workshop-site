# Manufacturing incident lab: from evidence to action

You are the incoming shift lead. Two machines faulted repeatedly. Decide which one maintenance should review first, explain your evidence, and propose a small pilot. Use the same synthetic incident across all three days. No one needs to author a Power BI report.

**Before the workshop:** Sign in to the assigned Microsoft Fabric workspace. Download [plants.csv](../data/plants.csv) and [machine_events.csv](../data/machine_events.csv) if your team will load its own data. The facilitator shares any prepared items privately and tells you which features are available. If a feature is missing, use that step's fallback and carry the same evidence forward.

## Day 1 — Find and verify the incident

1. In your team Lakehouse, upload [plants.csv](../data/plants.csv) under **Files**. On the file's **More options** menu, select **Load to Tables > New table**. Keep `dbo`, `plants`, the comma separator, and **Use header for column names**. Open the table under **Tables > dbo**. **Check:** Three rows. `P01` is Plant North and `P02` is Plant South. The facilitator prepares other tables; don't wait for everyone to import all eight participant files.
2. In your test Eventhouse, open the KQL database and select **Get data > Local file**. Choose **New table**, name it `machine_events`, upload the CSV, confirm **First row header** and that `EventTime` is a datetime, and finish ingestion. Wait for it to complete; don't upload the file twice. Skip ingestion if using a prepared database. Open **Query > Query with code** and run these blocks in separate query tabs.

   ```kusto
   machine_events
   | count
   ```

   ```kusto
   machine_events
   | where EventTime >= datetime(2026-11-01) and EventTime < datetime(2026-11-11)
   | where EventType == "Fault"
   | summarize FaultCount = count() by PlantId, MachineId
   | where FaultCount >= 3
   | order by FaultCount desc, MachineId asc
   ```

   ```kusto
   machine_events
   | where EventTime >= datetime(2026-11-01) and EventTime < datetime(2026-11-11)
   | where Severity == "High"
   | project EventTime, PlantId, MachineId, EventType
   | order by EventTime asc
   ```

   **Check:** 500 rows; two machines with eight faults each; six High-severity events, all on `MC-N01`. The third query checks severity independently of event type. If you can't open Eventhouse, use the evidence card in **Help and references**.

3. In the workspace, select **New item > Data agent**. In the draft agent, select **Add a data source**, choose the workshop **KQL database** (not the Eventhouse container), select **Add**, and check `machine_events` under **Explorer > Sources > Tables**. Don't publish or add other sources.
4. In **Test the agent's responses**, ask: “From November 1 through November 10, 2026 UTC, which machines had at least 3 Fault events? Give the plant ID, machine ID, and exact fault count for each.” Then ask: “Independently of event type, how many High-severity events occurred in that window, on which machines, and when was the earliest?” **Check:** Compare both replies to KQL. If the agent is unavailable, use the two questions as a peer exercise and check the answers against KQL.

**Hand off:** Write down the two machine IDs, counts, UTC window, and the evidence that distinguishes them.

## Day 2 — Give the evidence business context

1. **Semantic model: how we count.** Open the facilitator-prepared model and choose **DAX query view > New query**. Run the query below to count faults by plant and machine for the incident window. **Check:** `P01 / MC-N01`: 8; `P02 / MC-S01`: 8. This works in Viewing mode and does not change the model.

   ```dax
   EVALUATE
   SUMMARIZECOLUMNS(
       machine_events[PlantId],
       machine_events[MachineId],
       "Fault count",
       CALCULATE(
           COUNTROWS(machine_events),
           KEEPFILTERS(machine_events[EventType] = "Fault"),
           machine_events[EventTime] >= DATE(2026, 11, 1),
           machine_events[EventTime] < DATE(2026, 11, 11)
       )
   )
   ```

   If the facilitator has verified the one-to-many `plants[PlantId]` to `machine_events[PlantId]` relationship and saved a fault-count measure, compare the query with the measure filtered to Plant North. The query above groups event rows directly; it does not test the relationship. If attendees cannot open the model, use the Day 1 KQL result and map `P01` to Plant North using the plants table or the evidence card. No model or report authoring is required.

2. **Ontology: what the records represent.** Draw `Plant → Machine → Fault Event` and trace `P01 → MC-N01` to one of its fault rows. Label the keys: `PlantId` for a plant, `PlantId + MachineId` for a machine, and machine key plus `EventTime` for an event in this dataset. Unlike the semantic model exercise, which defines how to count, this exercise defines which entities exist and how they relate. If the facilitator has a prepared and verified Fabric IQ ontology, trace the same path there. The diagram does not require the preview feature.
3. **Team challenge:** Both machines have eight faults. Which should a maintenance lead review first, and why? Include the date window and one uncertainty. **Check:** `MC-N01` is a defensible first review because it is the only machine with High-severity events. The event data does not establish the cause of the faults.

**Hand off:** One maintenance recommendation, its supporting evidence, and one question for the maintenance lead.

## Day 3 — Review an Operations Agent rule and design a pilot

1. **Facilitator demo:** In a test workspace, choose **New item > Operations agent**. In **Agent setup**, dismiss the Copilot panel if it covers the form. Under **Knowledge > Add data**, choose the synthetic KQL database. For **Agent instructions**, enter: “Use only `machine_events`. Group Fault events by PlantId and MachineId in the evaluated UTC window. Flag groups with at least 3 faults. Recommend human maintenance review only. Do not send messages or execute actions.” Select **Save**, then **Generate playbook**. Leave the agent stopped; do not add actions or send notifications.
2. Expand the generated rule. Check the event type (`Fault`), grouping keys (`PlantId`, `MachineId`), UTC window, and threshold (`>= 3`). The generated query may return groups before the rule threshold is applied. Replay the rule using the **repeated-fault query from Day 1** rather than editing the generated code. It is the reference for this fixed historical window, not a test of the agent scheduler. **Check:** The rule flags both machines; it does not by itself rank them. Use the High-severity evidence to prioritize `MC-N01`. As a negative check, change the reference query threshold to `>= 9`; it should flag no machines. If the playbook is unavailable, review the same rule and checks as a team.
3. **Plan the pilot:** Pick one plant, one shift process, and one human owner. Fill in the handoff below. Do not treat this batch as a live stream.

   | Write down | Example |
   |---|---|
   | Scope and owner | Plant North shift handover; maintenance lead reviews the queue. |
   | Evidence and priority | At 3+ faults both machines qualify. Review `MC-N01` first because it also had six High-severity events. Confirm the cause separately. |
   | Human approval | The rule only recommends review. The lead decides whether to inspect or open a ticket; nothing is sent automatically. |
   | Success criteria | Before live use, replay ten labeled windows with both positive and negative cases. Compare flagged cases to the lead's labels and record false alarms and missed cases separately. |
   | Next checkpoint | Name the owner, set a review date, and agree on acceptable error counts before running the pilot. |

**Finish:** Hand over a one-page pilot proposal with the owner, success criteria, and next checkpoint. Foundry and continuous monitoring are extensions, not requirements.

## Help and references

### Evidence card

| Check | Expected result |
|---|---|
| Plant lookup and event count | Plant North (`P01`), Plant South (`P02`), Plant East (`P03`); 500 event rows |
| Machines with at least three Fault events, November 1–10, 2026 UTC | `P01 / MC-N01`: 8; `P02 / MC-S01`: 8 |
| High-severity events, same window, regardless of event type | 6, all on `MC-N01`; earliest `2026-11-01 00:00:00 UTC` |
| What the evidence does not prove | Root cause or any connection between the faults and material shortages |

### Resources

- [Synthetic data inventory](../data/validation-summary.md)
- [Microsoft Fabric references](references.md)
