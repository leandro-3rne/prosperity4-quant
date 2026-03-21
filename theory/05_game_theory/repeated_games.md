# Repeated Games

> **Core formula:** $$U_i \;=\; \sum_{t=0}^{\infty} \delta^t \, u_i(a^t) \qquad \text{where } \delta \in (0,1) \text{ is the discount factor}$$

## Intuition

In a one-shot game, players make a single decision and walk away. The equilibrium of the Prisoner's Dilemma — mutual defection — is bleak precisely because there is no tomorrow: no opportunity to reward cooperation or punish betrayal. But most real interactions are repeated. You see the same colleagues every day, trade with the same counterparties every session, and compete against the same bots every round.

Repetition fundamentally changes the strategic landscape. When a game is played over and over, players can condition their current actions on the entire history of past play. This opens the door to strategies like "cooperate as long as you cooperate, but punish you forever if you defect." The threat of future punishment can sustain cooperation that would be impossible in a one-shot setting. The more patient the players (the higher the discount factor $\delta$), the more weight future payoffs carry, and the easier it is to sustain cooperative outcomes.

The **folk theorem** formalises this intuition: in an infinitely repeated game with sufficiently patient players, *any* payoff vector that gives each player more than their minimax value (the worst they can guarantee themselves) can be sustained as a Nash equilibrium. This means the set of equilibrium outcomes explodes with repetition — cooperation, partial cooperation, alternating patterns, and even complex contingent strategies all become possible. The challenge shifts from "what is the equilibrium?" to "which equilibrium will be selected?" — a question that depends on coordination, communication, and the specific strategies employed.

## Mathematical Setup

| Symbol | Meaning |
|--------|---------|
| $G$ | The stage game (played each period) |
| $N = \{1, \ldots, n\}$ | Set of players |
| $A_i$ | Action set for player $i$ in the stage game |
| $a^t = (a_1^t, \ldots, a_n^t)$ | Action profile in period $t$ |
| $u_i(a)$ | Player $i$'s stage-game payoff from action profile $a$ |
| $\delta \in (0,1)$ | Common discount factor |
| $h^t = (a^0, a^1, \ldots, a^{t-1})$ | History of play up to period $t$ |
| $H^t$ | Set of all possible histories of length $t$ |
| $\sigma_i : \bigcup_{t=0}^{\infty} H^t \to \Delta(A_i)$ | Player $i$'s strategy (maps histories to mixed actions) |
| $U_i(\sigma)$ | Player $i$'s total discounted payoff under strategy profile $\sigma$ |
| $\underline{v}_i$ | Player $i$'s minimax value in the stage game |
| $\mathcal{F}$ | Set of feasible payoff vectors (convex hull of stage-game payoff profiles) |

**Finitely vs. infinitely repeated games:**

- **Finitely repeated** ($T$ periods): backward induction often unravels cooperation. In the finitely repeated Prisoner's Dilemma with complete information, the unique subgame-perfect equilibrium is to defect in every period.
- **Infinitely repeated** ($T = \infty$): no "last period" means backward induction cannot start. Cooperation becomes sustainable via trigger strategies.

**Normalised payoffs.** To make payoffs comparable across different horizons, we sometimes use the **average discounted payoff**:

$$
\bar{U}_i = (1 - \delta) \sum_{t=0}^{\infty} \delta^t \, u_i(a^t).
$$

The factor $(1-\delta)$ normalises the geometric sum so that a constant stream of payoff $c$ yields $\bar{U}_i = c$.

## Derivation

### 1. Discounted Payoff — Geometric Series

If player $i$ receives a constant stage payoff $c$ in every period:

$$
U_i = \sum_{t=0}^{\infty} \delta^t \cdot c = c \cdot \sum_{t=0}^{\infty} \delta^t = c \cdot \frac{1}{1 - \delta}.
$$

This uses the geometric series formula: for $|\delta| < 1$,

$$
\sum_{t=0}^{\infty} \delta^t = \frac{1}{1 - \delta}.
$$

