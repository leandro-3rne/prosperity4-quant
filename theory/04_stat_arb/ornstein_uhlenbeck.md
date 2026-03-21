# Ornstein-Uhlenbeck Process

> **Core formula:** $$dX_t = \theta(\mu - X_t)\,dt + \sigma\,dW_t$$

## Intuition

Imagine a rubber band attached to a post at position $\mu$. When you stretch the band away from the post and release it, it snaps back — quickly at first when the displacement is large, then more slowly as it approaches the post. The Ornstein-Uhlenbeck (OU) process captures exactly this behavior, but with random noise constantly jostling the band. The parameter $\theta$ controls how stiff the rubber band is: a large $\theta$ means a strong, fast snap-back; a small $\theta$ means a lazy, slow drift toward the center.

In finance, the OU process is the workhorse model for any quantity that fluctuates around a stable level — interest rate spreads, pairs-trading residuals, implied volatility deviations, or any mean-reverting signal. Unlike Geometric Brownian Motion (which can drift to infinity), the OU process is pulled back to its long-run mean. This makes it the natural model for the spread $s_t = Y_t - \beta X_t$ in statistical arbitrage: when the spread deviates far from $\mu$, we expect it to revert, and we trade that expectation.

The OU process is also the continuous-time analogue of a discrete AR(1) process, which connects the elegant stochastic calculus formulation directly to simple linear regression — the primary tool we use to estimate its parameters from data.

## Mathematical Setup

**State variable:** $X_t \in \mathbb{R}$, the value of the mean-reverting process at time $t$.

**Parameters:**
- $\theta > 0$: mean-reversion speed (units: $\text{time}^{-1}$)
- $\mu \in \mathbb{R}$: long-run equilibrium level
- $\sigma > 0$: volatility of the noise (units: $\text{value} \cdot \text{time}^{-1/2}$)

**Driving noise:** $W_t$ is a standard Brownian motion with $W_0 = 0$, independent Gaussian increments, and $\mathbb{E}[dW_t] = 0$, $\text{Var}(dW_t) = dt$.

**Initial condition:** $X_0$ is given (deterministic or random, independent of $W$).

**SDE:**

$$
dX_t = \theta(\mu - X_t)\,dt + \sigma\,dW_t
$$

The drift term $\theta(\mu - X_t)$ is negative when $X_t > \mu$ (pulling $X$ down) and positive when $X_t < \mu$ (pulling $X$ up). The diffusion term $\sigma\,dW_t$ adds Gaussian noise.

## Derivation

### Step 1: Exact Solution via Integrating Factor

We want to solve the SDE $dX_t = \theta(\mu - X_t)\,dt + \sigma\,dW_t$. Rearrange:

$$
dX_t + \theta X_t\,dt = \theta\mu\,dt + \sigma\,dW_t
$$

Multiply both sides by the integrating factor $e^{\theta t}$:

$$
e^{\theta t}\,dX_t + \theta e^{\theta t} X_t\,dt = e^{\theta t}\theta\mu\,dt + e^{\theta t}\sigma\,dW_t
$$

Recognize that the left-hand side is the differential of the product $e^{\theta t} X_t$. By the product rule for Itô processes (note that $e^{\theta t}$ is a deterministic, smooth function, so no second-order Itô correction arises):

$$
d\!\left(e^{\theta t} X_t\right) = e^{\theta t}\,dX_t + \theta e^{\theta t} X_t\,dt
$$

Therefore:

$$
d\!\left(e^{\theta t} X_t\right) = \theta\mu\,e^{\theta t}\,dt + \sigma\,e^{\theta t}\,dW_t
$$

### Step 2: Integrate Both Sides from $0$ to $t$

$$
e^{\theta t} X_t - e^{0} X_0 = \int_0^t \theta\mu\,e^{\theta s}\,ds + \int_0^t \sigma\,e^{\theta s}\,dW_s
$$

Evaluate the deterministic integral:

$$
\int_0^t \theta\mu\,e^{\theta s}\,ds = \theta\mu \cdot \frac{e^{\theta s}}{\theta}\Bigg|_{s=0}^{s=t} = \mu\!\left(e^{\theta t} - 1\right)
$$

Substituting:

$$
e^{\theta t} X_t = X_0 + \mu\!\left(e^{\theta t} - 1\right) + \sigma\int_0^t e^{\theta s}\,dW_s
$$

