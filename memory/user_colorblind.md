---
name: user_colorblind
description: User is colorblind — HTML reports must use high-contrast colorblind-safe palette, not red/green
type: user
---

User is colorblind. Red/orange/green badges and row backgrounds lack sufficient contrast.

Use colorblind-safe palette (Wong/Okabe-Ito) in all generated HTML:
- PASS → Blue (#0072B2) instead of green
- FAIL → Vermillion (#D55E00) instead of red
- WARN/CAUTION → Amber (#E69F00) instead of orange
- SKIPPED/INCOMPLETE → Dark grey (#666666)
- Light backgrounds: use same hues at ~10% opacity
