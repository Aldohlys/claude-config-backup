---
name: /analyze — "lot" = full structure, not single contract
description: $300 per-lot risk cap applies to one complete structure (all legs together), not per leg
type: feedback
originSessionId: 791d7ea0-3b98-46b6-b585-5209bd9337c8
---
In `/analyze`, "one lot" = one complete instance of the option structure (a bull call spread is one lot of two contracts; an iron condor is one lot of four; an outright call is one contract). The $300 per-lot cap is `max_risk` of the structure as a whole — NOT per leg.

**Why:** I initially interpreted "single contract" literally, which would have ruled out every multi-leg structure incorrectly. The user trades verticals routinely; the cap is meant to bound the worst-case loss of the structure, since the legs net against each other.

**How to apply:** Filter `compute_spread_risk_reward` results on `max_risk <= 300` (the whole-structure value the function returns), not on per-leg debit. For outright calls, `max_risk` = call premium. Multi-lot sizing (e.g., 4× the same spread for $1200 total exposure) is a separate user decision that this gate does not constrain.
