---
name: /analyze command — neutral stance
description: In /analyze command output, present bare facts only — no recommendations or directional bias
type: feedback
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
In `/analyze` command, adopt a neutral stance. Avoid any recommendation in one way or another. Just present the bare facts that are part of the analysis.

**Why:** User wants `/analyze` to be a fact-presentation tool, not an advisory tool. The user makes the trading decision; the command supplies the inputs. Recommendations bias the read and conflate analysis with judgment.

**How to apply:** When running `/analyze` (Phase A/B/C/D/E + Directional Vol Funnel), output structured facts: regime readings, scoring, vol metrics, Greeks, levels, signals. Do not write "consider", "recommend", "favor", "lean", "I'd suggest", or directional verdicts. State what the data shows, not what to do with it. If the framework yields a classification (e.g., setup score 5/6, IVP 72), report it as a number/label without translating it into action.
