---
name: project-bs-calculator-design
description: "BSM Calculator PWA (tools/bs_calculator/) design conventions: shared inputs + per-tab selectors, Taylor-decomposed PnL on Risks tab, IV mirror block."
metadata: 
  node_type: memory
  type: project
  originSessionId: 3ed4255d-f6de-4459-bd06-43a46d51dc9a
---

`tools/bs_calculator/index.html` is a single-file PWA built on top of [[reference-pwa-offline-pattern]]. Two structural conventions to preserve when extending it:

**Shared inputs above tabs.** Spot/Strike/Start/Expiration/r/q/σ/Position/Call-Put all live in one `<div class="shared">` block above the tabs. Each tab reads them via `readShared()` and never duplicates fields. Field order is fixed and user-chosen (2026-05-19): Start·Position / Spot·q / Exp·K / σ·r. Position is a signed integer (default 1) and the contract multiplier is hardcoded 100.

**Per-tab segmented selectors instead of multi-axis tables.** The Risks tab demonstrates the pattern: rather than render a (σ-move × day × vega-bump) 3D cube, two segmented controls collapse it — Horizon picks one of {1d, 2d, 5d}, Vega-bump picks one of {0%, +5%, +10%, +20%}. The matrix then has just rows = σ-move and cols = greek-component. Persisted in localStorage as `bsm.day`, `bsm.vbump`.

**Why:** mobile screens cannot display dense matrices without horizontal scroll. The selector approach also matches the user's mental model ("show me PnL at +2d if IV bumps +10%") better than a flat dump of every combination.

**How to apply:**
- New tab with multiple dimensions → use selectors to collapse all-but-two before drawing a table.
- Risks tab Total cell = Δ+Γ PnL + Θ PnL + Vega PnL @ selected bump. The 0% option is mandatory in the bump list — it represents "no vol change" and zeroes Vega's contribution to Total.
- Greeks on the Risks tab are evaluated at t=0 (standard Taylor decomposition): `Δ·dS + ½·Γ·dS² + Θ·d + vega·dσ_pct`. Do not switch to "greeks-at-horizon" without re-discussion — user preferred the Taylor form for interpretability.
- IV tab mirrors the Price+Greeks block at the bottom (uses element IDs prefixed `iv-px-*` / `iv-g-*`). `computePx()` writes to both element sets in one pass. Don't remove this mirror — user asked for it explicitly.
- Bump `sw.js` CACHE constant on every meaningful HTML/JS change (v1 → v2 → v3 sequence in May 2026). See [[reference-pwa-offline-pattern]] for full cache-key discipline.

**Reference commit:** `b599139` (2026-05-19) — Position + reordered shared inputs + Vol/Risks tabs.
