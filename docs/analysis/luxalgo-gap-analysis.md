# LuxAlgo SMC vs TradingClaude SMC Suite -- Gap Analysis

> Generated: 2026-02-12
> Source: `src/reference/luxalgo-smc-original.pine` (601 lines)
> Target: `docs/plans/2026-02-12-smc-suite-design.md`

---

## 1. Feature Mapping Table

| # | Feature | LuxAlgo Implementation | TradingClaude Design Requirement | Gap Status |
|---|---------|----------------------|--------------------------------|:----------:|
| 1 | **Swing Detection** | `leg(size)` L246-255, `getCurrentStructure()` L283-322 using `ta.highest/ta.lowest` with configurable size | `f_detectSwings()` using `ta.pivothigh()/ta.pivotlow()` with configurable swing length | PARTIAL |
| 2 | **Internal Structure (BOS/CHoCH)** | `displayStructure(internal=true)` L383-427, size=5 fixed at L539 | `f_detectBOS()` + `f_detectCHoCH()` with configurable internal length | COMPLETE |
| 3 | **Swing Structure (BOS/CHoCH)** | `displayStructure(internal=false)` L383-427, uses `swingsLengthInput` L92 | `f_detectBOS()` + `f_detectCHoCH()` for swing level | COMPLETE |
| 4 | **Trend Direction** | `trend` type L169-170, `swingTrend`/`internalTrend` updated in `displayStructure()` L404/L422 | Trend direction with HH/HL/LH/LL sequence + ranging detection | PARTIAL |
| 5 | **Order Blocks** | `storeOrdeBlock()` L352-366, `deleteOrderBlocks()` L333-350, `drawOrderBlocks()` L368-381 | `f_findOrderBlocks()` with unmitigated/mitigated visual distinction | PARTIAL |
| 6 | **Fair Value Gaps** | `drawFairValueGaps()` L438-450, `deleteFairValueGaps()` L431-436 | `f_findFVG()` with partial fill tracking (box reduces dynamically) | PARTIAL |
| 7 | **Equal Highs/Lows** | `drawEqualHighLow()` L267-281, detected via `getCurrentStructure(len, true)` L542 | Liquidity pools (EQH/EQL) with BSL/SSL labeling | PARTIAL |
| 8 | **Liquidity Sweeps** | NOT IMPLEMENTED | Sweep detection: wick past level + close back, BSL/SSL labels, visual "SWEEP" marker | MISSING |
| 9 | **Premium/Discount Zones** | `drawPremiumDiscountZones()` L515-519, 3 zones (5%/5%/5% bands) | Premium/Discount with Fibonacci sub-zones (0.618, 0.705, 0.786) | PARTIAL |
| 10 | **MTF Levels** | `drawLevels()` L458-483, D/W/M previous highs/lows | Auto HTF detection + HTF structure overlay + HTF OB/FVG projection + confluence detection | PARTIAL |
| 11 | **Dashboard** | NOT IMPLEMENTED | Full `table.new()` dashboard with 6 rows: Trend, Structure, Zones, Liquidity, Position, Signal | MISSING |
| 12 | **Confluence Alerts** | NOT IMPLEMENTED (only individual `alertcondition()` L580-599) | Multi-signal confluence detection + dynamic `alert()` with context messages | MISSING |
| 13 | **Swing Labels (HH/HL/LH/LL)** | In `getCurrentStructure()` L305/L322, optional via `showSwingsInput` | Required as part of structure module, labels for all swing types | COMPLETE |
| 14 | **Strong/Weak High/Low** | `drawHighLowSwings()` L493-506, labels "Strong High"/"Weak High" etc. | Implied by structure module (trailing extremes tracking) | COMPLETE |
| 15 | **Color Candles (Trend)** | `plotcandle()` L526 colored by `internalTrend.bias` | Not explicitly required but useful -- keep as optional toggle | COMPLETE |
| 16 | **Individual Alerts** | 16 `alertcondition()` calls L580-599 covering BOS/CHoCH/OB/EQH/EQL/FVG | 10 alert categories including OB Touch, FVG Fill, Sweep, Confluence, Premium/Discount entry | PARTIAL |

### Summary

| Status | Count | Percentage |
|--------|:-----:|:----------:|
| COMPLETE | 5 | 31% |
| PARTIAL | 8 | 50% |
| MISSING | 3 | 19% |

---

## 2. What to KEEP from LuxAlgo (adapt/refactor)

### 2.1 Market Structure Engine

**Functions to keep:**

- `leg(int size)` (L246-255) -- Core pivot detection using `ta.highest()`/`ta.lowest()`. The approach of detecting directional legs is sound and efficient. Refactor: rename to `f_detectLeg()`, add `barstate.isconfirmed` guard.

- `startOfNewLeg()` / `startOfBearishLeg()` / `startOfBullishLeg()` (L257-259) -- Clean helper functions using `ta.change()`. Keep as-is.

- `getCurrentStructure(int size, bool equalHighLow, bool internal)` (L283-322) -- Core function that processes pivots, updates swing levels, and triggers EQH/EQL detection. This is the backbone. Refactor: split into separate functions per responsibility (`f_updateSwings()`, `f_checkEqualHL()`).

- `displayStructure(bool internal)` (L383-427) -- BOS/CHoCH detection and order block creation. The dual-purpose (internal + swing) design is clever and should be preserved. Refactor: extract BOS/CHoCH detection into `f_detectBOS()` / `f_detectCHoCH()`, keep the trend update logic, add `barstate.isconfirmed` guard.

- `drawStructure()` (L324-331) -- Drawing logic for BOS/CHoCH lines and labels. Refactor: update styles per design spec (BOS = dashed width 1, CHoCH = solid width 2).

**Data structures to keep:**

- `pivot` type (L176-181) -- Tracks `currentLevel`, `lastLevel`, `crossed`, `barTime`, `barIndex`. Clean and extensible.
- `trend` type (L169-170) -- Simple but effective. Extend with `ranging` state.
- `trailingExtremes` type (L154-160) -- Tracks global swing high/low. Essential for Premium/Discount.
- `alerts` type (L136-152) -- Boolean flags per alert. Extend with new alert types.

