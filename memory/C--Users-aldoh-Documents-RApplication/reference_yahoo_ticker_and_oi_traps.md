---
name: reference_yahoo_ticker_and_oi_traps
description: "GOLD is no longer Barrick (now Gold.com Inc; Barrick is B); CTRA/NGD return 404 from Yahoo though IBKR has them; Yahoo option OI must be read at the monthly expiry, not tk.options[:3]"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6df3dc0a-4099-43b9-b384-ce0952b42b01
  modified: 2026-09-03T18:57:01.435Z
---

Found 2026-09-03 while screening miners for the BOT universe. All three would silently
produce wrong analysis rather than an error.

## GOLD is not Barrick any more

Barrick Mining Corporation changed its NYSE ticker **GOLD -> B** in 2025. `GOLD` now
resolves to a different company, **Gold.com, Inc.**, ADV ~14 M CHF against Barrick's 309.
Anything screening `GOLD` for Barrick pulls the wrong instrument at a plausible-looking
price (43.82 vs 44.13 on the day, so the number does not look wrong either). The universe
CSV records Barrick as `B` with the rename in its `note`.

Two more corporate actions in the same complex: **SAND** (Sandstorm) and **MAG** (MAG
Silver) return no data — both acquired.

## Yahoo 404s that are not delistings

**CTRA** (Coterra Energy) and **NGD** (New Gold) return a hard `Quote not found` 404 from
Yahoo on every period, but IBKR has the instruments (`search_contracts` finds COTERRA ENERGY
INC). Do **not** call these delisted. Any Yahoo-driven pipeline — `bot_scan_universe.py` is
one — simply cannot carry them until that data path works. Verify against IBKR before
concluding a name is gone; see [[reference_isIBAvailable]] for the TWS route.

## Yahoo option open interest: read the monthly

`yf.Ticker(t).options[:3]` picks the **near weeklies**, which carry almost no OI. That read
gave RRC 88 contracts against a real 33 554 at the monthly, and EQT 9 434 against 102 742 —
enough to disqualify a name on the >= 5000 floor for entirely spurious reasons.

```python
exp = EXPIRY if EXPIRY in tk.options else next((x for x in tk.options if x >= EXPIRY), None)
ch = tk.option_chain(exp)
oi = int(ch.calls.openInterest.fillna(0).sum() + ch.puts.openInterest.fillna(0).sum())
```

At the standard monthly the numbers are sane and comparable to the IBKR-sourced values
already in the CSV (same order of magnitude, not identical — different source).

The IBKR MCP `get_option_data` returns **contract structure only**, no OI, volume or IV — it
would need a `get_price_snapshot` per strike, so it is not a practical OI source.

Related: [[reference_ibkr_symbol_with_space]], [[reference_bot_tradable_universe_csv]].
