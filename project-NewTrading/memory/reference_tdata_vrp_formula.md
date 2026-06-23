---
name: Tdata VRP storage is log-ratio; trader narrative uses vol-points — do not confuse
description: Persisted vrp column (Prices, scanner CSV) is log(iv30/rv30)*100. Trade journal Remarques and Sharpe2 reasoning use the vol-point form (iv30-rv30) for safety-margin intuition. The two forms must never be confused — they have different scales and thresholds.
type: reference
originSessionId: 9cbc474c-d67a-496b-92d1-fb42a1346715
---
## Two co-existing definitions of VRP in this workspace

### 1. Persisted form (data layer) — log-ratio
- **Formula:** `VRP = log(iv30/rv30) * 100`
- **Source:** `Tdata/R/volatility.R::getVolMetrics` (line ~427), since Tdata 5.10.3 (2026-04-21)
- **Stored in:** `Prices.vrp` (SQLite DB)
- **Re-emitted by:** v5 scanner CSV `vrp` column, RStudies `vol_profile.R`, render_html tooltip
- **Thresholds in `cheap_score.R`:** ≤0 → 2 pts, ≤10 → 1 pt, else 0
- **Concrete example:** DBA on 2026-04-27, iv30=17.75%, rv30=8.58% → stored vrp = 72.7 (NOT 9.17)

### 2. Trader narrative form (reasoning layer) — vol-points
- **Formula:** `VRP_pts = iv30 - rv30` (in vol points, e.g. 9.2 vp)
- **Used in:** Trades DB Remarques, Sharpe2 strategy reasoning, conversational analysis
- **Why this form:** vol-points are the natural scale for **safety margin when selling premium** — it answers "how many vol points of cushion does IV give me over what the stock has actually been doing?" Log-ratio normalizes but loses the directly-interpretable cushion magnitude.

## Read pattern

When reading data, the persisted `vrp` is log-ratio. Always.

When generating analysis or commentary in trader-facing language (especially anything touching Sharpe2 logic, premium-selling, or "how much margin does this give me"), prefer the vol-point form — but **compute it explicitly from the raw `iv30` and `rv30` columns**, do NOT pretend the stored `vrp` is in vol points.

For sign-only statements ("negative VRP = options cheap"), both forms agree at the iv30=rv30 boundary, so qualitative bull/bear surface reads stay valid under either form.

## Trap to avoid

Reading the scanner CSV `vrp` value as vol-points produces wildly wrong reads. e.g. DBA vrp=72.7 is "options very rich vs realized" (log-ratio top end), not "IV exceeds RV by 73 vol points" — the actual vol-pt gap is ~9.2.

## Related

- `Tdata/CHANGELOG.md` 5.10.3 entry documents the log-ratio choice
- `RStudies/.../vol_profile.R` line ~100 has the load comment
- `.claude/commands/analyze.md` was patched 2026-04-27 to remove conflicting "VRP = IV − RV vol points" language and explicitly distinguish the two forms (search for "log-ratio" in that file)
