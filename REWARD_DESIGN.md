# REWARD DESIGN
### The Scoreboard — What "Winning" Means to the Bot
*The reward tells PPO what to optimize for. Get this wrong and the bot optimizes for the wrong thing.*
*Laws are NOT in the reward. Laws run before. Reward governs what happens inside the legal space.*

**Status key:** 🔴 Locked | 🟡 Suggested/In Progress | ❓ Open Question
**Repo:** https://github.com/monty313/Qunatra_Deep-Reinforcement-Learning-in-Trading.git

---

## NORTH STAR — What the Bot Is Optimizing 🔴
The bot is NOT optimizing for maximum raw PnL.
The bot is optimizing for: **hit DAILY_TARGET while staying within DAILY_RISK — consistently over months.**

Two user inputs per session:
- DAILY_TARGET — how much I want to make today
- DAILY_RISK — how much I am willing to lose today (default 4% trailing)

The reward must reflect this, not raw profit.

---

## THE PROGRESSIVE STANCE 🔴 (intent)
*From course section 7.6 — your reframe.*

Because the laws mask out bad actions, the bot can no longer be penalized for taking wrong actions — it can't take them. So the reward must shift to penalizing **stagnation inside a favorable setup.**

> **Stagnation is the new failure.**
> If the bot is in a favorable legal state and does nothing — or holds a position that is going nowhere — that is bad. Flat when conditions are favorable is a negative signal.

This is a progressive reward stance: not "did you avoid losing?" but "did you make progress given what the law allowed?"

---

## REWARD FORMULA 🟡
*Suggested from course notes section 7.6. Not locked — α and β are dials to be tuned.*

```
r(t) = ΔPnL(t) + α · momentum_signal(t) − β · stagnation_penalty(t)
```

| Term | What it means | Status |
|---|---|---|
| ΔPnL(t) | Change in net PnL this bar (realized + unrealized) | 🔴 core |
| α · momentum_signal | Bonus for being in a trade where momentum continues in your direction | 🟡 α = TBD |
| β · stagnation_penalty | Penalty for holding (hold→hold→hold) with no equity improvement in a favorable state | 🟡 β = TBD |

**Note:** Bot always sees NET PnL only. Platform costs are deducted by the cost layer before the reward is computed. The policy never sees gross PnL.

---

## PAIN ZONE 🔴 (intent)
*Hard backing-off behavior before the FTMO wall.*

| Zone | Behavior |
|---|---|
| 0% → 3.5% daily loss | Normal reward. Bot trades freely inside laws. |
| 3.5% → 4.0% daily loss | Reward punishes hard. Bot backs off. Position sizes shrink. |
| 4.0%+ daily loss | FTMO hard wall. Bot stops trading for the day. |

*The pain zone is a reward signal — it creates urgency to stop before the wall.*
*The 4% wall itself is a hard stop in risk/, not a reward.*

---

## DELAYED GRATIFICATION — HOLDING WINNERS 🟡
*From course sections 2.7 and 7.6.*

The bot must learn that holding through a pullback is worth it for the full move.
This is taught through:

**1. High gamma + high lambda (the math)**
- High gamma = future rewards nearly as valuable as now → bot won't cut early
- High lambda = advantage is smoothed across many bars → holding through a pullback gets properly credited
- The "reason to hold" = positive advantage Â > 0 while in position
  - This only happens when the critic network V(s) predicts the next state is worth more than the current one
  - Your momentum features (CCI, ATR slope) feed directly into this prediction

**2. Curriculum (the teaching method)**
- Train first on pullback setups only (as you described)
- Bot learns: "when HTF is bullish + LTF pulls back = wait for the entry, then hold for the continuation"
- This is prospective curriculum (you filter setups before training) — NOT retroactive labeling

---

## REWARD COMPONENTS NOT YET DEFINED ❓

| Component | Why it matters | Status |
|---|---|---|
| α (momentum bonus weight) | How much to reward being in a continuing trade | ❓ TBD — needs testing |
| β (stagnation penalty weight) | How hard to punish holding flat in a favorable state | ❓ TBD — needs testing |
| Stagnation window | How many bars of flat equity counts as "stagnation"? | ❓ TBD |
| Momentum signal definition | Exactly which features define "momentum continuing"? | 🟡 CCI + ATR slope candidates |

---

## WHAT THE COURSE USES (for reference — not what we're building)
The course (DQN) uses simple reward functions:
- Categorical PnL: +1 for win, −1 for loss
- Positive categorical: +1 for win, 0 for loss
- Exponential PnL: log-scaled reward

These are too simple for our setup. They don't account for risk, sizing, or holding behavior.

---

*Last updated: Blueprint session — progressive stance locked as intent, formula suggested from section 7.6, dials TBD*
