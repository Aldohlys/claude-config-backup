---
name: reference_ibkr_tsej_bad_mark_price
description: "Without TSE data permission, IBKR's mark price for TSEJ stocks can be far off the real close; TWS market value and both P&L columns use the mark, not LAST, and portfolio snapshots store it"
metadata:
  node_type: memory
  type: reference
  originSessionId: e40a1acd-c1c5-473a-88ac-f0f85358d74d
  modified: 2026-10-07T17:56:27.743Z
---

The account has **no Tokyo (TSEJ) market-data permission**. The IBKR connector's `get_price_snapshot` returns `top-status: REJECT`, and `get_price_history` returns "No market data permissions". IBKR still publishes a mark price (`plprice`), and that mark can be wrong.

**Observed 2026-10-07, after about 18:00 CEST, with TSE closed:** IBKR's mark for 5658 was 325, while the real close was 403 and the day's low was 402. Other U25343478 positions were also too low: 9845 by about 22%, 5982 by 6%, 6797 by 8% and 7399 by 6%. In TWS, the LAST column showed the correct close (`c403.0`, where the `c` prefix marks a closing price). **MARKET VALUE, DLY P&L and UNRLZD P&L are computed from the mark, not from LAST.** As a result, 5658 showed −135 CHF unrealized instead of about +275, even though LAST was above AVG PRICE.

**How to detect it:** compute the implied price as mktValue(CHF) × CHFJPY ÷ pos and compare it with LAST. To get CHFJPY, use a row whose daily P&L is 0, where mark equals LAST; on 2026-10-07 that gave about 190. A mismatch beyond rounding means the mark is bad.

**How to verify the real price:** use yfinance `Ticker("5658.T").history(period="1d", interval="5m")`. The daily-interval `Close` for today comes back NaN, while the intraday bars hold today's prices.

**Risk to DB:** the account portfolio tables (e.g. `U25343478`: `mktPrice`, `mktValue`, `unPnL`) store whatever IBKR reports at snapshot time. A snapshot taken while the mark is bad stores wrong values, and those flow into TWR and the strategy P&L. On 2026-10-07 the 17:56:02 snapshot was clean (5658 mktPrice 403.16), so the mark went bad after it. If a bad snapshot gets written, delete that `date`/`heure` row set. See [[reference_strategy_value_pitfalls]].

Related: [[reference_ibkr_data_entitlements]], [[reference_stock_quote_frozen_vs_live_and_cost]], [[reference_position_snapshot_uprice_unadjusted]].
