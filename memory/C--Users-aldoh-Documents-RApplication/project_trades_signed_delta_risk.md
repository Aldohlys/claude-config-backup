---
name: project_trades_signed_delta_risk
description: Trades table Risk is a signed delta per row; closure balances sum(Risk) to 0 and stores Return on one close row. BOT resolver since 2026-05, generic resolver for OFI/WHEEL/BPT/CS/Sharpe2/Perso since Phase 5; the 2026-09-15 backfill left 0 broken trades among those with a Close row (56 close-less remain, TODO #68).
metadata: 
  node_type: memory
  type: project
  originSessionId: 4ed7508e-d187-4257-a7e0-5e399d43e3c1
---

After TODO #38 Phase 1-4 (shipped 2026-05-18 for BOT only), the Trades table has a fundamentally different Risk semantics than before. Future work that touches `Risk`, `Return`, or aggregations over trade rows MUST account for this.

**New schema columns:**
- `EventType` (TEXT): one of {Open, Adjust, Close}, immutable per row. Set at insert time. Required because `Statut` is mutated on prior rows when a trade is adjusted/closed, destroying lifecycle signal.
- `Return` (REAL): NULL on every row except the first Close-block row of a closed trade. Written once at closure as `sum(PnL) / MaxOutlay` where MaxOutlay = `max(cumsum(Risk))` over the trade's full row history.

**New Risk semantics (BOT only for now):**
- Per-row `Risk` is a signed delta to live trade risk.
- Open row 1: `+abs(sum(Total_open_block))` for options BOT; user-entered for futures/shares/FOREX.
- Adjust row 1: `+abs(sum(Total_adjust))` for net debit (add or roll wider); `-(closed_qty/open_qty) × current_risk` for partial reduce.
- Close row 1: `-(sum of prior Risk for the TradeNr)` — always auto-computed, never user-entered.
- Companion rows in multi-leg blocks (rows 2+): always 0.
- Invariant: `sum(Risk over TradeNr)` = current live trade risk. Closed trades = 0.
  **Aspirational, not enforced on history:** re-measured 2026-09-03, **256 of 674**
  fully-closed trades have `sum(Risk) != 0` — by strategy: OFI 67/184, WHEEL 31/41,
  BPT 30/66, Sharpe2 26/36, LTO 23/31, Perso 21/62, CS 17/19, CAL 8/13, Gonet 6/15.
  Only `modify_trade` warns (`check_trade_risk_invariant`). Don't treat the
  invariant as a usable precondition when reading legacy trades — MaxOutlay reads
  the running peak, not the sum, so display is unaffected.

  The cause is that **Phase 5 was going-forward only**: only BOT was ever
  backfilled. As of 2026-09-03 that backfill *is* TODO #38 — the item was rescoped
  to exactly this and dropped from HIGH to MEDIUM, its two former residuals having
  been closed by #75 back in June. See [[feedback_verify_before_acting]] §4.

**Resolvers in `RReporting/app/R/compute_functions.R`:**
- `is_options_block(rows)` — Strike/Right primary, Instrument-string regex fallback for legacy NULL-Strike rows
- `compute_risk_delta_BOT(existing_rows, new_rows)` — signed delta resolver, returns NA for non-options
- `compute_close_risk(existing_rows)` — `-sum(prior Risk)`
- `compute_max_outlay(rows)` — `max(cumsum(Risk[order(TradeDate)]))`; delegates to
  `compute_max_outlay_vec(risk, trade_date)`, the vector twin for use inside
  `summarize()`. Both return `NA_real_` if ANY delta is missing (empty rows -> 0).
  Never let a raw `max(cumsum(...), na.rm = TRUE)` back in — see
  [[feedback_max_cumsum_na_rm_minus_inf]].
- `compute_return_at_close(rows)` — `sum(PnL) / compute_max_outlay(rows)`
- `replay_trade_risks_BOT(rows)` — full-history Risk reconstruction; handles legacy rows where Adjusts had Risk=0; balances Close rows regardless of underlying. Used by close_trade and Phase 3 backfill.

**`compute_return()` return contract (Tuser `core/core.R`) — indexing trap:** it
answers with a metrics list (`$return`, `$yield`, `$premium_ratio`, `$time_ratio`)
only when it CAN compute one. Every abstain path — no Risk, no PnL, Risk `NA`,
Risk negative — returns a **bare `NA`**, on which `$` and `[[` both raise
"$ operator is invalid for atomic vectors". Never index the result directly:
use the exported `extract_field(x, field)` (`vapply(metrics, extract_field,
numeric(1), field = "return")`). The abstain is logged at INFO
("Risk is NA" / "Risk is negative, must be positive") — those INFO lines in the
app log are the tell that a crash is one `$` away.

**Writers in `RReporting/app/R/trade_operations.R`:**
- `insert_new_trade` writes EventType="Open" + Return=NA.
- `adjust_trade` writes EventType="Adjust"; for BOT options, overrides user-entered Risk with `compute_risk_delta_BOT()`.
- `close_trade` writes EventType="Close", auto-computes Risk via close-balance, auto-computes Return on row 1.

**Modal split** (`RReporting/app/R/modal_functions.R`):
- `showModalAdjust` keeps Risk/Reward inputs (OK button: `ModalAdjust_ok`).
- `showModalClose` is new — no Risk/Reward inputs since Risk is deterministic at close (OK button: `ModalClose_ok`).
- Legacy `showModalUpdate` aliased to `showModalAdjust` for backward compat.

**Reader side** (`RReporting/app/R/reactives.R`):
- `summary_open`: Risk column = MaxOutlay walk; Return computed on the fly via compute_return with MaxOutlay denominator (= "Virtual Return", what it would be if closed now).
- `summary_closed`: Risk column = MaxOutlay; Return prefers stored DB value, falls back to compute_return for legacy non-BOT closes.

**State (as of 2026-05-18):**
- BOT: 101 closed trades backfilled (`scripts/backfill_risk_return_BOT.R`); all have `sum(Risk)=0` and 99 have Return populated. 3 currently-open BOTs (722 UPS, 723 DD, plus whatever was open today) live under the new writers.
- Non-BOT strategies: unchanged. Risk values still legacy per-row pattern; Return still computed on the fly via compute_return. close_trade for non-BOT trades still auto-balances Risk (universal close behavior, not BOT-specific).
- Phase 5 (per-strategy resolvers for OFI/WHEEL/BPT/CS/Sharpe2/Perso) pending.

**Canonical references:**
- `docs/RISK_REFACTORING_PLAN.md` — full architecture and phase decisions
- `tests/fixtures/risk_BOT/` — 6 worked-example fixtures with expected per-row Risk + Return
- `RReporting/tests/test_risk_resolvers.R` — 50 unit tests
- `scripts/migrate_event_type.R` — Phase 1 schema migration
- `scripts/backfill_risk_return_closed.R` — historical backfill for all strategies (2026-09-15; the Phase 3 BOT script `backfill_risk_return_BOT.R` was never committed)
- `scripts/smoke_test_savetrades.R` — round-trip safety check (see [[reference_savetrades_overwrite]])
- TODO #38, TODO #67 (Stop column dependency), TODO #68 (Trades table cleanup)

## Computing a trade's COST from its legs: net, never gross (2026-09-01)

For any multi-leg trade, cost = **signed** sum of `Total` over opening rows
(`-sum(Total)` for a debit), **never** `sum(abs(Total))`. Summing absolute
values counts both sides of a spread and roughly doubles it.

Cost of getting this wrong, measured on BOT: gross said "median premium 558,
only 39% within the 400 budget, 26% over 1000". The truth is **net debit median
293, 77% within 400, 55% within 300, 8% over 1000** - i.e. sizing discipline
looked broken when it was fine. Individual cases: GLD 423/428 vertical gross
1,634 -> **net 217**; BRK B 520/525 gross 1,168 -> **net 239**; QQQ gross 1,714
-> net 176; SMH gross 2,529 -> net 276.

The error also inverts design conclusions: a proposed universe filter of
"typical ATM option must cost <= 400" would have excluded GLD, BRK B, QQQ and
SMH - the very names a vertical makes affordable. See
[[project_bot_three_class_framework]].

## UPDATE 2026-09-15 — historical invariant restored for trades with a Close row (TODO #38 closed)

`scripts/backfill_risk_return_closed.R` repaired the 199 closed trades (all rows 'Fermé' plus a Close row) whose sum(Risk) != 0. Stored Open/Adjust Risk kept as entered; first row of the LAST Close block = -(sum of the other rows), companions 0; Return = sum(PnL) / MaxOutlay on that row only, NA when MaxOutlay <= 0. Trade 51 kept Return NA (`--skip-return=51`: Open Risk -407 then Adjust +411 gives MaxOutlay 4). One transaction after an online backup, `logs/mydb_pre_backfill_risk_return_20260915_095828.db`.

- Now: 503 trades with a Close row, **0** with sum(Risk) != 0. 197 still have no Return because MaxOutlay <= 0, mostly legacy credit trades (OFI, Gonet) stored with negative Open Risk.
- Still broken: 56 all-'Fermé' trades with NO Close row (20 without a Strategy), moved to TODO #68. Treat the invariant as a precondition only for trades that have a Close row.
- RReporting reads a trade's FIRST non-NA Return across its rows (`reactives.R`, `first(Return[!is.na(Return)])`), so Return must sit on exactly one row.
- `scripts/backfill_risk_return_BOT.R` cited above was never committed and no longer exists; only its CSV logs remain.
- Legacy non-BOT Risk signs are unreliable: check MaxOutlay before trusting a Return derived from pre-2026-06 rows.
