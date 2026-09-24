---
name: feedback_import_ui_placement
description: Data-import features (e.g. dividends) belong in the app tab where the data is viewed — Tuser routine Trade tab — AND in RReporting's New Trades tab; the logic stays in a UI-agnostic Tdata function
metadata:
  type: feedback
---

When I proposed wiring the dividend import only into RReporting's Flex upload, the user corrected it: it should be done in Tuser (symbol/trade modules, the Trade tab in Tuser/routine), with RReporting's New Trades tab getting the same button "in the same way". The layout they asked for: an "Import dividends" button directly below the History header, with a small remark "(IBKR Statement CSV)".

**Why:** the user looks at trades and their P&L in the Tuser Trade tab, so an import that changes what that tab shows belongs next to it. RReporting remains the trade-entry app and gets the same control.

**How to apply:**
- Put the logic in Tdata as a no-write plan function plus an apply function (`planIBKRDividends` / `importIBKRDividends`), and add thin UI in both apps: upload → preview modal (new / already booked / no match) → confirm.
- In RReporting, append to the in-memory `all_trades()` and let the user Save. Never write the DB behind it: `saveTrades` overwrites the table.
- Default to a statement/CSV upload the user controls, rather than a background job.
