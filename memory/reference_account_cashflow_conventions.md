---
name: reference-account-cashflow-conventions
description: "Account.CashFlow conventions - native currency storage, FX via AccountWithConversionRate view, position transfers require explicit rows"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 13d07ea0-0f5c-4fb7-8397-97b2e1c8249a
---

### Storage convention
`Account.CashFlow` is stored in **native currency** (matching the row's `Currency` column). One row per currency per leg. Sign = from this account's perspective (positive = IN, negative = OUT).

### FX conversion happens on read
`Tdata::readAccount()` queries the `AccountWithConversionRate` SQL view, which left-joins `ConvertToCHF` / `ConvertToUSD` and multiplies `CashFlow * chf_conversion_rate` (or USD, depending on `BaseCurrency`). So:
- **Writers**: store native amount. Do not pre-convert.
- **Readers**: receive base-currency values. Do not re-convert.
- Daily FX rates come from `ConvertToCHF` table (date, currency, chf_value). The view falls back to the most recent prior date if the exact date isn't present.

### Position transfers must be recorded as CashFlow rows
Inter-account in-kind security transfers (e.g., `Internal Out/In` rows in IBKR's `Transfers` section) are **not** captured by the regular portfolio snapshot loop. They must be inserted into `Account` as synthetic CashFlow rows or the TWR will read the NLV drop as a market loss.

**Pattern** (one row per currency per leg per account):
```sql
INSERT INTO Account(account, date, heure, NetLiquidation, CashFlow, Currency)
  VALUES (<account>, <YYYYMMDD>, '00:00:0N', 0, <±native_amount>, <CCY>);
```
- `heure` slot starting at `00:00:01` and incrementing per leg
- `NetLiquidation = 0` (the regular snapshots later that day carry the post-transfer NLV)
- Same sign rule as cash flows: positive = IN to this account

### Why this matters
The accountf.R TWR caller (`group_by(date) |> summarize(CashFlow = sum(CashFlow))`) sums across all rows for the day. Since `readAccount` has already FX-converted each row, the sum is in base currency — meaningful even when legs span multiple currencies. Missing or wrong-signed legs produce phantom market moves in the TWR series.

### Related
- [[reference-ibkr-activity-statement]] — where to source the per-symbol/per-currency ground truth
- [[project-account-transfer-cashflow-signs]] — the 2026-04-16 case study and the Tdata 5.10.18 `twr` interpolation fix
