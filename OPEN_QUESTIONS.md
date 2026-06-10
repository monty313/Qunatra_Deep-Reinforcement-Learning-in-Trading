# OPEN QUESTIONS
### Everything Undecided — Flagged by What It Blocks
*Nothing here is a problem. These are decisions we park until we have enough information to answer them well.*
*Answer format: bring the question, define it in plain language, offer options — then decide.*

**Repo:** https://github.com/monty313/Qunatra_Deep-Reinforcement-Learning-in-Trading.git

---

## HOW TO USE THIS FILE
When a question gets answered, move it to the relevant file (PPO_ENGINE, REWARD_DESIGN, etc.) with its status updated to 🔴 or 🟡.
When a new question comes up, add it here with the BLOCKS field filled in.

---

## OPEN QUESTIONS

---

### Q-001: LSTM vs MLP Backbone
**What it means in plain language:**
Does the bot have memory of recent bars (LSTM) or does it only see the current bar's features (MLP)?
LSTM = "I remember momentum has been building for 5 bars."
MLP = "I only see what's in front of me right now."

**Why it matters:** Your edge is momentum-based across time. LSTM is better for that. But LSTM is slower to train and harder to debug.

**Options:**
- LSTM — memory across bars, better for momentum/pullback trading
- MLP — stateless, faster, simpler
- Test both and compare

**Blocks:** PPO_ENGINE.md backbone section
**Source:** Course section 4.14, 7.6

---

### Q-002: raw_size — Continuous Head Parameterization
**What it means in plain language:**
The bot outputs a number between 0.0 and 1.0 for position size (continuous = confirmed).
The question is: what shape of distribution does the output head use to generate that number?

**Options:**
- Beta distribution — naturally bounded between 0 and 1, good for size fractions
- Gaussian (clipped) — standard normal, but must be clipped to 0–1
- Squashed Gaussian — tanh-squashed, used in SAC-style setups

**Why it matters:** The distribution shape affects how the bot explores different sizes during training.
**Blocks:** PPO_ENGINE.md — Head 2 implementation detail
**Note:** We know raw_size is continuous 0.0–1.0. This is just the implementation detail of how that head is built.

---

### Q-003: Z-Score Lookback Length
**What it means in plain language:**
The z-score measures how far price has moved from its average. How many bars back should that average be calculated?
Short lookback = sensitive to recent price. Long lookback = measures deviation from longer-term equilibrium.

**Options:** 20 bars / 50 bars / 100 bars / per-timeframe (different for 5m vs 30m)
**Blocks:** STATE_VECTOR.md — z-score feature spec
**Source:** Course section 7.6

---

### Q-004: Alpha (Momentum Bonus Weight)
**What it means in plain language:**
How much extra reward does the bot get for being in a trade where momentum is continuing in its direction?
Higher α = bot is more motivated to stay in winning trades.

**Blocks:** REWARD_DESIGN.md — formula
**Note:** Needs training experiments to calibrate. Cannot be decided theoretically.

---

### Q-005: Beta (Stagnation Penalty Weight)
**What it means in plain language:**
How hard does the bot get penalized for holding a position that isn't going anywhere in a favorable setup?
Higher β = bot is more impatient with stagnant trades and will exit or resize sooner.

**Blocks:** REWARD_DESIGN.md — formula
**Note:** Needs training experiments. Risk of too-high β = bot exits every small pause in a trend.

---

### Q-006: Stagnation Window
**What it means in plain language:**
How many bars of flat equity (no improvement) counts as "stagnation"?
Too short = bot gets penalized for every normal pause in price.
Too long = bot holds dead trades for too long before acting.

**Blocks:** REWARD_DESIGN.md — stagnation_penalty definition
**Note:** Likely needs to be different for 5m bars vs 30m bars.

---

### Q-007: Exact Gamma + Lambda Values
**What it means in plain language:**
See PPO_ENGINE.md Dial 2 (gamma) and Dial 3 (lambda) for full plain-language explanations.
These control how patient the bot is and how it credits holding a winning position.

**Suggested range:** Gamma 0.995–0.999 | Lambda 0.95–0.98
**Blocks:** PPO_ENGINE.md — dials section
**Note:** Will be set during first training runs. Suggested values are a starting point.

---

### Q-008: CCI Law — Exact Conditions
**What it means in plain language:**
You said: "2 of 3 CCIs above level 60 AND shift confirmation → forbids sells."
This needs to be fully specified before it becomes a law:
- Which 3 CCIs (which periods)?
- What counts as "shift confirmation"?
- What is the exact mirror for buys?
- What timeframe(s)?
- Name?

**Blocks:** THE_TRADING_CODE.md — Laws In Progress section
**Source:** Course section 6.1

---

### Q-009: Shifted SMA Law — Exact Conditions
**What it means in plain language:**
You said: "price below SMA period-1 shift-4 applied to HIGH AND LOW on 2 TFs → forbids buys."
Needs full specification:
- Which 2 timeframes exactly?
- Does "below" mean close below, or any tick below?
- Mirror for sells?
- Name?

**Blocks:** THE_TRADING_CODE.md — Laws In Progress section
**Source:** Course section 6.1

---

### Q-010: Additional Timeframes
**What it means in plain language:**
Currently using 5m and 30m. Will you add a higher timeframe (1H, 4H, Daily) for macro regime context?

**Blocks:** STATE_VECTOR.md — timeframes section
**Note:** More timeframes = richer state but larger state vector and slower training.

---

### Q-011: Section 8.1 (Part 2 of Course)
**Status:** Notes not yet taken. Will be added when you reach this section.
**Blocks:** Unknown until notes arrive.

---

*Last updated: Blueprint session — 11 open questions logged*
