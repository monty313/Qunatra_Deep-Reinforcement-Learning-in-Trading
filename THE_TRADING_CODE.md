# THE TRADING CODE
### The Bot's Permanent Laws
*Laws are permanent. A law is a law because it defines a trend you will never trade against.*
*Laws run BEFORE the reward — they are hard masks, not penalties.*
*The bot thinks and trades fully inside the law. Only the forbidden direction is off limits.*

**Status key:** 🔴 Locked | 🟡 Suggested/In Progress | ❓ Open Question
**Repo:** https://github.com/monty313/Qunatra_Deep-Reinforcement-Learning-in-Trading.git

---

## LAW NAMING SYSTEM
- Name = describes the logic. You recognize it on sight.
- Number = order within that law family.
- Tier = Federal (supreme) or Statute (binding, narrower).
- All laws are 🔴 CONSTANT. Never modified without explicit approval.

---

## FEDERAL LAWS
*Supreme. Nothing overrides a Federal Law.*

### F-001 — The Trend Sovereignty Act 🔴
> When the trend has total dominance, you never trade against it.

| Field | Definition |
|---|---|
| Condition | Live tick (current price) is above ALL 4 outer bands on BOTH timeframes |
| Bands | BB20 upper + BB20 lower + BB200 upper + BB200 lower (deviation 1) |
| Timeframes | 5m AND 30m — both must agree (same direction) |
| Sell ban | Price above all 4 bands on both TFs → NO sells, ever |
| Buy ban | Price below all 4 bands on both TFs → NO buys, ever |
| If TFs disagree | Act does not trigger. Bot trades freely (subject to other laws) |
| Trigger basis | Raw live tick. Applies the instant price crosses. No buffer. |
| Known property | Tick-flicker at band edge is theoretically possible but practically negligible — requires price to hover on edge of all 8 band-points across 2 TFs simultaneously |
| Within the law | Full judgment: size, timing, hold, exit, stand-aside all belong to the bot |
| Nesting | When F-001 is active, Bollinger Push No. 1 is automatically also active (price past outer bands has already cleared midlines). No double-counting — bot checks "is ANY law banning this direction?" |

---

## STATUTES
*Binding. Permanent. Narrower scope than Federal.*

### Bollinger Push — No. 1 🔴
> When price has cleared the center of both Bollinger bands on both timeframes, the trend is leaning. You do not fade a leaning market.

| Field | Definition |
|---|---|
| Condition | Live tick is above/below the MIDLINES of BB20 and BB200 on BOTH timeframes |
| Bands | BB20 midline + BB200 midline — 2 per timeframe, 4 total (deviation 1) |
| Timeframes | 5m AND 30m — both must agree (same direction) |
| Sell ban | Price above all 4 midlines, both TFs agreeing → NO sells |
| Buy ban | Price below all 4 midlines, both TFs agreeing → NO buys |
| If TFs disagree | Does not trigger |
| Trigger basis | Raw live tick. Same as F-001. |
| Tier | Statute — binding, permanent, weaker than Federal (midlines < outer bands) |
| Within the law | Full judgment |

---

## LAWS IN PROGRESS ❓
*(Confirmed logic, name pending or details being decided)*

- CCI Confluence Law — 2 of 3 CCIs above/below level 60 + shift confirmation → forbids counter-trend entry. Name TBD. Details TBD.
- Shifted SMA Law — price below SMA period-1 shift-4 applied to high AND low on 2 TFs → forbids buys (mirror for sells). Name TBD.

---

## FUTURE LAWS 🟡
*(From course notes — logic identified, not yet legislated)*
- ATR regime law (volatility context — may govern sizing more than direction)
- Z-score law (pullback depth — may govern when entries are allowed vs forbidden)

---

*Last updated: Blueprint session — 2 laws locked (F-001, Bollinger Push No. 1)*
