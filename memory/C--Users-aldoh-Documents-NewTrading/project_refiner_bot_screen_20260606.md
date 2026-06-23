---
name: project_refiner_bot_screen_20260606
description: US refiner /analyze long screen 2026-06-06; resume DK/DINO BOT workup June 8
metadata: 
  node_type: memory
  type: project
  originSessionId: bd976eeb-0f33-429a-889f-ea458256c068
---

Screened 7 US refiners long via `/analyze` on 2026-06-06 (export-to-world theme): VLO MPC PBF DINO DK CVI PARR. Reports at `NewTrading/reports/analyze_<TICKER>_20260606.html` (EMA50 basis — see [[project_ma50_ema_switch]]).

**Conclusion:** PARR is NOT the BOT pick despite screening "best" on the optionality lens (cheap_score 5/9, IVP ~37% cheapest, best R:R) — it's below a falling EMA50 (−4.8%, slope −0.75%), setup 1/5, breakout 0/4, MISMATCH. That's the cheap-vol / anti-breakout signature, not a breakout. Crossed lenses: cheap IV = vol lens, not the breakout (S1/S2/BK) lens. See [[feedback_classify_breakout_vs_meanreversion]].

**Genuine BOT breakout setups (deferred — user will work out ~June 8):**
- **DK** — strongest: setup 5/5, breakout 3/4, ALIGNED, well above EMA50.
- **DINO** — setup 4/5, breakout 3/4, ALIGNED.
- Both have the mirror problem: rich IV (IVP mid-80s) → vehicle = stock or spread to neutralize vega, NOT outright long premium. Work up entry/structure next session.

The full export-refiner ticker universe (incl. Canada IMO/SU/CVE/PKI.TO, Africa Sasol SSL/SOL) is in this session's first answer if scope widens.

**RESOLVED 2026-06-08 → pivoted to XOP energy-long.** Re-ran DK/DINO live (TWS up): both poor *trades* despite ok charts — DK pass (chain-pinned @50, R:R 0.51, options 22-67% wide, IV rich), DINO softer (range pos 63%, mid-consolidation, not breaking). User reframed goal to **energy-sector long, not refiner-specific**, and flagged XLE's ~42% XOM+CVX / Middle-East concentration. Screened XLE/XOP/XES/CRAK live: **XOP wins** (US shale, no major concentration, cheap_score 4/9 = cheapest IV, IVP 66/VRP negative, 77 structures, target \$190 = retest highs). XES disqualified (stage=none/MISMATCH, richest IV). Plan saved: `Trades/XOP_breakout_call_plan_20260608.md` — triangle breakout PENDING, trigger cash-hold >\$172, vehicle **Jul17 \$185 calls** (monthly = 8,686 call OI vs 119 Jul10 weekly; \$185 for \$1-2 handle + leg in/out; don't use \$180 = 39 OI). Flat call skew to \$190 (no wing penalty), positions for skew-steepening vega kicker. See [[feedback_classify_breakout_vs_meanreversion]], [[feedback_structure_selection_vs_vol_thesis]].
