# Jump-Diffusion: Merton (1976) Model

> **Core formula:**
> $$\frac{dS}{S} = (\mu - \lambda\bar{k})\,dt + \sigma\,dW_t + (J-1)\,dN_t$$

## Intuition

The Black–Scholes model assumes stock prices move smoothly — no gaps, no sudden crashes. But real markets are full of jumps: earnings surprises, central bank announcements, geopolitical shocks, or in the context of Prosperity, sudden large orders that move the price instantaneously. When you look at the histogram of real stock returns, the tails are fatter than a normal distribution predicts. There are far more extreme moves than GBM allows.

Robert Merton's 1976 model addresses this by layering a Poisson jump process on top of the standard GBM diffusion. Between jumps, the stock moves continuously via Brownian motion. At random Poisson arrival times, the stock price multiplies by a random factor $J$ (which can be greater or less than 1). The result: most of the time the price behaves normally, but occasionally it makes a large discrete move — exactly the behaviour we observe empirically.

The financial consequence is dramatic for option pricing. Jump risk makes out-of-the-money puts more valuable than BS predicts (because crashes are more likely), generating the **volatility skew** seen in real markets. It also makes near-expiry options more valuable, because a jump can push them from worthless to deep in-the-money in an instant. The Merton model was the first rigorous framework to explain these phenomena.

## Mathematical Setup

### Symbols

| Symbol | Meaning |
|--------|---------|
| $S_t$ | Stock price at time $t$ |
| $\mu$ | Total expected return under $\mathbb{P}$ |
| $\sigma$ | Diffusive volatility (continuous component) |
| $W_t$ | Standard Brownian motion |
| $N_t$ | Poisson process with intensity $\lambda$ |
| $\lambda$ | Average number of jumps per year |
| $J$ | Jump size multiplier ($S \to JS$ at a jump) |
| $\mu_J$ | Mean of $\ln J$ |
| $\sigma_J$ | Standard deviation of $\ln J$ |
| $\bar{k}$ | $\mathbb{E}[J-1] = e^{\mu_J + \sigma_J^2/2} - 1$, the mean relative jump size |
| $K$ | Strike price |
| $T$ | Time to expiry |
| $r$ | Risk-free rate |

### Assumptions

