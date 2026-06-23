---
name: reference_closed_market_option_marks_stale
description: "Exchange-closed option pulls return populated bid/ask but null last/close = frozen, not executable"
metadata: 
  node_type: memory
  type: reference
  originSessionId: ee50ec03-6504-49a0-a229-909d602ca8db
---

When the option's exchange is **closed** (e.g. EUREX overnight, OESX before ~09:00 CET), `getOptValue` still returns **populated bid/ask but null `last` / `close` / `uPrice`**. These are **frozen snapshot / market-maker model marks, NOT a live two-sided market** — you cannot execute against them.

**The tell that they're stale:** they haven't repriced to the overnight move. On 2026-06-02 the FESX future was +0.45% overnight, but the 4am OESX marks barely moved — a quick **delta × spot-move check** (e.g. a –0.28-delta put should drop ~7 pts on a +27-pt index move) exposes that the marks are frozen. Live-market quotes also show **much tighter bid/ask** (OESX 5,700P: 0.8–1.4 wide live at the open vs ~1.4–3.2 frozen).

**How to apply:** for any execution decision, only trust the pull taken when the option's own exchange is open. Before the open, use a live **futures/CFD spot proxy** ([[reference_cfd_cash_synthetic_perpetual]]) plus BS-repricing to *estimate* the open, clearly labelled indicative — then re-pull live at the open and set limits off those mids. Don't quote frozen marks as executable. Related: [[reference_oesx_eurex_option_mechanics]], [[project_estx50_hedge_roll_20260601]].
