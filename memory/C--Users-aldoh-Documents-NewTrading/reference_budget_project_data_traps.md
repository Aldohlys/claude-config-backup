---
name: reference_budget_project_data_traps
description: "Personal budget RStudio project (Documents\\Budget, CLER CHF + Boursorama EUR → yearly Google Sheets) — export quirks found 2026-10-08 that silently corrupt totals/balances"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9b36fcb1-d3de-49ad-8593-4371aca8da6f
  modified: 2026-10-08T21:15:31.281Z
---

Project `C:\Users\aldoh\Documents\Budget` (not a git repo): `Bourso.R` + `CLER.R` (interactive, ask export file name + o/n questions) → `Bourso.csv` / `CLER.csv` → `Budget.R` → `budget_AAAA_MM-MM.csv` per calendar year → imported into Google Sheet "Budget" (Drive id `107L_uHXBMerdA4na2P05LZM4GQO9xprff3qz25hG8eE`, one tab per year). Manual: `Budget  - Mode d'emploi.pdf`. Summary workbook `Budget_2022-2026.xlsx`.

Data traps (verified 2026-10-08):
- **Boursorama export = joint account + card accounts** (VISA PREMIER ..6087 until 12-2025, then ULTIM ..6576). Each card purchase is a line, AND the monthly "Relevé différé Carte" debit on the joint account repeats it → double count (~16.4k CHF/yr). `Budget.R` drops the Relevé rows. Export must cover all accounts.
- **Bourso second "Solde" column on card lines = card running total**, not account balance → month-end balance only from `compte == "BoursoBank (joint)"`.
- **Both exports list newest operation first within a day**; month-end balance = first row of last date, not last row (CLER 30.09.2026: 32,176.21 not 35,677).
- **CLER has two accounts**: courant + épargne; expenses are sometimes paid from épargne (insurance, taxes). Épargne export has no balance → solde set NA so balances stay courant-only.
- **Google Sheet locale Switzerland: CSV import reads neither `16668,30` (×100) nor `16668.3` (text)** → files written as whole numbers; imported with "Remplacer les données à partir de la cellule sélectionnée" on the first month's cell, so files have no header; import wipes CHF format (repaint).
- Older tabs differ in layout: Revenus/Dépenses/Epargne cumulée in cols E/F/I (2022), D/E/H (2023), B/C/F (2024+); Epargne cumulée restarts each January. Yearly "taux d'épargne" in the sheet = Solde / Revenus.

Next run (Oct–Dec 2026 into 2026 tab): `report_delta = 57739.89`, `report_epargne = 59563.93` (Sept 2026 cumulatives).

Related: [[reference_windows_doc_tooling]]
