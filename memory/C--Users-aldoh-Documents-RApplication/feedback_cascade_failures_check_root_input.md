---
name: feedback_cascade_failures_check_root_input
description: "A report where nearly every cell reads FETCH FAILED usually means one unresolved input, not a data outage — trace the shared upstream value before blaming the feeds"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f5cc75fc-7625-4c6e-bd01-646cb965032a
  modified: 2026-08-27T02:19:02.313Z
---

When a generated report comes back almost entirely `FETCH FAILED` / `n/a`, the
shape of the failure is the clue: a genuine outage degrades **some** dimensions,
while a bad **input** kills all of them at once. Read the failure *reasons*, not
the count — 48 of 48 cells in `analyze_BRK.B_20260826.html` said "spot price
unavailable", and the single upstream cause was that `BRK.B` never resolved to
`Tickers.Name = "BRK B"`, so `.live_price()` returned `NA` and every downstream
fetch inherited it.

The tell that it is **not** an outage:
- the failing cells all name **the same** missing value ("spot price unavailable")
- the one or two dimensions that don't depend on it still report `LIVE` — but
  hollow (`cheap_score` read `LIVE 0/9` with every component `n/a`)

**Why:** a report that renders a bad ticker identically to a dead data feed
sends you off diagnosing TWS, yfinance and the DB in turn, when nothing is down.
The nearly-total failure is itself evidence *against* an outage.

**How to apply:** before touching any data source, find the one value every
failing cell depends on and test it in isolation for the exact input given
(here: `getYahooName("BRK.B")` → `NA` vs `getYahooName("BRK B")` → `BRK-B`).
Then fix the resolution *and* make the unresolved case **stop with a clear
error** rather than rendering a full report of failures — an unknown ticker
should never be indistinguishable from a dead feed. Also check whether the same
root cause has been silently degrading other rows: the same session's fix
turned out to affect 46 other tickers, not just the one reported
([[reference_ibkr_symbol_with_space]]).
