# Inventory Risk

> **Core formula:** $$\mathrm{Var}[q_t \, \Delta S_t] = q_t^2 \, \sigma^2 \, \Delta t$$

## Intuition

A market maker earns money by repeatedly buying at the bid and selling at the ask, pocketing the spread. But between the buy and the sell, the market maker holds inventory — and during that holding period, the mid-price can move. If you just bought 10 units and the price drops by \$1, you have lost \$10 on inventory regardless of how much spread you earned. Inventory risk is the risk that the value of your held position changes before you can offload it.

The danger is asymmetric in practice because of **adverse selection**: the orders that fill against you tend to be on the wrong side. When an informed trader knows the price is about to rise, they buy from your ask — you just sold right before a rally. When the price is about to drop, they sell into your bid — you just bought before a decline. This means the inventory you accumulate is systematically biased toward the losing side. Adverse selection is the fundamental reason why a positive bid-ask spread is necessary: it compensates the market maker for this informational disadvantage.

Think of inventory risk like a warehouse owner storing a perishable commodity. The longer goods sit in the warehouse, the greater the chance of spoilage (price decline). A small inventory is manageable, but a large inventory compounds the risk quadratically — ten times the stock means one hundred times the variance in potential loss. Smart warehouse management means turning over inventory quickly, even if that means accepting slightly lower margins per unit.

## Mathematical Setup

**State variables and definitions:**

| Symbol | Definition |
|--------|-----------|
| $q_t$ | Inventory at time $t$ (signed: positive = long, negative = short) |
| $S_t$ | Mid-price at time $t$ |
| $\Delta S_t$ | Price change over interval $\Delta t$: $\Delta S_t = S_{t+\Delta t} - S_t$ |
| $\sigma$ | Volatility of mid-price: $\mathrm{Var}[\Delta S_t] = \sigma^2 \Delta t$ |
| $\gamma$ | Risk-aversion parameter (CARA utility) |
| $\delta$ | Half-spread: market maker quotes at $S_t \pm \delta$ |

**Assumptions:**

1. Mid-price follows arithmetic Brownian motion: $dS_t = \sigma \, dW_t$ (no drift).
2. Over short intervals, inventory $q_t$ is approximately constant (changes only at fill events).
3. Fill events arrive as Poisson processes with intensity $\lambda(\delta) = A e^{-\kappa\delta}$.

## Derivation

### PnL Decomposition

The total profit and loss of a market maker over a period $[0, T]$ decomposes into two independent components:

$$\text{Total PnL} = \underbrace{\text{Spread Capture}}_{\text{always positive}} + \underbrace{\text{Inventory PnL}}_{\text{can be very negative}}$$

**Spread capture** is the cumulative revenue from completed round-trips. Each time the market maker buys at the bid ($S - \delta^b$) and sells at the ask ($S + \delta^a$), the revenue is $\delta^a + \delta^b$ (the total spread). If $N_{\text{rt}}$ round-trips occur:

$$\text{Spread Capture} = \sum_{i=1}^{N_{\text{rt}}} (\delta^a_i + \delta^b_i)$$

For symmetric quoting with half-spread $\delta$, this simplifies to:

$$\text{Spread Capture} = 2\delta \cdot N_{\text{rt}}$$

This is always non-negative (assuming $\delta > 0$), and under a Poisson fill model with balanced flow, the expected number of round-trips is approximately $\lambda(\delta) \cdot T / 2$.

**Inventory PnL** is the mark-to-market gain or loss on the held position:

$$\text{Inventory PnL} = \sum_{t} q_t \, \Delta S_t$$

In continuous time:

$$\text{Inventory PnL} = \int_0^T q_t \, dS_t = \int_0^T q_t \, \sigma \, dW_t$$

This is a stochastic integral with zero expectation (since $dS_t$ has no drift) but potentially very large variance.

### Variance of Inventory PnL

For a fixed inventory $q$ held over a short interval $\Delta t$:

$$\mathbb{E}[q \, \Delta S] = q \, \mathbb{E}[\Delta S] = 0$$

$$\mathrm{Var}[q \, \Delta S] = q^2 \, \mathrm{Var}[\Delta S] = q^2 \, \sigma^2 \, \Delta t$$

The risk grows **quadratically** in inventory. Doubling your position quadruples the variance of your inventory P&L. This is the central reason why inventory management is critical:

| Inventory $q$ | Relative Variance |
|:-:|:-:|
| 1 | $1 \times \sigma^2 \Delta t$ |
| 2 | $4 \times \sigma^2 \Delta t$ |
| 5 | $25 \times \sigma^2 \Delta t$ |
| 10 | $100 \times \sigma^2 \Delta t$ |

Over a longer horizon where inventory evolves stochastically:

$$\mathrm{Var}\!\left[\int_0^T q_t \, \sigma \, dW_t\right] = \sigma^2 \int_0^T \mathbb{E}[q_t^2] \, dt$$

