---
published: true
type: workshop
title: Manufacturing Operations with Microsoft Fabric and Foundry
short_title: Fabric Manufacturing Workshop
description: Two hands-on Fabric days on machine-fault evidence, followed by a facilitator-led Foundry walkthrough and pilot scoping.
level: beginner
authors:
  - Kiran Panchal
contacts:
  - "@kirannn-kp"
duration_minutes: 840
tags: microsoft-fabric, manufacturing, onelake, lakehouse, real-time-intelligence, microsoft-foundry, data-agent
---

# Start here

**Problem statement:** You are the incoming shift lead. Two machines have repeated faults. **Which machine should maintenance review first, and what evidence justifies that choice?** Build the data and queries on Day 1, use the same assets to model and explain the evidence on Day 2, and decide how to pilot the workflow on Day 3.

**Bengaluru local time:** Monday and Tuesday 09:30–16:00; Wednesday 09:30–14:00.

## Prerequisites and downloads

- **Laptop:** Bring a charger and use Microsoft Edge or Google Chrome. Check sign-in to [Microsoft Fabric](https://app.fabric.microsoft.com/) and Microsoft Teams if your group uses it for collaboration.
- **Workspace:** Open the Fabric workspace shared by the organizer in the correct tenant. Check that you can see **New item** and have permission to create and edit items. At least one person per team needs working edit access on an active Fabric capacity. Pair with that person if you have read-only access. Keep using the same workspace and item names on both Fabric days.
- **Features:** Confirm access to Lakehouse, Eventhouse, and a semantic model. Data Agent, model editing, ontology, and Operations Agent depend on the environment. Use the fallback in the relevant lab if a feature is unavailable. The facilitator shares prepared items privately.
- **Files:** Download [plants.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/plants.csv) and [machine_events.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/machine_events.csv). Save them as `.csv`, not `.csv.txt`. The data is synthetic; there is no live SAP connection or data preparation before class. GitHub is only for instructions and downloads; no GitHub Copilot, VS Code, Power BI Desktop, or report authoring is required.
- **Day 3:** No participant Foundry account, Azure subscription, or deployment is required. The facilitator leads the walkthrough.
- **If blocked:** Tell the facilitator privately which product and step failed, the exact error, and the time. Include a screenshot with sensitive information redacted. Switch to the prepared item or evidence card rather than waiting through the lab.

**Team setup:** One driver uses Fabric, one person verifies the result, and one records the evidence. Rotate roles between labs. Keep your notes and workspace links private.

---

# Day 1 — Build the data assets in Fabric

| Time | Activity | What you leave with |
|---|---|---|
| 09:30–10:00 | Welcome and the maintenance problem | A shared decision and list of evidence needed |
| 10:00–10:45 | Fabric concepts: workspace, OneLake, Lakehouse, and Eventhouse | A map of where files, tables, and telemetry live |
| 10:45–11:00 | Break | |
| 11:00–11:30 | Facilitator demo: from file to table and query | A working example to follow |
| 11:30–12:30 | Lakehouse build and SQL validation | Two queryable tables and a plant-level quality check |
| 12:30–13:15 | Lunch | |
| 13:15–14:30 | Eventhouse build and guided KQL | One KQL database and three verified results |
| 14:30–14:45 | Break | |
| 14:45–15:35 | Independent investigation and Real-Time Dashboard | A maintenance dashboard or saved query results |
| 15:35–16:00 | Show what you built and save the Day 2 handoff | Workspace/item names and reproducible evidence |

### 09:30–11:30 — Understand the flow before building

- **Frame the decision:** Ask what a shift lead needs to know before recommending a maintenance review. Agree that repeated faults and High severity are evidence for priority, not proof of root cause.
- **Explain the Fabric items:** A workspace holds the items; OneLake stores the Lakehouse data; the Lakehouse makes files queryable as tables; the Eventhouse KQL database is the event-analysis copy. Briefly show where semantic models and agents fit tomorrow. No second scenario or tool setup is needed.
- **Demonstrate:** Show `plants.csv` as a file, then as a table. Show the separate KQL database and a row-count query. Explain why the same telemetry CSV is loaded twice: the Lakehouse copy feeds the semantic model, and the KQL copy feeds the agents.

### 11:30 — Build the Lakehouse

1. **Guided (15 min):** Choose **New item > Lakehouse**, name it `MaintenanceLH_<Team>`, and keep Lakehouse schemas enabled. In **Files**, upload `plants.csv`. From the file's **More options**, select **Load to Tables > New table**. Keep `dbo`, name it `plants`, and keep the comma separator and header option enabled. Open **Tables > dbo > plants**. Check `P01`, `P02`, and `P03`.
2. **Build independently (20 min):** Repeat with `machine_events.csv`, naming the table `machine_events`. Identify `EventTime`, `PlantId`, `MachineId`, `EventType`, and `Severity`. Make sure both tables appear under **Tables**, not only **Files**. If an import is pending or throttled, don't repeatedly upload it; use the prepared Lakehouse.
3. **Validate in SQL (25 min):** Switch the Lakehouse to its **SQL analytics endpoint**, open **New SQL query**, and run the query below. The endpoint lets you query the same tables without copying them or changing the source data. Predict whether Plant East will appear before running. Then change `LEFT JOIN` to `INNER JOIN` and explain the difference to a teammate.

   ```sql
   SELECT p.PlantId, p.PlantName, COUNT(e.PlantId) AS FaultCount
   FROM dbo.plants AS p
   LEFT JOIN dbo.machine_events AS e
       ON e.PlantId = p.PlantId
       AND e.EventType = 'Fault'
       AND e.EventTime >= '2026-11-01T00:00:00'
       AND e.EventTime <  '2026-11-11T00:00:00'
   GROUP BY p.PlantId, p.PlantName
   ORDER BY p.PlantId;
   ```

   **Check:** North 8, South 8, East 0. The inner join drops East. If tables are still syncing or the SQL endpoint is unavailable, use the prepared endpoint or group the CSV and retain all three plants from the lookup.

   **Fast finish:** Reverse the join: start from `machine_events`, left join `plants` on `PlantId`, and count rows where `p.PlantId IS NULL`. Expect zero unmatched events. Explain how an unmatched event could affect a plant filter in tomorrow's model.

### 13:15 — Build and query Eventhouse

1. **Set up (25 min):** Choose **New item > Eventhouse**, name it `MaintenanceEvents_<Team>`, and open its KQL database. Select **Get data > Local file > New table**. Upload `machine_events.csv`, name the table `machine_events`, choose CSV with **First row header**, and confirm `EventTime` is datetime. Finish and wait. If using a prepared KQL database, skip the upload. Do not ingest twice.
2. **Query (35 min):** Open **Query > Query with code**. Paste one block per tab and select **Run**. Write down the UTC window, filters, and each result.

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

3. **Check (15 min):** Expect 500 rows; 8 faults per qualifying machine; and 6 High events, all on `MC-N01`. The severity query does not filter on `EventType`. If KQL is unavailable, filter the CSV or use the evidence card and explain how the results would be reproduced.

### 14:45 — Investigate, build a dashboard, and hand off

- **Independent investigation (20 min):** Copy the repeated-fault query into two new tabs. Change the first window to November 1–5 inclusive (`< datetime(2026-11-06)`) and the second to November 6–10 inclusive (`>= datetime(2026-11-06)`). Predict the results, then verify both machines go from five to three faults.
- **Build a Real-Time Dashboard (30 min):** In a [KQL queryset](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/dashboard-real-time-create) connected to your Day 1 database, run the original repeated-fault query and choose **Save to dashboard > To a new Dashboard**. Name it `MaintenanceReview_<Team>`. Run the High-severity query in another tab and save it to that dashboard as a second tile. Name the tiles **Repeated faults** and **High-severity events**. Verify two groups in the first tile and six events in the second. This is a Fabric Real-Time Dashboard, not a Power BI report or a live feed. If dashboard creation is unavailable, keep the two results in named query tabs or private team notes.
- **Handoff (25 min):** Ask a teammate to use only your dashboard or query results to recommend a machine, explain the evidence, and name one thing the records cannot prove. Record the workspace and item names, the two results, one representative event, and the recommendation. Reopen these assets tomorrow.

---

# Day 2 — Use the same assets to build in Fabric

| Time | Activity | What you leave with |
|---|---|---|
| 09:30–09:50 | Reopen Day 1 assets and confirm the evidence | Same tables, database, dates, and decision |
| 09:50–11:10 | Build a semantic model and check the numbers | Two-table model or read-only result |
| 11:10–11:25 | Break | |
| 11:25–12:30 | Build a Fabric Data Agent and test the first questions | A draft agent connected to your KQL database |
| 12:30–13:15 | Lunch | |
| 13:15–14:05 | Improve and retest the agent | A completed answer-check scorecard |
| 14:05–14:45 | Map the incident with an ontology | An entity diagram or guided live walkthrough |
| 14:45–15:00 | Break | |
| 15:00–15:45 | Review and replay an Operations Agent rule | A threshold check, with no actions running |
| 15:45–16:00 | Demonstrate the Fabric flow and prepare Day 3 evidence | One recommendation and one limitation |

### 09:30 — Reopen and build the semantic model

1. **Reopen (20 min):** Locate yesterday's `plants` and `machine_events` Lakehouse tables and rerun the repeated-fault KQL against yesterday's database. The model will use the Lakehouse copy; the Data Agent will use the KQL copy. This lab does not require connecting the Data Agent to the semantic model.
2. **Create (20 min):** In the Lakehouse choose **New semantic model**, name it `MaintenanceModel_<Team>`, select only `plants` and `machine_events`, and confirm. If creation is blocked by licensing or permissions, open the prepared two-table model. No report is needed.
3. **Validate (30 min):** Open **DAX query view > New query**, paste the following, and select **Run**. It works in Viewing mode and does not change the model. Check 8 faults per machine, then replace `"Fault"` with `"Run"` and compare the totals.

   ```dax
   EVALUATE
   SUMMARIZECOLUMNS(
       machine_events[PlantId], machine_events[MachineId],
       "Events",
       CALCULATE(
           COUNTROWS(machine_events),
           KEEPFILTERS(machine_events[EventType] = "Fault"),
           machine_events[EventTime] >= DATE(2026, 11, 1),
           machine_events[EventTime] < DATE(2026, 11, 11)
       )
   )
   ```

4. **Explain the relationship (30 min):** Draw `plants[PlantId]` (one) → `machine_events[PlantId]` (many). Predict what a Plant North filter should do. If the facilitator has verified editing, create the active single-direction relationship in **Manage relationships** and add a measure to `machine_events`: `Machine fault count = CALCULATE(COUNTROWS(machine_events), machine_events[EventType] = "Fault")`. In the same incident window, check 16 overall and 8 when filtered through `plants` to `P01`. If editing is unavailable, keep the drawing and validated read-only result; the query above does not prove the relationship works. If the model cannot open, use KQL and the plant lookup.

### 11:25 and 13:15 — Build and improve the Data Agent

1. **Connect (25 min):** Choose **New item > Data agent**. In the draft, choose **Add a data source**, select your Day 1 **KQL database**, not the Eventhouse container, and select **Add**. Under **Explorer > Sources > Tables**, check `machine_events`. Keep the agent a draft.
2. **Test (25 min):** In **Test the agent's responses**, use the first two prompts below. Compare each answer to the Day 1 query and mark pass, partial, or fail.
3. **Compare (15 min):** Ask “Which machine is worst?” and compare its answer to the more precise prompt for repeated faults. Discuss what the vague question omitted.
4. **Retest (30 min):** After lunch, run the remaining three prompts and rewrite one weak prompt to include a date window, exact filters, and output format. Record what changed between answers.
5. **Explain (20 min):** Exchange scorecards. Use the Day 1 dashboard or saved queries to verify one answer independently and identify an unsupported claim. If the agent is unavailable, take turns answering with only the Day 1 evidence and score the answers using the same criteria.

**Fast finish:** Compare fault counts with fault rates by plant using the same window. In KQL, summarize `TotalEvents = count(), Faults = countif(EventType == "Fault") by PlantId`, then calculate `FaultRatePct = round(100.0 * Faults / TotalEvents, 2)` with `extend`. Explain why a fault rate does not replace the severity evidence. If the dashboard is available, add the result as a third tile.

| Prompt (use November 1–10, 2026 UTC) | Check before accepting |
|---|---|
| Which machines had at least three Fault events? Return plant, machine, and exact count. | `P01 / MC-N01`: 8; `P02 / MC-S01`: 8. |
| How many High events occurred, regardless of event type, and on which machines? | 6, all on `MC-N01`. |
| What was the earliest High event? Give the UTC timestamp. | November 1 at 00:00 UTC on `MC-N01`. |
| Did any machine record at least nine Fault events? | None. |
| Which production orders were affected by these faults? | The connected data does not say; don't invent orders or causal links. |

### 14:05 — Map the evidence with an ontology

- **Explain (10 min):** The semantic model defines how to count. An ontology describes the things in the operation and their relationships.
- **Draw and trace (25 min):** Draw `Plant → Machine → Fault Event`. Key a plant by `PlantId`, a machine by `PlantId + MachineId`, and an event by the machine key plus `EventTime` in this dataset. Trace `P01 → MC-N01 → 2026-11-01 00:00 UTC`. If a prepared Fabric IQ ontology is available, trace the same record there. No live ontology item is required.
- **Peer check (5 min):** Explain what extra data would be needed to relate an event to a production order.

### 15:00 — Review the Operations Agent and hand off

1. **Inspect (15 min):** Open the facilitator's stopped Operations Agent, connected to the synthetic Day 1 KQL database or a prepared copy with the same data. If explicitly enabled for the lab, follow along by creating a test item, adding that database under **Knowledge**, and saving instructions to group `Fault` events by plant and machine, flag 3+ in the evaluated UTC window, and recommend human review only. Generate or open the playbook. Keep it stopped; configure no actions. If unavailable, annotate the rule in your notes.
2. **Replay (20 min):** Use the Day 1 reference query at `>= 3`, `>= 8`, and `>= 9`. Check two machines qualify at both 3 and 8, and none at 9. The replay checks the rule against historical data, not scheduling or notifications. Explain why High severity is a separate signal for ranking.
3. **Decide (10 min):** Write a one-sentence handoff: which machine should the lead review first, what is the supporting evidence, and what still needs a human decision?
4. **Showcase (15 min):** Share your model or read-only result, one checked Data Agent answer, and the rule decision. Carry those three items into Day 3.

---

# Day 3 — Foundry walkthrough and PoC, finish at 14:00

| Time | Activity | What you leave with |
|---|---|---|
| 09:30–10:00 | Recap the Fabric investigation | The decision and its evidence |
| 10:00–11:00 | Facilitator-led Foundry overview and example | A clear idea of what Foundry adds and when it isn't needed |
| 11:00–11:15 | Break | |
| 11:15–12:15 | Scope one proof of concept | Draft pilot with boundaries and success checks |
| 12:15–13:00 | Lunch | |
| 13:00–13:40 | Review the scope and identify blockers | Dependencies, owners, and approval gates |
| 13:40–14:00 | Agree next steps and close | A dated checkpoint and action list |

### Foundry overview (no participant setup)

Use about 20 minutes for the model, agent instructions, read-only tools, and evaluation; 20 minutes to show how an assistant might answer the Day 2 maintenance question using Fabric evidence; and 20 minutes to discuss when the existing Fabric Data Agent is enough. If the facilitator has an already working environment, show it. Otherwise use a diagram and the Day 2 answers. On both paths, compare an evidence-backed response with one that incorrectly claims a root cause. Don't publish team drafts or provision a project during the workshop.

### One-page PoC

| Decide | Write down |
|---|---|
| Problem and scope | One plant, one shift handoff, and one maintenance decision. What is out of scope? |
| User and owner | Who reviews the evidence, who owns the pilot, and who approves any action? |
| Data and access | What approved fields are needed, who owns them, and what mapping is missing? |
| Success | Replay ten labeled windows with positive and negative cases. Agree acceptable false alarms and missed cases **before** running the pilot. |
| Boundaries | Read-only recommendations. No automatic messages, tickets, or maintenance actions. |
| Next step | One blocker, one owner, one due date, and a review checkpoint. |

Draft privately for 30 minutes, exchange plans for 20 minutes, then revise for 10 minutes. After lunch, check the data and approval dependencies against the scope. Close by reading back the owner, next action, and date. No production deployment is part of the workshop.

---

# Evidence card and help

| Check | Reference result |
|---|---|
| Lakehouse inputs | Three plants; `P01` is Plant North, `P02` Plant South, `P03` Plant East. |
| KQL sample | 500 events: 484 Run and 16 Fault. |
| SQL validation | Plant North 8, Plant South 8, Plant East 0; zero unmatched event plant IDs. |
| Fault-rate extension | North 8/169 (4.73%); South 8/170 (4.71%); East 0/161 (0%). |
| November 1–10 UTC | `P01 / MC-N01`: 8 faults; `P02 / MC-S01`: 8 faults. |
| First vs. second five-day windows | Both machines: 5 then 3 faults. |
| Severity | 6 High events, all on `MC-N01`; first at `2026-11-01 00:00 UTC`. |
| Recommendation | Review `MC-N01` first. Neither root cause nor affected orders can be inferred. |

If you used a fallback, label the result **prepared item**, **CSV check**, or **walkthrough**. That is valid workshop evidence, but it is not a live product test.

- [Synthetic data inventory](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/validation-summary.md)
- [Microsoft Fabric documentation](https://learn.microsoft.com/en-us/fabric/)
- [Microsoft Foundry documentation](https://learn.microsoft.com/en-us/azure/foundry/)
