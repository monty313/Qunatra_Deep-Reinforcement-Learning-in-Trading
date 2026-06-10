# STATE VECTOR
### Everything the Bot Sees When It Makes a Decision
*These are the inputs. The bot reads this snapshot at every bar and decides what to do.*
*Current state = bare bones from course notes. More features will be added as the course progresses.*

**Status key:** 🔴 Locked | 🟡 Suggested/In Progress | ❓ Open Question
**Repo:** https://github.com/monty313/Qunatra_Deep-Reinforcement-Learning-in-Trading.git

---

## ENCODING RULE 🟡
All features are normalized into compact scalars before entering the state vector.
Regime signals: −1 (bearish) / 0 (neutral) / +1 (bullish) per timeframe.
Raw values (ATR level, z-score) are normalized to a common scale.
*Reason: keeps the state vector compact so the network doesn't choke on scale differences.*

---

## TIMEFRAMES IN USE 🔴 (intent)
- Higher Timeframe (HTF): 30m
- Lower Timeframe (LTF): 5m
- More timeframes may be added as the course progresses

---

## FEATURE LIST

### 1. Bollinger Band Position 🔴 (now laws)
| Feature | Detail |
|---|---|
| Bands | BB20 + BB200, deviation 1 |
| Timeframes | 5m + 30m |
| What it captures | Where is price relative to the structure of its own range? |
| Encoding | Position relative to midline and outer bands → scalar per TF |
| Note | These are also the basis of F-001 and Bollinger Push No. 1 |

### 2. CCI Momentum 🟡
| Feature | Detail |
|---|---|
| Setup | 3 CCIs per timeframe (different periods — small, medium, large) |
| Timeframes | 5m + 30m = 6 CCIs total |
| Shift applied | Shift-4 (period 1) applied to each CCI → compares now vs 4 bars ago |
| What it captures | Momentum direction AND whether it's consistent across speeds |
| Your logic | When small CCI diverges from larger ones, it usually reverts back to their direction — that's a pullback signal |
| Encoding | CCI value + CCI vs its shift → direction scalar per TF |

### 3. ATR Volatility Regime 🟡
| Feature | Detail |
|---|---|
| Setup | ATR + identical ATR shifted forward 4 bars |
| What it captures | Is volatility/momentum increasing or decreasing? |
| Your logic | If current ATR > shifted ATR → volatility rising (momentum building). ATR level = how extreme the regime is |
| Encoding | ATR level (normalized) + ATR slope (current vs shifted) → 2 scalars |

### 4. Shifted SMA Trend Filter 🟡
| Feature | Detail |
|---|---|
| Setup | SMA period-1 shift-4, applied to HIGH and LOW |
| Timeframes | 5m + 30m |
| What it captures | Is price above or below the recent high/low trend line? |
| Your logic | Price below SMA-shift on high AND low = trend is down on that TF |
| Encoding | Position vs each shifted SMA → scalar per TF |

### 5. Z-Score (Pullback Depth) 🟡
| Feature | Detail |
|---|---|
| Formula | z = (price − rolling_mean) / rolling_std |
| Timeframes | Per timeframe |
| What it captures | How far has price deviated from equilibrium? = pullback opportunity signal |
| Your logic | High z on HTF + negative z on LTF = pullback in an uptrend → buy setup |
| Lookback length | ❓ TBD |
| Encoding | z-score per TF → scalar |

### 6. Open Position Context 🟡
| Feature | Detail |
|---|---|
| What it captures | Is momentum continuing or weakening on the current open trade? |
| Inputs | Current unrealized PnL + direction + CCI state while in trade |
| Why it matters | The bot needs to know if holding is still justified (delayed gratification) |
| Encoding | Position direction (−1/0/+1) + momentum state → 2 scalars |

---

## FEATURES STILL BEING DEFINED ❓
- OBV-based shifted SMA (mentioned in notes section 7.6 — logic: price below both BB midlines = bearish continuation. Needs exact spec.)
- Additional timeframes (1H? Daily? — not yet decided)
- Time-of-day feature (session filter — not yet decided)

---

## STATE VECTOR SHAPE (current estimate) 🟡
| Group | Approx. scalars |
|---|---|
| BB position (2 TFs) | ~4 |
| CCI + shifts (6 CCIs × 2) | ~12 |
| ATR regime | ~2 |
| Shifted SMAs (2 TFs × 2) | ~4 |
| Z-score (2 TFs) | ~2 |
| Open position context | ~2 |
| **Total (bare bones)** | **~26 scalars** |

*This will grow as more features and laws are added through the course.*

---

*Last updated: Blueprint session — bare bones feature set from course sections 2.7, 4.14, 7.6*
