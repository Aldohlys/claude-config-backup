---
name: reference_gonet_table_mktvalue_native_ccy
description: "DB Gonet table mktValue is in the holding's own currency (USD/EUR), only CASH rows are in CHF — convert before summing NLV"
metadata:
  node_type: memory
  type: reference
  originSessionId: 250890ed-cd37-47c6-bd39-228637ce7645
  modified: 2026-10-16T04:46:22.011Z
---

In `mydb.db` table `Gonet`, `mktValue = pos × mktPrice` in the line's `currency` (QQQ 44,607 = USD). CASH rows (CHF/USD/EUR) instead carry CHF value in mktValue (pos × FX rate in mktPrice). Summing mktValue directly overstated the 15-Oct-2026 book as 426k CHF; correct NLV was 398.5k.

**How to apply:** multiply non-cash rows by USDCHF/EURCHF (CASH rows' mktPrice gives the rate of that snapshot) before any bucket or NLV total.
