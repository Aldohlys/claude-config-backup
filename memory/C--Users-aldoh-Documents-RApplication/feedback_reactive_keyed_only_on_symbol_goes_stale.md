---
name: feedback_reactive_keyed_only_on_symbol_goes_stale
description: "A reactive whose only dependency is the symbol fetches once and freezes for the session — check the computation, not just the label, and give time-varying values a refresh trigger"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 81e8468e-53c1-4b02-9a9c-9c1f7f2651b6
  modified: 2026-09-03T18:51:18.300Z
---

`Tuser/scan/view/breakoutUI.R` showed a stale spot price on the RPreTrade
Breakout tab (found 2026-09-03). The cause was not the data layer —
`getStockPrice()` was returning live quotes — but this shape:

```r
current_price <- reactive({ sym <- symbol(); ... getStockPrice(sym) ... })
```

One dependency. It evaluated the first time the tab was opened for a symbol and
was then **frozen for the rest of the session**.

**Why:** Shiny reactives cache until a dependency invalidates. "Fetch it inside
a reactive" reads like "fetch it when needed" but means "fetch once per
dependency change". A symbol is not a clock.

**How to apply:**

- A time-varying value (a quote, a snapshot, anything a market moves) needs a
  refresh trigger the user or the clock can pull: a host-owned input that gets
  refreshed, an `actionButton`, or `invalidateLater`. Symbol alone is not one.
- **Check every consumer, not just the display.** Here `eventReactive` for the
  Calculate button also called `current_price()`, so the R/R table, direction,
  breakevens *and* the OTM strike window were all computed off the frozen spot.
  A stale label is cosmetic; a stale value inside an `eventReactive` corrupts
  results silently. Grep the reactive's name before concluding scope.
- The clean fix is usually to **inject** the value: add a module parameter
  defaulting to `NULL` (old self-fetch preserved → backward compatible) and let
  the host pass its reactive. That refreshes it *and* deletes a duplicate fetch.
- Watch for the sibling bug in the same wiring: a module handed the **raw**
  `input$symbol` instead of the app's debounced `sym` fires one fetch per
  keystroke, and prefixes like `A`/`AA` are real tickers whose prices flash up.

Verify the fix with [[reference_testserver_shiny_wiring]] — assert that the
display *follows* a price change, which is exactly what was broken.
Related: [[reference_shiny_tabpanel_suspends_output]], [[feedback_no_shiny_priority]].
