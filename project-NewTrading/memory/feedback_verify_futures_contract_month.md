---
name: feedback-verify-futures-contract-month
description: "For commodities with steep calendar curve (CL, oil), front-month and back-month prices can diverge by $4+. Always confirm WHICH contract month the user holds before quoting \"current price\"."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 243c14ae-fe86-4883-bf49-ab3946ae5c8e
---

When discussing an open futures position in a commodity with a non-flat curve, the "current price" depends critically on which contract month is held. Don't pull front-month spot and assume it represents the user's contract.

**Concrete error (2026-05-14):**
- User: "current MCL trade"
- I pulled `CL=F` (front-month proxy) → $101.48
- User correction: "MCL JUL26 quotes around $97.03"
- Front/Jul backwardation = $4.45 (crisis-level per [[reference-wti-calendar-spread-calibration]])
- My unrealized P&L estimate was off by $445 per contract until corrected

**Why:** WTI in mid-2026 has steep backwardation; oil curve dislocations are exactly the kind of regime where this error matters most. The user's contract was lagging the front-month spike by ~60% of the move. All R/R math depended on the correct price anchor.

**How to apply:**
1. Before quoting a current price for an open futures trade, identify the **specific contract month** from the DB (`Instrument` field) or ask explicitly.
2. For commodities with calendar structure (CL, NG, ZC, etc.), do NOT use `=F` ticker (front-month continuous) as a proxy without confirming it matches the held contract.
3. If only front-month data is available, **state the gap explicitly**: "front WTI is at $X, your JUL contract may differ — please confirm".
4. Watch for memory `[[reference-wti-calendar-spread-calibration]]` bands: if curve is in stress/crisis territory, the front-vs-held divergence is the dominant risk factor in the trade.

Related lesson: a position quoted at front-month prices but stopped at back-month prices will systematically overestimate unrealized P&L and underestimate stop distance.
