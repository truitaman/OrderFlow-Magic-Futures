# 🚀 Wick Magic Futures
### *The Ultimate Sharpshooter Strategy — Now Built for Hyperliquid.*

> **Long. Short. Classic TA. Pure OrderFlow. All in one unified execution engine.**  
> *Trade alongside institutions with native Futures precision. No guessing. Pure market structure action.*

---

Most bots fail at Futures because they treat it like Spot. **Wick Magic Futures** is engineered from the ground up for derivatives. It understands true bidirectional mechanics, native cross-margin tracking, funding fees, and liquidation risks. 

It fuses **two powerful engines into one strategy**: the battle-tested Wick Magic TA engine, and the bleeding-edge OrderFlow engine that reads live market pressure instead of lagging candles.

---

## ⚡ The Futures Edge

**🔄 True Bidirectional Execution**  
Shorts are not just "reversed longs". The bot understands that in a Short, you fear supports, and in a Long, you fear resistances. Its DCA engine flawlessly differentiates between *averaging down* (saving a trade) and *scaling up* (adding to a winner), adapting its triggers dynamically.

**📊 Native Exchange Telemetry**  
No guessed math. It hooks directly into the API of the Exchange (For now Hyperliquid) to extract your exact **Account Margin Usage**, **native ROE**, and **Unrealized PnL** (with funding fees already calculated by the exchange). It mathematically monitors your liquidation risk, keeping your account safe.

**🔪 Kill Position — USD Hard Stop & Trailing Stop**  
A last-line-of-defence emergency exit, independent of all other logic. Define a maximum acceptable loss in USD (`KILL_POSITION_LOSS`): if unrealized PnL drops below that threshold, the position is closed immediately at market, regardless of DCA state, trend, or any other gate. Optionally enable a **trailing mode** (`KILL_POSITION_TRAILING`): the bot tracks your peak uPnL and triggers the kill only if the drawdown from that peak exceeds `KILL_POSITION_TRAIL_USD`. This lets a winning trade breathe while still protecting you if it reverses aggressively.

---

## 🌊 The OrderFlow Engine (Active Focus)

*Why look at where the price was, when you can see where it is going?*

**⚖️ Live Order Book Imbalance**  
Calculates buying vs selling pressure in real time as a normalized score from -1 (pure sell pressure) to +1 (pure buy pressure). This is the primary entry signal.

**🧬 Self-Calibrating via Genetic Algorithm**  
The GA evolves buy/sell thresholds against up to 1000 real candles of this pair's history, across 60 candidates and 220 generations. It recalibrates every 4 hours. No manual tuning needed. 

**🏗️ Multi-Timeframe S/R Confluence**  
Entries and Rebuys only fire when the price is near a validated support or resistance level. 
* *Longs* are blocked from buying just under HTF resistances. 
* *Shorts* are blocked from selling just above HTF supports. 

**🪤 Bull & Bear Trap Detection**  
Cross-checks order book shape against actual trade flow. Suspicious divergences block the entry (bull trap) or the DCA rebuy (bear trap).

**🐋 Institutional Absorption**  
Detects when large players are defending a level — high sell volume on tape, price barely moving, bid side holding. Used as extra confirmation or to override a bearish tape read.

**🏦 Institution Hours**  
Full power during London–NY overlap (7–21 UTC). Outside those hours: scout mode raises thresholds 1.4× and enforces absorption, or block mode stops entries entirely. Exits are never affected.

**📡 Cross-Exchange Signal Pair**  
Some exchanges (like Hyperliquid) have thin public trade history or no PTH endpoint at all. This feature solves that permanently. Configure `OF_SIGNAL_PAIR` and `OF_SIGNAL_EXCHANGE` to point to a more liquid market — for example, read Order Book and PTH from `USDT-ETH` on OKX while executing trades on Hyperliquid. The strategy fetches all OrderFlow signals from the source exchange and fires orders on your target exchange. Full sidebar transparency: the tile shows `USDT-ETH @ okx` when cross-exchange mode is active.

