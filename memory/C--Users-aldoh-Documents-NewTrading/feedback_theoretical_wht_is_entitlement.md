---
name: feedback_theoretical_wht_is_entitlement
description: "\"Theoretical\" dividend net = user's entitlement as Swiss-resident individual (FR 12.8%, US 15%), never the rate the bank actually applied"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 33052e2d-37a0-45a3-a80b-5483b7c161ac
  modified: 2026-09-27T18:25:18.129Z
---

When comparing dividends received against "theoretical", compute theoretical at the rate the user is
entitled to as a Swiss-resident private individual — France 12.8%, US 15% (treaty, W-8BEN), CH 35%
(fully refundable) — not the rate Gonet/IBKR applied. On 2026-09-27 a Gonet email draft used 25%
(Gonet's applied FR rate) as theoretical and showed the user EUR 13 ahead on L'Oréal; at 12.8% they
were EUR 165.86 short over 2022–2026.

**Why:** the applied rate is the thing under audit; using it as the benchmark hides the main leak.

**How to apply:** any reconciliation or bank query letter — table columns = declared DPS × shares ×
(1 − entitled rate) vs net credited. Flag FR reclaim deadline (31 Dec of 2nd year after payment —
verify). Related: [[project_gonet_dividend_reconciliation]].
