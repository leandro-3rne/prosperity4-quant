# Cointegration

> **Core formula:** $$s_t = Y_t - \beta X_t \;\sim\; I(0) \quad\text{(stationary)}$$

## Intuition

Correlation and cointegration are fundamentally different concepts, and confusing them is one of the most common mistakes in quantitative trading. Correlation measures whether two series tend to move in the same direction over short intervals — their returns are linearly related. Cointegration is a much stronger, structural claim: two series share a long-run equilibrium, so their linear combination is mean-reverting. Correlated series can drift arbitrarily far apart over time; cointegrated series cannot.

The classic analogy: imagine a drunk person walking home — their path is a random walk, unpredictable. Now imagine two drunk friends who happen to be walking in the same general direction. Their paths are *correlated* (similar drift, similar randomness), but the distance between them can grow without bound. Now imagine two drunks tied together by a rope. Each one still staggers randomly, but they can never be further apart than the length of the rope. When one drifts too far, the rope pulls them back. That rope is cointegration — it enforces a maximum deviation, guaranteeing mean-reversion in the spread.

In pairs trading, we seek cointegrated assets: individually, each price is a non-stationary random walk ($I(1)$), but their spread $s_t = Y_t - \beta X_t$ is stationary ($I(0)$), fluctuating around a stable mean. When the spread deviates far from this mean, we trade the expectation that it will revert.

## Mathematical Setup

**Integrated processes.** A time series $Z_t$ is said to be integrated of order $d$, written $Z_t \sim I(d)$, if it must be differenced $d$ times to become stationary.
- $I(0)$: stationary — bounded variance, mean-reverting. Example: white noise, OU process.
- $I(1)$: unit root — a random walk. The series itself is non-stationary, but first differences $\Delta Z_t = Z_t - Z_{t-1}$ are stationary. Example: $Z_t = Z_{t-1} + \varepsilon_t$.

**Cointegration definition.** Two $I(1)$ series $Y_t$ and $X_t$ are cointegrated if there exists a constant $\beta$ (the cointegrating vector / hedge ratio) such that:

$$
s_t = Y_t - \beta X_t \;\sim\; I(0)
$$

The spread $s_t$ is stationary even though $Y_t$ and $X_t$ individually are not.

**Key distinction from correlation:**

| Property | Correlation | Cointegration |
|----------|-------------|---------------|
| Measures | Co-movement of returns (short-run) | Long-run equilibrium relationship |
| Can series diverge? | Yes, unboundedly | No, spread is mean-reverting |
| Stationarity of linear combination | Not required | Required (defining property) |
| Stability over time | Can change rapidly | More structural, slower to break |

## Derivation

### Step 1: Engle-Granger Two-Step Procedure

The Engle-Granger method (1987) is the standard test for cointegration between two series.

**Step 1a: Cointegrating regression.** Run an OLS regression of $Y_t$ on $X_t$:

$$
Y_t = \hat{\alpha} + \hat{\beta}\,X_t + \hat{\varepsilon}_t
$$

The OLS estimator for the hedge ratio is:

$$
\hat{\beta} = \frac{\text{Cov}(Y_t, X_t)}{\text{Var}(X_t)} = \frac{\sum_{t=1}^{T}(Y_t - \bar{Y})(X_t - \bar{X})}{\sum_{t=1}^{T}(X_t - \bar{X})^2}
$$

The intercept is:

$$
\hat{\alpha} = \bar{Y} - \hat{\beta}\,\bar{X}
$$

Under cointegration, the OLS estimator $\hat{\beta}$ is *super-consistent*: it converges to the true $\beta$ at rate $T$ (not $\sqrt{T}$), which means even a relatively short sample gives a precise hedge ratio.

**Step 1b: Extract the residuals (spread).**

$$
\hat{\varepsilon}_t = Y_t - \hat{\alpha} - \hat{\beta}\,X_t
$$

These residuals are the estimated spread. If $Y$ and $X$ are truly cointegrated, $\hat{\varepsilon}_t$ should be stationary.

**Step 2: Test the residuals for stationarity** using the Augmented Dickey-Fuller (ADF) test.

### Step 2: The Augmented Dickey-Fuller (ADF) Test

The ADF test checks whether a time series has a unit root (is non-stationary).

**Test equation.** Regress:

$$
\Delta\hat{\varepsilon}_t = \rho\,\hat{\varepsilon}_{t-1} + \sum_{j=1}^{p} c_j\,\Delta\hat{\varepsilon}_{t-j} + u_t
$$

where $\Delta\hat{\varepsilon}_t = \hat{\varepsilon}_t - \hat{\varepsilon}_{t-1}$, the lagged differences $\Delta\hat{\varepsilon}_{t-j}$ control for serial correlation, and $u_t$ is white noise.

**Hypotheses:**

$$
H_0: \rho = 0 \quad\text{(unit root} \Rightarrow \text{no cointegration)}
$$

$$
H_1: \rho < 0 \quad\text{(stationary} \Rightarrow \text{cointegrated)}
$$

