# Z-Score Signals

> **Core formula:** $$z_t = \frac{s_t - \hat{\mu}_t}{\hat{\sigma}_t}$$

## Intuition

Once we have a mean-reverting spread from a cointegrated pair, we need a way to decide *when* to trade. The raw spread value is hard to interpret directly — is a spread of 3.5 "large"? That depends on how much it normally fluctuates. The z-score solves this by normalizing the spread: it measures how many standard deviations the current spread is from its recent mean. A z-score of $+2$ means the spread is two standard deviations above its mean — unusually high, suggesting it will likely revert downward. A z-score of $-2$ means it's unusually low and likely to revert upward.

Think of the z-score as a thermometer for the spread. Just as a body temperature of 39 degrees Celsius tells you something is wrong (relative to the normal 37 degrees), a z-score of $\pm 2$ tells you the spread has deviated enough from "normal" to be worth trading. The thresholds you choose (entry at $|z| > 2$, exit at $|z| < 0.5$) define your trading strategy's aggressiveness: higher thresholds mean fewer but higher-conviction trades.

The z-score framework also naturally handles position sizing and risk management. Because the z-score is in standardized units, we can set universal stop-loss levels (e.g., $|z| > 4$ suggests the relationship may have broken down) regardless of the specific spread's scale or volatility.

## Mathematical Setup

**Spread:** Given a cointegrated pair $(Y_t, X_t)$ with hedge ratio $\beta$:

$$
s_t = Y_t - \beta\,X_t
$$

**Lookback window:** $L \in \mathbb{N}$, the number of recent observations used to estimate the rolling mean and standard deviation.

**Rolling statistics (simple moving average):**

$$
\hat{\mu}_t = \frac{1}{L}\sum_{i=0}^{L-1} s_{t-i}
$$

$$
\hat{\sigma}_t = \sqrt{\frac{1}{L-1}\sum_{i=0}^{L-1}(s_{t-i} - \hat{\mu}_t)^2}
$$

**Z-score:**

$$
z_t = \frac{s_t - \hat{\mu}_t}{\hat{\sigma}_t}
$$

**EMA-based alternative:** For faster adaptation to regime changes, replace the simple rolling mean with an exponential moving average:

$$
\hat{\mu}_t^{\text{EMA}} = \lambda\,s_t + (1-\lambda)\,\hat{\mu}_{t-1}^{\text{EMA}}, \quad \lambda = \frac{2}{L+1}
$$

$$
\hat{v}_t^{\text{EMA}} = \lambda\,(s_t - \hat{\mu}_t^{\text{EMA}})^2 + (1-\lambda)\,\hat{v}_{t-1}^{\text{EMA}}
$$

$$
\hat{\sigma}_t^{\text{EMA}} = \sqrt{\hat{v}_t^{\text{EMA}}}
$$

$$
z_t^{\text{EMA}} = \frac{s_t - \hat{\mu}_t^{\text{EMA}}}{\hat{\sigma}_t^{\text{EMA}}}
$$

The EMA z-score places exponentially more weight on recent observations, making it more responsive to changing dynamics but also noisier.

## Derivation

### Step 1: Rolling Z-Score Construction

Given observations $s_1, s_2, \ldots, s_T$ and a lookback window $L$, the z-score at time $t \geq L$ is:

$$
\hat{\mu}_t = \frac{1}{L}\sum_{i=0}^{L-1} s_{t-i} = \frac{s_t + s_{t-1} + \cdots + s_{t-L+1}}{L}
$$

$$
\hat{\sigma}_t^2 = \frac{1}{L-1}\sum_{i=0}^{L-1}(s_{t-i} - \hat{\mu}_t)^2
$$

Expanding the variance using the computational formula:

$$
\hat{\sigma}_t^2 = \frac{1}{L-1}\left(\sum_{i=0}^{L-1} s_{t-i}^2 - L\,\hat{\mu}_t^2\right)
$$

The z-score is then:

$$
z_t = \frac{s_t - \hat{\mu}_t}{\hat{\sigma}_t}
$$

**Interpretation under the OU model.** If $s_t$ follows an OU process with stationary distribution $\mathcal{N}(\mu, \sigma^2/(2\theta))$, then in the stationary regime, $z_t$ is approximately standard normal $\mathcal{N}(0,1)$. The probability of $|z_t| > 2$ under the null (stationary OU) is:

$$
P(|z| > 2) = 2\,\Phi(-2) = 2 \times 0.0228 = 0.0456 \approx 4.6\%
$$

