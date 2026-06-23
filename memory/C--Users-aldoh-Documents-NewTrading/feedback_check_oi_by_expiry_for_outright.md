---
name: feedback_check_oi_by_expiry_for_outright
description: "For an outright option, rank expiries by OPEN INTEREST first; the cheapest-IV chain can be a near-empty weekly"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 0e0bd681-94d6-415a-b2bc-412fd2f0cd23
---

When picking a single-leg outright (or any structure where you cross the bid/ask), rank
candidate expiries by **open interest before** optimizing on IV level or call skew.
Standard **monthlies (3rd Friday) carry the bulk of OI**; the surrounding weeklies are
often near-empty.

XOP 2026-06-08: the Jul10 *weekly* had the cheapest ATM IV (31.9% vs 33.2%) AND the
flattest call skew — but only **119 call OI** vs the Jul17 *monthly*'s **8,686 (~70×)**.
The ~1.3 vol-point IV edge is a mirage: the spread tax on a no-OI weekly dwarfs it.

**Why:** an outright pays the ask — liquidity (deep OI → tight spread → clean leg-in/out)
beats a small IV/skew saving.

**How to apply:**
- Pull `get_chain_oi` per candidate expiry (`tdata_py.chains_manager.get_chain_oi(sym, expiration, strike_min, strike_max)`), then pick the **deepest-OI strike in the monthly**. Don't anchor on the round number — XOP Jul17 \$180 had 39 OI while \$182/\$185 had 1,416/879.
- **/analyze's Phase A liquidity probe samples ONE expiry near 45 DTE.** If that lands on a thin weekly (XOP probed Jul24, 68 OI → a bogus 53% ATM spread), the wide-spread reading is about the *weekly*, not the name — re-probe the monthly before concluding "options too wide / trade stock".
- `option_skew_history` DB table is near-empty (7 unrelated names, 1 row each) → there is **no "call skew vs history"** available; fall back to **cross-sectional** skew across chains.

See [[feedback_analyze_vehicle_rule]], [[feedback_analyze_live_data_fallback]],
[[project_refiner_bot_screen_20260606]].
