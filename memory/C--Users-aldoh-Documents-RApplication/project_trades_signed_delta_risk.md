---
name: project_trades_signed_delta_risk
description: Trades table Risk is now a signed-delta per row; closure auto-balances to 0 and stores Return on the close row. BOT v1 shipped 2026-05-18; OFI/WHEEL/BPT/CS/Sharpe2/Perso still on old per-row scheme until Phase 5.
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

**Resolvers in `RReporting/app/R/compute_functions.R`:**
- `is_options_block(rows)` — Strike/Right primary, Instrument-string regex fallback for legacy NULL-Strike rows
- `compute_risk_delta_BOT(existing_rows, new_rows)` — signed delta resolver, returns NA for non-options
- `compute_close_risk(existing_rows)` — `-sum(prior Risk)`
- `compute_max_outlay(rows)` — `max(cumsum(Risk[order(TradeDate)]))`
- `compute_return_at_close(rows)` — `sum(PnL) / compute_max_outlay(rows)`
- `replay_trade_risks_BOT(rows)` — full-history Risk reconstruction; handles legacy rows where Adjusts had Risk=0; balances Close rows regardless of underlying. Used by close_trade and Phase 3 backfill.

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
- `scripts/backfill_risk_return_BOT.R` — Phase 3 historical backfill
- `scripts/smoke_test_savetrades.R` — round-trip safety check (see [[reference_savetrades_overwrite]])
- TODO #38, TODO #67 (Stop column dependency), TODO #68 (Trades table cleanup)
