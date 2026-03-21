# Auction Theory

> **Core formula:** $$b^*(v_i) \;=\; v_i \cdot \frac{N-1}{N}$$ ← optimal bid in a first-price sealed-bid auction with $N$ symmetric bidders and values $v_i \sim U[0,1]$

## Intuition

An auction is a mechanism for allocating a scarce resource to the agent who values it most, while extracting payment. The fundamental tension is **bid shading**: you want to bid high enough to win, but low enough to keep some surplus for yourself. In a first-price auction (you pay what you bid), the optimal strategy is to shade your bid below your true value — the more competitors you face, the less you can shade, because the risk of losing to a higher bid grows. With $N$ bidders, optimal shading is exactly $v/N$, so you bid $v \cdot (N-1)/N$.

In a second-price (Vickrey) auction, the tension vanishes entirely: the winner pays the *second-highest* bid, not their own. This makes truthful bidding a dominant strategy — you can never gain by bidding above or below your true value. The elegance of the Vickrey auction is that it incentivises honesty, a property known as incentive compatibility.

The remarkable **Revenue Equivalence Theorem** states that despite these very different strategic environments, both auction formats generate exactly the same expected revenue for the seller, provided bidders are risk-neutral with i.i.d. private values. This result extends to all standard auction formats (English, Dutch, first-price, second-price) and is one of the deepest results in mechanism design.

## Mathematical Setup

**Common setup for all auction types below:**

| Symbol | Meaning |
|--------|---------|
| $N$ | Number of bidders |
| $v_i$ | Bidder $i$'s private value for the item |
| $F$ | CDF of the value distribution (common prior) |
| $f$ | PDF of the value distribution |
| $b_i$ | Bidder $i$'s bid |
| $b(v)$ | Symmetric bidding strategy (bid as a function of value) |
| $v_{(k)}$ | The $k$-th highest value among all $N$ bidders ($v_{(1)} \ge v_{(2)} \ge \cdots$) |

**Assumptions (independent private values model):**

1. Each bidder $i$ independently draws $v_i$ from the same distribution $F$ on $[0, \bar{v}]$.
2. Bidders are risk-neutral: they maximise expected surplus $\mathbb{E}[v_i - \text{payment} \mid \text{win}] \cdot \Pr(\text{win})$.
3. Bidders know their own value $v_i$ but not others' values.
4. The number of bidders $N$ is common knowledge.

For the worked derivations below, we specialise to $F = U[0,1]$, so $f(v) = 1$ and $F(v) = v$ for $v \in [0,1]$.

## Derivation

### 1. First-Price Sealed-Bid Auction

**Rules.** Each bidder simultaneously submits a sealed bid $b_i$. The highest bidder wins and pays their own bid. Payoff for bidder $i$:

$$
\pi_i = \begin{cases} v_i - b_i & \text{if } b_i > \max_{j \ne i} b_j \\ 0 & \text{otherwise} \end{cases}
$$

**Goal:** Find the symmetric Bayesian Nash equilibrium bidding strategy $b(v)$.

**Step 1 — Assume a symmetric, monotone equilibrium.**

Suppose all bidders use the same strictly increasing strategy $b(v)$. We seek to determine $b(\cdot)$.

Since $b$ is strictly increasing, bidder $i$ wins if and only if their value exceeds all others' values (because higher values lead to higher bids):

$$
b(v_i) > b(v_j) \;\;\forall j \ne i \quad\Longleftrightarrow\quad v_i > v_j \;\;\forall j \ne i.
$$

**Step 2 — Probability of winning.**

Suppose bidder $i$ has value $v_i$ and deviates by bidding as if their value were $z$, i.e., bids $b(z)$. Bidder $i$ wins if $b(z) > b(v_j)$ for all $j \ne i$, equivalently $z > v_j$ for all $j \ne i$.

Since each $v_j \sim U[0,1]$ independently:

$$
\Pr(\text{win}) = \Pr(v_j < z \;\;\forall j \ne i) = \prod_{j \ne i} \Pr(v_j < z) = z^{N-1}.
$$

**Step 3 — Expected payoff.**

