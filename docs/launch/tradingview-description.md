# TradingClaude SMC Suite (TC-SMC)

## Overview

TradingClaude SMC Suite is a comprehensive, all-in-one Smart Money Concepts indicator for TradingView. It combines market structure analysis, order blocks, fair value gaps, liquidity detection, premium/discount zones, multi-timeframe analysis, and an integrated dashboard -- all in a single, free indicator. Built for intermediate to advanced traders who use SMC/ICT methodology across any asset class and timeframe.

## Features

- **Market Structure (BOS/CHoCH)** -- Detects Break of Structure and Change of Character at both internal (short-term) and swing (long-term) levels. Identifies trend direction with HH/HL/LH/LL labeling and Strong/Weak High/Low markers.

- **Order Blocks (OB)** -- Automatically identifies institutional order blocks validated by confirmed structure breaks. Tracks unmitigated vs mitigated status with distinct visual styles. Supports both internal and swing-level OBs with configurable display limits.

- **Fair Value Gaps (FVG)** -- Detects price imbalances with automatic threshold filtering to remove insignificant gaps. Features dynamic partial fill tracking -- FVG boxes shrink in real time as price fills the gap. Supports multi-timeframe FVG detection.

- **Liquidity (BSL/SSL + Sweeps)** -- Identifies equal highs (buy-side liquidity) and equal lows (sell-side liquidity) with BSL/SSL labels and dotted level lines. Detects confirmed liquidity sweeps when price wicks past a level and closes back -- one of the highest-probability SMC signals.

- **Premium/Discount Zones** -- Calculates the current swing range and displays premium, discount, and equilibrium zones. Optional Fibonacci sub-levels (0.236, 0.382, 0.618, 0.705, 0.786) for identifying optimal trade entry (OTE) areas.

- **Multi-Timeframe (MTF) Analysis** -- Auto-detects the appropriate higher timeframe based on your chart. Projects HTF structure (BOS/CHoCH), HTF order blocks, and HTF fair value gaps directly onto your current chart with thicker borders and [HTF] labels. Includes optional previous daily, weekly, and monthly high/low levels (PDH/PDL, PWH/PWL, PMH/PML).

- **Real-Time Dashboard** -- On-screen table showing current trend (LTF + HTF), last BOS/CHoCH levels, active zone counts, nearest BSL/SSL levels, premium/discount position with percentage, and active confluence signals. Fully configurable position, size, and rows.

## What Makes This Different

**1. Verified Anti-Repainting Architecture**
Every signal in TC-SMC is confirmed only at bar close using `barstate.isconfirmed`. All `request.security()` calls use the `[1]` offset with `lookahead_on` pattern -- the standard anti-repainting technique for multi-timeframe data. Signals never appear, disappear, or move after the bar closes.

**2. Complete Suite, Completely Free**
Seven integrated modules that work together -- not seven separate paid indicators. Confluence detection automatically identifies when multiple signals align (e.g., OB touch + discount zone + HTF alignment), giving you the highest-probability setups without manual cross-referencing.

**3. Intelligent Signal Filtering**
Order blocks are only created when validated by a confirmed BOS. FVGs use automatic threshold filtering based on cumulative bar delta. Liquidity levels use ATR-based tolerance to avoid noise. The result: fewer but more meaningful signals.

**4. Clean, Non-Intrusive Visuals**
Designed with the principle that the chart belongs to the trader, not the indicator. Semi-transparent zones, concise labels, consistent color coding (green = bullish, red = bearish, blue = bullish FVG, orange = bearish FVG), and every element is individually toggleable.

## How to Use

1. **Add to chart** -- Search for "TradingClaude SMC Suite" in the Indicators panel and add it to your chart.
2. **Default setup** -- The indicator loads with Market Structure, Internal Order Blocks, Equal Highs/Lows, Liquidity Sweeps, and Dashboard enabled. This is a good starting point.
3. **Enable additional modules** -- Open Settings and toggle on Fair Value Gaps, Premium/Discount Zones, or MTF Analysis as needed.
4. **Set your HTF** -- If using MTF Analysis, leave it on "Auto" or select a specific higher timeframe. Auto maps: 1-5m to 1H, 15m to 1H, 1H to 4H, 4H to 1D, 1D to 1W.
5. **Configure alerts** -- Go to the Alerts tab in TradingView, select TC-SMC, and choose which signals to receive notifications for.

## Modules Detail

### 1. Market Structure (Group 1)