**Test statistic.** Compute the t-statistic for $\hat{\rho}$:

$$
\text{ADF stat} = \frac{\hat{\rho}}{\text{SE}(\hat{\rho})}
$$

This statistic does **not** follow a standard $t$-distribution. It follows the Dickey-Fuller distribution, with critical values tabulated by MacKinnon.

**Critical values** (no trend, approximately 250 observations):

| Significance level | Critical value |
|--------------------|---------------|
| 1% | $-3.43$ |
| 5% | $-2.86$ |
| 10% | $-2.57$ |

**Decision rule:** If $\text{ADF stat} < \text{critical value}$, reject $H_0$ and conclude the residuals are stationary (i.e., the pair is cointegrated at that significance level).

**Important caveat:** When testing residuals from a cointegrating regression (Engle-Granger Step 2), the critical values should technically be the Engle-Granger/MacKinnon cointegration-specific critical values, which are more negative than the standard ADF values because $\hat{\beta}$ was estimated in the first step. For two variables with no trend ($\sim$250 obs): 1%: $-3.90$, 5%: $-3.34$, 10%: $-3.04$.

### Step 3: Hedge Ratio Estimation — Details

The hedge ratio $\hat{\beta}$ from OLS has a clean interpretation: for every 1 unit of $X$ held short, hold $\hat{\beta}^{-1}$ units of $Y$ long (or equivalently, for each unit of $Y$ long, hold $\hat{\beta}$ units of $X$ short). The spread is:

$$
s_t = Y_t - \hat{\beta}\,X_t
$$

In practice, $\hat{\beta}$ should be re-estimated on a rolling window to adapt to slowly changing equilibrium relationships. A common window length is 60–120 observations.

### Step 4: Practical Checklist for Testing a Pair

1. **Confirm both series are $I(1)$.** Run ADF on levels of $Y_t$ and $X_t$ separately. Both should *fail* to reject $H_0$ (non-stationary). Then run ADF on $\Delta Y_t$ and $\Delta X_t$. Both should reject $H_0$ (stationary first differences).

2. **Run the cointegrating regression.** Regress $Y_t = \alpha + \beta X_t + \varepsilon_t$ via OLS. Record $\hat{\beta}$.

3. **Test residuals for stationarity.** Run ADF on $\hat{\varepsilon}_t$. If the ADF statistic is more negative than the critical value, the pair is cointegrated.

4. **Estimate OU parameters on the spread.** Fit $\hat{\varepsilon}_t$ to an OU process to get $\hat{\theta}$, $\hat{\mu}$, $\hat{\sigma}$ (see [ornstein_uhlenbeck.md](./ornstein_uhlenbeck.md)).

5. **Check the half-life.** Compute $t_{1/2} = \ln 2 / \hat{\theta}$. If the half-life is longer than the trading horizon, the mean-reversion is too slow to exploit.

6. **Check stability.** Repeat steps 2–5 on rolling windows. If $\hat{\beta}$ or $\hat{\theta}$ vary wildly, the cointegration relationship is unstable and unreliable.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Hedge ratio | $\beta$ | Units of $X$ per unit of $Y$ in the spread | Asset dependent, estimated via OLS |
| Intercept | $\alpha$ | Level offset in cointegrating regression | Asset dependent |
| ADF statistic | $\text{ADF}$ | Test statistic for unit root in residuals | Must be $< -2.86$ (5% level) to reject |
| Lag order | $p$ | Number of lagged differences in ADF regression | 1–10, selected by AIC/BIC |
| Rolling window | $L$ | Number of observations for re-estimating $\hat{\beta}$ | 60–120 for daily data |
| Stationarity order | $I(d)$ | Integration order of the series | Must be $I(1)$ for both legs |

## Numerical Example

**Setup:** Two price series observed over 5 periods.

| $t$ | $X_t$ | $Y_t$ |
|-----|--------|--------|
| 1 | 50.0 | 102.0 |
| 2 | 51.0 | 104.5 |
| 3 | 49.5 | 101.2 |
| 4 | 52.0 | 106.0 |
| 5 | 50.5 | 103.1 |

**Step 1: Estimate hedge ratio via OLS.**

$$
\bar{X} = \frac{50 + 51 + 49.5 + 52 + 50.5}{5} = 50.6
$$

$$
\bar{Y} = \frac{102 + 104.5 + 101.2 + 106 + 103.1}{5} = 103.36
$$

$$
\sum(X_t - \bar{X})^2 = (-0.6)^2 + (0.4)^2 + (-1.1)^2 + (1.4)^2 + (-0.1)^2 = 0.36 + 0.16 + 1.21 + 1.96 + 0.01 = 3.70
$$

$$
\sum(X_t - \bar{X})(Y_t - \bar{Y}) = (-0.6)(-1.36) + (0.4)(1.14) + (-1.1)(-2.16) + (1.4)(2.64) + (-0.1)(-0.26)
$$

$$
= 0.816 + 0.456 + 2.376 + 3.696 + 0.026 = 7.370
$$

$$
\hat{\beta} = \frac{7.370}{3.70} = 1.9919
$$

