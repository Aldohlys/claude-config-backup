---
name: reference_gonet_csv_bookkeeping
description: "How GonetTrades.csv / GonetPos.csv feed getGonet — lot walk, cash-row semantics, the baseline cut-off trap, and the user's recurring sign errors"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9086880f-22ad-4f6d-a57f-6eda3257a22b
  modified: 2026-09-22T07:58:02.563Z
---

Two semicolon-delimited CSVs (dates `DD.MM.YYYY`) in `NewTrading/` feed `getGonet()`:
- GonetTrades.csv — `TradeNr;orig_date;sym_yahoo;sym_ibkr;init_position;init_price;init_cost;currency`
- GonetPos.csv — `sym_yahoo;sym_ibkr;position;exchange;type`

Canonical spec lives in `RApplication/Tdata/R/account.R` (~lines 900-1200): `gonet_lots`,
`gonet_cash_events`, `gonet_cash_balances`. Read it before editing the CSVs — it carries long doc
comments. Verify against the code, it has been rewritten at least once.

Stock legs. Average-cost lot walk, sorted by date inside `gonet_lots`: buy adds shares and basis,
sell relieves the closed fraction and banks the difference as realized. `init_position` IS used —
an older note calling it documentary was wrong. BUY = positive position, negative init_cost (cash
out). SELL = negative position, positive init_cost (cash in). `init_cost` is the actual net
cashflow, so it embeds commission. Sales reuse the original buy's TradeNr.

Cash rows. A row where `sym_ibkr == currency` is a cash ledger row, and its TradeNr decides
everything:
- TradeNr matching a stock leg -> attributed cash event (dividend, coupon, compensation).
  `init_position` = cash balance delta, `init_cost` = amount attributed as P&L. Cross-currency is
  fine; `gonet_lots` converts to the position currency.
- TradeNr matching no stock leg -> baseline row (26 CHF / 27 USD / 28 EUR at 10.07.2026), carrying
  the bank balance at that date with `init_cost = -init_position`.

```
balance(ccy) = sum(init_position) over ALL cash rows in ccy        # no date filter
             + sum(init_cost) over stock legs in ccy dated > cut_off
cut_off      = min(date) over baseline rows
```

Two traps, both hit in the 2026-09-22 session:

1. Never add an unattached cash row with an early date. It is read as a baseline, moves `cut_off`
   back, and every stock leg since then gets double-counted into the balance. This is why six
   SARASELECT credits stayed out — the fund has no leg to attach to.
2. A dividend dated before the baseline is already inside that bank figure. Give it
   `init_position = 0` and `init_cost = amount`: no balance delta, full P&L attribution. Loading
   five years of history as balance deltas instead forces the baselines negative, which is the
   model rejecting the method, not a number to accept.

Free-share attributions (Air Liquide loyalty ~1:10): zero cash cost, so bump GonetPos and add a
zero-cost line `TradeNr;date;sym;ibkr;<n>;0;0;CCY`. avgCost drops as the free shares dilute basis.

Verify the user's manual sell entries — recurring SIGN ERRORS. Twice in one session: ABBN sale cost
sign wrong (negative not positive), FXC sale position sign wrong (+177 not -177). The 27.09.2024 LVMH sale was booked `16;...;+28;...;+17021` (a buy);
fixed 2026-09-23 to `2;27.09.2024;MC.PA;MC;-28;607.89;17021;EUR`, which the user confirmed closed.

Leg history for the Tuser Trade tab: `Tdata::getGonetLegs()` / `getGonetTradeDates()` (5.20.5).
Symbols are keyed through sym_yahoo to GonetPos names (433080107 in trades = IE00B67T5G21 in pos).

Gonet account: Swiss private bank, CHF base, holds the equity book (ABBN/HOLN/RO/SLHN/AMRZ/AI/TTE/
OR/GTT + ETFs), buys UCITS ETFs. Hefty per-order commission — prefer one order over tranches.
Dividend statements: [[reference_gonet_movements_statement]]. Related:
[[project_savetrades_overwrite_bug]], [[reference_trades_right_column_sparse]],
[[feedback_build_payload_before_open_w]].
