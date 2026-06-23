---
name: feedback_selectize_plugin_a11y
description: Default selectInput crashes Shiny binding pass with 'selectize-plugin-a11y' missing — use selectize=FALSE
type: feedback
originSessionId: 29cdc5d5-050d-4ed2-a819-c20c72b37017
---
When `selectInput()` is rendered (especially inside a `modalDialog`), the
default selectize.js initialization tries to load a plugin called
`selectize-plugin-a11y`. In some Shiny / selectize bundle versions in this
project's renv libraries that plugin isn't shipped, and the JS error

```
[shiny] Error on client while running Shiny app -
Unable to find "selectize-plugin-a11y" plugin
```

is thrown. The error aborts Shiny's input-binding pass, which silently leaves
all other inputs and buttons in the same render unbound. **Symptom**: a modal
opens visually but typing/clicking inside it does nothing — no observers fire,
nothing logs server-side. From the R terminal you'd just see a normal
`Listening on http://...` and then silence; the failure is browser-side only
and only visible in DevTools console.

**Why:** confirmed on the Tickers CRUD app (`Tuser/ticker/app.R`) on
2026-04-28 — clicking *Add Ticker* opened the modal but *Add* did nothing.
A `shinytest2` run surfaced the JS error and showed `f_Name`, `save_btn`,
etc. were missing from `app$get_values()$input` even though they were in the
DOM. Switching every `selectInput()` to `selectize = FALSE` fixed it
immediately.

**How to apply:**
- Default to `selectInput(..., selectize = FALSE)` in this codebase. Plain
  `<select>` is fine for the dropdown sizes we use; we don't need the
  search/typeahead features that selectize provides.
- If you ever need selectize specifically, test the modal in a real browser
  with DevTools open — the failure is invisible from the R side.
- When debugging "modal opens but nothing inside it works", suspect this
  first; run the app under `shinytest2::AppDriver` and check
  `app$get_logs()` for client-side JS errors before chasing R-side bugs.
