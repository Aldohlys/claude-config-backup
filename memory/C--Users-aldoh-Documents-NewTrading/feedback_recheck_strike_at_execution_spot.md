---
name: feedback_recheck_strike_at_execution_spot
description: "Re-check a protection strike against the LIVE spot at execution; a rally between analysis and fill thins a fixed strike's cushion"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ee50ec03-6504-49a0-a229-909d602ca8db
---

A strike chosen to deliver "–X% protection" at the analysis-day spot **drifts more OTM if the index rallies before you fill** — its cushion (moneyness, ITM-depth at the target drawdown) silently erodes. Always recompute the protection band at the **live execution-morning spot** and re-confirm the strike before sending the order.

**Why:** In the ESTX50 hedge (2026-06-02), the long leg was sized at 5,600 against spot 6,012 (6.9% OTM, ITM by 258 at the inflation –10%-portfolio point). Overnight the index rallied to 6,100 (+1.5%), pushing 5,600 to 8.2% OTM and only +183 ITM at that point — the cushion thinned right at the weakest-covered (inflation-grind) scenario. The user caught it ("is 5,600 high enough now?"); the fix was bumping the long leg to **5,700** to restore ~original moneyness. The protection *band* itself also shifts proportionally with spot (it moved from 4,940–5,342 to 5,014–5,417).

**How to apply:** on execution day, before quoting a ladder, (1) pull/confirm live spot, (2) recompute the target-drawdown band at that spot, (3) check the strike's %-OTM and ITM-depth vs the band, (4) if spot moved materially (>~1%), re-strike rather than execute the stale plan. Cousin of [[feedback_resurvey_stale_todos]] (re-survey before executing an old plan) and [[feedback_verify_futures_contract_month]] (verify the live reference before quoting). See [[project_estx50_hedge_roll_20260601]].
