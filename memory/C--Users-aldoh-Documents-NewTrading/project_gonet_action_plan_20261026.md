---
name: project_gonet_action_plan_20261026
description: "Gonet action plan 26.10 (Reports/Gonet_action_plan_20261026.md) follow-up — AMRZ sold 28.10, CNYA under 5.25 intraday 29.10, NUCL clear; ledger reconciled to 29.10 statement"
metadata:
  node_type: memory
  type: project
  originSessionId: ed36ec08-571d-41be-80c7-00a6d85f703d
  modified: 2026-10-29T08:24:42.774Z
---

Plan file: `Reports/Gonet_action_plan_20261026.md` (dated actions, exit alerts 81–114, 13.11 hedge sizing).

Follow-up 2026-10-29 (movements 29.09–29.10 + estimation PDF 29.10):
- AMRZ: all 260 sold 28.10 @ 33.22, value 30.10, net CHF 8,460.70 (commission 150 + courtage 20 + ~6.50 tax ≈ 2.0% all-in). Realized ≈ +CHF 2,377 vs 6,084 basis. Results day 29.10 the stock opened 30.30 (−8.4%), low 29.89. Booked as TradeNr 19 row dated 30.10; AMRZ removed from GonetPos; alert 94 deactivated.
- QQQ dividend USD 32.61 (08.10) booked as TradeNr 9.
- Ledger cash after booking = statement to the cent: CHF 22,987.50 / USD 15,033.02 / EUR 1,931.69. The PDF already counted the AMRZ cash before its value date.
- CNYA (alert 83, level 5.25, weekly close rule): Yahoo CNYA.L 5.22 intraday 29.10; 28.10 bar missing on Yahoo. Decision point = Friday 30.10 weekly close → sell all 2,670 if below 5.25.
- NUCL (alert 81, 41.9): 44.75 on 29.10, ~6% above; uranium rebound (URA 34.1 → 36.6, 23–28.10).
- L'Oréal still under trading ISIN FR0000120321 on 29.10 (nominatif form sent 26.10, not yet processed) — [[project_gonet_dividend_reconciliation]].
- No Gonet DB snapshot on 28.10 (last 27.10 10:02).

**How to apply:** next follow-up starts from the plan table; check CNYA outcome of the 30.10 close, then 13.11 hedge ([[project_estx50_hedge_roll_20260601]]).
