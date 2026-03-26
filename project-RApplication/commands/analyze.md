# Analyze BOT Candidate Command

Deep analysis of a BOT (Breakout Option Trade) candidate through 4 sequential gates, 4-dimension IV assessment, and DOW pattern scoring.

**INSTRUCTIONS FOR CLAUDE:**

You MUST follow these steps sequentially. Stop at the first FAIL gate and report the verdict unless the user explicitly asks to continue. **Always produce the HTML report (Step 5) regardless of GO/NO-GO verdict** — only `--no-html` flag skips it.

## Step 1: Parse Arguments

Parse from $ARGUMENTS:
- **ticker** (required): Stock symbol, e.g. AAPL, GLD, TGT
- **direction** (required): `long` or `short`
- **Flags** (optional):
  - `--force`: Continue analysis even if a gate fails
  - `--no-html`: Skip HTML output, terminal only

Example: `/analyze TGT long`, `/analyze AAPL short --force`

## Step 2: Load Data Sources

### 2a. Swing Scanner CSV
Find the most recent swing scanner CSV:
```bash
ls -t C:/Users/aldoh/Documents/NewTrading/reports/swing_scanner_*.csv | head -1
```
Parse it (semicolon-delimited, comma as decimal separator, **fields are double-quoted**). Extract the row matching the ticker using:
```bash
grep "\"TICKER\"" <csv_file>
```
Do NOT use `^TICKER;` — fields are quoted so the line starts with `"TICKER"`, not bare `TICKER`.

**If ticker not found in CSV**: Report "Ticker not in scanner" and ask user whether to proceed with manual data or abort. If proceeding, user must provide: IV30, RV30, IVP, RVP, VRP values.

**Available CSV columns**: Ticker, Sector, Sector_ETF, Price, ATR_pct, Long_Score, Long_Signal, Long_Entry, Long_Stop, Long_Target, Short_Score, Short_Signal, Short_Entry, Short_Stop, Short_Target, ADX10, RSI14, RSI_slope, MA50_dist, OBV_rising, UpDn_ratio, RS_vs_ETF, Macro_Note, Best_Signal, IV30, RV30, IVP, VRP, Optionality, TermStr, Vol_Date, Composite, Gate_Tech, Gate_Opt, All_Gates, Rank

### 2b. Macro Context HTML
Find the most recent macro context HTML:
```bash
ls -t C:/Users/aldoh/Documents/NewTrading/reports/macro_context_*.html | head -1
```
Parse the HTML to extract:
- **Daily Bias**: from `.bias-badge` text (NEUTRAL, LONG BIAS, SHORT BIAS)
- **VIX spot**: from section 01, row "VIX spot" `.value` 
- **VVIX**: from section 01, row "VVIX" `.value`
- **VIX term structure**: from section 01, "Term structure" row (FLAT, CONTANGO, BACKWARDATION)
- **S5FI (breadth)**: from section 03, "MA50 Breadth" `.value`
- **DXY**: from section 04, "DX-Y.NYB" `.value`
- **GLD direction**: from section 04, "GLD" `.note` (above/below MA20)
- **Events**: from section 07, list all events with dates and severity
- **Dominant scenario**: from section 08, the scenario card with `.sc-dominant` class + probability
- **Stress banner**: whether `.stress-banner` element exists (VIX > 25)

## Step 3: Execute Gate Analysis

### GATE 1: Direction × Macro Regime

**Purpose**: Avoid fighting the market.

Read S5FI (breadth) from macro context and apply:

| S5FI (breadth) | Long BOT | Short BOT |
|:---:|:---:|:---:|
| < 20% (capitulation) | **HIGH CONVICTION** | NO — shorts exhausted |
| 20-35% (oversold) | **PREFERRED** | CAUTION — only if Gate 3 is A+ |
| 35-65% (neutral) | Yes | Yes |
| 65-80% (overbought) | CAUTION | PREFERRED |
| > 80% (euphoria) | NO — longs exhausted | **HIGH CONVICTION** |

