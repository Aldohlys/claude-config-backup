---
name: reference-tdata-vs-ibkr-mcp
description: "Prefer Tdata over the claude.ai IBKR MCP for positions/option data; MCP gaps (IV, frozen quotes, combo orders)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 9086880f-22ad-4f6d-a57f-6eda3257a22b
---

For retrieving IBKR positions and option market data, prefer the user's **Tdata** library over the claude.ai **Interactive Brokers (IBKR) MCP** — verified against Tdata source (`Tdata/inst/python/tdata_py/account.py`, `Tdata/R/account.R`) and observed MCP behavior on 2026-06-30.

**Tdata advantages (confirmed in code):**
- `getIBKRData()` → `retrievePortfolioData()` returns each option position WITH TWS `modelGreeks` (delta/gamma/vega/theta/IV/undPrice). The MCP returned invalid IV on every option and no Greeks.
- Uses `reqMarketDataType(2)` (frozen) which works outside US RTH and on EUREX. The MCP `get_price_snapshot` explicitly does NOT support frozen/delayed-frozen → returned EMPTY bid/ask on US options pre-open and on thin ABBN Eurex chain.
- BS-solves IV when TWS returns null (`impliedvol.py`).
- Multi-currency CHF conversion + prices the Gonet book (incl. `PM_` precious metals via web scrape in `getGonet`). The MCP only sees IBKR, never Gonet.
- Same TWS the user trades in (source of truth); can target sub-accounts (U1804173 etc.).

**IBKR MCP advantages:** zero setup (cloud auth, no TWS/R session); fast + correct for live STOCK snapshots, option-chain STRUCTURE (`get_option_parameters`/`get_option_data`), account balances, trade list.

**Key gap — neither cleanly shows pending combo/BAG orders.** MCP `get_account_orders` silently DROPPED a resting IAU collar (BAG); Tdata `getIBKRData` reads `ib.portfolio()` (positions, not open orders) so it misses unfilled combos too. Would need a `reqAllOpenOrders` call (not wired up). For resting combos, the TWS order window stays the source of truth.

**How to call Tdata (verified working 2026-06-30):** the r-btw MCP is DOCS-ONLY (help/vignettes + list/select session) — it has NO R-eval tool, so you can't run `getIBKR()` through it. Drive Tdata by calling the Python lib directly from a shell with the reticulate env interpreter:
- Python: `C:\Users\aldoh\AppData\Local\r-miniconda\envs\r-reticulate\python.exe` (Python 3.10, ib_async 2.1.0, pandas 2.3.1). NOT `miniconda3/envs/r-reticulate` (that path doesn't exist — the env is the R-managed r-miniconda under AppData\Local).
- `sys.path.insert(0, r"C:\Users\aldoh\Documents\RApplication\Tdata\inst\python")` then `import tdata_py as td`. It self-discovers `config.yml` (DB=`...RApplication/data/mydb.db`, TWS port default 7496) on import.
- `td.getIBKRData("U1804173")` → `[account_df, portfolio_df, currency_balances_df]`; portfolio_df carries per-leg delta/gamma/vega/theta/IV/uPrice/multiplier — compute net via multiplier*greek*pos. Needs TWS running. Other useful entrypoints: `td.getValue`, `td.getOptValue`, `td.compute_spread_risk_reward`, `td.get_volatility_metrics`.

Requires TWS running. Use the IBKR MCP as the zero-setup fallback for quick stock quotes, chain discovery, balances, trades. Related: [[reference-tdata-option-fetch-internals]], [[reference-iv-solve-when-tws-returns-null]], [[reference-tdata-install-topology]], [[reference-python-conda-launch]].
