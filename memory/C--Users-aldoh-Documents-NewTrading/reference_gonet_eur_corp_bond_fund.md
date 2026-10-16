---
name: reference_gonet_eur_corp_bond_fund
description: "Gonet \"Euro bond fund\" IE00B67T5G21 is EUR investment-grade CORPORATE credit (dur 4.36, 49% BBB), not govies — no crash ballast"
metadata:
  node_type: memory
  type: reference
  originSessionId: 250890ed-cd37-47c6-bd39-228637ce7645
  modified: 2026-10-16T04:46:13.295Z
---

IE00B67T5G21 (Gonet line, Yahoo proxy 0P0000VJRQ.F) = iShares Euro Investment Grade Corporate Bond Index Fund (IE) Inst EUR, accumulating, OCF 0.15%. Benchmark Bloomberg Euro Corporate (same as ETF IEAC, IE00B3F81R35). Factsheet Sep-2026: effective duration 4.36y, YTM 4.34%, ratings 49% BBB / 43% A / 7.5% AA. Benchmark sector mix (from IEAC sheet): banks ~31%, top issuers French banks (Crédit Mutuel, BNP, BPCE, CA, SocGen).

Older reports (ESTX50 hedge doc, Gonet hedge report 09-30) call it a "Euro bond fund" and treat it as duration — wrong framing. It is spread risk: in crashes government-yield rally is offset by spread widening (COVID −7.8%, 2022 −13.6% calendar / −14.7% Jan-Sep). Weekly regression on US10y/ESTX50 is weak (R² 0.45). Especially exposed in a French/European bank scenario.

**How to apply:** in Gonet stress tests model it as credit (episode moves or duration + spread duration ≈4.4), not as a deflation hedge. Source of the crash simulation: [[project_gonet_crash_simulation_20261016]].
