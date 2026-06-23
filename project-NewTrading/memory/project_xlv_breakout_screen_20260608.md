---
name: project_xlv_breakout_screen_20260608
description: "XLV healthcare component breakout screen 2026-06-08; ABBV = vol-sell candidate (not breakout) pending /analyze, JNJ dropped"
metadata: 
  node_type: memory
  type: project
  originSessionId: 2bd4384e-3b59-46f3-a4cf-4b6e7212fea0
---

Screened XLV top components via BOT gates on live bars (`Strategies/Breakouts/xlv_screen.py`), 2026-06-08. Thesis = defensive rotation ("liquidity to healthcare if not the AI trade").

**Conclusion — the XLV "breakout" is NARROW and not yet confirmed at the index level:** XLV \$152.79 is −4.2% *below* its 52w high (\$159.54); 3m sector return +0.2%. The move is levitated by **LLY** (16% wt, at new ATHs) + **UNH** (recovery). ~77% of components (everything below LLY/UNH) show **negative RS vs XLV** and sit below their own highs — not a broad sector bid. A confirmed XLV breakout needs a close >~\$159.5; otherwise it's "approaching resistance," not through it.

Per-name:
- **LLY** — true breakout (new ATH, no overhead) but +14% above EMA50, RSI 71 = extended, no asymmetry. User rejected chasing.
- **UNH** — strongest score (S5/BK3) but a recovery rally off the 2025 collapse: +74% off an old based low yet still −30% below ATH (heavy overhead supply). Also extended (+12.7%, RSI 71).
- **JNJ** — NOT a breakout (below highs, neg RS, EMA50 flat/rolling). Low-beta (~8% payable-move freq) → poor long-call vehicle. **Dropped.**
- **ABBV** — `/analyze ABBV long` run 2026-06-08 (`Reports/analyze_ABBV_20260608.html`). **Live data killed the vol-sell premise:** IVP **57 (mid)**, NOT the stale DB ~97 — there is no fat premium to harvest. VRP only modestly positive (+3.6 vol pts / log +13.5), skew puts-bid (cheap_side short). Earnings 2026-07-30 → Jul-17 (39 DTE) expiry clean. Stock had run +12% in 20d, near range highs (89%, RSI 69); **long breakout PRICED OUT** (R:R 0.31, chain "crowded"/pinning at 230 OI cap; best debit vertical 240/250 is a 15.6%-prob lottery).

**RESOLVED — pass, or tiny size.** Neither lens compelling: long breakout priced out, vol-sell edge muted by mid IVP (the rich-IV thesis was a stale-data artifact). Only mildly favorable signal = puts-bid; if-anything = small put-sale into 210 OI support, Jul-17. JNJ already dropped. **Healthcare screen closed** — no clean trade now (LLY/UNH extended, JNJ low-beta, ABBV edge evaporated). Lesson: confirm IVP live before calling a name a vol-sell — DB IVP drifts stale fast. See [[reference_atr_move_multiples]], [[feedback_classify_breakout_vs_meanreversion]], [[feedback_structure_selection_vs_vol_thesis]]. Sibling screens: [[project_refiner_bot_screen_20260606]], [[project_xlf_breakout_screen_20260608]].
