---
name: project_gonet_dividend_reconciliation
description: "Gonet dividend history loaded through 22.09.2026 — open TODOs: register L'Oreal as nominatif, query the OR per-share difference, and the withholding-relief filings"
metadata: 
  node_type: memory
  type: project
  originSessionId: bcd2f874-bc6c-429b-a57e-f3de0e852923
  modified: 2026-09-22T07:58:43.090Z
---

## TODO — raise with Gonet

**1. Register the L'Oréal holding as nominatif.** The 30 OR.PA shares (TradeNr 6) at Gonet are held
au porteur and earn no loyalty premium. Air Liquide at the same bank IS registered — it pays two
credits every May in a 1:10 ratio (EUR 3.70 + EUR 0.37/sh in 2026), the Air Liquide 10% premium,
both withheld at 25%. So Gonet supports nominatif administré and the user already uses it; this is
a form at the bank, not a custody move. Worth ~EUR 21.60 gross a year at the current size, first
payable after two full calendar years registered. Do NOT repeat the earlier framing that
registration means shares leaving the broker for two years — that assumed the OR position sat at
IBKR, and it sits at Gonet. Mechanics: [[reference_loreal_loyalty_bonus]].

**2. Ask why Gonet's L'Oréal per-share figure exceeds the declared dividend.** Unexplained, small,
in the user's favour. Gonet's stated gross is what the 25% withholding is computed on, which is why
comparing against the declared DPS produces a fake ragged rate.

| Pay date | Gonet stated DPS | L'Oréal declared | Diff | Extra received net |
|---|---|---|---|---|
| 29.04.2022 | 4.80 | 4.80 | 0 | — |
| 28.04.2023 | 6.03575 | 6.00 | +0.60% | EUR 1.61 |
| 30.04.2024 | 6.75077 | 6.60 | +2.28% | EUR 6.79 |
| 07.05.2025 | 7.11877 | 7.00 | +1.70% | EUR 2.67 |
| 04.05.2026 | 7.30702 | 7.20 | +1.49% | EUR 2.41 |

Exact in 2022, divergent from 2023, no constant factor. Booked separately against the same 2022
dividend: an `Indemnisation` of EUR 4.11 value-dated 29.04.2022. Ask what the supplement is and
whether other French lines carry the same treatment.

**3. Withholding relief — not filed.** Gonet claims none at source.
- US: 30% applied against a 15% CH-US treaty rate. The first Vanguard credit (23.09.2021) was taxed
  at 15% and every one since at 30%, so the W-8BEN LAPSED rather than never existing. Renewing it
  is a form. Excess so far USD 309.27 on USD 2,214.05 of US gross. Largest of the three.
- France: 25.00% applied (26.50% pre-2022) against a theoretical 12.8%, the FR domestic
  non-resident individual rate, below the 15% treaty cap. Forms 5000/5001. Excess EUR 1,240.17 on
  EUR 10,134.20 of French gross.
- Switzerland: 35% Verrechnungssteuer, fully refunded via the Swiss tax return. No leak.
- Irish UCITS: 0%, correct as applied.

**4. Fees.** Flagged 2026-09-22, not pursued: Comm. mandat CHF 6,036.19, Frais admin. CHF 8,624.76,
Comm. de gestion CHF 3,290.98 — CHF 17,951.93 over five years, against CHF 10,313 of CHF dividend
income in the same period.

## Status of the ledger

`GonetTrades.csv` carries the full dividend history from the Gonet movements statement: 103 income
rows from 23.09.2021 to 22.09.2026 across 16 trade legs, totalling CHF 10,313.45 / USD 2,982.09 /
EUR 7,795.37. Before 2026-09-22 it carried 8 events, all from July 2026 onward.

Balances: CHF 56,459.94, USD 703.41, EUR 1,762.94 — confirmed by the user against the account and
against Tuser. Nothing net changed this session.

## Resolved — the four "double bookings" were reversals

Closed 2026-09-22 (commits 7334dc8, e98efda). A full credits-and-debits export showed six
`Extourne` rows cancelling them: Swiss Life and Saraselect were plain re-bookings, QQQ booked the
wrong rate then reversed its own correction, Amrize was credited to CHF then reversed and re-posted
to USD both times. Write-up in `Reports/gonet_double_booking_report_20260922.md`; per-payment data
in `Reports/gonet_dividend_reconciliation_20260922.csv`.

Two things I got wrong and had to undo: the CHF balance briefly moved to 56,488.54 on an Amrize
credit that a reversal cancels, and I re-dated the baselines and two trades to "fix" USD to 487.72
before the user confirmed 703.41 was right. See [[reference_gonet_movements_statement]] on why the
522.42 baseline is correct as dated, and [[feedback_build_payload_before_open_w]] on backing up
before mutating these files.

Statement format: [[reference_gonet_movements_statement]]. Ledger mechanics:
[[reference_gonet_csv_bookkeeping]]. Six SARASELECT credits (CHF 552.25) stay unbooked by design —
the fund has no leg in the ledger.