This means a z-score beyond $\pm 2$ occurs only about 4.6% of the time if the spread is behaving normally — these are the rare, high-conviction trading opportunities.

### Step 2: Entry and Exit Thresholds

Define two threshold parameters:
- $z_{\text{entry}}$: the z-score level at which we open a position (typically $2.0$)
- $z_{\text{exit}}$: the z-score level at which we close a position (typically $0.5$)

**Trading rules:**

| Signal | Condition | Action |
|--------|-----------|--------|
| Enter long spread | $z_t < -z_{\text{entry}}$ | Buy $Y$, sell $\beta$ units of $X$ |
| Enter short spread | $z_t > +z_{\text{entry}}$ | Sell $Y$, buy $\beta$ units of $X$ |
| Exit long spread | $z_t > -z_{\text{exit}}$ | Close the long-spread position |
| Exit short spread | $z_t < +z_{\text{exit}}$ | Close the short-spread position |

**Rationale:**
- When $z_t < -z_{\text{entry}}$, the spread is unusually low relative to its mean. Under mean-reversion, we expect $s_t$ to increase (revert toward $\hat{\mu}_t$), so we *buy* the spread (long $Y$, short $X$).
- When $z_t > +z_{\text{entry}}$, the spread is unusually high. We expect it to decrease, so we *sell* the spread (short $Y$, long $X$).
- We exit when the z-score returns to near zero (within $z_{\text{exit}}$), as the mean-reversion opportunity has been captured.

**Threshold tradeoff:**

| Higher $z_{\text{entry}}$ | Lower $z_{\text{entry}}$ |
|---------------------------|--------------------------|
| Fewer trades | More trades |
| Higher win rate per trade | Lower win rate per trade |
| Larger expected profit per trade | Smaller expected profit per trade |
| Risk of missing opportunities | Risk of trading noise |

Backtested defaults that balance these tradeoffs: $z_{\text{entry}} = 2.0$, $z_{\text{exit}} = 0.5$.

### Step 3: Position Sizing

**Equal dollar allocation.** Allocate $D$ dollars to each leg of the trade:

$$
q_Y = \frac{D}{Y_t}, \quad q_X = \frac{\beta\,D}{X_t}
$$

When entering a long spread position: buy $q_Y$ units of $Y$, sell $q_X$ units of $X$. The dollar exposure on each side is approximately $D$, making the position approximately dollar-neutral.

**Spread P&L.** If we enter at time $t_0$ and exit at time $t_1$:

$$
\text{P\&L} = q_Y(Y_{t_1} - Y_{t_0}) - q_X(X_{t_1} - X_{t_0})
$$

$$
= D\left(\frac{Y_{t_1} - Y_{t_0}}{Y_{t_0}} - \beta\,\frac{X_{t_1} - X_{t_0}}{X_{t_0}}\right)
$$

$$
\approx D\,\frac{s_{t_1} - s_{t_0}}{Y_{t_0}} \quad\text{(for small price changes)}
$$

**Graduated entry (optional).** Instead of all-or-nothing at $z_{\text{entry}}$, scale position size proportionally to the excess deviation:

$$
q_{\text{scale}} = \min\!\left(\frac{|z_t| - z_{\text{entry}}}{z_{\text{max}} - z_{\text{entry}}},\; 1\right)
$$

where $z_{\text{max}}$ is the z-score at which we reach full position size. This avoids committing full capital at the first threshold breach and allows averaging into the position if the spread continues to diverge.

### Step 4: Stop-Loss Design

Stop-losses protect against the spread *not* reverting — a sign of regime change or structural break in the cointegration relationship.

**Z-score stop.** If the z-score exceeds $z_{\text{stop}}$ (e.g., $4.0$), the spread has blown out far beyond historical norms:

$$
\text{If } |z_t| > z_{\text{stop}}: \quad\text{close all positions immediately}
$$

Under a stationary $\mathcal{N}(0,1)$ distribution, $P(|z| > 4) = 0.006\%$. If we observe this, it is strong evidence that the spread is no longer stationary — the cointegration relationship has likely broken down.

**Time-based stop.** If the position has been open for more than $T_{\text{max}}$ periods without converging:

$$
\text{If } t - t_{\text{entry}} > T_{\text{max}}: \quad\text{close the position}
$$

