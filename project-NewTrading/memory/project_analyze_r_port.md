---
name: /analyze ported to R script (2026-04-30)
description: /analyze split — RStudies/reports/analyze/ R pipeline + thin slash-command wrapper. Layout, defaults, known gaps, and where the slash-command stub lives.
type: project
originSessionId: 75f9ac77-f927-4c30-a86a-44aad89326ce
---
/analyze was ported from a ~330-line slash-command spec to an R script. Mechanical computation lives in code; the slash-command shells out and surfaces the HTML in chat. No verdicts, no rankings, no conviction — enforced in `report.R`, not in the prompt.

**Why:** the slash-command version produced inconsistent verdicts despite `feedback_analyze_neutral_stance.md` forbidding them. Code can't drift the way a prompt can.

**How to apply:** `/analyze TICKER long|short` shells out to `Rscript RStudies/reports/analyze/main.R`. The slash-command stub is a 30-line wrapper, not the spec. To recalibrate thresholds, edit `defaults.R` (or override via `config.yml`); never re-introduce GO/NO-GO into the slash-command file.

## Where it lives

- **R pipeline** (`Aldohlys/RStudies` repo, branch `main`): `reports/analyze/{main,phases,funnel,structures,report,defaults}.R`. First commit: `c8054d8` (2026-04-30).
- **Built-in defaults** in `defaults.R`: `risk_cap_lot_usd: 300`, `rr_min: 0.5`, IVP/VRP/term thresholds, spread widths `[5, 10]`, moneyness `0.20`, earnings window 14d. Script runs without any local `config.yml`.
- **Optional override:** `RStudies/config.yml` is local-only (not tracked, may contain secrets). If a `default.analyze` block exists there, keys deep-merge over the defaults — set only what you want to override.
- **Batch wrapper** (`Aldohlys/RApplication` repo, branch `master`): `scripts/run_analyze.bat`. First commit: `4ae3e43` (2026-04-30). Auto-discovered by `scripts/launcher.py` (called from Desktop `RTrading.bat`); the `%~1`/`%~2` args trigger the launcher's arg-entry box.
- **Slash-command stub** (`NewTrading/.claude/commands/analyze.md`): 30-line wrapper that calls `Rscript`. **Gitignored** — `.claude/` is line 9 of NewTrading's `.gitignore`. Open TODO `project_track_claude_commands_todo.md` (2026-04-27) tracks lifting that rule and adding a NewTrading remote.

## Known gaps (deferred — fix when actually needed)

1. **Phase C is NA for v5 Phase-B failures.** v5 CSV only populates cheap_score/vehicle/strike for rows that pass A+B. Tickers like UPS that fail B come back with NA across C/D. /analyze surfaces NA truthfully. Fix when wanted: source `swing_scanner/cheap_score.R` and re-run on the single ticker.
2. **Live OI walk + spread enumeration need TWS reachable.** `compute_spread_risk_reward` (via reticulate) returns NULL when TWS is down; structures table falls back to placeholder rows. Real-data path is wired but untested without TWS up.
3. **Funnel signals show "unavailable" when `Prices` / `option_skew_history` lack rows for the ticker.** Pipeline-coverage issue, not a bug — funnel correctly tallies what's there. UPS had zero `Prices` rows on first test; JPM had some.
4. **`getLastSymPrice` returns a tibble** with `date, sym, value` — the price scraper picks the last numeric column to avoid pulling the date integer. Documented in `phases.R::.live_price`.

## Validated on first runs (2026-04-30)

- UPS short → A PASS · B SKIP (pull_score 0, neutral, sector_rs_rank 99) · C SKIP NA · funnel 1/0/5 · D NO SIGNAL · E SKIP, phase_of_drop=B.
- JPM long → A PASS · B SKIP (pull_score 1, up, ALIGNED) · C SKIP NA · funnel 2/1/3 · D SKIP · E SKIP, phase_of_drop=B.
