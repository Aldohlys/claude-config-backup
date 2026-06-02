---
name: Use shinytest2 AppDriver to surface silent client-side bugs
description: When a Shiny app has unresponsive widgets but a clean R terminal, drive it via shinytest2::AppDriver and read app$get_logs() before debugging R code
type: feedback
originSessionId: 29cdc5d5-050d-4ed2-a819-c20c72b37017
---
When a Shiny app misbehaves silently — buttons don't respond, a modal opens
but typing/clicking inside does nothing, and the R terminal shows nothing
unusual — the failure is almost always client-side (JS error, missing asset,
binding crash). The R-side observer never fires, so no `tryCatch`, `message()`,
or `cat()` you add will help.

**Why:** debugged the Tickers CRUD app on 2026-04-28. *Add Ticker* opened the
modal but *Add* did nothing; R terminal was clean (`Listening on
http://...` and silence). I tried three rounds of R-side refactors blind.
What actually surfaced the bug — `Unable to find "selectize-plugin-a11y"
plugin` — was a 30-line `shinytest2::AppDriver` script that read
`app$get_logs()`. The same approach showed `f_Name`, `save_btn` etc. were
absent from `app$get_values()$input` even though they were in the DOM,
proving the binding pass had crashed.

**How to apply:**
- Reach for `shinytest2::AppDriver$new()` *first* when symptoms are silent
  unresponsiveness; don't iterate on R-side guesses.
- Set `Sys.setenv(NOT_CRAN = "true")` before constructing the driver, or
  `AppDriver$new` aborts with `Reason: On CRAN`.
- Useful probes:
  - `app$get_logs()` — JS console + R stderr; client-side errors live here
  - `app$get_values()$input` — what Shiny actually has bound
  - `app$get_html("body")` — DOM at the moment in question
- `shinytest2` + `chromote` are already installed in this project's renv.
- Note that `AppDriver` spawns a *separate* R process; `Sys.setenv` you set
  in the parent process does not propagate to the child. Use `app_dir =`
  pointing at a self-contained app, or set env vars via `setEnv = list(...)`.
