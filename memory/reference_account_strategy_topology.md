---
name: reference_account_strategy_topology
description: "Which accounts hold which strategies, and the per-trade risk sizing base (IBKR sleeve, not total NLV)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 676039b8-1e19-48ce-a135-d4119be5ae8d
---

How the trading capital and strategy tags map across accounts — the basis for any
per-trade / aggregate risk sizing (established 2026-06-16 while tuning `Strategies.MaxRisk`).

**Strategy-tagged `Trades` live ONLY in the IBKR accounts.** Almost everything is in
**U1804173** (the main options/futures/FX account); **VALUE** is the exception — it trades
in **U25343478**. The big **Gonet** book is **stocks-only, sourced from CSV
(`NewTrading/GonetTrades.csv`), and is NOT strategy-tagged / not in the `Trades` table** —
so it never backs the strategy MaxRisk budgets. `DU5221795` is the paper account (ignore for
sizing). "Live" is the virtual sum (U1804173 + U25343478 + Gonet).

**NLV snapshot 2026-06-16 (CHF, drifts — re-query `Account.NetLiquidation`):** Live ~483k =
Gonet ~402k + U1804173 ~58.6k + U25343478 ~22.2k.

**Sizing base decision:** per-trade risk budgets size against the **IBKR sleeve
(U1804173 + U25343478 ≈ 80.7k), NOT total Live NLV (~483k)** — because the strategies only
deploy there and Gonet is a passive stock book. 1% of the sleeve ≈ 800 CHF (the per-trade
ceiling); 1% of U1804173 alone ≈ 586 ≈ the historical 600 default. The one place the *home
sub-account* matters more than the sleeve: **VALUE** stacks ~12 concurrent names in the 22.2k
U25343478, so its aggregate can hit ~38% of that sub-account even while small vs the sleeve.

See [[project_strategies_maxrisk_cap]] (the budgets) and TODO #76 (aggregate/concurrent
overlay, with the measured concurrency baseline).
