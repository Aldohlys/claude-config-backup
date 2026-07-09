---
name: feedback_correlation_regime_and_outliers
description: "For this user, always test correlations for regime-dependence AND outlier/crisis-concentration — not just full-sample Pearson"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9086880f-22ad-4f6d-a57f-6eda3257a22b
---

When analyzing correlations (hedge proxies, cross-asset links, diversification), do NOT rely on a single full-sample Pearson number. Test two things the user proactively asks about:

1. **Regime-dependence.** Split by volatility regime (VIX/VSTOXX or trailing realized vol, lagged one day). Example: ABBN~ESTX50 rose **0.54 (VIX<15) → 0.84 (VIX>21) → 0.91 (VIX>30 stress)**, with beta stable ~1.0 across regimes. So cross-asset/index hedge correlations are **loose in calm tape, tight in a crash**.

2. **Outlier / crisis-concentration.** A moderate full-sample correlation can be an artifact of a few common-shock days. Test by excluding the top-N biggest-combined-move days, Spearman vs Pearson, and quiet-days-only. Example: CNYA~SMH = **0.46 full → 0.21 excluding the top 10% of days → 0.68 on the biggest days only** — the co-movement is concentrated in shared global risk-off events (Apr-2025 tariff shock; Jun-2026 Iran-war-type risk-off), NOT a persistent driver. Day-to-day it's ~0.2.

**Why:** the user raised both angles unprompted (the regime question on the ABBN hedge; the Iran-war-outlier point on CNYA~SMH), and both **change the conclusion**. Full-sample correlation over/under-states what matters.

**How to apply:**
- Frame **index hedges as crash insurance, not correction/tracking hedges** — they engage (correlations→1) exactly in the stress they're for, and track loosely otherwise. A calm-period correlation understates a crash hedge's effectiveness (and vice-versa).
- Warn that **diversification vanishes in a crisis** — a low normal-day correlation (good diversifier in calm tape) still co-crashes on a global shock. So diversification is a calm-time benefit; tail hedges, not diversification, protect a book in a panic.
- A proxy that "looks linked" on a chart may just co-crash on shared shocks — check before attributing a driver (e.g. "semis drive CNYA" was false; it was global risk-off, and CNYA's real day-to-day co-mover is China itself, MCHI 0.75).

Related: [[feedback_counterparty_positioning_lens]], [[project_estx50_hedge_roll_20260601]], [[user_em_china_view]].
