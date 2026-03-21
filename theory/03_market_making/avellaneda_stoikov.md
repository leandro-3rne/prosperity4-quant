# Avellaneda-Stoikov Optimal Market Making

> **Core formula — Reservation price:**
> $$r_t = S_t - q_t \, \gamma \, \sigma^2 (T - t)$$

## Intuition

A market maker continuously posts a bid price (to buy) and an ask price (to sell), hoping to earn the spread between the two. The fundamental tension is this: every time an order fills, the market maker acquires or disposes of inventory, and holding that inventory is risky because the mid-price can move against you. If you just bought a unit and the price drops, you lose money. The Avellaneda-Stoikov (2008) model solves for the optimal way to set bid and ask quotes that balances the revenue from spread capture against the cost of inventory risk.

The key insight is the **reservation price**: rather than quoting symmetrically around the current mid-price $S_t$, the market maker shifts the centre of the quotes toward the side that reduces inventory. If the market maker is long ($q > 0$), the reservation price sits below $S_t$, making the bid less aggressive and the ask more aggressive, thereby encouraging sells and discouraging buys. The strength of this inventory penalty is controlled by the risk-aversion parameter $\gamma$, the volatility $\sigma$, and the remaining time horizon $T - t$.

Think of it like a shopkeeper with perishable goods approaching closing time. If the shop is overstocked, the rational response is to lower prices to attract buyers and reduce the risk of holding unsold inventory overnight. The parameter $\gamma$ captures how nervous the shopkeeper is about leftover stock; high $\gamma$ means aggressive markdowns, low $\gamma$ means the shopkeeper is comfortable holding more inventory in exchange for potentially higher margins.

## Mathematical Setup

**State variables:**

| Symbol | Definition |
|--------|-----------|
| $S_t$ | Mid-price at time $t$, following arithmetic Brownian motion |
| $q_t$ | Inventory (signed integer): number of units held, positive = long, negative = short |
| $X_t$ | Cash process: cumulative cash from completed trades |
| $\delta^a_t$ | Ask distance: ask price is $S_t + \delta^a$ |
| $\delta^b_t$ | Bid distance: bid price is $S_t - \delta^b$ |
| $T$ | Terminal time (end of trading session) |

**Assumptions:**

1. **Mid-price dynamics** — no drift under the market maker's pricing measure:

$$dS_t = \sigma \, dW_t$$

where $W_t$ is a standard Brownian motion and $\sigma > 0$ is constant volatility.

2. **Order arrival** — market buy and sell orders arrive as independent Poisson processes. The fill intensity at distance $\delta$ from mid is:

$$\lambda(\delta) = A \exp(-\kappa \delta)$$

where $A > 0$ is the baseline arrival rate when quoting at mid ($\delta = 0$) and $\kappa > 0$ is the order-book depth parameter. This exponential decay captures the empirical observation that the further your quote sits from the mid-price, the less likely it is to be hit. The parameter $\kappa$ measures how rapidly fill probability drops with distance: large $\kappa$ corresponds to a thin order book where even small displacements drastically reduce fill probability; small $\kappa$ means a deep, liquid book.

**Justification of the Poisson model:** Each incoming market order either fills your resting limit order or does not. At high frequency, the number of fills in a short interval $[t, t+dt)$ is well approximated by a Poisson random variable with intensity $\lambda(\delta) \, dt$. The exponential form $A e^{-\kappa \delta}$ is the simplest parametric model consistent with the stylised fact that fill probability is monotonically decreasing and convex in quote distance.

3. **Cash process** — when the ask fills:

$$X_{t+dt} = X_t + (S_t + \delta^a)$$

when the bid fills:

$$X_{t+dt} = X_t - (S_t - \delta^b)$$

4. **Inventory process:**

$$q_{t+dt} = q_t - 1 \quad (\text{ask fill}), \qquad q_{t+dt} = q_t + 1 \quad (\text{bid fill})$$

5. **Objective** — maximise expected terminal utility with CARA (constant absolute risk aversion) preferences:

