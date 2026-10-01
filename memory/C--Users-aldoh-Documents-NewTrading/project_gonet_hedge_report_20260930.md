---
name: project_gonet_hedge_report_20260930
description: "Gonet-wide hedge review 2026-09-30 — −10% stress, ESTX50 midterm add-lot rejected, TLT put rejected, GLD Mar-27 350/310 put spread left open"
metadata:
  node_type: memory
  type: project
  originSessionId: c30847a6-cfac-4f2c-929f-c04e990c3499
  modified: 2026-09-30T21:46:44.827Z
---

Report: `Trades/Gonet_hedge_report_20260930.md` (commit bb2822d). Companion to [[project_estx50_hedge_roll_20260601]].

- Stress: −10% on all equities and bonds = −30.3k CHF (−7.5%) on 405k. Gold 10.8% + cash 14.5% untouched. Real-world worse: CHF strength (130k in USD/EUR) and bonds only fall with equities in the inflation regime.
- ESTX50 vs the Gonet equity basket: corr 0.85, beta 1.07 (1.20 in down months) → −10% basket ≈ ESTX50 −10.7% (≈5,660). The Dec 5,700/4,600 × 1 gains ≈2.2k CHF = 7–8% of the loss.
- Adding a second Dec lot before the 3-Nov-2026 US midterms: rejected (US event vs EU hedge, 40–65% theta by 3 Nov, it's the token lot the 21-Sep pre-commitment ruled out). If acting, pull the first 2027 lot forward (6.5%-OTM rule), only if the 2027 hedge will be funded.
- Size ≠ risk: US Treasuries (DTLA/CSBGU0/TRE7) = 40k CHF but only ≈27.6k USD TLT-equivalent; TLT −10% costs ≈2.3k CHF. USD/CHF on that sleeve is as large as its rate risk. The biggest bond line is the Euro bond fund (TLT doesn't touch it). Gold coins = 2nd-largest risk (≈7.3k CHF 1σ/yr), beta 0.86 to GLD.
- Neither TLT nor GLD fell in equity stress (18 ESTX50 < −3% weeks: TLT +0.4%, GLD −0.5%) → their puts don't help an equity crash; they pay only in the sleeve's own shock.
- TLT bear put rejected: IV 16–18% vs 10% realised (≈2.5× fair). If duration worries, sell DTLA directly.
- GLD open decision: Mar-27 350/310 × 1 ≈ USD 577 (fair-to-cheap, IV ≈20% vs RV 23–25%) vs selling 8–10 coins vs nothing. IAU chain unusable (0 bids, wide spreads).

**Why:** the user asked for hedges on the largest Gonet sleeves by size; the analysis showed risk, not size, should drive hedging.
**How to apply:** at the 13-Nov ESTX50 decision or any Gonet hedge review, start from this report; re-price live (US open / EUREX hours).