### 2.2 Order Blocks

**Functions to keep:**

- `storeOrdeBlock(pivot, bool internal, int bias)` (L352-366) -- Identifies the OB candle by finding the max/min in the parsed arrays between the pivot and current bar. The `parsedHighs`/`parsedLows` approach (L234-235) that handles high-volatility bars is clever. Refactor: rename to `f_storeOrderBlock()`, add state tracking (mitigated vs unmitigated).

- `deleteOrderBlocks(bool internal)` (L333-350) -- Mitigation check using configurable source (close or high/low, L229-230). Refactor: instead of deleting, mark as mitigated and change visual style (dotted border, reduced opacity per design spec).

- `drawOrderBlocks(bool internal)` (L368-381) -- Box rendering from array. Keep the approach. Refactor: add visual distinction for mitigated OBs.

**Data structures to keep:**

- `orderBlock` type (L183-188) -- Fields: `barHigh`, `barLow`, `barTime`, `bias`. Extend with: `int state` (0=unmitigated, 1=mitigated), `box b_ox` reference.
- `swingOrderBlocks` / `internalOrderBlocks` arrays (L206-207) -- Separate tracking by type is good.
- Pre-allocated box arrays `swingOrderBlocksBoxes` / `internalOrderBlocksBoxes` (L208-209, initialized at L222-227) -- Good pattern to avoid re-creating boxes.

### 2.3 Fair Value Gaps

**Functions to keep:**

- `drawFairValueGaps()` (L438-450) -- FVG detection with `request.security()` for MTF FVGs. The threshold filtering (L442) using cumulative bar delta is a good quality filter. Refactor: add partial fill tracking (update box size when price fills part of the gap).

- `deleteFairValueGaps()` (L431-436) -- Removal when fully penetrated. Refactor: instead of binary delete, implement 3 states: `unfilled` -> `partially filled` -> `filled`.

- `fairValueGapBox()` (L429) -- Helper for box creation. Keep.

**Data structures to keep:**

- `fairValueGap` type (L162-168) -- Fields: `top`, `bottom`, `bias`, `topBox`, `bottomBox`. The dual-box approach (top half + bottom half) creates the visual split effect. Extend with: `float fillLevel` for partial fill tracking.

### 2.4 Equal Highs/Lows

**Functions to keep:**

- `drawEqualHighLow(pivot, float level, int size, bool equalHigh)` (L267-281) -- Clean detection using ATR-based threshold (`equalHighsLowsThresholdInput * atrMeasure`, L291/L308). Good approach. Refactor: add BSL/SSL labeling per design spec (currently labels are "EQH"/"EQL", change to "BSL"/"SSL" or add both).

### 2.5 Premium/Discount Zones

**Functions to keep:**

- `drawPremiumDiscountZones()` (L515-519) -- Uses `trailing.top`/`trailing.bottom` for range. The 5% band at extremes is simplistic. Refactor: expand to full premium/discount with Fibonacci sub-levels.

- `updateTrailingExtremes()` (L487-491) -- Updates global high/low tracking. Keep as-is.

- `drawZone()` (L508-513) -- Generic zone-drawing helper with box + label. Keep and reuse.

### 2.6 MTF Levels

**Functions to keep:**

- `drawLevels(string timeframe, bool sameTimeframe, string style, color levelColor)` (L458-483) -- Draws previous period high/low for D/W/M. The approach of using `request.security()` with `lookahead_on` for `[1]` data is standard. Refactor: extend to support auto-HTF detection per design table.

- `higherTimeframe(string timeframe)` (L485) -- Checks if current TF is higher than target. Useful guard. Keep.

### 2.7 Utility Patterns

- **Mode switching** (Historical/Present, L73): Good UX pattern. In "Present" mode, old labels/lines are deleted before drawing new ones (e.g., L263-264, L327-329). Keep this pattern.
- **Style switching** (Colored/Monochrome, L74): Nice accessibility feature. Keep and extend with TradingClaude palette.
- **Inline input groups** (L78-79): Good TradingView UX. Keep grouping pattern, reorganize into 8 groups per design.

---

## 3. What's MISSING (new features to build)

### 3.a Liquidity Sweep Detection

**Context:** LuxAlgo detects Equal Highs/Lows (L267-281) but has NO mechanism to detect when price sweeps past these levels and reverses. This is one of the highest-value SMC signals.

**Implementation Specification:**

```
Data Model:
-----------
type liquidityLevel
    float price          // the EQH/EQL price level
    int   bias           // BULLISH (SSL) or BEARISH (BSL)
    int   barTime        // when the level was established
    int   barIndex
    bool  swept          // has this level been swept?
    int   sweepBarTime   // when the sweep occurred
    line  l_ine          // visual line object
    label l_abel         // visual label object

var array<liquidityLevel> liquidityLevels = array.new<liquidityLevel>()
```

```
Detection Logic (runs on barstate.isconfirmed):
-----------------------------------------------
For each unswept liquidityLevel in liquidityLevels:

  Bearish Sweep (BSL sweep -- price wicks above buy-side liquidity):
    Condition: high > level.price AND close < level.price
    Meaning:   Price swept above the BSL (grabbed buy-side liquidity)
               and closed back below -- bearish signal
    Action:    Mark level.swept = true
               Draw label "SWEEP" at high of the candle
               Set currentAlerts.bearishSweep = true

  Bullish Sweep (SSL sweep -- price wicks below sell-side liquidity):
    Condition: low < level.price AND close > level.price
    Meaning:   Price swept below the SSL (grabbed sell-side liquidity)
               and closed back above -- bullish signal
    Action:    Mark level.swept = true
               Draw label "SWEEP" at low of the candle
               Set currentAlerts.bullishSweep = true
```

