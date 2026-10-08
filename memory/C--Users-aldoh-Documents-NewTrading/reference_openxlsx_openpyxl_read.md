---
name: reference-openxlsx-openpyxl-read
description: "openxlsx (R) workbooks fail in openpyxl's default loader; read them read_only + reset_dimensions. Also Rscript crashes on the session scratchpad path; Excel COM not available"
metadata:
  node_type: memory
  type: reference
  originSessionId: e13833e0-75b1-4cd3-b4b2-9a6d425e2a9d
  modified: 2026-10-08T19:00:58.786Z
---

Found 2026-10-08 building the bot_daily workbook (RStudies openxlsx 4.2.9).

- openxlsx writes `xl/worksheets/_rels/sheetN.xml.rels` pointing at `../drawings/drawing1.xml` but omits the part when the sheet has no drawing. `openpyxl.load_workbook(path)` raises `KeyError: There is no item named 'xl/drawings/drawing1.xml'`. Every openxlsx workbook is affected, intermarket_sectors included.
- Fix in Python: `openpyxl.load_workbook(path, read_only=True, data_only=True)`, then `ws.reset_dimensions()` per sheet (openxlsx's `<dimension>` reads A1, so without the reset a sheet reads as 1x1). Used in `RStudies/reports/bot_fwd/intake.py::read_rows`.
- python-calamine is not installed in the conda env.
- Excel is not scriptable by COM on this machine (class not registered), so a "does Excel open it" check needs the user.
- Rscript crashed silently (exit 139 / 0xC0000005, no output) twice on 2026-10-08: once with `Rscript <file>` pointing at a scratchpad script (`AppData\Local\Temp\claude\...`), once when saving an xlsx into the scratchpad. Cause not established. `Rscript -e 'source("<scratchpad>/x.R")'` run from the RStudies root (so renv loads) worked all day, writing its output under the repo or Reports. Use that form.

Related: [[feedback-report-xlsx-with-legend]], [[project-bot-forward-test]]

Rebuilding an existing workbook while the user has it open fails ("Accès refusé" in R, "Device or resource busy" for mv): `write_bot_daily_xlsx()` leaves `<name>.xlsx.tmp.xlsx` next to it. Ask the user to close the file, then `mv -f` the tmp over it. Normal 17:10 runs write a new time-stamped name, so they are not affected.
