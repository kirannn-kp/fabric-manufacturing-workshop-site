# Define the question before answering it

**Goal:** Make five business terms precise. **Time:** 70 minutes. **Start:** Team model or worksheet, reference report, and the [business questions](../business-questions.md).

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

[Next: Power BI report](03-power-bi-report.md)