```
BSL/SSL Labeling Logic:
-----------------------
When a new Equal High is detected (from getCurrentStructure with equalHighLow=true):
  -> Create liquidityLevel with bias=BEARISH, label text="BSL"
  -> This is BUY-SIDE liquidity (stops of shorts sit above equal highs)
  -> Color: #FF5252 (bearish red)

When a new Equal Low is detected:
  -> Create liquidityLevel with bias=BULLISH, label text="SSL"
  -> This is SELL-SIDE liquidity (stops of longs sit below equal lows)
  -> Color: #00BCD4 (bullish cyan)
```

```
Visual Representation:
---------------------
Unswept level:
  - Horizontal dotted line from detection bar extending right
  - Label "BSL" or "SSL" at the right end
  - Color: cyan (#00BCD4) for SSL, light red (#FF5252) for BSL

Swept level:
  - Line stops extending at the sweep bar
  - "X" marker or cross at the sweep point
  - Label "SWEEP" above/below the candle that swept
  - Label size: size.tiny, style: label.style_label_down (BSL) or label.style_label_up (SSL)

Cleanup:
  - Max levels tracked: configurable input (default 10)
  - Swept levels can optionally be hidden after N bars
```

```
Integration Points:
-------------------
- Feed sweep events into the confluence detection engine (Section 3.d)
- A sweep + OB touch in the same zone = very high probability signal
- A sweep + FVG fill = strong reversal signal
- Display sweep count in dashboard (Section 3.b)
```

### 3.b Dashboard (Table)

**Context:** LuxAlgo has NO dashboard. The design requires a comprehensive on-screen table showing real-time status of all modules.

**Implementation Specification:**

```
Table Structure:
---------------
Position: Configurable (top_right default)
Size: Configurable (size.small default)
Rows: 7 (header + 6 data rows)
Columns: 2 (label + value)

var table dashboard = table.new(
    position.top_right,
    columns = 2,
    rows = 7,
    bgcolor = color.new(#363A45, 15),
    border_width = 1,
    border_color = color.new(#363A45, 50),
    frame_width = 2,
    frame_color = color.new(#363A45, 30)
)
```

```
Row Layout and Data Sources:
----------------------------

Row 0 -- HEADER:
  Cell(0,0): "TRADINGCLAUDE SMC SUITE", colspan conceptual (use 2 cells)
  Color: #D1D4DC text, #363A45 bg

Row 1 -- TREND:
  Cell(1,0): "Trend"
  Cell(1,1): LTF trend + HTF trend
  Data source:
    LTF: internalTrend.bias (from displayStructure)
         BULLISH -> "LTF: Bullish" green
         BEARISH -> "LTF: Bearish" red
    HTF: htfTrend.bias (from MTF module)
         BULLISH -> "HTF: Bullish" green
         BEARISH -> "HTF: Bearish" red
  Format: "LTF: Bullish | HTF: Bearish"
  Cell color: based on LTF trend

Row 2 -- STRUCTURE:
  Cell(2,0): "Structure"
  Cell(2,1): Last BOS + Last CHoCH with levels
  Data source:
    Track last BOS price and direction in displayStructure()
    Track last CHoCH price and direction in displayStructure()
    var float lastBOSPrice = na
    var int lastBOSBias = 0
    var float lastCHoCHPrice = na
    var int lastCHoCHBias = 0
  Format: "BOS: 1.0845 (Bull) | CHoCH: 1.0790"

Row 3 -- ZONES:
  Cell(3,0): "Zones"
  Cell(3,1): Active OB count + Active FVG count (separate LTF/HTF)
  Data source:
    OB count: internalOrderBlocks.size() + swingOrderBlocks.size()
    FVG count: fairValueGaps.size()
    HTF counts: from MTF module arrays
  Format: "OB: 3 (1 HTF) | FVG: 2 (1 HTF)"

Row 4 -- LIQUIDITY:
  Cell(4,0): "Liquidity"
  Cell(4,1): Nearest BSL and SSL levels
  Data source:
    Scan liquidityLevels array for nearest unswept BSL above price
    Scan liquidityLevels array for nearest unswept SSL below price
  Format: "BSL: 1.0900 | SSL: 1.0750"

Row 5 -- POSITION:
  Cell(5,0): "Position"
  Cell(5,1): Current price position in premium/discount + EQ level
  Data source:
    range = trailing.top - trailing.bottom
    position_pct = (close - trailing.bottom) / range * 100
    eq_level = (trailing.top + trailing.bottom) / 2
    zone = position_pct > 50 ? "Premium" : "Discount"
  Format: "Discount (38.2%) | EQ: 1.0825"
  Cell color: green if discount, red if premium

Row 6 -- SIGNAL:
  Cell(6,0): "Signal"
  Cell(6,1): Current confluence signals (if any)
  Data source:
    Check if any confluence conditions are active this bar:
      - OB touch + FVG overlap
      - Sweep + OB zone
      - HTF alignment + LTF signal
    If none: "No active signal"
  Format: "OB + FVG confluence at 1.0800 (HTF aligned)"
  Cell color: highlighted yellow/gold if active signal
```

```
Update Logic:
-------------
// Only update on confirmed bars to avoid flicker
if barstate.isconfirmed or barstate.islast
    updateDashboard()

f_updateDashboard() =>
    if showDashboardInput
        // Row 1: Trend
        ltfTrendText = internalTrend.bias == BULLISH ? "Bullish" : "Bearish"
        ltfTrendColor = internalTrend.bias == BULLISH ? color_bullish : color_bearish
        table.cell(dashboard, 1, 0, "Trend", text_color=#D1D4DC, text_size=dashboardSize)
        table.cell(dashboard, 1, 1, "LTF: " + ltfTrendText, text_color=ltfTrendColor, text_size=dashboardSize)
        // ... repeat for each row

Inputs:
  dashboardPositionInput  = input.string("Top Right", "Position", options=["Top Right","Top Left","Bottom Right","Bottom Left"], group=DASHBOARD_GROUP)
  showDashboardInput      = input(true, "Show Dashboard", group=DASHBOARD_GROUP)
  showTrendRowInput       = input(true, "Show Trend Row", group=DASHBOARD_GROUP)
  showStructureRowInput   = input(true, "Show Structure Row", group=DASHBOARD_GROUP)
  showZonesRowInput       = input(true, "Show Zones Row", group=DASHBOARD_GROUP)
  showLiquidityRowInput   = input(true, "Show Liquidity Row", group=DASHBOARD_GROUP)
  showPositionRowInput    = input(true, "Show Position Row", group=DASHBOARD_GROUP)
  showSignalRowInput      = input(true, "Show Signal Row", group=DASHBOARD_GROUP)
  dashboardSizeInput      = input.string("Small", "Size", options=["Small","Normal","Large"], group=DASHBOARD_GROUP)
```

