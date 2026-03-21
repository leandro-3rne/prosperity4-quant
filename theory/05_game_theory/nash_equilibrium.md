# Nash Equilibrium

> **Core formula:** $$s^* = (s_1^*,\ldots,s_n^*) \text{ is a Nash equilibrium if } \forall\, i,\; \forall\, s_i \in S_i:\quad u_i(s_i^*,\, s_{-i}^*) \;\ge\; u_i(s_i,\, s_{-i}^*)$$

## Intuition

Imagine a crowded intersection with no traffic lights. Each driver must choose when to accelerate and when to yield. If every driver has settled on a pattern of behaviour such that no single driver can reduce their travel time by changing their own timing alone — everyone else's timing held fixed — then the system is in a Nash equilibrium. The equilibrium need not be "good" in any global sense; it simply means nobody has an individual incentive to deviate. The infamous gridlock that sometimes results is perfectly consistent with equilibrium: each driver is doing the best they can given everyone else's simultaneous choices.

In financial markets the same logic governs order-book competition. Multiple market makers simultaneously choose where to place their bid and ask quotes. A configuration of quotes is an equilibrium when no single maker can improve their expected profit by unilaterally moving their quotes — tighter quotes win more fills but earn thinner margins, while wider quotes are safer but rarely execute. Understanding Nash equilibrium lets you predict where rational competitors will quote, and therefore where you should quote relative to them.

The concept is named after John Nash, who proved in 1950 that every finite game possesses at least one such equilibrium (possibly requiring players to randomise). This existence guarantee is what makes the equilibrium concept so powerful: we know solutions exist, so the question is always "which equilibrium?" rather than "does one exist?".

## Mathematical Setup

A **normal-form game** is a triple

$$
G = \bigl(N,\; \{S_i\}_{i \in N},\; \{u_i\}_{i \in N}\bigr)
$$

where:

| Symbol | Meaning |
|--------|---------|
| $N = \{1, 2, \ldots, n\}$ | Finite set of players |
| $S_i$ | Strategy set (set of pure strategies) available to player $i$ |
| $S = S_1 \times S_2 \times \cdots \times S_n$ | Set of all strategy profiles |
| $s = (s_1, \ldots, s_n) \in S$ | A strategy profile — one strategy per player |
| $s_{-i} = (s_1, \ldots, s_{i-1}, s_{i+1}, \ldots, s_n)$ | Strategies of all players except $i$ |
| $(s_i, s_{-i})$ | Profile where player $i$ plays $s_i$ and others play $s_{-i}$ |
| $u_i : S \to \mathbb{R}$ | Player $i$'s payoff (utility) function |
| $\Delta(S_i)$ | Set of probability distributions (mixed strategies) over $S_i$ |
| $\sigma_i \in \Delta(S_i)$ | A mixed strategy for player $i$ |
| $\sigma_i(s_i)$ | Probability that player $i$ plays pure strategy $s_i$ under $\sigma_i$ |

