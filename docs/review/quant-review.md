# Quant Strategist Technical Review -- TradingClaude SMC Suite v1.0

> Reviewed: 2026-02-12
> File: `src/TradingClaude_SMC_Suite.pine` (1361 lines)
> Reviewer: Quant Strategist

---

## Executive Summary

The implementation is **solid overall** and addresses the majority of gaps identified in the gap analysis. The anti-repainting architecture is well-designed with a centralized `barstate.isconfirmed` guard. However, I found **3 bugs**, **4 medium-risk issues**, and **several minor improvements** needed before publication.

**Verdict: CONDITIONAL PASS -- fix the 3 bugs and the critical repainting issue before release.**

---

## 1. Anti-Repainting Verification

### 1.1 request.security() Calls (14 total)

All 14 `request.security()` calls are centralized at global scope (L350-406). This is excellent practice -- no security calls buried inside functions.

| Line | Data | Offset | Lookahead | Safe? | Notes |
|------|------|--------|-----------|:-----:|-------|
| L355 | FVG data | `close[1], open[1], time[1], high[1], low[1], time[0], high[3], low[3]` | `lookahead_on` | YES | Fixed from LuxAlgo original. All price data uses `[1]` or `[3]` offsets. `time[0]` is only for positioning, not signal logic. |
| L358 | Daily levels | `high[1], low[1], time[1], time` | `lookahead_on` | YES | `time` (no offset) is for visual positioning only. |
| L361 | Weekly levels | `high[1], low[1], time[1], time` | `lookahead_on` | YES | Same pattern as daily. |
| L364 | Monthly levels | `high[1], low[1], time[1], time` | `lookahead_on` | YES | Same pattern as daily. |
| L396 | HTF pivot high | `ta.pivothigh(5)[1]` | `lookahead_on` | YES | `[1]` offset applied to pivothigh result. |
| L397 | HTF pivot low | `ta.pivotlow(5)[1]` | `lookahead_on` | YES | `[1]` offset applied to pivotlow result. |
| L398 | HTF close | `close[1]` | `lookahead_on` | YES | Standard safe pattern. |
| L399 | HTF high | `high[1]` | `lookahead_on` | YES | Standard safe pattern. |
| L400 | HTF low | `low[1]` | `lookahead_on` | YES | Standard safe pattern. |
| L401 | HTF open | `open[1]` | `lookahead_on` | YES | Standard safe pattern. |
| L402 | HTF time | `time[1]` | `lookahead_on` | YES | Standard safe pattern. |
| L405 | HTF high[3] | `high[3]` | `lookahead_on` | YES | For FVG detection, 3-bar lookback. |
| L406 | HTF low[3] | `low[3]` | `lookahead_on` | YES | For FVG detection, 3-bar lookback. |

**Result: ALL 14 request.security() calls use safe anti-repainting patterns. PASS.**

### 1.2 barstate.isconfirmed Guard

The main signal detection block at L1191 wraps ALL critical logic:

```
if barstate.isconfirmed              // L1191
    f_ms_getCurrentStructure(...)    // L1193-1194
    f_ms_getCurrentStructure(...)    // EQH/EQL - L1197
    f_ms_displayStructure(...)       // L1200, L1203
    f_ob_checkMitigation(...)        // L1207, L1209
    f_fvg_detect()                   // L1213
    f_liq_detectSweeps()             // L1217
    f_mtf_processHTFStructure()      // L1221
    f_confluence_detect()            // L1224
```

**Result: All signal detection is gated by barstate.isconfirmed. PASS.**

### 1.3 Remaining Repainting Risks

#### **BUG #1 (MEDIUM): FVG partial fill check runs outside isconfirmed guard**

**Location:** L1183-1184
```pinescript
if showFairValueGapsInput
    f_fvg_checkFill()    // <-- runs on EVERY tick, not just confirmed bars
```

**Problem:** `f_fvg_checkFill()` (L686-720) modifies `eachFVG.top` / `eachFVG.bottom` and deletes FVGs based on current bar's `low` and `high` values. On an unconfirmed bar, `low` might wick into an FVG temporarily and then retrace, causing the FVG box to shrink or disappear prematurely. On the next tick the damage is done -- the FVG top/bottom have been permanently altered.

