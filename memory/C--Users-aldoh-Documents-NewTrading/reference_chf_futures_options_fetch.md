---
name: reference_chf_futures_options_fetch
description: "How to pull CME CHF (6S) futures-option quotes — IBKR MCP can't find them; ib_async Future('CHF') + class CHU works; error 10197 blocks some strikes"
metadata:
  node_type: memory
  type: reference
  originSessionId: 250890ed-cd37-47c6-bd39-228637ce7645
  modified: 2026-10-16T05:27:59.278Z
---

The claude.ai IBKR MCP `search_contracts` returns nothing usable for CHF futures ("6S", "Swiss franc", "EUR.CHF futures" all empty; "CHF" shows FUT/FOP sections but no underlying id; micro MSF id 81829079 errors on option params). Use TWS via ib_async instead (r-reticulate python, see [[reference_tdata_vs_ibkr_mcp]]):

- `Future("CHF", "202706", "CME")` → qualify; `reqSecDefOptParams("CHF", "CME", "FUT", conId)`.
- Option trading class `CHU`, multiplier 125,000, quoted USD per CHF. Monthly serials under each quarterly future; quarterly expiries: Mar-27 = 20270305, Jun-27 = 20270604 (futures LTD 20270614).
- `reqMarketDataType(4)` returned real-time (type 1) quotes for most strikes on 2026-10-16 (CME hours).
- Error 10197 "No market data during competing live session" blanks bid/ask on some strikes when the TWS on 7496 shares data with another session. modelGreeks IV still arrives → price the missing strike with Black-76 from that IV (reproduced quoted mids within 0.0003).
- USDCHF spot: `qualifyContracts(Forex("USDCHF"))` first, else ib_async raises "can't be hashed".

Script: `Trades/Gonet_crash_sim/chf_quotes.py`. Carry check: Jun-27 future sat 3.0% above spot (USD–CHF rate gap ~4.5%/yr), so CHF call strikes must be read against spot, not the future. Related: [[project_gonet_crash_simulation_20261016]], [[ibkr_fx_exposure]].
