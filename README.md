# 🚀 Wick Magic Futures
### *The Ultimate Sharpshooter Strategy — Built for Professional Derivatives Trading.*
> **Long. Short. Classic TA. Pure OrderFlow. All in one unified execution engine.**  
> *Read live market pressure. Trade alongside institutions. On Hyperliquid, dYdX v4, Binance Futures, Kraken Futures and AsterDex.*

**Current version: v1.5.16**

---
Most bots fail at Futures because they treat it like Spot. **Wick Magic Futures** is engineered from the ground up for derivatives. It understands true bidirectional mechanics, native cross-margin tracking, funding fees, open interest and liquidation risk.

It fuses **two powerful engines into one strategy**: the battle-tested Wick Magic TA engine, and the bleeding-edge OrderFlow engine that reads live market pressure instead of lagging candles.

---
## ⚡ The Futures Edge

**🔄 True Bidirectional Execution**  
Shorts are not just "reversed longs". The bot understands that in a Short, you fear supports, and in a Long, you fear resistances. Its DCA engine differentiates between *averaging down* (saving a trade) and *scaling in* (adding to a winner), adapting its triggers dynamically.

**📊 Native Exchange Telemetry**  
No guessed math. It reads your **Account Margin Usage**, **native ROE**, **Unrealized PnL** (with funding already included by the exchange) and **liquidation price** straight from Hyperliquid, dYdX v4, Binance Futures, Kraken Futures and AsterDex. The futures adapter is picked from the pair's real Gunbot exchange, so a position is never read with the wrong adapter.

**🔪 Kill Position — USD & ATR Dynamic Stop, Trailing Stop & Take Profit**  
A last-line-of-defence emergency exit, independent of all other logic. Two modes:

- **USD Mode** — Fixed dollar thresholds for stop, trail, and take profit.
- **ATR Mode** — Thresholds that scale with volatility and position size: `multiplier × ATR × qty`. On a volatile pair the stop is naturally wider; when volatility contracts it tightens automatically. Your USD loss stays as a safety floor, so a quiet market can never shrink the stop below your minimum.

Three protection layers, all independent and combinable:
- **Hard Stop** — If unrealized PnL drops below the threshold, the position closes at market regardless of DCA state, trend, or any other gate.
- **Trailing Stop** — Arms only once real profit reaches the activation level, then trails your peak uPnL. It can never close a position at a loss.
- **Flat Take Profit** — Close immediately when unrealized PnL reaches a fixed target.

**🧯 Kill Position DCA Guard**  
Never doubles your position right before the stop. If an adverse DCA would leave too little room before the Kill Stop, it is skipped.

**🚨 Circuit Breaker — Daily Loss Protection**  
Tracks every real close of the day, including kills and liquidation-guard closes. If daily losses exceed a percentage of your CAPITAL, **all entries are blocked until UTC midnight**. Exits and trailing keep working.

**🪂 Exchange Safety Net (Hyperliquid)**  
Real stop-loss and take-profit **trigger orders resting on Hyperliquid**. They survive a bot crash, a server reboot or an internet outage, and are resized and moved automatically as the position changes. The native take profit can be used on its own.

