---
name: feedback_no_edit_running_rscript
description: "Never edit an R script while Rscript is running it — Rscript reads the file expression by expression, so an edit shifts the text and crashes the run mid-way"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 1c26deb6-0a13-4dd0-99ab-91d81259daed
  modified: 2026-10-01T01:23:25.028Z
---

Rscript (R -f) does NOT parse the whole file at start: it reads and evaluates
top-level expressions incrementally. Editing the file while it runs shifts the
remaining bytes; R then parses a garbled fragment and stops ("constante de type
chaîne de caractères inattendue", a truncated string).

2026-10-01: I fixed a message in RStudies/reports/bot_monthly/main.R during the
user's BOT_monthly run, claiming "R already loaded the file" — wrong. Phase 1
(348 names) had finished; the run crashed at phase 2 before writing Tickers.

**Why:** a crash mid-run wastes the run and can leave partial writes.
**How to apply:** while a job launched from a script is running (BOT_monthly,
BOT_daily, collectors, /analyze), do not edit that script or anything it
`source()`s later in its flow; queue the edit until the run ends. For
BOT_monthly, a phase-2 failure can be resumed with `--resume` (phase-1 cache
data/bot_monthly_phase1.rds).