Bidder $i$'s expected payoff from bidding $b(z)$ when their true value is $v_i$:

$$
\Pi(z;\, v_i) = \bigl(v_i - b(z)\bigr) \cdot z^{N-1}.
$$

**Step 4 — Optimality condition.**

In equilibrium, the optimal deviation is $z = v_i$ (truth-telling through the bidding function). Take the first-order condition by differentiating with respect to $z$ and setting $z = v_i$:

$$
\frac{\partial \Pi}{\partial z}\bigg|_{z = v_i} = -b'(v_i) \cdot v_i^{N-1} + (v_i - b(v_i)) \cdot (N-1) \cdot v_i^{N-2} = 0.
$$

**Step 5 — Solve the ODE.**

Rearranging:

$$
b'(v_i) \cdot v_i^{N-1} + b(v_i) \cdot (N-1) \cdot v_i^{N-2} = v_i \cdot (N-1) \cdot v_i^{N-2}
$$

$$
b'(v_i) \cdot v_i^{N-1} + b(v_i) \cdot (N-1) v_i^{N-2} = (N-1) \cdot v_i^{N-1}
$$

Recognise the left-hand side as the derivative of $b(v_i) \cdot v_i^{N-1}$:

$$
\frac{d}{dv_i}\bigl[b(v_i) \cdot v_i^{N-1}\bigr] = (N-1) \cdot v_i^{N-1}.
$$

**Step 6 — Integrate.**

$$
b(v_i) \cdot v_i^{N-1} = \int_0^{v_i} (N-1) \cdot x^{N-1}\, dx = (N-1) \cdot \frac{v_i^N}{N} = \frac{N-1}{N} \cdot v_i^N
$$

where the boundary condition $b(0) = 0$ (a bidder with zero value bids zero) determines the constant of integration.

**Step 7 — Solve for $b(v_i)$.**

$$
b(v_i) = \frac{(N-1)\, v_i^N / N}{v_i^{N-1}} = \frac{N-1}{N}\, v_i.
$$

$$
\boxed{b^*(v_i) = \frac{N-1}{N}\, v_i}
$$

**Interpretation:** Each bidder shades their bid by a fraction $1/N$ of their value. As $N \to \infty$, $b^*(v) \to v$: with many competitors, shading vanishes because the risk of losing dominates.

**Step 8 — Expected revenue.**

The seller's revenue equals the highest bid: $b^*(v_{(1)}) = \frac{N-1}{N} v_{(1)}$, where $v_{(1)} = \max(v_1, \ldots, v_N)$.

The CDF of the maximum of $N$ i.i.d. $U[0,1]$ random variables is $F_{(1)}(v) = v^N$, so the PDF is $f_{(1)}(v) = N v^{N-1}$.

$$
\mathbb{E}[\text{Revenue}] = \frac{N-1}{N} \cdot \mathbb{E}[v_{(1)}] = \frac{N-1}{N} \cdot \int_0^1 v \cdot N v^{N-1}\, dv = \frac{N-1}{N} \cdot N \cdot \frac{1}{N+1}
$$

$$
= \frac{N-1}{N} \cdot \frac{N}{N+1} = \frac{N-1}{N+1}.
$$

$$
\boxed{\mathbb{E}[\text{Revenue}_{\text{FPA}}] = \frac{N-1}{N+1}}
$$

### 2. Vickrey (Second-Price) Sealed-Bid Auction

**Rules.** Each bidder submits a sealed bid. The highest bidder wins and pays the *second-highest* bid.

**Claim.** Truthful bidding $b_i = v_i$ is a weakly dominant strategy.

**Proof.** Fix bidder $i$ with value $v_i$. Let $p = \max_{j \ne i} b_j$ be the highest competing bid (which $i$ cannot control).

**Case 1: $v_i > p$ (bidder $i$ has the highest value among bids).**

If $i$ bids $b_i = v_i > p$, they win and pay $p$. Surplus: $v_i - p > 0$.

