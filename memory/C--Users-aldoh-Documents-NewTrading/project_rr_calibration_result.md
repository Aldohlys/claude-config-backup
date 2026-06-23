---
name: BOT R:R_min calibration result 2026-04-27
description: Empirical R:R_min derived from realized BOT winner exits in Trades table; threshold = 0.5 at 25th percentile of R:R_max
type: project
originSessionId: 53dab338-46f2-4aba-b466-d9cda0aa1007
---
Calibration done 2026-04-27 from `Trades` table BOT winners (37 closed, single-leg long-option, paired entry/exit). Script: `C:/Users/aldoh/Documents/NewTrading/scripts/calibrate_rr_min.R`. Output CSV: `reports/rr_calibration.csv`.

**Method:** Per Instrument, R:R = (max_exit_prix − avg_entry_prix) / avg_entry_prix. The "max exit" is the highest price across closing legs — the best target the trader actually hit. This is the closest analog to the live scanner's BS-forward-at-structural-target output. No external data; pure DB.

**Distribution (R:R_max):**
- Min: −0.73, Median: 1.82, Mean: 2.04, Max: 6.40
- Percentiles: 5%=0.01, 10%=0.16, 25%=0.49, 50%=1.82, 75%=3.45, 90%=4.34

**Threshold seed:** `R:R_min = 0.5` (25th percentile of R:R_max). Anchors Step E.2 TOP PICK gate condition #5 and Step D.4 `Entry_Ceiling` BS-inversion.

**Key insight:** the lens delivers **asymmetric small wins** more often than home runs. The 25% of winners with R:R<0.5 took fast small profits (e.g., STNG R:R=0.81, GOLD short trade R:R=0.81, AMRZ-class). Setting R:R_min higher (e.g., median 1.82) would reject these legitimate smaller-edge winners.

**Why not external data:** Yahoo `quantmod::getSymbols` was unstable in script context (rate-limited/connection failures after first few calls). Trades table has entry_prix and exit_prix — direct empirical anchor, no modeling needed.

**Tuning note:** revisit during validation. JPM-class trades (call spreads with $262 debit, ~$1000 max payout) have ceiling R:R ≈ 2.8 if hit at max but realistic ≈ 1.0–1.5. R:R_min = 0.5 lets JPM through; scanner has to reject it via Phase B (sector RS) or Phase C (cheap) — which is the design intent. R:R is meant to filter "no edge left at this entry price," not "low-conviction trade."
