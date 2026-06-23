---
name: reference-analyze-tws-down-degraded
description: "/analyze in TWS-down degraded mode — pipeline does NOT dead-end; specific FETCH FAILED markers identify the TWS-closed case so it's not mistaken for a real data failure. BS-priced outright grid and structural targets still work."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 0bb4ef14-7deb-4012-92d7-6a1eda0ffb8a
---

When TWS is closed (or 127.0.0.1:7496 otherwise unreachable), the `/analyze` R pipeline keeps running and produces an HTML, but several fields legitimately return `FETCH FAILED` rather than synthesizing fake data. This is **expected degraded behaviour**, not a real failure — reading the report needs to account for it.

**FETCH FAILED markers that mean "TWS not connected":**

| Phase | Field | Failure string |
|---|---|---|
| C — Vol Funnel | RR 25Δ (risk reversal) | `FETCH FAILED: getOptValue empty for both wings on <YYYYMMDD>` |
| C — Cheap Score | RR alignment component | `n/a` (scored 0) |
| D — Chain | oi_cap_call | `FETCH FAILED: get_chain_oi: ConnectionError: Not connected` |
| D — Chain | oi_cap_put | same |
| D — Chain | chain_state | same |
| D — Chain | entry_state | `NO CHAIN` |
| D — Spreads | Vertical spread table | `FETCH FAILED: compute_spread_risk_reward returned no rows for any width` |
| Data Summary | Structures within cap | `0` |

**What still works in this mode:**
- IV30, IVP, RV30, RVP, VRP (log + vol-pts) from Tdata DB cache.
- Term-structure IV30/IV90, sector RS, MA50/squeeze/OBV/RSI/range-pos technicals.
- Structural targets (prior swing high / 52w / round / Fib) — derived from OHLC, not TWS.
- BS-priced **outright option grid** — entries at current spot/IV30, forwards at effective target with `IV +2pp` and `DTE − 5d` theta buffer. R:R numbers in that grid are real, just derived from BS not from chain mids.
- Cheap-score components other than RR (so the score caps at 8/9 rather than 9/9).

**How to read a TWS-down /analyze:**
- `entry_state = NO CHAIN` + `Structures within cap = 0` ≠ pipeline failure. It means: "rerun with TWS open to populate cap-compliant verticals and chain OI."
- The outright-grid R:R is still valid for ranking-by-time-axis and lot-sizing-vs-cap, just without the smile/mid-tick precision of live-IBKR.
- If the user wants the chain-anchored numbers (oi_cap, chain_state crowded/capped/open, vertical R:R), prompt them to open TWS and rerun. Don't synthesize the chain side.

**Example (2026-05-12 GM short, TWS closed):** Phase B SKIP on sector rank, C PASS with cheap_score 6/9 (RR slot scored 0 from FETCH FAILED), D SKIP with R:R 0.53 + NO CHAIN, E SKIP phase_of_drop=B. Outright put grid populated with 8 rows; vertical spread table empty.

Related: [[feedback-analyze-live-data-fallback]] — the "no n/a cells" rule applies to *fetchable* data; it does not require synthesizing values when the upstream service is offline.
