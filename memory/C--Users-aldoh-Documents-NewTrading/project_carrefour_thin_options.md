---
name: CA.PA Carrefour deactivated from scanner universe
description: Carrefour (CA.PA) Euronext Paris listing has thin options chain; removed from ScannerUniverse 2026-04-27
type: project
originSessionId: 53dab338-46f2-4aba-b466-d9cda0aa1007
---
CA.PA (Carrefour SA, Euronext Paris) was deactivated in `ScannerUniverse` table (set `IsActive = 0`) on 2026-04-27. It surfaced as a Phase B Pull-passer in the v5 scanner first run.

**Why:** Carrefour Paris-listed options have insufficient liquidity for the front-run-option-flow strategy. Wide bid/ask, sparse strikes, low OI per strike — the lens of "buy cheap, sell expensive but still buyable" requires a chain where another buyer will actually arrive at the exit price. Thin chains break that mechanism.

**How to apply:** When proposing scanner improvements or universe additions, exclude European single-stock listings (.PA, .DE, .AS, .SW, etc.) from the option-flow strategy unless explicitly verified to have liquid weekly+monthly options. The user's broader portfolio includes some Swiss/EU names (NOVN, ROG, KNIN, etc.) for stock holdings, but they should not enter the BOT/option-flow scanner unless their option liquidity is empirically demonstrated.

**Note:** The Phase A rich-options gate (Step A.1 — weeklies + ATM b/a ≤ 12%) would eventually catch this once the daily option fetch wires the bid/ask check. Until then, dropping from universe at the source is cleaner.
