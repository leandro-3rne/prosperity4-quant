# Bid-Ask Spread

> **Core formula — Roll (1984) effective spread estimator:**
> $$c = \sqrt{-\mathrm{Cov}(\Delta p_t, \, \Delta p_{t-1})}, \qquad \text{Effective spread} = 2c$$

## Intuition

The bid-ask spread is the price of immediacy. A buyer who wants to trade right now pays the ask; a seller who wants to trade right now receives the bid. The difference between ask and bid — the spread — compensates the market maker for three costs: (1) the risk of holding inventory that may lose value, (2) the risk of trading against someone who knows more than you (adverse selection), and (3) the fixed operational costs of maintaining a presence in the market.

In a perfectly competitive market with no information asymmetry, inventory costs, or processing costs, the spread would be zero. Every real-world spread reflects some combination of these frictions. Understanding which component dominates in a given market is essential for calibrating a market-making strategy: in a market dominated by adverse selection, you need wider spreads and fast information processing; in a market dominated by inventory risk, you need efficient inventory management and hedging.

A useful analogy is a currency exchange booth at an airport. The booth quotes a buy rate and a sell rate for foreign currency. The gap between them compensates the booth for: the risk that the exchange rate moves while they hold foreign currency (inventory), the risk that a sophisticated forex trader exploits a stale quote (adverse selection), and the rent and staff costs (order processing). Different booths in different locations set different spreads based on which cost dominates their business.

## Mathematical Setup

**Definitions:**

| Symbol | Definition |
|--------|-----------|
| $p_t$ | Transaction price at time $t$ |
| $m_t$ | Efficient (true) price at time $t$ |
| $a_t, b_t$ | Ask and bid prices at time $t$ |
| $c$ | Half-spread: $c = (a_t - b_t)/2$ |
| $\Delta p_t$ | Price change: $\Delta p_t = p_t - p_{t-1}$ |
| $\alpha$ | Probability that counterparty is informed |
| $\mu$ | Informed trader's information advantage (expected price move) |
| $\lambda$ | Kyle's lambda: price impact coefficient |
| $x_t$ | Signed order flow (positive = buy-initiated) |

**Assumptions for the Roll model:**
1. The efficient price $m_t$ follows a random walk: $m_t = m_{t-1} + u_t$, where $u_t \sim (0, \sigma_u^2)$ i.i.d.
2. The transaction price bounces between bid and ask: $p_t = m_t + c \, d_t$, where $d_t \in \{-1, +1\}$ with equal probability and is i.i.d.
3. The direction $d_t$ is independent of the efficient price innovation $u_t$.

## Derivation

### Part 1: Economic Sources of the Spread

The bid-ask spread $s = a - b = 2c$ decomposes into three additive components:

$$s = s_{\text{inventory}} + s_{\text{adverse selection}} + s_{\text{processing}}$$

**1. Inventory holding cost ($s_{\text{inventory}}$):** the market maker demands compensation for bearing directional risk while holding inventory. From the Avellaneda-Stoikov model, the inventory-related component of the half-spread is:

$$\frac{s_{\text{inventory}}}{2} = \frac{\gamma \sigma^2 (T-t)}{2}$$

This grows with volatility $\sigma^2$ and time horizon $T-t$, and is proportional to the risk-aversion coefficient $\gamma$.

**2. Adverse selection cost ($s_{\text{adverse selection}}$):** the market maker loses to informed traders on average. In the Glosten-Milgrom framework (derived below), the adverse-selection component equals the expected loss to informed traders:

$$\frac{s_{\text{adverse selection}}}{2} = \alpha \mu$$

where $\alpha$ is the probability of facing an informed trader and $\mu$ is the informed trader's information advantage.

**3. Order processing cost ($s_{\text{processing}}$):** fixed costs of maintaining infrastructure, co-location, exchange fees, staff, etc. This component is constant and does not depend on market conditions. In electronic markets, it is typically small relative to the other two components.

### Part 2: The Roll (1984) Model — Spread Estimation from Autocovariance

Roll's model provides a way to estimate the effective spread purely from transaction prices, without observing the order book.

**Setup:** transaction prices are:

$$p_t = m_t + c \, d_t$$

where $m_t = m_{t-1} + u_t$ is the efficient price and $d_t \in \{-1, +1\}$ is the trade direction indicator (equally likely).

**Step 1:** compute the price change:

$$\Delta p_t = p_t - p_{t-1} = (m_t + c \, d_t) - (m_{t-1} + c \, d_{t-1})$$

$$= u_t + c(d_t - d_{t-1})$$

**Step 2:** compute the autocovariance $\mathrm{Cov}(\Delta p_t, \Delta p_{t-1})$. First expand:

