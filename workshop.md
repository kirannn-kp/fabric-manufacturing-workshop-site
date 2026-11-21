---
published: true
type: workshop
title: Manufacturing Operations with Microsoft Fabric and Foundry
short_title: Fabric Manufacturing Workshop
description: A three-day workshop on ERP-to-OneLake architecture, Microsoft Fabric, grounded agents, and production readiness.
level: beginner
authors:
  - Kiran Panchal
contacts:
  - "@kirannn-kp"
duration_minutes: 990
tags: microsoft-fabric, manufacturing, onelake, lakehouse, real-time-intelligence, microsoft-foundry, data-agent
---

# Start here

Use one manufacturing scenario throughout: **Which machine should maintenance review first, and what evidence justifies a pilot?** You will also investigate whether open production orders can be used as context without implying that machine faults caused a production delay.

**Hours (Bengaluru local time):** Monday 09:30–16:00; Tuesday 09:30–17:00; Wednesday 09:30–16:00. Breaks are included below. No report authoring is required.

**Setup:** Use an assigned Fabric workspace and synthetic data only. Download [plants.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/plants.csv), [production_orders.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/production_orders.csv), and [machine_events.csv](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/machine_events.csv). The first two are stand-ins for exported ERP data; the third is machine telemetry. There is no live SAP connection in this lab. The facilitator supplies prepared Fabric items and, if available, an approved Foundry project and deployed model. Do not paste customer data, credentials, or internal architecture into GitHub or an agent.

**Team roles:** Driver uses the portal, navigator reads the steps, verifier checks every result against source data, and recorder captures evidence. Rotate after each major block. Keep outputs in a private shared document.

---

# Day 1 — Understand Fabric and Foundry, then investigate

| Time | Activity | Output |
|---|---|---|
| 09:30–10:00 | Kickoff and decision framing | One decision, decision owner, and three pieces of evidence |
| 10:00–10:55 | Ignite briefing and capability exercise | Capability map: usable today, preview, or design only |
| 10:55–11:10 | Break | |
| 11:10–12:30 | Fabric deep dive and Lakehouse lab | Loaded ERP sample, row counts, grain and data-quality checks |
| 12:30–13:15 | Lunch | |
| 13:15–14:15 | Eventhouse investigation | KQL evidence card and peer verification |
| 14:15–15:00 | Foundry deep dive and evaluation exercise | Three scored agent responses |
| 15:00–15:15 | Break | |
| 15:15–16:00 | SAP-to-agent architecture and walkthrough | Architecture sketch, access boundary, unresolved dependency |

### Kickoff and Ignite (85 minutes)

