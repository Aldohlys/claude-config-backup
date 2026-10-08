---
name: project_abs_breadth_weights_untested
description: "macro_context ABS_BREADTH fingerprint weights are judgment, untested until enough daily breadth readings exist (stored only since 2026-03-17)"
metadata:
  node_type: memory
  type: project
  originSessionId: 9768c881-e087-47ba-8fe2-7db428bfa064
  modified: 2026-10-07T23:34:30.037Z
---

The absolute-breadth input (ABS_BREADTH = (% of S&P 500 stocks above their 50-day average − 50) / 15, added 2026-10-08 in RStudies `archetypes.R`) has judgment weights: dash for cash −1, bond rout −0.5, stagflation −0.5, reflation +1.5, goldilocks +1, growth scare −1, narrow leadership −1. The 2026-10-08 calibration (`calibrate_scenarios.py`, `Reports/scenario_weight_calibration_20261008.md`) excluded it because breadth readings are stored daily only since 2026-03-17 (`macro_context_results.s5fi`), with no historical episodes to score.

**Why:** the user wants these weights treated as untested until enough daily readings build up to test them.

**How to apply:** when discussing scenario scores or calibration, flag ABS_BREADTH weights as judgment. Re-test when the stored history covers at least one dated scenario episode. The faster alternative is to rebuild historical breadth from constituent prices; that has survivorship bias with the current member list. Related: [[project_intermarket_macro_scenarios]].
