# 🚀 Wick Magic Futures
### *The Ultimate Sharpshooter Strategy — Built for Professional Derivatives Trading.*
> **Long. Short. Classic TA. Pure OrderFlow. All in one unified execution engine.**  
> *Read live market pressure. Trade alongside institutions. On Hyperliquid, dYdX v4, Binance Futures, and Kraken Futures.*
---
Most bots fail at Futures because they treat it like Spot. **Wick Magic Futures** is engineered from the ground up for derivatives. It understands true bidirectional mechanics, native cross-margin tracking, funding fees, and liquidation risk.
It fuses **two powerful engines into one strategy**: the battle-tested Wick Magic TA engine, and the bleeding-edge OrderFlow engine that reads live market pressure instead of lagging candles.
---
## ⚡ The Futures Edge
**🔄 True Bidirectional Execution**  
Shorts are not just "reversed longs". The bot understands that in a Short, you fear supports, and in a Long, you fear resistances. Its DCA engine flawlessly differentiates between *averaging down* (saving a trade) and *scaling up* (adding to a winner), adapting its triggers dynamically.

**📊 Native Exchange Telemetry**  
No guessed math. It hooks directly into the exchange API (Hyperliquid / dYdX v4 / Binance Futures / Kraken Futures) to extract your **Account Margin Usage**, **native ROE**, and **Unrealized PnL** with funding fees already calculated by the exchange. It mathematically monitors your liquidation risk, keeping your account safe at all times.

**🔪 Kill Position — USD Hard Stop, Trailing Stop & Take Profit**  
A last-line-of-defence emergency exit, independent of all other logic. Three modes available:
- **Hard Stop** — If unrealized PnL drops below a configured USD threshold, the position closes immediately at market regardless of DCA state, trend, or any other gate.
- **Trailing Stop** — Tracks your peak uPnL. Fires only if the drawdown from that peak exceeds your trail distance in USD. Lets winning trades breathe while protecting against reversals.
- **Flat Take Profit** — Close immediately when unrealized PnL reaches a fixed USD target. Ideal for small entries where a flat dollar goal makes more sense than a percentage.

All three modes are independent and can be combined.

**💸 Live Funding Rate Intelligence** *(New in v1.2.7)*  
Perpetual futures charge a periodic funding payment between Long and Short holders. In bull frenzies these rates can hit 200–500% annualized — silently bleeding your margin every hour. Wick Magic now monitors this in real time:
- Live rate and mark price via WebSocket on all four exchanges
- Countdown to next settlement with per-exchange precision
- Cumulative funding paid/received since position open (Hyperliquid and dYdX v4)
- Next-period rate prediction (Kraken Futures)
- **Funding Rate Guard**: automatically blocks new entries and DCA when the annualized rate exceeds your configured maximum — exits are never blocked

---
## 🌊 The OrderFlow Engine (Active Focus)
*Why look at where the price was, when you can see where it is going?*

**⚖️ Live Order Book Imbalance**  
Calculates buying vs selling pressure in real time as a normalized score from -1 (pure sell pressure) to +1 (pure buy pressure). This is the primary entry signal.

**🧬 Self-Calibrating via Genetic Algorithm**  
The GA evolves buy/sell thresholds against up to 2000 real candles of this pair's history, across 60 candidates and 220 generations. It recalibrates every 4 hours, or instantly if ATR volatility shifts. No manual tuning needed.

**🏗️ Multi-Timeframe S/R Confluence**  
Entries and rebuys only fire when price is near a validated support or resistance level.
- *Longs* are blocked from buying just under HTF resistances.
- *Shorts* are blocked from selling just above HTF supports.

**🧲 Smart S/R Bias**  
Goes beyond confluence. Gates *which direction* you can enter based on where price is relative to the zone. Near support → LONG only. Near resistance → SHORT only. Break confirmation requires N full candles on the other side before bias flips — no false breakout entries.

**🪤 Bull & Bear Trap Detection**  
Cross-checks order book shape against actual trade flow. Suspicious divergences block the entry or the DCA rebuy.

