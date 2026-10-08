---
name: feedback_ui_constant_cells_and_tooltips
description: In app tables leave cells blank when their value is fixed by construction; put explanations of computed amounts and codes in tooltips
metadata:
  node_type: memory
  type: feedback
  originSessionId: 3f7efce0-fd27-406b-b771-59dc993e4d3e
  modified: 2026-10-08T10:49:13.981Z
---

Leave a cell blank when its value is fixed by construction (e.g. the Actual
column of a "% of |P&L|" row is always ±100%), rather than displaying it.
Explanations of a computed amount or of a code column go into a hover tooltip
on that cell / header (dotted underline, `cursor: help`), not into extra text
on the page.

**Why:** 2026-10-08, P&L path table in the routine Trade tab — the user asked
"What means -100% ... If value is fixed at 100%, better leave it blank", then
asked for the entry-execution explanation and the Flag meanings as tooltips.
A constant cell reads as information and prompts a question; a tooltip keeps
the table compact and the explanation one hover away.

**How to apply:** when designing a table, drop or blank any cell whose value
follows from the row's definition. For DT, put the tooltip as
`<span title="...">` in the cell (unescape only that column via
`escape = setdiff(names(tbl), "Col")`) and on the header through a custom
`container` with `tags$th(title = ...)`; multi-line titles use `&#10;` in cells.
Also: show weekdays in English via a fixed vector — `%a` follows the French
locale here ("lun."). Related: [[feedback_design_before_code]],
[[reference_dt_idioms]].