### 3.c Enhanced MTF

**Context:** LuxAlgo has basic MTF via `drawLevels()` (D/W/M previous highs and lows only, L458-483). The design requires full HTF structure analysis projected onto the current chart.

**Implementation Specification:**

```
3.c.1 Auto HTF Detection:
--------------------------
f_getAutoHTF() =>
    currentTFSeconds = timeframe.in_seconds()
    if currentTFSeconds <= 300        // 1m - 5m
        "60"                          // -> 1H
    else if currentTFSeconds <= 900   // 15m
        "60"                          // -> 1H
    else if currentTFSeconds <= 3600  // 1H
        "240"                         // -> 4H
    else if currentTFSeconds <= 14400 // 4H
        "D"                           // -> 1D
    else if currentTFSeconds <= 86400 // 1D
        "W"                           // -> 1W
    else
        "M"                           // -> 1M

Inputs:
  htfModeInput = input.string("Auto", "HTF Timeframe",
      options=["Auto","15","60","240","D","W","M"],
      group=MTF_GROUP)
  htfTimeframe = htfModeInput == "Auto" ? f_getAutoHTF() : htfModeInput
```

```
3.c.2 HTF Structure Overlay (BOS/CHoCH from HTF):
--------------------------------------------------
Strategy: Use request.security() to fetch HTF OHLC data, then run
the same structure detection logic on that data.

// Fetch HTF candle data
[htfOpen, htfHigh, htfLow, htfClose, htfTime] = request.security(
    syminfo.tickerid, htfTimeframe,
    [open[1], high[1], low[1], close[1], time[1]],
    barmerge.gaps_off, barmerge.lookahead_on
)

// Run swing detection on HTF data
// Note: Cannot call ta.pivothigh/ta.highest inside request.security
// tuple return. Must compute HTF pivots from the returned series.

Alternative approach (more robust):
  Compute swings, BOS, CHoCH entirely inside request.security
  by packaging the logic into a single expression block:

[htfSwingHigh, htfSwingLow, htfBOS, htfCHoCH, htfTrendBias] =
    request.security(syminfo.tickerid, htfTimeframe,
        f_computeHTFStructure(),  // returns tuple of structure data
        barmerge.gaps_off, barmerge.lookahead_on)

Visual:
  - HTF BOS/CHoCH lines: same as LTF but with width+1 and "[HTF]" label prefix
  - HTF lines use line.style_solid with width=2
  - Color: same bullish/bearish palette
```

```
3.c.3 HTF OB/FVG Projection on Current Chart:
----------------------------------------------
// HTF Order Blocks: fetch the OB levels from HTF
[htfOBTop, htfOBBottom, htfOBBias, htfOBTime] = request.security(
    syminfo.tickerid, htfTimeframe,
    [ob_top[1], ob_bottom[1], ob_bias[1], ob_time[1]],
    barmerge.gaps_off, barmerge.lookahead_on
)

// Draw HTF OB on current chart as box
// Differentiation from LTF OBs:
//   - Border width: 2 (vs 1 for LTF)
//   - Opacity: LTF opacity + 10%
//   - Label: "[4H] OB" or "[1D] OB" etc.

// HTF FVG: same approach
[htfFVGTop, htfFVGBottom, htfFVGBias] = request.security(
    syminfo.tickerid, htfTimeframe,
    [fvg_top[1], fvg_bottom[1], fvg_bias[1]],
    barmerge.gaps_off, barmerge.lookahead_on
)

// Max HTF zones to display: configurable (default 3)
```

```
3.c.4 Confluence Detection HTF+LTF:
------------------------------------
A confluence signal fires when a LTF signal occurs within/near a HTF zone.

f_checkConfluence() =>
    confluence = false
    confluenceText = ""

    // Check: LTF OB touch at a level that overlaps with HTF OB
    for each ltfOB in orderBlocks:
        for each htfOB in htfOrderBlocks:
            if f_zonesOverlap(ltfOB, htfOB)
                confluence := true
                confluenceText := "OB + HTF OB"

    // Check: LTF BOS/CHoCH aligns with HTF trend direction
    if currentAlerts.swingBullishCHoCH and htfTrendBias == BULLISH
        confluence := true
        confluenceText := "CHoCH + HTF Trend"

    // Check: Sweep near HTF OB/FVG zone
    if currentAlerts.bullishSweep or currentAlerts.bearishSweep
        for each htfZone in htfZones:
            if priceNearZone(close, htfZone)
                confluence := true
                confluenceText := "Sweep + HTF Zone"

    [confluence, confluenceText]

f_zonesOverlap(zone1, zone2) =>
    zone1.top >= zone2.bottom and zone1.bottom <= zone2.top
```

### 3.d Confluence Alerts

**Context:** LuxAlgo has 16 individual `alertcondition()` calls (L580-599) but NO multi-signal confluence detection and NO dynamic `alert()` messages.

**Implementation Specification:**

