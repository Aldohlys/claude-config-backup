---
name: feedback-anchor-net-claims-on-reconciliation
description: "Before asserting the SIGN/direction of any net aggregate (exposure, P&L, balance), derive it from the conservation identity (total = sum of parts), not from a suggestive line-item display."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b86d6480-ddcc-4334-8f1c-778a329b3786
---

During the IBKR FX-exposure session (2026-06-02) I flip-flopped 4–5 times on whether the user was net long or short USD, because I kept reasoning from a displayed per-position daily P&L (the virtual USD.CHF short's −198) instead of anchoring on the account-total reconciliation (Net Liq 58,248 = Σ per-currency `Nt Lqdtn`, with the virtual FX positions provably excluded; total Unrealized = securities only). Each reversal eroded trust and the user had to push back repeatedly before it resolved.

**Why:** display/attribution columns (per-position DLY P&L, FX Cash, virtual FX positions) can mislead on direction; the balance-sheet identity that MUST hold is ground truth. The first turn I ran the `Net Liq = Σ Nt Lqdtn` check, the answer was unambiguous (net long).

**How to apply:** for any net/aggregate directional claim, first find the conservation identity that has to balance (account total = sum of parts), derive the sign from it, and state the proof. Don't flip the conclusion based on a single line item that "looks like" a position. If a display contradicts the reconciliation, explain the display as an artifact — don't let it overturn the identity. See [[ibkr_fx_exposure]].
