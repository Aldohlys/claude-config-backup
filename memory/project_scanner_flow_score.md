---
name: project_scanner_flow_score
description: "Swing scanner Phase B \"Flow_Score\" — what it is, the gates, and the 2026-06-02 fixes"
metadata: 
  node_type: memory
  type: project
  originSessionId: 6521c13e-ba17-4066-985c-d891cb840ed3
---

**Flow_Score** (swing_scanner, Phase B; renamed from `Pull_Score` 2026-06-02) is a flow/CONTEXT composite, NOT the raw technical-breakout score. File: `RStudies/reports/swing_scanner/flow_score.R` (`score_flow()`, fields `flow_score`/`flow_direction`).

- `flow_score = stage_pts + sector_pts + footprint_pts`, max ~10.
  - **stage_pts**: early=4, continuation=3, extended=2, none=0. Stage from `score_breakout()` counts + `Tdata::isTrendContinuation()` (Yahoo data, full Stage-2 trend).
  - **sector_pts**: top-3 sector +3, ≤6 +2, else 0; non-trending sector (both gates closed) = 0 (was −2, see below).
  - **footprint_pts**: OBV / up-down-vol / 3m-RS alignment (0–3).
- The RAW technical synthesis is separate: `score_breakout()` → `bot_setup` (S1–S6) / `bot_breakout` (BK1–BK4). Don't confuse the two — that overlap was the original "Pull" naming confusion.
- Pass: `(flow_score>=6 AND stage∈{early,continuation}) OR (extended AND flow_score>=8)`.

**2026-06-02 Phase-B fixes (after a 0-candidate run):**
- **extended stage_pts 1→2**: the `extended && flow_score>=8` escape was DEAD (extended max was 1+3+3=7). Now ceiling=8, so an extended leader escapes only with top-3 sector + full 3/3 footprint.
- **sector closed-gate −2→0**: the −2 was a hard cap that kept any non-top-sector stock below 6 regardless of merit; now top-sector is a bonus, not a gate.
- `SCANNER_SCHEMA_VERSION` 6→7 marks the scoring change (columns unchanged).
- `main.R` now logs a funnel breakdown (Universe→A→in-sector→stage→flow≥6→B→C→drop counts).

Sector membership (the gate/RS/sector_pts input) is still GICS/ad-hoc labels — TODO #71 wants to replace it with return-correlation clustering. Phase-B passers still face Phase C (cheap IV) + D (R:R). See also [[project_asymmetric_trade_thesis]], [[project_scanner_universe]].