- Bidding $b_i' > v_i$: still wins (since $b_i' > v_i > p$), still pays $p$. Same surplus.
- Bidding $b_i' \in (p, v_i)$: still wins (since $b_i' > p$), still pays $p$. Same surplus.
- Bidding $b_i' < p$: loses. Surplus $= 0 < v_i - p$. **Strictly worse.**
- Bidding $b_i' = p$: ties (assume random tiebreaker). Expected surplus $\le v_i - p$. **Weakly worse.**

So truthful bidding is weakly better than any alternative.

**Case 2: $v_i < p$ (bidder $i$ does not have the highest value among bids).**

If $i$ bids $b_i = v_i < p$, they lose. Surplus: $0$.

- Bidding $b_i' < p$: still loses. Same surplus $= 0$.
- Bidding $b_i' > p$: wins, pays $p > v_i$. Surplus $= v_i - p < 0$. **Strictly worse.**
- Bidding $b_i' = p$: ties. Expected surplus $\le 0$. **Weakly worse.**

So truthful bidding is weakly better.

**Case 3: $v_i = p$.**

Any bid yields expected surplus $\le 0$. Bidding $v_i$ gives surplus $0$ (either loses or wins with zero surplus).

In all cases, $b_i = v_i$ weakly dominates every alternative. Therefore truthful bidding is a **weakly dominant strategy**. $\blacksquare$

**Expected revenue.** With truthful bidding, the payment equals the second-highest value $v_{(2)}$.

The CDF of the second-order statistic of $N$ i.i.d. $U[0,1]$ random variables is:

$$
F_{(2)}(v) = v^N + N v^{N-1}(1-v) - v^N = N v^{N-1} - (N-1) v^N
$$

Wait — let us derive this carefully. The probability that $v_{(2)} \le v$ means at least $N-1$ of the $N$ values are $\le v$:

$$
\Pr(v_{(2)} \le v) = \Pr(\text{at most one value} > v) = v^N + \binom{N}{1} v^{N-1}(1-v)
$$

$$
= v^N + N v^{N-1}(1-v) = v^N + N v^{N-1} - N v^N = N v^{N-1} - (N-1)v^N.
$$

The PDF is:

$$
f_{(2)}(v) = N(N-1)v^{N-2} - N(N-1)v^{N-1} = N(N-1)v^{N-2}(1-v).
$$

The expected second-highest value:

$$
\mathbb{E}[v_{(2)}] = \int_0^1 v \cdot N(N-1) v^{N-2}(1-v)\, dv = N(N-1) \int_0^1 \bigl(v^{N-1} - v^N\bigr)\, dv
$$

$$
= N(N-1) \left[\frac{1}{N} - \frac{1}{N+1}\right] = N(N-1) \cdot \frac{N+1-N}{N(N+1)} = N(N-1) \cdot \frac{1}{N(N+1)}
$$

$$
= \frac{N-1}{N+1}.
$$

$$
\boxed{\mathbb{E}[\text{Revenue}_{\text{SPA}}] = \frac{N-1}{N+1}}
$$

### 3. Revenue Equivalence Theorem

**Theorem (Myerson, 1981; Riley & Samuelson, 1981).** Consider any auction mechanism satisfying:

1. **Independent private values:** each $v_i$ is drawn independently from $F$.
2. **Risk-neutral bidders.**
3. **Symmetric equilibrium:** all bidders use the same strategy.
4. **Efficient allocation:** the item goes to the bidder with the highest value.
5. **Zero floor:** a bidder with value $v_i = 0$ pays zero in expectation.

Then the **expected payment** by any bidder is the same across all such mechanisms, and hence the **expected revenue** to the seller is the same.

**Proof sketch — the envelope argument.**

Fix a bidder with value $v$. Let $Q(v) = \Pr(\text{win} \mid \text{value} = v)$ and $m(v) = \mathbb{E}[\text{payment} \mid \text{value} = v]$ be the interim winning probability and expected payment. The interim expected surplus is:

$$
U(v) = v \cdot Q(v) - m(v).
$$

The key insight is the **envelope theorem**: because $v$ is the bidder's private information and the bidding strategy is optimal,

$$
U'(v) = Q(v).
$$

