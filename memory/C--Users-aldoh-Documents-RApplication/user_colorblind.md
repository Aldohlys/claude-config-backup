---
name: user_colorblind
description: "User is colorblind — HTML reports must use high-contrast colorblind-safe palette, not red/green"
metadata: 
  node_type: memory
  type: user
  originSessionId: 3f4e5a5e-17d7-445c-8433-e74f630f76ef
---

User is colorblind. Red/orange/green badges and row backgrounds lack sufficient contrast.

Use colorblind-safe palette (Wong/Okabe-Ito) in all generated HTML:
- PASS → Blue (#0072B2) instead of green
- FAIL → Vermillion (#D55E00) instead of red
- WARN/CAUTION → Amber (#E69F00) instead of orange
- SKIPPED/INCOMPLETE → Dark grey (#666666)
- Light backgrounds: use same hues at ~10% opacity

Applies to **new** UI/reports. Exception confirmed 2026-06-30: the existing routine Trade-tab stats tables use a green/red/yellow (viridis-ish) P&L gradient — user said "current palette is fine, keep it" and to NOT retrofit those (would need a cross-table consistent change). Match the existing scheme when extending those tables; only switch if the user explicitly asks for a palette pass.