```
3.d.1 Multi-Signal Confluence Detection:
-----------------------------------------
// Run after all individual detections are complete

confluenceCount = 0
confluenceSignals = ""

// Count simultaneous signals
if currentAlerts.swingBullishCHoCH or currentAlerts.swingBearishCHoCH
    confluenceCount += 1
    confluenceSignals += "CHoCH "

if currentAlerts.internalBullishOrderBlock or currentAlerts.internalBearishOrderBlock
    or currentAlerts.swingBullishOrderBlock or currentAlerts.swingBearishOrderBlock
    confluenceCount += 1
    confluenceSignals += "OB_Touch "

if currentAlerts.bullishFairValueGap or currentAlerts.bearishFairValueGap
    confluenceCount += 1
    confluenceSignals += "FVG "

if currentAlerts.bullishSweep or currentAlerts.bearishSweep  // new
    confluenceCount += 1
    confluenceSignals += "Sweep "

if priceInDiscount  // new
    confluenceCount += 1
    confluenceSignals += "Discount "
else if priceInPremium  // new
    confluenceCount += 1
    confluenceSignals += "Premium "

if htfAligned  // new
    confluenceCount += 1
    confluenceSignals += "HTF_Aligned "

isConfluence = confluenceCount >= 2
```

```
3.d.2 Dynamic alert() with Context:
------------------------------------
// Use alert() (not alertcondition()) for dynamic messages

if isConfluence and barstate.isconfirmed
    bias = determineBias()  // aggregate direction from all signals
    biasText = bias == BULLISH ? "BULLISH" : "BEARISH"

    alertMessage = "TradingClaude SMC Suite\n"
        + "CONFLUENCE ALERT (" + str.tostring(confluenceCount) + " signals)\n"
        + "Direction: " + biasText + "\n"
        + "Signals: " + confluenceSignals + "\n"
        + "Price: " + str.tostring(close) + "\n"
        + "Timeframe: " + timeframe.period + "\n"
        + "HTF Trend: " + (htfTrendBias == BULLISH ? "Bullish" : "Bearish") + "\n"
        + "Position: " + (priceInPremium ? "Premium" : "Discount")
            + " (" + str.tostring(positionPct, "#.#") + "%)"

    alert(alertMessage, alert.freq_once_per_bar)
```

```
3.d.3 Individual Alert Messages (Enhanced):
--------------------------------------------
// Keep alertcondition() for each signal (for TradingView's alert setup UI)
// But ALSO fire alert() with richer messages

// Example for CHoCH:
if currentAlerts.swingBullishCHoCH and barstate.isconfirmed
    alert("TradingClaude: CHoCH Bullish at " + str.tostring(close)
        + " | TF: " + timeframe.period
        + " | HTF: " + (htfTrendBias == BULLISH ? "Aligned" : "Counter"),
        alert.freq_once_per_bar)

// Example for Sweep:
if currentAlerts.bullishSweep and barstate.isconfirmed
    alert("TradingClaude: SSL Sweep at " + str.tostring(close)
        + " | Liquidity grabbed below " + str.tostring(sweepLevel)
        + " | TF: " + timeframe.period,
        alert.freq_once_per_bar)

Inputs:
  enableBOSAlertsInput          = input(true, "BOS Alerts", group=ALERTS_GROUP)
  enableCHoCHAlertsInput        = input(true, "CHoCH Alerts", group=ALERTS_GROUP)
  enableOBAlertsInput           = input(true, "OB Alerts", group=ALERTS_GROUP)
  enableFVGAlertsInput          = input(true, "FVG Alerts", group=ALERTS_GROUP)
  enableSweepAlertsInput        = input(true, "Liquidity Alerts", group=ALERTS_GROUP)
  enableConfluenceAlertsInput   = input(true, "Confluence Alerts", group=ALERTS_GROUP)
  enablePDZoneAlertsInput       = input(false, "Premium/Discount Alerts", group=ALERTS_GROUP)
  alertFormatInput              = input.string("Detailed", "Message Format",
                                    options=["Simple","Detailed"], group=ALERTS_GROUP)
```

---

## 4. Repainting Risk Analysis

### 4.1 `request.security()` Calls

**Location: `drawFairValueGaps()` L439**
```
[...] = request.security(syminfo.tickerid, fairValueGapsTimeframeInput,
    [close[1], open[1], time[1], high[0], low[0], time[0], high[2], low[2]],
    lookahead = barmerge.lookahead_on)
```

- **RISK: MEDIUM-HIGH.** Uses `barmerge.lookahead_on` but fetches `high[0]` and `low[0]` (current bar data). With `lookahead_on`, this means on historical bars it will use data from the NEXT HTF bar's high/low, which is future data. On realtime bars, it uses the current bar's incomplete data.
- **FIX:** Change `high[0]` to `high[1]` and `low[0]` to `low[1]`, or switch to `barmerge.lookahead_off` and adjust all offsets. The cleanest fix:
  ```
  request.security(..., [close[1], open[1], time[1], high[1], low[1], time[0], high[3], low[3]],
      lookahead = barmerge.lookahead_on)
  ```
  Note: This shifts all data by 1 bar, which is the standard anti-repainting pattern with `lookahead_on`.

**Location: `drawLevels()` L459**
```
[topLevel, bottomLevel, leftTime, rightTime] = request.security(syminfo.tickerid, timeframe,
    [high[1], low[1], time[1], time],
    lookahead = barmerge.lookahead_on)
```

- **RISK: LOW.** Uses `high[1]`/`low[1]`/`time[1]` with `lookahead_on` -- this is the correct anti-repainting pattern. The `time` (current bar) is only used for positioning, not for signal logic.
- **FIX:** None needed for signal data. The `time` (no offset) is acceptable here since it is only used for visual right-boundary of the level lines.

### 4.2 `barstate.isconfirmed` Usage

**RISK: HIGH.** The LuxAlgo code does NOT use `barstate.isconfirmed` anywhere in its core logic.

