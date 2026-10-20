---
name: reference_gonet_tradenr_reused
description: Gonet TradeNr is NOT a stable per-instrument key — reused across instruments; key per-TradeNr history on symbol instead
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8392b3e4-5d8b-457c-9eab-5c60ef858317
  modified: 2026-10-20T14:58:12.168Z
---

Gonet assigns TradeNr from `GonetTrades.csv` per `sym_yahoo`. A retired holding's number gets **reassigned to a different instrument**, so the same TradeNr spans unrelated instruments over time:
- TradeNr 21 was `IE00B67T5G21` for months, then reused for CNYA (2026-07).
- TradeNr 24 is the gold position `PM_15606539` (since 2021).

Consequences:
- Joining per-TradeNr across snapshots pulls an **unrelated instrument's** data. `compute_weekly_unpnl` now takes `key_col`: **symbol for Gonet, TradeNr for IBKR** (portf.R). Any per-TradeNr history/aggregation over Gonet must key on `symbol`.
- Two live instruments sharing one TradeNr make the Trade-tab selector ambiguous (returns both). Give each Gonet instrument a **unique** TradeNr in `GonetTrades.csv` (next free was 25→CNYA, 26/27/28→cash).
- IBKR TradeNr **is** stable (real Trades table) and does not get reused — but it is **not unique across accounts**. Gonet numbers from 1 and IBKR numbers independently (659..746 today), so under the virtual **Live** account, where `portfDateTime()` binds U1804173 + U25343478 + Gonet into one frame, a bare TradeNr can select two unrelated trades at once. Any filter over a multi-account frame must key on **account + TradeNr**, not TradeNr alone. `Tuser/routine/logic/tradef.R` does this: selector values are bare (`"6"`) for a single account and account-qualified (`"Gonet:6"`, `"U1804173:706"`) when the snapshot spans several, with `trade_key_parse()` / `trade_key_nr()` / `filter_trade()` as the only way to read one. The choices are grouped into one optgroup per account so the two schemes are never shown interleaved.

Bit the routine WeeklyunPnL column 2026-07-10. See [[reference_gonet_pnl_and_stats]], [[project_gonet_cash_positions]].
