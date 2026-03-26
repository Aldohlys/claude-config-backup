---
name: FX trade risk methodology
description: How Risk is set for FX/cash positions in the Trades table — 30% of notional based on historical drawdown analysis
type: project
---

FX/cash trade Risk should be set to **30% of notional**, not 100%.

**Why:** A currency going to zero in a 2-year timeframe is unrealistic. Historical analysis of CHF/JPY (2003-2026) shows worst 2-year max drawdown was ~27% (Jul-Dec 2008, 104.74→76.20). 30% covers the worst case with margin.

**How to apply:** When entering new FX trades in the Trades table, set Risk = 0.30 × |Total|. Example: trade 687 (JPY, Total=262,163 JPY) → Risk = 78,649 JPY. If a different currency pair is traded, re-run historical drawdown analysis for that pair to confirm 30% is appropriate.

Also note: `convert_fx_trade_amounts()` in `symf.R` now handles the currency conversion from trade Currency to portfolio display currency for Total/Risk/Reward. This was needed because FX cash positions in the portfolio have currency = base currency (CHF), but the Trades table stores amounts in the trade's own currency.
