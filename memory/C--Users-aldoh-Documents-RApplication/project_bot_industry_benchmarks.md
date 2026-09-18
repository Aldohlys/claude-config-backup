---
name: project_bot_industry_benchmarks
description: "BOT scanner S3 is industry-relative (SMH for a semi, not XLK) via the CSV bench column; swapping a benchmark shifts a whole group by one constant and only moves the pass/fail line"
metadata: 
  node_type: memory
  type: project
  originSessionId: 6df3dc0a-4099-43b9-b384-ce0952b42b01
  modified: 2026-09-03T18:56:24.107Z
---

Shipped 2026-09-03 at the user's request: *"Sector ETF are not enough focused — within XLK
the software and semis behave very differently."*

`bot_scan_universe.py` used to map `db_sector` (14 broad buckets) to one sector ETF, so all
14 semis AND all 7 software names AND ARKK were measured against **XLK**. Now the CSV's
**`bench` column** is the primary source and names an industry ETF; `SECTOR_ETF` on
`db_sector` is only a fallback for a blank. `resolve_bench()` enforces both, plus a
**self-benchmark guard**.

## The one thing to understand before touching it

Swapping XLK to SMH shifts **all 14 semis' `rs20` by the same constant** (SMH's own excess
return over XLK). It does not re-rank names inside the group — dispersion within S1 was
already wide (+23.3 to -16.7). **All it moves is the S3 pass/fail line.** That is the point:
against a broad benchmark, in a semis-led tape every semi clears S3 for free and the gate
measures the industry rather than the name. It is not a way to get more information per name.

Measured effect on the 2026-09-03 scan: 62 of 75 names re-benched, **11 S3 flips (18%)**,
GREEN 9 -> 8. CL=F's +7.4 had been measuring XLE not crude; COIN's +18.0 was measuring
bitcoin (now IBIT).

## Two bugs the old map hid

- **TLT was benched against TLT** (`db_sector 'US bonds' -> TLT`): `rs` identically 0.00, so
  `S3: rs20 > 0` was False forever and TLT scored S=0 on every scan. Hence the guard —
  a name is never its own benchmark; fall back to the broad ETF.
- **GLD/IAU were benched against GDX** — bullion against its own levered miners, structurally
  -13pp. Now SPY.

## Map shape

Industry, not sector: SMH semis / IGV software / IBIT COIN / IAI HOOD / PPH pharma /
OIH services / FCG gas E&P / XOP oil E&P / COPX copper / SLX steel / URA CCJ / LIT ALB /
REMX MP / XME diversified miners / GDX gold miners / SIL silver / ITB homebuilders /
XRT retail / CARZ autos / KWEB China / DBA grains+softs / IEF TLT.

**FX-clean benchmarks for non-USD names**: the stock return is in CHF/EUR and a USD ETF's is
not, so `rs` was FX-contaminated. Swiss names -> `^SSMI`, Paris-listed banks -> `EXV1.DE`
(both EUR), but USD-listed ADRs (DB, ING, UBS) -> `EUFN`. A benchmark needs only a price
series, so thin ADV is fine there (FCG is 16 M CHF and unusable as a *candidate*).

The concentration report groups by `bench`, so finer benchmarks make it useful — it now
names real correlated clusters ("FCG 3: EQT, RRC, AR — size as ONE bet") instead of lumping
everything tech under XLK.
