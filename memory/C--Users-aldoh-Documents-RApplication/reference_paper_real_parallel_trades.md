---
name: reference_paper_real_parallel_trades
description: "Identical fills in DU5221795 (paper) and U1804173 (real) can both be real — user traded strategies in parallel; no IBKR statements for 2023"
metadata:
  type: reference
---

The user ran the same option strategies in the paper account DU5221795 and the real account U1804173 at the same time (e.g. SLV income trades Sep-Nov 2023, #346 / #361), to compare slight variations. Identical fills across the two accounts are **not** duplicates; only the same fill twice in the **same** account under two TradeNrs is (cleaned 2026-10-05, `scripts/cleanup_trades_slv_2023.R`).

IBKR Activity Statements are not available for 2023, so 2023 Trades anomalies (e.g. #315's ITM SLV 21.5 P, #478's Total/PnL gap) cannot be settled against a statement — decide from the DB rows and the user's answers.

Related: [[reference_ibkr_activity_statement]], [[project_trades_signed_delta_risk]].