**Fix:** Move `f_fvg_checkFill()` inside the `barstate.isconfirmed` block (between L1211 and L1213), or add a separate `barstate.isconfirmed` guard around it.

**Severity: MEDIUM** -- This is a repainting risk. FVG boxes will visually flicker and their stored state (top/bottom) can be permanently corrupted by intra-bar wicks.

#### **BUG #2 (LOW): Liquidity line extension runs outside isconfirmed**

**Location:** L1187-1188
```pinescript
if showLiquiditySweepsInput and showEqualHighsLowsInput
    f_liq_extendLines()
```

**Assessment:** This is ACCEPTABLE. `f_liq_extendLines()` (L467-475) only updates visual line endpoints to `last_bar_time` -- it doesn't change any signal state. Running on every tick is fine for visual smoothness. **NO FIX NEEDED.**

#### **BUG #3 (LOW): Premium/Discount runs outside isconfirmed**

**Location:** L1175-1180
```pinescript
if showHighLowSwingsInput or showPremiumDiscountZonesInput
    f_pd_updateTrailingExtremes()
    ...
    f_pd_drawPremiumDiscountZones()
```

**Assessment:** `f_pd_updateTrailingExtremes()` (L746-750) updates `trailing.top` / `trailing.bottom` on every tick. However, these are trailing extremes (only ever go up for top, down for bottom), so intra-bar updates are valid -- a new high IS a new extreme even if unconfirmed, and the values monotonically expand.

**However**, `f_pd_drawPremiumDiscountZones()` at L794-801 updates `pdPositionPct`, `priceInPremium`, `priceInDiscount` and fires alerts (`premiumEntry`, `discountEntry`). The alert flags will be set on unconfirmed bars and then read by the `alertcondition()` calls.

**Fix:** The `alertcondition()` calls at L1280-1281 will only fire at bar close per TradingView's design, so the flag being set early is harmless for `alertcondition()`. But the `alert()` calls at L1353-1357 are inside `barstate.isconfirmed` so they are safe. The `pdPositionPct` value updating on every tick is also OK since it's only used for dashboard display.

**Assessment: LOW RISK, acceptable behavior. No fix strictly needed**, but for consistency the premium/discount alerts should be computed inside the isconfirmed block. The trailing extremes update can remain outside.

### 1.4 Swing Detection Retroactive Analysis

`f_ms_leg()` (L561-569) uses `high[size] > ta.highest(size)` -- same logic as LuxAlgo. Since this is now inside `barstate.isconfirmed` (called via `f_ms_getCurrentStructure()` at L1193), the risk is fully mitigated. `ta.highest(size)` evaluates on confirmed data only.

**Result: PASS.**

---

## 2. Signal Logic Correctness

### 2.1 BOS/CHoCH Detection

**Location:** `f_ms_displayStructure()` L620-677

**Logic review:**
- Bullish BOS: `ta.crossover(close, p_ivot.currentLevel) and not p_ivot.crossed` (L632)
  - When trend is already BULLISH, tag = BOS. When trend is BEARISH, tag = CHoCH.
  - After crossing, `p_ivot.crossed := true` prevents re-triggering.
  - Trend bias updates to BULLISH after the cross.
- Bearish BOS: `ta.crossunder(close, p_ivot.currentLevel) and not p_ivot.crossed` (L657)
  - Mirror logic for bearish.

**Assessment: CORRECT.** The `crossed` flag prevents duplicate triggers. The trend direction update ensures CHoCH is only triggered on the FIRST break against the prevailing trend.

**Minor concern:** `ta.crossover()` and `ta.crossunder()` use `close` vs `close[1]` internally. Since this runs inside `barstate.isconfirmed`, both values are confirmed. SAFE.

**Note on internal structure:** The confluence filter at L623-625 uses body/wick ratio:
```pinescript
bullishBar := high - math.max(close, open) > math.min(close, open - low)
```
This formula has a subtle issue: `open - low` could be negative if the bar is bullish (close > open), making `math.min(close, negative_value)` always return the negative value. However, since this is wrapped in `math.min()`, it produces the lower wick correctly: `close - low` when bullish, `open - low` when bearish. Actually, re-reading: `math.min(close, open - low)` -- this is NOT computing the lower wick. `open - low` is the distance, not a price. This should probably be `math.min(close, open) - low` for the lower wick.

