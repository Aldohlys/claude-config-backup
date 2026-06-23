---
name: data-only-doesn-t-mean-stripping-mechanical-synthesis
description: "/analyze \"data-only\" rule bans prose verdicts and editorialization — NOT code-defined ranking, sorting, scoring, or filtering. The April 2026 neutralization went too far and made the tool useless."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3b70ddd8-ed66-422b-bfce-e9ef777ec479
---

The `/analyze` "data-only" feedback memories (`feedback_analyze_data_only`, `_neutral_stance`, `_no_verdict_card`) collectively banned: GO/NO-GO, conviction labels, edge narratives, Best/Alternative/Avoid rankings, "consider"/"favor"/"lean" prose.

**The 2026-05-12 redesign clarified the boundary:** the rule bans **prose verdicts**, NOT **mechanical synthesis defined in code**.

**Why:** the April-2026 neutralization over-stripped. By May 2026 `/analyze` produced "data" that was wrong-direction, missing fields, and mixed CREDIT+DEBIT junk rows — but everything was "neutral" because no narrative violated the rule. The user called it useless. Restoring code-defined sorting, filtering, scoring, and ranking made it useful again without re-introducing prose verdicts.

**How to apply:**
- ✅ OK: sort a structures table by `expected_value desc` (code, not prose)
- ✅ OK: filter to DEBIT-only for the trade direction (mechanical, deterministic)
- ✅ OK: rank sectors descending for long / ascending for short (code-defined)
- ✅ OK: a 0-9 cheap_score composite with explicit thresholds (mechanical)
- ✅ OK: PASS/SKIP per-phase classification (mechanical labels, not advisory)
- ❌ NOT OK: "this spread is the Best because of edge X" (prose verdict)
- ❌ NOT OK: "I'd lean toward the 30-DTE because..." (advisory language)
- ❌ NOT OK: conviction levels (HIGH/MEDIUM/LOW) (subjective synthesis)

Test: would removing this line make the report less informative without making it less neutral? If yes, keep it. If the line reads like a recommendation, drop it.