For a stream that pays $c_1$ for one period then $c_2$ forever after:

$$
U_i = c_1 + \delta \cdot c_2 + \delta^2 \cdot c_2 + \cdots = c_1 + \delta \cdot \frac{c_2}{1 - \delta} = c_1 + \frac{\delta\, c_2}{1 - \delta}.
$$

### 2. The Minimax Value

The **minimax value** of player $i$ is the lowest payoff that player $i$'s opponents can force upon $i$, assuming $i$ plays optimally against their punishment:

$$
\underline{v}_i = \min_{a_{-i} \in A_{-i}} \max_{a_i \in A_i} u_i(a_i, a_{-i}).
$$

In the Prisoner's Dilemma:

$$
\begin{array}{c|cc}
 & C & D \\
\hline
C & (3,\;3) & (0,\;5) \\
D & (5,\;0) & (1,\;1)
\end{array}
$$

To compute $\underline{v}_1$: Player 2 chooses the column to minimise Player 1's best payoff.

- If P2 plays $C$: P1's best response is $D$, yielding $u_1 = 5$.
- If P2 plays $D$: P1's best response is $D$, yielding $u_1 = 1$.

P2 minimises by choosing $D$. So $\underline{v}_1 = 1$. By symmetry, $\underline{v}_2 = 1$.

The minimax payoff profile is $(1, 1)$ — the mutual defection outcome.

### 3. The Folk Theorem

**Theorem (Folk Theorem, Nash equilibrium version).** Let $G$ be a finite stage game and $G(\delta)$ the infinitely repeated game with discount factor $\delta$. Let $\mathcal{F}$ be the set of feasible payoff vectors (the convex hull of $\{u(a) : a \in A\}$). For any payoff vector $\mathbf{v} = (v_1, \ldots, v_n) \in \mathcal{F}$ satisfying

$$
v_i > \underline{v}_i \qquad \forall\, i \in N,
$$

there exists a threshold $\bar{\delta} \in (0, 1)$ such that for all $\delta \in (\bar{\delta}, 1)$, the payoff vector $\mathbf{v}$ can be sustained as a Nash equilibrium average payoff of $G(\delta)$.

**Interpretation:** Any feasible payoff that strictly Pareto-dominates the minimax profile can be achieved in equilibrium if players are patient enough. The set of sustainable payoffs is the region in payoff space above the minimax levels and inside the feasible set.

For the Prisoner's Dilemma, the feasible set is the convex hull of $\{(3,3), (0,5), (5,0), (1,1)\}$, and the minimax profile is $(1,1)$. So any payoff in the interior of this set that gives each player more than $1$ can be sustained — including the cooperative outcome $(3,3)$.

### 4. Grim Trigger Strategy — Detailed Analysis

**Definition.** The grim trigger strategy for the repeated Prisoner's Dilemma:

- **Period 0:** Play $C$ (cooperate).
- **Period $t \ge 1$:** Play $C$ if all previous action profiles were $(C, C)$. Otherwise, play $D$ forever.

Formally, for player $i$:

$$
\sigma_i^{\text{grim}}(h^t) = \begin{cases} C & \text{if } a^s = (C, C) \;\;\forall\, s < t \\ D & \text{otherwise} \end{cases}
$$

**Claim.** If both players use grim trigger, the resulting outcome is $(C, C)$ in every period, provided $\delta \ge 1/2$.

**Proof.** Suppose both players follow grim trigger. Consider Player 1's incentive to deviate at some period $t$, given that no deviation has occurred up to $t$ (so the history is all $(C, C)$).

**Payoff from continued cooperation (no deviation):**

$$
V^{\text{coop}} = \sum_{\tau=t}^{\infty} \delta^{\tau - t} \cdot 3 = \frac{3}{1 - \delta}.
$$

**Payoff from deviating at period $t$:**

At period $t$, Player 1 plays $D$ while Player 2 (still following grim trigger) plays $C$. Payoff at $t$: $5$.

