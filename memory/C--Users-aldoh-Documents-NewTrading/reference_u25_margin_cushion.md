---
name: reference-u25-margin-cushion
description: U25343478 holdings get ~85% margin requirement; size new positions from the cushion, never from Buying Power
metadata:
  type: reference
---

Account **U25343478** holds illiquid foreign small caps (Japanese net-nets, KOSDAQ, LSE small cap). IBKR gives them almost **no loan value**: on 2026-09-02, maintenance margin was 21,059 CHF on 24,604 of stock — an **85.6% average requirement**. The JPY names are almost certainly charged 100%, the rest around 50%. The account is capital-constrained by the collateral haircut, **not by leverage** — gross stock was only 1.07x net liquidation value.

## Margin Cushion
**Cushion = Excess Liquidity / Net Liquidation Value.** IBKR alerts below 10%. It issues **no margin calls** — if Excess Liquidity goes negative the account is force-liquidated automatically, at market, on names chosen precisely because nobody trades them. A forced exit can cost more than the whole buffer being asked for.

**2026-09-02 event:** cushion hit 7.90% (EL 1,817 / NLV 23,006), triggering an IBKR Excess Liquidity alert. Fixed by a 5,000 CHF internal transfer from U1804173 -> cushion 24.3%.

## Why the cushion is insensitive to price but fragile to rule changes
The requirement is a percentage of market value, so a price drop cuts NLV and maintenance margin together and barely moves the cushion. A **house-requirement change hits only one side**. Stress test at the time: if every holding were reclassified to 100%, maintenance would go to 24,604 and EL to 3,265 — an 11.7% cushion, still compliant. That is what the 5,000 transfer bought, versus the 537 CHF that would have technically cleared the 10% test.

## Buying Power is misleading here - do not size from it
After the transfer TWS showed Buying Power 26,520 and SMA 16,065. **Real headroom for new positions of this type was about 1,200-1,400 CHF.** A single 5,000 CHF purchase would have added ~4,250 of maintenance margin and dropped the cushion to 9.1% — straight back into violation.

**Sizing rule.** Buying stock with cash leaves NLV unchanged, so:

    max position = (Excess Liquidity - floor x NLV) / requirement

At a 20% floor and 85% requirement that gave ~1,426 CHF; at a 15% floor, ~3,000. **Keep the cushion at or above 25%.**

## Timing mechanics
- **Selling releases margin immediately** — the requirement drops the moment the trade fills, no settlement wait. NLV is unchanged (stock becomes cash), so EL rises by the released requirement.
- **A deposit does not protect you same-day.** The alert text is explicit that funds in transit or under credit hold are not counted when liquidating. **Internal transfers between the user's own IBKR accounts post same-day**; an external bank transfer takes 1-3 business days.

## The meta-lesson
The problem was never funding. U1804173 was sitting at a **96.8% cushion** with 50,649 CHF of idle CHF cash (maintenance margin 1,132 on NLV 51,308, leverage 0.02) while U25343478 took a liquidation warning at 7.9%. **The constraint was allocation between accounts, not capital.** Check both accounts' cushions together.

Related: [[project-u25-fx-hedge-policy]], [[ibkr-fx-exposure]], [[project-daubasses-portfolio]]
