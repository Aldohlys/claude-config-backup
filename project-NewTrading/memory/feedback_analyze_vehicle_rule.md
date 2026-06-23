---
name: /analyze must apply Phase E vehicle rule before structure grid
description: For low-priced underlyings (stock < $150) with IVP < 60, /analyze should pick outright option as the primary structure, not spreads.
type: feedback
originSessionId: a5c2009f-c1bb-4d78-aafa-fafb8bd015a9
---
In `/analyze`, when building the Phase D structure grid, apply the Phase E vehicle rule **before** computing structures, not after:

- `outright option`: stock < $150 AND IVP < 60 → outright call (long) or put (short)
- `spread`: stock > $150 OR IVP > 60
- `stock direct`: price < $10 OR options illiquid

**Why:** On 2026-04-28 /analyze IBIT short, I led with put-debit-spread structures because the cheap_score breakdown emphasized the skew tax on outright puts. User flagged: "for such low-priced stocks, spreads don't make sense — see Phase E." IBIT at $43 with IVP ~30-40 is a textbook outright-option case; spreads waste the cheap absolute IV by capping the upside leg.

**How to apply:**
1. Read price + IVP first; pick vehicle from Phase E rule.
2. Build the structure grid for that vehicle only as the primary section.
3. Other vehicles can appear as a secondary "off-rule context" table if the surface analysis raises a specific objection (e.g., extreme skew tax) — but the primary recommendation set must respect Phase E.
4. For SHORT direction with outright vehicle = put. For LONG = call. Don't lose the direction-to-right mapping.
