---
name: feedback_counterparty_positioning_lens
description: "User values the \"who is on the other side\" market-structure lens (counterparty + positioning/COT); do not dismiss it as inert"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5ecd340f-8024-44ae-bbb4-897cfde9f171
---

When reviewing the BOT trading plan (2026-06-15) I cut the MM-vs-retail "counterparty" preamble as "operationally inert." User pushed back: understanding **who is on the other side** (market maker vs institutional vs retail) is a core, deliberate way they read markets — explicit in futures via the CFTC COT report. They were right; I restored it.

**Two distinct counterparty questions — keep them separate, both matter:**
1. **Execution counterparty** (who fills my order): single-name options ≈ always the MM (delta-hedged, prices off own vol surface; indifferent to direction). Explains why IV crush taxes exits and why a dead book just takes the BS mark. Maps to work-mid (court natural flow) vs hit-bid (accept MM vol mark) = the execution ladder. → lives in the **exit** section.
2. **Positioning counterparty** (who's already in the directional bet): COT for futures/commodity underlyings (commercials vs managed money vs small specs; managed-money extreme = crowded), dealer gamma / OI walls for options, short interest short-side. → lives at **entry** (Gate 4 crowdedness check).

**User's crowded-trade rule:** do NOT add to an already-crowded/consensus "easy" trade — marginal buyer exhausted, small catalyst → violent unwind. Template they cited: the **yen carry-trade unwind (Aug 2025, after a small BoJ rate hike)** — over-leveraged consensus carry detonated into a cross-asset reversal. (I initially flagged Aug 2024 as the canonical one; user confirmed they mean Aug 2025 — docs use 2025.)

**Why:** durable framework preference, not a one-off. **How to apply:** treat market-structure / positioning framing as first-class; when tempted to call it "philosophical," find the operational form instead of cutting it. See [[reference_cftc_cot_urls]], [[project_positioning_r_automation_todo]], [[project_bot_trading_plan_review]].