$$\max_{\delta^a, \delta^b} \; \mathbb{E}\!\left[-\exp\!\left(-\gamma \left(X_T + q_T S_T\right)\right)\right]$$

The argument of the exponential is $-($ risk aversion $) \times ($ terminal wealth $)$. Terminal wealth is cash $X_T$ plus the mark-to-market value $q_T S_T$ of remaining inventory, liquidated at the mid-price. The CARA utility $u(w) = -e^{-\gamma w}$ implies constant absolute risk aversion: the market maker treats a \$1 gain and a \$1 loss symmetrically regardless of wealth level.

## Derivation

### Step 1: Value Function and HJB Equation

Define the value function:

$$u(t, x, S, q) = \sup_{\delta^a, \delta^b} \; \mathbb{E}\!\left[-\exp\!\left(-\gamma(X_T + q_T S_T)\right) \;\Big|\; X_t = x, \, S_t = S, \, q_t = q\right]$$

Between order arrivals, the mid-price diffuses and no trades occur. Over an infinitesimal interval $dt$, three things can happen: (i) no fills — the mid-price evolves via $dS = \sigma \, dW$, (ii) the ask fills with probability $\lambda(\delta^a) \, dt$, or (iii) the bid fills with probability $\lambda(\delta^b) \, dt$.

Applying dynamic programming (the Hamilton-Jacobi-Bellman equation), we require $u$ to satisfy:

$$\frac{\partial u}{\partial t} + \frac{1}{2}\sigma^2 \frac{\partial^2 u}{\partial S^2} + \sup_{\delta^a}\!\Big[\lambda(\delta^a)\Big(u(t,\, x + S + \delta^a,\, S,\, q-1) - u(t,x,S,q)\Big)\Big] + \sup_{\delta^b}\!\Big[\lambda(\delta^b)\Big(u(t,\, x - S + \delta^b,\, S,\, q+1) - u(t,x,S,q)\Big)\Big] = 0$$

with terminal condition:

$$u(T, x, S, q) = -\exp\!\left(-\gamma(x + q S)\right)$$

The first two terms come from the diffusion of $S_t$ (no drift, variance $\sigma^2$). The last two terms come from the jump contributions of Poisson order fills on the ask and bid sides, weighted by their arrival intensities.

### Step 2: Exponential Ansatz

The CARA structure suggests an exponential separation. We guess:

$$u(t, x, S, q) = -\exp\!\left(-\gamma(x + q S)\right) \cdot w(t, q)$$

where $w(t,q)$ is a function to be determined, with terminal condition $w(T,q) = 1$ for all $q$.

**Computing the partial derivatives:**

Since $u = -e^{-\gamma(x + qS)} \cdot w(t,q)$, let $\phi = x + qS$ for brevity:

$$\frac{\partial u}{\partial t} = -e^{-\gamma \phi} \cdot \frac{\partial w}{\partial t}$$

$$\frac{\partial u}{\partial S} = -e^{-\gamma \phi} \cdot (-\gamma q) \cdot w = \gamma q \, e^{-\gamma \phi} \cdot w$$

$$\frac{\partial^2 u}{\partial S^2} = -\gamma^2 q^2 \, e^{-\gamma \phi} \cdot w$$

So the diffusion terms give:

$$\frac{\partial u}{\partial t} + \frac{1}{2}\sigma^2 \frac{\partial^2 u}{\partial S^2} = -e^{-\gamma \phi}\left(\frac{\partial w}{\partial t} + \frac{1}{2}\gamma^2 q^2 \sigma^2 w\right)$$

### Step 3: Substituting the Ask Fill Term

When the ask fills, cash increases by $S + \delta^a$ and inventory decreases by 1. The new exponent becomes:

$$-\gamma\big((x + S + \delta^a) + (q-1)S\big) = -\gamma\big(x + qS + \delta^a\big) = -\gamma(\phi + \delta^a)$$

Therefore:

$$u(t, x + S + \delta^a, S, q-1) = -e^{-\gamma(\phi + \delta^a)} \cdot w(t, q-1)$$

The ask jump contribution is:

