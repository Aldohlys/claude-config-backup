---
name: project_gonet_crash_simulation_20261016
description: "Gonet 5-scenario crash sim 2026-10-16 — GFC −26%, 2011 −11%, COVID −20%, 2022 −18%, CHF shock −14%; 1-lot ESTX50 hedge covers 9-20%"
metadata:
  node_type: memory
  type: project
  originSessionId: 250890ed-cd37-47c6-bd39-228637ce7645
  modified: 2026-10-16T05:27:11.992Z
---

Report `Trades/Gonet_crash_simulation_20261016.md`, scripts `Trades/Gonet_crash_sim/` (crash_factors.py, betas.py, scenarios.py — rerun scenarios.py on a new snapshot). NLV 398.5k CHF (15-Oct snapshot).

Losses before hedge: A GFC −105k (−26.4%), B Euro/French 2011 −45k (−11.4%), C COVID −79k (−19.8%), D 2022 −71k (−17.8%), E SNB-type CHF shock −57k (−14.4%). ESTX50 Dec 5700/4600 × 1 pays ~9-10k in A-D, 0 in E. CH equity is the largest loss everywhere (SLHN down-beta 1.47, ABBN 1.28). Defensive 41% only works in deflationary crashes. Scenario B uses no-floor FX (EUR/CHF −10, USD/CHF −5), not the 2011 floor-distorted moves.

CHF hedge (report §8): euro-led EUR/CHF falls are offset by the USD holdings; rising-franc falls hit both (−25 to −36k). User view: franc won't move dramatically → 2 × CME 6S Jun-27 1.25/1.30 call spreads (≈2.9k CHF mid on 16-Oct, full 9.6k at USD/CHF −7%), bought with 3 ESTX50 Jun-27 lots on 13-Nov. SLHN second trim rejected (saves only 0.6–2.0 pts). TWS port 7496 serves paper/sim; some FOP strikes blocked by "competing live session" — price them from TWS model IV.

**Why:** user asked for a crash simulation on Gonet; feeds the 13-Nov-2026 2027 hedge sizing decision ([[project_estx50_hedge_roll_20260601]], [[project_gonet_hedge_report_20260930]]).
**How to apply:** at the 13-Nov review compare hedge payoff to these losses; CHF shock is unhedged by any equity put. Related: [[reference_gonet_eur_corp_bond_fund]], [[reference_gonet_table_mktvalue_native_ccy]].