#### **BUG #2 (MEDIUM): Confluence filter formula is incorrect**

**Location:** L624-625
```pinescript
bullishBar := high - math.max(close, open) > math.min(close, open - low)
bearishBar := high - math.max(close, open) < math.min(close, open - low)
```

**Problem:** The right side `math.min(close, open - low)` computes the minimum of `close` (a price, e.g., 1850) and `open - low` (a distance, e.g., 5). This is comparing apples to oranges. The intent is to compare upper wick to lower wick.

**Correct formula should be:**
```pinescript
bullishBar := high - math.max(close, open) > math.min(close, open) - low
bearishBar := high - math.max(close, open) < math.min(close, open) - low
```

Where:
- `high - math.max(close, open)` = upper wick length
- `math.min(close, open) - low` = lower wick length

**Impact:** This is the same bug as in the LuxAlgo original (L387-388). The filter defaults to `false` (disabled), so it does not affect most users. But when enabled, it will produce incorrect filtering -- `math.min(close, open - low)` will almost always be `open - low` (a small number), making the comparison effectively `upper_wick > small_number`, which will be true for almost all bars. So the bullishBar filter is overly permissive and bearishBar is overly restrictive.

**Severity: MEDIUM** -- Only affects users who enable "Confluence Filter". The default is off. But it should be fixed.

### 2.2 Order Block Detection

**Location:** `f_ob_store()` L481-495

**Logic review:**
- For BEARISH OB: finds the bar with MAX parsed high between pivot bar and current bar.
- For BULLISH OB: finds the bar with MIN parsed low between pivot bar and current bar.
- Uses `parsedHighs`/`parsedLows` arrays (L340-341) which swap high/low on high-volatility bars -- this is the LuxAlgo approach to handle engulfing candles.
- OB is stored only when a BOS is confirmed (called from `f_ms_displayStructure()` at L652/L677).

**Assessment: CORRECT.** The OB is identified as the last institutional candle before the impulse that caused the BOS. The parsedHighs/parsedLows inversion for high-volatility bars is a reasonable heuristic.

**Mitigation logic** (`f_ob_checkMitigation()` L497-522):
- Iterates reverse to handle index shifting on removal.
- Skips already-mitigated OBs (`state == 1`).
- Uses configurable mitigation source (close or high/low).
- When `hideMitigatedOBsInput` is false, sets `state := 1` instead of removing.

**Assessment: CORRECT.** The reverse iteration with bounds check (`index >= orderBlocks.size()`) is proper for in-loop modification.

**Drawing logic** (`f_ob_draw()` L524-555):
- Mitigated OBs: opacity 95%, dotted border, `extend.none`.
- Unmitigated OBs: normal opacity, solid border, `extend.right`.
- Unused pre-allocated boxes are cleared to `na` (L552-555).

**Assessment: CORRECT and matches the gap analysis spec.**

### 2.3 FVG Calculation

**Location:** `f_fvg_detect()` L722-740

**Logic review:**
- Bullish FVG: `fvgCurrentLow > fvgLast2High` -- current candle's low is above the candle-from-2-bars-ago's high. Gap between candle 1 top wick and candle 3 bottom wick.
- Also requires `fvgLastClose > fvgLast2High` -- the middle candle's close must be above the gap.
- Threshold filter: cumulative bar delta percentage.

**Assessment: CORRECT.** The FVG definition matches standard SMC methodology.

**However**, the variable naming is misleading after the anti-repainting offset change. With `[1]` offsets applied:
- `fvgCurrentHigh` and `fvgCurrentLow` are actually `high[1]` and `low[1]` on the HTF -- the PREVIOUS completed HTF bar, not the current one.
- `fvgLast2High` and `fvgLast2Low` are `high[3]` and `low[3]` -- 3 HTF bars ago.
- `fvgLastClose` is `close[1]` -- previous HTF bar.

So the FVG is detected between HTF bar [3] and HTF bar [1], with bar [2] as the impulse. This is correct -- it's looking at completed data only. The naming just doesn't match the semantic offset.

