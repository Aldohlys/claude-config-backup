---
name: reference-macro-heatmap-vol-adjusted
description: "macro_context section 09 heatmap colour = move / own usual volatility (heat_z, summary_render.R), not raw %; corn -13.9% dark vs URA -16% medium is correct"
metadata:
  node_type: memory
  type: reference
  originSessionId: dd8f0bea-ff17-47d3-80cd-59333ebf1b0c
  modified: 2026-10-27T11:42:57.898Z
---

The section 09 cross-asset heatmap colours each cell by z = move / (stdev of daily log changes over 120 sessions × √days) of that instrument (`heat_z()`, RStudies `reports/macro_context/summary_render.R`). Bands: light 0.5–1×, medium 1–2×, strong ≥2×, none below 0.5×. The cell text is the raw %.

On 2026-10-27 the user flagged as wrong: corn 1M −13.9% dark red vs uranium miners (URA) 1M −16.0% lighter. Recomputed from Yahoo: corn −2.03× (daily vol 1.6%), wheat −1.46×, URA −1.28× (daily vol 3.0%). The colours were correct; a high-vol name needs a bigger % move for the same shade.

**How to apply:** when a heatmap colour looks inconsistent with the %, recompute z before calling it a bug. If the user wants raw-% colouring or z printed in the cell, change `heat_row()`. Related: [[project-macro-context-cot-section11]].