by the Itô isometry. This shows that the cumulative risk depends on the expected squared inventory integrated over time — reinforcing the importance of keeping $q_t$ small.

### Adverse Selection

Adverse selection is the phenomenon where the market maker's fills are correlated with subsequent unfavourable price moves. Formally, let $\Delta S_{t+1}$ be the next price change after a fill at time $t$. Under adverse selection:

$$\mathbb{E}[\Delta S_{t+1} \mid \text{ask fills at } t] < 0$$

$$\mathbb{E}[\Delta S_{t+1} \mid \text{bid fills at } t] > 0$$

The intuition: if an informed buyer hits your ask, it signals the price is about to rise — you just sold into strength. If an informed seller hits your bid, the price is about to fall — you just bought into weakness.

The magnitude of adverse selection is related to the information content of order flow. Let $\alpha$ denote the fraction of informed traders. On each trade:

- With probability $\alpha$: the counterparty is informed, and the market maker loses on average $\mu$ (the information advantage of the informed trader).
- With probability $1 - \alpha$: the counterparty is uninformed, and the market maker earns the half-spread $\delta$.

The expected profit per trade is:

$$\mathbb{E}[\text{profit per trade}] = (1-\alpha)\delta - \alpha(\mu - \delta) = \delta - \alpha\mu$$

For the market maker to break even, the spread must satisfy:

$$\delta \geq \alpha \mu$$

This is the adverse-selection lower bound on the spread: even with zero inventory risk and zero processing costs, the spread cannot go below $\alpha\mu$ without the market maker losing money on average.

### How $\gamma$ Controls the Tradeoff

In the Avellaneda-Stoikov framework, the reservation price is:

$$r_t = S_t - q_t \gamma \sigma^2 (T-t)$$

The parameter $\gamma$ governs the speed at which the market maker's quotes respond to inventory:

**High $\gamma$ (aggressive inventory management):**
- The reservation price moves sharply with $q_t$: even a small inventory $q = 1$ produces a noticeable shift.
- Quotes are strongly skewed toward reducing position.
- Inventory mean-reverts rapidly toward zero.
- Consequence: low variance of inventory PnL, but also less spread capture because the market maker frequently quotes aggressively (tighter on one side, wider on the other), reducing the effective average spread earned.

**Low $\gamma$ (passive inventory management):**
- The reservation price is nearly insensitive to $q_t$.
- Quotes remain close to the mid-price regardless of inventory.
- Inventory can wander far from zero before quotes adjust.
- Consequence: higher fill rate and more spread capture in benign conditions, but much higher variance of inventory PnL. During adverse moves, large inventory can produce losses that dwarf the accumulated spread revenue.

The expected utility under CARA preferences $u(w) = -e^{-\gamma w}$ penalises variance proportionally to $\gamma$:

$$\mathbb{E}[u(w)] \approx -\exp\!\left(-\gamma\left(\mathbb{E}[w] - \frac{\gamma}{2}\mathrm{Var}[w]\right)\right)$$

so the certainty equivalent of wealth $w$ is approximately:

$$\text{CE} = \mathbb{E}[w] - \frac{\gamma}{2}\mathrm{Var}[w]$$

Higher $\gamma$ means more variance penalty, which directly translates to faster inventory correction.

### Practical Inventory Management

Beyond the continuous-time optimal control, real implementations enforce discrete safeguards:

1. **Hard position limits:** enforce $|q_t| \le Q_{\max}$. When the limit is reached, only quote on the side that reduces inventory (e.g., if $q = Q_{\max}$, only post asks). This provides a safety net when the model's assumptions break down.

2. **Forced liquidation:** if $|q_t|$ exceeds a soft limit $Q_{\text{soft}} < Q_{\max}$, begin placing aggressive market orders (or very tight limit orders) to reduce position, accepting adverse price impact as a cost of risk reduction.

3. **Volatility-adaptive limits:** reduce $Q_{\max}$ when volatility rises. Since risk scales as $q^2 \sigma^2$, the same inventory is riskier in a volatile market. A simple rule: $Q_{\max}(\sigma) = Q_{\max}^{\text{base}} \times (\sigma_{\text{base}} / \sigma)$.

4. **Time-decay limits:** reduce $Q_{\max}$ as $T - t \to 0$ (end of session approaches). The model's inventory penalty $q\gamma\sigma^2(T-t) \to 0$ as $T \to t$, which seems to suggest less urgency — but in reality, the market maker faces liquidation risk at $T$ and should flatten more aggressively.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Inventory | $q_t$ | Current signed position | $[-Q_{\max}, Q_{\max}]$, often $|q| \le 20$ |
| Volatility | $\sigma$ | Mid-price diffusion coefficient | Asset-dependent |
| Risk aversion | $\gamma$ | CARA parameter controlling inventory penalty | 0.01–1.0 |
| Informed fraction | $\alpha$ | Proportion of trades from informed traders | 0.05–0.30 |
| Information advantage | $\mu$ | Expected price move known by informed traders | Spread-dependent |
| Position limit | $Q_{\max}$ | Hard cap on absolute inventory | 5–50 units |

