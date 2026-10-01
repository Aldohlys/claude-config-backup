---
name: feedback_scenario_persistence
description: "Macro scenarios must be judged by age across the previous 3-5 reports - new = caution, building = reinforcing, long-standing = may vanish"
metadata:
  node_type: memory
  type: feedback
  originSessionId: d0bd0d7e-3332-405c-afbe-bdb65214a63b
  modified: 2026-10-01T20:09:38.744Z
---

When reading a macro/intermarket scenario, never judge it from one day. Compare with the previous 3-5 macro reports: an entirely new scenario calls for caution; one that reinforces an existing trend is the useful phase; one present for a long time may be near its end.

**Why:** user (2026-10-01): "A scenario takes some time to grow - one day does not make a trend."

**How to apply:** macro_context intermarket section (archetypes.R) recomputes scenario scores for the last 60 trading days and labels NEW / BUILDING / WAVERING / ESTABLISHED / MATURE / FADED (in place = score >= 40%; 15 and 40 trading-day cut-offs). Apply the same age lens in any chat macro read. Related: [[project_intermarket_macro_scenarios]].
