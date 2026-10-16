---
name: reference_eurchf_fall_two_drivers
description: "EUR/CHF falls come in two kinds — euro-led (USD/CHF up, book self-hedges) vs franc-led (USD/CHF down too, both sleeves lose); CHF calls only pay the second"
metadata:
  node_type: memory
  type: reference
  originSessionId: 250890ed-cd37-47c6-bd39-228637ce7645
  modified: 2026-10-16T05:28:06.184Z
---

Measured 2026-10-16 (Yahoo, `Trades/Gonet_crash_sim/chf_hedge.py`):

- Franc-led: 2011 pre-floor EUR/CHF −12.3% with USD/CHF −10.4%; SNB 2015 −17.2% / −16.2%. CME CHF calls pay.
- Euro-led: 2010 Greece −8.0% / +2.2%; 2022 −8.6% / +6.5%; GFC −10.5% / +10.1% (EUR/USD −10 to −19%). CHF calls pay nothing.
- Weekly since 2015, EUR/CHF < −1% weeks (n=39): USD/CHF also down 79% of the time, beta 1.58.

For a CHF-based book holding both EUR and USD (Gonet: ≈112k / 109k CHF), euro-led falls net to −1k to −7k because the USD sleeve gains; franc-led falls hit both (−25k to −36k). So the hedge gap is a rising franc, and a CHF call (spread) on the USD cross is the right instrument even for an "EUR/CHF crash" worry. Full FX hedging (forwards / CHF-hedged classes) costs the rate gap (~4%/yr USD, ~2.8% EUR) and removes the euro-led offset.

**How to apply:** before proposing FX hedges for a CHF investor, classify the shock by driver and net the currency sleeves first. Related: [[project_gonet_crash_simulation_20261016]], [[reference_chf_futures_options_fetch]].
