# /analyze — single-ticker data report (R-script wrapper)

Thin wrapper. The actual computation lives in the RStudies repo at
`reports/analyze/` and is invoked as a batch process — same pattern as
`/macro_context` and `/swing_scanner`.

**INSTRUCTIONS FOR CLAUDE:**

1. Parse `$ARGUMENTS` as `TICKER DIRECTION [flags]`.
   - `TICKER` (required): symbol, e.g. `UPS`, `JPM`, `SMH`.
   - `DIRECTION` (required): `long` or `short`.
   - Flags (optional): `--no-html`, `--no-vol-funnel`.
2. Run the R pipeline. The exact invocation depends on host:

   **Windows host** (laptop):
   ```bash
   "C:/Program Files/R/R-4.4.3/bin/Rscript.exe" \
     C:/Users/aldoh/Documents/RApplication/RStudies/reports/analyze/main.R \
     <TICKER> <DIRECTION> [flags]
   ```
   (CWD must be `C:/Users/aldoh/Documents/RApplication/RStudies` so renv activates and Tdata is found.)

   **Linux VM host** (`trading-vm`): call the entrypoint wrapper that pre-checks
   IB Gateway before Rscript. Do not invoke Rscript directly on the VM — the
   wrapper reads `production.ibkr.api_port` from `/home/aldohlys/config.yml`,
   verifies the socket is listening, and gives a loud failure if the trading
   stack is down (otherwise Phase D silently produces chain_state=unknown).
   ```bash
   /opt/scripts/analyze.sh <TICKER> <DIRECTION> [flags]
   ```
   Source-of-truth lives at `C:/Users/aldoh/Documents/RApplication/scripts/analyze.sh`;
   deploy with `scripts/push-analyze-to-vm.ps1`. Wrapper exit codes:
   `0`=ok, `1`=usage, `2`=gateway-down, `3`=R-failure.
3. Read the generated HTML file at
   `C:/Users/aldoh/Documents/NewTrading/reports/analyze_<TICKER>_<YYYYMMDD>.html`
   and surface its key tables in the chat:
   - Phase A/B/C/D/E results (PASS / SKIP / NO SIGNAL / STALE).
   - Phase C funnel grid + tally.
   - Phase D structural targets, chain, R:R.
   - Structures-within-cap data table.
4. Open the HTML automatically when interactive.
5. Answer follow-up questions about the data conversationally.

## What this command IS NOT

- **Not a recommender.** No GO / NO-GO / CONDITIONAL verdicts.
- **Not a ranker.** No "★ Best / ★ Alternative / ★ Aggressive / ✗ Avoid" structures.
- **Not a conviction estimator.** No HIGH / MEDIUM / LOW labels, no edge-source classification.
- **Not a planner.** No pre-committed scale-out / stop / time-stop suggestions.

The user reads the data and decides. This rule is enforced in `reports/analyze/report.R` — not in this prompt — so editing this stub cannot re-introduce verdicts.

## Configuration

Constants live in `RStudies/config.yml` under `default.analyze`:
- `risk_cap_lot_usd` (per-lot risk cap, default $300)
- `rr_min` (calibrated R:R floor, default 0.5)
- IVP / VRP / term-structure scoring thresholds and band labels
- `spread_widths`, `moneyness_pct` for spread enumeration
- `earnings_window_days`, `skew_lookback_days`
- `out_dir` (HTML output directory)

Edit the YAML to recalibrate; no code change required.

## When the R script can't run

If TWS is unreachable, `compute_spread_risk_reward` returns NULL and the structures table falls back to placeholder rows. v5 CSV row + DB snapshots are still consumed. If neither v5 CSV nor DB has data, Phase A/B/C return STALE and the user sees that in the report. Do not synthesise verdicts to fill the gap.

On the VM, gateway-down is rejected up-front by `analyze.sh` (exit 2). If you see that exit code, the fix is `sudo systemctl start trading-stack.service` — re-running /analyze without it produces a stale Phase D, not a valid report.
