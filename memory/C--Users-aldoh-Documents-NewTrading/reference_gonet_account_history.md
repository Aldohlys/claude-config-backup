---
name: reference_gonet_account_history
description: "Gonet rows in the Account table are USD before 2025-10-27 and CHF after; 2024-2025 rebuilt 2026-09-23 from the bank statement (cash, missing holdings, cashflows)"
metadata:
  node_type: memory
  type: reference
  originSessionId: e519485e-d771-4455-8a95-717177a51a58
  modified: 2026-09-22T22:50:56.401Z
---

Account rows with account='Gonet' carry `Currency = USD` up to 2025-08-21 and `CHF` from 2025-10-27. The old `getAccountGonet` used `convert_to_usd_date`. The two series are not comparable without converting (the −70k "step" on 2025-10-27 is only the switch). Always compare values in the row's own currency: ConvertToUSD for USD rows, ConvertToCHF for CHF rows.

Repairs done 2026-09-22/23 (every changed row has a `Notes` entry):
- Snapshots with price 0/−1 (Dec 2025 – Sep 2026, LSEETF lines) repriced from Yahoo closes.
- 2026-07-06 17:01: ABBN sale proceeds added to cash. 2025-12-19 and 12-30: missing cash restored.
- All 220 rows from 2024 to 2025 rebuilt. Cash comes from the Gonet movements statement by BOOKING date, anchored on the 2026-09-22 balances (CHF 56,459.94 / USD 703.41 / EUR 1,762.94); this reproduces every known balance. Varenne (62, sold 2024-08-20) and SARASELECT (8, sold 2024-09-24) are counted as their sale proceeds in cash, per the user. Snapshot holding gaps added (AMRZ 2025-06-23 to 08-06, RO/HOLN/OR Aug 2024, SXLV/TRE7 Sep 2024, FXC/GTT/IE00B67T5G21/NUCL Nov–Dec 2025). The CashFlow entries of 2024-09-09 (+23,830.55) and 2024-09-27 (−17,021) were stand-ins for missing cash and are now 0. The 2024-03-14 deposit (USD 3,980 from Big Picture Trading) is booked as CashFlow.
- Backups: `RApplication/data/mydb_before_gonet_reprice_20260922.db`, `mydb_before_gonet_rebuild_2024_2025.db`.

Done later on 2026-09-23:
- Napoléon coins added to 214 rows from 2024 to 2025 (92 until 2025-11-07, then 67). ZKB has no exportable history, so the price is gold spot × 5.806 g × USD/CHF × a ratio moving from 1.0325 (Aug 2023 chart point) to 1.020 (Dec 2025). ZKB/spot has averaged 1.002 ± 0.8% since Dec 2025.
- FXC was stored at ~70 instead of ~112 USD from 2025-11-11 to 12-24; repriced from Yahoo.
- AMRZ: the Tickers row is the US listing (USD, kept for the scanner), so IBKR quoted USD for CHF-booked SIX shares. `gonet_quote_to_position_ccy` (Tdata 5.20.6) now converts quotes to the position currency; 285 snapshots and 287 Account rows repaired.
- 2026: cash = 0 on 15 rows 2026-02-25 20:45 → 03-02 restored from statement. Four snapshots (2026-04-01 15:06/15:24, 04-03 06:46, 04-23 06:46) priced Roche as symbol ROG (~110–120) instead of RO (~330): repriced; the 04-23 06:46 ROG-only snapshot and its Account row were deleted. Cause of the ROG runs unknown (old GonetPos copy?). Backup `mydb_before_2026_cash_rog_fix.db`.
- 2026-01-01 → 07-09 rebuilt (205 rows, backup `mydb_before_gonet_rebuild_2026H1.db`): statement cash; stock value recomputed from the snapshot at ConvertToCHF (the stored value had been 2–4k high Feb–Apr); holdings aligned with booking dates (ABBN −200 from 07-03, AI grant +16 from 06-10; AI is 153 not 154 before the grant, per the bank's dividend count). From 07-10 on, cash comes from the ledger and was left alone. Remaining jumps above 9.5k anywhere since 2024 are market moves.
- AI share counts, per the bank's dividend counts: 180 (2022), 199 after the Jun-2022 grant, 139 after the Dec-2023 sale of 60, 153 after the 12.06.2024 grant (+14), 169 after the 10.06.2026 grant (+16). The 2024–25 snapshots showed 139 until Aug 2024 (grant missing), then 152, then 154. 114 Account rows corrected 2026-09-23 (backup `mydb_before_ai_share_fix.db`), plus 2026 in the H1 rebuild. GonetTrades.csv books the grants as +18 (2022) and +15 (2024) instead of +19 / +14; the end total of 169 is right.

Related: [[reference_gonet_csv_bookkeeping]], [[reference_gonet_movements_statement]].
