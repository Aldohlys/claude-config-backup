---
name: project_tradenr_backfill_stale
description: When a trade leg is entered into Trades AFTER one or more portfolio snapshots, those snapshots keep TradeNr=NULL forever; RReporting unPnL underreports
type: project
originSessionId: 3e9e8622-37ca-48f5-8116-4573ce74aa2f
modified: 2026-09-30T22:54:56.289Z
---
If you add an adjustment/leg to the Trades table after the IBKR portfolio snapshots containing that leg have already been written, the historical portfolio rows (U1804173 etc.) keep `TradeNr=NULL`. `getIBKR` only matches at snapshot time and never rewrites prior rows.

**Why:** RReporting's `summary_open` groups portfolio by TradeNr to compute unPnL/curCost per trade. Unmatched rows fall into the `NA` bucket and silently drop out of the affected trade's summary — the displayed unPnL reflects only the matched legs.

**How to apply:**
- Symptom: trade summary unPnL "looks too good" for a multi-leg trade; in DB the missing leg appears in portfolio with `TradeNr=NULL` despite being present in `Trades`.
- Fix: `scripts/fix_tradenr.bat <portfolio_table> [YYYYMMDD] [--dry-run]` (R script `scripts/fix_tradenr.R`, moved from the gitignored `data/` 2026-10-01). First arg is the TABLE, never a TradeNr (user tried `753`). Date defaults to 60 days back.
- Known case 2026-05-11: NVDA 29MAY26 215 C in U1804173 (TradeNr 718) — fixed 5 rows from 20260507 onward.
- Since 2026-10-01 (RApplication 460cae2) every lookup is restricted to the table's own account, and column names are post-#35 (Symbol/Status) — the old Ssjacent/Statut cash fallback crashed on any unmatched cash row (GBP on U1804173) before options were linked. Cash balances with no CASH trade (GBP on U1804173) stay NULL by design.
- Production DB writes by Claude are blocked by the permission classifier: dry-run it, then hand the user the .bat command.
- After the fix, reload RReporting to pick up corrected unPnL.
