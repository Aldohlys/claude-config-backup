---
name: feedback_no_shiny_priority
description: Avoid observeEvent/observe priority argument — ordering is buggy and unreliable; restructure reactivity instead
type: feedback
originSessionId: 29cdc5d5-050d-4ed2-a819-c20c72b37017
---
Never use `priority =` on `observeEvent()` / `observe()` to control execution
order between reactive handlers in Shiny.

**Why:** Priority-based ordering in Shiny's reactive scheduler is notoriously
unreliable — same-cycle observers can fire in unexpected orders, especially
when `reactiveValues` are read by both observers, and the bugs are hard to
reproduce. We hit it on the Tickers CRUD app: a `priority=10` observer that
set `rv$editing_orig <- NULL` before the modal-opening observer ran was
producing intermittently wrong save-mode (Add vs Edit), causing the Add
button to silently no-op.

**How to apply:**
- One observer per UI trigger. Put all the state writes that must precede the
  side-effect (modal show, navigation, etc.) into the same observer body, in
  source order — that ordering is guaranteed.
- If two pieces of work genuinely need to be decoupled, link them via an
  intermediate `reactiveVal` and `observeEvent` on that val — the dependency
  graph determines order, not `priority`.
- For "do this last" effects, use `session$onFlushed(once = TRUE)` instead.
- Apply to every Shiny app in this repo (Tuser modules, RPreTrade, RReporting,
  RJournal, ROrder, RStudies, the standalone `Tuser/ticker/app.R`).
