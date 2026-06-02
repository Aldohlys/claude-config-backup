---
name: project-account-transfer-cashflow-signs
description: 2026-04-16 U1804173↔U25343478 transfer TWR fix - missing in-kind position transfer rows + Tdata::twr interpolation bug
metadata: 
  node_type: memory
  type: project
  originSessionId: 13d07ea0-0f5c-4fb7-8397-97b2e1c8249a
---

On 2026-04-16 the user transferred 5 securities (CA, CRST, 3440.T, 7399.T, 9273.T) plus a cash dowry from U1804173 → U25343478 as part of moving the VALUE strategy. 6797.T followed on 2026-04-27. The DB-computed TWR for U1804173 was wildly wrong (catastrophic -5.32% on 04-16). Two independent bugs converged:

### Bug 1 — missing in-kind position-transfer CashFlow rows

DB recorded only the **cash** legs of the transfer (signs were correct against IBKR):
- U1804173: CHF -20000, JPY +618744, JPY +3058, GBP +441 (cash dowry OUT + FX cash sent BACK from receiver)
- U25343478: mirror

But the **in-kind security transfer** (~7,340 CHF of stocks moving) was not recorded as a CashFlow at all. TWR therefore read the NLV drop as a market loss. Fix: insert 5 rows on 2026-04-16 + 1 row on 2026-04-27 per account (native currency, signed per account perspective). Also a missing 2026-04-01 EUR -5,000 withdrawal was added.

**Source of truth**: IBKR Activity Statement (`U1804173_YYYYMMDD_YYYYMMDD.csv` / same for U25343478), sections:
- **Cash Report** rows for `Account Transfers` per currency
- **Transfers** section for in-kind security transfers (per-symbol, with `Internal Out`/`Internal In` direction)
- **Net Asset Value** → `Time Weighted Rate of Return` is the ground-truth TWR to match

### Bug 2 — Tdata::twr linear-interpolated NLV across missing days

`Tdata::twr` (pre 5.10.18) called `zoo::na.approx` to fill NLV for every missing calendar day, then chained daily returns. When a cashflow event followed a multi-day gap, the interpolated NLV on the day before the cashflow lacked the cashflow adjustment → split the real NLV move into a phantom market loss + phantom gain that compounded. U1804173 had 57/131 days interpolated YTD 2026 → +8.96% computed vs IBKR's official +1.20%.

**Fix in Tdata 5.10.18** (`R/account.R`): removed `xts`/`zoo` merge + interpolation, chain links consecutive **observed** dates directly. After: U1804173 TWR = -0.27% vs IBKR +1.20%. Residual gap explained by: snapshot ending 1 day earlier than IBKR statement + intraday cashflow-timing granularity. Big swings gone.

### How to apply
- When a user moves securities between accounts, record both **cash legs** AND **in-kind position-transfer legs** in `Account.CashFlow`. One row per currency per leg per account; sign = positive for IN to that account. Native currency (FX conversion handled by `AccountWithConversionRate` view).
- IBKR Activity Statement is the authoritative cross-check: its `Time Weighted Rate of Return` should be reachable within ~1-2pp once cashflows are complete.
- Backup before bulk DB inserts: `data/mydb_pre_<fix-name>_<timestamp>.db`. Done for this fix at 2026-05-18.

### Earlier (wrong) hypothesis
This memory previously claimed the FX-leg signs were inverted. **That was wrong.** The signs match IBKR. The real bugs were the missing in-kind rows + the interpolation in `twr`. Earlier diagnosis was based on the user's stated mental model ("everything moved from A to B"), which omitted that the foreign-currency cash legs actually flowed from B to A as part of the rebalance.

### Related
- Tdata CHANGELOG 5.10.18 documents the `twr` patch.
- Scripts: `scripts/apply_transfer_cashflow_fix.R` (inserts), `scripts/verify_twr_patch.R` (verification), `scripts/diagnose_twr_gap.R` (residual investigation).
