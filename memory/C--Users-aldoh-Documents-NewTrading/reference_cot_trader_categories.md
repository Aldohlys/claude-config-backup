---
name: reference_cot_trader_categories
description: "COT legacy vs disaggregated trader categories — the exact mapping, and the spreading fold that makes Commercial gross figures differ"
metadata: 
  node_type: memory
  type: reference
  originSessionId: eceac09b-8ed5-4491-b373-20a43b99833d
  modified: 2026-09-03T10:35:32.374Z
---

The CFTC publishes the same positions under two vocabularies. They reconcile exactly, so a disaggregated pull answers legacy questions too — no second download.

**Mapping (verified to the contract on WTI, gold, copper, corn, soy, SRW wheat, week ending 2026-08-25):**

| legacy | = disaggregated |
|---|---|
| Commercial | Producer/Merchant/Processor/User + Swap Dealers |
| Non-Commercial ("large spec") | Managed Money + Other Reportables |
| Non-Reportable ("small spec", retail) | Non-Reportable |

**The one trap — spreading.** The legacy report gives Commercial *no* spreading column, so a commercial trader's spread position is distributed into BOTH gross legs; Non-Commercial keeps spreading as its own column. So to reproduce published legacy gross long/short you must fold the swap-dealer spread into commercial long and short (and into their week-on-week), and must NOT fold for large spec. Net is unaffected either way — only the gross figures move. WTI 2026-08-25: naive sum gave commercial 765,818/922,064; folding the 126,326 swap spread gives the published 892,144/1,048,390.

**Why it matters for reading a screen:** "large spec" is not "the fast money". On gold 2026-08-25 it was 243.3k net long, of which Managed Money was 144.7k and Other Reportables 98.6k — 40% of the headline number is a grab-bag whose crowding says little. Prefer Managed Money when the question is speculative positioning; see [[reference_cotsignal_api_rejected]], which fails exactly here.

Implemented in `RStudies/reports/macro_context/refresh_cot.R`, which emits both vocabularies to `NewTrading/Reports/cot_actors_latest.csv`. Related: [[reference_cftc_cot_urls]], [[project_positioning_r_automation_todo]].