A reasonable default is $T_{\text{max}} = 3 \times t_{1/2}$ (three half-lives). Under the OU model, the probability of not having reverted by $3\,t_{1/2}$ is $(1/2)^3 = 12.5\%$ in expectation — if we're still in the trade, something may be wrong.

**Maximum loss stop.** Set a hard dollar loss limit:

$$
\text{If unrealised P\&L} < -\text{max\_loss}: \quad\text{close the position}
$$

This is a safety net independent of the model. Even if the z-score hasn't triggered a stop, a large dollar loss should force an exit to preserve capital.

**Stop-loss summary:**

| Stop type | Trigger | Typical value | Rationale |
|-----------|---------|---------------|-----------|
| Z-score blowout | $\|z_t\| > z_{\text{stop}}$ | $z_{\text{stop}} = 4.0$ | Spread regime break |
| Time-based | $t - t_{\text{entry}} > T_{\text{max}}$ | $T_{\text{max}} = 3\,t_{1/2}$ | Mean-reversion too slow |
| Maximum loss | P&L $< -$max\_loss | 2–5% of capital | Capital preservation |

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Lookback window | $L$ | Number of observations for rolling stats | 20–100 ticks |
| Entry threshold | $z_{\text{entry}}$ | Z-score level to open a trade | 1.5–2.5 |
| Exit threshold | $z_{\text{exit}}$ | Z-score level to close a trade | 0.0–0.75 |
| Stop threshold | $z_{\text{stop}}$ | Z-score level for emergency exit | 3.5–5.0 |
| Dollar allocation | $D$ | Capital per leg of the trade | Strategy dependent |
| Max holding period | $T_{\text{max}}$ | Time-based stop | $2$–$5 \times t_{1/2}$ |
| EMA decay factor | $\lambda$ | Weight on most recent observation | $2/(L+1)$ |

## Numerical Example

**Setup:** Spread $s_t$ over the last $L = 5$ observations (for illustration; use $L \geq 20$ in practice):

| $t$ | $s_t$ |
|-----|--------|
| 96 | 1.2 |
| 97 | 0.8 |
| 98 | 1.5 |
| 99 | 0.9 |
| 100 | -0.6 |

**Step 1: Rolling mean.**

$$
\hat{\mu}_{100} = \frac{1.2 + 0.8 + 1.5 + 0.9 + (-0.6)}{5} = \frac{3.8}{5} = 0.76
$$

**Step 2: Rolling standard deviation.**

$$
\sum(s_{t-i} - \hat{\mu})^2 = (1.2 - 0.76)^2 + (0.8 - 0.76)^2 + (1.5 - 0.76)^2 + (0.9 - 0.76)^2 + (-0.6 - 0.76)^2
$$

$$
= 0.1936 + 0.0016 + 0.5476 + 0.0196 + 1.8496 = 2.6120
$$

$$
\hat{\sigma}_{100} = \sqrt{\frac{2.6120}{4}} = \sqrt{0.6530} = 0.8081
$$

**Step 3: Z-score.**

$$
z_{100} = \frac{-0.6 - 0.76}{0.8081} = \frac{-1.36}{0.8081} = -1.683
$$

**Decision:** $|z_{100}| = 1.683 < z_{\text{entry}} = 2.0$. No trade signal — the spread is below its mean but not yet extreme enough to trigger entry.

**What if $s_{101} = -1.5$?** Updating the rolling window to $(0.8, 1.5, 0.9, -0.6, -1.5)$:

$$
\hat{\mu}_{101} = \frac{0.8 + 1.5 + 0.9 + (-0.6) + (-1.5)}{5} = \frac{1.1}{5} = 0.22
$$

$$
\hat{\sigma}_{101} = \sqrt{\frac{(0.58)^2 + (1.28)^2 + (0.68)^2 + (-0.82)^2 + (-1.72)^2}{4}} = \sqrt{\frac{0.3364 + 1.6384 + 0.4624 + 0.6724 + 2.9584}{4}} = \sqrt{\frac{6.0680}{4}} = \sqrt{1.5170} = 1.2317
$$

$$
z_{101} = \frac{-1.5 - 0.22}{1.2317} = \frac{-1.72}{1.2317} = -1.396
$$

Still below the entry threshold. The lookback window has adapted: the mean has dropped (incorporating the recent downturn), which moderates the z-score. This illustrates why a longer lookback window $L$ produces more stable signals — short windows track recent movements too closely.

