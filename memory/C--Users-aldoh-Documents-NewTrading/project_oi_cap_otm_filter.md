---
name: oi-cap-otm-filter
description: /analyze OI caps now filter to OTM strikes + thin-chain bypass (shipped 2026-05-27 from XOP run)
metadata: 
  node_type: memory
  type: project
  originSessionId: 8c65e4a9-c01b-4a86-95fe-f6292e3db0e8
---

# /analyze OI cap — OTM filter + thin-chain bypass (2026-05-27)

Shipped in `RStudies/reports/shared/live_sources.R::.summarize_oi()` after XOP long run surfaced the bug.

## What changed

- `oi_cap_call` now restricted to strikes **> spot** (real resistance from OTM call writers).
- `oi_cap_put` restricted to strikes **< spot** (real support from OTM put hedgers).
- If max OTM OI on a side < `thin_oi_threshold` (default 100), that side's cap is `NA`. Caller falls back to structural target.
- `chain_state = "thin"` when both sides bypass.
- Concentration metric (top-3 / total) computed over OTM portion only.
- Threshold is config-driven: `default.analyze.thin_oi_threshold` in `RStudies/config.yml`.

**Why:** XOP long run produced `oi_cap_call = \$135` with spot \$166 (18.7% ITM — was stock-replacement / covered-call cover, not resistance). Pipeline then capped `effective_target = \$135` *below* entry for a long trade, giving R:R = -0.97 and meaningless PRICED OUT.

**How to apply:**
- When reading future /analyze reports: `chain_state = "thin"` and NA caps now mean "chain has no usable OI signal" → structural target is the read. Don't try to interpret a missing cap as bullish/bearish.
- When debugging a "weird effective_target" result, first check whether the chain is thin (low OTM OI both sides) — that's now visible as `chain_state = "thin"`.
- For ETFs and small-caps with thin chains, the structural target now dominates (correct).
- Related: [[audit-direction-blindness-when-adding-shorts]] — same class of bug (long-authored shared rules silently producing wrong-direction results).

See [[xop-thin-option-chain]] for the specific case that surfaced this.
