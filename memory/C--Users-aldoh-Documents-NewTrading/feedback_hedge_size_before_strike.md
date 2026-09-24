---
name: feedback_hedge_size_before_strike
description: "For a portfolio hedge, settle coverage ratio (size) before debating strike/expiry; a token lot is the worst of the three options"
metadata:
  node_type: memory
  type: feedback
---

When the user challenges a portfolio hedge ("this is mild", "it has proven useless", "should I roll it?"), **compute the coverage ratio first and let it drive the answer — do not go straight to re-striking or rolling.**

Coverage ratio = max hedge payoff (in book currency) / the loss it is meant to offset, measured against the *block the hedge actually correlates to*, not the whole book. If it is in the low teens, no choice of strike or expiry changes the outcome, and the strike debate is a distraction.

**Why:** On 2026-09-21 the ESTX50 Dec 5,700/4,600 spread had drifted OTM for the third time and the instinct was to re-strike or roll out. The arithmetic settled it instead: max payoff 9,750 EUR (~9,240 CHF) against a 435k CHF book that is 60.7% equity with a ~195k CHF EUR+CHF equity block — **~12% of a crisis drawdown, ~17% of the European block.** One lot is a token. Rolling it forward would have re-bought the same token at a new strike and produced the identical complaint in three months. The binding roll rule ([[project_estx50_hedge_roll_20260601]]) already fixes the strike, so strike was never the open variable — size was.

Also separate the two things inside "useless": an untested crash hedge that did not pay during a rally is the *base case*, not a design failure (the payoff function was intact — a -20% month marked the spread at 13x the remaining premium). What is a real finding is the cumulative carry (~2,670 EUR over six months) set against a coverage ratio that cannot move the portfolio outcome.

**How to apply:** Before pricing any roll or re-strike, state (a) max payoff in book currency, (b) the correlated block's value, (c) coverage % at the realistic crisis move, (d) cumulative premium spent to date. Then force the binary: **size it so it matters, or close the line** — and pre-commit which, in the alert, so the decision is not re-litigated. The half-position costs real carry and cannot change the outcome; it is the worst of the three choices. If the answer is "close the line", the fallback is the standing framework — trim winners into strength and hold cash ([[feedback_hedge_wing_sale_timing]] is void; see the 2026-07-09 reframe in the project memory), which is already most of the job when the book carries heavy ballast.

Related: [[feedback_size_risk_before_flagging_materiality]] (sizes a *risk*; this one sizes the *hedge*), [[feedback_portfolio_drawdown_vs_index_move]] (translate the portfolio target through effective beta before picking the index level), [[reference_oesx_eurex_option_mechanics]].