### Step 3: Solve for $X_t$

Divide both sides by $e^{\theta t}$:

$$
X_t = X_0\,e^{-\theta t} + \mu\!\left(1 - e^{-\theta t}\right) + \sigma\int_0^t e^{-\theta(t-s)}\,dW_s
$$

Rearranging:

$$
\boxed{X_t = \mu + (X_0 - \mu)\,e^{-\theta t} + \sigma\int_0^t e^{-\theta(t-s)}\,dW_s}
$$

The three terms are: (1) the long-run mean, (2) a deterministic decay of the initial deviation, and (3) a stochastic integral representing accumulated noise.

### Step 4: Expectation

Since the Itô integral has zero expectation ($\mathbb{E}\!\left[\int_0^t f(s)\,dW_s\right] = 0$ for any adapted $f$ with $\mathbb{E}\!\left[\int_0^t f(s)^2\,ds\right] < \infty$):

$$
\mathbb{E}[X_t] = \mu + (X_0 - \mu)\,e^{-\theta t}
$$

As $t \to \infty$:

$$
\mathbb{E}[X_t] \;\longrightarrow\; \mu
$$

### Step 5: Variance via Itô Isometry

The Itô isometry states that for a deterministic integrand $g(s)$:

$$
\text{Var}\!\left(\int_0^t g(s)\,dW_s\right) = \mathbb{E}\!\left[\left(\int_0^t g(s)\,dW_s\right)^2\right] = \int_0^t g(s)^2\,ds
$$

Apply this with $g(s) = \sigma\,e^{-\theta(t-s)}$:

$$
\text{Var}(X_t) = \int_0^t \sigma^2\,e^{-2\theta(t-s)}\,ds
$$

Substitute $u = t - s$, so $du = -ds$, and when $s=0$ we get $u=t$, when $s=t$ we get $u=0$:

$$
\text{Var}(X_t) = \int_0^t \sigma^2\,e^{-2\theta u}\,du
$$

Evaluate:

$$
\text{Var}(X_t) = \sigma^2 \cdot \left[-\frac{e^{-2\theta u}}{2\theta}\right]_0^t = \sigma^2 \cdot \frac{1 - e^{-2\theta t}}{2\theta}
$$

$$
\boxed{\text{Var}(X_t) = \frac{\sigma^2}{2\theta}\!\left(1 - e^{-2\theta t}\right)}
$$

As $t \to \infty$:

$$
\text{Var}(X_t) \;\longrightarrow\; \frac{\sigma^2}{2\theta}
$$

### Step 6: Stationary Distribution

Since $X_t$ is a Gaussian process (as a linear functional of Gaussian noise), its distribution is fully characterized by its mean and variance. In the long run:

$$
\boxed{X_\infty \sim \mathcal{N}\!\left(\mu,\;\frac{\sigma^2}{2\theta}\right)}
$$

The stationary standard deviation is $\sigma_\infty = \sigma / \sqrt{2\theta}$.

### Step 7: Half-Life of Mean Reversion

The expected deviation from the mean decays exponentially:

$$
\mathbb{E}[X_t] - \mu = (X_0 - \mu)\,e^{-\theta t}
$$

The half-life $t_{1/2}$ is the time for this deviation to shrink to half its initial value:

$$
(X_0 - \mu)\,e^{-\theta t_{1/2}} = \frac{1}{2}(X_0 - \mu)
$$

Divide both sides by $(X_0 - \mu)$:

$$
e^{-\theta t_{1/2}} = \frac{1}{2}
$$

Take the natural logarithm:

$$
-\theta\,t_{1/2} = \ln\!\left(\frac{1}{2}\right) = -\ln 2
$$

$$
\boxed{t_{1/2} = \frac{\ln 2}{\theta}}
$$

### Step 8: MLE Parameter Estimation from Discrete Observations

Suppose we observe $X_0, X_1, \ldots, X_n$ at equally spaced times $0, \Delta t, 2\Delta t, \ldots, n\Delta t$.

**Conditional distribution.** From the exact solution, conditional on $X_i$:

$$
X_{i+1} \mid X_i \;\sim\; \mathcal{N}\!\left(\mu + (X_i - \mu)\,e^{-\theta\Delta t},\;\frac{\sigma^2(1 - e^{-2\theta\Delta t})}{2\theta}\right)
$$

