---
name: project-bot-pnl-greeks-decomposition
description: Realized P&L of 116 closed BOT option trades split into spot/vega/theta — spot R² 0.89, vega ~0.03, theta the main cost; losers mostly theta not direction
metadata:
  type: project
---
Measured 2026-10-05 with `NewTrading/Strategies/Breakouts/bot_pnl_greeks.py`. Outputs are `Reports/bot_pnl_greeks_{trades,lots}_20261005.csv`.

Method: FIFO lots per contract. Each fill's vol is implied from its price, using Black-Scholes with no dividends. Each lot's P&L is split exactly by Shapley over spot, vol and time. Spot at the fill comes from Yahoo 60-minute bars where available. Older DB rows carry only a date, so those use the midpoint of open and close.

Results (gross of commission):
- Total: +11.6k = spot +24.8k, vega +4.4k, theta −16.4k.
- R² against realized P&L: spot 0.89, vega 0.03, theta 0.02.
- Winners: spot provides at least half the P&L in 46 of 47. Vega is the largest component in only 4 (AAPL and QCOM puts, MCD, a USO spread).
- Losers (69): theta is the largest component in 35. Only −5.6k of the −12.7k loss comes from spot, i.e. the stock didn't move within the ~14-day median hold.
- The vega figure is inflated by spot-timing noise: +673 on lots with hourly spot at both fills, +3.7k on the date-only lots.
- Median IV change while holding: calls +1.3 pp, puts +2.5 pp. No crush after breakouts, but no edge either.

Caveat: the DB includes trades from strategy LTO and rows marked "Erreur" that belong to BOT trade numbers. The script selects by TradeNr and excludes Erreur.

**How to apply:** BOT selection should predict the stock move. Option choice should minimise theta per unit of expected move, not chase cheap IV. Related: [[project-swing-scanner-retired]], [[feedback-breakout-iv-direction]].
