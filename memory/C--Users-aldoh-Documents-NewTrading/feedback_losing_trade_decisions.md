---
name: Losing-trade decision framework
description: Two rules for evaluating a losing trade — path-vs-direction diagnostic and balancer-frame conditions — with don't-add-capital as the hard constraint
type: feedback
originSessionId: 6777ad2b-a236-467c-b8db-01c856609c09
---
## Rule 1: Don't add capital to a losing trade, especially past risk allowance

If the trade is down and the technical thesis is still intact, ask: **is it a path issue or a direction issue?**
- **Path issue** (technicals fine, just wrong timing): the missing resource is **time (DTE)**, not strike. Rolling strike down on the same expiry reduces how much move you need but doesn't buy you the time you actually lack. It usually adds capital on top of the loss — violating the rule without fixing the cause.
- **Direction issue** (technicals broken): respect the pre-committed stop. No rolling, no repositioning.

**Acceptable responses to a losing trade with intact thesis:**
- Hold to the pre-committed stop (no action).
- Close and walk away (preserve capital for a cleaner re-entry signal).
- Close and re-express with **recovered capital only** (no fresh budget) into a better structure/DTE — counts as a new decision, not an add.

**Unacceptable:**
- Rolling strike on same expiry to "improve delta." Fixes symptom, not cause, and typically adds capital.
- Quietly dissolving a pre-committed stop because "technicals are still good." The stop exists for that exact day.

**Why:** Stated explicitly by the user on 2026-04-24 during the JPM May 15 330/340 call spread review — spot had drifted to the invalidation level on day 1, and the temptation was to roll 330/340 → 320/330 (same expiry). Correctly identified that this would (a) add ~$290 capital to a loser and (b) not fix the path problem, only the strike problem.

**How to apply:** When the user describes a losing trade, lead with the path-vs-direction diagnostic before suggesting any restructure. If rolling is proposed, explicitly check whether it adds capital and whether it addresses the actual cause (time vs strike). Default recommendation for path-issue trades is hold-to-stop or close-and-walk, not roll.

---

## Rule 2: The "balancer / hedge is doing its job" frame only justifies holding on symmetric-payoff days

A trade sized as a balancer (hedge/offset against a larger opposing book) can legitimately go to zero by design on days when the other side of the book pays off. *"Don't close on first red day — it's doing its job."*

**But that justification only holds when the other side is actually paying that day.** On rotation days (e.g., sector rotation where both the long pick and the bear book are red), the balancer frame doesn't apply — it's pure directional error, not hedged behavior. Cannot be used to rationalize holding through a broken stop.

**Why:** JPM long balancer on 2026-04-24 — bear book (USO/PSX/SPY put/ESTX50 put/QQQ bear spread) did not pay off; financials down, tech up. User correctly observed that "balancer doing its job" was not invokable that day.

**How to apply:** Before accepting a "hedge role" justification for holding a losing trade, verify the offsetting book is in the black that day. If not, evaluate the trade on pure directional merits against its pre-committed stop.