**AR(1) representation.** Define $a = \mu(1 - e^{-\theta\Delta t})$, $b = e^{-\theta\Delta t}$, and $\sigma_\varepsilon^2 = \sigma^2(1 - e^{-2\theta\Delta t})/(2\theta)$. Then:

$$
X_{i+1} = a + b\,X_i + \varepsilon_i, \quad \varepsilon_i \sim \mathcal{N}(0,\;\sigma_\varepsilon^2)
$$

This is a standard AR(1) model, estimable by OLS regression of $X_{i+1}$ on $X_i$.

**OLS estimators.** Define the notation $\bar{X} = \frac{1}{n}\sum_{i=0}^{n-1} X_i$ and $\bar{X}_+ = \frac{1}{n}\sum_{i=1}^{n} X_i$. Then:

$$
\hat{b} = \frac{\sum_{i=0}^{n-1}(X_i - \bar{X})(X_{i+1} - \bar{X}_+)}{\sum_{i=0}^{n-1}(X_i - \bar{X})^2}
$$

$$
\hat{a} = \bar{X}_+ - \hat{b}\,\bar{X}
$$

**Recovering continuous-time parameters:**

$$
\hat{\theta} = -\frac{\ln \hat{b}}{\Delta t}
$$

$$
\hat{\mu} = \frac{\hat{a}}{1 - \hat{b}}
$$

For the volatility, compute the residual variance $\hat{\sigma}_\varepsilon^2 = \frac{1}{n-2}\sum_{i=0}^{n-1}(X_{i+1} - \hat{a} - \hat{b}\,X_i)^2$, then:

$$
\hat{\sigma}^2 = \frac{2\hat{\theta}\,\hat{\sigma}_\varepsilon^2}{1 - \hat{b}^2}
$$

Equivalently, since $\hat{b}^2 = e^{-2\hat{\theta}\Delta t}$, this is $\hat{\sigma}^2 = 2\hat{\theta}\,\hat{\sigma}_\varepsilon^2 / (1 - e^{-2\hat{\theta}\Delta t})$.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Mean-reversion speed | $\theta$ | Rate at which $X_t$ is pulled toward $\mu$ | 0.5 – 50 $\text{day}^{-1}$ for intraday spreads |
| Long-run mean | $\mu$ | Equilibrium level of the process | Asset/spread dependent |
| Volatility | $\sigma$ | Intensity of random fluctuations | Asset dependent |
| Half-life | $t_{1/2} = \ln 2/\theta$ | Time for expected deviation to halve | Minutes to days in stat-arb |
| Stationary std dev | $\sigma/\sqrt{2\theta}$ | Long-run fluctuation scale | Sets z-score normalization |

## Numerical Example

**Setup:** $\theta = 5.0\;\text{day}^{-1}$, $\mu = 100$, $\sigma = 2.0$, $X_0 = 103$, $\Delta t = 1/252$ (one trading day).

**Half-life:**

$$
t_{1/2} = \frac{\ln 2}{5.0} = \frac{0.6931}{5.0} = 0.1386\;\text{days} \approx 0.99\;\text{trading hours}
$$

**Stationary standard deviation:**

$$
\sigma_\infty = \frac{2.0}{\sqrt{2 \times 5.0}} = \frac{2.0}{\sqrt{10}} = \frac{2.0}{3.162} = 0.6325
$$

**Expected value after 0.5 days:**

$$
\mathbb{E}[X_{0.5}] = 100 + (103 - 100)\,e^{-5.0 \times 0.5} = 100 + 3\,e^{-2.5} = 100 + 3 \times 0.0821 = 100.246
$$

**Variance after 0.5 days:**

$$
\text{Var}(X_{0.5}) = \frac{4.0}{10}(1 - e^{-10 \times 0.5}) = 0.4\,(1 - e^{-5}) = 0.4 \times 0.9933 = 0.3973
$$

**AR(1) parameters for daily observations ($\Delta t = 1/252$):**

$$
b = e^{-5.0/252} = e^{-0.01984} = 0.9804
$$

$$
a = 100 \times (1 - 0.9804) = 100 \times 0.0196 = 1.9604
$$

