---
name: reference_ibkr_fx_conversion_mechanics
description: "IBKR FX conversion mechanics — EUR-base pair convention (no USD.EUR), quantity in base ccy, Currency Converter for sub-25k, IDEALPRO session hours, per-currency interest"
metadata: 
  node_type: memory
  type: reference
  originSessionId: c36265f1-a17a-4548-8c42-0a7fe7edb7b4
  modified: 2026-08-31T07:37:18.774Z
---

Executing a currency conversion in IBKR (as opposed to FX accounting in the codebase — see [[project_fx_risk]], [[reference_cash_fx_pnl_in_totals]]).

**Pair convention — inverses do not exist.** IBKR follows market convention with a fixed base-currency precedence: `EUR > GBP > AUD > NZD > USD > CAD > CHF > JPY`. So `EUR.USD`, `USD.CHF`, `EUR.CHF` exist; `USD.EUR` and `CHF.USD` do not. To go USD→EUR you **BUY EUR.USD**.

**Order quantity is denominated in the pair's BASE currency.** On a BUY EUR.USD ticket you enter the EUR amount, not the USD amount. Consequence: you cannot pin the USD side exactly — dollars spent depend on the fill rate, leaving a small residual either way.

**Use the Currency Converter, not an order ticket, for conversions.** Client Portal → Transfer & Pay → Convert Currency (or the TWS equivalent). You specify "sell N of currency A, buy currency B", which handles both the pair/side lookup and the base-currency quantity problem, and lets you zero a balance exactly. Routes via FXCONV, which converts without opening an FX position.

**IDEALPRO minimum is 25,000 USD-equivalent.** Smaller conversions (a ~$5k retail transfer) are odd-lot routed at a wider spread regardless of how you enter them — another reason to use the Converter.

**Session hours — "Inactive" on Sunday is normal, not a bug.** IDEALPRO opens **Sunday 17:15 ET** and runs to Friday 17:00 ET, with a daily 17:00–17:15 ET break. In CEST (= ET+6 in summer) the Sunday open is **23:15 Swiss time**, so an order placed at a normal Sunday evening hour in Geneva sits before the open. TWS labels orders submitted outside a product's session as literally **"Inactive"** — that status word is the tell. Don't leave such an order resting: the ask you priced off is a stale Friday-close/indicative quote, and the Sunday reopen has the week's widest spreads. Cancel and re-enter when a real two-sided market exists — best is the **London–NY overlap, ~14:00–18:00 CEST**.

**Interest is computed PER CURRENCY — no cross-currency netting.** A EUR debit accrues at ESTR + 1.5% (tier 1, NLV < 100k) *while* a large CHF credit balance sits earning ~nothing. You cannot offset one against the other. Also: IBKR pays credit interest only on cash above 10,000 of a currency, scaled down further for NLV < 100k — so a ~$5k idle USD balance earns exactly zero.
