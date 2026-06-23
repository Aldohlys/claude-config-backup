---
name: feedback_plain_language_no_jargon
description: "In docs, spell things out — avoid abbreviations, variables, and invented shorthand as much as possible"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5ecd340f-8024-44ae-bbb4-897cfde9f171
---

User dislikes obscure shorthand in trading docs: "Instead of using obscure expressions like small-N write instead small sample size. Avoid using abbreviations, variables or specific expressions as much as possible." (2026-06-15)

**How to apply:**
- No invented shorthand: "small-N" → "small sample size".
- No bare variables in prose: removed `N` from pay-for-entry ("sell 1 of N at N×" → "sell one contract once it reaches a multiple of entry equal to the number you hold") and from table column headers ("N" column → "Sample size"). A defined variable inside an actual formula/code block is OK.
- Expand lazy abbreviations: WR → win rate, R/R → reward/risk, DD → drawdown, MM → market maker, BS → Black-Scholes. Done across both BOT plans 2026-06-15.
- Standard technical-indicator abbreviations (ATR, IV, IVP, EMA, OBV, RSI, ADX, DTE, OTM, OI, COT, ATH) are domain-standard — ASK before expanding those rather than auto-bloating; user may want them kept.

**Why:** clarity for trade-triggering; durable writing-style preference. See [[feedback_minimal_bold_formatting]], [[feedback_concise_no_repeated_phrases]].
