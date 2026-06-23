---
name: feedback-hedge-wing-sale-timing
description: "For long-put hedges, time the wing-sale (convert to bear put spread) into a vol spike — never at trough vol; if entry IV was elevated and has fully crushed, holding the residual vega is asymmetric upside."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 350b1975-d6ea-43f7-9086-8a8012f5cbc2
---

For long-put portfolio hedges (SPY, ESTX50, SPX, etc.), the "convert naked-long to bear-put-spread by selling a further-OTM wing" maneuver is a **vol-timing decision**, not a structural one. The wing-sale collects 3-4× more premium during a vol spike (wing IV 38-42% with skew steepening) than during a calm regime (wing IV 28-29%). Pre-selling the wing at trough vol monetizes optionality at its weakest point.

**Default operational pattern for index put hedges:**
- **Default state:** hold the naked long. Accept theta + delta-on-rallies as carrying cost.
- **Triggered state:** convert to spread by selling the wing only when a defined underlying level AND wing-IV threshold are both met (use a two-condition AND, not just spot or just IV).
- **Trigger discipline:** stage limit orders in TWS in advance; under panic, traders typically freeze or sell too early at the first vol pop.

**Two related rules for hedges that have already bled:**

1. **Don't crystallize residual vega at trough vol.** If entry IV was elevated and has now fully crushed (e.g. ESTX50 5200P bought at own-IV 37% in late-March spike, now at 26%), the remaining mark is asymmetric upside on any vol re-expansion. Closing or converting at this point sells cheap vol — usually the wrong move unless sizing pressure forces it. The painful part of the loss is behind; the remaining mark = bounded forward downside.

2. **Sunk cost vs. forward-looking position.** Distinguish the unrealized loss (sunk, psychological) from the forward-looking position (residual mark = max additional loss). A position that's down 66% but has 9.3 vega and 127 DTE remaining is **not** the same position you bought — it's a much smaller, much cheaper position with different convexity. Treat it as the latter when deciding next steps.

**Why:** validated 2026-05-15 on the ESTX50 18-SEP-26 5200P trade — entry 26-Mar at IV 37% (during the late-March spike, an outlier event), now at IV 26% with –1,488 unPnL. My initial recommendation to convert to a 4700-wing spread *today* would have sold the wing at 28% IV (~45 EUR/share) for –450 EUR cash; the wait-and-convert strategy targets selling the same wing at IV 38-42% in a future stress event (~140-170 EUR/share, +950 to +1,250 EUR/lot extra value). User caught this. The right framing is: 5200P is now a long-vol position at cheap vol with bounded downside, not a depleted hedge to be unwound.

**How to apply:**
- When analyzing a long-put hedge that has bled significantly, first decompose the loss — if vega-driven (IV-crush) and IV has reverted to recent lows, default recommendation is HOLD, not convert/close.
- For "should I reduce the daily bleed" requests, push back on the framing if reducing bleed means selling the residual long vega at trough IV.
- When setting wing-sale triggers, always require both an underlying-level condition AND a wing-IV (or wing-mid-premium) condition — single-variable triggers fire on grind-down-without-vol-spike scenarios that don't capture the edge.
- For sizing the wing strike on the trigger plan: moderate-stress trigger uses a closer wing (e.g. –20% moneyness); sharp-stress trigger uses a further wing (e.g. –25% moneyness) to preserve more downside convexity in a deeper crash.

See [[reference-u1804173-iv-column]] for the per-option vs underlying-iv30 attribution gotcha that initially obscured the vega story on this trade.
See [[feedback-structure-selection-vs-vol-thesis]] for the inverse case (when verticals discard vol edge for outright thesis).
