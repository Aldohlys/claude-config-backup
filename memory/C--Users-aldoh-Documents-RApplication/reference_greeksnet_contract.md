---
name: reference_greeksnet_contract
description: "Tdata::greeksNet output contract (5.12.0): per-group net greeks + deltanotional in TRADE currency, type-based notional; uPrice is option-only; stored-portfolio greek/price field conventions."
metadata: 
  node_type: memory
  type: reference
  originSessionId: e7c37a4a-46bb-4b8f-a2ed-8b13b66f3b38
---

`Tdata::greeksNet(portf)` (R/account.R) is a per-GROUP aggregation primitive — group the input (by symbol / trade / position / datetime) and `summarize()` computes one row per group. Returns columns: `delta, deltanotional, gamma, theta, vega, currency`.

**As of Tdata 5.12.0 (2026-06-26):** `deltanotional` is in the group's own **trade currency** (NOT base), and a `currency = first(currency)` column is carried back. Contract: **each group must be single-currency.** Callers that aggregate across currencies must `convert_to_base_date(deltanotional, currency, date)` per group BEFORE summing. (Before 5.12.0 it converted to base internally — inconsistently — see git history.)

**Notional is chosen by INSTRUMENT TYPE, not by uPrice value:**
- `Stock` → `pos*mktPrice` ; `Future` → `multiplier*pos*mktPrice` ; `Call`/`Put` (incl FOP) → `multiplier*delta*pos*uPrice` ; `CASH` → `mktValue` ; `TreasuryBill` → `mktValue` (bond-like). `CFD` and anything else → `0` (CFD intentionally excluded per user).
- Gonet (no greek columns) takes the first branch → `pos*mktPrice` (== `mktValue` for every Gonet row, incl bond mutual funds typed `Bond`).

**Why uPrice must NOT be used for a future's notional:** `uPrice` = IBKR `undPrice` from option `modelGreeks` (`account.py` maps it; non-options get `greeks_zero`). So `uPrice = 0` for stocks/futures/CFD/T-Bill. A stock/future IS its own underlying → use `mktPrice`. Do NOT add a value-based `effective_uPrice = if_else(uPrice==0, mktPrice, uPrice)` fallback — make it type-based.

**Stored-portfolio field conventions (readPortfolio / IBKR account, non-obvious):**
- Greek `delta`: stocks carry `delta = 1`; futures / CFD / TreasuryBill carry `delta = 0` (options carry real delta). So any `pos*multiplier*delta*…` notional silently ZEROES futures.
- CASH rows: `symbol` = foreign currency, `currency` = **base** (CHF), `mktPrice` = FX rate, `mktValue` = value already in base — so a CASH `deltanotional = mktValue` is self-consistent in its `currency`.

`Tuser/portfolio/logic/portf.R` has its OWN notional pipeline (independent of greeksNet) — it already follows compute-native-then-convert-before-sum and was made type-based the same way (helper `add_notional_cols`). See also [[reference_trades_portfolio_join_key_by_type]], [[reference_adding_currency_two_pipelines]].