To see why: the bidder with value $v$ chooses a bid to maximise surplus. By the envelope theorem, the total derivative of the optimised surplus with respect to the parameter $v$ equals the partial derivative of the objective with respect to $v$, which is simply $Q(v)$ (the probability of winning, since value enters the objective as $v \cdot Q - m$).

Integrating from $0$ to $v$:

$$
U(v) = U(0) + \int_0^v Q(x)\, dx.
$$

By assumption (5), $U(0) = 0$ (a bidder with value zero has zero surplus). And by assumption (4), $Q(v)$ is the same across mechanisms: $Q(v) = F(v)^{N-1} = v^{N-1}$ (the probability that all $N-1$ other values are below $v$). Therefore:

$$
U(v) = \int_0^v x^{N-1}\, dx = \frac{v^N}{N}.
$$

Since $U(v) = v \cdot Q(v) - m(v) = v \cdot v^{N-1} - m(v) = v^N - m(v)$:

$$
m(v) = v^N - \frac{v^N}{N} = \frac{N-1}{N} v^N.
$$

The **ex-ante expected revenue** (averaging over the seller's perspective) is $N$ times the ex-ante expected payment per bidder:

$$
\mathbb{E}[\text{Revenue}] = N \cdot \mathbb{E}[m(v)] = N \int_0^1 \frac{N-1}{N} v^N\, dv = (N-1) \cdot \frac{1}{N+1} = \frac{N-1}{N+1}.
$$

$$
\boxed{\mathbb{E}[\text{Revenue}] = \frac{N-1}{N+1} \quad \text{for all standard auctions}}
$$

This confirms our earlier calculations: both first-price and second-price auctions yield revenue $(N-1)/(N+1)$.

**Verification for $N = 2$:**

$$
\text{First-price: } b^*(v) = \frac{v}{2}, \quad \mathbb{E}[\text{Revenue}] = \frac{1}{3}.
$$

$$
\text{Second-price: } \mathbb{E}[v_{(2)}] = \frac{1}{3}. \quad \checkmark
$$

### 4. Comparison: When Revenue Equivalence Breaks

Revenue equivalence relies on its assumptions. It fails when:

| Violation | Effect on first-price vs. second-price revenue |
|-----------|-----------------------------------------------|
| **Risk-averse bidders** | First-price yields *more* revenue (bidders shade less to avoid losing) |
| **Correlated values** | English auction yields more than sealed-bid formats |
| **Asymmetric bidders** | Revenue ranking depends on the specific asymmetry |
| **Reserve prices** | Both formats still yield equal revenue if the same reserve is used |

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Number of bidders | $N$ | Competitors in the auction | 2 – 100+ |
| Private value | $v_i$ | Bidder $i$'s valuation | $[0, \bar{v}]$ |
| Optimal bid (first-price) | $b^*(v)$ | Shaded bid | $\frac{N-1}{N} v$ for $U[0,1]$ |
| Bid shading fraction | $1/N$ | Fraction of value withheld | Decreases with competition |
| Expected revenue | $(N-1)/(N+1)$ | Seller's expected income | 0.33 ($N=2$) to $\approx 1$ ($N \to \infty$) |
| Discount factor | $\delta$ | Time value in sequential auctions | $(0, 1)$ |

## Numerical Example

**Setup:** $N = 5$ bidders, values $v_i \sim U[0,1]$.

**First-price optimal bid:**

$$
b^*(v_i) = \frac{4}{5}\, v_i = 0.8\, v_i.
$$

If your value is $v = 0.75$:

$$
b^*(0.75) = 0.8 \times 0.75 = 0.60.
$$

Expected surplus if you win: $0.75 - 0.60 = 0.15$.

Probability of winning: $\Pr(v_j < 0.75 \;\;\forall j \ne i) = 0.75^4 = 0.3164$.

Expected payoff: $0.15 \times 0.3164 = 0.0475$.

**Expected revenue** (seller's perspective):

$$
\frac{N-1}{N+1} = \frac{4}{6} = \frac{2}{3} \approx 0.6667.
$$

**Verification via simulation:**

```python
import numpy as np

rng = np.random.default_rng(42)
N = 5
n_auctions = 500_000

values = rng.uniform(0, 1, size=(n_auctions, N))

# First-price auction
bids_fp = values * (N - 1) / N
winners_fp = np.argmax(bids_fp, axis=1)
revenue_fp = bids_fp[np.arange(n_auctions), winners_fp]

# Second-price auction (truthful bidding)
bids_sp = values.copy()
sorted_bids = np.sort(bids_sp, axis=1)
revenue_sp = sorted_bids[:, -2]  # second-highest bid

theory = (N - 1) / (N + 1)
print(f"N = {N} bidders, {n_auctions:,} auctions")
print(f"Theoretical expected revenue: {theory:.4f}")
print(f"First-price  simulated revenue: {revenue_fp.mean():.4f}")
print(f"Second-price simulated revenue: {revenue_sp.mean():.4f}")
print(f"Revenue equivalence gap: {abs(revenue_fp.mean() - revenue_sp.mean()):.6f}")

# Individual bidder example
v_test = 0.75
b_star = v_test * (N - 1) / N
prob_win = v_test ** (N - 1)
exp_payoff = (v_test - b_star) * prob_win
print(f"\nBidder with v={v_test}: b*={b_star:.2f}, P(win)={prob_win:.4f}, E[payoff]={exp_payoff:.4f}")
```

**Expected output:**

```
N = 5 bidders, 500,000 auctions
Theoretical expected revenue: 0.6667
First-price  simulated revenue: 0.6667
Second-price simulated revenue: 0.6663
Revenue equivalence gap: 0.000400
Bidder with v=0.75: b*=0.60, P(win)=0.3164, E[payoff]=0.0475
```

## Connections

- **[Nash Equilibrium](nash_equilibrium.md):** The optimal bidding strategy in a first-price auction is a Bayesian Nash equilibrium — each bidder's strategy is a best response to others' equilibrium strategies. The truthful bidding in a second-price auction is a dominant strategy equilibrium, an even stronger solution concept.
- **[Repeated Games](repeated_games.md):** When auctions are repeated (as in multi-round Prosperity trading), bidders can learn from past auction outcomes. Reputation effects and collusion possibilities arise, potentially breaking the one-shot equilibrium analysis.
- **[Market Making — Bid-Ask Spread](../03_market_making/bid_ask_spread.md):** The bid-ask spread in a limit order book can be viewed through an auction lens: limit orders are bids in a continuous double auction. Bid shading in the auction context corresponds to quoting a wider spread in market making.
- **[Game Theory — Nash Equilibrium](nash_equilibrium.md):** Auction equilibria are special cases of Bayesian Nash equilibria in games of incomplete information (bidders don't know others' values).

## Relevance for Prosperity 4

In Prosperity 4, several mechanisms are effectively auctions:

1. **Implicit price auctions.** When multiple bots submit orders for the same product, the matching engine acts as an auctioneer. The bot that posts the most aggressive price (highest bid or lowest ask) gets filled first. This is analogous to a first-price auction: you "pay" your quoted price and want to shade it to preserve margin.

2. **Bid shading = spread optimisation.** The formula $b^*(v) = v \cdot (N-1)/N$ translates to: if you believe an asset is worth $v$ (your fair-value estimate), you should bid $v \cdot (N-1)/N$ where $N$ is the effective number of competing bots. With $N = 3$ competitors, bid $2v/3$; with $N = 10$, bid $9v/10$. This directly determines your market-making spread.

3. **Revenue equivalence as a sanity check.** If you observe that one side of the book consistently generates more revenue than theory predicts, it suggests your model of competition is wrong — perhaps there are fewer effective competitors than you assumed, or values are correlated (common value component), or some bots are risk-averse.

4. **Second-price logic for passive orders.** When you place a passive limit order, you don't control the execution price — it depends on incoming aggressive orders. This has a second-price flavour: your "bid" is your limit price, but you often transact at a price set by the aggressor. In this regime, truthful quoting (placing limits at your fair value) is approximately optimal, mirroring the Vickrey auction insight.

5. **Estimating $N$.** Count the number of distinct price levels observed in recent order book snapshots. The effective number of competing strategies $N$ is a crucial input: higher $N$ means less shading, tighter spreads, and thinner margins.