## Numerical Example

**Setup:** a market maker with $\gamma = 0.1$, $\sigma = 2$ (so $\sigma^2 = 4$), half-spread $\delta = 0.5$, baseline fill rate $\lambda = 10$ fills/unit time, and $T = 1$.

**Spread capture per round-trip:** $2\delta = 1.0$.

**Expected round-trips:** with balanced flow, approximately $\lambda T / 2 = 5$ round-trips.

**Expected spread capture:** $5 \times 1.0 = 5.0$.

**Inventory PnL risk:** suppose at some point $q_t = 5$. Over a short interval $\Delta t = 0.01$:

$$\text{Std dev of inventory PnL} = |q| \sigma \sqrt{\Delta t} = 5 \times 2 \times 0.1 = 1.0$$

So holding 5 units for 0.01 time units exposes the market maker to a one-standard-deviation loss of \$1.0 — comparable to the entire spread earned on a single round-trip. Over a longer interval $\Delta t = 0.1$:

$$\text{Std dev} = 5 \times 2 \times \sqrt{0.1} = 3.16$$

A two-standard-deviation adverse move would wipe out more than the entire expected spread capture.

**Certainty equivalent:**

$$\text{CE} = \mathbb{E}[\text{PnL}] - \frac{\gamma}{2}\mathrm{Var}[\text{PnL}] = 5.0 - \frac{0.1}{2} \times 10.0 = 5.0 - 0.5 = 4.5$$

```python
import numpy as np

gamma, sigma, q, delta, lam, T = 0.1, 2.0, 5, 0.5, 10.0, 1.0

spread_capture = 2 * delta * (lam * T / 2)
inv_std_short = abs(q) * sigma * np.sqrt(0.01)
inv_std_long = abs(q) * sigma * np.sqrt(0.1)
var_pnl = q**2 * sigma**2 * T
ce = spread_capture - (gamma / 2) * var_pnl

print(f"Expected spread capture: {spread_capture:.2f}")
print(f"Inventory std (dt=0.01): {inv_std_short:.2f}")
print(f"Inventory std (dt=0.10): {inv_std_long:.2f}")
print(f"Inventory PnL variance (full period): {var_pnl:.2f}")
print(f"Certainty equivalent: {ce:.2f}")
```

## Connections

- [Avellaneda-Stoikov](./avellaneda_stoikov.md): the reservation price $r_t = S_t - q\gamma\sigma^2(T-t)$ is the primary mechanism for managing inventory risk in the AS model. The quadratic variance $q^2\sigma^2$ derived here motivates the $q$-dependent shift in the reservation price.
- [Bid-Ask Spread](./bid_ask_spread.md): inventory risk is one of the three fundamental components of the bid-ask spread. The inventory holding cost term in the spread decomposition arises directly from the analysis in this file.
- [Brownian Motion](../01_stochastic/brownian_motion.md): the variance formula $\mathrm{Var}[\Delta S] = \sigma^2\Delta t$ comes from the properties of Brownian motion increments. The quadratic variation of Brownian motion is also the reason inventory risk scales linearly in time.
- [Greeks — Delta](../02_options/greeks.md): in options market making, inventory risk is managed via delta hedging. The delta of the options portfolio plays the same role as $q_t$ here: it measures the directional exposure that must be controlled.
- [Ornstein-Uhlenbeck](../04_stat_arb/ornstein_uhlenbeck.md): the inventory process under the AS optimal control is mean-reverting toward zero, resembling an OU process. The speed of mean-reversion is controlled by $\gamma\sigma^2(T-t)$.

## Relevance for Prosperity 4

Inventory risk is the dominant concern for any market-making strategy in Prosperity 4:

1. **Monitor $q_t$ continuously.** At every tick, compute your current inventory across all products. Never let a position grow unchecked — the quadratic risk scaling means large positions can generate losses that overwhelm days of accumulated spread revenue.

2. **Set hard limits.** Define $Q_{\max}$ for each product before the competition starts. A good starting point is to ensure that the maximum inventory PnL standard deviation ($Q_{\max} \times \sigma \times \sqrt{\Delta t_{\text{session}}}$) does not exceed your total expected spread capture for the session.

3. **Adaptive $\gamma$.** If you observe that your inventory is consistently drifting in one direction (suggesting adverse selection or a trending market), temporarily increase $\gamma$ to flatten the position faster. If inventory oscillates healthily around zero, you can afford a lower $\gamma$ to maximise fill rates.

4. **End-of-round flattening.** Prosperity rounds have a fixed duration. As the end approaches, increase the effective $\gamma$ (or equivalently, treat $T - t$ as smaller than the actual remaining time) to ensure you enter the settlement phase with minimal inventory.

5. **Cross-product inventory.** If two products are correlated (e.g., an underlying and a derivative), monitor the net delta across both positions. Inventory risk should be evaluated on the portfolio level, not per-product in isolation.
