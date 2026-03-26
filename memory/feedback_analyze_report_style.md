---
name: Analyze report HTML style
description: /analyze HTML reports must use colorblind-safe palette, DOW-style Gate 2 breakdown, consistent T/M/RS/V labels, terse notes, no WATCH/TRADE signal
type: feedback
---

Use DOW report (analyze_DOW_20260325.html) as the reference template for all /analyze HTML reports.

**Why:** User is colorblind (red/green/orange indistinguishable). Earlier reports used inconsistent criterion labels, overly verbose notes, and scanner WATCH/TRADE signal which is just a ranking cap.

**How to apply:**
- **Colors**: Colorblind-safe: PASS = Blue (#0072B2), FAIL = Vermillion (#D55E00), WARN = Amber (#E69F00), SKIP = Grey (#666). Never use green-bg/red-bg/orange-bg.
- **Labels**: Gate 2: T1-T4 (Trend), M1-M3 (Momentum), RS1 (RS), V1-V2 (Volume). Same for longs and shorts. Never S4a/S5a/F2a.
- **No WATCH/TRADE signal**: Scanner signal (TRADE/WATCH) is a ranking cap (top 3 = TRADE), not quality. Don't show it in /analyze reports. Only check it's not SKIP. Show score X/10 only.
- **M3 computed value**: Show actual ratio `(price - low20) / ATR` for shorts, `(high20 - price) / ATR` for longs. E.g., "0.4 ✗" with note "near 20d low". Say "ATR" not "1×ATR".
- **Header**: date+time on top, then `BOT Analysis: TICKER DIRECTION` (h1), then `Company · Sector (ETF) · Price` (subtitle)
- **Gate 2**: Decompose into 4 sub-categories. Show ✓/✗ per criterion with values. Use group-header rows and pass-light/fail-light backgrounds.
- **Gate 3**: Show Stage 2 criteria individually, then DOW pattern scoring, each with actual values.
- **Notes**: Only add if it provides NEW information not deducible from value/label. If value + ✓/✗ tells the story, leave empty.