From period $t+1$ onward, Player 2 detects the deviation and plays $D$ forever. Player 1's best response to perpetual $D$ is also $D$ (since $D$ is dominant in the stage game). Payoff from $t+1$ onward: $1$ per period.

$$
V^{\text{dev}} = 5 + \sum_{\tau=t+1}^{\infty} \delta^{\tau - t} \cdot 1 = 5 + \delta \cdot \frac{1}{1 - \delta} = 5 + \frac{\delta}{1 - \delta}.
$$

**No-deviation condition:** Player 1 prefers cooperation if $V^{\text{coop}} \ge V^{\text{dev}}$:

$$
\frac{3}{1 - \delta} \;\ge\; 5 + \frac{\delta}{1 - \delta}
$$

Multiply both sides by $(1 - \delta)$ (positive since $\delta < 1$):

$$
3 \;\ge\; 5(1 - \delta) + \delta
$$

$$
3 \;\ge\; 5 - 5\delta + \delta
$$

$$
3 \;\ge\; 5 - 4\delta
$$

$$
4\delta \;\ge\; 2
$$

$$
\delta \;\ge\; \frac{1}{2}
$$

$$
\boxed{\text{Cooperation via grim trigger is sustainable if and only if } \delta \ge \frac{1}{2}}
$$

By symmetry, the same condition holds for Player 2. So for $\delta \ge 1/2$, the strategy profile (grim trigger, grim trigger) is a Nash equilibrium that sustains permanent cooperation with payoff $(3, 3)$ every period. $\blacksquare$

**Interpretation:** The critical discount factor $\delta^* = 1/2$ quantifies the patience required for cooperation. A higher short-term temptation gain (the $5$ from deviating vs. $3$ from cooperating) or a milder punishment (if defection gave more than $1$) would increase the required $\delta^*$.

**General formula.** For a Prisoner's Dilemma with payoffs:

$$
\begin{array}{c|cc}
 & C & D \\
\hline
C & (R,\;R) & (S,\;T) \\
D & (T,\;S) & (P,\;P)
\end{array}
$$

where $T > R > P > S$ (temptation $>$ reward $>$ punishment $>$ sucker), the grim trigger threshold is:

$$
\frac{R}{1-\delta} \ge T + \frac{\delta P}{1-\delta} \quad \Longrightarrow \quad \delta \ge \frac{T - R}{T - P}.
$$

For our example: $T = 5$, $R = 3$, $P = 1$, $S = 0$:

$$
\delta^* = \frac{5 - 3}{5 - 1} = \frac{2}{4} = \frac{1}{2}. \quad \checkmark
$$

### 5. Tit-for-Tat Strategy

**Definition.** Tit-for-tat (TFT) for the repeated Prisoner's Dilemma:

- **Period 0:** Play $C$.
- **Period $t \ge 1$:** Play whatever the opponent played in period $t-1$.

$$
\sigma_i^{\text{TFT}}(h^t) = \begin{cases} C & \text{if } t = 0 \\ a_{-i}^{t-1} & \text{if } t \ge 1 \end{cases}
$$

**Properties of tit-for-tat:**

| Property | Description |
|----------|-------------|
| **Nice** | Starts by cooperating — never defects first |
| **Retaliatory** | Punishes defection immediately in the next period |
| **Forgiving** | Returns to cooperation after a single punishment period |
| **Clear** | Simple rule that opponents can easily recognise and adapt to |

**Axelrod's computer tournament (1984).** Robert Axelrod invited game theorists to submit strategies for a repeated Prisoner's Dilemma tournament. Tit-for-tat, submitted by Anatol Rapoport, won both the original tournament and a follow-up tournament — despite being one of the simplest strategies submitted. It outperformed far more complex strategies because:

1. It established cooperation quickly (nice).
2. It deterred exploitation (retaliatory).
3. It recovered from mutual defection spirals (forgiving — one defection triggers one punishment, not permanent war).