**Additional checks** (each can override to FAIL):
- VIX > 30 → WARN: high uncertainty, reduce position size
- VIX term structure in backwardation → WARN: stress regime, favor puts
- DXY trend → NOTE if impacts the ticker (precious metals, commodities)
- Event calendar → **FAIL if FOMC, CPI, or OpEx within 24h of today**

**Sector-specific macro context** — read from scanner CSV `Macro_Note` column and sector_gate tailwinds/headwinds:
- PreciousMetals: tw = weak USD, gold rising, VIX stress, curve inverted | hw = strong USD, high rates
- Technology: tw = breadth bullish | hw = high rates, curve inverted, VIX stress, backwardation
- Energy: tw = oil surging | hw = strong USD, breadth bearish, VIX stress
- Healthcare: tw = VIX stress, breadth bearish | hw = high rates, breadth bullish
- ConsumerStaples: tw = VIX stress, breadth bearish | hw = high rates, breadth bullish

**Output**: PASS (with conviction level: HIGH/PREFERRED/YES/CAUTION) or FAIL + reason

### GATE 2: Scanner Quality + Directional Fit

**IMPORTANT**: This is a BOT (Breakout Option Trade) analysis. Use **BOT columns**, not swing Long/Short columns.

Read from scanner CSV:
- **BOT_Score**: must be **≥ 7** to pass. This is the composite BOT score.
- **BOT_Flags**: Contains setup and breakout scores in format `S:X/6 BK:Y/4 | S1:+/- S2:+/- ... | BK1:+/- ...`. Setup ≥ 4/6 AND Breakout ≥ 3/4 = confirmed (green in HTML). Report which individual criteria pass/fail.
- **RS_vs_ETF**: must be positive for longs, negative for shorts
- **Long_Entry / Long_Stop / Long_Target** (or Short_ equivalents): Use for R/R ratio and M3 computation. These are directional levels from the swing scanner but still relevant for BOT entry/stop/target planning.

Do NOT use `Long_Signal` or `Short_Signal` — those are swing scanner ranking signals, not BOT signals. A ticker can be SKIP for swing but a valid BOT candidate.

**Sector historical performance** — add conviction note:
| Sector | Historical Edge | Note |
|--------|:---:|------|
| Precious Metals | +$395 avg, 66.7% WR | Best sector |
| Consumer non-cyclical | +$264 avg, 60% WR | Clean breakouts |
| Technology | +$214 avg, 44.4% WR | Decent but noisy |
| Basic Materials | +$134 avg, 36.4% WR | Mixed |
| Energy | -$66 avg, 30% WR | Weak — macro-dependent |

**Gate 2 criterion decomposition** — show actual computed values for each criterion:
- M3 (room to 20d low/high): compute `(price - low20) / ATR` for shorts or `(high20 - price) / ATR` for longs. Show the ratio value (e.g., "0.4 ✗" or "1.8 ✓"). Threshold is > 1 ATR. Add note only if the value adds context (e.g., "near 20d low").

**Output**: PASS (BOT S:X/6 BK:Y/4, RS, sector conviction) or FAIL + reason

### GATE 3: Vol Profile — Optionality Assessment

**Gate 3 is only reached if Gates 1 and 2 PASS.** This avoids costly data refreshes for candidates that fail on macro or technicals.

Gate 3 answers one question: **are options favorably priced for a breakout trade?** It proceeds in 3 stages, each deeper and more costly. Early stages can produce a FAIL verdict without needing later stages.

#### Stage 1: Data Freshness Check

Check the `Vol_Date` column in the scanner CSV for this ticker. Format is "YYYYMMDD HH:MM".

**Decision tree:**
1. **Vol_Date is today or yesterday** → Data is fresh. Proceed.
2. **Vol_Date is 2+ days old** → Data is stale. Ask user:
   "Vol data for <TICKER> is from <Vol_Date> (X days old). Gates 1-2 passed. Refresh via TWS? (y/n)"
   - If yes: execute TWS refresh (see below), then re-read updated data
   - If no: proceed with stale data but flag in verdict as "STALE VOL DATA"
