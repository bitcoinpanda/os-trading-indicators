# CC Model PRO · TheStrat Candle Confirmation

🇫🇷 [Version française](./README.md)

TradingView indicator (Pine Script v6) that detects **TheStrat's CC (Candle Confirmation) model**, filters it with an **ICT market structure bias (BOS/CHoCH)** and shows the full execution on the chart: entry, stop, TP1, TP2 and lot size.

> Concept: TheStrat CC model, as presented by Trader Mike. This script is an independent implementation by OS Trading, with an added bias filter and execution helper.

> ⚠️ **The indicator's interface is in French.** This guide gives the French label next to each English explanation so you can find every setting.

---

## Contents

1. [Install](#1-install)
2. [The model in 3 steps](#2-the-model-in-3-steps)
3. [Reading the banner](#3-reading-the-banner)
4. [Reading the chart](#4-reading-the-chart)
5. [Settings](#5-settings)
6. [Alerts (text or JSON for MT5)](#6-alerts)
7. [Stats table](#7-stats-table)
8. [Known limits](#8-known-limits)

---

## 1. Install

1. Open [`CC_Model_PRO.pine`](./CC_Model_PRO.pine) and click **Copy raw file** (copy icon at the top right of the code).
2. On TradingView: **Pine Editor** (bottom of the screen) → **Open** → **New indicator**.
3. Delete the default code, paste the script, **Save**, then **Add to chart**.
4. Set the chart to the right timeframe:

| Setup | Chart (execution) | Signal candle | Bias (default) |
|---|---|---|---|
| **A** | 15m | 1H | 4H |
| **B** | 5m | 15m | 4H |

The signal timeframe is picked automatically: chart ≤ 5m → 15m · 15m → 1H · 1H → 4H.

**NAS100 on the 5m (recommended)** — change these settings before trading:

| Setting (French label) | Value |
|---|---|
| Buffer beyond the F2 (*Marge au-delà du failed 2*) | 2 to 5 pts (default 0.30 is for gold) |
| TP1 before target (*TP1 avant la cible*) | 2 to 3 pts |
| Point value per lot (*Valeur de 1.0 de mouvement pour 1 lot*) | check your broker, usually 1 |
| MT5 symbol (*Symbole MT5*) | your broker's name, e.g. `NAS100` |

---

## 2. The model in 3 steps

**① Signal candle (higher timeframe)**
- Bullish: the candle **closes above the high** of the previous candle.
- Bearish: the candle **closes below the low** of the previous candle.
- If it also broke the other extreme (outside bar) it is tagged "3", otherwise "2".
- **CC target** = the high (long) or low (short) of the signal candle.

**② "F2" confirmation (failed 2) on the chart**
During the confirmation window (default: first half of the next signal candle, 2 bars minimum):
- Long: a bar trades **below the previous bar's low**, then **closes back above** it.
- Short: a bar trades **above the previous bar's high**, then **closes back below** it.
- The **orange line** shows the exact level to watch; it updates on every close.

**③ Execution**
- If the F2 close gives at least the minimum RR to TP1 → **market entry**.
- Otherwise → **limit order** at the price that gives that RR.
- The setup is cancelled if the target is hit before the F2, or if the window expires.

**Example (NAS100, setup B, 4H bias bullish)**
- The 15m candle closing at 10:15 NY closes above the previous 15m high → **aligned LONG** setup, target = high of that candle.
- At 10:20 on the 5m, a bar wicks below the previous low and closes back above → **F2**.
- The banner shows `✔ LONG MAINTENANT` (long now) with entry, SL, TP1, TP2 and lot size.
- TP1 hit → close 50 %, stop to breakeven, the rest runs to TP2.

---

## 3. Reading the banner

Only the banner (top center) tells you what to do:

| Banner (French) | Meaning | Action |
|---|---|---|
| Bias color (`H4 ▲ HAUSSIER · LONGS UNIQUEMENT`) | 4H bullish, longs only | No trade, wait for an aligned setup |
| `◉ EN PRÉPARATION` | Getting ready | Aligned setup: prepare the order (lot already computed) |
| `⚡ F2 EN FORMATION` | F2 forming | The current bar is forming the F2: be ready to enter on the close |
| `✔ LONG / SHORT MAINTENANT` | Long / short now | Enter: entry, SL, TP and lot are written |
| `⏳ LIMITE` | Limit | Place the limit order at the price shown |
| `● EN POSITION` / `● RUNNER` | In position / runner | Manage the trade (TP1, then runner at BE to TP2) |
| `✖ CONTRE-BIAIS` | Counter-bias | Grey ghost signal: **do not trade** |
| `⚠ TP1 … % → STOP & ANALYSE` | Win rate warning | Recent TP1 rate fell below your threshold |

---

## 4. Reading the chart

| Element | Meaning |
|---|---|
| Thin black lines | Start (solid) and end (dashed) of the signal session |
| Solid box "ALIGNÉ ✔" | Tradable signal candle |
| Grey dotted box "CONTRE-BIAIS ✖" | Counter-bias signal: never alerts, never entry lines |
| Orange dotted line | F2 zone: the level to watch for confirmation |
| White line | Entry (solid = market, dashed = limit) |
| Red line | Stop loss |
| Green lines | TP1 (solid) / TP2 (dashed) |
| Grey step line | Bias flip level: a close beyond it = CHoCH |

---

## 5. Settings

### 1 · Model (*Modèle*)
| Setting | Default | Role |
|---|---|---|
| Signal timeframe (*Timeframe signal*) | auto | Empty = picked from the chart timeframe |
| Max confirmation bars (*Bougies de confirmation max*) | 0 (auto) | Auto = first half of the next candle, min 2 |
| Allow late confirmation (*Accepter une confirmation tardive*) | off | Off = strict model |

### 2 · Bias filter (*Filtre de biais*)
| Setting | Default | Role |
|---|---|---|
| Bias filter | Structure BOS/CHoCH (ICT) | Trend = last swing broken **on a close** on the bias timeframe. Other options: TheStrat candle (original) or none (not recommended) |
| Bias timeframe (*Timeframe du biais*) | 4H | |
| Swing strength (*Force des swings*) | 5 | Bars on each side to confirm a swing. Higher = slower, more stable bias |
| Show counter-bias signals in grey | on | Shown and simulated separately to compare stats |
| Tint background by bias | off | |
| Show bias flip level | on | |

### 3 · Execution (*Exécution*)
| Setting | Default | Role |
|---|---|---|
| Minimum RR to TP1 (*RR minimum vers TP1*) | 1.0 | Below that, a limit is placed at the price that gives this RR |
| Stop | 1:1 symmetric | Symmetric = same distance as the target (and at least beyond the F2). Structural = F2 extreme + buffer |
| Buffer beyond the F2 | 0.30 | XAUUSD: 0.30 · NAS100: 2 to 5 pts. Add your spread for shorts |
| TP2: 50 % at TP1, stop to BE | on | |
| TP2 in R | 2.0 | |
| TP1 before target | 0 | Exit just before liquidity. NAS100: 2 to 3 pts · XAUUSD: 0.30 |

### 4 · Risk & lot (*Risque & lot*)
| Setting | Default | Role |
|---|---|---|
| Capital ($) | 100,000 | |
| Risk per trade (%) (*Risque par trade*) | 0.25 | |
| Point value per lot ($) | 1.0 | MT5 → right-click the symbol → Specification → Contract size. NAS100: often 1 · XAUUSD: 100. **Check with your broker** |
| Lot step (*Pas de lot*) | 0.01 | |
| Broker offset (*Décalage broker*) | 0 | MT5 price − TradingView price, measured at the same moment. MT5 prices are then shown in brackets |

**Lot size example**: capital $100,000, risk 0.25 % → $250. Stop at 25 pts on NAS100 with $1/pt/lot → 250 / (25 × 1) = **10 lots**.

### 5 · Session
| Setting | Default | Role |
|---|---|---|
| Signals only during session | on | Outside the session: no new setup, no F2 |
| Session (New York time) | 08:00-12:00 | = 13:00-17:00 London = 14:00-18:00 Paris |

### 6 · Alerts, 7 · Display (*Affichage*), 8 · Performance tracking (*Suivi de performance*)
See the sections below. Colors, banner size and table position can be changed.

---

## 6. Alerts

Create **one single alert** on the indicator with the condition **"Any alert() function call"**. Only **aligned** setups send alerts.

### Text format (*Texte*)
```
CC Model · NAS100 · ✔ LONG MAINTENANT · Entrée 21 450.0 · SL 21 425.0 (25.0 pts) · TP1 21 475.0 · TP2 21 500.0 · LOT 10
```

### JSON format (webhook to an MT5 bridge)
```json
{"symbol":"NAS100","action":"buy","entry":21450.0,"sl":21425.0,"tp1":21475.0,"tp2":21500.0,"lots":10,"msg":"..."}
```

| `action` | When |
|---|---|
| `info` | New setup, SL/TP hit, performance warning |
| `prepare` | F2 forming on the current bar |
| `buy` / `sell` | Market entry on the F2 close |
| `buy_limit` / `sell_limit` | Limit order to place |
| `partial_be` | TP1 hit: close 50 % (`close_pct`) and move stop to BE (`sl`) |
| `cancel` | Limit expired or target hit without fill: cancel the order |

The `symbol` field comes from the "MT5 symbol" setting. Prices include the broker offset.

---

## 7. Stats table

Simulated on the visible history, aligned trades only (counter-bias trades are counted separately):

- Number of aligned trades and missed setups (limit never filled)
- TP1 hit rate, overall and over the last N trades (default 20)
- Total R and average R
- Long / short breakdown
- Before / after the 09:30 NY cash open
- Risk in $ per trade

If the rolling TP1 rate drops below the threshold (default 55 %, from 10 trades), an alert fires and the banner shows `⚠ STOP & ANALYSE`.

---

## 8. Known limits

- Stats **exclude spread and slippage**.
- Stop and target hit in the same bar = counted as a **loss** (conservative).
- **Only one aligned trade tracked at a time.**
- The `⚡ F2 forming` alert fires intrabar: the F2 is only valid on the close.
- The signal candle and the bias use closed candles (no repaint).
- Mainly tested on **NAS100** and **XAUUSD**. Adjust point value and buffer for any other market.

---

## Disclaimer

Decision-support tool, not investment advice. Past performance, including the stats table, does not guarantee future results. Test in bar replay or on demo before trading live.

## License

[Mozilla Public License 2.0](../../LICENSE) · © OS Trading (SakuraTradeLab)
