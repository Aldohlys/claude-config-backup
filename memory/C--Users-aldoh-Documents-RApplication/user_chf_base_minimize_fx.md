---
name: user_chf_base_minimize_fx
description: "Base currency is CHF and user actively minimizes FX exposure — fund foreign outflows from the currency already held long, never pre-hold or borrow foreign cash"
metadata: 
  node_type: memory
  type: user
  originSessionId: c36265f1-a17a-4548-8c42-0a7fe7edb7b4
  modified: 2026-08-31T07:37:34.583Z
---

Base currency is **CHF**; the user actively wants to minimize currency risk, and asks for conversion routing decisions on that basis.

**Rule for a foreign-currency outflow: fund it from the non-base currency you are already long, not from CHF.** Selling CHF leaves the existing foreign exposure intact on a now-smaller equity base, so exposure as a % of NLV actually *rises*. Selling the foreign balance takes it toward zero. Cost is a wash — single conversion either way, same 0.20 bp / USD 2 minimum commission — so there is no execution argument for the CHF route.

**Idle foreign cash is the whole problem; options are FX-light.** A position's contribution to CHF-denominated NLV is its **market value**, not its notional — there is no mechanical channel from USD/CHF into a US stock's USD quote. Worked example (Aug 2026): a book of 5 USD option positions controlling large notional carried only ~$1,006 of FX exposure (1.5% of NLV), while $4,981 of *idle cash* carried 8.1%. So opening more US option trades barely moves currency risk. Always decompose exposure as cash + position MV per currency before advising.

**Don't pre-hold or borrow foreign currency to "pre-hedge" future trades.** Borrowing USD against long USD assets is a real natural hedge, but it is exact only at inception and drifts with P&L — and for *long options* it decays badly, since the asset bleeds to zero at expiry leaving a bare debit to close at an unknown rate. Combined with asymmetric carry (pay the USD debit rate, earn ~nothing on CHF — see [[reference_ibkr_fx_conversion_mechanics]]), it is a certain cost to hedge an uncertain risk. **Convert CHF→foreign at the moment each trade is opened, sized to it** (~$2 + a fraction of a pip, trivial vs a 2–3% currency move).

**Small residual debits are fine and can simplify execution.** Letting a wire finish on a small foreign debit rather than topping up to zero can eliminate an entire second conversion leg; at a few hundred CHF it is noise against NLV. Weigh the ~€25–30/yr interest drag against the ~€3 conversion cost.

**Scope caveat:** the IBKR MCP connector exposes only ONE account. Gonet and other sub-accounts are not in that view — see [[reference_account_strategy_topology]] and [[reference_live_virtual_account_no_table]] before reasoning about consolidated "Live" exposure.