$$\lambda(\delta^a)\Big(u(\ldots, q-1) - u(\ldots, q)\Big) = \lambda(\delta^a)\Big(-e^{-\gamma(\phi + \delta^a)} w(t,q-1) + e^{-\gamma\phi} w(t,q)\Big)$$

$$= e^{-\gamma\phi}\,\lambda(\delta^a)\Big(w(t,q) - e^{-\gamma\delta^a} w(t,q-1)\Big)$$

### Step 4: Substituting the Bid Fill Term

When the bid fills, cash decreases by $S - \delta^b$ and inventory increases by 1. The new exponent:

$$-\gamma\big((x - S + \delta^b) + (q+1)S\big) = -\gamma\big(x + qS + \delta^b\big) = -\gamma(\phi + \delta^b)$$

Therefore:

$$u(t, x - S + \delta^b, S, q+1) = -e^{-\gamma(\phi + \delta^b)} \cdot w(t, q+1)$$

The bid jump contribution is:

$$e^{-\gamma\phi}\,\lambda(\delta^b)\Big(w(t,q) - e^{-\gamma\delta^b} w(t,q+1)\Big)$$

### Step 5: Simplified HJB (Dividing by $-e^{-\gamma\phi}$)

Dividing the full HJB by $-e^{-\gamma\phi}$ (which is always negative, so we flip the optimisation signs back), we obtain:

$$\frac{\partial w}{\partial t} + \frac{1}{2}\gamma^2 q^2 \sigma^2 w - \sup_{\delta^a}\!\Big[\lambda(\delta^a)\Big(w(t,q) - e^{-\gamma\delta^a}w(t,q-1)\Big)\Big] - \sup_{\delta^b}\!\Big[\lambda(\delta^b)\Big(w(t,q) - e^{-\gamma\delta^b}w(t,q+1)\Big)\Big] = 0$$

Note: when we divide through by $-e^{-\gamma\phi}$, the $\sup$ operators remain $\sup$ because we factor out a negative sign twice (once from $u$ and once from the jump difference), leaving the net sign unchanged.

### Step 6: Optimal Spread — First-Order Conditions

Focus on the ask side. We maximise over $\delta^a$:

$$\max_{\delta^a} \; \lambda(\delta^a)\Big(w(t,q) - e^{-\gamma\delta^a}w(t,q-1)\Big)$$

Substituting $\lambda(\delta^a) = A e^{-\kappa\delta^a}$, let $H^a(\delta^a)$ denote the objective:

$$H^a(\delta^a) = A e^{-\kappa\delta^a}\Big(w(t,q) - e^{-\gamma\delta^a}w(t,q-1)\Big)$$

Taking the derivative with respect to $\delta^a$ and setting it to zero:

$$\frac{dH^a}{d\delta^a} = A e^{-\kappa\delta^a}\Big[-\kappa\big(w(t,q) - e^{-\gamma\delta^a}w(t,q-1)\big) + \gamma e^{-\gamma\delta^a}w(t,q-1)\Big] = 0$$

Since $A e^{-\kappa\delta^a} > 0$, the bracket must vanish:

$$-\kappa\big(w(t,q) - e^{-\gamma\delta^a}w(t,q-1)\big) + \gamma e^{-\gamma\delta^a}w(t,q-1) = 0$$

Solving for $e^{-\gamma\delta^a}$:

$$e^{-\gamma\delta^a} w(t,q-1)(\gamma + \kappa) = \kappa \, w(t,q)$$

$$e^{-\gamma\delta^a} = \frac{\kappa}{\gamma + \kappa} \cdot \frac{w(t,q)}{w(t,q-1)}$$

Taking logarithms:

$$\delta^{a*} = \frac{1}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right) + \frac{1}{\gamma}\ln\!\frac{w(t,q)}{w(t,q-1)}$$

By symmetry, the optimal bid distance is:

$$\delta^{b*} = \frac{1}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right) + \frac{1}{\gamma}\ln\!\frac{w(t,q)}{w(t,q+1)}$$

### Step 7: Approximation for the Ratio $w(t,q)/w(t,q\mp1)$