3. **Optionality = "NO DATA"** → No vol data at all. Ask user:
   "No vol data for <TICKER>. Gates 1-2 passed. Fetch via TWS? (y/n)"
   - If yes: execute TWS fetch, then continue
   - If no: report Gate 3 as **INCOMPLETE**, skip to Gate 4

**TWS refresh command** (write temp script):
```bash
cat > /tmp/analyze_refresh_vol.R << 'EOF'
library(Tdata)
args <- commandArgs(trailingOnly = TRUE)
ticker <- args[1]
result <- tryCatch(getVolMetrics(ticker), error = function(e) {
  message("TWS refresh failed: ", e$message)
  NULL
})
if (!is.null(result)) message("OK: refreshed ", ticker) else message("FAIL: TWS not connected?")
EOF
cd C:/Users/aldoh/Documents/RApplication && Rscript /tmp/analyze_refresh_vol.R <TICKER>
```

#### Stage 2: Primary Screening (CSV data only — no DB cost)

Evaluate 5 criteria from scanner CSV columns: IV30, RV30, IVP, RVP, VRP, TermStr.

| # | Criterion | Threshold | CSV Column | Rationale |
|---|-----------|-----------|------------|-----------|
| 1 | **IV30 < 40%** | Low absolute vol | IV30 | Winners avg 27.7% vs losers 35.9% |
| 2 | **IVP < 60%** | Room for IV expansion | IVP | Enter before vol reprices upward |
| 3 | **RVP > 30%** | Stock is actually moving | RVP | Breakout has fuel, not sleeping |
| 4 | **VRP negative or near zero** | Options not overpriced | VRP + IVP | See interpretation below |
| 5 | **Term structure: contango** | Near-term IV not inflated | TermStr | Near-term cheaper = favorable |

**VRP interpretation** (criterion 4 requires nuance, not just sign):

| VRP | IVP | Reading | Criterion 4 |
|:---:|:---:|---------|:---:|
| Negative | Low (< 50%) | Options cheap + IV hasn't repriced → **mispricing** | **MET** |
| Negative | High (> 70%) | IV expanded but RV spiked more → **chasing** | **NOT MET** |
| Near zero (±2) | Any | Fair pricing, no vol edge | **MET** (neutral) |
| Positive | Low (< 50%) | Slightly expensive but quiet → breakout not started | **MET** (acceptable) |
| Positive | High (> 70%) | Expensive + high IV → **overpaying** | **NOT MET** |

**Primary screening verdict:**
- **5/5 criteria met** → PASS with note "A+ vol setup"
- **4/5 criteria met** → PASS
- **3/5 criteria met** → PASS (marginal) — proceed to Stage 3 for confirmation
- **2/5 criteria met** → **FAIL** — "Weak vol profile: [list which criteria failed and why]"
- **0-1/5 criteria met** → **FAIL** — "No vol edge: [list details]"

**If FAIL at Stage 2** → Gate 3 = FAIL. Stop here (unless --force). Report which criteria failed and the actual values.

#### Stage 3: Deep IV Assessment (DB query — only if Stage 2 ≥ 3/5)

This stage adds conviction by checking whether the vol setup is genuinely mispriced or just looks OK on surface metrics. Load deeper data from database:

```bash
cat > /tmp/analyze_vol.R << 'EOF'
library(Tdata)
library(jsonlite)
args <- commandArgs(trailingOnly = TRUE)
ticker <- args[1]

conn <- safe_db_connect()

# Peer VRP: latest vol data per ticker, fresh only (< 5 trading days)
cutoff <- format(Sys.Date() - 7, "%Y%m%d")
peer_data <- dbGetQuery(conn, sprintf(
  "SELECT sym, iv30, rv30, ivp, rvp, datetime FROM Prices
   WHERE iv30 IS NOT NULL AND datetime >= '%s'
   GROUP BY sym HAVING ROWID = MAX(ROWID)", cutoff))

# Term structure for target ticker
term_data <- dbGetQuery(conn, sprintf(
  "SELECT iv30, iv60, iv90, iv180, datetime FROM Prices
   WHERE sym = '%s' AND iv30 IS NOT NULL ORDER BY ROWID DESC LIMIT 1",
  ticker))

dbDisconnect(conn)
cat(toJSON(list(peers = peer_data, term = term_data), auto_unbox = TRUE))
EOF
cd C:/Users/aldoh/Documents/RApplication && Rscript /tmp/analyze_vol.R <TICKER>
```

