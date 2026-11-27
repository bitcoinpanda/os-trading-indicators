# London Breakout NQ · 4am-9am ET range, entry 9:30-11am

🇫🇷 [Version française](./README.md)

TradingView indicator (Pine Script v6) for **NASDAQ (NQ / NAS100) on the 1-minute chart**. It draws the London session range (04:00-09:00 New York time), waits for the **first breakout** during the US open (09:30-11:00), shows entry, SL and TP, and **backtests the strategy automatically** on the visible history.

**One trade per day max.**

> ⚠️ **The indicator's interface is in French.** This guide gives the French label next to each English explanation so you can find every setting.

---

## Contents

1. [Install](#1-install)
2. [The strategy in 4 steps](#2-the-strategy-in-4-steps)
3. [Reading the chart](#3-reading-the-chart)
4. [Settings](#4-settings)
5. ["Today" panel](#5-today-panel)
6. [Stats table](#6-stats-table)
7. ["Fixed SL" grid](#7-fixed-sl-grid)
8. [Alerts](#8-alerts)
9. [Known limits](#9-known-limits)

---

## 1. Install

1. Open [`London_Breakout_NQ.pine`](./London_Breakout_NQ.pine) and click **Copy raw file** (copy icon at the top right of the code).
2. On TradingView: **Pine Editor** (bottom of the screen) → **Open** → **New indicator**.
3. Delete the default code, paste the script, **Save**, then **Add to chart**.
4. Set the chart to **NAS100 / NQ, 1 minute**.

Times are always **New York time (ET)**, whatever your chart's time zone:

| Session | ET | London | Paris |
|---|---|---|---|
| London range | 04:00-09:00 | 09:00-14:00 | 10:00-15:00 |
| Entry window | 09:30-11:00 | 14:30-16:00 | 15:30-17:00 |
| Forced exit | 16:00 | 21:00 | 22:00 |

*(Outside the weeks when US and European daylight saving dates differ.)*

---

## 2. The strategy in 4 steps

**① The range (04:00-09:00 ET)**
The blue box tracks the session high and low. At 09:00 the range is locked: two black lines extend the **High** and the **Low**.

**② The open (09:30 ET)**
- If price opens **already outside the range** → the day is skipped (setting: enter at the open instead).
- Otherwise → the entry window opens (orange background) until 11:00.

**③ The breakout**
- Price touches the **High** → **BUY** at the High.
- Price touches the **Low** → **SELL** at the Low.
- **SL**: below the low (buy) or above the high (sell) of the breakout candle, + optional buffer.
- **TP**: 2R by default.
- Both sides broken in the same candle → day skipped (ambiguous).
- No breakout before 11:00 → no trade.

**④ The exit**
- SL, TP, or **forced exit at 16:00 ET** if neither is hit.
- Option: partial TP1, then SL to breakeven and the rest to TP2.

**Real example (NAS100 1m, Nov 27, 2026)**
- Range: High **26,829.47** · Low **26,550.67** (279 pts, 0.29 daily ATR).
- 09:30 ET: price opens inside the range → window open.
- The first 1m candle breaks the Low → **SELL at 26,550.67**.
- SL above the breakout candle: **26,561.30** (≈ 10.6 pts risk).
- 2R TP: **26,529.42** → hit a few minutes later: **+2R**.

---

## 3. Reading the chart

| Element | Meaning |
|---|---|
| Blue box "SESSION" | London range 04:00-09:00 ET |
| Dotted line inside the box | Range midpoint |
| Black lines | Range High and Low, extended to 16:00 (or 11:00) |
| Grey dashed box "SAUTÉ" | Range rejected by the size filter (*sauté* = skipped) |
| Orange background | Entry window 09:30-11:00 |
| Green / red zone | TP / SL of the open trade |
| "SL → BE" | TP1 hit, stop moved to entry |
| Green / red label | Day's result in R: `+2R`, `-1R`, `(temps)` = exit at 16:00, `(TP1+BE)` |

---

## 4. Settings

### Sessions
| Setting (French label) | Default | Role |
|---|---|---|
| Session time zone (*Fuseau des sessions*) | America/New_York | Don't change for NQ |
| London range (*Range London (ET)*) | 0400-0900 | |
| Entry window (*Fenêtre d'entrée (ET)*) | 0930-1100 | No new entry outside it |
| Management (*Gestion (sortie forcée à la fin)*) | 0930-1600 | Trade closed at market at the end |

### Trade
| Setting | Default | Role |
|---|---|---|
| TP R:R (*R:R du TP*) | 2.0 | TP distance as a multiple of risk (TP2 if TP1 is on) |
| SL buffer, points (*Buffer SL*) | 0 | Points added beyond the breakout candle. NAS100: 1 to 3 pts to cover spread |
| Price already outside range at 9:30 (*Prix déjà hors range à 9h30*) | Skip the day (*Sauter le jour*) | Or: enter at the 9:30 open (*Entrer à l'ouverture 9h30*) |
| Entry candle closing back inside = loss (*Bougie d'entrée refermée dans le range = perte*) | on | **Applies on 1m** (and whenever intrabar is off): if the breakout candle closes back inside the range, the trade counts as a loss (conservative, since price order inside the candle is unknown) |
| Side (*Sens*) | Both (*Les deux*) | Buys only (*Achats*) / sells only (*Ventes*). If the first side broken is excluded, no trade that day |

### TP1 / BE
| Setting | Default | Role |
|---|---|---|
| Partial TP1 + SL to BE (*TP1 partiel + SL à BE*) | off | Off = single TP. On = TP1, stop to entry, rest to TP2 |
| TP1 (R) | 1.0 | |
| Share closed at TP1 (*Part fermée au TP1*) | 50 % | |

### Range filter (*Filtre range*)
| Setting | Default | Role |
|---|---|---|
| Range size filter (*Filtre taille du range*) | None (*Aucun*) | **Points** or **× daily ATR** (previous day's ATR 14): skips ranges that are too small or too large (news days) |
| Min / Max | 0 / 1.0 | Example in × ATR: min 0.15, max 0.60 |

### Resolution (*Résolution*)
| Setting | Default | Role |
|---|---|---|
| Simulate intrabar (*Simuler en intrabar*) | on | On a chart **above 1m** (5m, 15m…), replays each candle minute by minute for the real breakout → SL → TP order. **No effect on 1m** (the table then shows "Bougie 1") |
| Intrabar TF | 1 | |
| SL below candle (*SL sous la bougie*) | Chart candle | With intrabar: SL below the chart candle (e.g. 5m) or below the breaking 1-minute candle |

### Display, Session box, Period
- "Today" panel, stats table, fixed SL grid, result labels, window background: on/off, positions adjustable.
- Box colors, dotted midpoint, tag (text "SESSION" editable), line width, extend lines to 11:00 or 16:00.
- **Limit stats to a period** (*Limiter les stats à une période*): compare two timeframes or two settings on exactly the same days.

---

## 5. "Today" panel

| Row | Content |
|---|---|
| High / Low | Today's range levels |
| Size (*Taille*) | Size in points and in daily ATR |
| Price vs High / Low | Distance from current price to both levels |
| State (*État*) | `Range en cours` (building) → `Range prêt · attente 9h30` (ready, waiting for 9:30) → `Fenêtre ouverte · attente cassure` (window open, waiting for breakout) → `EN TRADE ACHAT/VENTE` (in a buy/sell trade, live R) → `Terminé : +2R` (done); or the skip reason (`Sauté : déjà hors range à 9h30` = skipped, already outside range; `Pas de cassure aujourd'hui` = no breakout today) |

---

## 6. Stats table

| Row (French) | Meaning |
|---|---|
| Mode | Resolution used (`Bougie 1` on 1m, `Intrabar 1 / SL…` above) |
| Jours avec range / Trades | Days analysed / trades taken |
| Sautés (hors range 9h30) / (filtre range) | Days skipped: outside range at 9:30 / range filter |
| Pas de cassure | Days with no breakout before 11:00 |
| Ambigus / sens exclu | Both sides broken in one candle / side filtered out |
| Win rate (R > 0) | Green if above the **break-even win rate** (*WR mini rentable*: 100 / (1 + R:R), i.e. 33.3 % at 2R) |
| R moyen / trade, Total R | Expectancy and cumulative result |
| Achats / Ventes : n / R moy | Buys / sells: count and average R |
| Sorties temps / TP1+BE | Trades closed at 16:00 / at breakeven |
| Pire série de pertes | Longest losing streak |
| SL moyen (pts) / SL moyen / range | Average stop size, in points and as % of the range |
| Jours sans intrabar | Days with missing 1-minute data (simulated on the chart candle) |

---

## 7. "Fixed SL" grid

Second table: the **same entry**, but with a **fixed SL = k × range size** instead of the candle-based SL, and a 2R or 3R TP.

| k | TP 2R | TP 3R |
|---|---|---|
| 0.10 · 0.20 · 0.35 · 0.50 | average R · win rate · number of trades | same |

Use it to check whether a wider (or tighter) stop than the breakout candle would have worked better on your history.

---

## 8. Alerts

Create **one single alert** on the indicator with the condition **"Any alert() function call"**.

| When | Message |
|---|---|
| 09:00 ET, range locked | `LDN BO NAS100 \| range prêt  H 26829.47  L 26550.67` |
| During the window, real time | `LDN BO NAS100 : cassure du LOW London 26550.67` (London low broken) |
| Entry (candle close) | `LDN BO NAS100 VENTE  entrée 26550.67  SL 26561.30  TP 26529.42` (VENTE = sell, ACHAT = buy) |

---

## 9. Known limits

- Stats **exclude spread and slippage**. Entry is assumed filled **exactly** at the High/Low: live, a stop order on a fast breakout can slip a few points.
- On 1m, the SL below the breakout candle is often **tight** (a few dozen points on NAS100): make sure your broker accepts it and add a buffer.
- If SL and TP are hit in the same candle, **SL wins** (conservative).
- **One trade per day**: only the first breakout counts.
- Above 1m with intrabar simulation, TradingView caps the replay at about 100,000 one-minute bars: the tested history is shorter.
- Tested on **NAS100 CFD, 1m**. Session times are built for NQ; adapt them for any other market.

---

## Disclaimer

Decision-support tool, not investment advice. The stats cover the visible history only and do not guarantee future results. Test in bar replay or on demo before trading live.

## License

[Mozilla Public License 2.0](../../LICENSE) · © OS Trading (SakuraTradeLab)
