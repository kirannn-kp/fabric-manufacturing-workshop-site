# Proof-of-concept planning

**Goal:** Leave with one scoped experiment and named owners. **Time:** 60 minutes to draft, then a 45-minute separate action review. **Start:** The Day 2 use-case checkpoint and a private team document. Do not copy real data or internal identifiers into the workshop repository.

1. Describe one user, one recurring decision, and what they do today.
2. Choose one plant or process, one time window, and the minimum data needed. Write one item that is out of scope.
3. Sketch the smallest Fabric-first flow: approved source → Lakehouse → semantic model → report. Add Eventhouse only if event latency matters. Add a Data Agent or Foundry only if the team has a specific question-led need and the access path is approved.
4. Choose how to verify the outcome. Example: two shift leads can reproduce the answer on ten known scenarios using the same definitions and filters.
5. Assign owners and book the next checkpoint. If a dependency is not approved, mark it blocked rather than assuming it will be available.

| Field | Team notes |
|---|---|
| Priority use case, user, and decision | |
| Current PoC status | |
| Main blocker and support needed | |
| In scope / out of scope | |
| Architecture and why each item is needed | |
| Required data, grain, freshness, and data owner | |
| Security, residency, access, retention, review owner | |
| Dependencies, capacity, and time needed | |
| Evidence and success criteria | |
| Owner, date, and next action | |

**Check:** The business owner can repeat the decision and success criteria in one sentence, and every next action has an owner and date.

**If information is missing:** Record the assumption, who will resolve it, and by when. Use synthetic data for early discussion until real data is approved.