```python
import numpy as np

theta, mu, sigma, X0 = 5.0, 100.0, 2.0, 103.0
half_life = np.log(2) / theta
stat_std = sigma / np.sqrt(2 * theta)

t = 0.5
E_Xt = mu + (X0 - mu) * np.exp(-theta * t)
Var_Xt = (sigma**2 / (2 * theta)) * (1 - np.exp(-2 * theta * t))

print(f"Half-life:         {half_life:.4f} days")
print(f"Stationary std:    {stat_std:.4f}")
print(f"E[X_0.5]:          {E_Xt:.4f}")
print(f"Var(X_0.5):        {Var_Xt:.4f}")
print(f"Std(X_0.5):        {np.sqrt(Var_Xt):.4f}")
```

## Connections

- [Cointegration](./cointegration.md): the spread $s_t = Y_t - \beta X_t$ from a cointegrated pair is modeled as an OU process. The OU parameters $(\theta, \mu, \sigma)$ are estimated from the spread to determine trading viability.
- [Z-Score Signals](./zscore_signals.md): the z-score $z_t = (s_t - \hat{\mu})/\hat{\sigma}$ uses the OU stationary mean and standard deviation as normalizing constants. Entry/exit thresholds are set in units of the stationary distribution.
- [Brownian Motion](../01_stochastic/brownian_motion.md): the OU process is driven by Brownian motion $W_t$. When $\theta = 0$, the OU process degenerates to a drifting Brownian motion $\mu t + \sigma W_t$.
- [Avellaneda-Stoikov](../03_market_making/avellaneda_stoikov.md): in market making, the mid-price is modeled as BM, but the inventory penalty term introduces a mean-reverting structure similar to OU, pulling the reservation price toward the mid.

## Relevance for Prosperity 4

In IMC Prosperity, several product pairs exhibit mean-reverting spread dynamics — for example, synthetic baskets versus their constituents, or related commodities. The OU model provides a principled framework for this: (1) estimate $\theta$, $\mu$, $\sigma$ from the rolling spread history using the AR(1)/OLS procedure, (2) compute the half-life $\ln 2 / \hat{\theta}$ to confirm the spread reverts within the competition's trading horizon (if the half-life is longer than the round length, the trade is unlikely to pay off), and (3) use the stationary distribution $\mathcal{N}(\hat{\mu}, \hat{\sigma}^2/(2\hat{\theta}))$ to normalize the spread into z-scores for signal generation. Because Prosperity rounds are short (typically hundreds of ticks), only pairs with fast mean-reversion ($t_{1/2}$ on the order of tens of ticks) are actionable. The discretized AR(1) estimators can be computed in a single pass over the price history, making them suitable for the competition's latency constraints.

## Python Snippet

```python
import numpy as np

def simulate_ou(theta, mu, sigma, X0, T, n_steps, rng=None):
    """Simulate an OU process via Euler-Maruyama."""
    if rng is None:
        rng = np.random.default_rng(42)
    dt = T / n_steps
    X = np.zeros(n_steps + 1)
    X[0] = X0
    for i in range(n_steps):
        dW = rng.normal(0, np.sqrt(dt))
        X[i + 1] = X[i] + theta * (mu - X[i]) * dt + sigma * dW
    return np.linspace(0, T, n_steps + 1), X

def estimate_ou_params(X, dt):
    """Estimate OU parameters from equally spaced observations via OLS."""
    Y = X[1:]
    Xlag = X[:-1]
    n = len(Xlag)
    b_hat = np.sum((Xlag - Xlag.mean()) * (Y - Y.mean())) / np.sum((Xlag - Xlag.mean())**2)
    a_hat = Y.mean() - b_hat * Xlag.mean()
    theta_hat = -np.log(b_hat) / dt
    mu_hat = a_hat / (1 - b_hat)
    residuals = Y - a_hat - b_hat * Xlag
    sigma_eps2 = np.var(residuals, ddof=2)
    sigma_hat = np.sqrt(2 * theta_hat * sigma_eps2 / (1 - b_hat**2))
    half_life = np.log(2) / theta_hat
    return {"theta": theta_hat, "mu": mu_hat, "sigma": sigma_hat, "half_life": half_life}

t, X = simulate_ou(5.0, 100.0, 2.0, 103.0, T=1.0, n_steps=252)
params = estimate_ou_params(X, dt=1/252)
print(params)
```
