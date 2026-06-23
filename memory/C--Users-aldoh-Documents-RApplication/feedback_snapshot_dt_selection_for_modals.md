---
name: feedback_snapshot_dt_selection_for_modals
description: "In Shiny+DT modal flows, snapshot `input$<table>_rows_selected` data at button-press time; reading it again at modal-OK time can silently act on a different row than the modal was opened for"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c9c28ae6-c8a6-4bc5-9fb8-30733c82e211
---

Rule: When a Shiny modal acts on rows the user selected in a DT, capture `<table_data>()[input$<table>_rows_selected,]` into a reactiveVal at the moment the trigger button (Open/Adjust/Close/etc.) is pressed. Have every downstream handler — modal OK, post-op cleanup observers, auto-recompute observers — read from that snapshot, not from a fresh `input$<table>_rows_selected` lookup.

**Why:** Between button press and modal OK the live selection / underlying reactive can drift:
- DT preserves multi-select across re-renders, so a previous click can still be in `_rows_selected`.
- An intermediate op (e.g. an Open that marks a row "Ouvert") changes the filter result; `input$_rows_selected` indices then point at different rows.
- Double-fire of the OK button (or reactive recomputation inside the OK handler) re-evaluates the live selection.

If the OK handler re-reads the live selection it operates on a different row than the modal was opened for, silently. RReporting trade 721 (2026-05-14): user pressed Adjust on the MCL row; OK handler re-read the live selection and passed an extra NVDA row into `adjust_trade`, recording an unrelated NVDA short call into trade 721. Modal display vs. modal action used different rows.

**How to apply:** For Open/Adjust/Close-style flows, structure as:
1. Button observer (`input$adjust` etc.): capture `<table_data>()[input$xxx_rows_selected,]` into a `staged_data` reactiveVal, validate consistency (single underlying / single row / non-empty), then open the modal.
2. OK observer: read `staged_data()` only. Error out if it's NULL/empty rather than re-reading the live selection.
3. `staged_data(NULL)` on success — this also blocks double-fires (second OK reads NULL and surfaces an error instead of silently re-acting).
4. Any "auto-recompute" observers in the modal (e.g. risk recompute on strategy change) should also read from `staged_data()`, not the live selection — otherwise the modal's displayed value can diverge from what the OK handler will actually use.

Anti-pattern (what trade 721 hit):
```r
observeEvent(input$adjust, {
  showModalUpdate(new_trade_data()[input$newtrades_rows_selected,])
})
observeEvent(input$ModalUpdate_ok, {
  data <- new_trade_data()[input$newtrades_rows_selected,]  # ← drifted
  adjust_trade(..., data = data)
})
```
