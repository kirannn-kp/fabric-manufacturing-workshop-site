# Troubleshooting

Work in your assigned workshop workspace, not **My workspace**. Do not change tenant settings to make a lab work.

| Symptom | Check or recovery |
|---|---|
| Can't sign in or workspace not listed | Confirm the invited account and tenant. Ask the facilitator to check group membership, guest access, and assigned workspace. Pair with a teammate while waiting. |
| **New item**, **Lakehouse**, or **Upload** is missing | Check capacity state and workspace edit access. Use the facilitator Lakehouse for inspection if creation or upload stays blocked. |
| Lakehouse columns are wrong or the file appears twice | Check **Column header** and comma separator. Do not append a second load. Ask the facilitator to repair a team-owned table or use the prepared copy. |
| Tables are absent in the SQL analytics endpoint | Wait for sync and refresh Explorer. Check that you opened the right endpoint and that Tables, not just Files, exist. |
| Eventhouse or Queryset unavailable | Check that the KQL database is selected and your account can query it. If blocked, use the [machine-event report path](day-1/02-eventhouse.md#if-eventhouse-is-not-enabled). |
| KQL count is not 500 | Verify the typed columns and header. Look for an accidental second ingest or a failed ingestion. Switch to the backup before deleting anything. |
| Cannot edit model or save a report | Check the team model, Editing mode, workspace role, tenant setting, and model Build permissions. Use the read-only reference and record the authoring gap. |
| Risk card or supplier bar differs | Clear cross-highlights, check visual filters, use the correct snapshot, and compare the delivered-only counts. Do not sum shortage over mixed units in another dataset. |
| Data Agent source or item missing | Check capacity, region, tenant flags, and model Read access. Do not change tenant settings. Use the [manual fallback](day-1/03-data-agent.md#if-fabric-data-agent-is-not-enabled). |
| Agent answer sounds right but cannot be verified | Compare filters and measures to the report, record the mismatch, and ask a more precise question in a clean chat. |
| Foundry demonstration fails | Use the Fabric-only version. No participant setup changes are needed. |

Follow [support](../SUPPORT.md) when escalating. Include the step, product, complete redacted error, time and time zone, and a redacted screenshot in a private message.