$$
\hat{\alpha} = 103.36 - 1.9919 \times 50.6 = 103.36 - 100.79 = 2.57
$$

**Step 2: Compute residuals (spread).**

$$
\hat{\varepsilon}_1 = 102.0 - 2.57 - 1.9919 \times 50.0 = 102.0 - 2.57 - 99.60 = -0.17
$$

$$
\hat{\varepsilon}_2 = 104.5 - 2.57 - 1.9919 \times 51.0 = 104.5 - 2.57 - 101.59 = 0.34
$$

$$
\hat{\varepsilon}_3 = 101.2 - 2.57 - 1.9919 \times 49.5 = 101.2 - 2.57 - 98.60 = 0.03
$$

$$
\hat{\varepsilon}_4 = 106.0 - 2.57 - 1.9919 \times 52.0 = 106.0 - 2.57 - 103.58 = -0.15
$$

$$
\hat{\varepsilon}_5 = 103.1 - 2.57 - 1.9919 \times 50.5 = 103.1 - 2.57 - 100.59 = -0.06
$$

The residuals fluctuate around zero with small magnitude — consistent with stationarity. In practice, we would run ADF on a much longer series to formally test.

```python
import numpy as np

X = np.array([50.0, 51.0, 49.5, 52.0, 50.5])
Y = np.array([102.0, 104.5, 101.2, 106.0, 103.1])

beta_hat = np.cov(Y, X, ddof=0)[0, 1] / np.var(X, ddof=0)
alpha_hat = Y.mean() - beta_hat * X.mean()
residuals = Y - alpha_hat - beta_hat * X

print(f"beta_hat  = {beta_hat:.4f}")
print(f"alpha_hat = {alpha_hat:.4f}")
print(f"residuals = {np.round(residuals, 4)}")
print(f"residual mean = {residuals.mean():.6f}")
print(f"residual std  = {residuals.std(ddof=1):.4f}")
```

## Connections

- [Ornstein-Uhlenbeck](./ornstein_uhlenbeck.md): the spread from a cointegrated pair is modeled as an OU process. After confirming cointegration, we estimate OU parameters $(\theta, \mu, \sigma)$ on the spread to determine mean-reversion speed and trading viability.
- [Z-Score Signals](./zscore_signals.md): once the spread is constructed and OU parameters estimated, the z-score normalizes the spread for signal generation. Entry/exit thresholds are expressed in z-score units.
- [Brownian Motion](../01_stochastic/brownian_motion.md): each individual price $Y_t, X_t$ is modeled as having a unit root (integrated Brownian motion), but their linear combination cancels the random walk component, leaving a stationary residual.
- [Prosperity 3 Analysis](../06_prosperity/prosperity3_analysis.md): past Prosperity rounds have featured product pairs with structural relationships (e.g., gift baskets and their components), which are natural candidates for cointegration testing.

## Relevance for Prosperity 4

Cointegration is the foundational test for identifying pairs-tradeable products in Prosperity. The competition features multiple correlated products — for example, a basket product and its constituents, or two commodities with shared supply/demand drivers. The workflow is: (1) test all product pairs for cointegration using the Engle-Granger procedure on historical round data, (2) estimate the hedge ratio $\hat{\beta}$ to construct the spread, (3) verify the half-life is short enough relative to the round length (a half-life of 10–50 ticks is ideal; 200+ ticks is too slow), and (4) feed the spread into the z-score signal framework. Because Prosperity provides new data each round, re-testing cointegration on updated data is essential — a pair that was cointegrated in Round 1 may lose its equilibrium relationship in Round 3 due to changing bot behaviors or market structure. Rolling-window stability checks are critical for robustness.

## Python Snippet

```python
import numpy as np
from statsmodels.tsa.stattools import adfuller

def engle_granger_test(Y, X, adf_maxlag=None):
    """Run the Engle-Granger cointegration test on two series."""
    n = len(Y)
    beta_hat = np.cov(Y, X, ddof=0)[0, 1] / np.var(X, ddof=0)
    alpha_hat = Y.mean() - beta_hat * X.mean()
    spread = Y - alpha_hat - beta_hat * X

    adf_result = adfuller(spread, maxlag=adf_maxlag, regression="c")
    adf_stat, p_value = adf_result[0], adf_result[1]

    return {
        "beta": beta_hat,
        "alpha": alpha_hat,
        "spread": spread,
        "adf_stat": adf_stat,
        "p_value": p_value,
        "cointegrated_5pct": adf_stat < -2.86,
    }

rng = np.random.default_rng(42)
n = 500
X = np.cumsum(rng.normal(0, 1, n))
Y = 2.0 * X + 3.0 + np.cumsum(rng.normal(0, 0.1, n)) * 0  # cointegrated
Y += rng.normal(0, 0.5, n)  # add stationary noise

result = engle_granger_test(Y, X)
print(f"beta = {result['beta']:.4f}, ADF = {result['adf_stat']:.4f}, "
      f"p = {result['p_value']:.4f}, cointegrated = {result['cointegrated_5pct']}")
```