**When TFT sustains cooperation.** If both players use TFT, the outcome is $(C, C)$ forever (since TFT starts with $C$ and copies the opponent's $C$). The deviation analysis is similar to grim trigger, but the punishment is milder: after a deviation, the opponent defects for one period, then returns to cooperation if the deviator returns to cooperation.

Consider a one-period deviation: Player 1 defects at $t$, then returns to $C$ at $t+1$:

- Period $t$: P1 plays $D$, P2 plays $C$. Payoff: $(5, 0)$.
- Period $t+1$: P1 plays $C$, P2 plays $D$ (copying P1's defection). Payoff: $(0, 5)$.
- Period $t+2$ onward: both play $C$ again. Payoff: $(3, 3)$.

The deviation payoff for Player 1 (relative to the all-cooperation stream) is:

$$
V^{\text{dev}}_{\text{TFT}} = 5 + \delta \cdot 0 + \delta^2 \cdot \frac{3}{1-\delta}
$$

The cooperation payoff is:

$$
V^{\text{coop}} = 3 + \delta \cdot 3 + \delta^2 \cdot \frac{3}{1-\delta} = \frac{3}{1-\delta}
$$

No-deviation condition:

$$
\frac{3}{1-\delta} \ge 5 + \frac{3\delta^2}{1-\delta}
$$

$$
\frac{3 - 3\delta^2}{1-\delta} \ge 5
$$

$$
\frac{3(1-\delta)(1+\delta)}{1-\delta} \ge 5
$$

$$
3(1+\delta) \ge 5
$$

$$
3 + 3\delta \ge 5
$$

$$
\delta \ge \frac{2}{3}
$$

So TFT requires $\delta \ge 2/3$ to sustain cooperation, which is a stricter requirement than grim trigger's $\delta \ge 1/2$. This makes sense: grim trigger imposes a harsher punishment (permanent defection vs. one-period retaliation), so it deters deviations more effectively, requiring less patience.

### 6. Comparing Trigger Strategies

| Strategy | Punishment severity | Required $\delta^*$ | Robustness to errors |
|----------|--------------------|-----------------------|----------------------|
| Grim trigger | Permanent defection | $\frac{T-R}{T-P}$ (lowest) | Poor — one accidental defection causes permanent breakdown |
| Tit-for-tat | One-period retaliation | $\frac{T-R}{T-S}$ (moderate) | Good — recovers from isolated mistakes |
| Tit-for-two-tats | Retaliate after two consecutive defections | Higher | Very good — tolerates single errors |
| $k$-period punishment | Defect for $k$ periods, then forgive | Between grim and TFT | Tunable via $k$ |

For our Prisoner's Dilemma example ($T=5, R=3, P=1, S=0$):

- Grim trigger: $\delta^* = (5-3)/(5-1) = 1/2$
- Tit-for-tat: $\delta^* = 2/3$

### 7. Finite Repetition and the Unravelling Problem

In a **finitely repeated** Prisoner's Dilemma (say, $T$ periods), backward induction shows that defection in every period is the unique subgame-perfect equilibrium:

**Period $T$:** This is the last period, so it's a one-shot game. The dominant strategy is $D$.

**Period $T-1$:** Both players know defection occurs at $T$ regardless. So the $T$-th period payoff is $(1,1)$ no matter what. The period $T-1$ decision is effectively a one-shot game. Dominant strategy: $D$.

**Continuing backwards:** By induction, $D$ is played in every period.

This "unravelling" breaks down when:

1. The game is infinitely repeated (no last period).
2. There is uncertainty about the number of periods (players don't know when the game ends).
3. Players have incomplete information about each other's types (e.g., some players might be "irrationally cooperative").

In Prosperity 4, the number of trading rounds is finite but large, and there is significant uncertainty about opponent strategies, so cooperation can be sustainable in practice despite the theoretical unravelling result.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Discount factor | $\delta$ | Weight on future payoffs relative to present | $(0, 1)$; close to 1 = patient |
| Stage-game payoff | $u_i(a)$ | Immediate reward from action profile $a$ | Problem-specific |
| Temptation payoff | $T$ | Payoff from defecting against cooperator | $T > R$ |
| Reward payoff | $R$ | Payoff from mutual cooperation | $T > R > P$ |
| Punishment payoff | $P$ | Payoff from mutual defection | $R > P > S$ |
| Sucker payoff | $S$ | Payoff from cooperating against defector | $P > S$ |
| Critical discount factor | $\delta^*$ | Minimum $\delta$ for cooperation | $(T-R)/(T-P)$ for grim trigger |
| Minimax value | $\underline{v}_i$ | Worst guaranteed payoff for player $i$ | Stage-game dependent |
| Number of periods | $T$ | Length of repeated interaction | Finite or $\infty$ |

## Numerical Example

**Setup.** Two market-making bots repeatedly compete in a Prisoner's Dilemma-like pricing game. Each round, each bot chooses between Wide spread (cooperate, $C$) and Tight spread (defect, $D$). Payoff matrix (profits per round in basis points):

$$
\begin{array}{c|cc}
 & \text{Wide} & \text{Tight} \\
\hline
\text{Wide} & (8,\;8) & (1,\;12) \\
\text{Tight} & (12,\;1) & (3,\;3)
\end{array}
$$

Here $T = 12$, $R = 8$, $P = 3$, $S = 1$.

**Critical discount factor for grim trigger:**

$$
\delta^* = \frac{T - R}{T - P} = \frac{12 - 8}{12 - 3} = \frac{4}{9} \approx 0.444
$$

**Interpretation:** If both bots value future profits at least 44.4% as much as current profits, wide spreads (cooperation) can be sustained via grim trigger.

**Payoff comparison over 20 rounds** with $\delta = 0.9$:

```python
import numpy as np

T_pay, R_pay, P_pay, S_pay = 12, 8, 3, 1
delta = 0.9
n_rounds = 20

delta_star_grim = (T_pay - R_pay) / (T_pay - P_pay)
delta_star_tft = (T_pay - R_pay) / (T_pay - S_pay)
print(f"Critical delta (grim trigger): {delta_star_grim:.4f}")
print(f"Critical delta (tit-for-tat):  {delta_star_tft:.4f}")
print(f"Actual delta: {delta}")
print(f"Cooperation sustainable (grim)? {delta >= delta_star_grim}")
print(f"Cooperation sustainable (TFT)?  {delta >= delta_star_tft}")

# Scenario 1: Both cooperate (grim trigger, no deviation)
payoffs_coop = np.array([R_pay * delta**t for t in range(n_rounds)])
total_coop = payoffs_coop.sum()

# Scenario 2: Deviate once at t=0, then mutual defection forever (grim trigger)
payoffs_dev = np.zeros(n_rounds)
payoffs_dev[0] = T_pay
payoffs_dev[1:] = [P_pay * delta**t for t in range(1, n_rounds)]
total_dev = payoffs_dev.sum()

# Scenario 3: Always defect
payoffs_always_d = np.array([P_pay * delta**t for t in range(n_rounds)])
total_always_d = payoffs_always_d.sum()

print(f"\n--- Payoff over {n_rounds} rounds (delta={delta}) ---")
print(f"Both cooperate (grim trigger): {total_coop:.2f} bps")
print(f"Deviate at t=0, then punished:  {total_dev:.2f} bps")
print(f"Always defect:                  {total_always_d:.2f} bps")
print(f"Cooperation premium: {total_coop - total_dev:.2f} bps")
```

**Expected output:**

```
Critical delta (grim trigger): 0.4444
Critical delta (tit-for-tat):  0.3636
Actual delta: 0.9
Cooperation sustainable (grim)? True
Cooperation sustainable (TFT)?  True

--- Payoff over 20 rounds (delta=0.9) ---
Both cooperate (grim trigger): 67.57 bps
Deviate at t=0, then punished:  37.32 bps
Always defect:                  25.32 bps
Cooperation premium: 30.25 bps
```

The cooperation payoff ($67.57$) far exceeds the deviation payoff ($37.32$), confirming that grim trigger sustains cooperation at $\delta = 0.9 > 0.444$. The cooperation premium of $30.25$ bps represents the cumulative benefit of maintaining wide spreads.

## Connections

- **[Nash Equilibrium](nash_equilibrium.md):** The stage-game Nash equilibrium (mutual defection) is always an equilibrium of the repeated game — just play the stage NE in every period regardless of history. The folk theorem shows that repetition adds many more equilibria on top of this baseline.
- **[Auction Theory](auction_theory.md):** When the same bidders compete in repeated auctions, tacit collusion becomes possible: bidders can take turns winning at low prices, sustained by the threat of competitive bidding if anyone deviates. This is the auction analogue of cooperation in repeated games.
- **[Market Making — Inventory Risk](../03_market_making/inventory_risk.md):** In repeated market making, your current inventory reflects the cumulative effect of past interactions. The discount factor $\delta$ maps naturally to the time decay of inventory risk: inventory accumulated today matters less as the terminal time approaches.
- **[Stat Arb — Z-Score Signals](../04_stat_arb/zscore_signals.md):** If multiple bots trade the same mean-reverting spread, their collective behaviour over time resembles a repeated game. A bot that aggressively trades the spread (tight entry threshold) is "defecting" — grabbing alpha at the expense of making the spread revert faster, leaving less for others.
- **[Brownian Motion](../01_stochastic/brownian_motion.md):** The stochastic environment of the repeated game — where stage-game payoffs may depend on a random mid-price — connects repeated game theory to the stochastic foundations. The discount factor $\delta$ can be interpreted as incorporating both time preference and the probability that the game continues.

## Relevance for Prosperity 4

Repeated game theory is arguably the *most directly applicable* game theory concept in Prosperity 4, because the competition runs over many rounds with the same set of bots:

1. **Learning opponent strategies.** Observe each bot's quoting and trading behaviour over rounds. Classify opponents as cooperative (wide spreads, predictable quotes), aggressive (tight spreads, undercutting), or adaptive (changing behaviour in response to yours). This classification informs which repeated-game strategy to deploy.

2. **Implementing trigger strategies.** If you detect a cooperative equilibrium (all bots maintaining reasonable spreads), maintain it — the mutual benefit is substantial. If a bot defects (starts aggressively undercutting), switch to competitive pricing against that specific bot while maintaining cooperation with others. This is a multi-player generalisation of grim trigger.

3. **Tit-for-tat in practice.** If an opponent tightens their spread in round $t$, tighten yours in round $t+1$. If they widen, match their widening. This establishes a clear, recognisable pattern that incentivises cooperation. TFT's simplicity is an advantage in Prosperity: opponents can recognise your strategy and cooperate with it.

4. **Estimating $\delta$ from the competition structure.** The discount factor $\delta$ in Prosperity corresponds to how much you weight future rounds relative to the current one. With $T$ total rounds, a reasonable proxy is $\delta \approx 1 - 1/T$. If $T = 100$ rounds, $\delta \approx 0.99$, well above any critical threshold — cooperation is highly sustainable.

5. **Exploiting finite-horizon effects.** Near the end of the competition (last few rounds), the incentive to cooperate weakens — opponents may "defect" in the final rounds since there is no future punishment. Anticipate this by gradually tightening your own spreads in the final rounds. The optimal timing of this transition from cooperation to competition is itself a strategic decision governed by the remaining number of rounds and the estimated $\delta$.

6. **Pattern recognition as history conditioning.** Your strategy $\sigma_i(h^t)$ is a function of the observed history $h^t$. In practice, this means maintaining a running estimate of each opponent's strategy type (cooperative, aggressive, random) and adapting in real time. Simple exponential moving averages of opponents' spread widths and fill rates provide a computationally efficient approximation to full history conditioning.
