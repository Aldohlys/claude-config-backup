---
name: feedback_no_tautological_signal_rows
description: "Macro→sector (and any condition→outcome) maps must use an upstream signal observable independently of the outcome, not restate it"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5ecd340f-8024-44ae-bbb4-897cfde9f171
---

User flagged self-referencing rows in the BOT plan's sector table as useless (2026-06-15): "Inflation/commodities rising → Basic Materials, Energy", "Gold rising → Precious Metals", "Consumer strength → Consumer". The condition just restates the sector's own price, so the row predicts nothing.

**Rule:** the left column of a condition→outcome map must be a LEADING signal observable *before and independently of* the outcome. If the condition is the outcome's own price restated, delete or replace it. Rewrote the table so conditions are upstream/causal: e.g. gold row → "interest rates falling while inflation stays elevated (falling real yields); dollar weakening; central-bank buying" → Precious Metals.

**Macro fact user emphasized:** a weak dollar drives commodities up because commodities are priced in dollars — so the dollar is a SHARED upstream driver across energy/materials/precious-metals rows. Note shared drivers as one theme, don't list them as independent signals.

**Why:** analytic-quality standard, applies to any signal table I build (scanner scoring, regime maps). **How to apply:** when authoring condition→outcome rows, ask "can I observe the left side without already seeing the right side?" If no, it's circular.
