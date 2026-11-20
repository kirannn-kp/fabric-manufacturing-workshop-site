# Day 1: Load and explore

| Time | Activity |
|---|---|
| 09:30–09:50 | Welcome. Read the [scenario](../01-business-scenario.md). Ask what evidence is needed to flag a production order. |
| 09:50–10:20 | Fabric and Ignite updates. Use the [official Fabric blog](https://blog.fabric.microsoft.com/) to pick at most two updates relevant to this tenant. Say whether each is available here, in preview, or plan-only. If none changes the labs, show the current UI and move on. |
| 10:20–10:50 | Explain the item map below. Identify the two optional branches. |
| 10:50–11:05 | Break. |
| 11:05–12:10 | [Lakehouse](01-onelake-lakehouse.md) hands-on. |
| 12:10–12:30 | Verify counts and discuss what stays in OneLake. |
| 12:30–13:15 | Lunch. |
| 13:15–14:20 | [Eventhouse or report fallback](02-eventhouse.md) hands-on. |
| 14:20–14:35 | Compare the fault and high-severity findings. |
| 14:35–14:50 | Break. |
| 14:50–15:50 | [Data Agent or manual answer checking](03-data-agent.md). |
| 15:50–16:00 | Checkpoint: show one Lakehouse table, name one repeated-fault machine, and explain one change that improved a question or answer. |

**Item map:** A workspace holds the items. OneLake stores the files and tables. The Lakehouse makes those files and tables easy to inspect. Eventhouse holds a separate copy of the machine events in a KQL database to practice event analysis. The semantic model names, relates, and measures the data. A Power BI report shows the results. A Fabric Data Agent can answer questions using the approved model. You use the prepared model and report today, then improve them tomorrow.

[Workshop home](../../workshop.md) · [Day 2](../day-2/README.md)
