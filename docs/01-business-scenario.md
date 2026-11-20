# Morning production risk review

You work with three fictional factories: Plant North, Plant South, and Plant East. The shift lead has time to call only a few teams before the next shift. Your task is to identify open production orders with material shortages and recommend what someone should verify next.

Use inventory as of **10 November 2026** and planned starts from **11 to 17 November 2026**, inclusive. Net usable stock is available stock minus reservations, floored at zero. An order is at risk if it needs more than that amount. This is a fixed training example, not a forecast or a shared-stock allocation model.

Look at supplier deliveries, machine faults, and quality separately. Those signals can guide a follow-up call. They do not prove why a particular order has a shortage. All data is invented and machine events are a stored batch, not a live feed.

At the end, give your shift lead one priority, one piece of evidence, and one limitation. Start with [Day 1](day-1/README.md) or inspect the [dataset definitions](../data/validation-summary.md).
