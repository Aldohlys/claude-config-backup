---
name: reference_smi_osmi_option_mechanics
description: "SMI index options (EUREX product OSMI) — multiplier, style, expiries, liquidity, and IBKR MCP data behavior"
metadata: 
  node_type: memory
  type: reference
  originSessionId: bd9ad6e2-6358-4db8-8add-e9ee8e2cfee1
---

SMI index options on EUREX (discovered live 2026-07-10 via IBKR MCP).

- Underlying: SMI (Swiss Market Index), EUREX, IBKR underlying_contract_id 1328305; spot ~14,200 (Jul-2026).
- Options product code **OSMI**; **European-style, cash-settled, CHF**; **multiplier CHF 10 per index point** (look up before sizing, cf. [[reference_cl_options_multiplier]]). So one point of strike width = CHF 10 face → deep width needed for large CHF notional (e.g. 100k face = 10,000 pts of width, or many narrow boxes).
- Strikes in 50-pt steps near spot. Expiries: nearby weeklies + monthlies (3rd Fri), quarterly LEAPS out to ~2030.
- Swiss single-stock EUREX options (Nestlé/Roche/Novartis…) are **American-style** → early-assignment risk; only the *index* OSMI options are European (needed for a clean box / parity trade).
- IBKR MCP returns live OSMI bid/ask + open interest, but `option_midpoint_iv` can come back invalid (-1) on some legs, and far/less-used strikes can show 0 OI — thin book, wide legs (~20 pts on near-money 69-DTE). Trade as combo/net-price, not legging.
- Analogous EUREX index note for ESTX50: [[reference_oesx_eurex_option_mechanics]]. CHF-box viability: [[reference_box_spread_cash_parking]].
