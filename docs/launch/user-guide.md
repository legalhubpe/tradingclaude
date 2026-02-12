# TradingClaude SMC Suite -- User Guide

> Version 1.0 | Last updated: 2026-02-12

---

## Table of Contents

1. [Getting Started](#1-getting-started)
2. [Module 1: Market Structure](#2-module-1-market-structure)
3. [Module 2: Order Blocks](#3-module-2-order-blocks)
4. [Module 3: Fair Value Gaps](#4-module-3-fair-value-gaps)
5. [Module 4: Liquidity](#5-module-4-liquidity)
6. [Module 5: Premium/Discount Zones](#6-module-5-premiumdiscount-zones)
7. [Module 6: Multi-Timeframe Analysis](#7-module-6-multi-timeframe-analysis)
8. [Module 7: Dashboard](#8-module-7-dashboard)
9. [Alert Setup Walkthrough](#9-alert-setup-walkthrough)
10. [Recommended Settings by Trading Style](#10-recommended-settings-by-trading-style)
11. [Tips and Best Practices](#11-tips-and-best-practices)
12. [Troubleshooting](#12-troubleshooting)

---

## 1. Getting Started

### Step 1: Add the Indicator

1. Open any chart on TradingView.
2. Click the "Indicators" button at the top of the chart (or press `/`).
3. Search for **TradingClaude SMC Suite**.
4. Click it to add it to your chart.
5. The indicator loads as an overlay directly on your price chart.

### Step 2: Understand the Defaults

When first loaded, TC-SMC enables these modules by default:

| Module | Default State | Why |
|--------|:---:|-----|
| Internal Structure (BOS/CHoCH) | ON | Core SMC analysis |
| Swing Structure (BOS/CHoCH) | ON | Longer-term trend context |
| Internal Order Blocks | ON (max 5) | Key institutional zones |
| Equal Highs/Lows (BSL/SSL) | ON | Liquidity level identification |
| Liquidity Sweeps | ON | High-probability reversal signals |
| Dashboard | ON | At-a-glance market state |
| Fair Value Gaps | OFF | Enable when needed |
| Premium/Discount Zones | OFF | Enable when needed |
| MTF Analysis | OFF | Enable when needed |
| Swing Order Blocks | OFF | Enable when needed |

### Step 3: Open Settings

1. Hover over the indicator name in the top-left of the chart.
2. Click the gear icon to open Settings.
3. You will see 8 input groups, one per module.

### Step 4: Choose Your Timeframe

TC-SMC works on all timeframes. Pick the one that matches your trading style:

- **Scalping**: 1m, 3m, 5m
- **Intraday**: 15m, 30m, 1H
- **Swing**: 4H, 1D
- **Position**: 1D, 1W

---

## 2. Module 1: Market Structure

Market structure is the foundation of Smart Money Concepts. This module detects swing highs and lows, then identifies when price breaks those levels.

### What It Detects

**Break of Structure (BOS)**: Price closes beyond a swing level in the same direction as the existing trend. This confirms trend continuation.
- Bullish BOS: Price closes above the last swing high during an uptrend.
- Bearish BOS: Price closes below the last swing low during a downtrend.

**Change of Character (CHoCH)**: The first BOS in the opposite direction of the current trend. This signals a potential trend reversal -- the most important structure signal.
- Bullish CHoCH: Price closes above swing high during a downtrend (potential reversal to bullish).
- Bearish CHoCH: Price closes below swing low during an uptrend (potential reversal to bearish).

### Internal vs Swing Structure

TC-SMC tracks structure at two levels:

- **Internal Structure**: Uses a fixed 5-bar swing length. Captures short-term structure breaks. Displayed with dashed lines.
- **Swing Structure**: Uses a configurable swing length (default: 50 bars). Captures major structure breaks. Displayed with solid lines.

Both run simultaneously, giving you both the micro and macro view of market structure.

### Visual Elements

- **BOS lines**: Horizontal dashed lines (internal) or solid lines (swing) from the broken level to the current bar.
- **CHoCH lines**: Same style but labeled "CHoCH" instead of "BOS".
- **Colors**: Green for bullish breaks, red for bearish breaks (customizable).
- **Swing labels (optional)**: "HH" (Higher High), "HL" (Higher Low), "LH" (Lower High), "LL" (Lower Low) at each swing point.
- **Strong/Weak High/Low (optional)**: Labels the most recent extremes. A "Strong High" is one that has NOT been broken (the trend respects it). A "Weak High" is one that is likely to be taken out.

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Show Internal Structure | true | Toggle internal BOS/CHoCH |
| Bullish/Bearish Structure | All | Show All, only BOS, or only CHoCH |
| Confluence Filter | false | Filter out weak internal breaks based on candle body/wick ratio |
| Show Swing Structure | true | Toggle swing-level BOS/CHoCH |
| Swing Length | 50 | Number of bars for swing point detection (higher = fewer, more significant swings) |
| Show Swing Points | false | Display HH/HL/LH/LL labels |
| Show Strong/Weak High/Low | true | Highlight trailing extremes |
| Color Candles by Trend | false | Override candle colors based on internal trend |

### Use Cases

- **Trend identification**: If you see a series of BOS bullish events, the trend is up. A CHoCH bearish signals a potential reversal.
- **Entry timing**: After a CHoCH, wait for a pullback into a discount zone or order block before entering.
- **Stop placement**: Place stops beyond the last swing point that defines the current structure.

---

## 3. Module 2: Order Blocks

Order blocks represent the last institutional candle before a significant price move. They act as supply/demand zones where smart money is likely to have unfilled orders.

### What It Detects

**Bullish Order Block**: The last bearish (down) candle before a bullish BOS. When price returns to this zone, institutional buy orders may still be resting there.

**Bearish Order Block**: The last bullish (up) candle before a bearish BOS. When price returns to this zone, institutional sell orders may still be resting there.

TC-SMC only creates OBs that are validated by a confirmed structure break. This filtering removes low-quality OBs that do not have institutional backing.

### Visual Elements

- **Unmitigated OB**: Semi-transparent colored box extending to the right. Green for bullish, red for bearish. Swing OBs have visible borders; internal OBs have no border to reduce clutter.
- **Mitigated OB**: When price returns to the zone, the OB status changes to "mitigated". The box becomes nearly transparent with a dotted border and stops extending to the right.

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Internal Order Blocks | true | Show OBs from internal structure |
| Internal OB count | 5 | Maximum internal OBs displayed |
| Swing Order Blocks | false | Show OBs from swing structure |
| Swing OB count | 5 | Maximum swing OBs displayed |
| OB Filter Method | ATR | ATR or Cumulative Mean Range for filtering volatile bars |
| OB Mitigation Source | High/Low | Close or High/Low to detect when OB is touched |
| Hide Mitigated OBs | false | Remove mitigated OBs entirely vs keeping them with reduced visuals |

### Use Cases

- **Entry zones**: When price pulls back to an unmitigated bullish OB in an uptrend, this is a potential buy zone.
- **Confluence**: An unmitigated OB that overlaps with a FVG or HTF zone is a high-probability setup.
- **Invalidation**: If price closes through the entire OB zone, the OB is mitigated and no longer expected to hold.

---

## 4. Module 3: Fair Value Gaps

Fair Value Gaps (FVGs) represent price inefficiencies -- gaps between candle wicks where the market moved too fast, leaving unfilled orders. Price tends to return to fill these gaps.

### What It Detects

**Bullish FVG**: A gap where candle 3's low is above candle 1's high. The impulse candle (candle 2) moved up so fast that it created a gap between candles 1 and 3.

**Bearish FVG**: A gap where candle 3's high is below candle 1's low. The impulse candle moved down creating a gap.

### Dynamic Partial Fill Tracking

One of TC-SMC's unique features is dynamic FVG tracking. Instead of simply deleting FVGs when touched, the boxes shrink in real time as price partially fills the gap:

- As price fills into a bullish FVG from above, the box narrows to show only the remaining unfilled portion.
- When the price fills the gap completely (closes through the other side), the FVG is removed.

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Show Fair Value Gaps | false | Toggle FVG detection |
| Auto Threshold | true | Filters out insignificant FVGs based on cumulative bar delta |
| FVG Timeframe | (blank) | Current timeframe by default; set a specific TF for HTF FVGs |
| Extend FVG (bars) | 1 | How far FVG boxes extend to the right |

### Visual Elements

- Bullish FVGs: Blue boxes (two-tone split at midpoint).
- Bearish FVGs: Orange boxes (two-tone split at midpoint).
- Boxes shrink dynamically as the gap fills.

### Use Cases

- **Retracement targets**: After an impulse, expect price to return and fill the FVG before continuing.
- **Entry refinement**: Use a FVG within an OB zone as a precise entry level.
- **Confluence**: A FVG overlapping with an OB in the discount zone aligned with HTF trend is a high-probability setup.

---

## 5. Module 4: Liquidity

Liquidity is the fuel that drives smart money moves. This module identifies where liquidity pools sit and detects when they are swept.

### What It Detects

**Equal Highs (EQH / BSL)**: Two or more swing highs at nearly the same price level (within the threshold). These represent buy-side liquidity (BSL) -- stop losses of short sellers and breakout buy orders sit above these levels.

**Equal Lows (EQL / SSL)**: Two or more swing lows at nearly the same price level. These represent sell-side liquidity (SSL) -- stop losses of long traders and breakout sell orders sit below these levels.

**Liquidity Sweeps**: When price wicks past a BSL or SSL level but closes back on the other side. This "sweep" grabs the resting orders and often precedes a reversal.
- Bullish sweep (SSL sweep): Price wicks below SSL, closes back above. Bearish liquidity was grabbed; bullish reversal expected.
- Bearish sweep (BSL sweep): Price wicks above BSL, closes back below. Bullish liquidity was grabbed; bearish reversal expected.

### Visual Elements

- **BSL levels**: Red dotted horizontal lines with "BSL" label, extending to the right until swept.
- **SSL levels**: Cyan dotted horizontal lines with "SSL" label, extending to the right until swept.
- **Sweep markers**: "SWEEP" label on the candle that performed the sweep. Red for bearish sweeps (above BSL), cyan for bullish sweeps (below SSL).
- Swept levels stop extending and the line terminates at the sweep bar.

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Show Equal Highs/Lows | true | Toggle BSL/SSL detection |
| EQH/EQL Bars Confirmation | 3 | Bars used to confirm equal levels |
| EQH/EQL Threshold | 0.1 | Sensitivity (0-0.5). Lower = fewer, more precise levels |
| Show Liquidity Sweeps | true | Toggle sweep detection |
| Max Liquidity Levels | 10 | Maximum simultaneous tracked levels |

### Use Cases

- **Anticipate targets**: When trend is bullish, the nearest BSL above is a likely target. When bearish, the nearest SSL below.
- **Reversal entries**: A confirmed sweep + OB zone + discount position is one of the highest-probability SMC setups.
- **Stop placement**: Avoid placing stops right at obvious BSL/SSL levels; these are targeted by smart money.

---

## 6. Module 5: Premium/Discount Zones

This module divides the current swing range into premium (expensive) and discount (cheap) zones, helping you determine whether the current price is favorable for buying or selling.

### What It Detects

The module uses the trailing swing high and swing low to define the current range:
- **Equilibrium (EQ)**: The 50% level of the range. This is the "fair price".
- **Premium Zone**: Above equilibrium (top half). In an uptrend, selling in premium is higher probability. In a downtrend, this is where bears enter.
- **Discount Zone**: Below equilibrium (bottom half). In an uptrend, buying in discount is higher probability.

### Fibonacci Sub-Levels

When enabled, the module adds key Fibonacci retracement levels within the range:
- **0.236**: Shallow retracement
- **0.382**: Moderate retracement
- **0.618**: Golden ratio level
- **0.705**: Optimal Trade Entry (OTE) lower bound
- **0.786**: OTE upper bound / deep retracement

The 0.618-0.786 zone is the classic ICT "Optimal Trade Entry" area.

### Visual Elements

- Premium zone: Soft red background tint (8% opacity by default).
- Discount zone: Soft green background tint (8% opacity by default).
- Equilibrium: Labeled "EQ" box at the 50% level.
- Fibonacci lines: Gray dotted lines with labels (0.236, 0.382, 0.618, 0.705, 0.786).

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Show Premium/Discount Zones | false | Toggle zone display |
| Show Fibonacci Levels | false | Add Fibonacci sub-levels |
| Premium Color | Red | Customizable zone color |
| Discount Color | Green | Customizable zone color |
| Equilibrium Color | Blue | Customizable EQ line color |

### Use Cases

- **Trade direction filter**: Only take long entries in the discount zone and short entries in the premium zone.
- **Target setting**: Enter in discount, target equilibrium or premium.
- **OTE entries**: With Fibonacci enabled, look for entries between 0.618 and 0.786 for optimal risk/reward.

---

## 7. Module 6: Multi-Timeframe Analysis

MTF analysis lets you see the bigger picture without switching charts. This module projects higher timeframe structure, order blocks, and fair value gaps directly onto your current chart.

### Auto HTF Detection

When set to "Auto", TC-SMC selects the appropriate higher timeframe:

| Your Chart TF | Auto HTF |
|:-:|:-:|
| 1m - 5m | 1H |
| 15m | 1H |
| 1H | 4H |
| 4H | 1D |
| 1D | 1W |
| 1W | 1M |

You can also manually select 15m, 1H, 4H, 1D, 1W, or 1M.

### What It Shows

**HTF Structure**: BOS and CHoCH lines from the higher timeframe, drawn on your current chart with thicker lines (width 2) and prefixed labels like "[1H] BOS" or "[4H] CHoCH".

**HTF Order Blocks**: Institutional OBs from the HTF projected as zones with thicker borders (width 2) and more visible opacity than LTF OBs.

**HTF Fair Value Gaps**: FVGs from the HTF shown with dotted borders (width 2) to distinguish from LTF FVGs.

**Previous Period Levels (optional)**:
- PDH/PDL: Previous Daily High/Low
- PWH/PWL: Previous Weekly High/Low
- PMH/PML: Previous Monthly High/Low

These levels are significant support/resistance and liquidity targets.

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Show MTF Analysis | false | Master toggle for HTF overlay |
| HTF Timeframe | Auto | Auto-detect or manual selection |
| Show HTF Structure | true | Display HTF BOS/CHoCH |
| Show HTF Order Blocks | true | Project HTF OBs |
| Show HTF FVGs | true | Project HTF FVGs |
| Max HTF Zones | 3 | Limit for HTF OBs and FVGs |
| Daily/Weekly/Monthly Levels | false | Previous period highs/lows |

### Anti-Repainting in MTF

All HTF data uses the `[1]` offset with `barmerge.lookahead_on` pattern. This means HTF data is always from the previous completed HTF bar, preventing any future data from leaking into historical calculations.

### Use Cases

- **Trend alignment**: Trade in the direction of the HTF trend for higher probability. The dashboard shows both LTF and HTF trend at a glance.
- **HTF zone entries**: When LTF price reaches an HTF OB or FVG zone and shows a reversal signal (CHoCH, sweep), this is a high-confidence entry.
- **Confluence stacking**: The confluence engine automatically detects when LTF signals align with HTF zones or HTF trend direction.

---

## 8. Module 7: Dashboard

The dashboard provides a real-time overview of all modules in a compact on-screen table.

### Dashboard Rows

| Row | Information | Example |
|-----|-------------|---------|
| **Header** | Indicator name | TRADINGCLAUDE SMC SUITE |
| **Trend** | LTF + HTF trend direction | LTF: Bullish \| HTF: Bearish |
| **Structure** | Last BOS and CHoCH levels | BOS: 1.0845 (Bull) \| CHoCH: 1.0790 |
| **Zones** | Active OB and FVG counts | OB: 3 (1 HTF) \| FVG: 2 (1 HTF) |
| **Liquidity** | Nearest unswept BSL and SSL | BSL: 1.0900 \| SSL: 1.0750 |
| **Position** | Premium/Discount + EQ level | Discount (38.2%) \| EQ: 1.0825 |
| **Signal** | Active confluence or "No active signal" | CHoCH OB Discount HTF |

### Color Coding

- Trend row: Green text for bullish, red for bearish.
- Position row: Green for discount, red for premium.
- Signal row: Gold/yellow when a confluence signal is active.

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Show Dashboard | true | Master toggle |
| Position | Top Right | Choose any corner |
| Size | Small | Small, Normal, or Large text |
| Row toggles | All true | Hide individual rows to save space |

---

## 9. Alert Setup Walkthrough

### Step 1: Open the Alert Dialog

Right-click anywhere on the chart and select "Add Alert", or press **Alt+A** (Windows/Linux) or **Option+A** (Mac).

### Step 2: Select TC-SMC as the Condition

In the alert dialog:
1. Under "Condition", select **TradingClaude SMC Suite** from the indicator dropdown.
2. A second dropdown appears showing all available alert conditions.

### Step 3: Choose Your Alert Type

TC-SMC offers 21 individual `alertcondition()` alerts:

**Structure Alerts:**
- Internal Bullish BOS
- Internal Bearish BOS
- Internal Bullish CHoCH
- Internal Bearish CHoCH
- Swing Bullish BOS
- Swing Bearish BOS
- Swing Bullish CHoCH
- Swing Bearish CHoCH

**Order Block Alerts:**
- Bullish Internal OB Mitigated
- Bearish Internal OB Mitigated
- Bullish Swing OB Mitigated
- Bearish Swing OB Mitigated

**Liquidity Alerts:**
- Equal Highs (BSL)
- Equal Lows (SSL)
- Bullish Liquidity Sweep (SSL)
- Bearish Liquidity Sweep (BSL)

**FVG Alerts:**
- Bullish FVG
- Bearish FVG

**Zone Alerts:**
- Premium Zone Entry
- Discount Zone Entry

**Confluence:**
- HTF+LTF Confluence

### Step 4: Set Up the Dynamic Alert (Recommended)

For the richest alert messages with full context (HTF trend, position, multiple signal details):

1. In the Condition dropdown, select **TradingClaude SMC Suite**.
2. Select **Any alert() function call** as the alert type.
3. This captures all dynamic alerts from TC-SMC with detailed context.

### Step 5: Configure Notifications

Choose how you want to be notified:
- **Popup**: On-screen notification in TradingView.
- **Email**: Sent to your TradingView email.
- **Webhook**: Send to a URL (useful for bots, Discord, Telegram).
- **Mobile push**: Via the TradingView mobile app.

### Step 6: Configure Alert Format in TC-SMC Settings

In the indicator's settings under the "Alerts" group:
- **Simple format**: Short messages like "TC-SMC: CHoCH Bullish 1.0845"
- **Detailed format**: Full context like "TradingClaude: CHoCH Bullish at 1.0845 | TF: 15 | HTF: Aligned | Trend reversal"

You can also toggle individual alert categories on/off (BOS, CHoCH, OB, FVG, Sweeps, Confluence, Premium/Discount).

### Alert Tips

- Set CHoCH alerts to high priority -- these are the most important reversal signals.
- The Confluence alert fires when 2+ signals align. This is the highest-probability alert.
- Sweep alerts combined with OB zones often produce the best setups.
- Premium/Discount alerts are disabled by default since they fire frequently. Enable them only if you want zone entry notifications.

---

## 10. Recommended Settings by Trading Style

### Scalping (1m-5m charts)

**Goal**: Catch quick moves on internal structure breaks.

```
Market Structure:
  Internal Structure: ON
  Swing Structure: ON (for context)
  Confluence Filter: ON (reduces noise on low TFs)
  Color Candles by Trend: ON (quick visual)

Order Blocks:
  Internal OBs: ON, max 3
  Swing OBs: OFF

Fair Value Gaps:
  FVGs: ON (current TF)
  Auto Threshold: ON

Liquidity:
  Equal Highs/Lows: ON
  Sweeps: ON
  Max Levels: 5

Premium/Discount:
  Optional (can add clutter on low TFs)

MTF Analysis:
  ON (Auto = 1H for most scalping TFs)
  HTF Structure: ON
  HTF OBs: ON
  Max HTF Zones: 2

Dashboard: ON
```

### Intraday (15m-1H charts)

**Goal**: Balanced view of structure, zones, and HTF context.

```
Market Structure:
  Internal Structure: ON
  Swing Structure: ON
  Show Strong/Weak High/Low: ON

Order Blocks:
  Internal OBs: ON, max 5
  Swing OBs: ON, max 3

Fair Value Gaps:
  FVGs: ON
  Auto Threshold: ON

Liquidity:
  Equal Highs/Lows: ON
  Sweeps: ON
  Max Levels: 10

Premium/Discount:
  ON with Fibonacci Levels

MTF Analysis:
  ON (Auto = 1H or 4H)
  All HTF elements: ON
  Max HTF Zones: 3
  Daily Levels: ON

Dashboard: ON (all rows)
```

### Swing Trading (4H-1D charts)

**Goal**: Focus on high-quality swing-level signals with HTF alignment.

```
Market Structure:
  Internal Structure: OFF or CHoCH only (less noise)
  Swing Structure: ON
  Show Swing Points: ON
  Show Strong/Weak High/Low: ON

Order Blocks:
  Internal OBs: OFF
  Swing OBs: ON, max 5

Fair Value Gaps:
  FVGs: ON
  Auto Threshold: ON

Liquidity:
  Equal Highs/Lows: ON
  Sweeps: ON
  Threshold: 0.2 (more selective)

Premium/Discount:
  ON with Fibonacci Levels

MTF Analysis:
  ON (Auto = 1D or 1W)
  Weekly and Monthly Levels: ON

Dashboard: ON (all rows)
```

---

## 11. Tips and Best Practices

### Signal Hierarchy

Not all signals are equal. From highest to lowest probability:

1. **Confluence signals** (2+ signals on the same bar) -- especially when HTF-aligned
2. **CHoCH** (Change of Character) -- trend reversal signal
3. **Liquidity Sweep + OB zone** -- sweep followed by OB reaction
4. **OB touch in discount/premium** -- zone reaction with position context
5. **BOS** (Break of Structure) -- trend continuation confirmation
6. **Individual FVG fill** -- useful for entry refinement but not standalone signals

### Reading the Dashboard Effectively

The dashboard gives you a "snapshot" of the market. Before entering a trade, check:

1. **Trend row**: Are LTF and HTF aligned? Trading with both trends increases probability.
2. **Position row**: Are you buying in discount or selling in premium? This is essential for SMC.
3. **Signal row**: Is there a confluence? If yes, this bar has multiple signals aligning.
4. **Liquidity row**: Where is the nearest BSL/SSL? This tells you where price is likely to reach next.

### Combining Modules for High-Probability Setups

The best SMC setups combine multiple concepts. Example of a high-probability long setup:

1. HTF trend is bullish (dashboard shows "HTF: Bullish").
2. Price sweeps an SSL level (liquidity grab below).
3. Sweep occurs within a bullish OB zone.
4. Price is in the discount zone (Position shows < 50%).
5. A bullish CHoCH confirms the reversal.
6. Confluence alert fires.

### What NOT to Do

- Do not enable all modules on 1-minute charts with maximum zones -- it will be overwhelming and slow.
- Do not use signals in isolation. A single BOS or FVG is not a trade setup by itself.
- Do not ignore the HTF context. A bullish signal on the 15m chart during a bearish HTF trend has lower probability.
- Do not move your stop loss to the opposite side of an order block zone -- it is there for a reason.

---

## 12. Troubleshooting

### Problem: Indicator is loading slowly

**Solutions:**
- Reduce max zone counts (Internal OBs, Swing OBs, FVGs, Liquidity Levels).
- Disable modules you are not using (especially MTF Analysis and Premium/Discount on low TFs).
- Avoid using 1-minute charts with extended history and all modules active.
- The dashboard updates only on confirmed bars and the last bar to minimize performance impact.

### Problem: Too many signals / chart is cluttered

**Solutions:**
- Disable Internal Structure and only use Swing Structure for a cleaner view.
- Set Bullish/Bearish Structure to show only "CHoCH" instead of "All".
- Reduce max OBs and max liquidity levels.
- Turn off Fair Value Gaps if not needed.
- Enable the Confluence Filter to remove weak internal structure breaks.

### Problem: Not enough signals

**Solutions:**
- Lower the swing length for swing structure (e.g., from 50 to 30).
- Lower the EQH/EQL threshold (e.g., from 0.1 to 0.2) for more liquidity levels.
- Disable the Auto Threshold on FVGs to see all gaps.
- Ensure the relevant modules are actually toggled ON in settings.

### Problem: I don't see HTF data on my chart

**Solutions:**
- Make sure "Show MTF Analysis" is turned ON in the Multi-Timeframe group.
- Ensure your chart TF is lower than the selected HTF. You cannot overlay 1D HTF data on a 1D chart.
- Check that individual HTF toggles (HTF Structure, HTF OBs, HTF FVGs) are enabled.

### Problem: Alerts are not firing

**Solutions:**
- Verify the alert is set up on "TradingClaude SMC Suite" (not another indicator).
- For dynamic alerts, use "Any alert() function call" as the condition.
- Check that the relevant alert category is enabled in the indicator's Alerts settings group.
- Ensure the alert is not expired (TradingView alerts expire by default).

### Problem: Dashboard text is too small or too large

**Solutions:**
- Change the Dashboard Size setting (Small / Normal / Large).
- Move the dashboard to a different corner if it overlaps with price action.
- Hide unnecessary rows (e.g., if you do not use MTF, the Trend row's HTF value will show "Neutral").

### Problem: OBs disappear immediately

**Solutions:**
- Check if "Hide Mitigated OBs" is set to true. When true, OBs are completely removed once mitigated.
- Set it to false to keep mitigated OBs visible with reduced opacity and dotted borders.
- If OBs are cycling too fast, increase the max OB count.

---

## Glossary

| Term | Definition |
|------|-----------|
| **BOS** | Break of Structure -- price closes beyond a swing level in the trend direction |
| **CHoCH** | Change of Character -- first BOS in the opposite direction, signaling potential reversal |
| **OB** | Order Block -- last institutional candle before a significant move |
| **FVG** | Fair Value Gap -- price inefficiency / imbalance between candle wicks |
| **BSL** | Buy-Side Liquidity -- resting orders above equal highs |
| **SSL** | Sell-Side Liquidity -- resting orders below equal lows |
| **Sweep** | Price wicks past a liquidity level and closes back (a "grab") |
| **Mitigated** | An OB or FVG zone that price has returned to and touched |
| **Premium** | Above the 50% equilibrium of the current range (expensive zone) |
| **Discount** | Below the 50% equilibrium (cheap zone) |
| **EQ** | Equilibrium -- the 50% midpoint of the current swing range |
| **OTE** | Optimal Trade Entry -- the 0.618-0.786 Fibonacci zone |
| **HTF** | Higher Timeframe |
| **LTF** | Lower Timeframe (your current chart timeframe) |
| **Confluence** | Two or more signals aligning on the same bar |
| **PDH/PDL** | Previous Daily High/Low |
| **PWH/PWL** | Previous Weekly High/Low |
| **PMH/PML** | Previous Monthly High/Low |

---

*TradingClaude SMC Suite is provided for educational purposes only. It is not financial advice. Always use proper risk management.*
