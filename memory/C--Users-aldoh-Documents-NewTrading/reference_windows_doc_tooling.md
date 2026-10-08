---
name: reference_windows_doc_tooling
description: "Doc/chart tooling on this Windows machine — LibreOffice headless for xlsx recalc + PNG render; reportlab/openpyxl via pip --target scratch; no node, no pdftoppm"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9b36fcb1-d3de-49ad-8593-4371aca8da6f
  modified: 2026-10-08T21:15:36.969Z
---

Verified 2026-10-08:
- **LibreOffice** at `/c/Program Files/LibreOffice/program/soffice.exe`. `--headless --convert-to xlsx` recalculates openpyxl formulas (openpyxl writes no cached values) → read back with `data_only=True` to check results/`#` errors. `--convert-to png` renders only the **first** sheet → reorder a copy to render another tab.
- LibreOffice drew negative bars upward with `overlap=100` clustered bar chart → use positive helper columns side by side.
- **reportlab / openpyxl not in base python or miniconda**: `python -m pip install --target <scratchpad>/pylib reportlab openpyxl`, run with `PYTHONPATH`. Windows python needs `C:/...` paths, not `/c/...`.
- Arial TTFs in `C:/Windows/Fonts` for accented French text in reportlab.
- **No node** (dataviz palette validator can't run); **no pdftoppm** (Read can't render PDF pages → render via LibreOffice PNG instead).
- Multi-line `Rscript -e '...'` prints nothing on Windows → write a script file.

Related: [[reference_markdown_pdf_pipeline]], [[reference_budget_project_data_traps]]
