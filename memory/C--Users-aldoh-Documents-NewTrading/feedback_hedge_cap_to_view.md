---
name: feedback_hedge_cap_to_view
description: "Size hedge structures to the user's own view of the maximum move — spreads capped where they expect the move to stop, not outright tails"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 250890ed-cd37-47c6-bd39-228637ce7645
  modified: 2026-10-16T05:28:10.536Z
---

When I proposed outright CME CHF calls for the Gonet CHF-shock hedge, the user replied that the franc "will not dramatically move" and asked for a call spread instead. Chosen: Jun-27 1.25/1.30, capped at USD/CHF −7%.

**Why:** the user pays only for the range of moves they believe in; tail beyond their view is not worth premium. Same logic as the ESTX50 put spread (wing at the 2022 trough).

**How to apply:** when proposing a hedge, first ask or state the plausible max move, then put the short strike there; quote outright only as comparison. Present cost, break-even and full-payoff points as % moves from spot. Related: [[feedback_structure_selection_vs_vol_thesis]], [[project_gonet_crash_simulation_20261016]].
