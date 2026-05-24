# 🌊 OrderFlow Magic Futures

**Entries driven by real market pressure. Not candles. Not RSI. The actual order book.**
*Read what institutions are doing. Trade alongside them with native derivatives precision.*

---

## 🔹 1. What is OrderFlow Trading in Futures?

Most trading strategies read **candles** — they look at where the price has been and try to predict where it goes next. That's called technical analysis.

OrderFlow is different. Instead of looking at the past, it looks at **what is happening right now** in the live market.

On high-performance Futures exchanges like Hyperliquid, dYdX v4, Binance Futures, and Kraken Futures, there are two live data sources:

### 📖 The Live Order Book

The order book is a real-time list of all pending Long (Bid) and Short (Ask) orders on the exchange. Think of it like a two-sided auction:
- The **bid side** = buyers waiting to go Long or close Shorts.
- The **ask side** = sellers waiting to go Short or close Longs.

When there are significantly more buyers than sellers by volume, the market has **bullish pressure**. When sellers dominate, it puts **bearish pressure** on the market.

Wick Magic Futures calculates this as an **imbalance score** from -1 (pure selling pressure) to +1 (pure buying pressure). This is your core sniper signal.

### 🔬 Public Trade History (PTH) & Tape Confirmation

The order book shows *pending* orders. The PTH shows orders that have *already executed* — real trades that happened in the last 90 seconds.

**Hyperliquid, dYdX v4, Binance Futures, and Kraken Futures** all provide live, granular Public Trade History. Wick Magic Futures hooks directly into their WebSockets to read the live tape, cross-checking order book imbalances with real executed volume. If the book looks bullish but the tape shows aggressive sellers, the bot blocks the entry. *(Note: If you ever trade on an exchange without PTH, the GA gracefully falls back to pure order book training and continues to run with high accuracy).*

### 🔄 True Bidirectional Logic

Because this is Futures, the OrderFlow engine understands direction:
- **Going Long:** You want a bullish OB imbalance (+). You fear hitting resistance levels. You DCA when price drops.
- **Going Short:** You want a bearish OB imbalance (-). You fear hitting support levels. You DCA when price rises.

The bot handles this inversion automatically. It doesn't just reverse the math; it reverses the market psychology.

---

## 🔹 2. The Machine Learning Engine

One of the most powerful parts of the OrderFlow Magic is that it **calibrates itself autoMagically** using a custom Genetic Algorithm (GA).

### What is a Genetic Algorithm?

Instead of hard-coding "buy when imbalance > 0.35", the algorithm:
1. Creates a **population of candidate thresholds** (e.g., 60 random buy/sell threshold pairs).
2. Tests each candidate against the last 200–2000 real candle samples for this specific pair.
3. **Scores each candidate** — did it correctly predict the next candle direction?
4. **Selects the best performers**, breeds them, and runs again.
5. After 220 generations, the best-performing threshold pair becomes the live trading threshold.

This means on `ETH-USD` the GA might settle on `buyTh: 0.52` while on `BTC-USD` it settles on `buyTh: 0.38`. No manual tuning needed. It recalibrates every 4 hours, or instantly if ATR volatility shifts.

**What you see in the sidebar:**
- `🧬 GA samples` — how many candles have been collected for training.
- `🟢 Buy threshold` — the GA's current calibrated entry level.
- `🔴 Sell threshold` — the GA's current calibrated exit level.

**Warmup Sampling**
When you add a brand new pair, the GA starts with zero samples. By default, **Warmup Sampling** is enabled — this blocks new orderflow entries until enough calibration samples have been collected (default 120–180 samples). This ensures your very first trade fires with properly calibrated thresholds, not blind guesses.

---

## 🔹 3. Support & Resistance Confluence (Futures Adapted)

Even with a perfect orderflow signal, entering in the middle of nowhere is reckless. Wick Magic adds a **multi-timeframe S/R gate**.

When S/R Confluence is enabled, orderflow entries are only allowed when the price is **near one of these confirmed zones**. But because this is Futures, it protects you bidirectionally:
- **In a Long:** It will **block** you from opening a long if you are right underneath an HTF Resistance.
- **In a Short:** It will **block** you from opening a short if you are right on top of an HTF Support.

**S/R Zone Exhaustion (The DCA Shield)**
When price oscillates around the same support/resistance zone, the orderflow signal can fire multiple re-entries (DCA) at essentially the same price. You end up stacking leverage without any real progression.

**Max Re-Entries per S/R Level** solves this. Once you exhaust your allowed entries at a specific price band, new re-entries are blocked until price reaches the **next distinct structural level**.

---

## 🔹 4. Smart S/R Bias (Direction Gate)

Standard S/R Confluence only gates *whether* to enter near a zone. **Smart S/R Bias** goes one step further — it gates *which direction* you can enter based on where price is relative to the zone.

**How it works**

| Price position | Zone type | Bias | Effect |
|---|---|---|---|
| Above a support zone | Support | **LONG only** | SHORT entries blocked |
| Below a support zone (confirmed break) | Support → Resistance | **SHORT only** | LONG entries blocked |
| Below a resistance zone | Resistance | **SHORT only** | LONG entries blocked |
| Above a resistance zone (confirmed break) | Resistance → Support | **LONG only** | SHORT entries blocked |

**Break Confirmation**
A price crossing a zone is **not enough** to flip bias. The bot waits until **N candles have fully closed** on the other side of the zone before considering the break confirmed. Until then, entries in the breakout direction are blocked (ambiguous zone).

This prevents trading false breakouts where price spikes through a level and immediately reverses.

**Rules applied to new entries only.** Rebuys and DCA into an existing position are never affected by Smart S/R Bias.
**Requires S/R Confluence enabled** — Smart S/R Bias reads from the same multi-timeframe confluence engine.

---

## 🔹 5. Bull & Bear Trap Detection

### How It Works

Markets fake moves to liquidate high-leverage traders. A **bull trap** is a sharp push upward that quickly reverses. A **bear trap** is a sudden drop that quickly bounces back.

Wick Magic cross-checks the order book shape against the actual tape. If the book is showing one direction but real executed trades contradict it, the entry is blocked before you get caught in the fake move.

You'll see `🪤 Trap guard: CLEAR` or `🐻 Bear trap` live in the sidebar.

---

## 🔹 6. Institutional Absorption

### What It Detects

When large market makers defend a price level, they absorb incoming pressure without allowing the price to break. This shows up as:
- High volume pushing against a level.
- Price is barely moving.
- Strong bid/ask side holding in the order book.

Wick Magic detects this pattern and uses it as **extra confirmation** for entries. Best used on major pairs (BTC, ETH, SOL) where institutional liquidity is thick.

---

## 🔹 7. Institution Hours

### Session Gating Logic

Crypto trades 24/7, but institutional volume isn't evenly distributed. The bulk of real order flow concentrates during the **London session** through the **New York close** (7–21 UTC). Outside those hours, order books are thinner, and spoofing is common.

- **Scout mode** (default off-hours) — entries are still possible, but the conviction bar is raised. The orderflow threshold is multiplied by 1.4x (40% stricter) and Institutional Absorption detection is forced on. Only the cleanest, highest-conviction signals get through.
- **Block mode** — zero entries allowed outside of institutional hours. Safe and simple.

*Note: Sells, partial exits, and trailing stops are NEVER affected by Institution Hours. You always take profits 24/7.*

---

## 🔹 8. Cross-Exchange Signal Pair

*Trade on Hyperliquid. Read the market on binance Futures. Best of both worlds.*

**The Problem: Not All Exchanges Are Equal**
Hyperliquid is an exceptional execution venue — low latency, deep liquidity, and native cross-margin Futures. But it does not expose a Public Trade History (PTH) endpoint via its public API. This is a real limitation for OrderFlow: without tape data, the GA trains only on order book imbalance, losing the second confirmation layer that filters out spoofed bids and fake walls.

**Signal Pair** and **Signal Exchange** solve this permanently. They let you point the OrderFlow engine at a completely different pair and exchange as its **data source**, while the actual trades still execute on your configured futures exchange.

**How it works**
1. You configure `Signal Pair: USDT-ETH` and `Signal Exchange: binanceFutures`.
2. On every strategy tick, the bot reads the **live order book** and **PTH tape** from binance's `USDT-ETH` market via Gunbot's internal ledger.
3. All signal calculations — imbalance score, GA calibration, trap detection, iceberg detection — run against binance data.
4. The resulting buy/sell signal is applied to your Hyperliquid `USD-ETH` position.

You get the full two-layer OrderFlow signal (OB + PTH) with the execution quality of Hyperliquid's native cross-margin engine.