To obtain closed-form expressions, Avellaneda and Stoikov approximate the solution to the reduced HJB. Under the approximation that $w(t,q) \approx \exp\!\left(-\frac{1}{2}\gamma^2\sigma^2 q^2(T-t)\right)$ (motivated by the quadratic inventory penalty in the exponent), we compute:

$$\frac{w(t,q)}{w(t,q-1)} = \frac{\exp\!\left(-\tfrac{1}{2}\gamma^2\sigma^2 q^2(T-t)\right)}{\exp\!\left(-\tfrac{1}{2}\gamma^2\sigma^2(q-1)^2(T-t)\right)}$$

$$= \exp\!\left(-\frac{1}{2}\gamma^2\sigma^2\big[q^2 - (q-1)^2\big](T-t)\right)$$

$$= \exp\!\left(-\frac{1}{2}\gamma^2\sigma^2(2q - 1)(T-t)\right)$$

Similarly:

$$\frac{w(t,q)}{w(t,q+1)} = \exp\!\left(-\frac{1}{2}\gamma^2\sigma^2\big[q^2 - (q+1)^2\big](T-t)\right) = \exp\!\left(\frac{1}{2}\gamma^2\sigma^2(2q+1)(T-t)\right)$$

Substituting into the optimal distances:

$$\delta^{a*} = \frac{1}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right) - \frac{1}{2}\gamma\sigma^2(2q-1)(T-t)$$

$$\delta^{b*} = \frac{1}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right) + \frac{1}{2}\gamma\sigma^2(2q+1)(T-t)$$

### Step 8: Reservation Price

The optimal ask and bid prices are:

$$p^a = S_t + \delta^{a*}, \qquad p^b = S_t - \delta^{b*}$$

The **reservation price** $r_t$ is the midpoint of the optimal quotes:

$$r_t = \frac{p^a + p^b}{2} = S_t + \frac{\delta^{a*} - \delta^{b*}}{2}$$

Computing the difference:

$$\delta^{a*} - \delta^{b*} = -\frac{1}{2}\gamma\sigma^2(2q-1)(T-t) - \frac{1}{2}\gamma\sigma^2(2q+1)(T-t)$$

$$= -\frac{1}{2}\gamma\sigma^2(T-t)\big[(2q-1) + (2q+1)\big] = -\frac{1}{2}\gamma\sigma^2(T-t)(4q) = -2q\gamma\sigma^2(T-t)$$

Therefore:

$$\boxed{r_t = S_t - q_t \, \gamma \, \sigma^2 (T - t)}$$

**Interpretation of each term:**

- $S_t$: the current mid-price, which is the fair-value baseline.
- $q_t \gamma \sigma^2 (T-t)$: the **inventory penalty**. It shifts the reservation price in the direction that reduces inventory. When $q_t > 0$ (long), the penalty is positive, so $r_t < S_t$ — the market maker quotes lower to attract sellers and offload inventory. When $q_t < 0$ (short), $r_t > S_t$ — quotes shift higher to attract buyers. The penalty is proportional to:
  - $q_t$: the size of the inventory imbalance,
  - $\gamma$: the degree of risk aversion,
  - $\sigma^2$: the variance rate of mid-price moves (more volatility $\Rightarrow$ holding inventory is riskier),
  - $(T-t)$: the time remaining (more time $\Rightarrow$ more exposure to adverse moves).

### Step 9: Optimal Total Spread

The optimal total spread is $\delta^{a*} + \delta^{b*}$:

$$\delta^{a*} + \delta^{b*} = \frac{2}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right) - \frac{1}{2}\gamma\sigma^2(2q-1)(T-t) + \frac{1}{2}\gamma\sigma^2(2q+1)(T-t)$$

$$= \frac{2}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right) + \frac{1}{2}\gamma\sigma^2(T-t)\big[(2q+1) - (2q-1)\big]$$

$$= \frac{2}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right) + \gamma\sigma^2(T-t)$$

The optimal **half-spread** $\delta^*$ (symmetric component, independent of $q$) is:

$$\boxed{\delta^* = \frac{\gamma \sigma^2 (T-t)}{2} + \frac{1}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right)}$$

**Interpretation of the two terms:**

1. **Inventory-risk compensation** $\frac{\gamma\sigma^2(T-t)}{2}$: the market maker demands a wider spread when volatility is high and time remaining is large, because holding inventory is riskier under those conditions. This term grows linearly in $\sigma^2$ and $(T-t)$.

2. **Adverse-selection / order-arrival compensation** $\frac{1}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right)$: this term accounts for the uncertainty in order arrivals and the depth of the order book. When $\kappa$ is large (thin book, fills drop off quickly), $\gamma/\kappa$ is small, and this term shrinks — but the market maker is also filling fewer orders. When $\kappa$ is small (deep book), fills are less sensitive to quote distance and the market maker widens the spread to compensate for increased adverse selection.

### Inventory Risk vs Spread Revenue Tradeoff

The model captures the fundamental tradeoff facing every market maker:

- **Wider spread** $\Rightarrow$ more revenue per fill (each round-trip earns approximately $2\delta^*$), but the fill rate $\lambda(\delta^*) = A e^{-\kappa\delta^*}$ drops exponentially with distance, so fewer trades occur.
- **Tighter spread** $\Rightarrow$ more fills per unit time, but each fill earns less and the market maker accumulates more inventory, incurring greater directional risk.

The risk-aversion parameter $\gamma$ controls this balance:

- **High $\gamma$** (very risk-averse): the reservation price reacts strongly to inventory ($q\gamma\sigma^2(T-t)$ is large), quotes skew aggressively toward reducing position, inventory mean-reverts rapidly toward zero, spread is wider.
- **Low $\gamma$** (risk-tolerant): the market maker is willing to hold larger inventory for longer, quotes remain closer to $S_t$, spread is tighter, and fill rate is higher — but the variance of inventory P&L is also higher.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Mid-price volatility | $\sigma$ | Std dev of mid-price increments per unit time | 0.1–5.0 (asset and time-scale dependent) |
| Risk aversion | $\gamma$ | CARA risk-aversion coefficient | 0.01–1.0 (higher = more risk-averse) |
| Order-book depth | $\kappa$ | Rate of exponential decay in fill probability | 1–50 (higher = thinner book) |
| Baseline arrival rate | $A$ | Fill intensity when quoting at mid | 1–100 orders/unit time |
| Time horizon | $T$ | End of trading session | Session-dependent |
| Inventory | $q_t$ | Current signed position | Integers, often clamped to $[-Q_{\max}, Q_{\max}]$ |

## Numerical Example

**Parameters:**
- Mid-price: $S_t = 100$
- Inventory: $q_t = 3$ (long 3 units)
- Risk aversion: $\gamma = 0.1$
- Volatility: $\sigma = 2$ (so $\sigma^2 = 4$)
- Time remaining: $T - t = 0.5$
- Order-book depth: $\kappa = 1.5$

**Reservation price:**

$$r_t = 100 - 3 \times 0.1 \times 4 \times 0.5 = 100 - 0.6 = 99.4$$

The market maker's reservation price is 0.6 below mid — reflecting the desire to sell down the long inventory.

**Optimal half-spread:**

$$\delta^* = \frac{0.1 \times 4 \times 0.5}{2} + \frac{1}{0.1}\ln\!\left(1 + \frac{0.1}{1.5}\right) = 0.1 + 10 \times \ln(1.0\overline{6})$$

$$= 0.1 + 10 \times 0.0645 = 0.1 + 0.645 = 0.745$$

**Optimal quotes:**

$$p^a = r_t + \delta^* = 99.4 + 0.745 = 100.145$$

$$p^b = r_t - \delta^* = 99.4 - 0.745 = 98.655$$

Note the asymmetry relative to mid: the ask ($100.145$) is only $0.145$ above mid, while the bid ($98.655$) is $1.345$ below mid. This asymmetry actively encourages selling (to reduce inventory) and discourages buying.

**Fill rates:**