**Partial fill tracking** (`f_fvg_checkFill()` L686-720):
- Bullish FVG: if `low <= originalBottom` -> fully filled, delete. If `low < top` -> partial fill, shrink.
- Bearish FVG: if `high >= originalTop` -> fully filled, delete. If `high > bottom` -> partial fill, shrink.
- Dual-box approach correctly updates midpoint for visual split.

**Assessment: CORRECT** logic, but see Bug #1 above about the isconfirmed guard.

### 2.4 Liquidity Sweep Detection

**Location:** `f_liq_detectSweeps()` L442-465

**Logic review:**
- BSL sweep (bearish): `high > lvl.price AND close < lvl.price` -- wick above BSL level but close below. (L450)
- SSL sweep (bullish): `low < lvl.price AND close > lvl.price` -- wick below SSL level but close above. (L458)
- Marks level as swept, sets alert flag, draws "SWEEP" label.
- Line stops extending at sweep bar.

**Assessment: CORRECT.** The sweep detection logic matches the gap analysis spec exactly. The `close` comparison ensures we only detect true sweeps (fake breakouts), not genuine breakouts.

**Edge case check:** The bounds check at L445 (`index >= liquidityLevels.size()`) handles potential array modification during iteration. The reverse iteration (L444: `size() - 1 to 0`) is correct for this pattern.

**Integration with EQH/EQL:** New liquidity levels are created inside `f_liq_drawEqualHighLow()` (L424-440) when a new equal high/low is detected. This is well-integrated.

---

## 3. MTF Implementation

### 3.1 Auto HTF Detection

**Location:** `f_mtf_getAutoHTF()` L367-382

**Mapping verification:**

| Current TF (seconds) | Condition | HTF Selected | Design Spec | Match? |
|----------------------|-----------|:------------:|:-----------:|:------:|
| 1m (60) | <= 300 | 60 (1H) | 1H | YES |
| 5m (300) | <= 300 | 60 (1H) | 1H | YES |
| 15m (900) | <= 900 | 60 (1H) | 1H | YES |
| 30m (1800) | <= 3600 | 240 (4H) | Not in spec (between 15m and 1H) | ACCEPTABLE |
| 1H (3600) | <= 3600 | 240 (4H) | 4H | YES |
| 4H (14400) | <= 14400 | D | 1D | YES |
| 1D (86400) | <= 86400 | W | 1W | YES |
| 1W (604800) | > 86400 | M | 1M | YES |

**Assessment: CORRECT.** All specified mappings match. The 30m case (not explicitly in the design) maps to 4H which is reasonable.

### 3.2 HTF Data Fetching

All HTF security calls (L396-406) use:
- `barmerge.gaps_off` -- correct, avoids NaN gaps between HTF bars.
- `barmerge.lookahead_on` with `[1]` offsets -- correct anti-repainting pattern.
- `ta.pivothigh(5)[1]` / `ta.pivotlow(5)[1]` -- the `[1]` is applied AFTER the pivothigh calculation, so it returns the previous confirmed pivot. This is safe.

**Assessment: PASS.**

### 3.3 HTF Structure Processing

**Location:** `f_mtf_processHTFStructure()` L903-997

**Logic review:**
- Detects new HTF swings by comparing current `htfSwH`/`htfSwL` to stored `prevHTFSwH`/`prevHTFSwL`.
- HTF BOS/CHoCH uses `htfClose > htfPivotHigh` / `htfClose < htfPivotLow` -- same BOS logic as LTF.
- OBs are created on HTF BOS events using `htfHigh_v`/`htfLow_v` as the OB candle boundaries.

**Concern:** The HTF OB candle (L939: `htfHigh_v`/`htfLow_v`) uses the SAME HTF bar that generated the BOS signal, not the last opposing candle before the impulse. In LTF code, `f_ob_store()` searches back through `parsedHighs`/`parsedLows` to find the actual OB candle. The HTF implementation takes a shortcut by using the most recent HTF bar data.

**Impact: LOW.** HTF OBs are approximations anyway since we're projecting them on the LTF chart. The zone will be close to the correct location but may not exactly align with the true HTF order block candle. This is acceptable for v1.0 but should be noted for future improvement.