$$\Delta p_t = u_t + c \, d_t - c \, d_{t-1}$$

$$\Delta p_{t-1} = u_{t-1} + c \, d_{t-1} - c \, d_{t-2}$$

Since $u_t$, $u_{t-1}$, $d_t$, $d_{t-1}$, $d_{t-2}$ are all mutually independent:

$$\mathrm{Cov}(\Delta p_t, \Delta p_{t-1}) = \mathrm{Cov}(u_t + c\,d_t - c\,d_{t-1}, \; u_{t-1} + c\,d_{t-1} - c\,d_{t-2})$$

The only term that produces a non-zero covariance is the cross-term involving $d_{t-1}$:

$$= \mathrm{Cov}(-c\,d_{t-1}, \; c\,d_{t-1})$$

$$= -c^2 \, \mathrm{Var}(d_{t-1})$$

Since $d_{t-1} \in \{-1, +1\}$ with equal probability, $\mathbb{E}[d_{t-1}] = 0$ and $\mathrm{Var}(d_{t-1}) = \mathbb{E}[d_{t-1}^2] = 1$. Therefore:

$$\mathrm{Cov}(\Delta p_t, \Delta p_{t-1}) = -c^2$$

**Step 3:** solve for the half-spread:

$$c = \sqrt{-\mathrm{Cov}(\Delta p_t, \Delta p_{t-1})}$$

and the effective spread is:

$$\boxed{\text{Effective spread} = 2c = 2\sqrt{-\mathrm{Cov}(\Delta p_t, \Delta p_{t-1})}}$$

**Step 4:** verify the variance of price changes:

$$\mathrm{Var}(\Delta p_t) = \mathrm{Var}(u_t) + c^2 \mathrm{Var}(d_t) + c^2 \mathrm{Var}(d_{t-1}) = \sigma_u^2 + 2c^2$$

This shows that observed price volatility is inflated by the bid-ask bounce: even if the efficient price has zero volatility ($\sigma_u = 0$), transaction prices would still fluctuate with variance $2c^2$ due to the mechanical bouncing between bid and ask.

**Important caveat:** the Roll estimator requires $\mathrm{Cov}(\Delta p_t, \Delta p_{t-1}) < 0$. If the empirical autocovariance is positive (which can happen with momentum or stale quotes), the estimator is undefined. Positive autocovariance suggests that the bid-ask bounce effect is dominated by other microstructure effects.

### Part 3: Glosten-Milgrom (1985) Framework

The Glosten-Milgrom model micro-founds the adverse-selection component of the spread by modelling the market maker's Bayesian updating in the presence of informed and uninformed traders.

**Setup:**

- The asset's true value is $V$, which is unknown to the market maker.
- With probability $\alpha$, the incoming trader is informed and knows $V$.
- With probability $1-\alpha$, the trader is uninformed and trades randomly (buy or sell with equal probability).
- An informed trader buys if $V > a$ (ask) and sells if $V < b$ (bid).

**Deriving the ask price:** the market maker sets the ask price $a$ such that the expected value of the asset, conditional on a buy order arriving, equals $a$:

$$a = \mathbb{E}[V \mid \text{buy}]$$

By Bayes' rule:

$$\Pr(\text{informed} \mid \text{buy}) = \frac{\alpha \cdot 1}{\alpha \cdot 1 + (1-\alpha) \cdot \frac{1}{2}} = \frac{\alpha}{\alpha + \frac{1-\alpha}{2}} = \frac{2\alpha}{1+\alpha}$$

For a simple two-state model where $V \in \{V_H, V_L\}$ with equal prior probability, and the informed trader knows the true state:

$$a = \Pr(\text{informed} \mid \text{buy}) \cdot V_H + \Pr(\text{uninformed} \mid \text{buy}) \cdot \mathbb{E}[V]$$

$$= \frac{2\alpha}{1+\alpha} \cdot V_H + \frac{1-\alpha}{1+\alpha} \cdot \frac{V_H + V_L}{2}$$

**Deriving the bid price:** by symmetric reasoning:

$$b = \Pr(\text{informed} \mid \text{sell}) \cdot V_L + \Pr(\text{uninformed} \mid \text{sell}) \cdot \mathbb{E}[V]$$

$$= \frac{2\alpha}{1+\alpha} \cdot V_L + \frac{1-\alpha}{1+\alpha} \cdot \frac{V_H + V_L}{2}$$

**The spread:**

$$s = a - b = \frac{2\alpha}{1+\alpha}(V_H - V_L)$$

