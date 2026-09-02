---
name: reference_shiny_tabpanel_suspends_output
description: "An output inside an unopened tabPanel is suspended — free laziness for an expensive TWS/DB fetch, no manual gating needed"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e2ceadf-edaa-4f9c-82d1-d1e2a412227d
  modified: 2026-09-01T08:57:03.937Z
---

Shiny suspends outputs that are not visible (`outputOptions(..., suspendWhenHidden = TRUE)` is the default). An output living in a `tabPanel` the user has not opened therefore **never executes**, and neither does any reactive only it depends on.

So "expensive work must not run at app start" needs no `observeEvent`/gating machinery: put the output in its own tab and depend on a reactive nothing else reads. Opening the tab is what triggers the first fetch; switching an input while the tab is hidden invalidates without recomputing.

Used by the routine app's **Orders** tab (`Tuser/order/view/openordersUI.R`): launching the app costs zero TWS round-trips, and the reactive takes an `input$reload` dependency so the button re-fetches. Worth preferring over a bare `eventReactive(input$go, ...)`, which forces the user to click before seeing anything.

Caveat: this also means a hidden tab shows stale content the instant it is revealed if you cache — and that a headless `shinytest2` drive must `set_inputs(tabs = "...")` before any of that tab's outputs exist in `get_values()`. See [[reference_routine_app_visual_pass]].
