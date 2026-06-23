---
name: reference-vertical-spread-exit-80pct
description: "For vertical debit spreads, realistic exit is ~80% of theoretical max — the last 20% requires 0 DTE and pin risk"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 83012921-786f-469a-b241-73e9f15b5317
---

For a vertical debit spread (puts or calls), the practical exit is **~80% of max profit**, set as a GTC sell-to-close limit at trade open.

**Computation:**
- Max value at expiry = width × 100
- 80% target price = entry_debit + 0.80 × (max_value − entry_debit)
- Example: 10-wide debit spread entered at \$2.76 → 80% target = \$2.76 + 0.80 × \$7.24 = **\$8.55**

**Why ~80% and not 100%:**
- Last 20% of premium accrues mostly at/near 0 DTE
- Market makers won't quote full intrinsic when meaningful DTE remains — they need theta and optionality buffer
- Holding for the last 20% means holding through:
  - Pin risk (price drift back across short strike)
  - Negative gamma near expiry
  - Falling vega (no more vol expansion benefit)
- Trade-off: ~25% more potential reward in exchange for ~10× the path risk

**When the 80% price actually prints:**
- (a) Price well past short strike with 1–2 weeks DTE: intrinsic at max, time value compressed
- (b) Deep ITM penetration (~10% below short strike): both legs lose most time value
- Path (a) is more realistic for typical swing-trade outcomes

**Pairs with [[feedback-hard-to-exit-winners]]:** drop the GTC limit at fill — removes the active-choice tax when the position pays.
