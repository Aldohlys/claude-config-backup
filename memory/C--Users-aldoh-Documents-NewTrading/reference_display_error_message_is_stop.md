---
name: reference-display-error-message-is-stop
description: Tbasics::display_error_message() is a bare stop(), not a UI notification — never call it from a tryCatch error handler
metadata:
  type: reference
---

`Tbasics/R/general.R` defines it as nothing but `stop(error_msg)`. It displays **nothing** — despite the name and despite its sibling `display_message()`, which does show a real `shiny::modalDialog`.

The trap: calling it inside a `tryCatch(..., error = function(e) ...)` handler throws a *second* time, and that throw has nothing left to catch it. It escapes to Shiny as a raw `Avis :` block plus traceback, so the user sees a stack dump instead of the message you meant to show. Swept 2026-08-26 — was live in `Tuser/spread/app.R`, `spread/view/spreadUI.R` and `journal/view/journalUI.R`.

Rules:
- User-facing problems in a Shiny app → `showNotification(msg, type = "error")`.
- Guards inside a render → `validate(need(cond, message = ...))`, not `stop()`.
- Argument preconditions in plain non-reactive functions → `display_error_message()` is fine; that is the intended contract, and ~25 `Tdata`/`Tbasics` call sites rely on it (`getTicker()`'s multi-name guard, etc.). Left alone deliberately.

Still reachable from Shiny renders and not converted: `Tuser/strategies/logic/strategief.R:109`, `Tuser/symbol/logic/datatablef.R:167,171,172`.

Related: [[reference_tuser_repo]], [[reference_tdata_py_active_binding]]
