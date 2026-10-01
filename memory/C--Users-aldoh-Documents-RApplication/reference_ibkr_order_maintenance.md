---
name: reference_ibkr_order_maintenance
description: "Re-placing IBKR-cancelled GTC exits (reqCompletedOrders), short-sale pre-check counts EVERY sell even inside an OCA group, cancel needs the owning clientId, margin check via whatIfOrder"
metadata:
  node_type: memory
  type: reference
  originSessionId: 1c26deb6-0a13-4dd0-99ab-91d81259daed
  modified: 2026-09-30T21:44:11.946Z
---

Learned 2026-09-30 on U25343478 (Japanese / Korean / Canadian stock exits, stop+limit OCA pairs).

- **Quarter-end GTC purge.** IBKR cancels GTC orders at a calendar limit (TWS preview shows an
  "Auto cancel date", e.g. 20270331). To re-place them *exactly*, read
  `ib.reqCompletedOrders(apiOnly=False)` — it returns contract (conId), qty, lmt/aux, tif,
  ocaGroup, ocaType of each cancelled order. Don't retype from a screenshot. Compare against
  `reqAllOpenOrders()` to see which are still missing, and check `ib.positions(acct)` first.
- **Half-cancelled pair:** if only the stop was cancelled, re-place it with the LIVE limit's
  `ocaGroup` string (ocaType 3) so they stay linked. A fully cancelled pair gets a NEW group
  name. ocaGroup of a live order cannot be modified — cancel and re-place.
- **Short-sale pre-check counts every open sell on the symbol, OCA or not.** Available =
  position − other open sells. Raising one leg of a 400/400 pair to 600 on a 600 position →
  "insufficient shares available for short sale, suggested size 200". Putting an extra DAY
  sell INTO the pair's OCA group does NOT clear it (whatIf still warns). Fix: cancel both and
  place a new pair at the new size, or cancel the pair while a DAY exit works. A whatIf of a
  duplicate of a live order always warns (it adds to the live one) — not a signal.
- **Who can cancel:** an order is cancellable only from its owning clientId. TWS-entered orders
  = clientId 0 (connect with clientId=0). `reqAllOpenOrders()` shows `order.clientId`.
- **Margin check before orders fill:** `ib.whatIfOrder(contract, order)` → init/maint margin
  before/after, commission, `warningText`; nothing transmitted. Account figures:
  `reqAccountUpdates(acct)` then `accountValues` tags AvailableFunds / ExcessLiquidity /
  EquityWithLoanValue (currency CHF on this account), CashBalance per currency, ExchangeRate.
  Japanese small caps on U25343478 get ~100% margin (margin delta ≈ order value).
- The user asks before transmission-type ops are done; they explicitly said "go ahead" each time.

See [[reference_ibkr_open_orders_semantics]], [[user_chf_base_minimize_fx]].
