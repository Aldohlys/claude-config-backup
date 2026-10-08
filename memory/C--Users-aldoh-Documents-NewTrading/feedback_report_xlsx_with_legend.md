---
name: feedback-report-xlsx-with-legend
description: Daily tabular report outputs go out as xlsx (Data + Legend sheets) written by the pipeline itself; drop the CSV when nothing reads it
metadata:
  node_type: memory
  type: feedback
  originSessionId: e13833e0-75b1-4cd3-b4b2-9a6d425e2a9d
  modified: 2026-10-08T17:56:22.126Z
---

User (2026-10-08), on intermarket_sectors: asked for a legend tab defining every column, then
"every daily run should produce the workbook with the legend. If not necessary for the rest of
codebase, remove the CSV production and keep only the workbook."

Done in RStudies 8a84f88: `reports/macro_context/intermarket_xlsx.R` (SECTORS_LEGEND +
write_sectors_xlsx, stops if a column lacks a definition), openxlsx 4.2.9 added to the RStudies
renv (it had no xlsx writer before). Same day the user asked the same for bot_daily ("as an
XLSX file, not a CSV file"): `reports/bot_daily/bot_daily_xlsx.R` (Data / Detail / Legend), and
the CSV reader (bot_fwd) was moved to the Detail sheet rather than keeping the CSV. So a reader
is a reason to port the reader, not automatically to keep the CSV — ask if porting is large.

Reading these workbooks from Python: [[reference-openxlsx-openpyxl-read]].

**Why:** the user reads these tables in Excel; a CSV can't carry a legend, and a one-off
converted file goes stale the next morning.

**How to apply:** for a new or reworked tabular report, generate the workbook with a Legend
sheet in the producing script. Before removing a CSV, grep all repos (RApplication, RStudies,
NewTrading scripts, .bat launchers) for readers; keep it if any exist. Legend content per
[[feedback-legend-definitions-not-rationale]].