$$\lambda^a = A \exp(-\kappa \times (p^a - S_t)) = A \exp(-1.5 \times 0.145) = A \times 0.804$$

$$\lambda^b = A \exp(-\kappa \times (S_t - p^b)) = A \exp(-1.5 \times 1.345) = A \times 0.132$$

The ask fills about 6 times more frequently than the bid, which is exactly what inventory management demands when $q = 3$.

```python
import numpy as np

S, q, gamma, sigma, T_minus_t, kappa = 100, 3, 0.1, 2, 0.5, 1.5

r = S - q * gamma * sigma**2 * T_minus_t
delta_star = gamma * sigma**2 * T_minus_t / 2 + (1/gamma) * np.log(1 + gamma/kappa)
ask = r + delta_star
bid = r - delta_star

print(f"Reservation price: {r:.3f}")
print(f"Optimal half-spread: {delta_star:.3f}")
print(f"Ask: {ask:.3f}  Bid: {bid:.3f}")
print(f"Ask fill rate factor: {np.exp(-kappa*(ask - S)):.3f}")
print(f"Bid fill rate factor: {np.exp(-kappa*(S - bid)):.3f}")
```

## Connections

- [Inventory Risk](./inventory_risk.md): the reservation price is the primary mechanism through which the Avellaneda-Stoikov model manages inventory risk. The inventory penalty $q\gamma\sigma^2(T-t)$ is derived from the quadratic variance of inventory P&L, which is analysed in detail in the inventory risk file.
- [Bid-Ask Spread](./bid_ask_spread.md): the optimal half-spread $\delta^*$ provides a micro-founded explanation for the existence and magnitude of the bid-ask spread. The two components (inventory risk and adverse selection) correspond to the spread decomposition discussed in the bid-ask spread file.
- [Brownian Motion](../01_stochastic/brownian_motion.md): the mid-price model $dS = \sigma\,dW$ is a pure Brownian motion without drift. Properties of Brownian motion (independent increments, quadratic variation) underpin the variance calculations in the inventory penalty.
- [Ornstein-Uhlenbeck Process](../04_stat_arb/ornstein_uhlenbeck.md): the inventory process under optimal control exhibits mean-reversion toward zero — the reservation price mechanism acts like an OU restoring force on $q_t$. The speed of mean-reversion is controlled by $\gamma$.
- [Black-Scholes](../02_options/black_scholes.md): both models involve stochastic control under diffusion dynamics. The HJB approach used here parallels the derivation of the Black-Scholes PDE via dynamic hedging.

## Relevance for Prosperity 4

In IMC Prosperity 4, market making is a core strategy for products where you act as a liquidity provider. The Avellaneda-Stoikov framework maps directly to the competition setting:

1. **Parameter estimation:** estimate $\sigma$ from a rolling window of recent mid-price returns (e.g., standard deviation of last 50–100 price changes). The depth parameter $\kappa$ can be calibrated from historical fill data: fit $\ln(\text{fill rate})$ vs. quote distance and take the negative slope.

2. **Reservation price:** compute $r_t = S_t - q_t \gamma \sigma^2 (T - t)$ each tick. In Prosperity, $T - t$ is the number of remaining ticks normalised to $[0,1]$, and $q_t$ is your current position. Set $\gamma$ via backtesting: start with $\gamma = 0.1$ and increase if inventory swings are too large.

3. **Quote placement:** set ask at $r_t + \delta^*$ and bid at $r_t - \delta^*$. Clamp quotes to integer price ticks if the exchange requires it.

4. **Inventory limits:** impose hard limits $|q_t| \le Q_{\max}$ (e.g., $Q_{\max} = 20$). When the limit is reached, only quote on the side that reduces inventory. This prevents catastrophic losses from runaway positions.

5. **Regime detection:** if realised volatility spikes suddenly (e.g., doubles within 10 ticks), widen the spread by increasing the effective $\gamma$ or pausing quotes entirely. The model assumes constant $\sigma$, so regime shifts require manual intervention.

6. **Multi-product extension:** run independent Avellaneda-Stoikov models for each product, but share inventory limits across correlated products to avoid hidden concentration risk.
