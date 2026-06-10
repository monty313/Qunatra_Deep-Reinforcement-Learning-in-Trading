# PPO ENGINE
### How the Bot Learns — Architecture and Dials
*Locked decisions = the bot's skeleton. Dials = settings we will test and tune.*
*Each dial is explained in plain language: what personality trait does it control?*

**Status key:** 🔴 Locked | 🟡 Suggested/In Progress | ❓ Open Question
**Repo:** https://github.com/monty313/Qunatra_Deep-Reinforcement-Learning-in-Trading.git

---

## LOCKED ARCHITECTURE 🔴

### Algorithm: PPO (Proximal Policy Optimization)
**Not DQN.** DQN cannot size positions on a smooth continuous scale. PPO can.
PPO learns a probability distribution over actions and slowly reshapes it toward profitable ones.
It uses on-policy rollouts (fresh recent trades), not a replay memory of old trades.

### Two Output Heads
The bot outputs two decisions every time it acts:

| Head | Output | What it means |
|---|---|---|
| Head 1: Direction | {long / short / flat} | Which way, or stay out |
| Head 2: raw_size | 0.0 → 1.0 continuous | How much of allowed risk to deploy |

**raw_size is platform-blind.** It is a fraction (0.0 = nothing, 1.0 = full risk allowed).
The RiskManager converts raw_size into platform language AFTER the policy decides:

| Platform | raw_size becomes |
|---|---|
| FTMO (MT5) | lots — scaled to account size (10k / 25k / 50k / 100k) |
| Apex / Tradovate | contracts |
| Schwab | shares |

*The policy never knows which platform it's on. It only knows raw_size.*

### No Replay Memory 🔴
PPO is on-policy. It trains on fresh rollouts (a short batch of recent trades), then discards them.
No long-term replay buffer like DQN. Each update is based only on recent policy behavior.

### Laws Run First 🔴
Before the policy samples any action, the Trading Code runs.
Forbidden actions get their logit set to −1e9 (effectively probability zero).
Then PPO picks from what remains.
**Laws are never in the reward. They are pre-reward hard masks.**

---

## THE DIALS 🟡
*These are suggestions from course notes and research. Not locked. Will be tested.*
*Each one is explained as a personality trait — what does turning this knob do to the bot?*

---

### Dial 1: BACKBONE — LSTM vs MLP

**What this controls:** Does the bot have memory across bars, or does it only see the current snapshot?

| Option | Plain language | Best for |
|---|---|---|
| MLP (basic network) | Bot looks at this bar only. No memory of what just happened. | Stateless, fast, simpler to train |
| LSTM (memory network) | Bot remembers recent bars. Can sense "momentum has been building for 5 bars." | Momentum and pullback trading |

**Suggested:** LSTM — because your entire edge is about momentum continuing or reversing across time. The LSTM hidden state becomes the bot's memory of position history.
**Current status:** 🟡 — not locked. Needs to be confirmed by you.

**Question for you when ready:** Do you want the bot to have memory of what just happened (LSTM), or is each bar's features enough context on their own (MLP)?

---

### Dial 2: GAMMA — Discount Factor

**What this controls:** How much does the bot care about future profit vs right now?

| Value | Plain language | Behavior |
|---|---|---|
| Low (0.90) | "Give me profit now. I don't trust what happens 10 bars later." | Cuts winners early, takes quick profits |
| High (0.999) | "I'll wait for the full move. Future profit is almost as valuable as now." | Holds positions through pullbacks |

**Suggested:** High (0.995–0.999) — because your pullback strategy needs patience. The bot must learn that holding through a small retracement is worth it for the full move.
**Current status:** 🟡

**Question for you when ready:** How patient do you want the bot to be? How many bars should a "full move" typically last on your setup?

---

### Dial 3: LAMBDA — GAE Smoothing

**What this controls:** How does the bot measure "was that trade good?" across time?

| Value | Plain language | Behavior |
|---|---|---|
| Low (0.50) | Only looks at the very next bar to judge the trade. Fast but noisy. | Reactive, doesn't learn from sustained moves |
| High (0.98) | Averages the judgment across many bars. Slower but more accurate for trends. | Rewards sustained positions, learns from the full trade arc |

**GAE (Generalized Advantage Estimation)** = the formula that tells the bot "was holding that position worth it?" It combines gamma and lambda together.
**Suggested:** High (0.95–0.98) — smooths the advantage estimate so holding through a pullback gets properly credited.
**Current status:** 🟡

---

### Dial 4: ENTROPY COEFFICIENT

**What this controls:** How much does the bot explore vs lock onto what works?

| Value | Plain language | Behavior |
|---|---|---|
| High (0.01+) | "Keep trying different things, don't get stuck." | Explores widely — good early in training |
| Low (0.0001) | "I know what works. Stop second-guessing." | Locks onto the best actions fast |

**Your laws already do most of the exploration-filtering** — the bot can't try dumb trades anyway. So you want a LOW entropy coefficient from the start, decaying toward near-zero as training progresses.
**Suggested start:** 1e-3, decay to 1e-4 over training.
**Current status:** 🟡

---

### Dial 5: CLIP EPSILON (PPO's trust region)

**What this controls:** How big a step can the bot take toward new behavior per update?

*Note: This is NOT the same as DQN's epsilon. DQN's epsilon = how often it explores randomly. PPO's epsilon = how much the policy is allowed to change per update.*

| Value | Plain language | Behavior |
|---|---|---|
| Low (0.1) | Tiny steps. Very stable. Slow to change. | Conservative, won't overreact to one lucky trade |
| High (0.3) | Bigger steps. Learns faster. Slightly riskier. | Can shift toward profitable actions quickly |

**Suggested:** ~0.2 — moderate. Combined with low entropy and your laws, this gives fast learning within PPO's stability guardrails.
**Current status:** 🟡

---

### Dial 6: EPOCHS PER ROLLOUT

**What this controls:** How many times does the bot study each batch of trades before moving on?

| Value | Plain language | Behavior |
|---|---|---|
| Low (4–5) | Looks at each batch briefly, moves on quickly | Slow learning per batch |
| High (10–15) | Studies each batch hard before discarding it | Squeezes more learning from your curated setups |

**Suggested:** 10–15 — because your training data is curated (high-edge setups only), so each batch is valuable and worth studying deeply.
**Current status:** 🟡

---

### Dial 7: LEARNING RATE

**What this controls:** How aggressively does the network update its weights after each batch?

| Value | Plain language | Behavior |
|---|---|---|
| Low (3e-4) | Small weight updates. Stable but slower. | Standard PPO |
| Higher (1e-3) | Bigger updates. Learns faster but can destabilize if noisy. | Aggressive exploitation of your edge |

**Suggested:** ~1e-3 range, combined with monitoring for instability.
**Current status:** 🟡

---

## PLATFORM SCALING 🔴 (intent)
raw_size × account_max_risk = dollar risk for this trade.
RiskManager then converts to platform units:
- FTMO: raw_size × (account_balance × daily_risk_%) → lots
- Account sizes: 10k / 25k / 50k / 100k
- Same policy, different RiskManager per platform. Policy never changes.

---

## OPEN QUESTIONS ❓
- LSTM or MLP — needs your confirmation after understanding both options
- Exact gamma / lambda values — needs testing
- All dial values — suggestions only until tested in training

---

*Last updated: Blueprint session — PPO architecture locked, all dials defined with plain-language explanations*