---

## 🎯 The Classic Engine (Included)

For traders who prefer structural Technical Analysis, the classic sniper engine is still fully available and adapted for Futures:
- **Ultra-precise entries:** Heikin Ashi, RSI, WRI, and Breakout logic.
- **Trend Shields:** Wick Magic Trend Algo & SMA200 guards.
- **Elastic Trailing:** Dynamically tightens or loosens based on volatility.
- **FGI Mimix Scaling:** Invest more during fear, scale down during greed.

**🏛️ Institutional Trend Mode**  
When using AUTO direction, the strategy now supports three independent methods to determine the macro trend bias, configurable via `INSTITUTIONAL_TREND_MODE`:
* **`sma200`** — Price above/below the 200-period SMA on the selected timeframe.
* **`supertrend`** — Direction from SuperTrend indicator, configurable period and multiplier.
* **`smacross`** — SMA100 vs SMA200 crossover: bullish above, bearish below.

Each mode can be pointed at either the short or the long timeframe (`INSTITUTIONAL_TREND_TF`), letting you gate direction decisions at a macro level while entries remain local.

---

## 📟 Sidebar — What you see live

Everything is transparent. The Gunbot sidebar renders the exact state of your strategy, your risk, and the order book in real-time.

```text
🎯 Wick Magic v2.9v       🌊 OrderFlow (LONG)
ROC (annualized)          85.4%
🏦 Total Capital          840.00 USD
📉 Unrealized P&L         -2.53 USD
📊 PnL (ROE %)            -5.43%
⚠️ Margin Usage           5.41%
💵 Margin available       799.75 USD
📦 Position size          0.0634 ETH
------------------------------------------------
🌊 Signal (LONG)          🟢 BULLISH
⚖️ Imbalance              0.6134
🟢 Buy threshold          0.5801 ✓
🔴 Sell threshold         -0.4920
🧬 GA samples             700 / 180 ✓
📡 Signal Pair            USDT-ETH @ okx
🪤 Trap guard             CLEAR
🏗️ S/R gate               🎯 IN ZONE (Support)
🏗️ S/R zone               1 / 2
🐋 Absorption             ON
🏦 Inst. Hours            FULL POWER (UTC 14h)
🌊 Partial SELL           READY
🔪 Kill Position          Trail: peak $4.20 | dist $50
```

---

## 📖 Full Documentation

Everything is heavily documented in the Wiki. Learn how to configure your francotirador:

- [What is OrderFlow Trading?](https://github.com/truitaman/OrderFlow-Magic/wiki#-1-what-is-orderflow-trading)
- [The Machine Learning Engine](https://github.com/truitaman/OrderFlow-Magic/wiki#-2-the-machine-learning-engine)
- [Support & Resistance Confluence](https://github.com/truitaman/OrderFlow-Magic/wiki#-3-support--resistance-confluence)
- [Trap Detection, Absorption, Institution Hours](https://github.com/truitaman/OrderFlow-Magic/wiki#-4-bull--bear-trap-detection)
- [Compound Profits & Dump Protection](https://github.com/truitaman/OrderFlow-Magic/wiki#-7-compound-profits)
- [OrderFlow vs Classic Branch](https://github.com/truitaman/OrderFlow-Magic/wiki#-14-orderflow-vs-classic-branch)

---

## 🛍️ Get the Strategy

Requires a valid **Gunbot Defi license** to run on Hyperliquid.

> 🛒 **[Purchase Wick Magic Here](https://www.gunbot.com/devcommunity/wickmagic)**

👉 **Join the Telegram community:**  
https://t.me/+pztaQ-2x7ZFmNDc0

No Defi license yet?  
→ Upgrade from Standard — $190: [Purchase here](https://checkout.gunbot.com/crazymop/promoStandardToDefi)  
→ Upgrade from Pro — $100: [Purchase here](https://checkout.gunbot.com/crazymop/promoProToDefi)  
→ New to Gunbot: [gunbot.com](https://checkout.gunbot.com/crazymop/ultimate)