**Notation convention:** We write $(s_i, s_{-i})$ to emphasise player $i$'s choice while holding others fixed. When player $i$ deviates from a profile $s^*$ by switching from $s_i^*$ to some alternative $s_i'$, the resulting profile is $(s_i', s_{-i}^*)$.

## Derivation

### 1. Nash Equilibrium — Definition

A strategy profile $s^* = (s_1^*, \ldots, s_n^*)$ is a **Nash equilibrium** (NE) if no player can strictly improve their payoff by a unilateral deviation:

$$
\forall\, i \in N, \quad \forall\, s_i \in S_i: \qquad u_i(s_i^*,\, s_{-i}^*) \;\ge\; u_i(s_i,\, s_{-i}^*).
$$

Equivalently, each player's strategy is a best response to the others:

$$
s_i^* \;\in\; \arg\max_{s_i \in S_i}\; u_i(s_i,\, s_{-i}^*) \qquad \forall\, i \in N.
$$

Define the **best-response correspondence** for player $i$:

$$
BR_i(s_{-i}) \;=\; \bigl\{s_i \in S_i : u_i(s_i, s_{-i}) \ge u_i(s_i', s_{-i}) \;\;\forall\, s_i' \in S_i\bigr\}.
$$

Then $s^*$ is a NE if and only if $s_i^* \in BR_i(s_{-i}^*)$ for every $i$.

### 2. Dominant Strategies and Iterated Elimination

**Strict dominance.** A strategy $s_i \in S_i$ is **strictly dominated** if there exists another strategy $s_i' \in S_i$ such that

$$
u_i(s_i',\, s_{-i}) \;>\; u_i(s_i,\, s_{-i}) \qquad \forall\, s_{-i} \in S_{-i}.
$$

A rational player will never play a strictly dominated strategy. If $s_i$ is strictly dominated, we can remove it from $S_i$ without affecting any NE.

**Iterated Elimination of Dominated Strategies (IEDS).** Repeatedly remove all strictly dominated strategies for each player. At each round, the reduced game may reveal new dominated strategies (a strategy that was not dominated before may become dominated once opponents' dominated strategies are removed). Continue until no further elimination is possible. Key properties:

1. **Order independence:** The set of surviving strategies does not depend on the order of elimination.
2. **NE preservation:** Every NE of the original game survives IEDS. Conversely, every NE of the reduced game is a NE of the original.
3. **Dominance solvability:** If IEDS reduces each player's strategy set to a singleton, the game is **dominance-solvable** and that unique surviving profile is the unique NE.

### 3. Finding NE in 2×2 Games — The Prisoner's Dilemma

Consider the Prisoner's Dilemma with payoff matrix (Player 1 chooses the row, Player 2 the column):

$$
\begin{array}{c|cc}
 & C & D \\
\hline
C & (3,\;3) & (0,\;5) \\
D & (5,\;0) & (1,\;1)
\end{array}
$$

**Step 1 — Check for dominated strategies.**

For Player 1, compare the two rows:

- If P2 plays $C$: $u_1(D, C) = 5 > 3 = u_1(C, C)$.
- If P2 plays $D$: $u_1(D, D) = 1 > 0 = u_1(C, D)$.

So $D$ strictly dominates $C$ for Player 1 regardless of Player 2's choice. By symmetry, $D$ also strictly dominates $C$ for Player 2. The game is dominance-solvable.

**Step 2 — Verify by checking each cell for NE.**

A cell $(s_1, s_2)$ is a NE if neither player can improve by unilateral deviation.

**Cell $(C, C)$: payoff $(3, 3)$.**

- P1 deviation $C \to D$: payoff changes $3 \to 5$. Strict improvement. So $(C, C)$ is **not** a NE.

**Cell $(C, D)$: payoff $(0, 5)$.**

- P1 deviation $C \to D$: payoff changes $0 \to 1$. Strict improvement. So $(C, D)$ is **not** a NE.

**Cell $(D, C)$: payoff $(5, 0)$.**

- P2 deviation $C \to D$: payoff changes $0 \to 1$. Strict improvement. So $(D, C)$ is **not** a NE.

**Cell $(D, D)$: payoff $(1, 1)$.**

- P1 deviation $D \to C$: payoff changes $1 \to 0$. Worse. No incentive to deviate.
- P2 deviation $D \to C$: payoff changes $1 \to 0$. Worse. No incentive to deviate.

Therefore $(D, D)$ is the **unique Nash equilibrium**, yielding payoffs $(1, 1)$.

Note the tragic structure: both players would prefer $(C, C)$ with payoff $(3, 3)$, but the equilibrium is $(D, D)$ with payoff $(1, 1)$. Individual rationality leads to a collectively suboptimal outcome.

### 4. Nash's Existence Theorem (1950)

**Theorem (Nash, 1950).** Every finite game — a game with a finite number of players $n$ and a finite strategy set $S_i$ for each player — has at least one Nash equilibrium in mixed strategies.

**Mixed strategies.** A mixed strategy for player $i$ is a probability distribution $\sigma_i \in \Delta(S_i)$ over their pure strategies. Player $i$'s expected payoff under mixed profile $\sigma = (\sigma_1, \ldots, \sigma_n)$ is

$$
U_i(\sigma) \;=\; \sum_{s \in S} \left(\prod_{j=1}^{n} \sigma_j(s_j)\right) u_i(s).
$$

A mixed strategy NE is a profile $\sigma^*$ satisfying

$$
U_i(\sigma_i^*,\, \sigma_{-i}^*) \;\ge\; U_i(\sigma_i,\, \sigma_{-i}^*) \qquad \forall\, i,\;\; \forall\, \sigma_i \in \Delta(S_i).
$$

**Proof sketch.** Define the combined mixed-strategy space $\Sigma = \Delta(S_1) \times \cdots \times \Delta(S_n)$. This is a non-empty, compact, convex subset of Euclidean space. Define the best-response correspondence $BR: \Sigma \rightrightarrows \Sigma$ by

$$
BR(\sigma) = BR_1(\sigma_{-1}) \times \cdots \times BR_n(\sigma_{-n}).
$$

One can verify that $BR$ satisfies the hypotheses of **Kakutani's fixed-point theorem**: $\Sigma$ is non-empty, compact, and convex; $BR(\sigma)$ is non-empty and convex for every $\sigma$ (since any convex combination of best responses is again a best response when payoffs are linear in own mixing probabilities); and $BR$ has a closed graph (upper hemicontinuity follows from continuity of the expected payoff functions). By Kakutani's theorem, there exists a fixed point $\sigma^* \in BR(\sigma^*)$, meaning $\sigma_i^* \in BR_i(\sigma_{-i}^*)$ for all $i$. This fixed point is a Nash equilibrium. $\blacksquare$

### 5. Mixed Strategy Nash Equilibrium — Matching Pennies

Consider the Matching Pennies game:

$$
\begin{array}{c|cc}
 & H & T \\
\hline
H & (+1,\;-1) & (-1,\;+1) \\
T & (-1,\;+1) & (+1,\;-1)
\end{array}
$$

This is a zero-sum game. Player 1 (the "Matcher") wins if both choose the same side; Player 2 (the "Mismatcher") wins if they differ.

**Step 1 — Verify no pure strategy NE exists.**

- $(H, H)$: P2 gets $-1$, deviates to $T$ for $+1$. Not a NE.
- $(H, T)$: P1 gets $-1$, deviates to $T$ for $+1$. Not a NE.
- $(T, H)$: P1 gets $-1$, deviates to $H$ for $+1$. Not a NE.
- $(T, T)$: P2 gets $-1$, deviates to $H$ for $+1$. Not a NE.

No pure strategy NE exists, so any NE must involve mixing.

**Step 2 — Find mixed strategy NE.**

Let Player 1 play $H$ with probability $p$ and $T$ with probability $1-p$.
Let Player 2 play $H$ with probability $q$ and $T$ with probability $1-q$.

**Indifference condition for Player 2.** In a mixed NE, Player 2 must be indifferent between $H$ and $T$ (otherwise they would strictly prefer one and play it with probability 1). Player 2's expected payoff from each pure strategy, given Player 1's mix:

$$
U_2(H) = p \cdot (-1) + (1-p) \cdot (+1) = -p + 1 - p = 1 - 2p
$$

$$
U_2(T) = p \cdot (+1) + (1-p) \cdot (-1) = p - 1 + p = 2p - 1
$$

Set $U_2(H) = U_2(T)$:

$$
1 - 2p = 2p - 1
$$

$$
2 = 4p
$$

$$
p = \frac{1}{2}
$$

**Indifference condition for Player 1.** By the same logic, Player 1's expected payoff from each pure strategy, given Player 2's mix:

$$
U_1(H) = q \cdot (+1) + (1-q) \cdot (-1) = q - 1 + q = 2q - 1
$$

$$
U_1(T) = q \cdot (-1) + (1-q) \cdot (+1) = -q + 1 - q = 1 - 2q
$$

Set $U_1(H) = U_1(T)$:

$$
2q - 1 = 1 - 2q
$$

$$
4q = 2
$$

$$
q = \frac{1}{2}
$$

**The unique Nash equilibrium** is the mixed profile $\sigma^* = \bigl((\tfrac{1}{2}, \tfrac{1}{2}),\; (\tfrac{1}{2}, \tfrac{1}{2})\bigr)$, where both players randomise uniformly. Each player's expected payoff is:

$$
U_1(\sigma^*) = \frac{1}{2}\cdot\frac{1}{2}\cdot(+1) + \frac{1}{2}\cdot\frac{1}{2}\cdot(-1) + \frac{1}{2}\cdot\frac{1}{2}\cdot(-1) + \frac{1}{2}\cdot\frac{1}{2}\cdot(+1) = 0
$$

$$
U_2(\sigma^*) = 0
$$

**Key insight:** In a mixed NE, each player's mixing probabilities are chosen to make the *opponent* indifferent, not to maximise one's own payoff directly. Player 1 choosing $p = 1/2$ is what makes Player 2 willing to randomise.

### 6. General Method for 2×2 Mixed NE

For a general 2-player, 2-strategy game with payoff matrix:

$$
\begin{array}{c|cc}
 & L & R \\
\hline
U & (a_1, a_2) & (b_1, b_2) \\
D & (c_1, c_2) & (d_1, d_2)
\end{array}
$$

Let Player 1 play $U$ with probability $p$ and $D$ with probability $1-p$.
Let Player 2 play $L$ with probability $q$ and $R$ with probability $1-q$.

Player 2's indifference (determines $p$):

$$
p \cdot a_2 + (1-p) \cdot c_2 = p \cdot b_2 + (1-p) \cdot d_2
$$

$$
p(a_2 - c_2) + c_2 = p(b_2 - d_2) + d_2
$$

$$
p\bigl[(a_2 - c_2) - (b_2 - d_2)\bigr] = d_2 - c_2
$$

$$
p = \frac{d_2 - c_2}{(a_2 - c_2) - (b_2 - d_2)}
$$

Player 1's indifference (determines $q$):

$$
q = \frac{d_1 - b_1}{(a_1 - b_1) - (c_1 - d_1)}
$$

A mixed NE exists (with both players genuinely mixing) when $p, q \in (0, 1)$.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Number of players | $n$ | Size of $N$ | 2 – 10 in stylised games; effectively large in order-book competition |
| Strategy set | $S_i$ | Available actions for player $i$ | Finite (discrete prices/quantities) in Prosperity |
| Payoff function | $u_i$ | Maps strategy profiles to real-valued rewards | Depends on market structure |
| Mixing probability | $\sigma_i(s)$ | Probability player $i$ plays pure strategy $s$ | $[0, 1]$, must sum to 1 |
| Best response | $BR_i$ | Optimal strategy set given opponents' play | Subset of $S_i$ or $\Delta(S_i)$ |

## Numerical Example

**Battle of the Sexes.** Two players simultaneously choose between Opera ($O$) and Football ($F$):

$$
\begin{array}{c|cc}
 & O & F \\
\hline
O & (3,\;2) & (0,\;0) \\
F & (0,\;0) & (2,\;3)
\end{array}
$$

**Pure strategy NE:** Check each cell.

- $(O, O)$: P1 deviates $O \to F$: $3 \to 0$. No. P2 deviates $O \to F$: $2 \to 0$. No. **NE.**
- $(O, F)$: P1 deviates $O \to F$: $0 \to 2$. Yes. Not a NE.
- $(F, O)$: P1 deviates $F \to O$: $0 \to 3$. Yes. Not a NE.
- $(F, F)$: P1 deviates $F \to O$: $2 \to 0$. No. P2 deviates $F \to O$: $3 \to 0$. No. **NE.**

Two pure NE: $(O, O)$ and $(F, F)$.

**Mixed NE.** Using the formula from Section 6:

Player 2's indifference determines $p$ (probability P1 plays $O$):

$$
p = \frac{d_2 - c_2}{(a_2 - c_2) - (b_2 - d_2)} = \frac{3 - 0}{(2 - 0) - (0 - 3)} = \frac{3}{2 + 3} = \frac{3}{5}
$$

Player 1's indifference determines $q$ (probability P2 plays $O$):

$$
q = \frac{d_1 - b_1}{(a_1 - b_1) - (c_1 - d_1)} = \frac{2 - 0}{(3 - 0) - (0 - 2)} = \frac{2}{3 + 2} = \frac{2}{5}
$$

**Verification.** Player 1's expected payoff from $O$, given $q = 2/5$:

$$
U_1(O) = \frac{2}{5}\cdot 3 + \frac{3}{5}\cdot 0 = \frac{6}{5}
$$

Player 1's expected payoff from $F$, given $q = 2/5$:

$$
U_1(F) = \frac{2}{5}\cdot 0 + \frac{3}{5}\cdot 2 = \frac{6}{5}
$$

Indifference confirmed: $U_1(O) = U_1(F) = 6/5$.

```python
import numpy as np

A1 = np.array([[3, 0],
               [0, 2]])
A2 = np.array([[2, 0],
               [0, 3]])

p = (A2[1,1] - A2[1,0]) / ((A2[0,0] - A2[1,0]) - (A2[0,1] - A2[1,1]))
q = (A1[1,1] - A1[0,1]) / ((A1[0,0] - A1[0,1]) - (A1[1,0] - A1[1,1]))

print(f"Mixed NE: P1 plays O with prob p = {p:.4f} = 3/5")
print(f"          P2 plays O with prob q = {q:.4f} = 2/5")

sigma1 = np.array([p, 1-p])
sigma2 = np.array([q, 1-q])

EU1 = sigma1 @ A1 @ sigma2
EU2 = sigma1 @ A2 @ sigma2
print(f"Expected payoffs: U1 = {EU1:.4f}, U2 = {EU2:.4f}")

# Verify indifference
U1_O = A1[0] @ sigma2
U1_F = A1[1] @ sigma2
print(f"P1 indifference check: U1(O)={U1_O:.4f}, U1(F)={U1_F:.4f}")
```

**Output:**

```
Mixed NE: P1 plays O with prob p = 0.6000 = 3/5
          P2 plays O with prob q = 0.4000 = 2/5
Expected payoffs: U1 = 1.2000, U2 = 1.2000
P1 indifference check: U1(O)=1.2000, U1(F)=1.2000
```

The game has three Nash equilibria: two pure $\{(O,O), (F,F)\}$ and one mixed $\{(3/5, 2/5)\}$. The mixed NE gives expected payoff $6/5 = 1.2$ to each player, which is worse than either pure NE — mixing introduces coordination failures.

## Connections

- **[Auction Theory](auction_theory.md):** Auctions are games, and the optimal bidding strategies derived there (e.g., $b(v) = v \cdot (N-1)/N$ in first-price auctions) are Nash equilibria of the auction game. Revenue equivalence compares the equilibria of different auction formats.
- **[Repeated Games](repeated_games.md):** A Nash equilibrium of the stage game is always a NE of the repeated game (play the stage NE every period). The folk theorem shows that repetition unlocks additional equilibria — including cooperative outcomes that are not NE of the one-shot game.
- **[Avellaneda–Stoikov Market Making](../03_market_making/avellaneda_stoikov.md):** When multiple market makers simultaneously optimise their quotes, the resulting quote configuration is a Nash equilibrium of the market-making game. The AS model gives each maker's best response given a model of order flow, which implicitly assumes opponents' quotes are fixed.
- **[Ornstein–Uhlenbeck / Mean Reversion](../04_stat_arb/ornstein_uhlenbeck.md):** If multiple stat-arb bots trade the same mean-reverting spread, their collective actions affect the speed of reversion. The equilibrium of this "crowded trade" game determines how much alpha remains.

## Relevance for Prosperity 4

In IMC Prosperity 4, you compete against multiple algorithmic bots on a shared order book. Nash equilibrium analysis applies directly in several ways:

1. **Quote placement as a strategic game.** Each bot simultaneously decides where to place bid/ask quotes. If you model each bot as choosing a spread width $\delta_i$, the payoff is a function of all spreads: tighter spread → more fills but thinner margin and greater adverse selection risk. The equilibrium spread is the one where no bot wants to unilaterally widen or tighten — this is a Nash equilibrium of the quoting game.

2. **Predicting competitor behaviour.** If you estimate opponent bots' payoff functions from observed behaviour (fill rates, quote positions), you can compute the Nash equilibrium of the inferred game and anticipate their future quotes. This is strictly more powerful than naive pattern matching because it accounts for the fact that opponents will also adapt to your changes.

3. **Dominant strategy identification via IEDS.** In some Prosperity rounds, certain strategies are strictly dominated (e.g., quoting a negative spread, or quoting so wide that you never fill). Eliminating these iteratively can narrow down the plausible opponent strategy set, simplifying your best-response calculation.

4. **Mixed strategies for unpredictability.** If your quoting pattern is deterministic, sophisticated opponents can exploit it. Mixing over a set of quote placements (analogous to a mixed NE) makes your strategy robust against adversarial opponents who try to learn and counter your behaviour.
