---
name: project_analyze_prefer_monthly_expiry
description: "/analyze now probes the standard monthly (3rd Friday) chain, not the nearest-DTE weekly; fixes wide-spread false \"stock\" verdict"
metadata: 
  node_type: memory
  type: project
  originSessionId: 766f90e9-4750-4c67-8300-8b0e485a5e25
---

**Fixed 2026-06-08.** `/analyze`'s expiry selector `.pick_expiry_for_dte()` (RStudies `reports/shared/live_sources.R`) picked purely `which.min(abs(dtes - target_dte))`, so it grabbed near-dead **weeklies** when one sat closer to the 45 DTE target than the monthly. Surfaced on **C (Citigroup) 2026-06-08**: it probed the Jul 24 weekly (ATM bid/ask **25%**, 30Δ **45%**, OI ~1) and the vehicle rule (>8% → stock) wrongly demoted C to a *stock* trade. The real Jul 17 monthly was **6–7% wide, OI ~5k** — a perfectly tradeable debit-vertical chain.

**Fix:** added `prefer_monthly = TRUE` (default). Detects the standard monthly = 3rd Friday (`wday==5 & mday 15–21`); among monthlies picks the one nearest `target_dte`; falls back to nearest-any only if no monthly exists (indices/weekly-only names). One chokepoint → cascades to the liquidity probe, the vehicle rule, the structure legs (`resolve_expiry` 30d/55d), and `.live_atm_iv`. Edit is in a sourced file → live next run, no rebuild. Same lesson as the XOP monthly-vs-weekly OI finding.

**Known interaction (not yet handled):** when both structure legs (target 30 & 55) resolve to the *same* monthly (e.g. monthlies at 39 & 74 DTE → both snap to 39), the two-expiry grid collapses to one expiry, partially defeating [[feedback_analyze_two_expiries]]. Option if it bites: have the long leg pick the *next distinct* monthly. Left for user decision.

**Committed** d45592f (RStudies main, 2026-06-08), bundled with the prior-uncommitted Phase A bid/ask liquidity probe (`resolve_option_spread`) it builds on + `quotes/` gitignore.

**Earnings-window gap — FIXED 8991ddb (2026-06-08):** the funnel labels earnings against a fixed 14-day macro window in Phase C, before expiries are picked, so C showed "outside event window" (36d) while the chosen **Jul 17** monthly (39d) carries the **2026-07-14** print 3d pre-expiry. Added `.earnings_vs_expiries()` (structures.R) — flags earnings strictly between today and each PROPOSED expiry, returns `earnings_expiry` from `run_phase_d`; report.R renders an amber `.warn-box` under the vehicle banner + a Data Summary tag. Trade consequence stands: C has no clean pre-earnings breakout vehicle (liquid monthly is post-earnings, pre-earnings expiries are dead weeklies) → pass as a breakout.
