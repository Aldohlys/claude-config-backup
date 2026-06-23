---
name: feedback-pyramid-per-lot-rr-audit
description: "When proposing a pyramid add (Lot N) under a global risk cap, compute Lot N's standalone R/R before recommending. If <1:1, the global-stop structure is forcing negative expectancy onto the new unit."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 243c14ae-fe86-4883-bf49-ab3946ae5c8e
---

When the user pyramids into a winning trade (adding lots above the original entry), always audit the **standalone R/R of the new lot** before recommending the structure.

**The math trap:** A global stop sized to cap total system loss at $X will force the latest add (which has the highest cost basis) to absorb the largest dollar loss at that stop. If the add's reward-at-TP is small relative to its loss-at-stop, the marginal unit has negative expectancy even though the system R/R looks fine.

**Concrete example (MCL Jul26, 2026-05-14):**
- 2 lots already long @ avg $95.30, stop $93.70
- Adding Lot 3 @ $99 with global stop $94.87 (math-required for $500 system cap)
- Lot 3 risk: ($94.87 - $99) × 100 = -$413
- Lot 3 reward at $102 TP: ($102 - $99) × 100 = +$300
- **Lot 3 standalone R/R = 0.73:1 — negative expectancy even at 100% win**

**The fix:** Use a **per-lot structural stop** on the add (anchored to a chart level near its entry), not the global system stop. In the example, Lot 3 stop at $97.50 gave 2:1 R/R standalone and kept system risk at $470 (under cap).

**Why:** The marginal unit must clear its own R/R hurdle. Letting the system R/R "absorb" a negative-expectancy add cannibalizes the existing position's edge.

**Why:** User caught that the global-stop design was structurally broken and switched to per-lot stops. Validated the audit as the correct first check.

**How to apply:** Whenever proposing "add lot N with stop S", compute (S - entry_N) × multiplier vs (TP - entry_N) × multiplier for that lot alone. If R/R < 1:1, redesign — either per-lot stop near entry, or don't add. Don't hide the negative-expectancy add inside a flattering system R/R number.

Related: [[bot-strategy-checklist]] pay-for-entry exit, [[reference-wti-calendar-spread-calibration]] backwardation context for futures pyramids.
