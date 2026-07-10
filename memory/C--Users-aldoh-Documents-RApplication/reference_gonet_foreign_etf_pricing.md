---
name: reference_gonet_foreign_etf_pricing
description: "getGonet prices via IBKR Tickers Exchange/ConId — cross-listed USD foreign ETFs need ConId pinned or a delayed exchange, not SMART"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8392b3e4-5d8b-457c-9eab-5c60ef858317
---

`getGonet` → `tdata_py.getValue` builds the IBKR contract from `Tickers.Exchange`/`Currency`/`ConId`. For a **USD-denominated non-US ETF**, `Exchange=SMART` + `Currency=USD` resolves the **US listing** (wrong price).

Fix pattern — pin the exact contract via `Tickers.ConId` (precedent already in DB: CSBGU0 conId 79000139, FXC 40668352 — both USD Swiss `.SW` ETFs kept on SMART with a ConId). TRE7 pins ConId **and** uses `Exchange=LSEETF`.

CNYA case (fixed 2026-07-10): was `SMART/USD/ConId=NA` → resolved US CNYA (conId 236798096, ~$36). Corrected to `Exchange=LSEETF, ConId=190114251` (Swiss/UCITS listing, ~$6). The wrong price had polluted the Gonet snapshot + Account rows; those were purged.

Two gotchas:
- `getGonet` fetches **delayed** data (reqType=4) only when `Tickers.Exchange` ∈ {`LSEETF`,`EBS`,`ALLFUNDS`}; else **frozen** (reqType=2). Frozen returned **NO price** for the UCITS conId — so a delayed exchange is required even with a ConId.
- Resolve conIds live via `ib.reqContractDetails(Stock(sym, exchange, currency))` (returns conId + primaryExchange + longName per candidate).

See [[reference_tickers_fut_row_convention]], [[project_chains_manager_fut_bug]].
