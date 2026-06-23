---
name: feedback_size_risk_before_flagging_materiality
description: "Quantify a portfolio risk as % of book before calling it material; don't conflate big %-move with big contribution"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ee50ec03-6504-49a0-a229-909d602ca8db
---

When flagging a portfolio risk (an unhedged sleeve, a tail, a "gap"), **size it as a share of the total book before asserting it matters.** Don't conflate a holding's large *percentage move* with its *contribution* to the drawdown — a long-duration bond can fall –30% yet, if it's only 4% of the book, that's ~–1% of the portfolio.

**Why:** In the 2026-06-01 ESTX50 hedge analysis I flagged "~21% of the book is bonds that fall with equities in an inflation grind — bigger lever is DTLA duration." The user pushed back: DTLA is <4% of GOnet, so its pain is ~–1.1% of book. I'd pinned the whole 21% sleeve on DTLA and never sized DTLA itself. The actual sleeve loss in a 2022 replay was ~–9.7k CHF (~2.6% of book) — real but second-order vs the ~–50k equity loss. The biggest duration exposure was actually a different holding (the 9.8% Euro-bond ETF).

**How to apply:** Before writing "X is the lever" or "this gap matters," compute (position value ÷ total book) and (estimated %-move × weight). State the contribution in absolute and %-of-book terms. Also check whether the "risk" is a *feature* in another regime (DTLA's long duration cushions a normal risk-off) — a one-regime cost is a regime bet to remove, not a free fix. Relates to [[feedback_verify_before_claiming_codebase_facts]] (size/verify before asserting) and [[project_estx50_hedge_roll_20260601]].
