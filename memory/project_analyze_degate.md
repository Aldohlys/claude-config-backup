---
name: project_analyze_degate
description: /analyze de-gated to a neutral coverage report (TODO
metadata: 
  node_type: memory
  type: project
  originSessionId: 6521c13e-ba17-4066-985c-d891cb840ed3
---

`/analyze` (RStudies `reports/analyze/`) was de-gated 2026-06-02 (TODO #60). It runs on ONE user-chosen ticker, so the scanner's funnel/verdict framing was wrong and is gone.

- **No more verdicts:** dropped `TOP PICK/WATCH/SKIP`, `phase_of_drop`, and per-phase `PASS/SKIP`. Phase B/C `result` is now a neutral PROVENANCE status; `run_phase_e` is a 6-row **data-coverage summary** (per-dimension provenance), not a classifier.
- **Provenance vocabulary** (the organizing idea): `LIVE` / `CACHED` / `NO DATA` / `FETCH FAILED`. Key distinction (was a lie before): **NO DATA** = request succeeded but response empty/NaN (illiquid strike, off-RTH); **FETCH FAILED** = no response (connect/timeout/exception). Set at the resolver boundary (`.ok`/`.miss`/`.nodata` in live_sources.R), surfaced via `.fmt_cell(x, reason, status=)`.
- **Two axes that were conflated** in each phase's `result`: analytical content (alignment, rank, cheap_score, R:R) vs data provenance. Separating them dissolved both "Phase D unclear" and "Phase E is a verdict".
- Structures: vehicle-preferred table renders open; added stock-only table; `enumerate_structures` placeholder splits FETCH FAILED vs NO DATA.

**Non-obvious:** `reports/shared/live_sources.R` lives in `shared/` but is **/analyze-only** (only `analyze/main.R` sources it). The scanner uses its OWN `swing_scanner/setup_chain_rr.R`. The only truly-shared scanner↔analyze files are `shared/indicators.R`, `universe.R`, `vehicle_rule.R`. So live_sources.R can change freely without scanner back-compat risk.

Builds on [[feedback_analyze_neutral_stance]], [[feedback_analyze_live_data_fallback]], [[project_analyze_tws_gate]].
