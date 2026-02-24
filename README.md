# XAUUSD Hedge EA MT5 — V2

A fully-featured MetaTrader 5 Expert Advisor that runs a dual-sided hedge grid strategy on XAUUSD (Gold). The EA opens both Buy and Sell positions simultaneously, layers in with doubling lot sizes when price moves against a side, and includes an advanced **Freeze Hedge Mode** for deep drawdown protection.

Converted from the original MT4 version with all platform-specific adaptations for MT5 (position-based model, `CTrade`, `CDealInfo`, `COrderInfo`, etc.).

---

## How It Works

The EA opens a Buy and a Sell trade at startup, each with a pending limit order one interval away. When a side profits (via Break Even and Trailing Stop), it restarts with a fresh cycle. When a side goes into loss, the pending limit fills and a new layer is placed with doubled lot size. All existing positions on that side have their TP adjusted to a common recovery level.

If the losing side reaches 8 layers deep, **Freeze Hedge Mode** activates automatically. This places the 9th layer as a normal limit order and simultaneously places a Stop order in the opposite direction to hedge the full exposure. While frozen, no new layers or cycles are opened. The EA monitors for exit conditions and cleanly returns to normal trading once resolved.

---

## Input Parameters

### Core Settings

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Trade_Direction` | Both | Buy only, Sell only, or Both sides |
| `Initial_LotSize` | 0.01 | Starting lot size for layer 1 |
| `Multipliar` | 2.0 | Lot multiplier per layer |
| `Next_Interval_Points` | 1000 | Distance between layers in points ($10 on XAUUSD) |
| `TP_Points` | 1000 | Take Profit distance in points |
| `Magic_Number` | 423423 | Unique identifier for EA trades |
| `Trade_Comment` | XAUUSD EA v2 | Comment attached to all orders |

### Break Even & Trailing Stop

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Use_Break_Even` | true | Enable Break Even on layer 1 positions |
| `Break_Even_At` | 30 | Points in profit before SL moves to entry |
| `Use_Trailing_Stop` | true | Enable Trailing Stop |
| `Trailing_Activate` | 30 | Points in profit before trailing begins |
| `Trail_Behind` | 30 | Distance trailing SL stays behind price |

### Freeze Hedge Mode

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Freeze_Hedge_Enable` | true | Enable/disable Freeze Hedge Mode |
| `Freeze_Trigger_Layer` | 8 | Layer count that triggers Freeze Mode |
| `StopTrade_BE_Enable` | false | Enable Break Even on the Stop trade only |
| `StopTrade_BE_TriggerPoints` | 50 | Points before Stop trade BE activates |
| `Max_Stop_Lot` | 200.0 | If Stop lot ≥ this value, split into 2 orders |

### Daily Limits

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Use_Daily_Profit` | true | Stop trading after daily profit target |
| `Day_Profit` | 100 | Daily profit target in USD |
| `Use_Daily_Loss` | true | Stop trading after daily loss limit |
| `Day_Loss` | 10 | Daily loss limit in USD |

### Trading Schedule

Each day of the week (Monday through Sunday) can be individually enabled or disabled with custom start and end times in `HH:MM` format.

### Friday Stop Trading

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Friday_Stop_Trading` | false | Close all trades on Friday if in profit |
| `Friday_Stop_Time` | 21:00 | Server time to start monitoring on Friday |

### News Filter

The EA includes a built-in Forex Factory news filter that can pause trading before and after high-impact news events. Configurable by impact level (High, Medium, Low, Speaks, Holidays) and by currency.

---

## Freeze Hedge Mode — Detailed Logic

### Activation

When the losing side reaches the configured trigger layer (default 8), Freeze Mode activates and the EA:

1. Places the 9th layer as a normal limit order with doubled lot and standard TP.
2. Places a Stop order in the opposite direction with lot size = Last Layer Lot × Multiplier × Multiplier. The Stop order has SL set to the 9th layer's TP and TP set to 0 (manual control).
3. If the Stop lot exceeds `Max_Stop_Lot`, it splits into two equal orders automatically.

### While Frozen

All normal trading logic is skipped. No new layers, no new cycles. Only the Freeze exit checks and optional Stop trade Break Even run.

### Exit Scenarios

**Scenario 1 — Stop hits at 9th layer TP:** The Stop order's SL triggers at the 9th layer TP price. EA detects this, closes all remaining positions, deletes all pending orders, and exits Freeze Mode.

**Scenario 2 — Stop hits Break Even, then 9th TP hits:** The Stop order moves to Break Even and gets closed. EA tracks this state. When the 9th layer later hits its TP, EA closes everything and exits.

**Scenario 3 — Manual exit:** If all positions are manually closed (Total_Positions == 0), EA deletes remaining pending orders and exits Freeze Mode.

**Bounce Scenario:** If price reverses and all layer TPs are hit before the 9th layer or Stop order triggers, EA detects Total_Positions == 0, cleans up pending orders (including the 9th limit and Stop), and returns to normal trading.

---

## TP/SL Protection

The Stop trade is fully protected from automatic modifications. Break Even, Trailing Stop, and TP modification functions all skip any position matching the Stop trade ticket(s). This ensures the Stop trade remains under manual control while Freeze Mode is active.

---

## Layer Progression Example

With `Initial_LotSize = 0.2` and `Multipliar = 2`:

| Layer | Lot | Cumulative |
|-------|-----|------------|
| 1 | 0.20 | 0.20 |
| 2 | 0.40 | 0.60 |
| 3 | 0.80 | 1.40 |
| 4 | 1.60 | 3.00 |
| 5 | 3.20 | 6.20 |
| 6 | 6.40 | 12.60 |
| 7 | 12.80 | 25.40 |
| 8 | 25.60 | 51.00 |
| 9 (Freeze) | 51.20 | 102.20 |
| **Stop Order** | **102.40** | — |

---

## Platform

- **MetaTrader 5** (MQL5)
- Tested on XAUUSD (Gold) with ICMarkets demo
- Uses standard MQL5 Trade library (`CTrade`, `CPositionInfo`, `CDealInfo`, `COrderInfo`)

---

## Files

| File | Description |
|------|-------------|
| `XAUUSD Hedge EA MT5 V2.mq5` | Main EA source code |
| `README.md` | This documentation |

---

## Developer

**Asad Sheikh**
- Email: kaleemullahkhan.contact@gmail.com