**🐋 Institutional Absorption**  
Detects when large players are defending a level — high sell volume on tape, price barely moving, bid side holding. Used as extra confirmation or to override a bearish tape read.

**🏦 Institution Hours**  
Full power during London–NY overlap (7–21 UTC). Outside those hours: scout mode raises thresholds 1.4× and enforces absorption, or block mode stops entries entirely. Exits are never affected.

**📡 Cross-Exchange Signal Pair**  
Some exchanges (like Hyperliquid) have thin public trade history or no PTH endpoint at all. Configure `OF_SIGNAL_PAIR` and `OF_SIGNAL_EXCHANGE` to point to a more liquid market — read Order Book and PTH from `USDT-ETH` on OKX while executing trades on Hyperliquid. Full sidebar transparency: tile shows `USDT-ETH @ okx` when cross-exchange mode is active.

---
## 🎯 The Classic Engine (Included)
For traders who prefer structural Technical Analysis, the classic sniper engine is fully adapted for Futures:
- **Ultra-precise entries:** Heikin Ashi, RSI, WRI, and Breakout logic.
- **Trend Shields:** Wick Magic Trend Algo & SMA200 guards.
- **Elastic Trailing:** Dynamically tightens or loosens based on volatility.
- **FGI Mimix Scaling:** Invest more during fear, scale down during greed.

**🏛️ Institutional Trend Mode**  
When using AUTO direction, three independent macro trend engines are available:
- **`sma200`** — Price above/below the 200-period SMA on the selected timeframe.
- **`supertrend`** — Direction from SuperTrend, configurable period and multiplier.
- **`smacross`** — SMA100 vs SMA200 crossover: bullish above, bearish below.

Each engine can be pointed at the short, long, or daily timeframe — letting you gate direction at macro level while entries remain fast and local.

---
## 📟 Sidebar — What You See Live
Everything is transparent. The Gunbot sidebar renders the exact state of your strategy, risk, order book, and funding in real-time.

```text
🎯 OrderFlow Fut 1.2.7    🌊 OrderFlow (SHORT)
ROC (annualized)          999.0%
🏦 Total Capital          1000.00 USD
📉 Unrealized P&L         -0.14 USD
📊 PnL (ROE %)            -0.43%
⚠️ Margin Usage           5.41%
💵 Margin available       2346.47 USD
📦 Position size          0.0091 ETH
------------------------------------------------
🌊 Signal (SHORT)         🔴 BEARISH
⚖️ Imbalance              -0.0687
🟢 Buy threshold          0.0500
🔴 Sell threshold         -0.0715
🧬 GA samples             225 / 120 ✓
📡 Signal Pair            USDT-ETH @ okx
🪤 Trap guard             CLEAR
🔬 PTH trades             1000 (27s ago)
🏗️ S/R gate               ON (±1%)
🏗️ S/R zone               0 / 2
🐋 Absorption             ON
🏦 Inst. Hours            FULL POWER (UTC 16h)
------------------------------------------------
📌 Mark Price             2174.35
💸 Funding Rate           🟢 -0.0028% / 1h
📊 Cum. Funding           -$0.24 since open
⏰ Next Funding           45m 12s
------------------------------------------------
🌊 Partial SELL           READY
🔪 Kill Position          Trail: peak $4.20 | dist $50
🛡️ Liq. Limit (%)         20% (SAFE ✓)
🪢 Rekt Liq. Price        1985.30 (8.7% dist)
```

---
## 🔥 What's New in v1.2.7

### 💸 Funding Rate Intelligence (All 4 Exchanges)
The biggest quality-of-life addition to date. Every supported exchange now streams live funding rate data directly into the strategy:

| Exchange | Feed | Settlement | Extras |
|---|---|---|---|
| Hyperliquid | WebSocket `activeAssetCtx` | 1h | Cumulative funding since open |
| dYdX v4 | WebSocket `v4_markets` | 1h | Oracle price, cumulative funding |
| Binance Futures | REST poll every 30s | 8h | Mark price, next settlement |
| Kraken Futures | WebSocket `ticker` | 1h | Live rate + next period prediction |