### 3.4 HTF FVG Detection

**Location:** L970-995

**Logic review:**
- Bullish HTF FVG: `htfLow_v > htfHigh3_v` -- HTF bar [1]'s low > HTF bar [3]'s high.
- Bearish HTF FVG: `htfHigh_v < htfLow3_v` -- HTF bar [1]'s high < HTF bar [3]'s low.
- Uses `prevHTFClose != htfClose` as a change detector to avoid duplicate FVGs.

**Assessment: CORRECT logic.** The FVG gap is between the correct bars with the [1] offset applied.

**Minor concern:** The dedup check `prevHTFClose != htfClose` at L972/L984 could miss FVGs if two consecutive HTF bars have the same close price (rare but possible). A better guard would be to check `htfTime_v` change, but this is a very minor edge case.

---

## 4. Dashboard Data Accuracy

### 4.1 Row-by-Row Verification

**Row 0 -- Header (L1095-1096):** Static text "TRADINGCLAUDE" + "SMC SUITE". CORRECT.

**Row 1 -- Trend (L1100-1106):**
- LTF trend: sources from `internalTrend.bias`. CORRECT -- this is updated by `f_ms_displayStructure(true)`.
- HTF trend: sources from `htfTrendBias`. CORRECT -- updated by `f_mtf_processHTFStructure()` at L997.
- Color: based on LTF trend. CORRECT per design.

**Row 2 -- Structure (L1109-1114):**
- Last BOS: sources from `lastBOSPrice` / `lastBOSBias`. CORRECT -- updated at L643-644 / L668-669 inside `f_ms_displayStructure()`.
- Last CHoCH: sources from `lastCHoCHPrice` / `lastCHoCHBias`. CORRECT -- updated at L646-647 / L671-672.
- Formats with `format.mintick` for proper decimal display. CORRECT.

**Row 3 -- Zones (L1117-1124):**
- OB count: `internalOrderBlocks.size() + swingOrderBlocks.size()`. This counts ALL OBs including mitigated ones (when `hideMitigatedOBsInput` is false). Slightly misleading -- the design says "active" zones.
- **Minor issue:** Should count only unmitigated OBs for a more accurate "active" count.
- FVG count: `fairValueGaps.size()`. CORRECT -- only unfilled/partially-filled FVGs are in the array.
- HTF counts: from array sizes. CORRECT.

**Row 4 -- Liquidity (L1127-1145):**
- Scans `liquidityLevels` for nearest unswept BSL above price and nearest unswept SSL below price.
- Logic at L1135-1140: BSL = `bias == BEARISH` and `price > close`; SSL = `bias == BULLISH` and `price < close`.
- Finds the NEAREST by comparing against current nearest. CORRECT.

**Row 5 -- Position (L1148-1156):**
- Range: `trailing.top - trailing.bottom`. CORRECT.
- Position %: `(close - trailing.bottom) / rangeV * 100`. CORRECT.
- EQ level: `math.avg(trailing.top, trailing.bottom)`. CORRECT.
- Zone text: Premium if > 50%, Discount otherwise. CORRECT.

