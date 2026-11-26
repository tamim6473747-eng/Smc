# Smart Money AI — personal SMC chart scanner (Android)

Personal-use Android app. A floating button scans the chart on your screen, runs a rule-based
Smart Money Concepts analysis and posts a BUY / SELL / WAIT notification. It is analysis only —
it never trades and never claims a win rate. The score is *technical confluence*, not probability.

## Build the APK (pick one)

**A. Android Studio (easiest):** File ▸ Open this folder ▸ wait for Gradle sync ▸
Build ▸ Build APK(s). APK: `app/build/outputs/apk/debug/app-debug.apk`.

**B. Command line (needs JDK 17 + Android SDK, `ANDROID_HOME` set):**
`./gradlew assembleDebug`

**C. No PC tools — GitHub Actions:** push this folder to a (private) GitHub repo ▸ Actions ▸
"Build APK" ▸ Run workflow ▸ download the `SmartMoneyAI-debug-apk` artifact.

Install the APK on your phone (allow "install unknown apps"). Min Android 8.0, built for 14+.

## Using it
1. Open app ▸ **START SCANNER**. Approve, in order: *Display over other apps*, *Notifications*,
   then the system *screen capture* dialog (choose "Entire screen").
2. A round button floats over every app. Drag it anywhere. Long-press hides it
   (bring back via the notification action or the app).
3. Open TradingView / Binance, show the chart, tap the button. Ring spins while
   SCANNING → ANALYZING, then SIGNAL FOUND or WAIT. Result arrives as a notification and in History.

Tips for reliable reading: candlestick chart (not line/Heikin-Ashi), default green/red candles,
price axis on the right, ≥ 40 candles visible, candles not extremely thin, hide extra indicator
panes, linear (not log) scale. If reading is unreliable the app says
**UNABLE TO ANALYZE — Please provide a clearer chart.** instead of guessing.

## How the decision is made
Swing/structure (HH HL LH LL, BOS, CHoCH/MSS) → liquidity (EQH/EQL, sweeps) → displacement →
FVG and order blocks → premium/discount → entry confirmation → stop beyond the sweep, targets at
real opposing swing levels with risk/reward ≥ 1.5.

| Category | Pts |
|---|---|
| Market structure | 20 |
| Liquidity | 20 |
| Order block | 15 |
| Fair value gap | 15 |
| Higher-timeframe alignment | 15 |
| Entry confirmation | 15 |

< 75 WAIT · 75–84 Potential · 85+ High-Confluence. **All hard gates must also pass** (structure shift in
direction, sweep, displacement, entry zone in the right half of the range, confirmation candle, RR).
Higher-timeframe points need data from a higher-TF chart; a single screenshot cannot provide it, so those
15 points stay 0 and the practical maximum on one chart is 85. It is never faked.

## Honest limitations
* Written and unit-tested logic, but **not run on a real phone** by the author. Expect to tune chart
  reading on your own screen/theme (colours live in `vision/CandleExtractor.kt`).
* One screenshot = one timeframe. Multi-timeframe (1D/4H/1H/15M/5M) is only possible if you scan each
  chart separately; the app does not merge scans and says higher-TF alignment was not scored.
* Timeframe is read from TradingView-style headers; if unreadable, your *Preferred timeframe* setting is
  used and labelled as such.
* Pure noise produced a signal in ~1% of simulated random charts. Rule-based SMC has no verified edge
  here; there is no backtest. Treat output as a checklist, decide manually, use stop losses.
* Some phones kill background apps — set battery to *Unrestricted* (Settings ▸ Permissions has a shortcut).
* No external AI/API is used or required. `ChartRecognizer` is the interface to plug a future
  OCR / vision model into (costs and limits would depend on that provider).

## Layout
`capture/` MediaProjection · `overlay/` floating button · `service/` foreground service ·
`vision/` chart recognition · `smc/` engine + scoring · `notify/` · `data/` Room + DataStore · `ui/` Compose.