Controls the detection of internal and swing-level BOS/CHoCH. Key inputs:
- **Show Internal Structure** (default: on) -- Short-term structure breaks using a fixed 5-bar internal swing length.
- **Show Swing Structure** (default: on) -- Longer-term structure using a configurable swing length (default: 50 bars).
- **Bullish/Bearish Structure** -- Filter to show All, only BOS, or only CHoCH for each direction.
- **Confluence Filter** -- Filters out non-significant internal breaks based on candle body/wick ratio.
- **Show Swing Points (HH/HL/LH/LL)** -- Labels each swing with its sequence type.
- **Show Strong/Weak High/Low** -- Highlights the most recent strong and weak extremes on the chart.
- **Color Candles by Trend** -- Colors candles based on the current internal trend direction.

### 2. Order Blocks (Group 2)

Controls OB detection and display. Key inputs:
- **Internal Order Blocks** (default: on, max 5) -- OBs from internal structure breaks.
- **Swing Order Blocks** (default: off, max 5) -- OBs from swing-level structure breaks.
- **OB Filter Method** -- ATR (default) or Cumulative Mean Range for filtering volatile bars.
- **OB Mitigation Source** -- Close or High/Low for determining when an OB is mitigated.
- **Hide Mitigated OBs** -- When off (default), mitigated OBs remain visible with reduced opacity and dotted borders.

### 3. Fair Value Gaps (Group 3)

Controls FVG detection and display. Key inputs:
- **Show Fair Value Gaps** (default: off) -- Toggle FVG detection.
- **Auto Threshold** (default: on) -- Filters insignificant FVGs based on cumulative bar delta.
- **FVG Timeframe** -- Blank for current TF, or select a specific timeframe.
- **Extend FVG (bars)** -- How many bars to extend FVG boxes to the right (default: 1).

### 4. Liquidity (Group 4)

Controls equal highs/lows and sweep detection. Key inputs:
- **Show Equal Highs/Lows** (default: on) -- Detects EQH (BSL) and EQL (SSL) levels.
- **EQH/EQL Bars Confirmation** (default: 3) -- Bars used to confirm equal levels.
- **EQH/EQL Threshold** (default: 0.1) -- Sensitivity (0-0.5). Lower = fewer but more precise levels.
- **Show Liquidity Sweeps** (default: on) -- Detects when price sweeps past BSL/SSL and closes back.
- **Max Liquidity Levels** (default: 10) -- Maximum simultaneous levels tracked.

### 5. Premium/Discount (Group 5)

Controls range-based zone display. Key inputs:
- **Show Premium/Discount Zones** (default: off) -- Displays premium (red), discount (green), and equilibrium zones.
- **Show Fibonacci Levels** (default: off) -- Adds 0.236, 0.382, 0.618, 0.705, 0.786 sub-levels within the range.

### 6. Multi-Timeframe (Group 6)

Controls HTF analysis and level overlays. Key inputs:
- **Show MTF Analysis** (default: off) -- Enables HTF structure overlay, HTF OBs, and HTF FVGs on your chart.
- **HTF Timeframe** -- Auto (recommended) or manual selection (15m, 1H, 4H, 1D, 1W, 1M).
- **Show HTF Structure / OBs / FVGs** -- Individual toggles for each HTF element.
- **Max HTF Zones** (default: 3) -- Limits HTF OBs and FVGs displayed.
- **Daily/Weekly/Monthly Levels** -- Optional previous period high/low with configurable line styles.

### 7. Dashboard (Group 7)

Controls the on-screen information table. Key inputs:
- **Show Dashboard** (default: on) -- Displays the real-time status table.
- **Position** -- Top Right, Top Left, Bottom Right, or Bottom Left.
- **Size** -- Small, Normal, or Large.
- **Row toggles** -- Individually show/hide Trend, Structure, Zones, Liquidity, Position, and Signal rows.

## Anti-Repainting Guarantee

TC-SMC uses a strict anti-repainting architecture:

1. **barstate.isconfirmed guard** -- All signal detection logic (structure breaks, OB creation, FVG detection, sweep detection, confluence) runs exclusively inside a `barstate.isconfirmed` block. Signals are only generated after a bar has fully closed.

2. **MTF anti-repainting pattern** -- All `request.security()` calls use `[1]` data offsets with `barmerge.lookahead_on`, the standard Pine Script technique to prevent HTF data from painting future values on historical bars.

3. **Close-based confirmation** -- BOS/CHoCH detection uses `close` (not `high` or `low`) to cross structure levels, preventing false signals from intra-bar wicks.

4. **Retroactive swing safety** -- Swing points are detected using a lookback offset, so the pivot is placed at its correct historical bar. The detection timing is fixed by the `isconfirmed` guard.

## Alerts

