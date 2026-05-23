# Heikin-Ashi Candlestick Alert — MQL4 Script

A MetaTrader 4 script that computes **Heikin-Ashi (HA) candlestick values** natively from raw OHLC data each cycle, tracks the smoothed open and close across consecutive iterations via persistent `PrevHAOpen` and `PrevHAClose` globals, and fires independent alerts for both **trend reversal** (HA candle color flip) and **trend continuation** (consecutive same-color HA candles) signals — using the standard HA formulas for close, open, high, and low reconstruction.

---

## Overview

Heikin-Ashi is a Japanese charting technique meaning "average bar" — it reconstructs each candle using averaged OHLC values rather than raw price data, producing a smoothed representation of price action that filters out much of the noise inherent in standard candlestick charts. The key property of Heikin-Ashi is that the open of each bar is computed as the average of the previous bar's HA open and close, creating a recursive smoothing effect that causes strong trends to produce long bodies with no lower wicks (bullish) or no upper wicks (bearish). This script implements the full HA construction pipeline natively in MQL4: `HAClose = (O + C + H + L) / 4`, `HAOpen = (PrevHAOpen + PrevHAClose) / 2`, `HAHigh = max(H, HAClose, HAOpen)`, `HALow = min(L, HAClose, HAOpen)`. The script then classifies each computed bar as bullish or bearish by comparing `HAClose` to `HAOpen` and comparing the current classification to the prior cycle's classification to determine reversal or continuation.

---

## Features

- **Native HA construction** — `CalculateHeikinAshi()` computes all four HA values from `iOpen()`, `iClose()`, `iHigh()`, `iLow()` for bar 0 and the persistent `PrevHAOpen` / `PrevHAClose` state, implementing the standard recursive HA formula without requiring `iCustom()`
- **Reversal detection** — `AlertReversals` gate: `PrevHAClose > PrevHAOpen && HAClose < HAOpen` → **Bearish Reversal**; `PrevHAClose < PrevHAOpen && HAClose > HAOpen` → **Bullish Reversal** — strict prior-cycle guard (`PrevHAOpen != 0 && PrevHAClose != 0`) prevents first-cycle false triggers
- **Continuation detection** — `AlertContinuations` gate: `HAClose > HAOpen && PrevHAClose > PrevHAOpen` → **Bullish Trend Continuation**; `HAClose < HAOpen && PrevHAClose < PrevHAOpen` → **Bearish Trend Continuation** — both current and prior bars must be same-color for continuation to fire
- **Independent alert gating** — `AlertReversals` and `AlertContinuations` boolean inputs allow selective monitoring; both can be active simultaneously for full HA signal coverage
- **Persistent HA state** — `PrevHAOpen` and `PrevHAClose` globals updated at every cycle end, maintaining the recursive smoothing chain across loop iterations
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)
- Alert message includes the current `HAClose` price value with symbol and timeframe for immediate context

---

## How It Works

1. Every minute, `CalculateHeikinAshi()` computes:
   - `HAClose = (open + close + high + low) / 4`
   - `HAOpen = (PrevHAOpen + PrevHAClose) / 2`
   - `HAHigh = MathMax(MathMax(high, HAClose), HAOpen)`
   - `HALow = MathMin(MathMin(low, HAClose), HAOpen)`
2. If `AlertReversals && PrevHAOpen != 0 && PrevHAClose != 0`: prior/current color flip conditions evaluated
3. If `AlertContinuations`: same-color consecutive bar conditions evaluated independently
4. `PrevHAOpen = HAOpen` and `PrevHAClose = HAClose` updated unconditionally at cycle end

---

## Input Parameters

| Parameter              | Type            | Default     | Description                                                    |
|------------------------|-----------------|-------------|----------------------------------------------------------------|
| `TradeSymbol`          | string          | `EURUSD`    | Symbol for analysis                                            |
| `Timeframe`            | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for HA computation                                   |
| `AlertReversals`       | bool            | `true`      | Fire alerts on HA candle color flips (trend reversals)         |
| `AlertContinuations`   | bool            | `true`      | Fire alerts on consecutive same-color HA bars (continuations)  |
| `EnableAlerts`         | bool            | `true`      | Fire an on-screen/sound alert                                  |
| `EnableEmail`          | bool            | `false`     | Send an email notification                                     |
| `EnablePush`           | bool            | `false`     | Send a mobile push notification                                |

---

## Alert Message Format

```
Bullish Reversal Detected detected on EURUSD (Timeframe: PERIOD_H1)
Price: 1.08420
```

---

## Installation

1. Copy `Heikin-Ashi_Candlestick_Alert_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
