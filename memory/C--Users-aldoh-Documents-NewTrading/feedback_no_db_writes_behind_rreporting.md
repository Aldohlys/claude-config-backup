---
name: feedback-no-db-writes-behind-rreporting
description: A direct SQL write to Trades is lost when an RReporting session loaded earlier presses Save (whole-table overwrite); have the user Load first or make the change in the app
metadata:
  node_type: memory
  type: feedback
  originSessionId: e87e0ecc-b1ff-410a-a8f3-4449c3418f84
  modified: 2026-10-08T08:26:47.182Z
---

RReporting keeps Trades in memory and `saveTrades` overwrites the whole table. The Tdata 5.20.8 guard only checks that DB dividend rows are present in what is being saved; it misses any other edit made behind the app.

Case 2026-10-08: I set trade 758 Risk from 585.99 to 85.38 by SQL at about 09:50. At 10:06 the user saved from an RReporting session loaded before that (to book the FCX sale), and 758 went back to 585.99. Edits made before that session's Load (rows 2471–2473 Risk/Stop) survived.

**Why:** the user relaunched the app, saw the old value, and had to report it a second time.

**How to apply:** before writing Trades by SQL, ask whether RReporting is open with unsaved work. After writing, tell the user to click Load before any other change. For edits the user can make in the app (Risk on one row), prefer having them do it there. Built in Tdata 5.20.14: getAllTrades() remembers the table, and saveTrades() refuses the save (naming the TradeNrs) when the DB differs from it, unless force = TRUE. A SQL write behind RReporting is no longer reverted; instead the user's Save is refused and they must Load and redo their edits. So still warn before writing.

Related: [[reference_ibkr_dividend_import]], [[project_savetrades_overwrite_bug]].
