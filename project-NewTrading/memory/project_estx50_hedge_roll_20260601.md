---
name: project_estx50_hedge_roll_20260601
description: ESTX50 GOnet hedge — pending roll Sep 5200P → Dec 5600/4600 spread; supersedes wait-and-convert
metadata: 
  node_type: memory
  type: project
  originSessionId: ee50ec03-6504-49a0-a229-909d602ca8db
---

ESTX50 GOnet portfolio hedge. Decided 2026-06-01 (spot 6,012); **EXECUTED 2026-06-02** at the EUREX open (spot 6,100 after +1.5% overnight rally).

**Executed:** closed Sep-18-26 5,200P @42.7 (realized –1,813 EUR), opened **Dec-18-26 5,700/4,600 put spread @125.0 net** (net roll debit 82.3 pts = 823 EUR/lot). Long leg bumped 5,600→**5,700** because the higher tape thinned the inflation-end cushion (the §7-flagged tweak). **Position: long Dec 5,700/4,600 spread, basis 1,250 EUR/lot, breakeven 5,575, max gain +9,750 (≤4,600), cap –24.6%, net delta ~–0.21.** Monitor: ESTX50 ~5,417 = –10% portfolio zone (monetise into VSTOXX spike ≥25-30); manage before Dec-18.

December-chain diligence: Dec-18 is the *only* December OESX expiry and the most liquid (quarterly: 1.7-pt spread vs Nov 2.0 / Jan 3.1); term IV flat ~18.5% so rolling out costs nothing on a vol basis. OESX quoted in index points × 10 EUR/pt multiplier.

**Forward management / time boundary:** protection ends **18-Dec-2026.** User's feared catalysts = European shocks (Russian escalation; French far-right + disorder). H2-2026 shocks, or markets *front-running* the French risk in late 2026, are covered. But the French presidential cycle (~April 2027) and any "early-2027 disorder" land **after expiry** → **roll forward to a 2027 expiry (Mar/Jun-27) as Dec approaches** if the thesis is still live (already the standing instruction in alert id=62). Two event shapes: sudden shock → monetise into the VSTOXX spike; slow grind → hold to expiry (deep 4,600 wing uncapped to –24.6%). Doc has Options A–D (Greek labels dropped); executed = Option D. Lessons captured: [[feedback_portfolio_drawdown_vs_index_move]], [[feedback_recheck_strike_at_execution_spot]], [[feedback_size_risk_before_flagging_materiality]], [[reference_oesx_eurex_option_mechanics]], [[reference_closed_market_option_marks_stale]].

**Why γ over alternatives:** deep 4,600 wing keeps the cap below the 2022 trough (~–26%), so uncapped across the grind the user fears; beats naked everywhere down to ~–24% (same long leg, cheaper). Near-wing β (5600/5000) is capped at –16.8% → fails the 2022-grind scenario. Naked owns crashes (vega) but bleeds most if benign. Dynamic "roll wing down in a crash" is dominated (no skew trigger in a grind; in a crash = γ minus slippage). Full sim + quotes: `Trades/ESTX50_hedge_management_20260601.md`.

**Portfolio coverage finding:** GOnet ≈ 371k CHF, only **67% equity** (52% EU/CH, 12% US, 4% China) + 33% defensive (21% bonds, 12% gold). So a –10% *portfolio* move = equity –11% (inflation, bonds fall too) to –18% (crisis). 1+1 hedge = **partial convexity, ~6–24% coverage** of a –10% drawdown. Inflation grind is structurally hardest (bonds fall *with* equity, equity put can't hedge them). **Bond gap is second-order:** sleeve loses ~–9.7k CHF in a 2022 replay (~2.6% of book); DTLA alone is only 3.9% of book → ~–1.1% pain — biggest duration exposure is the Euro-bond ETF (IE00B67T5G21, 9.8%), not DTLA. Un-actioned on purpose: DTLA's long duration is a cushion in normal/crisis risk-off (rates fall, +15-20%); cutting it is a regime bet, not a hedge. [[feedback_size_risk_before_flagging_materiality]]

**SPY 660P parked** (US only 12% of book) but same 18-Sep expiry — needs its own roll-up-and-out before Sep (alert id=61, 2026-08-15).

**Supersedes** [[feedback_hedge_wing_sale_timing]] wait-and-convert plan (`ESTX50_hedge_management_20260514.md`) — void: market rallied, no spike, no trigger fired. Alerts fixed: id=52 (stale SPY 580P) deactivated; 60→OESX-ROLL exec, 61→SPY-ROLL, 62→OESX-MONITOR.
