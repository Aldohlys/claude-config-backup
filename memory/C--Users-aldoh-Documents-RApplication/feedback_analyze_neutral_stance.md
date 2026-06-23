---
name: feedback_analyze_neutral_stance
description: /analyze must present bare facts only — no recommendations, no GO/NO-GO verdicts, no buy/sell/wait suggestions; let the user judge
type: feedback
originSessionId: 374e2580-c4c8-4600-b2d2-140902429eb7
---
In `/analyze` output (chat summary AND HTML report), adopt a strictly neutral stance. Present the facts of the analysis (Gate 1/2/3 pass/fail per criterion, computed values, scores, flags, vol metrics, earnings proximity, etc.) without recommending an action.

**Why:** User makes the trading decision themselves. Earlier `/analyze` outputs leaned toward GO/NO-GO verdicts, "consider taking the trade", "wait for confirmation", or similar editorial framing. The user wants the report to be a data sheet, not advice.

**How to apply:**
- **No verdict line.** Don't write "GO", "NO-GO", "WATCH", "TRADE", "WAIT", "PASS overall", "FAIL overall", or any equivalent overall recommendation. Per-criterion ✓/✗ is fine — that's a fact about the criterion, not advice.
- **No action verbs directed at the user.** Avoid "recommend", "suggest", "consider entering", "avoid this trade", "hold off", "looks promising", "looks weak", "I would", etc.
- **No editorial adjectives.** Drop "strong setup", "clean breakout", "concerning", "favorable", "unfavorable" framing. Stick to observable values: "RS rank 12/100", "ATR distance 0.4", "BOT_Score 6/10".
- **Notes stay descriptive.** A note like "near 20d low" is a fact; "near 20d low — risky entry" is advice. Drop the second clause.
- **Scope:** Applies to the chat summary, the HTML header, every section, and the conclusion. Still preserves the report (per `feedback_analyze_html.md`) — just neutral in tone.
- The user will form their own GO/NO-GO from the facts.
