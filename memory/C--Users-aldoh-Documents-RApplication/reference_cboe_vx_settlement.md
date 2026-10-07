---
name: reference_cboe_vx_settlement
description: "VIX futures VX1/VX2 from CBOE settlement CSV: URL, weekly-contract trap, no weekend files; Yahoo ^IRX is the 13-week bill, not 2Y"
metadata:
  type: reference
---

- **CBOE daily settlements:** `https://www.cboe.com/us/futures/market_statistics/settlement/csv?dt=YYYY-MM-DD` — free, no login, `read.csv(url(u))` works. Columns: Product, Symbol, Expiration Date, Price. Used by `fetch_vx_curve()` in `RStudies/reports/macro_context/fetch.R`.
- **Trap:** weekly VX contracts (`VX40/V6`, ...) are listed at the front monthly's price — read only monthly symbols `^VX/[A-Z][0-9]$`, and skip a contract expiring on the settlement date.
- No file on weekends/holidays and none for today before settlement (~16:15 ET): walk back day by day. The `cdn.cboe.com/api/global/...` JSON endpoints return 403.
- VX1 - VX2 is **negative in contango** (TODO 56 had the sign backwards).
- **Yahoo `^IRX` is the 13-week T-bill**, not the 2Y (Yahoo has no 2Y); the macro curve row is 3M/10Y.

Related: [[project_macro_bias_from_regimes]].