**💸 Live Funding Rate Intelligence**  
Perpetual futures charge a periodic funding payment between Long and Short holders. In bull frenzies these rates can hit 200–500% annualized. Wick Magic monitors this in real time:
- Live rate and mark price on Hyperliquid, dYdX v4, Binance Futures, Kraken Futures and AsterDex
- Countdown to next settlement with per-exchange precision (including AsterDex's per-symbol intervals)
- Cumulative funding paid/received since position open (Hyperliquid, dYdX v4, AsterDex)
- Next-period rate prediction (Kraken Futures)
- **Funding Rate Guard**: blocks new entries and DCA when the annualized rate exceeds your maximum, optionally only when *your* side pays. Exits are never blocked.

**📈 Open Interest Guard**  
Blocks new entries and DCA when the coin's market-wide Open Interest is dangerously high or moving abnormally fast, the classic setup for liquidation cascades. Hyperliquid, dYdX v4 and AsterDex.

---
## 🌊 The OrderFlow Engine (Active Focus)
*Why look at where the price was, when you can see where it is going?*

**⚖️ Live Order Book Imbalance**  
Calculates buying vs selling pressure in real time as a normalized score from -1 (pure sell pressure) to +1 (pure buy pressure). This is the primary entry signal, confirmed by the live tape (Public Trade History).

**🧬 Self-Calibrating via Genetic Algorithm**  
The GA evolves the buy/sell thresholds against this pair's own recent market history. It recalibrates on a schedule, instantly when volatility shifts, and early when the market runs strongly one way. No manual tuning needed.

**🌀 Market Regime Filters**  
Four optional filters decide whether the market is worth trading at all:
- **⚡ Volatility Percentile Gate** — demands a stronger signal when volatility is unusually high.
- **📊 ATR Strength Classifier** — raises the bar in dead or violent markets, fully automatic.
- **📉 ROC Momentum Filter** — skips entries at the tail end of a fading move.
- **🌀 Chop Guard** — hard-blocks fresh entries while the market has no trend.

**🏗️ Multi-Timeframe S/R Confluence**  
Entries and rebuys only fire near a validated support or resistance level (or anywhere, if you switch the S/R Proximity Gate off).
- *Longs* are blocked from buying just under HTF resistances.
- *Shorts* are blocked from selling just above HTF supports.

**🧲 Smart S/R Bias**  
Gates *which direction* you can enter. Near support → LONG only. Near resistance → SHORT only. A break must be confirmed by full candles of **the level's own timeframe** before bias flips, so a few 5-minute closes can never flip a 4h or daily level.

**🔄 S/R Direction Exit**  
Cuts a small, fresh position early when the market structure turns against it, with a minimum hold time, a configurable loss limit (on price or on margin), a timeframe filter and an optional Confluence confirmation.

**🪤 Bull & Bear Trap Detection**  
Cross-checks order book shape against actual trade flow. Suspicious divergences block the entry or the DCA rebuy.

**🐋 Institutional Absorption**  
Detects when large players are defending a level. Used as extra confirmation for entries.

**🏦 Institution Hours**  
Full power during the London–NY window (configurable, default 7–21 UTC). Outside it: scout mode with stricter requirements, or block mode. Separate weekend handling. Exits are never affected.

**📡 Cross-Exchange Signal Pair**  
Read the Order Book and tape from a more liquid market (e.g. `USDT-ETH` on Binance Futures) while executing on Hyperliquid. The sidebar shows `USDT-ETH @ binanceFutures` when active.

**🚦 Entry Gate**  
One sidebar tile that tells you exactly which gate is holding back a fresh entry. No more guessing why the bot is not entering.

---
## 📈 Position Management

**🧾 True Cost Basis**  
With below-entry partials on, the bot closes profitable fills even while the position average is underwater, and tracks the **real** cost of what is left. Target Exit, trailing, re-entries and DCA all follow that True Cost, shown live in the sidebar and on the chart.

**🚀 Scale-In**  
Add to a winning position with separate caps for Long and Short, only inside a profit window and never once the trail has armed.

**💧 Auto-ATR DCA**  
DCA spacing that adapts to volatility, with a ladder that keeps working after bounces.

**📐 DCA Trading Limit Multiplier** *(experimental)*  
Each DCA buy can grow the deeper the ladder goes, manually or by ATR distance, always capped.

**🔥 Win Streak Trading Limit** *(experimental)*  
The next entry grows after each winning cycle, and returns to base after a DCA or a losing close.

**💰 Compound Profits & Min Notional**  
Trade size follows your realized PnL. With Compound on, a size below the exchange minimum is raised to it; otherwise the order is blocked instead of rejected. A full close is never blocked by the exchange minimum.

---
## 🎯 The Classic Engine (Included)
For traders who prefer structural Technical Analysis, the classic sniper engine is fully adapted for Futures:
- **Ultra-precise entries:** Heikin Ashi, RSI, WRI, and Breakout logic.
- **Trend Shields:** Wick Magic Trend Algo & SMA200 guards.
- **Elastic Trailing:** Dynamically tightens or loosens based on volatility.
- **FGI Mimix Scaling:** Invest more during fear, scale down during greed (Alternative.me or CoinMarketCap).

**🏛️ Institutional Trend Mode**  
When using AUTO direction, five macro trend engines are available:
- **`sma200`** — Price above/below the 200-period SMA.
- **`supertrend`** — Direction from SuperTrend, configurable period and multiplier.
- **`smacross`** — SMA100 vs SMA200 crossover.
- **`emadual`** — A daily and a higher-timeframe filter must agree; SuperTrend breaks ties.
- **`confluence`** — Only commits to a direction when enough independent confirmations agree, and holds it when they do not.

Each engine can be pointed at the short, long, or daily timeframe. Add the experimental **Confluence Entry Gate** to only open fresh entries when the Confluence Score backs their side.

---
## 📟 Sidebar — What You See Live
Everything is transparent. The Gunbot sidebar renders the exact state of your strategy, risk, order book, funding and open interest in real time, grouped in clear sections.

```text
🌊 OrderFlow Fut 1.5.16   LONG
── 📊 PERFORMANCE & STATUS ──────────────────
ROC (annualized)          212.4%
⚙️ Mode                   💰 CLOSE
── 🎯 POSITION ──────────────────────────────
⚠️ Margin Usage           5.41%
💰 Position margin        33.10 USDT
📦 Position size          0.15 ZEC
🧾 True Cost              660.20 (+0.42%)
💵 True uPnL              +0.05 USDT (+0.14% ROE)
── 🛟 RISK & PROTECTION ─────────────────────
🪢 Rekt Liq. Price        426.70 (35.4% away)
🛡️ Liq. Limit             35.4% / 20% → 533.40
🔪 Kill Position          $+0.47 / -$3.0 [ATR]
🚨 Circuit Breaker        $-5.20 / -$14.0
🛟 Native SL/TP           SL 636.80 (3.59%) / TP off
── 🌊 SIGNAL & ENTRIES ──────────────────────
🌊 Signal (LONG)          🟢 BULLISH
🚦 Entry Gate             ⚪ N/A (position open)
⚖️ Imbalance              0.0576
🟢 Buy threshold          0.0576
🧬 GA samples             700 / 120 ✓
🪤 Trap guard             CLEAR
🔬 PTH trades             1000 (2s ago)
🏗️ S/R gate               ON (±0.8%)
🔄 S/R DirExit            ARMED · 4h
── 🏛️ TREND & FILTERS ───────────────────────
🏛️ Inst. Trend            LONG
🌀 Chop Guard             ✅ OK · ADX 27.4 / 18
⚡ Vol Gate               P45 / 80
── 💸 FUNDING & MARKET ──────────────────────
📌 Mark Price             660.50
💸 Funding Rate           🟢 +0.0028% / 1h
⏰ Next Funding           45m 12s
🛡️ Funding Guard          ✅ OK (24.5% PAY)
📈 Open Interest          $237,070,898
```

---
## 🔥 What's New in v1.5.16

**🛡️ Protection**
- **Kill Position DCA Guard** — no adverse DCA right before the Kill Stop.
- **Reliable trailing kill** — arms only when the open profit really reaches the activation level, and stays armed until close.
- **Native Take Profit on its own** — the Exchange Safety Net TP no longer needs Kill Position.
- **Stronger Circuit Breaker** — counts every real close, including Liquidation Guard closes.
- **Restart & desync safety** — no adds on the restart candle, no re-entry on top of a fill Gunbot cannot see, no entries on frozen candles.

**🎯 Smarter entries & exits**
- **S/R Proximity Gate** switch, and **HTF break confirmation** for Smart S/R Bias.
- **S/R Direction Exit**: Min Hold, Imbalance Mode, Strict Timeframe, Confluence Confirmation, Max Position up to 10×, and an experimental **Re-Entry Lock**.
- **Confluence Entry Gate** *(experimental)*.
- More precise **partial closes**: checked at the real exit price, valid between entry and True Cost, and working with small trade sizes.

**📈 Position management**
- **True Cost Basis** now drives Target Exit, trailing, re-entries and DCA, with new `🧾 True Cost` / `💵 True uPnL` tiles and a TCB chart line.
- **DCA Trading Limit Multiplier** and **Win Streak Trading Limit** *(experimental)*.
- **Scale-In: Allow While Underwater**.
- **Compound Profits** rebuilt: stored in the pair state, shorts counted correctly, reset when switched off.
- **Min Notional** now also covers closes, partials and dust.

**🌐 Exchanges, editor & chart**
- Futures adapter detected from the pair's exchange.
- Binance public tape keeps working when the core returns an API-key error.
- AsterDex Multi-Assets and BTC collateral mode (Kraken / Binance) capital fixes.
- New **Protections** and **Fear & Greed** editor tabs.
- Entry, Target Exit, TCB, Kill Stop and Kill Trail lines on the chart.

<details>
<summary><b>v1.5.4 highlights</b></summary>

<br>

- **Exchange Safety Net** on Hyperliquid and **Open Interest Guard**.
- **Kill Position Trail Activation** — the trailing kill never closes at a loss.
- **ATR Strength Classifier, ROC Momentum Filter and Chop Guard**.
- **`confluence` trend engine** and self-correcting GA calibration.
- **Scale-In** redesign with separate Long/Short caps.
- **AsterDex Futures** and **BYDFi Futures** support, **`hl_autopair`** preset.
- **🚦 Entry Gate** tile.

</details>

<details>
<summary><b>v1.3.0 and v1.2.7 highlights</b></summary>

<br>

- **Kill Position ATR mode** and **flat USD Take Profit**.
- **Circuit Breaker** — daily realized PnL limit.
- **Volatility Percentile Gate**.
- **Funding Rate Intelligence** on every supported exchange, with the annualized **Funding Rate Guard**.
- **Kraken Futures** moved to Final.

</details>

---
## 🌐 Supported Exchanges

| Exchange | Status | Features |
|---|---|---|
| **Hyperliquid** | ✅ Final | Full OF, PTH via Signal Pair, live funding, Open Interest, Exchange Safety Net, unified collateral |
| **dYdX v4** | ✅ Final | Full OF, live PTH, oracle price, live funding, Open Interest |
| **Binance Futures** | ✅ Final | Full OF, live PTH, mark price, funding rate (8h), BTC collateral preset |
| **Kraken Futures** | ✅ Final | Full OF, live PTH, multi-collateral, funding + next-period prediction, BTC collateral preset |
| **AsterDex** | ✅ Final | Full OF, live PTH, live funding with per-symbol interval, real cumulative funding, Open Interest |
| **BYDFi** | ✅ Available | Position, margin and liquidation telemetry |

---
## 📖 Full Documentation
Everything is documented in the Wiki:
- [What is OrderFlow Trading in Futures?](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-1-what-is-orderflow-trading-in-futures)
- [The Machine Learning Engine (GA)](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-2-the-machine-learning-engine)
- [Support & Resistance Confluence](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-3-support--resistance-confluence-futures-adapted)
- [Smart S/R Bias & Direction Exit](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-4-smart-sr-bias-direction-gate)
- [Trap Detection, Absorption, Institution Hours](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-5-bull--bear-trap-detection)
- [Cross-Exchange Signal Pair](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-8-cross-exchange-signal-pair)
- [Market Regime Filters & Entry Gate](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-9-market-regime-filters)
- [Kill Position, DCA Guard & Circuit Breaker](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-10-kill-position--usd-hard-stop--trailing-stop)
- [Liquidation Guard](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-11-liquidation-guard--native-margin-protection)
- [Exchange Safety Net (Hyperliquid)](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-12-exchange-safety-net-hyperliquid)
- [Funding Rate Intelligence](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-13-funding-rate-intelligence-)
- [Open Interest Guard](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-14-open-interest-guard)
- [Institutional Trend Mode & Confluence Entry Gate](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-15-institutional-trend-mode)
- [Margin, Compound Profits & Win Streak](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-16-futures-margin--compound-profits)
- [Below-Entry Partials & True Cost Basis](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-below-entry-partial-close-soft-reduction)
- [Auto-ATR DCA, DCA Multiplier & Scale-In](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-18-auto-atr-dca--scale-in)
- [OrderFlow Presets](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-19-orderflow-presets)
- [Full Settings Reference](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-20-editor-settings-reference)
- [Sidebar & Chart Lines](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-21-the-orderflow-sidebar-futures-edition)
- [OrderFlow vs Classic Branch](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-23-orderflow-vs-classic-branch)
- [Tips & Best Practices](https://github.com/truitaman/OrderFlow-Magic-Futures/wiki#-24-tips--best-practices-for-futures)

---
## 🛍️ Get the Strategy
Requires a valid **Gunbot Defi license** to execute.

> 🛒 **[Purchase Wick Magic Here](https://www.gunbot.com/devcommunity/wickmagic-futures)**

👉 **Join the Telegram community:**  
[https://t.me/+pztaQ-2x7ZFmNDc0](https://t.me/+xQtQ9Y4AOc9lZTNk)

No Defi license yet?  
→ Upgrade from Standard — $160: [Purchase here](https://checkout.gunbot.com/crazymop/promoStandardToDefi)  
→ Upgrade from Pro — $70: [Purchase here](https://checkout.gunbot.com/crazymop/promoProToDefi)  
→ New to Gunbot: [gunbot.com](https://checkout.gunbot.com/crazymop/ultimate)