**Row 6 -- Signal (L1159-1163):**
- Shows confluence text if `isConfluence` is true, otherwise "No active signal".
- `isConfluence` is set at L1225 from `f_confluence_detect()`.
- **Minor issue:** `confluenceText` persists across bars (it's a `var` at L286). If a confluence was detected on bar N, the dashboard will show "No active signal" on bar N+1 but `confluenceText` still holds the old value. This is fine because the display is gated by `isConfluence`, and `isConfluence` is reset by `f_confluence_detect()` on each confirmed bar. Wait -- actually `isConfluence` is also a `var` (L288). It gets reassigned at L1225 on each `barstate.isconfirmed` bar, so it will correctly reset to false when no confluence is detected. CORRECT.

### 4.2 Dashboard Update Timing

**Location:** L1252
```pinescript
if barstate.isconfirmed or barstate.islast
    f_dash_update()
```

This updates the dashboard on confirmed bars AND on the last (current) bar. This means the dashboard shows real-time data even on unconfirmed bars, which is good for UX (live updating). The data sources (trend, structure, zones) are only modified inside `barstate.isconfirmed`, so the dashboard shows the latest confirmed state plus real-time position data. **This is the correct behavior.**

---

## 5. Edge Cases

### 5.1 Minimal Data

**Concern:** What happens on a weekly chart with very little history (e.g., 50 bars)?

- `ta.atr(200)` at L337: With < 200 bars, `ta.atr(200)` returns `na` for the first 199 bars. This makes `atrMeasure = na`, which makes `volatilityMeasure = na` (L338), `highVolatilityBar = na` (L339), and `parsedHigh/parsedLow = na` (L340-341). The arrays will be populated with `na` values.
- `f_ob_store()` (L481) does `parsedHighs.slice(p_ivot.barIndex, bar_index)`. If the array contains `na` values, `a_rray.max()` / `a_rray.min()` will handle them (Pine ignores `na` in array min/max). But `a_rray.indexof(na_result)` could return -1 if all values are `na`.

**Impact:** On very early bars, OB detection may produce incorrect results or errors. Not a crash risk (Pine handles `na` gracefully) but the OBs may be positioned incorrectly.

**Recommendation:** Add a guard: `if bar_index > 200` before enabling OB detection, or use `nz(volatilityMeasure, ta.tr)` as a fallback.

### 5.2 Maximum Objects

**Assessment of object counts:**

| Object Type | Max from Code | Sources | Risk |
|-------------|:------------:|---------|:----:|
| Boxes (OB internal) | 20 (input max) | Pre-allocated at L331 | LOW |
| Boxes (OB swing) | 20 (input max) | Pre-allocated at L329 | LOW |
| Boxes (FVG) | Unlimited (array grows) | Created in `f_fvg_detect()` | **HIGH** |
| Boxes (HTF OB) | 10 (input max) | Capped at L933/L958 | LOW |
| Boxes (HTF FVG) | 10 (input max) | Capped at L973/L985 | LOW |
| Boxes (P/D zones) | 3 (var, reused) | `f_pd_drawZone()` | LOW |
| Lines (structure) | Unlimited | Created in `f_ms_drawStructure()` L576 | **HIGH** |
| Lines (EQH/EQL display) | 2 (var, reused) | `equalHighDisplay`/`equalLowDisplay` | LOW |
| Lines (liquidity) | 50 (input max) | `maxLiquidityLevelsInput` | LOW |
| Lines (fib) | 5 (var, reused) | `f_pd_drawFibLevel()` | LOW |
| Lines (MTF levels) | 6 (var, reused) | Daily/Weekly/Monthly top+bottom | LOW |
| Lines (HTF structure) | Unlimited | Created in L930/L955 | **MEDIUM** |
| Lines (strong/weak H/L) | 2 (var, reused) | `f_pd_drawHighLowSwings()` | LOW |
| Labels (structure) | Unlimited | Created in `f_ms_drawStructure()` L577 | **HIGH** |
| Labels (swing points) | Unlimited | Created in L601/L618 | **MEDIUM** |
| Labels (sweep) | Unlimited | Created in L457/L465 | **MEDIUM** |
| Labels (HTF structure) | Unlimited | Created in L931/L956 | **MEDIUM** |
| Table | 1 (var, reused) | Dashboard | LOW |

#### **BUG #3 (HIGH): No object count limits for BOS/CHoCH lines and labels**

**Problem:** `f_ms_drawStructure()` (L575-577) creates a new `line.new()` and `label.new()` on EVERY BOS/CHoCH event. On a 1m chart with several months of data, internal structure events can easily produce 500+ lines and 500+ labels, hitting Pine Script's `max_lines_count = 500` and `max_labels_count = 500` limits.

When the limit is hit, Pine automatically deletes the oldest objects. This means early BOS/CHoCH markers silently disappear, which is acceptable behavior but may confuse users who scroll back in history and see missing structure.

**Note:** The LuxAlgo original had the same issue but mitigated it with the "Present" mode (L263-264, L327-329) which deleted old objects. The TradingClaude implementation removed the Present/Historical mode toggle, so ALL objects accumulate.

**FVG boxes** also have no cap -- `fairValueGaps` array grows without limit. However, FVGs are deleted when filled, so the practical count is bounded by the number of simultaneously active FVGs.

**Fix recommendations:**
1. Add a max display count for structure lines/labels (e.g., last 50 BOS/CHoCH events).
2. Or re-implement a cleanup mechanism similar to LuxAlgo's "Present" mode that deletes oldest objects.
3. For FVGs, add a max cap: `if fairValueGaps.size() >= maxFVGsInput, delete oldest`.

**Severity: HIGH for 1m charts.** On 15m+ charts the count is unlikely to exceed limits in typical usage.

### 5.3 All Modules On at 1m

**Scenario:** BTCUSDT 1m chart, all modules enabled, default settings.

**Object budget estimate (per 1000 bars):**
- Internal BOS/CHoCH: ~100 events (1 line + 1 label each) = ~100 lines + ~100 labels
- Swing BOS/CHoCH: ~20 events = ~20 lines + ~20 labels
- Swing point labels: ~40 = ~40 labels
- OB boxes: 5 internal + 5 swing = 10 boxes (pre-allocated)
- FVG boxes: ~20 active at once = ~40 boxes (2 per FVG)
- Liquidity lines/labels: 10 max = 10 lines + 10 labels
- Sweep labels: ~10 = ~10 labels
- HTF structure lines/labels: ~5 = ~5 lines + ~5 labels
- HTF OB boxes: 3 max = ~3 boxes
- HTF FVG boxes: 3 max = ~3 boxes
- P/D zones: 3 boxes + 5 fib lines = 3 boxes + 5 lines
- MTF levels: 6 lines + 6 labels
- Dashboard: 1 table

**Totals for 1000 bars:**
- Lines: ~146 (well under 500)
- Labels: ~191 (well under 500)
- Boxes: ~59 (well under 500)

**For 5000 bars (typical 1m chart load):**
- Lines: ~530 (OVER 500 limit, oldest silently deleted)
- Labels: ~755 (OVER 500 limit)

**Conclusion:** On 1m with all modules, Pine will hit object limits after ~3000-4000 bars. This reinforces the need for object cleanup (Bug #3).

### 5.4 All Inputs Disabled

When all module toggles are off:
- No structure detection runs (gated by `showInternalsInput`/`showStructureInput` etc.)
- No OBs, FVGs, liquidity, P/D
- Dashboard will show but with default/empty values
- Alert conditions will all be `false`

**Assessment: SAFE.** No errors expected.

### 5.5 Division by Zero

- L338: `ta.cum(ta.tr) / bar_index` -- on bar_index = 0, this divides by zero. Pine handles this gracefully (returns `na`). However, `bar_index` is 0 on the very first bar, and `ta.cum()` also returns the first value. This could propagate `na` through `volatilityMeasure` for the first bar only. **LOW RISK.**

- L726: `ta.cum(...) / bar_index * 2` -- same concern. **LOW RISK.**

- L1150: `rangeV > 0` guard prevents division by zero for position %. **CORRECT.**

---

## 6. Additional Findings

### 6.1 OB Zone Count in Dashboard

**Location:** L1118
```pinescript
obCount = internalOrderBlocks.size() + swingOrderBlocks.size()
```

This counts ALL OBs including mitigated ones. When `hideMitigatedOBsInput` is false, mitigated OBs remain in the array with `state == 1`. The dashboard should ideally show only unmitigated (active) OBs.

**Fix:** Count only `state == 0` OBs, or show both: "OB: 3 active (2 mitigated)".

### 6.2 HTF OB Mitigation Not Tracked

HTF OBs (stored in `htfOBBoxes`/`htfOBTops`/`htfOBBots`) are never checked for mitigation. They persist indefinitely until pushed out by the max zone limit. If price passes through an HTF OB, it remains displayed as active.

**Fix:** Add mitigation check for HTF OBs similar to `f_ob_checkMitigation()`.

### 6.3 HTF FVG Fill Not Tracked

Same issue as HTF OBs. HTF FVGs are never checked for fill. They persist until pushed out.

**Fix:** Add fill check for HTF FVGs.

### 6.4 Confluence Filter Bug Inherited from LuxAlgo

As noted in Bug #2, the confluence filter formula at L624-625 is incorrect. This is inherited directly from the LuxAlgo original (L387-388).

### 6.5 `var` Keyword on Fib Lines

**Location:** `f_pd_drawFibLevel()` L775-776
```pinescript
var line fibLine   = line.new(...)
var label fibLabel = label.new(...)
```

This function is called 5 times (once per fib level) at L789-793. However, `var` means the variable is initialized only ONCE across all calls. So all 5 fib levels will share the SAME line and label objects -- each call overwrites the previous one's position.

**Result:** Only the LAST fib level (0.236) will be visible. The previous 4 (0.786, 0.705, 0.618, 0.382) are drawn but immediately overwritten.

#### **BUG #3b (HIGH): All Fibonacci levels share the same var line/label objects**

**Fix:** Remove the `var` keyword and create new objects each time, or pre-allocate 5 separate line/label pairs:
```pinescript
// Option A: Remove var, create new objects each bar (will hit limits faster)
f_pd_drawFibLevel(float price, string tag, color lineColor, string style) =>
    line fibLine = line.new(...)
    label fibLabel = label.new(...)

// Option B: Pre-allocate 5 pairs (better)
var line[] fibLines = array.new_line(5)
var label[] fibLabels = array.new_label(5)
// Then pass index to drawFibLevel
```

**Severity: HIGH** -- The Fibonacci sub-levels feature is visually broken. Only 1 of 5 levels appears.

---

## 7. Summary of Issues

### Bugs (Must Fix Before Release)

| # | Severity | Location | Description | Fix Complexity |
|---|:--------:|----------|-------------|:--------------:|
| B1 | MEDIUM | L1183-1184 | FVG partial fill runs outside `barstate.isconfirmed` -- box state can be corrupted by intra-bar wicks | LOW (move 2 lines) |
| B2 | MEDIUM | L624-625 | Confluence filter formula uses `math.min(close, open - low)` instead of `math.min(close, open) - low` | LOW (change 2 chars) |
| B3 | HIGH | L575-577 | No object count limits for BOS/CHoCH lines and labels -- will hit Pine limits on 1m charts | MEDIUM (add array tracking + cleanup) |
| B3b | HIGH | L775-776 | Fibonacci levels all share same `var` objects -- only 1 of 5 levels visible | MEDIUM (pre-allocate 5 pairs) |

### Medium-Risk Issues (Should Fix Before Release)

| # | Risk | Location | Description |
|---|:----:|----------|-------------|
| M1 | MEDIUM | L903-968 | HTF OBs use same-bar data as OB candle (shortcut) instead of searching for the actual opposing candle |
| M2 | MEDIUM | -- | HTF OB mitigation not tracked -- mitigated HTF OBs persist as active |
| M3 | MEDIUM | -- | HTF FVG fill not tracked -- filled HTF FVGs persist as active |
| M4 | LOW | L1118 | Dashboard OB count includes mitigated OBs |

### Minor Issues (Can Go to Backlog)

| # | Location | Description |
|---|----------|-------------|
| m1 | L972/L984 | HTF FVG dedup uses close comparison instead of time change |
| m2 | L337-341 | No fallback for `ta.atr(200)` returning `na` on early bars |
| m3 | L401 | `htfOpen_v` is fetched but never used in the code |

---

## 8. Final Assessment

| Category | Grade | Notes |
|----------|:-----:|-------|
| Anti-repainting | **A-** | Excellent architecture. One FVG fill issue (B1). |
| Signal correctness | **B+** | Core logic is correct. Confluence filter bug (B2) is inherited from LuxAlgo. |
| MTF implementation | **B** | Auto-detection correct. HTF zones lack mitigation/fill tracking (M2, M3). |
| Dashboard | **A-** | All data sources correct. Minor OB count issue (M4). |
| Object management | **C+** | Missing limits on structure lines/labels (B3). Fib levels broken (B3b). |
| Edge cases | **B** | Handles most cases. Early-bar na propagation is a minor concern. |

**Overall: B+ -- Solid implementation that needs 4 targeted fixes (B1, B2, B3, B3b) to be release-ready.**
