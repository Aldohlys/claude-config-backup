---
name: reference_macro_report_sparklines
description: "Macro report small graphs: panel sparklines = real Yahoo closes + EMA50; scenario 'Last 60 days' lines = recomputed match score, not a price"
metadata:
  node_type: memory
  type: reference
  originSessionId: 40261db9-cf1a-4c33-b2f5-d91f2eec6c7b
  modified: 2026-10-06T06:41:58.664Z
---

The user asked (2026-10-05) whether the macro_context report's small graphs show real data. They show two different things:

- **Panel sparklines** (intermarket panels, "1 year (EMA50 dashed)"): the solid line is 252 actual daily Yahoo closes and the dashed line is their EMA50 (`series_metrics()$spark` → `intermarket_render.R::sparkline()`). Each chart is min-max scaled on its own. Ratio rows plot the real quotient (e.g. XLE/^GSPC).
- **Scenario score lines** (scenario cards, "All scenarios" table column "Last 60 days"): a derived match score from −100% to +100%, recalculated for each of the last 60 trading days from z-scored price moves (`archetypes.R::scenario_history()` → `score_spark()`). The dashed line is the 40% "in place" threshold. It measures how long the scenario has been in place, not how any price moved.

Related: [[project_macro_bias_from_regimes]].
