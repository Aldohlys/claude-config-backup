---
name: xop-thin-option-chain
description: "XOP option chain is too thin for OI-cap analysis — total ~2k OI in ±25% band, ETF behaves like single-name midcap"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8c65e4a9-c01b-4a86-95fe-f6292e3db0e8
---

# XOP option chain depth — too thin for OI-cap analysis

Verified 2026-05-27 against the 20260626 expiry (30 DTE). Spot \$166.

## OI depth (single-strike OI within ±25% of spot)

- Total OI in band: ~2,000 contracts (all strikes, both sides combined)
- Single-strike max: 300 contracts (\$135 C, deep ITM — stock-replacement)
- Highest OTM call OI: \$170 with ~100-179 OI (snapshot-dependent)
- Highest OTM put OI: \$130 with 166 OI (deep OTM tail hedge)
- Most OTM strikes >+10% from spot: <20 OI

For reference, SPY/QQQ single-strike OI runs 5 figures routinely. XOP is closer to a single-name midcap than a major ETF in option depth.

## UPDATE 2026-06-08 — thinness is EXPIRY-dependent; monthlies are deep

The "XOP is thin" claim above was measured on **20260626, a WEEKLY**. Re-measured the
**Jul17 MONTHLY** (39 DTE): **8,686 call OI in the 155-190 band** — single strikes
\$175=3,327, \$182=1,416, \$185=879, \$170=880. Not thin at all. The weeklies remain
near-empty (Jul10 119, Jul24 68 total call OI). Two drivers: (1) monthlies (3rd Friday)
carry the OI; weeklies don't. (2) OI built up as XOP rallied toward the highs since
27-May. **Correction: XOP monthlies are tradeable for outrights; don't treat XOP as a
blanket thin chain.** Note /analyze still flags `chain_state=thin` for XOP because it
samples the scanner-CSV expiry (Jul10 weekly) — so its effective_target→structural-target
fallback is right *for the pipeline*, but manual outright selection should use the
monthly. See [[feedback_check_oi_by_expiry_for_outright]].

## Implication

Even after the [[oi-cap-otm-filter]] fix, OI caps on XOP carry low information. The "dealer-hedging resistance" signal isn't really there — there's nothing for dealers to hedge against. /analyze's `effective_target` should (and now does) default to the structural target for this kind of thin chain via the thin_oi_threshold=100 bypass.

## How to apply:
- For XOP trade planning, ignore the OI-cap-derived `effective_target` — use the structural target directly.
- Same suspicion warranted for other narrow-mandate sector ETFs (XME, XBI, KRE, etc.) and small-cap thematic ETFs. Verify OI depth before relying on chain-derived caps.
- See [[carrefour-thin-options]] for a similar issue on EU single-names (CA.PA deactivated from scanner).