**Funding Rate Guard** — configure a maximum annualized rate (e.g. 300%). When the live rate exceeds it, all entries and DCA are blocked. Exits are never affected. Rates are normalized to APR across all exchanges for a fair comparison regardless of settlement frequency.

### 🔪 Kill Position Take Profit (USD)
New third mode for Kill Position: set a flat USD profit target. The moment unrealized PnL hits +$X, the position closes immediately at market — no trailing required, no waiting for an OF signal. Ideal for small position sizes where dollar targets are more intuitive than percentages.

### 🏁 Kraken Futures — Now Final
Kraken Futures moves from Beta to **Final** status. The dedicated real-time WebSocket connection for multi-collateral margin reading is fully stable, and funding rate telemetry via the `ticker` feed is confirmed working on all major pairs.

### ⚙️ New Orderflow Parameters
- **`ORDERFLOW_REBUY_ATR_MULT`** — Fine-tune the ATR multiplier for auto DCA spread without changing the base timeframe.
- **`ORDERFLOW_EXIT_IGNORE_ALL`** — Bypass all OF exit conditions (exits fire on profit target and trailing only).
- **`OF_PARTIAL_TA_EXIT`** — Allow partial closes on HA color flip even in pure OF mode.
- **`MIN_VOLUME_TO_SELL`** — Prevent attempting to close dust positions below a USD notional threshold.

---
## 🌐 Supported Exchanges

| Exchange | Status | Features |
|---|---|---|
| **Hyperliquid** | ✅ Final | Full OF, PTH via Signal Pair, live funding, unified collateral |
| **dYdX v4** | ✅ Final | Full OF, live PTH, oracle price, live funding |
| **Binance Futures** | ✅ Final | Full OF, live PTH, mark price, funding rate (8h) |
| **Kraken Futures** | ✅ Final | Full OF, live PTH, multi-collateral, funding + prediction |

---
## 📖 Full Documentation
Everything is heavily documented in the Wiki:
- [What is OrderFlow Trading?](https://github.com/truitaman/OrderFlow-Magic/wiki#-1-what-is-orderflow-trading)
- [The Machine Learning Engine (GA)](https://github.com/truitaman/OrderFlow-Magic/wiki#-2-the-machine-learning-engine)
- [Support & Resistance Confluence](https://github.com/truitaman/OrderFlow-Magic/wiki#-3-support--resistance-confluence)
- [Smart S/R Bias](https://github.com/truitaman/OrderFlow-Magic/wiki#-4-smart-sr-bias-direction-gate)
- [Trap Detection, Absorption, Institution Hours](https://github.com/truitaman/OrderFlow-Magic/wiki#-5-bull--bear-trap-detection)
- [Funding Rate Intelligence](https://github.com/truitaman/OrderFlow-Magic/wiki#-11-funding-rate-intelligence-)
- [Kill Position — USD Hard Stop & Trailing](https://github.com/truitaman/OrderFlow-Magic/wiki#-9-kill-position--usd-hard-stop--trailing-stop)
- [Compound Profits & Dump Protection](https://github.com/truitaman/OrderFlow-Magic/wiki#-13-futures-margin--compound-profits)
- [OrderFlow vs Classic Branch](https://github.com/truitaman/OrderFlow-Magic/wiki#-20-orderflow-vs-classic-branch)
- [Full Settings Reference](https://github.com/truitaman/OrderFlow-Magic/wiki#-17-editor-settings-reference)

---
## 🛍️ Get the Strategy
Requires a valid **Gunbot Defi license** to run on Hyperliquid or dYdX v4.

> 🛒 **[Purchase Wick Magic Here](https://www.gunbot.com/devcommunity/wickmagic-futures)**

👉 **Join the Telegram community:**  
https://t.me/+pztaQ-2x7ZFmNDc0

No Defi license yet?  
→ Upgrade from Standard — $190: [Purchase here](https://checkout.gunbot.com/crazymop/promoStandardToDefi)  
→ Upgrade from Pro — $100: [Purchase here](https://checkout.gunbot.com/crazymop/promoProToDefi)  
→ New to Gunbot: [gunbot.com](https://checkout.gunbot.com/crazymop/ultimate)
