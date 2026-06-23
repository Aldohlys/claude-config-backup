---
name: TWS futures term structure chart
description: Path to the graphical term-structure chart for futures contracts in TWS (CL, NG, etc.)
type: reference
originSessionId: 8a62184d-ede9-4981-9f34-2762b2d293d8
---
## Path
In TWS Quote Monitor / watchlist, right-click on any futures contract row (e.g. `CL Jun'26 @NYMEX`) → **Charts** → **Term Structure**.

Opens a window titled "Futures TS - <ROOT> @<EXCHANGE>" with:
- Top panel: price vs Last Trading Day across all listed contract months (curve plot)
- Bottom panel: price difference vs prior reference dates (Today − Yesterday / 1 week ago / 2 weeks ago)
- Toggle reference dates via the "Days:" checkboxes at top (Today / Yesterday / 1 week ago / 2 weeks ago, plus "+" to add more)
- "Last/Close" toggle top-right

## Notes
- This is the **price** term structure (curve shape: contango / backwardation), NOT IV term structure.
- For IV term structure on futures options: same right-click menu → **Implied Volatility Viewer** (sits just above Term Structure in the Charts submenu).
- Works on any futures root that has multiple listed expiries.