- `getCurrentStructure()` (L283-322) runs on EVERY tick, meaning swing pivots can shift on intra-bar price changes. The `leg()` function (L246-255) uses `high[size]` and `ta.highest(size)` which are evaluated on the current (possibly unconfirmed) bar.
- `displayStructure()` (L383-427) uses `ta.crossover(close, p_ivot.currentLevel)` and `ta.crossunder(close, p_ivot.currentLevel)` -- these fire on the current tick's close, not the confirmed bar's close.
- The only bar-state check is at L559: `barstate.islastconfirmedhistory or barstate.islast` which gates the ORDER BLOCK DRAWING (visual rendering only), not the detection logic.
- Similarly, L569: `barstate.islastconfirmedhistory or (barstate.isrealtime and newBar)` gates MTF level drawing.

**FIX:** Wrap ALL signal detection logic in:
```pinescript
if barstate.isconfirmed
    getCurrentStructure(swingsLengthInput, false)
    getCurrentStructure(5, false, true)
    if showEqualHighsLowsInput
        getCurrentStructure(equalHighsLowsLengthInput, true)
    if showInternalsInput or showInternalOrderBlocksInput or showTrendInput
        displayStructure(true)
    if showStructureInput or showSwingOrderBlocksInput or showHighLowSwingsInput
        displayStructure()
    // ... all other detection
```

### 4.3 Swing Detection Retroactive Changes

**RISK: MEDIUM.** The `leg()` function (L246-255):
```pinescript
leg(int size) =>
    var leg = 0
    newLegHigh = high[size] > ta.highest(size)
    newLegLow  = low[size]  < ta.lowest(size)
```

- This compares `high[size]` against `ta.highest(size)` which looks at bars `[0]` through `[size-1]`. On the current unconfirmed bar, `high[0]` (part of `ta.highest(size)`) can change intra-bar, potentially making `newLegHigh` flip from true to false or vice versa.
- However, since `size` is an offset, the actual pivot being detected is `size` bars ago, so the pivot POINT itself does not move. The DETECTION timing can shift by 1 bar depending on whether the current bar's high exceeds the pivot threshold.
- **FIX:** Combined with the `barstate.isconfirmed` guard above, this risk is mitigated since detection only runs on confirmed bars.

### 4.4 Order Block Mitigation Timing

**RISK: LOW-MEDIUM.** `deleteOrderBlocks()` (L333-350):
```pinescript
if bearishOrderBlockMitigationSource > eachOrderBlock.barHigh and eachOrderBlock.bias == BEARISH
    crossedOderBlock := true
```

- `bearishOrderBlockMitigationSource` is either `close` or `high` (L229). If set to `high`, mitigation could be detected on an intra-bar wick that later retraces. If set to `close`, this is safe only when gated by `barstate.isconfirmed`.
- **FIX:** Gate with `barstate.isconfirmed`.

### 4.5 Alert Repainting

**RISK: MEDIUM.** All `alertcondition()` calls (L580-599) use the `currentAlerts` flags which are set during ungated detection logic. If a BOS crossover triggers mid-bar and then the bar closes without confirming it, the alert will have already fired.

- **FIX:** Since `alertcondition()` evaluates once per bar close by TradingView's design, this is partially mitigated by the platform. However, for `alert()` calls (which fire in realtime), the `barstate.isconfirmed` guard is essential.

### 4.6 Summary of Repainting Fixes Required

| Component | Risk Level | Fix Required | Effort |
|-----------|:----------:|:------------|:------:|
| `drawFairValueGaps()` request.security | Medium-High | Shift `high[0]`/`low[0]` to `[1]` offsets | Low |
| `drawLevels()` request.security | Low | No fix needed | N/A |
| Missing `barstate.isconfirmed` guard | High | Wrap all detection logic | Medium |
| `leg()` swing detection | Medium | Covered by isconfirmed guard | Low |
| OB mitigation timing | Low-Medium | Covered by isconfirmed guard | Low |
| Alert timing | Medium | Use isConfirmed for alert() calls | Low |

---

## 5. Improvement Recommendations

### 5.1 OB Mitigation Visual Distinction

**Current LuxAlgo behavior:** `deleteOrderBlocks()` (L333-350) completely REMOVES mitigated OBs from the array and their boxes become invisible (set to `na`). There is NO visual distinction between mitigated and unmitigated OBs.

**Recommended improvement:**

```
Instead of removing from array, change the visual state:

Unmitigated OB:
  - bgcolor: color at 20% opacity (e.g., #3179f5 at 80% transparency for bullish internal)
  - border: solid, 1px, same color family
  - extends right indefinitely (extend.right)

Mitigated OB:
  - bgcolor: same color at 5% opacity (barely visible)
  - border: dotted style (line.style_dotted)
  - stops extending at the mitigation bar
  - Optional: add small "X" label or strikethrough effect

Implementation:
  In deleteOrderBlocks(), instead of orderBlocks.remove(index):
    eachOrderBlock.state := 1  // mitigated
    // Update the corresponding box
    box b = getBoxForOB(eachOrderBlock)
    b.set_bgcolor(color.new(originalColor, 95))    // near transparent
    b.set_border_style(line.style_dotted)
    b.set_right(bar_index)  // stop extending

Add state field to orderBlock type:
  type orderBlock
      float barHigh
      float barLow
      int   barTime
      int   bias
      int   state = 0    // 0=unmitigated, 1=mitigated (NEW)

Toggle input:
  hideMitigatedOBsInput = input(false, "Hide Mitigated OBs", group=BLOCKS_GROUP)
```

### 5.2 FVG Partial Fill Tracking

**Current LuxAlgo behavior:** `deleteFairValueGaps()` (L431-436) uses a binary check -- if price fully penetrates the FVG, it deletes both boxes entirely. No partial fill tracking.

**Recommended improvement:**