TC-SMC provides 21 individual alert conditions plus dynamic context-rich alert messages:

**Individual Alert Conditions (for TradingView alert setup):**
- Internal Bullish/Bearish BOS (4 alerts)
- Internal Bullish/Bearish CHoCH (4 alerts)
- Swing Bullish/Bearish BOS (4 alerts)
- Swing Bullish/Bearish CHoCH (4 alerts -- these are the highest-priority reversal signals)
- Bullish/Bearish Internal OB Mitigated
- Bullish/Bearish Swing OB Mitigated
- Equal Highs (BSL) / Equal Lows (SSL)
- Bullish/Bearish FVG
- Bullish Liquidity Sweep (SSL) / Bearish Liquidity Sweep (BSL)
- Premium Zone Entry / Discount Zone Entry
- HTF+LTF Confluence

**Dynamic Alerts (rich context messages):**
All alerts support two formats configurable in the Alerts group:
- **Simple**: Short message with signal type and price.
- **Detailed**: Includes HTF trend, timeframe, position (premium/discount), and alignment context.

**How to set up alerts:**
1. Right-click on the chart and select "Add Alert" (or press Alt+A).
2. Under "Condition", select "TradingClaude SMC Suite".
3. Choose the specific alert condition from the dropdown.
4. Set your notification preferences (popup, email, webhook, etc.).
5. For the richest context, also create an "Any alert() function call" alert on TC-SMC to receive the detailed dynamic messages.

## Recommended Settings

### Scalping (1m-5m)
- Internal Structure: ON (primary signals)
- Swing Structure: ON
- Internal Order Blocks: ON, max 3-5
- Fair Value Gaps: ON (current timeframe)
- Liquidity Sweeps: ON
- MTF Analysis: ON (Auto will select 1H as HTF)
- Dashboard: ON
- Premium/Discount: Optional

### Intraday (15m-1H)
- Internal Structure: ON
- Swing Structure: ON
- Internal Order Blocks: ON, max 5
- Swing Order Blocks: ON, max 3
- Fair Value Gaps: ON
- Liquidity Sweeps: ON
- MTF Analysis: ON (Auto: 1H or 4H)
- Premium/Discount: ON with Fibonacci levels
- Dashboard: ON

### Swing Trading (4H-1D)
- Internal Structure: Optional (can be noisy)
- Swing Structure: ON
- Swing Order Blocks: ON, max 5
- Fair Value Gaps: ON
- Liquidity Sweeps: ON
- MTF Analysis: ON (Auto: 1D or 1W)
- Premium/Discount: ON with Fibonacci levels
- Daily/Weekly/Monthly Levels: ON
- Dashboard: ON

## FAQ

**Q: Is this indicator free?**
A: Yes. The complete suite with all seven modules is free to use.

**Q: Does this indicator repaint?**
A: No. All signals are confirmed at bar close using `barstate.isconfirmed`. MTF data uses the standard `[1]` offset anti-repainting pattern. Once a signal appears on a closed bar, it will never disappear or move.

**Q: Which assets and timeframes does this work on?**
A: TC-SMC is designed for any liquid asset -- crypto, forex, stocks, indices, commodities. It works on all timeframes from 1 minute to monthly. For best results, use it on assets with sufficient liquidity and volume.

**Q: The chart looks cluttered with all modules on. What should I do?**
A: Start with the defaults (Market Structure + Internal OBs + Liquidity + Dashboard). Add modules one at a time as you learn the indicator. Each module is independently toggleable. You can also reduce max zones (OBs, FVGs, liquidity levels) to show fewer elements.

**Q: Why are some modules off by default?**
A: To keep the initial experience clean and performant. Fair Value Gaps, Premium/Discount Zones, MTF Analysis, and Swing Order Blocks are off by default so you can enable them as needed for your trading style.

**Q: My chart is loading slowly. How can I improve performance?**
A: Reduce the number of max zones (Internal OBs, FVGs, Liquidity Levels). Disable modules you are not actively using. Avoid running all modules simultaneously on very low timeframes (1m) with extended history.

**Q: What does the "Confluence" signal in the dashboard mean?**
A: Confluence fires when two or more signals align on the same bar. For example: an OB touch + price in discount zone + HTF trend aligned = a high-probability setup. The dashboard shows which signals are confluent. This is the most powerful feature of the suite.

## Disclaimer

This indicator is provided for educational and informational purposes only. It is not financial advice. Trading involves significant risk of loss. Past performance of any signal or pattern does not guarantee future results. Always conduct your own analysis and use proper risk management. The authors are not responsible for any trading losses incurred using this tool.