This shows that:
- The spread is proportional to $\alpha$: more informed traders $\Rightarrow$ wider spread.
- The spread is proportional to $V_H - V_L$: greater uncertainty about fundamental value $\Rightarrow$ wider spread.
- When $\alpha = 0$ (no informed traders), the spread is zero — the market maker faces no adverse selection and can quote at the expected value.
- When $\alpha = 1$ (all traders informed), $s = V_H - V_L$ — the spread equals the full range of uncertainty.

**Spread decomposition:** the Glosten-Milgrom spread consists entirely of the adverse-selection component. In a full model, the total spread adds the inventory and processing components:

$$s_{\text{total}} = \underbrace{\frac{2\alpha}{1+\alpha}(V_H - V_L)}_{\text{adverse selection}} + \underbrace{\gamma\sigma^2(T-t)}_{\text{inventory risk}} + \underbrace{s_{\text{fixed}}}_{\text{processing}}$$

### Part 4: Kyle's Lambda — Price Impact

Kyle (1985) models a market with a single informed trader, noise traders, and a competitive market maker. The key result is that the market maker sets prices as a linear function of total order flow:

$$\Delta S = \lambda \cdot x$$

where $x$ is the net signed order flow (buy volume minus sell volume) and $\lambda$ is the **price impact coefficient** (Kyle's lambda).

**Derivation sketch:** in Kyle's single-auction model:
- The informed trader observes the true value $V \sim \mathcal{N}(\bar{V}, \Sigma_0)$.
- Noise traders submit random orders $u \sim \mathcal{N}(0, \sigma_u^2)$.
- The informed trader submits $x_I = \beta(V - \bar{V})$, choosing $\beta$ to maximise expected profit.
- The market maker observes total flow $y = x_I + u$ and sets price $p = \bar{V} + \lambda y$.

The equilibrium values are:

$$\lambda = \frac{1}{2} \frac{\sqrt{\Sigma_0}}{\sigma_u}, \qquad \beta = \frac{\sigma_u}{\sqrt{\Sigma_0}}$$

**Interpretation:**
- $\lambda$ is the informativeness of order flow per unit. Higher $\lambda$ means each trade moves the price more.
- $\lambda$ increases with prior uncertainty $\Sigma_0$ (more to learn from trades) and decreases with noise trader volume $\sigma_u$ (more noise camouflage for the informed trader).

**Relation to the spread:** in a continuous market, the effective half-spread is approximately:

$$c \approx \lambda \cdot \mathbb{E}[|x|]$$

where $\mathbb{E}[|x|]$ is the expected absolute order size. This connects Kyle's price impact to the observable bid-ask spread: markets with high $\lambda$ (illiquid, information-sensitive) have wide spreads; markets with low $\lambda$ (liquid, noise-dominated) have tight spreads.

**Empirical estimation:** regress price changes on signed order flow:

$$\Delta S_t = \hat{\lambda} \, x_t + \varepsilon_t$$

The slope coefficient $\hat{\lambda}$ estimates Kyle's lambda. Higher $\hat{\lambda}$ indicates a less liquid market with more informational content in order flow.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Half-spread | $c$ | Distance from mid to bid or ask | 0.01–5.0 (asset-dependent) |
| Informed fraction | $\alpha$ | Prob. counterparty is informed | 0.05–0.30 |
| Value range | $V_H - V_L$ | Uncertainty in fundamental value | Asset-dependent |
| Kyle's lambda | $\lambda$ | Price impact per unit of order flow | 0.001–1.0 |
| Noise volume | $\sigma_u$ | Std dev of uninformed order flow | Market-dependent |
| Prior uncertainty | $\Sigma_0$ | Variance of fundamental value | Asset-dependent |
| Autocovariance | $\mathrm{Cov}(\Delta p_t, \Delta p_{t-1})$ | First-lag autocovariance of returns | Typically negative for liquid assets |

## Numerical Example

### Roll Estimator

Suppose we observe 1000 transaction prices and compute:

$$\mathrm{Cov}(\Delta p_t, \Delta p_{t-1}) = -0.04$$

Then:

$$c = \sqrt{-(-0.04)} = \sqrt{0.04} = 0.20$$

$$\text{Effective spread} = 2c = 0.40$$

The observed price variance is $\mathrm{Var}(\Delta p_t) = 0.50$, so the efficient-price variance is:

$$\sigma_u^2 = \mathrm{Var}(\Delta p_t) - 2c^2 = 0.50 - 2(0.04) = 0.42$$

About $2c^2/\mathrm{Var}(\Delta p_t) = 0.08/0.50 = 16\%$ of observed price variance is due to the bid-ask bounce, not genuine price discovery.

### Glosten-Milgrom Spread

Suppose $\alpha = 0.2$, $V_H = 102$, $V_L = 98$, so $V_H - V_L = 4$ and $\mathbb{E}[V] = 100$:

$$s = \frac{2 \times 0.2}{1 + 0.2} \times 4 = \frac{0.4}{1.2} \times 4 = \frac{4}{3} \approx 1.33$$

Half-spread $c \approx 0.67$. Ask $= 100 + 0.67 = 100.67$, Bid $= 100 - 0.67 = 99.33$.

### Kyle's Lambda

Suppose $\Sigma_0 = 4$ (so $\sqrt{\Sigma_0} = 2$) and $\sigma_u = 10$:

$$\lambda = \frac{1}{2} \cdot \frac{2}{10} = 0.10$$

A net buy order of $x = 5$ units moves the price by $\Delta S = 0.10 \times 5 = 0.50$.

```python
import numpy as np

autocov = -0.04
c_roll = np.sqrt(-autocov)
spread_roll = 2 * c_roll
print(f"Roll half-spread: {c_roll:.4f}")
print(f"Roll effective spread: {spread_roll:.4f}")

alpha, VH, VL = 0.2, 102, 98
s_gm = 2 * alpha / (1 + alpha) * (VH - VL)
print(f"\nGlosten-Milgrom spread: {s_gm:.4f}")
print(f"Ask: {(VH+VL)/2 + s_gm/2:.4f}, Bid: {(VH+VL)/2 - s_gm/2:.4f}")

Sigma0, sigma_u = 4.0, 10.0
lam = 0.5 * np.sqrt(Sigma0) / sigma_u
print(f"\nKyle's lambda: {lam:.4f}")
print(f"Price impact of x=5: {lam * 5:.4f}")
```

## Connections

- [Avellaneda-Stoikov](./avellaneda_stoikov.md): the AS optimal half-spread $\delta^* = \frac{\gamma\sigma^2(T-t)}{2} + \frac{1}{\gamma}\ln(1+\gamma/\kappa)$ provides a micro-founded model of the spread. The first term corresponds to the inventory component and the second to adverse selection / order arrival uncertainty, matching the decomposition framework in this file.
- [Inventory Risk](./inventory_risk.md): the inventory holding cost component of the spread is a direct consequence of inventory risk. Higher inventory risk (larger $q^2\sigma^2$) demands a wider spread to compensate.
- [Brownian Motion](../01_stochastic/brownian_motion.md): the Roll model assumes the efficient price follows a random walk (Brownian motion in discrete time). The autocovariance structure of Brownian motion increments (zero for all lags) is what makes the bid-ask bounce the sole source of negative autocovariance.
- [Ornstein-Uhlenbeck](../04_stat_arb/ornstein_uhlenbeck.md): in mean-reverting markets, the Roll estimator may overestimate the spread because the efficient price itself contributes negative autocovariance. Distinguishing mean-reversion from bid-ask bounce requires additional analysis (e.g., using multiple lags).
- [Strategy Integration](../06_prosperity/strategy_integration.md): the spread decomposition informs parameter selection across all trading strategies — wider spreads in information-sensitive products, tighter spreads in noise-dominated products.

## Relevance for Prosperity 4

Understanding the components of the bid-ask spread is essential for both market making and taking liquidity in Prosperity 4:

1. **Spread estimation:** use the Roll estimator to compute effective spreads from historical transaction data for each product. This tells you how much it costs to trade (as a taker) and how much you can earn (as a maker). If the autocovariance is strongly negative, the spread is a significant friction and market making is potentially profitable.

2. **Adverse selection detection:** if your fills consistently precede adverse price moves, you are experiencing adverse selection. Track $\mathbb{E}[\Delta S_{t+1} \mid \text{fill at } t]$: if this is systematically negative for ask fills and positive for bid fills, widen your spread or add a delay before quoting after a fill (stale quote protection).

3. **Kyle's lambda calibration:** regress price changes on net order flow to estimate $\lambda$. Products with high $\lambda$ are informationally sensitive — you need wider spreads and faster quote updates. Products with low $\lambda$ are noise-dominated — you can quote tighter and earn more from flow.

4. **Spread decomposition in practice:** for each product, decompose the observed spread into its components. If inventory risk dominates (high $\sigma$, products with trending behaviour), focus on inventory management ($\gamma$ tuning). If adverse selection dominates (high $\alpha$, products where other bots have information advantages), focus on speed and information extraction.

5. **Competitive spread setting:** in Prosperity, you compete with other market makers. If your spread is too wide, you lose fills to competitors. If too tight, you lose money to adverse selection. The equilibrium spread is where your marginal cost (inventory + adverse selection) equals the competitive market spread. Monitor competitor quotes and adjust $\gamma$ and $\kappa$ to remain competitive while profitable.