```python
import numpy as np

spread = np.array([1.2, 0.8, 1.5, 0.9, -0.6])
L = len(spread)
mu_hat = spread.mean()
sigma_hat = spread.std(ddof=1)
z = (spread[-1] - mu_hat) / sigma_hat

print(f"Rolling mean:  {mu_hat:.4f}")
print(f"Rolling std:   {sigma_hat:.4f}")
print(f"Z-score:       {z:.4f}")
print(f"Entry signal:  {'YES (long spread)' if z < -2.0 else 'YES (short spread)' if z > 2.0 else 'NO'}")
```

## Connections

- [Cointegration](./cointegration.md): the spread $s_t = Y_t - \beta X_t$ that feeds into the z-score is constructed from the cointegration analysis. The hedge ratio $\beta$ determines the spread; a poorly estimated $\beta$ corrupts the z-score signal.
- [Ornstein-Uhlenbeck](./ornstein_uhlenbeck.md): the OU model provides theoretical justification for z-score trading. The stationary distribution $\mathcal{N}(\mu, \sigma^2/(2\theta))$ defines the "normal" range of the spread; z-scores measure deviations in units of this stationary distribution. The half-life $\ln 2 / \theta$ determines how long trades take to converge, informing the time-based stop.
- [Inventory Risk](../03_market_making/inventory_risk.md): in market making, z-score-like signals can be used to skew quotes — when the spread z-score is extreme, a market maker might widen their spread or lean their quotes in the direction of expected reversion.
- [Strategy Integration](../06_prosperity/strategy_integration.md): the z-score signal framework is the final output of the stat-arb pipeline and feeds directly into the trading algorithm's order generation logic.

## Relevance for Prosperity 4

In Prosperity 4, the z-score framework is the operational core of any pairs-trading strategy. After identifying cointegrated product pairs and estimating OU parameters, the z-score converts the abstract spread into actionable trading signals. The key implementation decisions for Prosperity are: (1) **lookback window** — shorter windows ($L = 20$–$30$ ticks) are better for fast-moving Prosperity markets than the longer windows used in equity stat-arb; (2) **entry thresholds** — Prosperity's transaction costs are typically zero or very low, so lower entry thresholds ($z_{\text{entry}} = 1.5$) may be profitable where they would not be in real markets; (3) **position limits** — Prosperity enforces hard position limits, so the position sizing formula must respect these constraints; (4) **stop-losses** — because bot behaviors can change between rounds, regime breaks are a real risk, and z-score blowout stops ($z_{\text{stop}} = 3.5$–$4.0$) are essential to avoid large losses when a previously cointegrated pair decouples. The entire pipeline (spread construction $\to$ z-score $\to$ entry/exit/stop) can be implemented in under 50 lines of Python and runs in constant time per tick, well within Prosperity's latency budget.

## Python Snippet

```python
import numpy as np

def rolling_zscore(spread, lookback):
    """Compute rolling z-score of a spread series."""
    n = len(spread)
    z = np.full(n, np.nan)
    for t in range(lookback, n):
        window = spread[t - lookback + 1 : t + 1]
        mu = window.mean()
        sigma = window.std(ddof=1)
        if sigma > 1e-10:
            z[t] = (spread[t] - mu) / sigma
    return z

def generate_signals(z, z_entry=2.0, z_exit=0.5, z_stop=4.0):
    """Generate trading signals from z-scores."""
    n = len(z)
    position = 0  # +1 = long spread, -1 = short spread, 0 = flat
    signals = np.zeros(n)
    for t in range(n):
        if np.isnan(z[t]):
            continue
        if position == 0:
            if z[t] < -z_entry:
                position = 1
            elif z[t] > z_entry:
                position = -1
        elif position == 1:
            if z[t] > -z_exit or z[t] < -z_stop:
                position = 0
        elif position == -1:
            if z[t] < z_exit or z[t] > z_stop:
                position = 0
        signals[t] = position
    return signals

rng = np.random.default_rng(42)
theta, mu, sigma = 5.0, 0.0, 1.0
n_obs = 500
dt = 1.0
spread = np.zeros(n_obs)
for i in range(1, n_obs):
    spread[i] = spread[i-1] + theta * (mu - spread[i-1]) * dt + sigma * rng.normal(0, np.sqrt(dt))

z = rolling_zscore(spread, lookback=30)
signals = generate_signals(z)
n_trades = np.sum(np.abs(np.diff(signals)) > 0)
print(f"Z-score range: [{np.nanmin(z):.2f}, {np.nanmax(z):.2f}]")
print(f"Number of position changes: {n_trades}")
```
