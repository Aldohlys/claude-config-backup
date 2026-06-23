---
name: feedback_portfolio_drawdown_vs_index_move
description: "Translate a \"-X% portfolio\" hedge target into the index move via the book's equity/ballast mix before striking — they are not the same"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ee50ec03-6504-49a0-a229-909d602ca8db
---

When a hedge objective is stated as "protect against a **–X% drawdown of the *portfolio***," do **not** strike the index hedge at –X% on the index. Translate first: defensive ballast (bonds, gold, cash, low-beta names) dilutes the equity move, so a –X% *portfolio* drawdown needs a **larger** index decline — and the multiple is **regime-dependent**.

**Why:** GOnet (2026-06-02) was only **67% equity** (33% bond+gold ballast). A –10% *portfolio* move = equity **–11%** (inflation grind — bonds fall *with* equities) to **–18%** (crisis risk-off — gold+bonds cushion). Striking the ESTX50 put at the –10% *index* level (5,490) would have mis-struck it; the real protection band was **5,014–5,417**. The 5,200 strike looked fine against "–10%" but paid ~nothing at the inflation-grind portfolio –10% point — which drove the roll to a 5,700 long leg.

**How to apply:** (1) decompose the book into equity vs ballast (% and currency); (2) build regime cases — what do bonds/gold do in each? — to get the equity-move-per-portfolio-move multiple; (3) strike the hedge ITM **across that band**, not at the headline %; (4) state coverage as a % (it's partial and regime-dependent). Also: an equity put can't hedge the ballast's own losses (bonds in an inflation grind) — see [[feedback_size_risk_before_flagging_materiality]]. Applied in [[project_estx50_hedge_roll_20260601]].