Evaluate 4 dimensions. Dimensions 2-4 add conviction but do not override a Stage 2 PASS:

| Dim | What to check | Favorable | Data Source | If unavailable |
|:---:|-----------|-----------|-------------|----------------|
| **1** | IVP: IV vs own history | < 50% | CSV: IVP | Already checked in Stage 2 |
| **2** | Peer VRP: IV vs sector peers | Cheapest in group | DB: compare iv30 across sector tickers | "INSUFFICIENT PEER DATA" if < 3 fresh peers |
| **3** | Term structure detail | Contango: iv30 < iv60 < iv90 | DB: iv30/iv60/iv90/iv180 | Use CSV TermStr as fallback |
| **4** | Skew: put/call skew | Flat (OTM not penalized) | Not in DB — note "requires TWS check" | Flag as "UNVERIFIED" |

**DOW A+ pattern** — check if this is a high-conviction mispricing (the DOW #692 / TGT #678 signature):
1. Negative VRP ✓/✗
2. Low IVP < 50% ✓/✗
3. High RVP > 60% ✓/✗
4. Large IVP-RVP gap < -15 pts ✓/✗
5. Technical breakout confirmed (BOT BK ≥ 3/4) ✓/✗

DOW score: X/5 → A+ (5/5) / Strong (4/5) / Acceptable (3/5) / Weak (≤2/5)

#### Gate 3 Final Verdict

| Stage 2 | Stage 3 (if reached) | Gate 3 Verdict | Meaning |
|:---:|:---:|:---:|---------|
| 5/5 | DOW ≥ 4/5 | **PASS — A+** | Highest conviction vol setup |
| 4-5/5 | DOW 3/5 + dims mostly favorable | **PASS** | Good vol setup |
| 3/5 | DOW ≥ 3/5 + dims confirm | **PASS (marginal)** | Acceptable but monitor closely |
| 3/5 | DOW < 3/5 or dims unfavorable | **FAIL** | Surface OK but deep analysis doesn't confirm |
| ≤ 2/5 | Not reached | **FAIL** | Weak vol profile |
| NO DATA | — | **INCOMPLETE** | Cannot assess — needs TWS data |

**Always explain**: state which specific criteria passed/failed with actual values. Example: "FAIL — IV30 42% (above 40% threshold), IVP 72% (above 60%), VRP +5.2 with high IVP (overpaying). Only term structure (contango) is favorable. 1/5 criteria met."

### GATE 4: Preliminary Viability

**Purpose**: Quick sanity checks before committing to trade preparation. Structure details (strikes, expiry, contracts, delta) are NOT assessed here — they belong in `/trade-prep`.

| Criterion | Type | Rule | How to Check |
|-----------|:---:|------|-------------|
| **Asymmetry articulable** | Hard | Write the R/R in 1 sentence before entry. If you can't, skip. Trades with noted asymmetry: +$349 avg vs +$51 without | **Ask user**: "In one sentence, what is the asymmetry?" |
| **No sector overconcentration** | Soft | Max 2 BOTs in same sector | **Ask user**: "How many open BOTs in [sector]?" |
| **No earnings in 4-week window** | Hard | Earnings date must be > 4 weeks from today | Check earnings calendar (ask user if unsure) |
| **No macro event within 24h** | Hard | FOMC, CPI, OpEx | Cross-reference macro_context events section |
| **Entry day** | Info | If today is NOT Tuesday or Wednesday, warn: "Entry day is <day> — historically Tue/Wed are best (+$177/$53 avg). Are you sure you want to proceed today?" | Check day of week. Warning only, not a gate failure. |

**Output**: PASS or FAIL + reason. Entry day warning is informational only.

## Step 4: Produce Terminal Verdict

Print a compact summary to the terminal:

```
══════════════════════════════════════════════════════════════
  /analyze <TICKER> <DIRECTION>    <DATE>
══════════════════════════════════════════════════════════════
  SECTOR: <sector>  |  PRICE: $<price>

  GATE 1  Macro Regime     <PASS ✓ / FAIL ✗>  <conviction>
          S5FI: <value>% | VIX: <value> | Bias: <bias>
          <sector tailwinds/headwinds if any>

  GATE 2  Scanner Quality  <PASS ✓ / FAIL ✗>  BOT: S:<X>/6 BK:<Y>/4
          RS vs ETF: <value> | Sector WR: <X>% | BOT flags: <summary>

  GATE 3  Vol Profile      <PASS ✓ / PASS~ / FAIL ✗ / INCOMPLETE ?>
          Stage 2: <X>/5 criteria met
          IV30: <X>% | IVP: <X>% | RVP: <X>% | VRP: <X> | TermStr: <contango/backw>
          <If Stage 3 reached>:
          DOW pattern: <X>/5 (<A+/Strong/Acceptable/Weak>)
          4-Dim: [1:IVP]<✓/✗> [2:Peer]<✓/✗/?> [3:Term]<✓/✗> [4:Skew]<✓/?> 
          <Explanation: why PASS or why FAIL with actual values>

  GATE 4  Viability        <PASS ✓ / FAIL ✗>
          Asymmetry: <stated> | Concentration: <OK/warning> | Events: <clear/warning>
          Earnings window: <clear / BLOCKED>
          <If not Tue/Wed>: ⚠ Entry day is <day> — unusual

──────────────────────────────────────────────────────────────
  VERDICT: <GO / NO-GO / CONDITIONAL>
  
  Conviction: <HIGH / MEDIUM / LOW>
  Edge source: <risk premia / vol mispricing / behavioural / institutional>
  DOW pattern: <A+ / Strong / Acceptable / Weak / None>
  
  <If GO>: Proceed to /trade-prep for structure (strikes, expiry, contracts, BS grid)
  <If NO-GO>: <reason — which gate failed>
  <If CONDITIONAL>: <what conditions must be met>
══════════════════════════════════════════════════════════════
```

**Conviction logic** — price action first, vol pricing is a bonus:

Start at 0, add points:
- Gate 2 BOT S:6 BK:4 (perfect): +2 pts. S:5+ BK:3+: +1 pt.
- Gate 3 Stage 2 ≥ 4/5: +1 pt (options are not fighting you).
- Gate 1 conviction HIGH/PREFERRED: +1 pt. CAUTION: 0.

Subtract points (soft flags):
- RSI < 30 or > 70 (oversold/overbought): -1 pt
- M3 fails (near 20d low/high): -1 pt
- Entry day not Tue/Wed: -0.5 pt

**Result**: HIGH ≥ 3 pts | MEDIUM 1.5-2.5 pts | LOW < 1.5 pts
**Cap**: Gate 1 CAUTION caps conviction at MEDIUM.
**DOW pattern**: Does not affect conviction. Report it separately as "edge amplifier" — it tells you how much leverage you're getting, not whether the trade works.

**Edge source identification**:
- Negative VRP + low IVP + high RVP = **vol mispricing** (DOW pattern)
- Sector in capitulation/euphoria zone = **behavioural bias** (overreaction)
- Mismatch detected in macro context = **institutional flow** (rotation)
- High VIX + contango normalizing = **risk premia** (selling fear)

## Step 5: Produce HTML Report (MANDATORY unless --no-html)

**IMPORTANT**: Always generate the HTML report, even for NO-GO verdicts. This preserves a record of the decision at this point in time.

Generate an HTML file at:
```
C:/Users/aldoh/Documents/NewTrading/reports/analyze_<TICKER>_<DATE>.html
```

**Colorblind-safe CSS palette** (MANDATORY — user is colorblind):
```css
:root {
  --pass-bg: #0072B2;       /* Blue — replaces green */
  --pass-light: rgba(0,114,178,.12);
  --fail-bg: #D55E00;       /* Vermillion — replaces red */
  --fail-light: rgba(213,94,0,.10);
  --warn-bg: #E69F00;       /* Amber — replaces orange */
  --warn-light: rgba(230,159,0,.10);
  --skip-bg: #666666;       /* Grey — skipped/incomplete */
  --skip-light: rgba(102,102,102,.10);
}
.badge-pass{background:var(--pass-bg)}
.badge-fail{background:var(--fail-bg)}
.badge-warn{background:var(--warn-bg)}
.badge-skip{background:var(--skip-bg)}
```
Use `--pass-light` for passing row backgrounds, `--fail-light` for failing rows, `--warn-light` for warnings. Do NOT use the old green-bg/red-bg/orange-bg variables.

**Gate 2 criterion labels** — use the canonical BOT naming (S1-S6 for setup, BK1-BK4 for breakout):

**Setup phase (S1-S6)** — is the base built?

| ID | Indicator | Condition | What it means |
|:--:|-----------|-----------|---------------|
| S1 | Price vs MA50 | `price > ma50` | Stock in uptrend — trading above 50-day average |
| S2 | MA50 slope | `ma50_slope > 0` | Trend strengthening — the average itself is rising |
| S3 | RS vs sector ETF | `rs > 0` | Outperforming its sector — relative strength built |
| S4 | OBV slope (20d) | `obv_slope > 0` | Institutional accumulation — volume confirms price |
| S5 | Squeeze ratio | `rng20/rng40 < 0.65` | Range contracting — coiled spring, energy building |
| S6 | Volume decline | `vol20/vol50 < 0.95` | Supply drying up — sellers exhausted before breakout |

**Breakout phase (BK1-BK4)** — is it breaking out NOW?

| ID | Indicator | Condition | What it means |
|:--:|-----------|-----------|---------------|
| BK1 | RSI + slope | `rsi14>50 && rsi_slope>0` | Momentum accelerating — buyers taking control |
| BK2 | Up/Down volume | `updn_ratio > 1.1` | Buying pressure — more volume on up days than down |
| BK3 | Range position | `rng_pct >= 70%` | Pushing against resistance — price near 20d high |
| BK4 | Volume surge | `today_vol/20d_avg >= 1.2` | Breakout volume — conviction behind the move |

**Supplementary (swing-only, not BOT criteria)**:

| ID | Indicator | Condition | What it means |
|:--:|-----------|-----------|---------------|
| ADX/DI | Directional conviction | `adx10>20 && dip>din` | Trend has directional strength, not just drift |
| MA dist | Distance to MA50 | `ma50_dist <= 15%` | Not overextended — room before mean reversion |
| Room | Room to run | `(high20-price)/atr > 1` | Not pinned to resistance — at least 1 ATR of upside |

Do NOT use the old T1/T2/M1/M2/RS1/V1/V2 aliases — one indicator, one name.

**HTML tooltips (MANDATORY)** — each indicator `<td>` in the HTML report MUST have a `title` attribute with a self-contained explanation: what it measures, how it's computed, and what the threshold means. Use these exact tooltips:

```
S1: "Uptrend filter. Is price above the 50-day moving average? MA50 smooths out noise — trading above it means the medium-term trend is up."
S2: "Trend momentum. Is the 50-day MA itself rising (5-day slope > 0)? A rising MA means the trend is accelerating, not just holding."
S3: "Relative strength. 20-day return of this stock minus 20-day return of its sector ETF. Positive = outperforming peers. Measures stock-specific strength vs sector drift."
S4: "Accumulation. On-Balance Volume (OBV) slope over 20 days. OBV adds volume on up-days, subtracts on down-days. Rising OBV = institutional buying — smart money accumulating before breakout."
S5: "Squeeze detection. 20-day price range divided by 40-day range. Below 0.65 = range has contracted to less than 65% of its recent norm — a coiled spring storing energy for a breakout move."
S6: "Supply exhaustion. 20-day average volume divided by 50-day average. Below 0.95 = volume declining — sellers are drying up, fewer shares changing hands before the breakout."
BK1: "Momentum ignition. RSI(14) above 50 AND its 5-day slope positive. Buyers have taken control and momentum is accelerating — the breakout is starting."
BK2: "Buying pressure. Sum of volume on up-days divided by sum on down-days (10-day window). Above 1.1 = more volume flows into up-moves than down-moves — demand exceeds supply."
BK3: "Resistance test. Current price position within the 20-day high-low range, as percentage. Above 70% = price is pushing against the top of its recent range — testing resistance."
BK4: "Breakout volume. Today's volume divided by 20-day average. Above 1.2 = 20%+ surge in participation — conviction behind the move, not just drift."
ADX/DI: "Directional conviction. ADX(10) measures trend strength (>20 = trending), DI+/DI- measures direction. Both together confirm the trend has real directional momentum, not just range drift."
MA dist: "Extension check. How far price is from MA50, as %. Below 15% = not overextended. Stocks too far above MA50 tend to mean-revert before continuing."
Room: "Upside room. Distance from price to 20-day high, measured in ATR units. Above 1.0 = at least one ATR of space before hitting recent resistance ceiling."
```

**Terse notes rule** (MANDATORY):
- A note cell should ONLY contain information NOT deducible from the value and ✓/✗ next to it.
- Do NOT restate thresholds ("above 60% threshold"), do NOT explain what ✓/✗ means ("favorable", "acceptable", "not overpaying"), do NOT add filler ("options are cheap in absolute terms").
- If the value + ✓/✗ tells the full story, leave the note **empty** or use 2-3 words max.
- Good note: "momentum fading" (adds context beyond the number). Bad note: "Well below threshold — options cheap. Below winner avg 27.7%." (restates what 27.4% ✓ already says).
- Good note: "stock near 20-day ceiling" (explains why ✗). Bad note: "slightly expensive but acceptable. Not overpaying." (redundant).

Use the same CSS style as macro_context and swing_scanner (`.section`, `.stitle`, `.snum`, badge classes, zone classes) but with the colorblind-safe palette above. The HTML report should contain:

1. **Header**: Ticker, direction, date **and time** (HH:MM), price, sector
2. **Gate Summary**: Visual pass/fail badges for each gate (same as terminal but with color coding)
3. **Gate 3 Detail**: Full 4-dimension IV table with values, DOW pattern scoring breakdown
4. **Macro Context Summary**: Relevant extracts from macro_context (bias, VIX, breadth, events, sector tailwinds/headwinds)
5. **Scanner Data**: Key technical metrics from CSV row
6. **Verdict**: Final recommendation with conviction and edge source
7. **Historical Reference**: Sector historical WR and avg P&L

**Open the HTML file automatically if running interactively.**

## Step 6: Edge Source Commentary

After the verdict, provide a brief narrative (3-5 sentences) explaining:
- What specific edge exists (or doesn't) for this trade
- How this compares to historical winners in the same sector
- What the main risk is
- If GO: what would make you exit early (the "no asymmetry" signal for this specific setup)

## Notes

- Gates are sequential: 1 → 2 → 3 → 4. Stop at first FAIL unless `--force` flag is used.
- Gate 4 requires user interaction (asymmetry statement, open positions count). Ask these questions clearly.
- If the scanner CSV is more than 2 days old, warn the user that data may be stale.
- If vol data is missing, don't fabricate numbers. Report INCOMPLETE and suggest TWS refresh.
- Never recommend a trade. Present the evidence and let the user decide.
- All monetary values are in the trade currency (USD, EUR, or CHF). Risk budget is CHF 600 max per trade.
