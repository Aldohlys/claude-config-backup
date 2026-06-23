# WHEEL Strategy Deep Dive Analysis

## Strategy Overview
**Total Performance**: +$3,020.10 across 124 trades (23.4% win rate)
**Average Trade**: +$24.36
**Best Trade**: +$371.60 (AI put spreads)
**Worst Trade**: -$2,158.10 (DSY covered calls)

---

## Key Success vs Failure Patterns

### **🟢 Successful WHEEL Patterns**

**Top Performers (>$300 profit):**
1. **AI Options** (Multiple): +$371.60, +$350.80, +$341.13
   - "Automatic 50% profit close"
   - "Automatic profit take trade close"
   - Clean execution, systematic exits

2. **Position Characteristics**:
   - **Small positions** (1-3 contracts): Average +$100-175
   - **Put selling**: Consistent premium collection
   - **Automatic exits**: 50% profit targets hit reliably

**Success Factors:**
- **Systematic profit-taking**: "Automatic 50% profit close"
- **Quality underlying**: AI showed strong technical behavior
- **Appropriate sizing**: 1-2 contracts optimal
- **Clear exit rules**: No emotional decision-making

### **🔴 Failure Patterns**

**Major Losers:**
1. **DSY Covered Calls** (-$2,158.10): Large position size (5 contracts)
2. **HOLN Series** (-$214.60): Complex rolling strategy gone wrong
   - "I sold calls that were a loss in the end... would have been +345 CHF instead of -214 CHF"
3. **TTE Rolling Disaster**: Multiple rolls, small losses accumulating

**Failure Factors:**
- **Oversizing**: Position size 5 = -$337.94 average loss
- **Complex rolling**: Multiple adjustments led to confusion
- **Covered call timing**: Selling calls at wrong market timing
- **Emotional decisions**: "Finally, finally... target price reached"

---

## Position Analysis

| Position Type | Trades | Avg P&L | Total P&L | Success Rate |
|---------------|--------|---------|-----------|--------------|
| **Long 1-3** | 46 | +$100.30 | +$4,610 | ✅ High |
| **Short 1-3** | 53 | +$0.00 | $0 | ⚠️ Neutral |
| **Size 5+** | 6 | -$337.94 | -$2,028 | ❌ Poor |

**Key Insight**: Small long positions (buying back puts/calls) highly profitable, while oversized positions consistently failed.

---

## Instrument Performance

**Winners:**
- **AI (C3.ai)**: Dominated top 10 trades - excellent technical behavior for wheel
- **SIE (Siemens)**: Single large winner (+$341.13)
- **ITA (Aerospace ETF)**: Recovered well from August volatility

**Problematic:**
- **DSY**: Catastrophic covered call assignment
- **HOLN**: Complex European stock rolling issues
- **TTE**: Multiple small losses from over-management

---

## Temporal Patterns

**Best Months:**
- **February 2025**: +$70.16 avg - Clean profit-taking
- **July 2024**: +$113.88 avg - Strong trending markets
- **August 2024**: +$82.50 avg - Post-volatility recovery

**Worst Months:**
- **August 2025**: -$266.89 avg - DSY disaster month
- **May 2025**: -$107.30 avg - HOLN rolling failure

---

## Critical Success Factors

### **✅ What Works:**
1. **Systematic Exits**: "Automatic 50% profit close" = consistent wins
2. **Quality Stocks**: AI showed predictable wheel behavior
3. **Size Discipline**: 1-2 contracts optimal risk/reward
4. **Put-focused**: Selling puts outperformed covered calls
5. **No Rolling**: Simple execution beats complex adjustments

### **❌ What Fails:**
1. **Oversizing**: 5+ contracts = guaranteed losses
2. **Complex Rolling**: Multiple adjustments create confusion
3. **Covered Calls**: Poor timing on call-selling side
4. **Emotional Exits**: "Finally, finally" suggests impatience
5. **European Stocks**: Currency/liquidity complications (TTE, HOLN)

---

## Strategic Recommendations

### **Immediate Actions:**
1. **Cap Position Size**: Maximum 3 contracts per wheel
2. **Focus on Puts**: Avoid covered call side initially
3. **Systematic Exits**: Implement automatic 50% profit targets
4. **Stock Selection**: Focus on liquid US tech (AI template)

### **Risk Management:**
1. **No Rolling**: Close losing positions rather than rolling
2. **Monthly Review**: Identify performance patterns
3. **Profit Banking**: Take profits systematically, don't get greedy
4. **Avoid Complexity**: Simple wheel execution only

### **Instrument Strategy:**
- **Increase**: AI, tech stocks with clear trends
- **Avoid**: European stocks (TTE, HOLN), low-liquidity names
- **Monitor**: Position sizing impact on all trades

**Bottom Line**: WHEEL works best as a simple, systematic put-selling strategy on quality stocks with disciplined exits. Complexity and oversizing kill performance.