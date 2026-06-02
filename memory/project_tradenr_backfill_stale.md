---
name: TradeNr=NULL in portfolio for late-entered legs
description: When a trade leg is entered into Trades AFTER one or more portfolio snapshots, those snapshots keep TradeNr=NULL forever; RReporting unPnL underreports
type: project
originSessionId: 3e9e8622-37ca-48f5-8116-4573ce74aa2f
---
If you add an adjustment/leg to the Trades table after the IBKR portfolio snapshots containing that leg have already been written, the historical portfolio rows (U1804173 etc.) keep `TradeNr=NULL`. `getIBKR` only matches at snapshot time and never rewrites prior rows.

**Why:** RReporting's `summary_open` groups portfolio by TradeNr to compute unPnL/curCost per trade. Unmatched rows fall into the `NA` bucket and silently drop out of the affected trade's summary — the displayed unPnL reflects only the matched legs.

**How to apply:**
- Symptom: trade summary unPnL "looks too good" for a multi-leg trade; in DB the missing leg appears in portfolio with `TradeNr=NULL` despite being present in `Trades`.
- Fix: `Rscript data/fix_tradenr.R <portfolio_table> <YYYYMMDD>` — backfills NULL TradeNr by Instrument match.
- Known case 2026-05-11: NVDA 29MAY26 215 C in U1804173 (TradeNr 718) — fixed 5 rows from 20260507 onward.
- **Gotcha:** `fix_tradenr.R`'s main lookup does NOT filter by Account. Safe for U1804173 (no cross-account Instrument collisions seen). UNSAFE for DU5221795 cash rows (EUR/USD/HKD) which match Trades in U1804173 by Instrument alone — would create wrong links. Add Account filter to the script before running it on demo accounts.
- After the fix, reload RReporting to pick up corrected unPnL.
