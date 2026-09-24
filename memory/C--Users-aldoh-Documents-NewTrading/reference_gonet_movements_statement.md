---
name: reference_gonet_movements_statement
description: "Gonet 'Movements' xlsx export — columns, per-currency IBANs, operation types, and that income rows are booked on the value date net of withholding"
metadata: 
  node_type: memory
  type: reference
  originSessionId: bcd2f874-bc6c-429b-a57e-f3de0e852923
  modified: 2026-09-22T07:58:29.266Z
---

Gonet's account statement exports as a single-sheet xlsx named
`document_<ISO timestamp>.xlsx`, sheet `Movements`, downloaded to `~/Downloads`. Header sits on row
6; data starts row 7. The user can pull any date range (2026-09-22 pull covered 23.09.2021 to
22.09.2026, 153 movements).

Columns (French): Date de comptabilisation (booking) · Date valeur · Operation · Détail ·
Prix d'exécution · Liquidité (IBAN) · Devise · Montant · ISIN · Montant des commissions ·
Frais de courtage.

One IBAN per currency, which is how to tell which cash account a row hit:

| suffix | ccy |
|---|---|
| CH61 ... 8000 0 | CHF |
| CH34 ... 8000 1 | EUR |
| CH07 ... 8000 2 | USD |

Operation types: `Encaissement` (dividend/coupon), `Indemnisation` (compensation — fractional-share
cash-in-lieu, or a dividend top-up), `Vente de` / `Achat`, `Change comptant` (FX), `Bonification`
(deposit), `Remise de caisse`.

Booking notes:

- Book on the VALUE date (`Date valeur`), not the booking date. They diverge by days to months —
  a QQQ credit value-dated 30.04.2024 was booked 13.08.2024.
- Amounts are NET of withholding. `Détail` carries shares and declared DPS
  ("250 TOTALENERGIES SE € EUR 0.85"), so gross and the implied WHT are derivable per line.
- Not every row has an ISIN — Holcim, Amrize and the Indemnisation lines are blank, so match on the
  Détail text as a fallback.
- Yahoo DPS is in the LISTING currency and will not match: Amrize declares USD but yfinance reports
  the CHF listing. Use the statement, not yfinance, whenever the statement exists.

ALWAYS REQUEST THE EXPORT THAT INCLUDES DEBITS. Gonet offers a credits-only variant (153 rows,
zero negatives, no purchases) and a full one (235 rows, 82 negatives). The credits-only version
omits the `Extourne` (reversal) rows, which manufactured four false "double bookings" on this
account in one session — all six Extournes turned out to reverse them. Operations in the full
export: Encaissement, Indemnisation, Extourne, Vente de, Achat de, Change comptant, Bonification,
Frais admin., Comm. mandat, Comm.de gestion, Intérêts, Transfert, Remise de caisse.

Glossary: `Extourne` = reversal (storno), an opposite entry cancelling an earlier booking, e.g. the
CHF 28.60 AMRZ dividend (value 20.05.2026) reversed 04.06.2026. `Indemnisation` on a free-share
grant date = cash for the fractional grant shares ("0.53 AIR LIQUIDE"), not a dividend.

Net every Extourne against one matching credit (same value date + Détail + currency + amount)
before booking anything.

BOOKING DATE != VALUE DATE (FXC sale booked 08.07.2026 / value 10.07.2026; CNYA purchase booked
09.07.2026 / value 13.07.2026). Do NOT "fix" ledger stock legs to value dates on that basis. Those
two flows are already inside the USD baseline of 522.42, so their booking dates before the cut-off
are deliberate: 522.42 = the post-purchase balance (557.72) less the QQQ credit of 35.30 that is
booked as its own row, and 522.42 + 35.30 + 28.60 + 117.09 = the confirmed 703.41. Re-dating them
double-counts -215.69. The user verifies Gonet cash independently in Tuser — ASK before rewriting
a balance that reconciles.

Rates applied are exact once measured against GONET's stated gross, not the declared DPS: France
25.00% (26.50% pre-2022), Switzerland 35.00%, US 30.00%, Irish UCITS 0%. Gonet's per-share figure
can exceed the declared dividend - L'Oréal 2023 shows EUR 6.03575 against a declared 6.00 - so
comparing with the declared DPS produces a ragged fake rate (the earlier "23.3-25%" reading).

Reconciliation output: `Reports/gonet_dividend_reconciliation_20260922.csv`. Ledger mechanics:
[[reference_gonet_csv_bookkeeping]]. Open bank queries:
[[project_gonet_dividend_reconciliation]].
