---
name: feedback_analyze_html
description: /analyze command must always produce HTML report regardless of GO/NO-GO verdict
type: feedback
---

Always generate HTML report in Step 5 of /analyze, regardless of whether the verdict is GO, NO-GO, or CONDITIONAL.

**Why:** The user wants to preserve a record of what the decision was at each point in time. Even NO-GO verdicts are worth documenting for future reference and pattern analysis.

**How to apply:** After completing the terminal verdict (Step 4), always proceed to Step 5 (HTML generation) and write the report to `C:/Users/aldoh/Documents/NewTrading/reports/analyze_<TICKER>_<DATE>.html`. Open the file automatically if running interactively.
