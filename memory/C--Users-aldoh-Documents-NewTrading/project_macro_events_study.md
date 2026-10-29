---
name: project_macro_events_study
description: "2026-10-20 event study: FOMC/Mag 7 earnings are not a macro force; FOMC = modifier (FOMC-made crossings reverse) + vol overlay; Mag 7 = leadership vol only; ported to macro_context (RStudies 4956bd5, pushed)"
metadata:
  node_type: memory
  type: project
  originSessionId: 61f1c2d1-088a-42b7-bdf0-d5e79120cc6a
  modified: 2026-10-20T20:56:43.362Z
---

On 2026-10-20 the user asked whether scheduled events (FOMC, NVDA/Mag 7 earnings) are macro forces or relate to the 7 forces (real yield, oil, dollar, gold, rotation, VIX, QQQ/RSP leadership).
- Script: `NewTrading/scripts/macro_forces/event_study.py` (`fetch` then run, ~5 min).
- Data: `scripts/macro_forces/events/`.
- Report: `Reports/macro_events_study_20261020.md`.

**Verdict**
- Not a force.
- FOMC is a modifier. When the decision day + next session make ≥50% of a 20-session |z|≥1 crossing, the move continued 36/35/34% at 5/10/20 sessions, vs 47% under a shifted calendar (p≈0.04; 137 crossings from 59 decisions). The effect is strongest for gold, VIX and the dollar.
- FOMC is also a vol overlay:
  - SPY realized vol is 1.17× higher in the 5 sessions after a decision than in the 5 before (p≈0.002).
  - Big 2-day SPY reactions continued over 20 sessions only 31% of the time vs 52%.
  - No pre-FOMC drift; the decision-day gain only shows in 2008–2015.
- Mag 7 earnings matter for leadership volatility only:
  - QQQ/RSP moves 98 bp on reaction days vs 69 bp (2023–26).
  - No forward direction.
  - Earnings-driven leadership crossings do not reverse.
- NVDA big-reaction test is inconclusive (16 events; 4 since 2023).

**Ported to macro_context (RStudies `4956bd5`, branch claude/macro-context-main-detail, pushed 10-20)**
- `event_history.R`: FOMC and Mag 7 rows for the next 28 days, with `boost = FALSE`.
- Summary tiles get a "FOMC-made" tag.
- All `events.R` action texts were rewritten as neutral "Not tested." descriptions.
- f17834e (main, pushed): the scenario `bot$note` texts and the yen-carry BOT text now cite the closest step-2 7-force analog (hit vs base, Fridays, episodes) or say "Not tested."; the label became "BOT, what followed historically.". Only stagflation (XLY<XLP 75% vs 46%) and goldilocks (QQQ up 79% vs 66%) show an edge.

**Why:** feeds the forces → scenario → trades redesign ([[project_macro_context_redesign]]). The history-only rule is in [[feedback_macro_context_no_advisory]].

**How to apply:**
- `FOMC_DATES` in event_history.R is hardcoded to Dec 2027. Extend it when the Fed publishes the 2028 calendar, or the FOMC rows and tile tag go silent.
- The study's conclusion table uses do/don't wording. Never copy that into macro_context; use the measured lines only.
