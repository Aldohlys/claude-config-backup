---
name: reference-legless-open-trade-in-summary
description: "A Trades row with Status='Ouvert' but no legs (no Instrument/Pos/Price/date) shows only in the RReporting Summary tab; clean by deleting the row after an INSERT backup"
metadata:
  node_type: memory
  type: reference
  originSessionId: 468c5bfa-a3c8-4e4c-89cb-5deb0579e9bd
  modified: 2026-10-22T06:57:16.015Z
---

A note-only `Trades` row (Instrument, Symbol, Pos, Price, TradeDate all NULL; Risk/Reward/PnL 0; `Status='Ouvert'`) is listed by the RReporting Summary tab — it shows every open TradeNr — but by no other tab, because the others are built from legs/positions. Example: trade 572 (DU5221795 paper, BPT, a WITT note on SOFR3), deleted 2026-10-21.

**How to diagnose:** scan every table with a `TradeNr` column for the number; if only `Trades` has it and the row has no Instrument, it is a leftover placeholder.

**How to clean:** check `nrow == 1`, the account and `is.na(Instrument)`, write the row out as an `INSERT` statement for restore, then `DELETE FROM Trades WHERE TradeNr = N AND Account = '...'`. Ask the user first, both because their rules say to and because the auto-mode classifier blocks an unapproved DELETE. The Notes text may be worth keeping, so mention it before deleting.

Related: [[reference-untagged-cash-hidden-in-summary]], [[feedback-open-positions-from-portfolio-not-trade-status]].
