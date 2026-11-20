# Fabric Data Agent: check the answer

**Goal:** Test answers against a prepared report and improve one answer. **Time:** 60 minutes. **Start:** Team workspace, approved baseline semantic model and reference report, and the [five questions](../business-questions.md). Do not use any other source.

1. In your workspace, select **New item** > **Fabric data agent**. Name it `ManufacturingRiskAgent_<TeamName>`.
2. In the OneLake catalog, select `Manufacturing Operations - Baseline` and then **Add**. In **Explorer**, select only `production_risk`, `plants`, `purchase_orders`, `suppliers`, `machine_events`, and `quality_inspections`. If model-level selection is controlled by its AI data schema in your tenant, use the facilitator's preset equivalent instead. No external data or extra sources are needed.
3. Select **Data agent instructions** (look under **Setup** if needed). Paste the text below. Save if prompted.

   > Answer only from the workshop's synthetic model. State the filter and measure used. For production risk, use the 10 November 2026 snapshot and open orders starting 11–17 November. At risk means a positive material shortage. The late percentage uses only delivered purchase orders. Machine faults count only Fault events and event time is UTC. Quality rejection rate is rejected units divided by inspected units. Supplier context is not cause. If evidence is missing, say so.

4. Append an **Example questions** heading and copy all five numbered questions from the [question sheet](../business-questions.md) into the agent instructions as plain text. Power BI semantic models do not support the agent's **Example queries** question/query-pair feature. Use the five items as test prompts, not verified answers. Do not change the shared model.
5. In the test chat, ask all five questions, one at a time. Compare with the report. Record the value, filters, and evidence. Expand the execution steps if shown. Do not publish the agent.
6. Try an ambiguous prompt: “Which supplier is late?” Find a weak or underspecified answer. Amend the team agent instruction to say: “For supplier comparisons, report the late count and delivered count and exclude open orders. Use 1–10 November 2026 by actual delivery date unless asked otherwise.” If the team has the editable model and time, add the same definition as a measure description instead.
7. Clear the chat so the first reply does not influence the second. Retest the troublesome question and the precise version from the sheet. Label the answer correct, incomplete, or wrong. An answer can be correct even if a different turn was weak; do not invent a failure.

**Check:** You can point to the source visual for each answer and explain whether the change improved the answer.

## If Fabric Data Agent is not enabled

1. Open `Manufacturing Operations - Reference`. Use the [same five questions](../business-questions.md).
2. For each, record the filters, evidence visual, and answer. For “Which supplier is late?”, first ask a teammate to answer without a shared definition.
3. Add a plain-language definition to your editable team model, or write it on the worksheet if you cannot edit. Clarify the date range and denominator. Ask the revised question and check the new answer against the same report. Compare what changed.

**If you don't see it:** Check the capacity and tenant setting with the facilitator. A model missing from the catalog may need Read permission or region alignment. If the agent times out or cannot answer, switch to manual checking. A plausible reply is not proof.

Screenshot placeholder: `03-agent-test.png`.

[Day 2](../day-2/README.md)
