---
name: project_nb_emd_hard_currency_coverage
description: "Neuberger EMD Hard Currency UCITS (IE00B99K4563, USD I Acc) vs Vanguard VEMT, 2026-09-22 — higher-beta index + some alpha; USD/CHF dominates; report in Trades/"
metadata:
  node_type: memory
  type: project
  originSessionId: 22d8b8f2-ea79-426c-af84-c96ab0bf81df
  modified: 2026-09-22T21:34:03.793Z
---

Analysed 2026-09-22 from the user's KID (dated 28.04.2026). Report: `Trades/NB_EMD_HardCurrency_analysis_20260922.md`.

Conclusions:
- Since end-2019: NB 3.6%/yr vs VEMT 2.1%/yr (USD, after fees). Beta 1.17 to VEMT explains only ~0.4 pt of the ~1.5 pt gap → some real alpha.
- Pattern = credit beta: lagged 2020-22 (2022 -19.0% vs -15.3%), led 2023-25 (+4.5/+6.7/+4.5 pts) on distressed frontier recoveries now mostly priced back. Max DD -32.5% vs -24.1%; KID risk class 3/7 understates it.
- Cost 0.93% all-in (0.79 + 0.14 trading) vs VEMT ~0.25%.
- In CHF, USD's ~20% fall since 2019 swamps fund choice (NB +7.4% vs VEMT -2.5% in CHF). Hedge vs unhedged is the bigger decision.

Data method: Yahoo via yfinance — NB NAV `0P0000YYAS` (only from 2019-01-02; nb.com rate-limits/429 blocks factsheets), VEMT `VDET.L` (USD dist, adj close OK; matches acc `VDEA.L`), CHF line `VEMA.SW`, FX `USDCHF=X`.

**Why:** user comparing an active EM-debt UCITS (likely bank-proposed) against an index UCITS; buys Irish UCITS only — see [[user_prefers_irish_ucits]].
**How to apply:** open items — I-class minimum and retrocession, current YTW/spread/duration, CHF-hedged class availability, pre-2019 history. Related: [[project_dspf_coverage]], [[user_em_china_view]].
