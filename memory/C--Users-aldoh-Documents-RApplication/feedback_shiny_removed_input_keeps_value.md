---
name: feedback_shiny_removed_input_keeps_value
description: "Shiny keeps an input's last value after its renderUI control is removed or re-rendered; validate it against current choices"
metadata:
  node_type: memory
  type: feedback
  originSessionId: ce455e19-218b-4fc1-9ac9-77fb8e60d0d6
  modified: 2026-10-29T16:04:24.573Z
---

A `renderUI` selector that returns NULL (or is re-rendered with new choices) does not reset its input: `input$x` keeps the last value the browser sent. If the control disappears, no new value ever arrives.

**Why:** routine app, 2026-10-29 (Tuser 1954b08). Switching account U1804173 → DU5221795 hid the "Choose a trade" selector. `input$tradeNr` stayed "767", so the Trade tab filtered the new account on the old trade, with no way to change it.

**How to apply:** when downstream code reads a dynamic selector, read it through a check against the current choices. In routine this is `tradef$trade_selection(input$tradeNr, trade_choices())`: NULL with no selector, "All" for a value not among the choices. Look for this whenever "switching X leaves Y stuck". Related: [[feedback_reactive_keyed_only_on_symbol_goes_stale]], [[reference_testserver_shiny_wiring]].
