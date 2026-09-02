---
name: reference-jheqx-collar-structure
description: JPMorgan Hedged Equity (JHEQX) quarterly collar rule — put strikes fixed at -5%/-20%, call solved for zero cost; how to replicate on SPY with 100 shares
metadata:
  type: reference
---

JPMorgan Hedged Equity Fund (JHEQX / JHQAX, "the JPM collar") rolls one SPX collar each quarter, all legs sharing the quarter-end expiry. Reference spot = index level on the roll day (last business day of the quarter), not any other date.

Rule — only the call strike is an output:
- Long put: 0.95 x spot (-5%) — where protection starts
- Short put: 0.80 x spot (-20%) — where protection ends, funds the long put
- Short call: strike solved so the package is zero net cost; historically lands +3.5% to +5.5% OTM

Result: hedged between -5% and -20%, unhedged below -20% (put spread is capped at its 15-point-of-spot width), upside capped at ~+4-5% per quarter. Sister funds (JHQDX, JHQTX) run the same rule on staggered months.

Replication on 100 SPY shares — 1 lot of each leg, short call covered by shares, short put covered by long put, so no naked leg. Worked example 2026-08-31, SPY 765.39, Dec 31 2026 expiry (122 DTE): buy 725P (-5.28%, \$11.94) / sell 610P (-20.30%, \$3.09) / sell 810C (+5.83%, \$8.48) = \$0.37 debit. Exact zero-cost call solved to ~808.8 (+5.7%) — wide end of the historical range because SPY vol was low (call IV 12.3% vs put IV 17.5%); the put skew is what makes the structure finance itself.

Two SPY-vs-SPX differences: SPY is American-style, so a deep-ITM short call can be assigned around the December ex-div date (SPX is European, cash-settled, 60/40); and SPY strike grid is $1 near money but $5 in the wings, so -5% and -20% usually round to the nearest available strike.

See [[feedback_hedge_wing_sale_timing]], [[project_estx50_hedge_roll_20260601]].