- **Kickoff (30):** Each team identifies who makes the maintenance decision, what counts as evidence, and what the available data cannot answer. Compare two teams' definitions, then agree on a common decision statement.
- **Briefing (20):** The facilitator picks up to three items from [Fabric what's new](https://learn.microsoft.com/en-us/fabric/fundamentals/whats-new) and [Foundry what's new](https://learn.microsoft.com/en-us/azure/foundry/whats-new-foundry). For each, separate the announcement from what the workshop workspace actually exposes. If the status cannot be verified, present the concept without claiming availability.
- **Capability exercise (35):** In teams, map storage, batch ingestion, event queries, governed metrics, Q&A, and rule monitoring to OneLake/Lakehouse, Data Factory, Eventhouse, semantic models, Data Agent/Foundry, and Operations Agent. Discuss where Warehouse, Power BI consumption, and Purview fit. Compare two trade-offs: Lakehouse versus Warehouse for serving curated data; Data Agent versus Foundry for a custom assistant.

### Lakehouse lab (80 minutes)

1. **15 min — Demonstrate:** In the assigned Lakehouse, upload `plants.csv` under **Files**. Use **More options > Load to Tables > New table** with header enabled. Open `Tables > dbo > plants`; verify three plants. Explain the difference between an uploaded file and a queryable table.
2. **25 min — Build independently:** Repeat for `production_orders.csv`. Identify each table's grain, a key, and the join column. In the Lakehouse SQL analytics endpoint, open **New SQL query** and run:

   ```sql
   SELECT COUNT(*) AS Orders, COUNT(DISTINCT ProductionOrderId) AS UniqueOrders
   FROM dbo.production_orders;

   SELECT p.PlantName, o.OrderStatus, COUNT(*) AS Orders
   FROM dbo.production_orders AS o
   JOIN dbo.plants AS p ON p.PlantId = o.PlantId
   GROUP BY p.PlantName, o.OrderStatus
   ORDER BY p.PlantName, o.OrderStatus;
   ```

3. **25 min — Investigate:** Independently amend the query to count open orders only. Then switch to a left join from orders to plants and filter for missing plant keys. Explain why this test uses a left join and what could go wrong if the lookup had duplicate IDs.
4. **15 min — Review:** Show your table preview and checks to a neighboring team. Expected: 80 distinct orders; 15 open at each plant; zero unmatched plant IDs. If imports are throttled, use prepared tables; if SQL access is unavailable, filter and count the CSV rows and record the same evidence.

### Eventhouse lab (60 minutes)

1. **15 min — Set up:** Open the prepared KQL database or create an Eventhouse and open its KQL database. Use **Get data > Local file > New table**; name the table `machine_events`, set **First row header**, and confirm `EventTime` is datetime. Skip ingestion if the table is already populated. Run `machine_events | count` via **Query > Query with code**; expect 500.
2. **15 min — Guided query:** Use one tab for fault counts. In a second tab, change the second filter to `Severity == "High"` and remove the threshold; do not keep both filters.

   ```kusto
   machine_events
   | where EventTime >= datetime(2026-11-01) and EventTime < datetime(2026-11-11)
   | where EventType == "Fault"
   | summarize EventCount = count() by PlantId, MachineId
   | where EventCount >= 3
   | order by EventCount desc, MachineId asc
   ```

3. **20 min — Independent challenge:** Compute daily faults with `bin(EventTime, 1d)`. Compare November 1–5 and November 6–10 UTC for both machines. Predict what changes before running. Then find the earliest High event by sorting raw High records by time.
4. **10 min — Peer check:** Share the query and recommendation, not just a number. If Eventhouse is unavailable, group the CSV using the same filters or use the evidence card and explain how you would reproduce it.

### Foundry deep dive and architecture (90 minutes)

- **Foundry (45):** Spend 15 minutes mapping model catalog, Agent Service, tools/retrieval, evaluation and tracing. For 20 minutes, compare three sample responses to “Which machine first?”: (A) “MC-N01: six High events; inspect first; cause unknown”; (B) “MC-N01 caused lost production”; (C) “Either machine; each had eight faults.” Score each 0/1 for agrees with data, explains priority, and avoids unsupported causality. Spend 10 minutes deciding what trace or tool output would help diagnose a wrong response. No deployment is needed for this exercise.
- **Architecture (45):** Sketch an approved ERP export or connector into OneLake landing, curated tables, metrics, Data Agent, and a Foundry assistant; add the telemetry-to-Eventhouse path separately. Mark human approval and access at each boundary. Swap diagrams for a 10-minute critique: identify one failure, who would notice it, and how to recover. Do not assume a join between faults and production orders proves causality.

**Day 1 handoff:** Lakehouse checks, two fault counts, severity evidence, and the architecture sketch.

---

# Day 2 — Prepare ERP data and build grounded Q&A

| Time | Activity | Output |
|---|---|---|
| 09:30–11:00 | SAP-to-OneLake design and data contract | Source-to-target mapping and ingest acceptance checks |
| 11:00–11:15 | Break | |
| 11:15–12:30 | Semantic modeling and ontology | Reconciled counts, relationship test design and entity map |
| 12:30–13:30 | Lunch | |
| 13:30–15:00 | Build and test a Fabric Data Agent | Five-question evaluation and one prompt improvement |
| 15:00–15:15 | Break | |
| 15:15–16:30 | Prioritize use cases and choose a product path | Ranked shortlist and one pilot charter |
| 16:30–17:00 | Lab preparation | Named Day 3 assets, team roles and fallback choice |

### SAP-to-OneLake design (90 minutes)

1. **20 min — Compare** an approved batch export, connector/pipeline, and mirroring where supported. Discuss latency, incremental key/change capture, network and licensing. Consult Microsoft's [SAP integration overview](https://learn.microsoft.com/en-us/azure/data-factory/industry-sap-overview) for patterns, then verify the specific connector and deployment prerequisites for your platform and SAP source. Do not assume direct SAP mirroring is available.
2. **30 min — Map fields:** For the sample orders, create a contract for `ProductionOrderId`, `PlantId`, `MaterialId`, `PlannedStartDate`, `RequiredQuantity`, and `OrderStatus`: source type, target type, key/null rule, and who owns it. Label source extraction details as assumptions, not SAP field mappings.
3. **25 min — Design three layers:** Landing preserves the source; curated rejects missing keys and tracks rejected rows; serving exposes approved fields and measures. Review scenarios: duplicate order ID, unmatched plant, corrupt planned date, restart of a partial batch. Decide reject/quarantine/retry, and how counts reconcile without silently dropping records.
4. **15 min — Peer review:** One team explains why rerunning an ingestion must not double-count. Another identifies one permission boundary and one quality alert. Save at least three acceptance tests for Day 3.

### Semantic model and ontology lab (75 minutes)

1. **15 min — Model walkthrough:** Open the prepared two-table Direct Lake semantic model. Draw the intended `plants[PlantId]` (one) → `machine_events[PlantId]` (many) relationship and predict the effect of a Plant North filter. The read-only query below tests event counts, **not** saved relationship behavior.
2. **25 min — DAX practice:** In **DAX query view > New query**, run the query. Then replace `"Fault"` with `"Run"` and compare grouped counts; explain why adding the two event types returns 500. If the model is unavailable, repeat the grouping in KQL or with the CSV.

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

3. **20 min — Ontology:** Draw `Plant → Machine → Fault Event` and trace Plant North → `MC-N01` → its fault at midnight UTC on November 1. Give each entity a key and name the relationships. Discuss why telemetry cannot establish that an ERP order was affected; design the missing machine-to-order assignment and its time validity. If a prepared Fabric IQ ontology is already verified, trace the same path there; the diagram is the default.
4. **15 min — Validate:** Compare your metric definition and ontology against another team. Define an actual relationship acceptance test: filtering the plant lookup to `P01` should give 8 faults with a saved measure, not 16. Execute it only if the prepared model already supports it; otherwise label it a test design.

### Fabric Data Agent lab (90 minutes)

1. **20 min — Set up:** Create a draft via **New item > Data agent**. Choose **Add a data source**, select the KQL database, and check `machine_events` under **Explorer > Sources > Tables**. Use **Test the agent's responses**. Don't publish team drafts.
2. **35 min — Test:** Ask which machines had at least three Fault events in November 1–10 UTC; which machine had High events regardless of type; what the earliest High timestamp was; whether any machine qualifies at nine faults; and which production orders were affected. Record output, reference query/result, and pass/fail. The last question must be declined or qualified: the connected source cannot answer it.
3. **20 min — Improve:** Start with “Which machine is worst?” Rewrite it to specify the date window, event type, severity rule, and desired columns. Compare before/after. If no agent is available, have a partner answer using only the connected dataset and score the responses with the same rubric.
4. **15 min — Compare architectures:** When does a Fabric Data Agent suffice? When does Foundry add value via orchestration, tools, or richer evaluation? Sketch one approved read-only tool call. Don't add write tools to the lab.

Use this scorecard on both days. Score each criterion 0 or 1; a response that invents an affected order fails the case regardless of its total.

| Test | Accuracy: expected evidence | Evidence: what to look for | Grounding: what must not be assumed |
|---|---|---|---|
| Repeated faults | `MC-N01` and `MC-S01`, 8 each | States November 1–10 UTC and 3+ threshold | Does not count Run rows |
| High severity | 6, all on `MC-N01` | Separates severity from event type | Does not require High events to be Faults by definition |
| Earliest High | Nov 1 at 00:00 UTC | One event timestamp and machine | Does not switch to local time silently |
| Negative control | None at 9+ | Same dates and event type | Does not reuse the 3+ result |
| Affected orders | Not enough evidence | Names the missing machine/order assignment | Does not invent an order or claim a delay |

### Prioritization and preparation (105 minutes)

- **75 min:** Propose three use cases (15); score value, data readiness, and controllability from 1–3 and write the reason for each score (25); have a partner team challenge the leading pick for missing access, bad data, and unsupported answers (20); choose one pilot and present a five-minute recommendation plus questions (15).
- **30 min:** Assign Day 3 roles, record the sample paths and prepared item names, and pick live or fallback route for pipeline, model, Foundry, and Operations Agent. Clarify explicitly which assets are synthetic and what permissions would change for a real SAP rollout.

**Day 2 handoff:** Data contract, five scored responses, pilot selection, and Day 3 lab routes.

---

# Day 3 — Build, evaluate, and plan production rollout

| Time | Activity | Output |
|---|---|---|
| 09:30–11:00 | Pipeline and serving lab | Copy result, reconciliation and semantic-model checkpoint |
| 11:00–11:15 | Break | |
| 11:15–12:30 | Foundry agent lab or structured replay | Tool trace or replay evidence, evaluation scorecard |
| 12:30–13:30 | Lunch | |
| 13:30–15:00 | Operations Agent and production hardening | Threshold replay, failure drill and rollout controls |
| 15:00–16:00 | Roadmap, commitments and showcase | One-page pilot proposal, owner and dated checkpoint |

### Pipeline and serving lab (90 minutes)

1. **15 min — Prepare:** In the assigned Lakehouse, stage `production_orders.csv` under **Files**. Agree on an isolated destination such as `dbo.orders_pipeline_<team>`; do not overwrite existing workshop tables. Choose a safe rerun behavior before authoring.
2. **30 min — Implement:** In the workspace, create a **Data pipeline** and add a **Copy data** activity. Configure a Lakehouse files source pointing to the staged CSV (delimited text, comma, first row as header). Use the Lakehouse **Tables** destination, select the isolated table, review column mapping, then run and inspect **Output/Monitor**. For the snapshot exercise choose overwrite only if supported and only for your isolated table; otherwise use a new destination per attempt.
3. **25 min — Validate:** Check rows read and written in monitoring, then run the Day 1 SQL checks against the new table. Verify 80 rows, 80 distinct orders, and zero missing plant IDs. Predict what a second run would produce and verify safely on that isolated destination (or use a new destination and compare counts). If Copy is unavailable, use **Files > Load to Tables** with a separate table, perform the same reconciliation, and draw the Copy activity configuration; that fallback doesn't demonstrate pipeline execution.
4. **20 min — Serve:** In an editable Lakehouse, use **New semantic model** and select the copied order table and `plants`; check table visibility and use DAX query view with `EVALUATE ROW("Orders", COUNTROWS('your_table_name'))`, substituting the actual table name. Expect 80. If creating a model is unavailable, use the Day 2 prepared model to review serving behavior and keep the SQL checks as copy evidence. The order pipeline and telemetry agent are separate paths in this workshop, not an asserted live SAP-to-agent integration.

### Foundry agent lab (75 minutes)

1. **15 min — Select your route:** The live route needs a prepared approved Foundry project/model and a **published** Fabric Data Agent on the synthetic `machine_events` source. A draft is not sufficient. The facilitator verifies the connection and logged-in identity before class using the [official Fabric tool setup](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/fabric).
2. **25 min — Build or replay:** In the approved Foundry project, create or open the workshop agent, select the deployed model, attach the published Data Agent as the Fabric tool using the linked instructions, and test in the playground. Instruct it to use Fabric for in-scope claims, include the dates and counts, and say when evidence is missing. No external write actions. **Fallback:** Use the Day 2 Data Agent outputs or reference evidence; one teammate plays the assistant, one plays the read-only tool returning only those records, and one records exactly which evidence the assistant used. Label this a tabletop, not a Foundry run.
3. **25 min — Evaluate:** Run or replay the five Day 2 cases. Score factual accuracy, evidence, and unsupported claims. Inspect the live tool call/trace if exposed; if not, mark trace unverified. Add an ambiguous date question, then compare the result after instructing the agent to ask for clarification rather than guess.
4. **10 min — Decide:** Share one failed case, a prompt or grounding change, and one condition that would block deployment. Don't claim automated evaluation was run unless the facilitator actually ran it.

### Operations Agent and production hardening (90 minutes)

1. **25 min — Review:** Open the prepared stopped Operations Agent or create a test item, add the synthetic KQL database under **Knowledge**, save instructions, and generate a playbook. Instruct it to group Fault events by plant and machine for the UTC window, recommend review at 3+ faults, and take no actions. Inspect the generated rule; the threshold may be separate from the generated KQL. Keep the agent stopped. If unavailable, annotate the same rule on paper.
2. **25 min — Replay:** Use the Day 1 repeated-fault query at thresholds 3, 8, and 9. Predict first, then compare. Repeat for November 1–5 and November 6–10 UTC at 3. Use severe events as a separate prioritization signal. Explain why the historical replay does not test live scheduling.
3. **25 min — Failure drill:** Assign one failure per team: empty batch, duplicate batch, lost data-source access, or an agent claim with no evidence. Write the detection signal, owner, response and recovery. Swap with another team and identify a missing control.
4. **15 min — Release gate:** Sketch dev/test/prod separation and an approval gate for pipeline, model, agent configuration, and a fixed evaluation set. Decide who owns monitoring, costs, least-privilege reviews, and rollback. No production deployment is part of the lab.

### Roadmap and showcase (60 minutes)

- **25 min:** Write a one-page pilot: one plant and shift; one maintenance decision and owner; approved data; candidate architecture; human approval before any inspection or ticket; 10 labeled replay windows; acceptable false alarms/missed cases set before the pilot; and a dated checkpoint.
- **25 min:** Swap plans, score evidence, feasibility, governance, and measurable outcomes. Present the strongest recommendation and one condition that would stop the pilot.
- **10 min:** Read back owners and dates. Record unverified features separately from validated evidence.

---

# Evidence card and references

| Check | Reference result |
|---|---|
| ERP samples | 3 plants; 80 unique orders; 45 open, 15 at each plant; no unmatched plant IDs |
| Telemetry | 500 rows: 484 Run and 16 Fault |
| November 1–10 UTC | `P01 / MC-N01`: 8 faults; `P02 / MC-S01`: 8 faults |
| First vs second five-day windows | `MC-N01`: 5 then 3; `MC-S01`: 5 then 3 |
| Severity | 6 High events, all on `MC-N01`; first `2026-11-01 00:00 UTC` |
| Threshold replay | Two qualifying machines at 3 or 8 full-window faults; none at 9 |
| Decision | `MC-N01` first for review; no supported conclusion about cause or affected orders |

If an entire service is unavailable, use this card and the downloaded CSVs to complete the same team outputs. Mark each output as **live**, **CSV check**, or **tabletop** so nobody confuses a fallback with a product test.

- [Quick-start technical rehearsal](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/docs/incident-lab.md)
- [Synthetic data inventory](https://raw.githubusercontent.com/kirannn-kp/fabric-manufacturing-workshop-site/main/data/validation-summary.md)