```
Track fill progress and dynamically resize the FVG box:

Bullish FVG (gap between candle 1 high and candle 3 low):
  Original: top = candle3_low, bottom = candle1_high
  As price fills from above (for bullish FVG, price drops into it):
    If high enters the gap but close stays above bottom:
      -> Partial fill: reduce topBox to show only unfilled portion
      newTop = math.min(original_top, low)  // price has filled down to here
      If newTop <= bottom: FVG fully filled

Bearish FVG:
  Original: top = candle1_low, bottom = candle3_high
  As price fills from below:
    newBottom = math.max(original_bottom, high)
    If newBottom >= top: FVG fully filled

Implementation in deleteFairValueGaps():
  for [index, eachFVG] in fairValueGaps
      if eachFVG.bias == BULLISH
          if low <= eachFVG.bottom      // fully filled
              eachFVG.topBox.delete()
              eachFVG.bottomBox.delete()
              fairValueGaps.remove(index)
          else if low < eachFVG.top     // partially filled
              eachFVG.top := low        // shrink the gap
              eachFVG.topBox.set_top_left_point(chart.point.new(na, na, low))
              eachFVG.bottomBox.set_top_left_point(chart.point.new(na, na, math.avg(low, eachFVG.bottom)))
      else  // BEARISH
          if high >= eachFVG.top        // fully filled
              eachFVG.topBox.delete()
              eachFVG.bottomBox.delete()
              fairValueGaps.remove(index)
          else if high > eachFVG.bottom // partially filled
              eachFVG.bottom := high
              eachFVG.bottomBox.set_bottom_right_point(chart.point.new(na, na, high))
              eachFVG.topBox.set_bottom_right_point(chart.point.new(na, na, math.avg(eachFVG.top, high)))
```

### 5.3 Premium/Discount Fibonacci Sub-Zones

**Current LuxAlgo behavior:** `drawPremiumDiscountZones()` (L515-519) draws 3 simple zones at fixed 5% bands:
- Premium: top to 0.95*top + 0.05*bottom (only the top 5%)
- Equilibrium: narrow band around 50%
- Discount: 0.95*bottom + 0.05*top to bottom (only the bottom 5%)

This is overly simplistic -- most of the chart has no zone shading.

**Recommended improvement:**

```
Replace 5% bands with full premium/discount plus Fibonacci OTE levels:

Zones:
  Premium:    range_top to equilibrium (full upper half, red tint at 8% opacity)
  Discount:   equilibrium to range_bottom (full lower half, green tint at 8% opacity)
  Equilibrium line: dashed line at 50%

Fibonacci sub-levels (within premium/discount):
  0.786 level: range_bottom + 0.786 * range  (deep premium / shallow discount)
  0.705 level: range_bottom + 0.705 * range  (optimal trade entry -- OTE zone)
  0.618 level: range_bottom + 0.618 * range  (golden ratio)
  0.500 level: equilibrium
  0.382 level: range_bottom + 0.382 * range
  0.236 level: range_bottom + 0.236 * range

OTE Zone highlight: Subtle highlight between 0.618 and 0.786 -- this is the
"Optimal Trade Entry" zone in ICT methodology.

Visual:
  - Fibonacci lines: line.style_dotted, width 1, gray (#787B86)
  - Small label at each fib level: "0.618", "0.705", "0.786"
  - OTE zone: slightly more opaque background between 0.618-0.786
  - Equilibrium: line.style_dashed, width 1, #2962FF

Implementation:
  drawPremiumDiscountZones() =>
      range = trailing.top - trailing.bottom
      eq = trailing.top - range * 0.5

      // Full zones
      drawZone(trailing.top, ..., trailing.top, eq, "Premium", premiumZoneColor, ...)
      drawZone(trailing.bottom, ..., eq, trailing.bottom, "Discount", discountZoneColor, ...)

      // EQ line
      drawFibLevel(eq, "EQ", #2962FF, line.style_dashed)

      // Fib levels (optional toggle)
      if showFibLevelsInput
          drawFibLevel(trailing.bottom + range * 0.786, "0.786", #787B86, line.style_dotted)
          drawFibLevel(trailing.bottom + range * 0.705, "0.705", #787B86, line.style_dotted)
          drawFibLevel(trailing.bottom + range * 0.618, "0.618", #787B86, line.style_dotted)
          drawFibLevel(trailing.bottom + range * 0.382, "0.382", #787B86, line.style_dotted)
          drawFibLevel(trailing.bottom + range * 0.236, "0.236", #787B86, line.style_dotted)

Toggle:
  showFibLevelsInput = input(false, "Show Fibonacci Levels", group=ZONES_GROUP)
```

### 5.4 Color Palette Mapping

**LuxAlgo palette -> TradingClaude palette:**

| Element | LuxAlgo Color | TradingClaude Color | Change? |
|---------|:------------:|:------------------:|:-------:|
| Bullish signals (BOS/CHoCH) | `#089981` (L12) | `#089981` | SAME |
| Bearish signals (BOS/CHoCH) | `#F23645` (L13) | `#F23645` | SAME |
| Internal bullish OB | `#3179f5` at 80% (L101) | `#089981` at 80% | CHANGE |
| Internal bearish OB | `#f77c80` at 80% (L102) | `#F23645` at 80% | CHANGE |
| Swing bullish OB | `#1848cc` at 80% (L103) | `#089981` at 80% | CHANGE |
| Swing bearish OB | `#b22833` at 80% (L104) | `#F23645` at 80% | CHANGE |
| Bullish FVG | `#00ff68` at 70% (L114) | `#2962FF` at 85% | CHANGE |
| Bearish FVG | `#ff0008` at 70% (L115) | `#FF6D00` at 85% | CHANGE |
| MTF levels | `#2157f3` (L14) | `#2157f3` | SAME |
| Equilibrium | `#878b94` (L15) | `#2962FF` | CHANGE |
| Premium zone | `#F23645` (L129) | `#EF5350` at 92% | CHANGE |
| Discount zone | `#089981` (L131) | `#26A69A` at 92% | CHANGE |
| Monochrome bull | `#b2b5be` (L16) | N/A -- remove monochrome | REMOVE |
| Monochrome bear | `#5d606b` (L17) | N/A -- remove monochrome | REMOVE |
| NEW: BSL levels | N/A | `#FF5252` | NEW |
| NEW: SSL levels | N/A | `#00BCD4` | NEW |
| NEW: Dashboard bg | N/A | `#363A45` | NEW |
| NEW: Dashboard text | N/A | `#D1D4DC` | NEW |
| NEW: Swing points | N/A | `#787B86` | NEW |

