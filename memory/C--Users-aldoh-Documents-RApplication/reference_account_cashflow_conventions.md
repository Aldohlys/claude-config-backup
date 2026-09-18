---
name: reference_account_cashflow_conventions
description: "Account.CashFlow conventions - native currency storage, FX via AccountWithConversionRate view, NLV=0 snapshot hazard, Cash Flow button entry point, inter-account transfer already shipped"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 13d07ea0-0f5c-4fb7-8397-97b2e1c8249a
  modified: 2026-09-04T00:00:00.000Z
---

### Storage convention
`Account.CashFlow` is stored in **native currency** (matching the row's `Currency` column). One row per currency per leg. Sign = from this account's perspective (positive = IN, negative = OUT). `Account.Currency` has a foreign key onto `Currencies.Name`, so only active currencies are valid.

### FX conversion happens on read
`Tdata::readAccount()` queries the `AccountWithConversionRate` SQL view, which left-joins `ConvertToCHF` / `ConvertToUSD` and multiplies `CashFlow * chf_conversion_rate` (or USD, depending on `BaseCurrency`). So:
- **Writers**: store native amount. Do not pre-convert.
- **Readers**: receive base-currency values. Do not re-convert.
- Daily FX rates come from `ConvertToCHF` table (date, currency, chf_value). The view falls back to the most recent prior date if the exact date isn't present.

### Entry point: the Cash Flow button (2026-08-31)
Deposits, withdrawals and inter-account transfers are entered from the **Cash Flow** button in `Tuser/account/view/accountUI.R`'s side panel — so it appears in RReporting, `Tuser/routine` and the standalone `Tuser/account` app at once. Logic lives in `Tuser/account/logic/accountf.R`: `record_cash_flow()`, `cashflow_rows()` (shared with the modal preview), `cashflow_accounts()` (excludes the derived `Live`), `cashflow_currencies()`, `cashflow_dates_without_snapshot()`. Transfer mode writes both mirrored legs in one confirm. Hand-written SQL is no longer the normal path.

### Inter-account transfer: shipped and verified (2026-09-04)
Recording a transfer between two IBKR accounts is **already possible** — do not rebuild it. Check the Cash Flow button before implementing anything new here. Modal flow: source account → date → tick *"Transfer between my own accounts"* → destination account → amount (always positive) → currency. The direction radio is hidden in transfer mode; the amount always leaves the source, and `cashflow_rows()` mirrors it with the opposite sign onto the destination.

End-to-end verified on a DB copy: both legs written, `cashflow_heure()` slot allocation across several same-day flows (00:00:01, 00:00:02 …), FX conversion on `readAccount()`, and every guard (source = destination, `Live` as account, amount 0, inactive currency).

Three deliberate limits — worth stating before agreeing to "add transfers":
- **One currency per entry.** A real IBKR transfer with a CHF dowry plus JPY/GBP legs needs one pass per currency.
- **No in-kind securities input.** Position transfers are entered as a value amount in their currency (as in the 2026-04-16 fix). Omit them and TWR reads the NLV step as a market loss.
- **`Live`** is excluded from both pickers and does not pick up a sub-account flow (see Known gap below).

### Verifying account-module DB writes without touching production
`safe_db_connect()` resolves the path via `config::get("DB")`, so a fixture DB is selected by copying `config.yml`, rewriting its `DB:` line, and `Sys.setenv(R_CONFIG_FILE=...)` at the top of the script — no `R_DB_PATH`, which a child Rscript overrides from Renviron.site. `shiny::testServer(accountUI$server, args=list(account=reactive("U1804173"), windowDate=..., PlotType=...))` then drives the modal end to end: `session$setInputs(cashflow=1)` opens it, the `cf_*` inputs fill it, `cf_ok=1` writes. Read `output$cf_preview` to see the exact rows. Confirm the production DB afterwards. See [[reference_testing_tuser_box_internals]].

### The NLV = 0 hazard (fixed, but know why)
Cash-flow rows carry `NetLiquidation = 0`. `accountf$account_extend_data()` picks each day's NLV with `slice_max(heure)`, and `Tdata::twr()` computes `rn[i] <- e_nlv[i] / denom` as a **running product** — so one day resolving to NLV 0 gives `rn = 0` and zeroes **every subsequent day**, pinning TWR to -1. The NA guard in `twr()` does not catch it (0 is not NA).

Two defences, both in place:
- Rows are written at `heure = 00:00:0N` (one slot per leg, allocated by `cashflow_heure()`) so they sort ahead of every real snapshot.
- `account_extend_data()` selects the day's NLV only from rows with a real `NetLiquidation`, and buckets flows onto the next observed snapshot date via `findInterval(..., left.open = TRUE)`. A flow on a date with **no** snapshot therefore counts from the next recorded day instead of destroying the series.

### Why this matters
The TWR caller sums CashFlow across all rows for the day. Since `readAccount` has already FX-converted each row, the sum is in base currency — meaningful even when legs span multiple currencies. Missing or wrong-signed legs produce phantom market moves in the TWR series.

### Known gap
`Tdata::getAccountLive()` inner-joins sub-accounts with `multiple = "any"` over `date >= today` only, and its rows carry no `Currency`. A flow booked on `U1804173` is **not** reliably reflected in the `Live` row. Worth a TODO.

### Related
- [[reference_ibkr_activity_statement]] — where to source the per-symbol/per-currency ground truth
- [[project_account_transfer_cashflow_signs]] — the 2026-04-16 case study and the Tdata 5.10.18 `twr` interpolation fix
- [[reference_live_virtual_account_no_table]]

## UPDATE 2026-09-15 — Account.Notes exists (TODO #81 closed)

`Account` (and the `TestAccount` fixture) now has a nullable `Notes TEXT` column, added by `scripts/migrate_account_notes.R`. The Cash Flow modal writes it on every leg with a default naming the action (`Deposit`, `Withdrawal`, `Transfer <from> → <to>`) that the user completes; blank stores NULL. `Tdata::readAccount()` returns it (5.19.3) as plain text, NOT multiplied by the conversion rate. Snapshot writers leave it NULL. Nothing in the apps lists cash flows yet, so notes are read in DB Browser.

- Tdata `test-account.R` asserts the EXACT ordered column list of `readAccount()` for DU5221795 and Gonet — any new Account column must be appended there, or two tests fail.
