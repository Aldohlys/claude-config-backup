---
name: project_vol_atr_band_guardrails_todo
description: TODO — add reliability guardrails to the Tuser/vol ATR empirical expected-move band
metadata: 
  node_type: memory
  type: project
  originSessionId: f74ba752-af98-461b-a9dd-da6fba63cfa6
---

TODO (created 2026-06-09, not started). Constrain the Tuser/vol "Empirical (ATR)" expected-move row so it can't be run in regimes its statistics don't support. Rationale + full math in [[reference_atr_empirical_band_limits]].

**Layer 1 — app-layer input clamps — DONE 2026-06-09 (`Tuser/vol/view/xmoveUI.R`):**
- Horizon `numericInput` (line ~28): `min 3, max 120` → `min 5, max 20`. Caps √N-validity envelope; keeps effective sample healthy.
- Confidence `numericInput` (line ~32): `max 99` → `max 90`. The 99% edge reads a ~0.5th pctile off ~200 effective obs = noise.
- Uncommitted in RApplication repo; restart vol Shiny app to take effect.

**Layer 2 — adaptive flag (better; `Tdata/R/atr_move.R` flags block in `atr_expected_move_from_dist`; needs package rebuild + R restart per [[feedback_tdata_rebuild_restart_r]]):**
- Fix existing `if (dist$n_obs < 250)` flag to use EFFECTIVE n = `n_obs / horizon_sessions` (overlapping windows), not raw n. Today it reads ~1986 for J and stays silent when independent count is ~200.
- Add tail-support guard: `tail_count = ((1-conf)/2) * (n_obs/horizon_sessions)`; flag when `tail_count < ~5` → "CI/horizon not supported by this symbol's history." Data-driven, adapts per symbol vs the blunt fixed clamps.

Recommendation: do Layer 1 now, batch Layer 2 into the next Tdata rebuild. Related: [[project_vol_module_refactor]].
