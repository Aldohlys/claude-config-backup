---
name: reference_testserver_shiny_wiring
description: "shiny::testServer drives box modules and whole apps without shinytest2 — the bootstrap needed for Tdata, and three harness traps (flushReact, update*Input never reaching input$, namespaced module ids) that produce false FAILs"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 81e8468e-53c1-4b02-9a9c-9c1f7f2651b6
  modified: 2026-09-03T18:51:05.702Z
---

Used 2026-09-03 to verify the RPreTrade Breakout price fix. `shinytest2` is
**not** in RPreTrade's renv — each app has its own library, so
[[feedback_shinytest2_for_silent_bugs]]'s "already installed in this project's
renv" is per-app, not global. `shiny::testServer` needs nothing extra and is
the right tool for *reactive wiring* questions (it cannot see the browser, so
client-side bugs still need AppDriver).

## Bootstrap (this project)

```r
setwd(".../RPreTrade")                       # config.yml lives here
suppressMessages(library(shiny)); suppressMessages(library(Tdata))
invisible(try(Tdata::tdata_py, silent = TRUE))   # touch the lazy binding FIRST
box::use(scan/view/breakoutUI)                   # R_BOX_PATH-relative
```

- Without the `tdata_py` touch, `box::use` dies with
  `ModuleNotFoundError: No module named 'tdata_py'` — cf.
  [[project_tdata_py_lazy_init_startup]].
- `box::use(../Tuser/scan/view/x)` resolves relative to the **script file**,
  not `setwd()`. From a scratchpad script use the `R_BOX_PATH`-relative form.

## Both granularities work

```r
testServer(breakoutUI$server,                       # module: id auto-supplied
           args = list(symbol = sy, current_price = px), { ... })
testServer(".",  { ... })                            # whole app: ui/server/global.R
```

## Three traps that fail correct code

1. **Writing an external `reactiveVal` does not recompute outputs.**
   `session$setInputs()` flushes; a bare `px(330.5)` does not. Follow every
   such write with `session$flushReact()`. Four of my ten module assertions
   "failed" until I added it.
2. **`update*Input()` NEVER reaches `input$` in a MockShinySession.** Proved
   in isolation: an observer calling `updateNumericInput(session,"x",42)`
   leaves `input$x` NULL after a flush. So an auto-populated field is
   *unassertable* here — set it with `setInputs()` and assert what depends on
   it instead. There is also no accessor for sent update messages
   (`getAllInputMessages` does not exist; methods are `setInputs`, `getOutput`,
   `elapse`, `flushReact`, `getReturned`, …).
3. **Module inputs need the namespaced id**:
   `session$setInputs("breakout-price_target" = 450)`. A bare name sets the
   top-level input and the module silently sees NULL.

`session$getOutput("mod-outputid")` asserts *across* a module boundary — that
is what proved sidebar → module here. `session$elapse(3500)` advances the mock
clock past a `debounce()`.

## UPDATE 2026-09-15 — whole-app testServer for RReporting; debounce; silent req() looks like an error

- **RReporting layout:** the app is `RReporting/app` (`global.R`, `ui.R`, `server.R`, launched by `runApp("app")`). Recipe: run Rscript from `RReporting` (its renv), `setwd("app")`, `source()` the three files into globalenv, then `as.character(ui)` for UI checks and `testServer(server, {...})` for the server. Example: `RReporting/tests/test_date_range.R`.
- **Debounced reactives** (`window_date`, `end_date` use `debounce(..., 2000)`): after `session$setInputs(...)` call `session$elapse(3000)` before reading them.
- **Set the whole sidebar.** `summary_open()` / `summary_closed()` `req(input$r_factor)`; left unset they stop with a `shiny.silent.error`, which the `renderDT` `tryCatch` logs as an **empty** `Error in opentrades output:` line. That looks like a regression and is a harness gap. Classify with `tryCatch(expr, shiny.silent.error = ..., error = ...)` before investigating further.
- **Module return values:** `session$getReturned()`. A module fed an external `reactiveVal`: change it, then `session$flushReact()`.