**Key changes:** OBs and FVGs get distinct colors to differentiate them visually. FVGs use blue/orange instead of green/red to avoid confusion with OBs. Premium/discount use softer colors at higher transparency.

### 5.5 Input Reorganization

**Current LuxAlgo groups (8 groups):**
1. `Smart Money Concepts` (mode, style, candles) -- L43
2. `Real Time Internal Structure` -- L44
3. `Real Time Swing Structure` -- L45
4. `Order Blocks` -- L46
5. `EQH/EQL` -- L47
6. `Fair Value Gaps` -- L48
7. `Highs & Lows MTF` -- L49
8. `Premium & Discount Zones` -- L50

**Proposed TradingClaude groups (8 groups per design spec):**

```
G1: MARKET STRUCTURE
  - Show Internal Structure: [true]
  - Internal Swing Length: [5]
  - Show Swing Structure: [true]
  - Swing Length: [50]
  - Show Swing Points (HH/HL/LH/LL): [false]
  - Show Strong/Weak High/Low: [true]
  - Confluence Filter: [false]
  - BOS Line Style: [Dashed]
  - CHoCH Line Style: [Solid]
  - Bullish Color: [#089981]
  - Bearish Color: [#F23645]
  - Label Size: [Tiny]

G2: ORDER BLOCKS
  - Show Internal OBs: [true]
  - Max Internal OBs: [5]
  - Show Swing OBs: [false]
  - Max Swing OBs: [5]
  - OB Filter Method: [ATR / Cumulative Mean Range]
  - OB Mitigation Source: [Close / High-Low]
  - Hide Mitigated OBs: [false]
  - Bullish OB Color: [#089981]
  - Bearish OB Color: [#F23645]
  - OB Opacity: [20]

G3: FAIR VALUE GAPS
  - Show FVGs: [false]
  - FVG Timeframe: [Current]
  - Auto Threshold: [true]
  - FVG Extend Bars: [1]
  - Bullish FVG Color: [#2962FF]
  - Bearish FVG Color: [#FF6D00]
  - FVG Opacity: [15]

G4: LIQUIDITY
  - Show Equal Highs/Lows: [true]
  - EQH/EQL Bars Confirmation: [3]
  - EQH/EQL Threshold: [0.1]
  - Show Liquidity Sweeps: [true]
  - BSL Color: [#FF5252]
  - SSL Color: [#00BCD4]
  - Label Size: [Tiny]
  - Max Liquidity Levels: [10]

G5: PREMIUM / DISCOUNT
  - Show Premium/Discount Zones: [false]
  - Show Fibonacci Levels: [false]
  - Premium Color: [#EF5350]
  - Discount Color: [#26A69A]
  - Equilibrium Color: [#2962FF]
  - Zone Opacity: [8]

G6: MULTI-TIMEFRAME
  - Show MTF Analysis: [false]
  - HTF Timeframe: [Auto / 15 / 60 / 240 / D / W / M]
  - Show HTF Structure: [true]
  - Show HTF Order Blocks: [true]
  - Show HTF FVGs: [true]
  - Show Daily Levels: [false]
  - Show Weekly Levels: [false]
  - Show Monthly Levels: [false]
  - Daily/Weekly/Monthly Line Style: [Solid/Dashed/Dotted]
  - MTF Level Color: [#2157f3]

G7: DASHBOARD
  - Show Dashboard: [true]
  - Position: [Top Right / Top Left / Bottom Right / Bottom Left]
  - Size: [Small / Normal / Large]
  - Show Trend Row: [true]
  - Show Structure Row: [true]
  - Show Zones Row: [true]
  - Show Liquidity Row: [true]
  - Show Position Row: [true]
  - Show Signal Row: [true]

G8: ALERTS
  - Enable BOS Alerts: [true]
  - Enable CHoCH Alerts: [true]
  - Enable OB Alerts: [true]
  - Enable FVG Alerts: [true]
  - Enable Liquidity Alerts: [true]
  - Enable Confluence Alerts: [true]
  - Enable Premium/Discount Alerts: [false]
  - Alert Message Format: [Simple / Detailed]
```

**Changes from LuxAlgo:**
- Merged `Internal Structure` + `Swing Structure` into `G1: Market Structure`
- Moved `EQH/EQL` into `G4: Liquidity` (conceptually they are liquidity pools)
- Added new groups: `G6: MTF` (consolidated), `G7: Dashboard`, `G8: Alerts`
- Removed: `Mode` (Historical/Present) -- always show present + recent historical
- Removed: `Style` (Colored/Monochrome) -- always colored, simplify
- Added: Opacity controls, Hide Mitigated toggle, Fibonacci toggle, Sweep toggle

---

## 6. Implementation Priority

Based on the gap analysis, recommended build order:

| Priority | Feature | Effort | Dependencies |
|:--------:|---------|:------:|:------------|
| 1 | Anti-repainting fixes (`barstate.isconfirmed` wrap) | Small | None |
| 2 | Input reorganization (8 groups) | Small | None |
| 3 | Color palette update | Small | None |
| 4 | OB mitigated vs unmitigated visual | Medium | Anti-repainting fixes |
| 5 | FVG partial fill tracking | Medium | Anti-repainting fixes |
| 6 | Premium/Discount Fibonacci sub-zones | Medium | None |
| 7 | Liquidity Sweep Detection | Large | EQH/EQL working |
| 8 | Enhanced MTF (auto HTF + HTF structure overlay) | Large | Structure engine working |
| 9 | Dashboard | Large | All data modules working |
| 10 | Confluence Alerts | Medium | Sweep + MTF + Dashboard data ready |

**Critical path: 1 -> 4,5 -> 7 -> 8 -> 9 -> 10**

Items 2, 3, 6 can be done in parallel at any time.