1. Between jumps, the stock follows GBM with volatility $\sigma$.
2. Jumps arrive as a Poisson process with constant intensity $\lambda$.
3. Jump sizes $J$ are i.i.d. lognormal: $\ln J \sim \mathcal{N}(\mu_J, \sigma_J^2)$.
4. The Brownian motion $W_t$, the Poisson process $N_t$, and the jump sizes $\{J_i\}$ are mutually independent.
5. Jump risk is **diversifiable** (Merton's key assumption), so it earns no risk premium.

## Derivation

### Step 1 — The compound Poisson process

A **Poisson process** $N_t$ with intensity $\lambda$ counts the number of events (jumps) in $[0,t]$:

$$\mathbb{P}(N_t = n) = \frac{(\lambda t)^n}{n!}e^{-\lambda t}, \qquad \mathbb{E}[N_t] = \lambda t, \qquad \text{Var}(N_t) = \lambda t$$

A **compound Poisson process** attaches a random size $Y_i$ to each jump:

$$X_t = \sum_{i=1}^{N_t} Y_i$$

Its expectation (by the law of total expectation, conditioning on $N_t$):

$$\mathbb{E}[X_t] = \mathbb{E}\!\left[\sum_{i=1}^{N_t}Y_i\right] = \mathbb{E}[N_t]\cdot\mathbb{E}[Y_1] = \lambda t\,\mathbb{E}[Y_1]$$

In Merton's model, the jump sizes are the multiplicative factors $J_i$, and the "size" attached to each jump in the return equation is $J_i - 1$. So the cumulative jump contribution to returns is:

$$\sum_{i=1}^{N_t}(J_i - 1)$$

with expectation:

$$\mathbb{E}\!\left[\sum_{i=1}^{N_t}(J_i - 1)\right] = \lambda t \cdot \mathbb{E}[J-1] = \lambda t\,\bar{k}$$

### Step 2 — The Merton SDE

The stock price dynamics combine continuous diffusion with discrete jumps:

$$\frac{dS}{S} = (\mu - \lambda\bar{k})\,dt + \sigma\,dW_t + (J-1)\,dN_t$$

Breaking this down:

- $(\mu - \lambda\bar{k})\,dt$: drift, adjusted downward by $\lambda\bar{k}$ to compensate for the expected upward contribution of jumps. This ensures $\mathbb{E}[dS/S] = \mu\,dt$.
- $\sigma\,dW_t$: the usual diffusive (continuous) component.
- $(J-1)\,dN_t$: at each Poisson event, the price jumps by a factor of $J$, so the return jumps by $J-1$.

**Verification of drift compensation:** Over $[0,dt]$:

$$\mathbb{E}\!\left[\frac{dS}{S}\right] = (\mu - \lambda\bar{k})\,dt + 0 + \mathbb{E}[(J-1)]\cdot\mathbb{E}[dN_t]$$

$$= (\mu - \lambda\bar{k})\,dt + \bar{k}\cdot\lambda\,dt = \mu\,dt \qquad \checkmark$$

### Step 3 — Solution of the SDE

Over $[0,T]$, suppose $N_T = n$ jumps occur at times $t_1, \ldots, t_n$ with sizes $J_1, \ldots, J_n$. Between jumps, the stock follows GBM with drift $\mu - \lambda\bar{k}$ and vol $\sigma$. At each jump, $S$ multiplies by $J_i$. Therefore:

$$S_T = S_0 \exp\!\left[(\mu - \lambda\bar{k} - \tfrac{1}{2}\sigma^2)T + \sigma W_T\right]\prod_{i=1}^{n}J_i$$

Taking logs:

$$\ln\frac{S_T}{S_0} = \left(\mu - \lambda\bar{k} - \frac{\sigma^2}{2}\right)T + \sigma W_T + \sum_{i=1}^{n}\ln J_i$$

Since $\ln J_i \sim \mathcal{N}(\mu_J, \sigma_J^2)$ independently:

$$\sum_{i=1}^{n}\ln J_i \sim \mathcal{N}(n\mu_J,\; n\sigma_J^2)$$

Conditional on $N_T = n$, the log-return is normal:

$$\ln\frac{S_T}{S_0}\;\Big|\;N_T = n \;\;\sim\;\; \mathcal{N}\!\left(\left(\mu-\lambda\bar{k}-\frac{\sigma^2}{2}\right)T + n\mu_J,\;\; \sigma^2 T + n\sigma_J^2\right)$$

### Step 4 — Risk-neutral pricing

Under Merton's assumption that jump risk is diversifiable and earns no premium, the risk-neutral dynamics replace $\mu$ with $r$:

$$\frac{dS}{S} = (r - \lambda\bar{k})\,dt + \sigma\,d\widetilde{W}_t + (J-1)\,dN_t$$

The option price is:

$$C = e^{-rT}\,\mathbb{E}^{\mathbb{Q}}\!\left[\max(S_T - K, 0)\right]$$

Condition on the number of jumps $N_T = n$ (Poisson with parameter $\lambda T$):

$$C = e^{-rT}\sum_{n=0}^{\infty}\mathbb{P}(N_T = n)\;\mathbb{E}^{\mathbb{Q}}\!\left[\max(S_T - K,0)\;\Big|\;N_T = n\right]$$

Conditional on $n$ jumps, $S_T$ is lognormal (as shown in Step 3), so the inner expectation is a Black–Scholes formula with modified parameters.

### Step 5 — Merton's closed-form pricing formula

Define modified parameters for the BS formula conditional on $n$ jumps:

**Modified volatility:**

$$\sigma_n^2 = \sigma^2 + \frac{n\sigma_J^2}{T}$$

This adds the jump variance $n\sigma_J^2$ (from $n$ i.i.d. jumps) to the diffusive variance $\sigma^2 T$, then divides by $T$ to get an annualised quantity.

**Modified risk-free rate:**

$$r_n = r - \lambda\bar{k} + \frac{n\ln(1+\bar{k})}{T}$$

The $-\lambda\bar{k}$ removes the drift compensation. The $+n\ln(1+\bar{k})/T$ accounts for the expected growth from $n$ jumps of average size $\bar{k}$.

**Modified Poisson intensity:**

$$\lambda' = \lambda(1 + \bar{k})$$

This is the risk-neutral jump intensity, adjusted for the mean jump size.

**The Merton formula:**

$$\boxed{C_{\text{Merton}} = \sum_{n=0}^{\infty} \frac{e^{-\lambda' T}(\lambda' T)^n}{n!}\;C_{\text{BS}}(S, K, T, r_n, \sigma_n)}$$

where $C_{\text{BS}}(S,K,T,r,\sigma) = S\,N(d_1) - Ke^{-rT}N(d_2)$ is the standard Black–Scholes call price.

The formula is an infinite series, but it converges rapidly because the Poisson weights $e^{-\lambda' T}(\lambda' T)^n/n!$ decay factorially. In practice, 10–20 terms suffice.

### Step 6 — Explicit first three terms

**$n = 0$ (no jumps):**

$$w_0 = e^{-\lambda' T}, \quad r_0 = r - \lambda\bar{k}, \quad \sigma_0 = \sigma$$

$$\text{Term}_0 = e^{-\lambda'T} \cdot C_{\text{BS}}(S, K, T, r_0, \sigma_0)$$

This is the BS price with reduced drift (because we removed the expected jump contribution) weighted by the probability of zero jumps.

**$n = 1$ (one jump):**

$$w_1 = e^{-\lambda' T}\cdot\lambda' T, \quad r_1 = r - \lambda\bar{k} + \frac{\ln(1+\bar{k})}{T}, \quad \sigma_1 = \sqrt{\sigma^2 + \frac{\sigma_J^2}{T}}$$

$$\text{Term}_1 = e^{-\lambda'T}\cdot\lambda'T \cdot C_{\text{BS}}(S, K, T, r_1, \sigma_1)$$

One jump adds extra variance $\sigma_J^2/T$ and shifts the drift by $\ln(1+\bar{k})/T$.

**$n = 2$ (two jumps):**

$$w_2 = e^{-\lambda' T}\cdot\frac{(\lambda' T)^2}{2}, \quad r_2 = r - \lambda\bar{k} + \frac{2\ln(1+\bar{k})}{T}, \quad \sigma_2 = \sqrt{\sigma^2 + \frac{2\sigma_J^2}{T}}$$

$$\text{Term}_2 = e^{-\lambda'T}\cdot\frac{(\lambda'T)^2}{2} \cdot C_{\text{BS}}(S, K, T, r_2, \sigma_2)$$

**Total (first three terms):**

$$C_{\text{Merton}} \approx \text{Term}_0 + \text{Term}_1 + \text{Term}_2 + \ldots$$

### Step 7 — Comparison to Black–Scholes

When $\lambda = 0$ (no jumps), the Merton formula reduces exactly to BS: only the $n=0$ term survives with $\lambda' = 0$, $r_0 = r$, $\sigma_0 = \sigma$.

When $\lambda > 0$, the key differences are:

1. **Fat tails:** The mixture of normals (different $\sigma_n$ for each $n$) produces heavier tails than a single normal. Large moves become more probable.

2. **Volatility skew:** OTM puts become more expensive relative to BS. For downward jumps ($\mu_J < 0$), the left tail is fatter, increasing the implied vol for low strikes — the classic equity skew.

3. **Term structure of implied vol:** Near-expiry options are most affected by jumps (a jump can move the option from OTM to ITM in one step), so short-dated implied vol is elevated relative to BS. Long-dated options are less affected because diffusion dominates over long horizons.

4. **Kurtosis:** The excess kurtosis of Merton log-returns over $[0,T]$ is:

$$\kappa_{\text{excess}} = \frac{\lambda T(\sigma_J^4 + 4\sigma_J^2\mu_J^2 + \mu_J^4 + 6\sigma_J^4)}{(\sigma^2 T + \lambda T(\sigma_J^2 + \mu_J^2))^2}$$

   which is always positive, confirming fatter tails than GBM.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Diffusive vol | $\sigma$ | Continuous-component volatility | 0.10–0.40 |
| Jump intensity | $\lambda$ | Average jumps per year | 0.5–5.0 |
| Mean log-jump | $\mu_J$ | Centre of $\ln J$ distribution | $-0.10$ to $0.05$ (often negative for equities) |
| Jump vol | $\sigma_J$ | Std dev of $\ln J$ | 0.05–0.40 |
| Mean jump size | $\bar{k}$ | $e^{\mu_J + \sigma_J^2/2} - 1$ | $-0.08$ to $0.05$ |
| Risk-free rate | $r$ | Continuous compounding rate | 0.00–0.06 |

## Numerical Example

**Setup:** $S = 100$, $K = 95$ (slightly ITM call), $T = 0.5$, $r = 0.05$, $\sigma = 0.15$, $\lambda = 1.0$, $\mu_J = -0.05$, $\sigma_J = 0.10$.

**Step 1 — Compute derived quantities:**

$$\bar{k} = e^{\mu_J + \sigma_J^2/2} - 1 = e^{-0.05 + 0.005} - 1 = e^{-0.045} - 1 = -0.04401$$

$$\lambda' = \lambda(1 + \bar{k}) = 1.0 \times 0.95599 = 0.95599$$

**Step 2 — First three terms:**

*$n = 0$:*

$$w_0 = e^{-0.95599 \times 0.5} = e^{-0.47800} = 0.62010$$

$$r_0 = 0.05 - 1.0\times(-0.04401) = 0.09401$$

$$\sigma_0 = 0.15$$

$$C_{\text{BS}}(100, 95, 0.5, 0.09401, 0.15) = 10.252$$

$$\text{Term}_0 = 0.62010 \times 10.252 = 6.357$$

*$n = 1$:*

$$w_1 = 0.62010 \times 0.47800 = 0.29641$$

$$r_1 = 0.09401 + \frac{\ln(0.95599)}{0.5} = 0.09401 + \frac{-0.04500}{0.5} = 0.09401 - 0.09000 = 0.00401$$

$$\sigma_1 = \sqrt{0.15^2 + 0.10^2/0.5} = \sqrt{0.0225 + 0.02} = \sqrt{0.0425} = 0.20616$$

$$C_{\text{BS}}(100, 95, 0.5, 0.00401, 0.20616) = 9.134$$

$$\text{Term}_1 = 0.29641 \times 9.134 = 2.707$$

*$n = 2$:*

$$w_2 = 0.62010 \times 0.47800^2 / 2 = 0.07086$$

$$r_2 = 0.09401 - 0.18000 = -0.08599$$

$$\sigma_2 = \sqrt{0.0225 + 0.04} = \sqrt{0.0625} = 0.25$$

$$C_{\text{BS}}(100, 95, 0.5, -0.08599, 0.25) = 7.586$$

$$\text{Term}_2 = 0.07086 \times 7.586 = 0.538$$

**Merton price (first 3 terms):** $\approx 6.357 + 2.707 + 0.538 = 9.602$

**BS price** (same $S, K, T, r, \sigma$): $C_{\text{BS}}(100, 95, 0.5, 0.05, 0.15) = 8.806$

The Merton price is higher because the jump component adds extra variance, increasing option value.

```python
import numpy as np
from scipy.stats import norm

def bs_call(S, K, T, r, sigma):
    d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
    d2 = d1 - sigma*np.sqrt(T)
    return S*norm.cdf(d1) - K*np.exp(-r*T)*norm.cdf(d2)

def merton_call(S, K, T, r, sigma, lam, mu_J, sigma_J, n_terms=20):
    kbar = np.exp(mu_J + 0.5*sigma_J**2) - 1
    lam_prime = lam * (1 + kbar)
    price = 0.0
    for n in range(n_terms):
        sigma_n = np.sqrt(sigma**2 + n*sigma_J**2 / T)
        r_n = r - lam*kbar + n*np.log(1 + kbar) / T
        weight = np.exp(-lam_prime*T) * (lam_prime*T)**n / np.math.factorial(n)
        price += weight * bs_call(S, K, T, r_n, sigma_n)
    return price

S, K, T, r, sigma = 100, 95, 0.5, 0.05, 0.15
lam, mu_J, sigma_J = 1.0, -0.05, 0.10

C_bs = bs_call(S, K, T, r, sigma)
C_merton = merton_call(S, K, T, r, sigma, lam, mu_J, sigma_J)

print(f"BS call     = {C_bs:.3f}")
print(f"Merton call = {C_merton:.3f}")
print(f"Difference  = {C_merton - C_bs:.3f}")
```

## Connections

- [Black–Scholes](black_scholes.md) — Merton's formula uses BS as a building block: each term in the series is a BS price with modified parameters. When $\lambda = 0$, Merton collapses to BS exactly.
- [Brownian motion](../01_stochastic/brownian_motion.md) — The diffusive component of the Merton SDE is driven by Brownian motion. The jump component is independent of the Brownian part.
- [Greeks](greeks.md) — The Merton Greeks are weighted sums of BS Greeks across the series terms. Notably, Merton Gamma is higher near the strike (the jump contribution adds convexity), which affects delta-hedging strategies.
- [Implied volatility](implied_volatility.md) — When you price options with Merton and then invert using BS to find implied vol, you get a volatility skew/smile. This is the key connection: the Merton model *generates* the smile that BS alone cannot explain.
- [Ornstein–Uhlenbeck](../04_stat_arb/ornstein_uhlenbeck.md) — Jump-diffusion models can also describe mean-reverting spreads with occasional regime changes, which is relevant for stat-arb strategies.

## Relevance for Prosperity 4

In Prosperity 4, price dynamics often exhibit jumps — sudden shifts when other bots execute large orders, or when information arrives between rounds. The Merton model is directly applicable in several ways:

- **Calibrating jump parameters:** From historical round data, estimate $\lambda$ (count the number of price moves exceeding 3$\sigma$), $\mu_J$ and $\sigma_J$ (fit a normal distribution to the log-sizes of those jumps), and $\sigma$ (from the remaining continuous returns after removing jumps). Use a simple threshold rule: classify any return $|r_t| > 3\sigma_{\text{diffusive}}$ as a jump.
- **Option pricing with jumps:** If the competition includes option-like instruments, the Merton model gives more accurate fair values than BS, especially for OTM puts. Quoting OTM puts at BS prices when jumps are present means you are systematically under-pricing crash risk.
- **Risk management:** The expected loss from a jump event is $\lambda \cdot \mathbb{E}[|J-1|] \cdot |q| \cdot S$ per unit time, where $q$ is your inventory. If this exceeds your risk budget, reduce position size regardless of what the continuous-time model suggests.
- **Spread widening:** When jump risk is high (e.g., volatile periods in the competition), widen your bid-ask spread to compensate for the possibility that the price jumps through your quote before you can cancel it. The extra spread should be approximately $\lambda\,\mathbb{E}[|J-1|]\,S\,\Delta t$, where $\Delta t$ is your cancel latency.