**Requirements**
- The signal pair **must be cycling in Gunbot** on the source exchange. The strategy fetches its data from Gunbot's internal ledger, which is only populated for active pairs. A dedicated cycling pair or a second active pair on the same Gunbot instance both work.
- If Signal Exchange is left empty, the bot defaults to the same exchange as the trading pair. This is useful for same-exchange signal routing (e.g., read a more liquid pair's OB to gate a low-cap pair on the same DEX).
- If both parameters are empty, the feature is fully disabled. Zero code path change, zero regression on existing configurations.

**Same-Exchange Mode**
Signal Pair also works within the same exchange. For example, if you trade a low-volume altcoin futures pair but want to use a more liquid pair's order book as the signal source, the same config applies with Signal Exchange left empty. The key requirement remains: the signal pair must be cycling.

**Sidebar**
When active, a dedicated tile appears in the sidebar immediately after the GA samples row:

```text
📡 Signal Pair    USDT-ETH @ binanceFutures

```

The `@ binanceFutures` suffix only appears when the signal exchange differs from the trading exchange. The tooltip confirms which pair and exchange are being used as signal source. The tile is hidden entirely when the feature is disabled.

---

## 🔹 9. Kill Position — USD Hard Stop & Trailing Stop

*Your last line of defence. Independent of everything. Always watching.*

### What It Is

Kill Position is an emergency exit mechanism that monitors your **unrealized PnL in USD** on every strategy tick, independently of all other logic — trend state, OF signal, DCA counter, cooldowns, nothing affects it. If the position hits the configured threshold, it closes immediately at market.

It is not a stop-loss in the traditional sense. It does not interact with the normal exit path. It runs **first**, before any other check, and returns immediately after closing. Think of it as a circuit breaker.

The uPnL value is read **directly from the exchange adapter** (Hyperliquid, dYdX, Binance, Kraken) — it already includes accumulated funding fees. This is your true net exposure at the time of the check, not a guessed calculation.

**Mode 1: Hard Stop (Kill Position Loss (USD))**
Define a maximum tolerable loss in USD. If unrealized PnL drops to or below the negative of your Kill Position Loss (USD), the position is closed at market immediately.

```text
Enable Kill Position (uPnL Stop): true
Kill Position Loss (USD):         100      ← close if uPnL <= -$100

```

**Mode 2: Trailing Stop (Enable Trailing Stop)**
The trailing mode adds a **profit-protection layer** on top of the hard stop. The bot continuously tracks the **peak unrealized PnL** reached during the life of the position. If the current uPnL drops more than your Trail Distance (USD) below that peak, the kill fires.

```text
Enable Kill Position (uPnL Stop): true
Enable Trailing Stop:             true
Trail Distance (USD):             50   ← close if uPnL drops $50 from peak

```

**Mode 3: Flat USD Take Profit (Kill Position Take Profit (USD))**
A fixed dollar profit target independent of ROE percentage. If your unrealized PnL reaches +$X, the position closes immediately at market — no trailing, no waiting for an OF signal. Ideal for small entries where a flat dollar target makes more practical sense than a percentage.

```text
Enable Kill Position (uPnL Stop): true
Kill Position Take Profit (USD):  30   ← close as soon as uPnL >= +$30

```

All three modes are independent and can be combined. Hard stop and trailing run in parallel. Take profit fires on its own.

**Interaction with other features**

* Kill Position runs **before** the OrderFlow check, trailing stop, partial sell logic, dump protection, and all other exits.
* It fires on every tick while a position is open. There is no cooldown between checks.
* The peak tracker resets to zero automatically when the position is fully flat.
* A **GUI notification** fires at the moment the kill triggers, displaying the reason (hard stop, trailing stop, or take profit) and the exact PnL at close.

**Sidebar**

```text
🔪 Kill Position    Trail: peak $4.20 | dist $50

```

When only hard stop is active: `🔪 Kill Position    HARD STOP only`
When disabled: `🔪 Kill Position    OFF`

---

## 🔹 10. Liquidation Guard — Native Margin Protection

*Your ultimate safety net against total position loss.*

### What It Is

The **Liquidation Guard** is a native safety mechanism designed specifically for the high-risk environment of Futures trading. While Kill Position focuses on your PnL in USD, the Liquidation Guard focuses on something more fundamental: **how far away are you from losing your entire margin?**

In Futures, every position has a **Liquidation Price** — the exact price at which the exchange will forcibly close your position and seize your collateral. If the market ever reaches that price, your margin is gone. The Liquidation Guard prevents this by closing your position automatically before you ever get there.

### How It Works

The bot reads your Liquidation Price directly from the exchange API on every tick and compares it to the current market price. When the gap between them shrinks below your configured safety buffer, it fires an immediate market close.

**Direction-aware:**

* **Long position:** Your liquidation price is *below* your entry. The guard fires if price drops too close to that floor.
* **Short position:** Your liquidation price is *above* your entry. The guard fires if price rises too close to that ceiling.

Think of the buffer as a safety corridor. The exchange will liquidate you at 0% distance. You're telling the bot to get out at 20% distance — giving you a comfortable margin for slippage, API latency, and volatility spikes.

### Parameters

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable Liquidation Guard** | `true` | Activates real-time liquidation distance monitoring. |
| **Liquidation Buffer (%)** | `20` | Minimum allowed distance between current price and liquidation price. If the gap drops below this %, the guard fires immediately. |

### Why 20%?

In highly volatile markets — flash crashes, sudden pumps — price can move faster than an API can report. A 20% buffer gives the bot enough room to:

1. Detect the danger zone
2. Submit the market order
3. Get it filled

...before the exchange's own liquidation engine activates. In calmer markets you'll rarely get close to it. In extreme conditions, it can be the difference between a controlled exit and a total margin wipe.

> **Tip:** On high-leverage setups (10x+), consider increasing the buffer to 30–40%. The higher your leverage, the closer your liquidation price is to your entry, and the faster price can close that gap.

### Sidebar

```text
🪢 Rekt Liq. Price    3105.42   (35.4% away)
🛡️ Liq. Limit (%)     20%       SAFE ✓

```

* **Rekt Liq. Price** — Your actual liquidation price from the exchange, with the current percentage distance shown in real time.
* **Liq. Limit** — Your configured buffer vs current distance. Changes to `⚠️ WARNING` when distance is getting close, and `🚨 TRIGGERED` the moment the guard fires.

---

## 🔹 11. Funding Rate Intelligence 💸

*The hidden tax of Futures trading — now fully visible and actively managed.*

### What is a Funding Rate? (For Beginners)

When you trade perpetual futures (the type of contract used on Hyperliquid, dYdX, Binance Futures, and Kraken Futures), you are **not buying or selling the underlying asset**. You are holding a synthetic contract that tracks the price of that asset indefinitely — it never expires.

This creates a structural problem: **the contract price can drift away from the real spot price**. To prevent this, exchanges invented the **funding rate** — a periodic payment between Long holders and Short holders that keeps the contract price anchored to spot.

**The mechanic is simple:**

* When the contract is trading **above spot** (demand is bullish, longs are dominant), **Longs pay Shorts**. This discourages excessive long positions and pulls the contract price back down toward spot.
* When the contract is trading **below spot** (demand is bearish, shorts are dominant), **Shorts pay Longs**. This discourages excessive short positions and pushes the contract price back up toward spot.

**This payment happens every 1–8 hours** (depending on the exchange) and is settled directly from your margin balance — no action required from you. It is automatic and unavoidable.

**A concrete example:**

> You open a $1,000 Long on ETH perpetuals on Binance Futures with 3x leverage. Your margin is $333.
> The current funding rate is +0.04% (positive — longs pay shorts).
> Every 8 hours, Binance deducts **0.04% × $1,000 notional = $0.40** from your margin and credits it to short holders.
> Over a single 24-hour day (3 settlement windows), you've paid **$1.20** just to hold the position — completely invisible if you're not watching.
> At high rates (e.g., +0.2% per 8h during a bull frenzy), holding $5,000 long notional for a week costs **$52.50 in funding alone**. That's real money silently bleeding from your margin.

**The annualized perspective:**
Funding rate impact becomes clear when annualized:

* Binance Futures: 3 settlements per day × 365 days = **1,095 per year**. A +0.04% 8h rate annualizes to **~43.8% APR**.
* Hyperliquid, dYdX, Kraken: 1 settlement per hour × 24 × 365 = **8,760 per year**. A +0.003% 1h rate annualizes to **~26.3% APR**.

This means the direction of the funding rate tells you something important about market sentiment **before it moves**:

* **High positive funding** = market is extremely long-heavy = crowded trade, potential for a sharp correction when longs get squeezed.
* **High negative funding** = market is extremely short-heavy = potential short squeeze on any bullish catalyst.

Experienced traders monitor funding as a **contrarian signal** and a **cost-of-carry alarm**.

---

### 🆕 Live Funding Rate Intelligence (v1.2.7)

Wick Magic Futures v1.2.7 adds **native, real-time Funding Rate monitoring** for all four supported exchanges. This is not a simple REST poll — the strategy connects via the same live WebSocket feeds already used for order book data, getting funding rate updates the moment they change:

| Exchange | Data Source | Settlement Frequency | What's Shown |
| --- | --- | --- | --- |
| **Hyperliquid** | WebSocket | Every 1 hour | Rate per 1h, Mark Price, Cumulative Funding since position open, Next settlement countdown |
| **dYdX v4** | WebSocket | Every 1 hour | Next funding rate prediction (oracle-derived), Oracle Price, Next settlement countdown |
| **Binance Futures** | REST polling (every 30s) | Every 8 hours | Rate per 8h, Mark Price, Next settlement countdown |
| **Kraken Futures** | WebSocket | Every 1 hour | Current rate + next period prediction, Mark Price, Next settlement countdown |

**Sidebar Tiles (per exchange)**

**For Hyperliquid**:

```text
📌 Mark Price         2174.35
💸 Funding Rate       -0.0028% / 1h  (red = negative, shorts pay longs)
📊 Cum. Funding       -$0.24 since open
⏰ Next Funding       45m 12s

```

**For dYdX v4**:

```text
📌 Oracle Price       78,360.00
💸 Funding Rate       -0.0034% / 1h
📊 Cum. Funding       -$0.12 since open
⏰ Next Funding       47m 35s

```

**For Binance Futures**:

```text
📌 Mark Price         87.43
💸 Funding Rate       +0.0047% / 8h   (green = positive, longs pay)
⏰ Next Funding       7h 40m

```

**For Kraken Futures**:

```text
📌 Mark Price         2,172.10
💸 Funding Rate       +0.0021% / 1h
🔮 Next Funding Est.  +0.0019% / 1h   (predicted next period rate)
⏰ Next Funding       3m 22s

```

**Funding Rate color coding:**

🟢 Green — Positive rate. Longs pay Shorts. Bullish market sentiment.
🔴 Red — Negative rate. Shorts pay Longs. Bearish market sentiment.

> **Note on Cumulative Funding:** Hyperliquid and dYdX v4 expose the cumulative funding paid/received **since the current position was opened**. This is displayed in the sidebar as `Cum. Funding` and shows whether funding has been a net positive or negative contributor to your position PnL. On Binance Futures, this value is not available via the API — so `Cum. Funding` is not shown for Binance.

---

### 🛡️ Funding Rate Guard

Knowing your funding rate is useful. **Acting on it automatically is powerful.**

The **Funding Rate Guard** is an entry blocker that prevents the bot from opening new positions or adding DCA when annualized funding exceeds a configurable threshold. It is designed to protect you from silently entering during periods of extreme funding cost — the kind that can quietly erase profit margins on every cycle.

**How the annualization works:**
The Guard compares rates on a standardized **annualized %** basis, regardless of settlement frequency:

* **Binance Futures:** `rate × 3 × 365 × 100` (8h × 3 per day × 365 days)
* **Hyperliquid, dYdX v4, Kraken Futures:** `rate × 24 × 365 × 100` (1h × 24 per day × 365 days)

This means a Binance rate of +0.04% and a Hyperliquid rate of +0.005%/h are both roughly equivalent to ~44% APR — and both would trigger a Guard set to 50%.

**What it blocks (by default):**

* All new OF entries (LONG and SHORT)
* All DCA/rebuy orders
* **Does NOT block exits, partial closes, or trailing stops.** You always close freely.

**Side-Aware Mode**
By default, the Funding Rate Guard blocks entries whenever the annualized rate exceeds your threshold — regardless of whether your side is paying or receiving. When you enable **Side-Aware Mode**, the guard becomes directional: it only blocks entries when YOUR side would be paying funding. If you are on the receiving side, the guard never fires — funding is your tailwind, not a cost.

| Funding Rate | Your Side | Side-Aware OFF | Side-Aware ON |
|---|---|---|---|
| +0.1% (longs pay) | LONG | ⛔ Blocked | ⛔ Blocked (you pay) |
| +0.1% (longs pay) | SHORT | ⛔ Blocked | ✅ Allowed (you receive) |
| -0.05% (shorts pay) | SHORT | ⛔ Blocked | ⛔ Blocked (you pay) |
| -0.05% (shorts pay) | LONG | ⛔ Blocked | ✅ Allowed (you receive) |

**New Entries Only**
When enabled, the Funding Rate Guard only blocks **brand-new position entries** (when you have no open position). DCA and re-entries into an existing position are allowed even if funding exceeds the threshold. This is useful if you want funding protection for your initial entry but still want to average into a trade that's already in progress.

**Sidebar when Guard is active:**

```text
🛡️ Funding Guard      ⚠️ BLOCKED (342% APR > 300% limit)

```

When funding is within limits:

```text
🛡️ Funding Guard      ✅ CLEAR (44% APR)

```

---

### 💡 How to Use Funding Rate Intelligence

**For position management:**
Watch the `💸 Funding Rate` tile during an open position. If funding suddenly spikes sharply positive while you're Long, the market is getting crowded on your side. Consider tightening your trailing stop or taking a partial exit before the inevitable squeeze.

**For entry decisions:**
Extremely positive funding (longs paying heavily) on an asset you want to Short means you'll be *receiving* funding payments for every hour you hold. Your carry is positive. That's an additional tailwind on top of your directional trade. Conversely, if funding is deeply negative and you want to go Short, you'll be *paying* funding every period. That's a headwind eating into your edge.

**For the Guard:**
Enable **Side-Aware Mode** if you want the guard to work with your direction instead of blindly blocking on any high rate. With Side-Aware ON, a Short trader during a high positive funding phase (longs paying shorts) will never be blocked — they're getting paid. Without Side-Aware, that same high rate blocks everyone regardless of direction.

Set **Max Annualized Funding Rate (%)** conservatively. A rate of 300% APR sounds extreme but in crypto it happens regularly during bull phases. The default suggestion is 300–500% depending on your risk tolerance. Below 200% is overly restrictive and will block too many good entries.

Enable **New Entries Only** if you want the guard to protect your initial entry but leave DCA untouched. This is ideal for traders who prefer funding awareness at the start of a new cycle but don't want it interfering once they're already managing a position.

**Quick reference:**

| Funding Rate | Direction | Long trader | Short trader |
|---|---|---|---|
| Positive (+) | Longs pay Shorts | ❌ You pay | ✅ You receive |
| Negative (-) | Shorts pay Longs | ✅ You receive | ❌ You pay |
| Near zero | Balanced | Neutral | Neutral |

**Exchange-specific tips:**
- **Hyperliquid**: Watch the Cum. Funding tile to see the real cost of carry since you opened. If it's eating into your theoretical PnL significantly, consider whether the trade thesis still holds.
- **dYdX v4**: The Next Funding Est. tile gives you a preview of the next settlement before it happens. If the prediction spikes, you can decide whether to exit before you get hit.
- **Binance**: The 8-hour settlement window means you have more time to react. Monitor the Next Funding countdown. If a settlement is minutes away and the rate is high, consider delaying a new entry until after it passes.
- **Kraken Futures**: The prediction tile is particularly valuable here. Kraken computes the next period's predicted rate from live order book imbalance, giving you a real-time forward indicator.

---

### Recommended Settings for Funding Rate Guard

| Market Condition | Suggested Max Annualized Funding Rate (%) |
|---|---|
| Conservative (low risk tolerance) | `200%` |
| Standard (most users) | `300%` |
| Aggressive (accept higher carry cost) | `500%` |
| Disabled (trust your OF signal) | `0` (set to 0 to disable) |

> **Pro tip:** During bull market phases, positive funding rates on major pairs routinely hit 200–400% annualized. If you run a short bias strategy (Institutional preset in SHORT mode), high positive funding is actually your friend because you receive funding — enable **Side-Aware Mode** and you won't be blocked. The Guard is most useful for Long-biased strategies where high positive funding silently erodes your edge, or for Short-biased strategies when funding goes deeply negative.

---

## 🔹 12. Institutional Trend Mode

*Let macro structure decide your direction. Then let OrderFlow decide your entry.*

**The Problem with Fixed Direction in AUTO Mode**
When Position Direction is set to `auto`, the bot decides each tick whether to go Long or Short based on a trend indicator. The default method uses SuperTrend on the main period — which works well on trending markets but can produce noisy directional flips on choppy pairs or when the period TF is too fast.

**Institutional Trend Mode** replaces the default auto-direction logic with a choice of three well-established macro bias methods, each configurable on any timeframe.

**Architecture: Direction vs Entry**
Institutional Trend Mode is a **direction selector only**. It answers one question: *should we be Long or Short right now?* It does not gate entries. Once the direction is set, the full OrderFlow signal pipeline (imbalance, S/R confluence, trap detection, absorption) still runs as normal and validates whether to actually fire an order.

This separation is deliberate and critical. **Trend = direction. OF = entry validator. These two layers never interfere with each other.**

**The Three Engines (Institutional Trend Engine)**

**`sma200`** *(default)*
Price above the 200-period SMA → LONG. Price below → SHORT. Clean, classic, and widely respected as a macro filter. No parameters to tune beyond the timeframe.

**`supertrend`**
Direction derived from the SuperTrend indicator on the selected timeframe (SuperTrend Period, SuperTrend Multiplier). SuperTrend adapts to volatility via ATR, making it more responsive than a fixed moving average on fast-moving pairs.

**`smacross`**
SMA100 vs SMA200 crossover. SMA100 > SMA200 → LONG. SMA100 < SMA200 → SHORT. The crossover is inherently slower than price-vs-SMA200, generating fewer direction flips and reducing noise on ranging pairs. Best suited for longer timeframes where you want maximum directional stability.

**Timeframe Selection (Institutional Trend Timeframe)**

All three engines can be pointed at any of three timeframes:

| Value | Description |
| --- | --- |
| `period` | Main candle period (e.g., 5m) |
| `long` | Long timeframe (e.g., 1h or 4h) |
| `daily` | Daily candles |

For most use cases, `long` gives a stable macro bias while execution stays fast on the short TF. `daily` gives the smoothest bias but may lag during fast trending moves.

**Fallback Behavior**
If the required indicator values are not yet computed (e.g., bot just started and hasn't collected enough candles for SMA200), the system gracefully falls back to the default SuperTrend direction from the main period. Direction locks in correctly once sufficient candle history has been collected — no manual intervention required.

---

## 🔹 13. Futures Margin & Compound Profits

### Native Exchange Telemetry & Funding

Unlike Spot bots that guess your capital, Wick Magic Futures reads raw websocket data from exchanges like Hyperliquid, dYdX v4, Binance Futures, and Kraken Futures. It knows your exact **Account Margin Usage**, **Unrealized PnL**, and **natively includes accumulated funding fees** directly into the ROE calculations. What you see is your true net profitability.

If your margin usage hits the danger zone (e.g., Liquidation Buffer: 20%), it hard-blocks all new entries to protect you from a margin call.

**Unified Accounts & Multi-Collateral**
Some exchanges allow you to use non-USD assets as margin for Futures positions. Wick Magic Futures natively supports this:

* **Hyperliquid (Unified Account):** If your account is set to Unified mode, your spot USDC balance is automatically available as collateral for perps positions. The strategy reads both your perps and spot balances via the Hyperliquid API, combining them into a single real available margin figure. No manual transfers needed — if you hold USDC in spot, it backs your futures trades.
* **Kraken Futures (Multi-Collateral):** Kraken Futures allows you to use BTC, ETH, EUR, and other assets as margin alongside USD. The strategy connects via a dedicated real-time WebSocket feed that reads your full multi-collateral balance across all assets. Your configured **Total Capital** acts as a per-pair allocation cap, while the WebSocket provides the real margin ceiling from your entire portfolio.
* **Binance Futures:** The strategy reads your native margin and position data directly from the Binance Futures API. Standard USDT-margined and USDC-margined contracts are fully supported.

> **Tip:** On exchanges where Gunbot cannot see your full collateral (e.g., non-USD assets on Kraken Futures, or spot USDC on Hyperliquid Unified Accounts), set your **Total Capital** to match your total available margin in USD. The strategy uses it as the effective margin budget.

### Compound Profits

Trade Size adjusts automatically based on cumulative realized PnL. Every time a full cycle closes, the bot calculates the net PnL and adds it to a running offset.

* **Winning cycle:** Next entries are slightly larger.
* **Losing cycle:** Next entries shrink to protect capital.
(Hard floor at 25% of base size to prevent infinite shrinking).

---

## 🔹 14. Dump & Pump Protection & Elastic Exits

In Futures, holding a losing position while the market trends against you is fatal — whether you're **long in a dump** or **short in a pump**.

**Dump & Pump Protection:** During a sharp, confirmed adverse orderbook move, the bot's normal Minimum Profit requirement might be too high for upper DCA lots to exit in time. Dump & Pump Protection temporarily **lowers the profit floor** (e.g., to 0.4%) when the OB confirms adverse pressure — **bearish for LONG positions (dump), bullish for SHORT positions (pump)** — allowing the bot to rapid-fire partial exits and reduce your exposure quickly before the move deepens.

> ⚠️ Hard rule: the position must still be **in profit** (price on the correct side of Break Even) before any exit fires. Closing at a loss is never permitted — unless **Allow Partial Close Below Entry** is enabled, which permits per-fill profitable reductions below the position average. See [Below-Entry Partial Close](#-below-entry-partial-close-soft-reduction).

**Elastic Trailing:** The bot dynamically widens its trailing stop during high volatility (so you don't get wicked out) and tightens it when the market slows down (to lock in gains). Applies to both LONG and SHORT exits.

---

## 📉 Below-Entry Partial Close (Soft Reduction)

> **Toggle:** `Allow Partial Close Below Entry`  
> **Default:** `false` (disabled)  
> **Works with:** Both OrderFlow and Legacy partial close modes, LONG and SHORT

By default, Wick Magic Futures only allows partial closes when the close price is **above your position's average entry** (for LONG) or **below it** (for SHORT). This is a safety measure — the exchange calculates PnL against your average entry, so closing below it registers as a realized loss.

When **Allow Partial Close Below Entry** is enabled, the strategy looks at your position **fill by fill** instead of as one averaged block. If you bought some units at a lower price and the current price is above *that specific fill*, the strategy will close those units — even though the current price is still below your overall position average.

**⚠️ This is technically a soft reduction, not a "profit".** The exchange will report it as a realized loss because it measures against your position's average entry price. However, the specific units being closed were acquired at a lower price — so from a per-fill perspective, they're profitable.

### Why use it

In ranging or slowly recovering markets, your position can get stuck:

- You DCA'd multiple entries at different prices
- Price recovered above your cheapest fills but hasn't reached your average entry yet
- Standard partial close is blocked — nothing happens, position is frozen
- You wait... and the market drops again

With this toggle, the cheap fills get closed while they're profitable. This **reduces your exposure** and **captures the volatility** instead of sitting frozen waiting for a full recovery that may never come.

### Example — LONG position

```
Fill 1:  Buy 1.0 ETH @ $2,500
Fill 2:  Buy 1.0 ETH @ $2,000
Fill 3:  Buy 1.0 ETH @ $1,800

Position:  3.0 ETH
Avg Entry: $2,100 (exchange-reported)
Current price: $1,850
```

| | Standard mode | Below-Entry mode |
|---|---|---|
| Can close Fill 3? | ❌ No — $1,850 < $2,100 avg entry | ✅ Yes — $1,850 > $1,800 fill price |
| Exchange reports | — | Realized loss (~$250) |
| Actual fill P&L | — | +$50 on that 1.0 ETH |
| Remaining position | 3.0 ETH @ $2,100 | 2.0 ETH (expensive fills remain) |

After the partial close, the exchange recalculates your average entry **downward** (the "loss" gets absorbed). But the **true cost** of your remaining 2.0 ETH is actually **higher** — you sold the cheap fill and kept the expensive ones. This is why the strategy tracks True Cost Basis.

### Example — SHORT position

```
Fill 1:  Short 1.0 SOL @ $150
Fill 2:  Short 1.0 SOL @ $180
Fill 3:  Short 1.0 SOL @ $200

Position:  3.0 SOL SHORT
Avg Entry: $176.67 (exchange-reported)
Current price: $195
```

| | Standard mode | Below-Entry mode |
|---|---|---|
| Can close Fill 3? | ❌ No — $195 > $176.67 avg entry | ✅ Yes — $195 < $200 fill price |
| Exchange reports | — | Realized loss |
| Actual fill P&L | — | +$5 on that 1.0 SOL |
| Effect | Position frozen | Exposure reduced |

### True Cost Basis (TCB)

This is the critical safety mechanism. Without it, below-entry partials would corrupt your GAIN target.

**The problem:** after closing cheap fills below entry, the exchange's average entry drifts lower (for LONG). If your GAIN target still uses the exchange entry, it would trigger too early — and the final close could be a net loss because you kept only the expensive fills.

**The solution:** Wick Magic tracks the **true weighted average cost** of your remaining fills. When TCB is active, the GAIN target is calculated from TCB instead of the exchange entry.

```
Before partial:  TCB = $2,100 (same as exchange entry)
After closing Fill 3 (1.0 @ $1,800):
  Remaining fills = Fill 1 ($2,500) + Fill 2 ($2,000)
  TCB = ($2,500 + $2,000) / 2.0 = $2,250  ← real cost now
  Exchange entry = ~$1,971  ← distorted, NOT used for GAIN
```

### TCB Persistence

TCB is stored in two places for crash safety:

- **Store** (`tcbTotalCost`, `tcbTotalQty`) — hot state, updated every tick
- **Config** (`OF_TRUE_COST_BASIS`) — written to `config.js` on every below-entry partial close. Survives restarts, store wipes, and JSON reconstruction from the exchange

If you need to recover from corrupted data, set `OF_TRUE_COST_BASIS` in your pair overrides to the correct value. Set it to `0` to force re-seeding from your current orders array on the next tick. On full position close, TCB resets to 0 automatically.

### What You See

**Sidebar:** a `📊 True Cost` tile showing your TCB value and drift % vs exchange entry. Green when tracking normally, red when TCB > exchange entry (expected — remaining fills are the expensive ones).

**Chart:** below-entry partial close targets appear as **orange lines** (instead of normal pink), making it easy to distinguish which partials are below your average entry.

### ⚠️ Important warnings

1. **The exchange will show realized losses.** This is expected. The exchange doesn't know your per-fill intent — it sees every close below avg entry as a loss. Your net P&L across the full position cycle should be the same or better.
2. **Your GAIN target shifts up.** After removing cheap fills, the remaining position is more expensive. TCB handles this automatically, but be aware that your full-close target will be higher than what the exchange entry suggests.
3. **Works best with multiple DCA entries at spread prices.** If all your fills are clustered at similar prices, there won't be any "cheap fills" to close. The feature shines with wide DCA spreads.
4. **FUTURES_MIN_CLOSE_PROFIT still applies.** The minimum ROE% floor is calculated against the fill price (not avg entry) when below-entry mode is active.

### When to enable this

✅ Ranging markets where price bounces between your DCA levels but never fully recovers. Wide DCA spreads where bottom fills keep getting profitable and then dropping again. Pairs with high volatility but no sustained trend — capture the chop instead of waiting.

❌ Strong trending markets where a full recovery is likely — standard mode is more efficient. Positions with only 1-2 entries at similar prices — nothing to "reduce" profitably.

### Interaction with other features

- **Compound Profits**: TCB and Compound are independent trackers. Both can run simultaneously.
- **Dump & Pump Protection**: Below-entry partials still respect `OF_PARTIAL_DUMP_PROTECT` — the lowered profit floor applies to the fill-based ROE calculation.
- **Kill Position**: Uses unrealized PnL from the exchange, not TCB. If KILL_POSITION triggers, it closes the full position regardless of TCB state.
- **Same Ratio**: Compatible. Only the price gate changes, not the sizing logic.

---

## 🔹 15. Auto-ATR DCA (Dynamic Spread)

Instead of guessing fixed percentage drops for your DCA (Re-Entry DCA Spread), Wick Magic Futures can look at the **Average True Range (ATR)**.

* High volatility? The DCA spread widens automatically to catch deeper wicks.
* Low volatility? The DCA spread tightens to keep accumulating in a tight range.

You can map this to the Period TF, Long TF, or Daily TF.

---

## 🔹 16. OrderFlow Presets

Wick Magic Futures ships with **10 ready-to-trade profiles** plus a full manual mode. Each preset auto-configures timeframes, profit targets, trailing behavior, session gating, and all internal tuning — while always preserving your personal settings: **Total Capital**, **Trade Size**, **Leverage**, **Futures Exchange**, **Position Direction**, **Entry Cooldown**, **Compound Profits**, **Institutional Absorption**, and **Max Annualized Funding Rate (%)** are never overwritten by a preset switch.

### How Presets Work

Select a **Strategy Preset** from the Gunbot editor dropdown. The engine applies the preset once, then locks — any manual change you make afterward is yours and won't be overwritten unless you switch to a different preset. Switching presets mid-run is fully supported: the engine detects the change on the next tick and re-applies the new profile cleanly.

---

### Standard Profiles

### `orderflow_scalp`

Fast entries on a 5-minute candle with 4-hour trend context. Designed for 24/7 operation on liquid pairs.

* No Institution Hours gating — runs around the clock.
* Moderate profit targets and trailing range.
* S/R Confluence and Smart S/R Bias enabled.

### `orderflow_scalp_fast`

Same 5-minute entry candle as Scalp, but with **1-hour trend context** instead of 4 hours. Reacts faster to trend changes with tighter profit targets and trailing stops. Ideal for volatile markets or pairs with fast-moving structure.

### `orderflow_institutional`

Higher-conviction entries on a 15-minute candle with 4-hour trend context. Built for quality over quantity.

* **Institution Hours active** (7 AM – 9 PM UTC, scout mode outside hours).
* Institutional Absorption detection ON.
* Wider trailing range and higher profit targets.
* Dump/Pump Protection enabled.

### `orderflow_institutional_fast`

Same 15-minute entry candle as Institutional, but with **1-hour trend context**. Combines the session-aware conviction of Institutional with faster trend reactivity and tighter profit targets.

---

### Kraken Futures Profiles

### `orderflow_krakenfutures_full`

For Kraken Futures traders with **multi-collateral portfolios** (USD, BTC, ETH, EUR, etc.). The strategy dynamically reads your true available margin in real time via a dedicated WebSocket connection and uses it as your trading budget. Your configured **Total Capital** acts as a per-pair allocation cap, while the live WebSocket data provides the real margin ceiling.

Uses 5-minute entry candles with 4-hour trend context (same cadence as Scalp).

### `orderflow_krakenfutures_full_fast`

Same as the Full Collateral profile above but with **1-hour trend context**. Tighter targets and faster trailing for more reactive trading on Kraken Futures multi-collateral accounts.

### `orderflow_krakenfutures_btc`

Built for Kraken Futures traders who hold and think in BTC. Set your **Total Capital** and **Trade Size** directly in Bitcoin — the strategy auto-converts everything to USD in the background using the live BTC price. All sidebar metrics (Total Capital, Trading Limit, Used Capital, PnL) display natively in BTC.

Uses 5-minute entry candles with 4-hour trend context.

### `orderflow_krakenfutures_btc_fast`

Same BTC-native experience as above but with **1-hour trend context** and tighter parameters. For Kraken Futures BTC holders who want faster cycle times.

---

### Binance Futures Profiles

### `orderflow_binancefutures_btc`

For Binance Futures traders who want to **set their capital and trade size in BTC** instead of USD. The strategy auto-converts to USD notional using the live BTC price — all position sizing, capital guards, and compound profit tracking operate in real USD behind the scenes, while you configure and monitor everything in BTC.

This is particularly useful for trading pairs like USDT-BTC where the minimum notional (0.001 BTC) makes USD-based sizing impractical at high BTC prices.

Uses 5-minute entry candles with 4-hour trend context (same cadence as Scalp).

### `orderflow_binancefutures_btc_fast`

Same BTC-native configuration as above but with **1-hour trend context**. Tighter profit targets and faster trailing for quicker cycle times on Binance Futures.

---

### Manual Mode

### `customized`

Full manual control. All parameters are user-defined — no preset values are applied. The engine only fills missing keys and normalizes types but never overwrites your settings. Use this when you want to build your own configuration from the ground up.

---

### Preset Comparison Table

The table below shows the **key differences** between each profile. All other settings (OrderFlow core, S/R, GA calibration, etc.) are shared across profiles.

| Setting | Scalp | Scalp Fast | Institutional | Inst. Fast | KF Full | KF Full Fast | KF BTC | KF BTC Fast | BF BTC | BF BTC Fast |
|---|---|---|---|---|---|---|---|---|---|---|
| **Candle Period (min)** | 5 | 5 | 15 | 15 | 5 | 5 | 5 | 5 | 5 | 5 |
| **Candle Period (long)** | 240 | 60 | 240 | 60 | 60 | 60 | 60 | 60 | 240 | 60 |
| **Target ROE (%)** | 1.2 | 1.2 | 2.0 | 1.4 | 0.9 | 0.9 | 0.9 | 0.9 | 1.2 | 0.9 |
| **Minimum Profit (%)** | 0.9 | 0.9 | 1.2 | 0.8 | 0.6 | 0.6 | 0.6 | 0.6 | 0.9 | 0.6 |
| **Trailing (%)** | 0.5 | 0.5 | 0.75 | 0.5 | 0.35 | 0.35 | 0.35 | 0.35 | 0.5 | 0.35 |
| **Min Trail (%)** | 0.3 | 0.3 | 0.4 | 0.3 | 0.2 | 0.2 | 0.2 | 0.2 | 0.3 | 0.2 |
| **Max Trail (%)** | 2 | 2 | 3 | 2 | 1.5 | 1.5 | 1.5 | 1.5 | 2 | 1.5 |
| **DCA Drop (%)** | 1 | 1 | 1.5 | 1 | 0.75 | 0.75 | 0.75 | 0.75 | 1 | 0.75 |
| **Entry Cooldown** | 2 | 2 | 5 | 4 | 2 | 2 | 2 | 2 | 2 | 2 |
| **Institution Hours** | OFF | OFF | ON | ON | OFF | OFF | OFF | OFF | OFF | OFF |
| **Dump/Pump Protect** | OFF | OFF | ON | ON | OFF | OFF | OFF | OFF | OFF | OFF |
| **S/R Proximity (%)** | 1.0 | 1.0 | 1.0 | 0.8 | 0.8 | 0.8 | 0.8 | 0.8 | 1.0 | 0.8 |

**How to read this table:**
* **Standard vs Fast**: Fast variants use a 1-hour long candle period instead of 4 hours. This makes the trend filter more reactive, and targets/trailing are tightened accordingly for shorter hold times.
* **Scalp vs Institutional**: Institutional uses 15-minute candles (fewer but higher-conviction entries), wider targets, and enforces Institution Hours. Scalp runs 24/7 with 5-minute candles and tighter targets.
* **KF / BF variants**: Same core logic as Fast profiles, with automatic collateral handling for their respective exchanges. Set your Total Capital and Trade Size in BTC — the strategy converts everything internally.

---

## 🔹 17. Editor Settings Reference

Below are the key parameters configurable directly from the Gunbot GUI editor, organized by section. Parameter names shown are the **GUI labels** users see in the editor.

### Preset Configuration

| Parameter | Default | Description |
| --- | --- | --- |
| **Strategy Preset** | `orderflow_scalp` | Choose a preset profile. Options: `orderflow_scalp`, `orderflow_scalp_fast`, `orderflow_institutional`, `orderflow_institutional_fast`, `orderflow_krakenfutures_btc`, `orderflow_krakenfutures_btc_fast`, `orderflow_krakenfutures_full`, `orderflow_krakenfutures_full_fast`, `orderflow_binancefutures_btc`, `orderflow_binancefutures_btc_fast`, or `customized`. See Section 16 for a full comparison of what each preset changes. |
| **Total Capital (USD)** | `1000` | Total margin capital this pair can use. Preserved across presets. For Kraken Futures with multi-collateral: set this to your total available margin value in USD. For BTC collateral presets (KF or BF): set this value in BTC — the strategy auto-converts to USD using the live BTC price. |
| **Trade Size Per Entry (USD notional)** | `25` | Notional amount used for each individual entry. Actual margin used depends on leverage. For BTC collateral presets: set this value in BTC (e.g. 0.001) — auto-converted to USD notional. |
| **Compound Profits** | `false` | Automatically adjusts Trade Size after each completed full close by adding/subtracting realized PnL. Hard floor at 25% of base value. |
| **Warmup Sampling** | `true` | Blocks new Orderflow entries until enough GA calibration samples are collected (default 120–180 samples). |
| **Compound Profits (Classic TA)** | `false` | Same as Compound Profits but for the classic TA entry branch. |
| **Futures Exchange** | `hyperliquid` | Select the futures exchange: `hyperliquid`, `dydx`, `binancefutures`, `krakenfutures`. Tells the strategy which adapter to use for reading native position data. |

### Essential Strategy Settings

| Parameter | Default | Description |
| --- | --- | --- |
| **Position Direction** | `auto` | Choose `long`, `short`, or `auto` (algorithm decides based on trend). |
| **Candle Period (min)** | `5` | Timeframe of each candle in minutes. Options: 1, 3, 5, 15, 30, 60. |
| **Candle Period (long)** | `240` | Long timeframe for trend filter and S/R levels. Options: 60, 120, 240, 360, 480, 720, 1440. |
| **Candle History Length** | `400` | Number of candles to fetch for indicator calculations. |
| **Ignore Trades Before** | `0` | Timestamp to avoid processing historical trading data. |
| **Open Positions Enabled** | `true` | Allow or disable opening new positions and increasing existing ones. |
| **Close Positions Enabled** | `true` | Allow or disable closing positions. |
| **Stop After Full Close** | `false` | After the position is fully closed, don't open a new one. |
| **Entry Cooldown (candles)** | `3` | Minimum number of candles to wait between any two entries. |

### Trend & Direction

| Parameter | Default | Description |
| --- | --- | --- |
| **⭐ Enable Institutional Trend** | `true` | Replaces the default AUTO direction logic with a configurable macro trend method. Only applies when Position Direction is `auto`. Does not gate entries — direction only. |
| **Institutional Trend Engine** | `supertrend` | Macro bias method: `sma200` (price vs SMA200), `smacross` (SMA100 vs SMA200 cross), `supertrend` (SuperTrend direction). |
| **Institutional Trend Timeframe** | `long` | Timeframe for the Institutional Trend indicator: `period` (main TF), `long` (HTF), `daily`. |
| **Enable Trend Shield (Entry Gate)** | `false` | Block entries when trend signals are against your position direction. Independent of Institutional Trend — this is an entry gate, not a direction selector. |
| **Trend Shield Engine** | `supertrend` | Engine for Trend Shield: `supertrend`, `sma200`, `confirm`. |
| **SuperTrend Period** | `10` | ATR period used by SuperTrend. |
| **SuperTrend Multiplier** | `3` | ATR multiplier used by SuperTrend. |
| **Trend Debug Logs** | `false` | Enable verbose trend diagnostics in console. |

### Futures Safety

| Parameter | Default | Description |
| --- | --- | --- |
| **Leverage** | `3` | Leverage multiplier. Set to 0 for dynamic leverage (exchanges like Kraken Futures that set leverage automatically). Range: 1–50. |
| **Enable Liquidation Guard** | `true` | Emergency-close the entire position if price nears the liquidation price. |
| **Liquidation Buffer (%)** | `20` | Trigger emergency close if price is within this percentage of the liquidation price. Range: 2–50%. |
| **Keep Minimum Position** | `false` | Always keep a minimum position size after partial closes. |
| **Keep Min Value (USD notional)** | `0` | Minimum position notional to preserve. |
| **Enable Kill Position (uPnL Stop)** | `false` | Master switch for the USD-based emergency exit. Closes the entire position at market if uPnL hits the configured threshold. Runs before all other logic on every tick. |
| **Enable Trailing Stop** | `false` | Activates trailing mode. Tracks the peak uPnL reached during the position. Fires if uPnL drops more than Trail Distance below that peak. Hard stop remains active in parallel. |
| **Kill Position Loss (USD)** | `100` | Maximum tolerable loss in USD. Closes position immediately if unrealized PnL (funding-adjusted, from exchange) drops to or below -$X. Range: 10–1000. |
| **Trail Distance (USD)** | `50` | Trail distance in USD from peak uPnL. Example: peak was +$200, trail is $50 → kill fires at +$150 uPnL. Range: 5–500. |
| **Kill Position Take Profit (USD)** | `0` | Flat USD take profit target. If unrealized PnL reaches +$X the position closes immediately. Set to 0 to disable. Independent of ROE trailing — ideal for small entries where a fixed dollar target makes more sense than a percentage. Range: 0–1000. |

### Smart Capital Management

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable Hard Stop** | `false` | Closes a portion of the position once capital usage threshold is reached. |
| **Usage Threshold (%)** | `90` | Percent of capital used to trigger reduction. Options: 65, 70, 75, 80, 85, 90. |
| **Hard Reduction (%)** | `20` | Portion of position to close when triggered. Options: 20, 30, 40, 50, 70, 80, 90. |

> **How Smart Capital measures usage:** Capital usage is calculated based on your **actual margin committed** to the position — not the full notional value. For example, with 5x leverage and a $200 notional position, your real margin is $40. Smart Capital compares that $40 against your configured Total Capital to determine your usage percentage. This ensures the threshold fires accurately regardless of your leverage level.

### Orderflow Mode

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable Orderflow Mode** | `true` | Master switch for live OB imbalance and tape analysis. |
| **Signal Pair** | *(empty)* | Pair to use as the OrderFlow signal source instead of the trading pair. Example: `USDT-ETH`. Must be actively cycling in Gunbot on the configured Signal Exchange. Leave empty to use the trading pair. |
| **Signal Exchange** | *(empty)* | Exchange to read Signal Pair data from. Example: `binance`. Leave empty to use the same exchange as the trading pair. Only has effect when Signal Pair is also set. |
| **Allow Orderflow Re-Entry** | `true` | Allow orderflow to increase position (DCA) if signal confirms. |
| **Re-Entry DCA Spread (%)** | `0.5` | Required % move against position before DCA allowed. Range: 0–3%. |
| **Re-Entry DCA: Auto Spread** | `true` | Compute DCA spread automatically from ATR instead of fixed percentage. |
| **Re-Entry DCA: Auto Spread Timeframe** | `long` | Timeframe for automatic DCA spread: `period`, `long`, `daily`. |
| **Re-Entry Scale Spread (%)** | `0` | Allow scaling up into profit (0 to disable). Range: 0–3%. |
| **Re-Entry Scale: Auto Spread** | `false` | Compute scaling spread automatically from ATR. |
| **Re-Entry Scale: Auto Spread Timeframe** | `long` | Timeframe for automatic scale spread: `period`, `long`, `daily`. |
| **Scale Only After Partial Close** | `true` | Scaling blocked until at least one partial close has occurred. |
| **Re-Entry: Ignore OF Side** | `false` | Allow re-entry based on price level even if OF disagrees. |
| **Exit: Ignore OF Side** | `false` | Allow exits based on profit targets even if OF disagrees. |
| **Exit: Extreme Bull Block** | `0.9` | Block long exit if bullish imbalance is too strong (prevents selling into a rally). Range: 0.5–1.0. |
| **Exit: Extreme Bear Block** | `-0.9` | Block short exit if bearish imbalance is too strong (prevents covering into a dump). Range: -1.0 to -0.5. |
| **Iceberg Confirmation** | `false` | Require hidden large orders near best price for entry. |
| **PTH Tape Confirmation** | `true` | Use Public Trade History as second-layer confirmation. |
| **Institutional Absorption** | `true` | Detect institutional absorption patterns. |
| **Orderbook Depth** | `10` | Number of orderbook levels to scan for imbalance calculation. Range: 1–30. |
| **Imbalance Threshold (raw)** | `0.45` | Raw imbalance threshold before GA calibration kicks in. Range: 0–1. |
| **Bullish Imbalance Fallback** | `0.45` | Fallback buy threshold when GA is not yet calibrated. Range: 0–1. |
| **Bearish Imbalance Fallback** | `-0.45` | Fallback sell threshold when GA is not yet calibrated. Range: -1 to 0. |
| **Iceberg Detection (%)** | `0.005` | Minimum hidden volume ratio to flag iceberg. |
| **Iceberg Near Price (%)** | `0.15` | How close to best price an iceberg must be to count. Range: 0–0.5%. |
| **Iceberg Min Levels** | `1` | Minimum orderbook levels with iceberg activity. Range: 1–10. |
| **Dump/Pump Protection** | `false` | Temporarily lower profit floor during confirmed adverse OB pressure for faster partial exits. |
| **Dump Protect Min Profit (%)** | `0.4` | Lowered profit floor during dump/pump events. Range: 0–1%. |
| **Regime ATR Delta (%)** | `0.5` | ATR change threshold to trigger GA recalibration. Range: 0–5%. |
| **Re-Entry ATR Multiplier** | `1.0` | Multiplier applied to the ATR-based auto DCA spread. Higher = wider spacing between re-entries. Only active when Auto Spread is enabled. Range: 0.1–5. |
| **Exit: Ignore All OF Conditions** | `false` | Completely bypass all orderflow exit conditions. Exits fire on profit target and trailing only. Use with caution. |
| **Partial Close: TA Exit Signal** | `true` | Allow partial closes triggered by HA color flip (TA signal) in addition to orderflow conditions. |
| **Min Position Size to Close (USD)** | `5` | Minimum position notional in USD before any close is attempted. Prevents trying to close dust positions too small to execute. Set to 0 to disable. |

### Smart S/R Bias

| Parameter | Default | Description |
| --- | --- | --- |
| **S/R Confluence** | `true` | Enable multi-timeframe Support/Resistance confluence detection. |
| **S/R Proximity (%)** | `1` | How close price must be to an S/R level to consider entry valid. Range: 0.1–5%. |
| **S/R Confluence Zone (%)** | `1` | Max gap between timeframe pivots to merge into one zone. Range: 0.1–5%. |
| **Enable Smart S/R Bias** | `true` | Restricts new entries based on proximity to S/R zones. Near support → only LONG. Near resistance → only SHORT. Requires S/R Confluence enabled. |
| **Break Confirmation Candles** | `3` | Number of candles that must close on the other side of an S/R zone to confirm a breakout or breakdown. Options: 1–10. |
| **Max Re-Entries per S/R Level** | `2` | Maximum re-entries allowed at the same S/R level before waiting for the next distinct zone. Range: 0–10. |

### Institution Hours (Orderflow)

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable Institution Hours** | `false` | Master switch for session gating. |
| **Dead Zone Mode** | `scout` | Off-hours behavior: `scout` (raised thresholds) or `block` (no entries). |
| **Weekend Mode** | `scout` | Weekend behavior: `scout`, `block`, or `full` (normal trading). |
| **Session Start (UTC hour)** | `7` | Hour when full-power trading begins. Options: 0–23. |
| **Session End (UTC hour)** | `21` | Hour when full-power trading ends. Options: 0–23. |
| **Scout: OF Threshold Multiplier** | `1.4` | Entry threshold multiplier in scout mode. Range: 1.0–2.5. |
| **Scout: PTH Delta Floor** | `0.62` | Minimum delta ratio required by PTH in scout mode. Range: 0.5–0.95. |
| **Scout: Require Absorption** | `true` | Require institutional absorption detection in scout mode. |
| **Scout: Require Volume** | `true` | Require above-average volume in scout mode. |
| **Scout: Volume Multiplier** | `1.3` | Volume must be at least this × average in dead zone. Range: 1.0–3.0. |

### Profit Taking Settings

| Parameter | Default | Description |
| --- | --- | --- |
| **Profit Target Mode** | `price` | How profit targets are calculated: `price` (percentage from break-even) or `roe` (return on equity / margin). |
| **Minimum Profit to Exit (%)** | `0.9` | Minimum profit % required before any full close is allowed. |
| **Target ROE (%)** | `1.2` | Profit target before taking profit. Trailing activates here. |
| **Partial Close Min ROE (%)** | `0.7` | Minimum profit % required to allow a partial position close. |
| **Require HA Color Flip to Exit** | `false` | Require Heikin Ashi candle color change confirmation before full close. |
| **Enable Partial Close** | `true` | Allow reducing position size before full exit when in profit. |
| **Same Ratio for Closes** | `true` | Partial closes use the same size ratio as entries. |
| **Legacy Fast Partial Close** | `true` | Use the fast legacy partial close path (lot-by-lot immediate dispatch). |
| **Partial Close: Orderflow Gate** | `false` | Decouple partial sell method from entry method — use OF exit gate for partial closes even when entries are classic TA. |
| **Allow Partial Close Below Entry** | `false` | Close individual fills that are profitable even when price is below avg entry. Tracks True Cost Basis (TCB) automatically. See [Below-Entry Partial Close](#-below-entry-partial-close-soft-reduction). |
| **True Cost Basis (auto)** | `0` | Auto-maintained weighted average cost of remaining fills after below-entry partials. Set to `0` to force re-seed from orders. Do not modify unless recovering from data loss. |

### Limit Orders

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable Limit Orders** | `false` | Place limit orders with smart orderbook placement instead of market orders. |
| **OL Max Price Drift (%)** | `0.1` | Maximum allowed price move before cancelling and re-placing limit order. Range: 0.05–2%. |
| **OL Orderbook Depth** | `10` | Number of top orderbook levels to consider for placement. |
| **OL Time To Live (seconds)** | `15` | Max seconds a limit order stays open before re-evaluation. |
| **OL Price Offset** | `0` | Price offset from best level. Range: 0–2. |
| **OL Close Slippage %** | `0` | Allowed slippage on close orders. Range: 0–0.5%. |


### Trailing Settings

| Parameter | Default | Description |
| --- | --- | --- |
| **Trailing (%)** | `0.5` | Dynamic trailing stop that captures best achievable profit. Range: 0–3%. |
| **Min Dynamic Trail (%)** | `0.3` | Minimum trailing distance. Range: 0–10%. |
| **Max Dynamic Trail (%)** | `2` | Maximum trailing distance. Range: 0.2–5%. |
| **Minimal Trailing** | `0.2` | Tighter trailing % for high-volatility situations. Range: 0.05–1%. |
| **Enable Elastic Trailing** | `true` | Recalculate trailing stop based on volatility, volume and other metrics. |
| **Detailed Trailing Logs** | `false` | Enable verbose trailing diagnostics in console. |

### Fear & Greed Index (FGI) Settings

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable Fear & Greed Index** | `false` | Use FGI to block entries in extreme sentiment. |
| **FGI Minimum Entry** | `40` | Below this value, new entries are blocked. |
| **FGI Tight Exit** | `80` | Above this value, tight trailing/exit mode is enabled. |
| **FGI Cache (minutes)** | `1340` | Cache duration for FGI API calls. |
| **FGI Strict Mode** | `false` | Apply FGI gating to DCA re-entries as well as new entries. |

### Coinmarketcap Fear & Greed Index

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable CMC Fear & Greed** | `false` | Activates CoinMarketCap FGI module (Requires API key). |
| **Use CMC Scale Trading Limit** | `false` | Dynamically scale Trade Size Per Entry using CMC FGI. |
| **CoinMarketCap API Key** | *(empty)* | Your CMC API key. |
| **CMC Minimum Entry** | `40` | Below this value, new entries are blocked. |
| **CMC Cache (minutes)** | `1440` | Cache duration for CMC API calls. |
| **CMC Strict Mode** | `false` | Apply CMC gating to DCA re-entries. |
| **CMC Debug Logs** | `false` | Enable verbose CMC diagnostics. |

### FGI Mimix Scaling Profile

| Parameter | Default | Description |
| --- | --- | --- |
| **Use FGI Mimix Profile** | `false` | Dynamically scales Trade Size Per Entry by sentiment: bigger entries in fear, smaller in greed. |
| **Multiplier: Extreme Fear (0-24)** | `1.25` | Capital multiplier during Extreme Fear. Range: 0.1–5. |
| **Multiplier: Fear (25-44)** | `1.1` | Capital multiplier during Fear. Range: 0.1–5. |
| **Multiplier: Neutral (45-54)** | `1.0` | Capital multiplier during Neutral sentiment. Range: 0.1–5. |
| **Multiplier: Greed (55-74)** | `0.8` | Capital multiplier during Greed. Range: 0.1–5. |
| **Multiplier: Extreme Greed (75-100)** | `0.6` | Capital multiplier during Extreme Greed. Range: 0.1–5. |

### Funding

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable Funding Rate Guard** | `false` | Blocks all new entries and DCA when the annualized funding rate exceeds the configured maximum. Active on Binance Futures, Kraken Futures, Hyperliquid, and dYdX v4. Exits, partial closes, and trailing stops are never blocked. |
| **Max Annualized Funding Rate (%)** | `300` | Maximum annualized funding rate allowed before entries are blocked. Example: 300 blocks entries when the annualized rate exceeds 300% per year. Standard rates during calm markets are 10–50% annualized. During bull frenzies rates can spike to 200–500%+. Recommended range: 200–500. This value is preserved across preset changes. |
| **Side-Aware Mode** | `false` | When enabled, the guard only blocks entries when YOUR side is paying funding. If you are on the receiving side (e.g. Short during high positive funding), the guard never fires. When disabled, the guard blocks any entry when annualized funding exceeds the threshold regardless of direction. |
| **New Entries Only** | `false` | When enabled, the funding guard only blocks brand-new position entries (no open position). DCA and re-entries into an existing position are allowed even if funding exceeds the threshold. Useful if you want funding protection for fresh entries but still want to average into an existing trade. |

### TA Indicators

| Parameter | Default | Description |
| --- | --- | --- |
| **RSI Length** | `14` | RSI indicator period. |
| **RSI Entry Level (oversold)** | `30` | RSI level considered oversold for entries. |
| **RSI Exit Level (overbought)** | `70` | RSI level considered overbought for exits. |
| **Ultra RSI Entry Level (oversold)** | `33` | Relaxed oversold level for Ultra Aggressive mode. Range: 20–50. |
| **Ultra RSI Exit Level (overbought)** | `71` | Relaxed overbought level for Ultra Aggressive mode. Range: 60–90. |
| **Low Volume Threshold** | `0.2` | Volume ratio below which the market is considered quiet. Range: 0–2. |
| **High Volume Threshold** | `2` | Volume ratio above which volume is considered unusually high. Range: 0–3. |
| **Volume Lookback Period** | `50` | Number of candles to compute average volume. |
| **Wick Ratio Index Threshold** | `0.2` | Minimum wick-to-body ratio for wick-based entries. Range: 0.1–1. |
| **Bollinger Bands STD** | `1.25` | Standard deviation multiplier for Bollinger Bands. |
| **SMA Period** | `50` | Simple Moving Average period. |
| **DCA Spread (%)** | `0.5` | Spread for TA-mode DCA entries. Range: 0.1–3%. |

### TA Entry Modes

| Parameter | Default | Description |
| --- | --- | --- |
| **Aggressive Mode** | `false` | Enable aggressive entry logic (relaxed conditions). |
| **Ultra Aggressive Mode** | `false` | Enable ultra aggressive entry logic (most relaxed). |
| **Enable Breakout Entry** | `false` | Detect and trade price breakouts. |
| **Breakout Without Trend Gate** | `false` | Allow breakout entries even without a confirmed trend. |
| **Stack Breakouts After Partial** | `false` | Allow additional breakout entries after a partial close. |
| **Enable Pullback Entry (Trend)** | `false` | Entry on trend pullbacks with confirmed trend. |
| **Enable Pullback Entry (No Trend)** | `false` | Entry on pullbacks without requiring a confirmed trend. |
| **Use Heikin Ashi Entry** | `true` | Use Heikin Ashi candle signals for TA entries. |
| **Use Heikin Ashi 2-Candle Entry** | `false` | Require two consecutive HA candles for confirmation. |
| **Exit Drop Fallback (%)** | `0.5` | Fallback exit trigger if price drops this % from peak. Range: 0–3%. |

### Breakout & Pullback Configuration

| Parameter | Default | Description |
| --- | --- | --- |
| **Breakout Lookback Bars** | `75` | Number of bars to scan for the highest high / lowest low. Range: 10–200. |
| **Breakout Buffer (%)** | `0.5` | Price must exceed the breakout level by this %. Range: 0–3%. |
| **Breakout RSI Level** | `80` | RSI must be above this for breakout confirmation. Range: 60–95. |
| **Breakout Cooldown (candles)** | `15` | Minimum candles between breakout entries. Range: 1–50. |
| **Breakout Timeframe** | `normal` | `normal` (main period) or `long` (HTF breakouts). |
| **Bollinger Band Breakout** | `false` | Use BB upper/lower band as breakout trigger. |
| **BB Breakout: Close Confirm** | `false` | Require candle close outside BB, not just wick. |
| **Pullback Cooldown (candles)** | `10` | Minimum candles between pullback entries. Range: 1–30. |
| **RSI Pullback Confirmation Level** | `31` | RSI must cross up through this level for pullback confirmation. Range: 20–50. |
| **Pullback Up Price (%)** | `0` | Required price rebound % to confirm pullback. Range: 0–3%. |

### DCA & Re-Entry Settings (Classic TA)

| Parameter | Default | Description |
| --- | --- | --- |
| **DCA Drop (%)** | `1` | Required price drop % before DCA fires. Range: 0.1–5%. |
| **DCA Zone Threshold (%)** | `0.5` | Dead-zone around DCA level to avoid micro-entries. Range: 0–2%. |
| **DCA Aggressive Range (%)** | `3` | DCA spread range for aggressive mode. Range: 0.5–10%. |
| **Max DCA Trades** | `2` | Maximum DCA trades allowed. Range: 0–10. |
| **RSI DCA Max Level** | `60` | RSI must be below this for DCA to fire. Range: 20–90. |
| **Force DCA Always On** | `false` | Bypass DCA cooldowns and conditions. |
| **DCA Freeze Threshold (%)** | `0` | Freeze DCA entries if price drops below this % from entry. Range: 0–3%. |

### Experimental & Logging

| Parameter | Default | Description |
| --- | --- | --- |
| **Enable Console Logging** | `true` | Show detailed strategy logs in the Gunbot console. |
| **Dump Log to File** | `false` | Write full logs to disk (performance impact). |
| **Enable DCA Heal** | `false` | Experimental: automatically unstick zombie DCA positions after timeout. |
| **DCA Heal Timeout (hours)** | `24` | Hours before DCA Heal kicks in. Range: 1–72. |
| **Love this Coin** | `false` | Unlock extra aggressive entries on your most trusted pair. |
| **Strategy Nickname** | `Wick Magic Futures` | Display name shown in tournament and leaderboard stats. |

---

## 🔹 18. The OrderFlow Sidebar (Futures Edition)

When you enable Orderflow Mode, the Gunbot sidebar replaces classic metrics with live derivatives diagnostics:

| Tile | Description |
| --- | --- |
| **🎯 Wick Magic v1.2.8** | Active branch: `🌊 OrderFlow (SHORT/LONG)` |
| **ROC (annualized)** | Annualized return on capital since the pair started trading |
| **🌊 Signal** | Current OB direction: `BULLISH` / `BEARISH` / `NEUTRAL` |
| **⚖️ Imbalance** | Normalized imbalance score `[-1..+1]` |
| **🟢 Buy threshold** | GA-calibrated entry threshold (green = signal active) |
| **🔴 Sell threshold** | GA-calibrated exit threshold |
| **🧬 GA samples** | Samples collected vs minimum required. ✓ = calibrated |
| **📡 Signal Pair** | Source pair and exchange when cross-exchange mode is active. Example: `USDT-ETH @ binanceFutures`. Tile hidden when disabled. |
| **🧊 Iceberg bid/ask** | Detected iceberg orders near best spread |
| **🪤 Trap guard** | `CLEAR` / `🐂 Bull trap` / `🐻 Bear trap` |
| **🔬 PTH trades** | Public Trade History sample count and age |
| **🔄 Rebuy enabled** | Whether DCA re-entries are active |
| **🔒 Above guard** | Scale-up guard state: `OFF` / `PARTIAL OK ✓` / `WAITING PARTIAL` |
| **📉 Entry if drops** | Target price for DCA down. Shows `🎯 IN ZONE` when active |
| **📈 Entry if rises** | Target price for scale-up. Shows `⏳ IN ZONE` when active |
| **🏗️ S/R gate** | `ON (±x%)` with proximity — or `OFF` |
| **🏗️ S/R zone** | DCA count vs max at the active S/R level (e.g. `2/3`) |
| **🐋 Absorption** | Institutional absorption detection: `ON` / `OFF` |
| **💰 Compound** | Compound offset: `+0.41 → 20.41 USD` |
| **📊 True Cost** | True Cost Basis after below-entry partials: `$2,250.00 (+3.42%)`. Shows TCB value and drift vs exchange entry. Only visible when Allow Partial Close Below Entry is enabled. |
| **🏦 Inst. Hours** | `FULL POWER` / `SCOUT` / `BLOCKED` with current UTC hour |
| **CMC FGI Index** | Live CMC Fear & Greed Index with sentiment band |
| **FGI Buy Gate** | `ALLOW` / `BLOCK` — whether FGI permits new entries |
| **FGI DCA Gate** | `ALLOW` / `BLOCK` — whether FGI permits DCA |
| **⚡ MIMIX Active** | Effective trading limit after MIMIX multiplier |
| **📌 Mark Price** | Live mark price from the exchange (Oracle Price on dYdX v4) |
| **💸 Funding Rate** | Live funding rate per settlement period. 🟢 Negative (shorts pay longs) / 🔴 Positive (longs pay shorts). Annualized context shown in tooltip. |
| **📊 Cum. Funding** | Cumulative funding paid/received since position open (Hyperliquid and dYdX v4 only). Shows real carry cost of the position. |
| **🔮 Next Funding Est.** | Predicted next period funding rate (Kraken Futures only — derived from live order book imbalance). |
| **⏰ Next Funding** | Countdown to next funding settlement. Format: h/m for Binance (8h cycle), m/s for Hyperliquid, dYdX, Kraken (1h cycle). |
| **🛡️ Funding Guard** | Shows current annualized rate vs configured limit. `✅ CLEAR (44% APR)` or `⚠️ BLOCKED (342% APR > 300% limit)`. Tile only appears when Funding Rate Guard is enabled. |
| **🕯️ Heikin Color** | Current Heikin Ashi candle color: `green` / `red` |
| **🏦 Total Capital** | Configured capital limit for this pair |
| **💼 Used Capital** | Actual margin committed to open positions (not notional). Accurate across all leverage modes. |
| **📊 Capital Used (%)** | Percentage of total capital in use |
| **🤖 Smart Capital** | `ACTIVE` / `OFF` — hard stop triggered state |
| **📈 Realized P&L** | Cumulative realized profit for this pair |
| **📉 Unrealized P&L** | Live floating PnL from exchange |
| **📊 PnL (ROE %)** | Return on Equity from exchange adapter (funding-adjusted) |
| **🪢 Rekt Liq. Price** | Liquidation price from exchange + distance to current price |
| **🛡️ Liq. Limit (%)** | Distance to liquidation vs configured threshold |
| **⚠️ Margin Usage** | Global Account Margin Usage direct from exchange API |
| **🕒 Days Trading** | How many days this pair has been running |
| **💰 Position margin** | Actual margin committed to the current position (notional divided by leverage). Shows your real collateral exposure. |
| **💵 Margin available** | Available margin for new positions |
| **📦 Position size** | Current open position size in quote currency |
| **📊 Avg Volume** | Average candle volume (used for volume gating) |
| **🕒 Buy Cooloff** | `ACTIVE` / `CLEAR` — entry cooldown state |
| **⚙️ Mode** | `💰 CLOSE` (in position) / `💼 OPEN` (flat) |
| **🎯 Target Exit** | Price target for full exit based on profit target |
| **🎗️ Dyn Trailing** | Current dynamic trailing % |
| **🔻 Trailing Stop** | Current absolute trailing stop price |
| **🌊 Partial SELL** | `ACTIVE` / `READY` — partial close state |
| **🔪 Kill Position** | `OFF` / `HARD STOP only` / `Trail: peak $X.XX | dist $Y` |
| **Order Type** | `🧱 Limit` (OL mode) / `💸 Market` |

---

## 🔹 19. Real Trade Examples (Long & Short)

Both of these examples reflect an aggressive 5m scalping configuration on Hyperliquid/dYdX (ETH-USD), utilizing **3x Leverage**, **~$20 notional base size**, and **Auto Spread DCA** (dynamically adapting to ATR volatility).

### 📈 Example 1: The Long Position (Precision OrderFlow Exit)

The bot successfully accumulated a position, rode the reversal, and is now securing profits at the exact local top.

* **Position:** LONG 0.0760 ETH.
* **Status:** In Profit (`PnL: +1.66%`).
* **The Engine at Work:** The price rallied to 2210.0, surpassing the 1.20% profit target. The Elastic Trailing engine was actively following the price up, securing a safety net at 2207.82. However, the live order book detected a bearish shift in market pressure at the top.
* **Action:** Instead of waiting for the price to drop back down and hit the trailing stop, the OrderFlow exit gate fired. The bot instantly executed a full market close exactly at 2210.0, front-running the reversal and locking in the maximum gain before the market could pull back.

![long_trade_hyperliquid-USD-ETH](https://github.com/user-attachments/assets/f81b4013-a495-4bac-ba69-336ee46872cd)

### 📉 Example 2: The Short Position (Actively Averaging Up)

The bot is managing an open SHORT position, automatically scaling its base size with compounded profits (`+0.41 → 20.41`).

* **Position:** SHORT 0.0091 ETH on a $1000 base capital.
* **Status:** Currently in a minor drawdown (`Unrealized P&L: -0.14 USD` | `ROE: -0.43%`). Funding fees are calculated natively into this ROE.
* **The Engine at Work:** The price has moved slightly against the entry, but the OrderFlow is strongly `🔴 BEARISH` (`-0.6558` imbalance vs `-0.3541` threshold). The trap guard is CLEAR, and the GA is fully calibrated (`2000 samples`).
* **Action:** The bot is patiently waiting. If the price continues to rise against the position, it is ready to hit the next structural DCA target (`~3221.79`). Because the live order book confirms heavy selling pressure is still present, the bot will confidently average up (add to the short) the moment price touches the resistance zone (`S/R zone 1/2`). Meanwhile, the Margin Usage is at a very safe `0.99%`, keeping the account completely protected from liquidation risk.

<img width="1280" height="620" alt="short-trade-hyperliquid-USD-ETH_lowres" src="https://github.com/user-attachments/assets/feffcde5-26fd-4fc5-ac25-cb4851bc0b14" />

---

## 🔹 20. OrderFlow vs Classic Branch

| Feature | Classic (TA) | OrderFlow Futures |
| --- | --- | --- |
| **Entry signal** | Heikin Ashi, RSI, WRI, breakouts | Live OB imbalance + Native Futures Mechanics |
| **DCA** | Configurable drop % ladder | Rebuy via OF signal near S/R |
| **Self-adapting** | No (fixed thresholds) | Yes (GA recalibrates every 4h) |
| **Rebuy spacing** | Fixed % drop between levels | ATR-derived per timeframe (period, long, daily) |
| **S/R zone limits** | None | Max re-buys per support/resistance level |
| **Smart S/R Bias** | None | Direction gate: support → LONG, resistance → SHORT |
| **Partial sell** | HA-candle or grouped legacy | OF exit gate (bearish signal) + Elastic Trailing |
| **Dump/Pump protection** | None | Lowers profit floor during confirmed adverse pressure |
| **Trap protection** | None | Bull/bear trap detection |
| **Session gating** | None | Institution Hours (scout or block) |
| **Auto-compounding** | Compound Profits (cumulative PnL offset) | Compound Profits (cumulative PnL offset) |
| **Cross-exchange signals** | None | Signal Pair + Signal Exchange |
| **Emergency exit** | Kill Position (USD hard stop + trailing from peak uPnL + flat TP) | Kill Position (USD hard stop + trailing from peak uPnL + flat TP) |
| **Liquidation safety** | Liquidation Guard (percentage-based distance buffer) | Liquidation Guard (percentage-based distance buffer) |
| **AUTO direction logic** | Institutional Trend Mode (sma200 / supertrend / smacross) | Institutional Trend Mode (sma200 / supertrend / smacross) |
| **Funding awareness** | Live rate, countdown, cumulative carry cost, annualized Guard | Live rate, countdown, cumulative carry cost, annualized Guard |

---

## 🔹 21. Tips & Best Practices for Futures

* **Leverage is a multiplier of risk, not just profit.** The GA thresholds do not care about your leverage. If you run 20x, keep your Minimum Profit to Exit and DCA spreads wider. If you run 3x, you can scalp tighter.
* **Max Re-Entries per S/R Level** — Set to 2 or 3. In Futures, stacking 8 entries at the exact same support level because of chop is how accounts blow up. Let the bot wait for the *next* structural level if the first one fails.
* **Smart S/R Bias + S/R Confluence is the ultimate entry filter.** Confluence gates *where* to enter, Bias gates *which direction*. Together they ensure you're always trading the right side of the right zone — not guessing.
* **Auto-ATR DCA is your friend.** In derivatives, fixed percentages (like 1%) mean nothing when BTC moves 5% in an hour. Use Re-Entry DCA: Auto Spread mapped to the `long` (4h) timeframe to ensure your DCA net is cast at realistic volatility extremes.
* **Start with `orderflow_scalp**` on major pairs (BTC, ETH, SOL). These have the thickest order books, giving the GA the best training data.
* **On Hyperliquid, always configure a Signal Pair.** HL doesn't expose PTH in all pairs. Point Signal Pair to the equivalent pair on Binance Futures for full two-layer confirmation. The signal pair just needs to be cycling with Wick Magic Futures — a second Gunbot instance or a second active pair on the same instance both work.
* **Kill Position is a safety net, not a strategy.** Set Kill Position Loss (USD) at a level you'd genuinely accept as a max loss, not as a micro stop-loss. It closes at market. Reserve it for black swan protection, not normal DCA management.
* **Institutional Trend Mode + AUTO works best on slow timeframes.** On `long` (4h) TF with `smacross`, direction flips once or twice a day at most. Combine with `orderflow_institutional` preset for a clean swing setup that only trades with the macro structure. If you want faster reactivity, try `orderflow_institutional_fast` — same session-aware logic but with 1-hour trend context.
* **Watch the Funding Rate tile before entering.** If you're about to open a Long and the funding rate is +0.15%/h (annualized ~1,314%), you're entering a trade where every single hour costs you 0.15% of notional. That's fine for a scalp that lasts 20 minutes. It's catastrophic for a swing trade held overnight.
* **Use negative funding as a signal, not just a perk.** High negative funding (shorts paying longs) means the market is heavily short-biased. On a strongly negative-funding day, any bullish OF signal is doubly confirmed — the crowd is short, the book shows buying pressure, and you're getting paid to be Long.
* **Enable Funding Rate Guard on Long-biased strategies.** Set `Max Annualized Funding Rate` to 300–400%. During a bull frenzy when everyone is piling into longs, the Guard silently sits out — then resumes as soon as the crowd unwinds. On Short-biased strategies (or `auto` mode where the bot picks direction independently), consider disabling it or setting it very high.
* **Kraken Futures traders: watch the `🔮 Next Funding Est.` tile.** Kraken predicts the next period's rate in real time from live order book data. If the estimate is spiking before settlement, you can decide to delay your next entry until after the settlement clears — especially useful if you're about to add a DCA lot.
* **Supported Exchanges:** Hyperliquid, dYdX v4, Binance Futures, and Kraken Futures are supported with live PTH, order book depth, and native Funding Rate telemetry. All four deliver real-time funding data via dedicated WebSocket connections (Binance via 30s REST poll). You get the ultimate, uncompromised OrderFlow experience with real tape confirmation on all supported venues.
* **Unified Accounts & Multi-Collateral:** On Hyperliquid, if you hold USDC in Spot and use a Unified Account, it automatically serves as collateral for your perps positions — no transfers needed. On Kraken Futures, BTC/ETH/EUR and other assets count as margin. Use the `orderflow_krakenfutures_full` preset for dynamic collateral reading, or `orderflow_krakenfutures_btc` if you configure in BTC. On Binance Futures, use `orderflow_binancefutures_btc` to set Total Capital and Trade Size directly in BTC — the strategy auto-converts to USD notional behind the scenes.

---

## 🛍️ Get the Strategy

> 🛒 **[Purchase Wick Magic Here](https://www.gunbot.com/devcommunity/wickmagic-futures)**

🛠 *To run this strategy on Hyperliquid/dYdX, you need a valid Gunbot Defi license.*

* Upgrade to Defi from Standard License – $160 → [Purchase here](https://checkout.gunbot.com/crazymop/promoStandardToDefi)
* Upgrade to Defi from Pro License – $70 → [Purchase here](https://checkout.gunbot.com/crazymop/promoProToDefi)

🔑 Grab Gunbot Defi at [gunbot.com](https://checkout.gunbot.com/crazymop/ultimate)
