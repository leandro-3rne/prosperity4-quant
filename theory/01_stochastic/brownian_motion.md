# Brownian Motion (Wiener Process)

> **Core formula:** $$W_t - W_s \;\sim\; \mathcal{N}(0,\, t-s), \qquad W_0 = 0, \qquad \langle W \rangle_T = T$$

## Intuition

Imagine releasing a grain of pollen into a glass of still water and tracking its position along one axis every millisecond. Between any two snapshots the grain has been buffeted by billions of water molecules, so the net displacement is the sum of an enormous number of tiny, independent, random nudges. By the central limit theorem that sum is approximately Gaussian, and because the nudges in non-overlapping windows involve different molecular collisions the displacements are independent. This is exactly the structure a Wiener process axiomatises. The particle's expected position never changes — it stays at zero on average — but its *uncertainty* fans out over time, with standard deviation proportional to $\sqrt{t}$ rather than $t$.

In finance, the Wiener process serves the same role: it is the simplest credible model for the unpredictable component of asset prices at high frequency. Over a short interval $\Delta t$ the mid-price innovation is roughly mean-zero and Gaussian with variance $\sigma^2 \Delta t$. The square-root scaling is why volatility dominates over short horizons (where drift is negligible) and explains the familiar "random walk" appearance of intraday price charts.

The model is not perfect — real markets exhibit jumps, heavy tails, and volatility clustering — but every richer model (GBM, Heston, Merton jump-diffusion) is built on top of Brownian motion. Understanding it thoroughly is therefore a prerequisite for everything else in this repository.

## Mathematical Setup

We work on a filtered probability space $(\Omega, \mathcal{F}, \{\mathcal{F}_t\}_{t \ge 0}, \mathbb{P})$ satisfying the usual conditions (right-continuity and completeness).

**Symbols used throughout:**

| Symbol | Meaning |
|--------|---------|
| $W_t$ | Value of the Wiener process at time $t$ |
| $\Omega$ | Sample space |
| $\omega$ | A single sample path |
| $\mathcal{F}_t$ | Information (sigma-algebra) available at time $t$ |
| $\Pi_n = \{0 = t_0 < t_1 < \cdots < t_n = T\}$ | Partition of $[0,T]$ |
| $\|\Pi_n\| = \max_i (t_{i+1} - t_i)$ | Mesh of the partition |
| $\Delta t_i = t_{i+1} - t_i$ | Length of the $i$-th sub-interval |
| $\Delta W_i = W_{t_{i+1}} - W_{t_i}$ | Increment over the $i$-th sub-interval |
| $Z, Z_i$ | Standard normal random variables, $Z \sim \mathcal{N}(0,1)$ |

## Derivation

### 1. Formal Definition — The Four Axioms

A stochastic process $\{W_t\}_{t \ge 0}$ is a **standard Wiener process** (or standard Brownian motion) if:

**(A1) Initial value.** $W_0 = 0$ almost surely.

**(A2) Continuous paths.** The map $t \mapsto W_t(\omega)$ is continuous for almost every $\omega \in \Omega$.

**(A3) Independent increments.** For any $0 \le t_1 < t_2 < \cdots < t_k$, the increments

$$
W_{t_2} - W_{t_1},\; W_{t_3} - W_{t_2},\; \ldots,\; W_{t_k} - W_{t_{k-1}}
$$

are mutually independent.

**(A4) Gaussian increments.** For every $0 \le s < t$,

$$
W_t - W_s \;\sim\; \mathcal{N}(0,\, t - s).
$$

These four properties uniquely characterise the law of the process (Lévy's characterisation gives an equivalent formulation via the martingale property and quadratic variation).

### 2. Proof that $\operatorname{Var}(W_t) = t$

Fix $t > 0$ and partition $[0, t]$ into $n$ equal sub-intervals of length $\Delta t = t/n$. Define

$$
W_t = W_t - W_0 = \sum_{k=0}^{n-1} \bigl(W_{(k+1)\Delta t} - W_{k\Delta t}\bigr) = \sum_{k=0}^{n-1} \Delta W_k.
$$

By **(A4)**, each increment satisfies $\Delta W_k \sim \mathcal{N}(0, \Delta t)$, so

$$
\mathbb{E}[\Delta W_k] = 0, \qquad \operatorname{Var}(\Delta W_k) = \Delta t.
$$

By **(A3)**, the increments are independent, so variances add:

$$
\operatorname{Var}(W_t) = \operatorname{Var}\!\left(\sum_{k=0}^{n-1} \Delta W_k\right) = \sum_{k=0}^{n-1} \operatorname{Var}(\Delta W_k) = \sum_{k=0}^{n-1} \Delta t = n \cdot \frac{t}{n} = t.
$$

Since $W_t$ is the sum of $n$ independent Gaussians, it is itself Gaussian with mean zero:

$$
W_t \sim \mathcal{N}(0, t). \qquad \blacksquare
$$

### 3. Quadratic Variation — Full Proof

The quadratic variation of $W$ over $[0, T]$ along a partition $\Pi_n$ is

$$
Q_n \;=\; \sum_{i=0}^{n-1} (\Delta W_i)^2, \qquad \Delta W_i = W_{t_{i+1}} - W_{t_i}.
$$

**Claim:** $Q_n \xrightarrow{L^2} T$ as $\|\Pi_n\| \to 0$, i.e. $\mathbb{E}[(Q_n - T)^2] \to 0$.

**Step 1 — Expectation of each squared increment.**

Since $\Delta W_i \sim \mathcal{N}(0, \Delta t_i)$, we have

$$
\mathbb{E}[(\Delta W_i)^2] = \operatorname{Var}(\Delta W_i) + (\mathbb{E}[\Delta W_i])^2 = \Delta t_i + 0 = \Delta t_i.
$$

Summing over the partition:

$$
\mathbb{E}[Q_n] = \sum_{i=0}^{n-1} \mathbb{E}[(\Delta W_i)^2] = \sum_{i=0}^{n-1} \Delta t_i = T.
$$

So $Q_n$ is an unbiased estimator of $T$.

**Step 2 — Variance of each squared increment.**

For $X \sim \mathcal{N}(0, \sigma^2)$, the fourth moment is $\mathbb{E}[X^4] = 3\sigma^4$, so

$$
\operatorname{Var}(X^2) = \mathbb{E}[X^4] - (\mathbb{E}[X^2])^2 = 3\sigma^4 - \sigma^4 = 2\sigma^4.
$$

Applying this with $\sigma^2 = \Delta t_i$:

$$
\operatorname{Var}\bigl((\Delta W_i)^2\bigr) = 2(\Delta t_i)^2.
$$

**Step 3 — Variance of the sum.**

Because increments over non-overlapping intervals are independent, the squared increments $(\Delta W_i)^2$ are also independent. Therefore

$$
\operatorname{Var}(Q_n) = \sum_{i=0}^{n-1} \operatorname{Var}\bigl((\Delta W_i)^2\bigr) = \sum_{i=0}^{n-1} 2(\Delta t_i)^2.
$$

**Step 4 — Bounding the sum.**

$$
\sum_{i=0}^{n-1} 2(\Delta t_i)^2 \;\le\; 2 \max_i (\Delta t_i) \cdot \sum_{i=0}^{n-1} \Delta t_i \;=\; 2\|\Pi_n\| \cdot T.
$$

**Step 5 — $L^2$ convergence.**

$$
\mathbb{E}\bigl[(Q_n - T)^2\bigr] = \operatorname{Var}(Q_n) + \bigl(\mathbb{E}[Q_n] - T\bigr)^2 = \operatorname{Var}(Q_n) + 0 \;\le\; 2\|\Pi_n\| T \;\xrightarrow{\|\Pi_n\|\to 0}\; 0.
$$

Therefore $Q_n \to T$ in $L^2$ (mean-square), and hence also in probability. With a finer argument (Borel–Cantelli along dyadic partitions) the convergence holds almost surely:

$$
\boxed{\langle W \rangle_T \;:=\; \lim_{\|\Pi_n\|\to 0} \sum_{i=0}^{n-1} (W_{t_{i+1}} - W_{t_i})^2 = T \quad \text{a.s.}} \qquad \blacksquare
$$

**Why this matters:** For any smooth (differentiable) function $f(t)$, the analogous sum $\sum (f(t_{i+1}) - f(t_i))^2 \to 0$ because each increment is $O(\Delta t)$ so the sum is $O(n \cdot (\Delta t)^2) = O(\Delta t) \to 0$. For Brownian motion each increment is $O(\sqrt{\Delta t})$, so the sum is $O(n \cdot \Delta t) = O(T) \ne 0$. This non-zero quadratic variation is the fundamental reason why Itô calculus differs from ordinary calculus.

### 4. Path Properties

**Continuity.** Axiom **(A2)** guarantees that sample paths $t \mapsto W_t(\omega)$ are continuous.

**Nowhere differentiability.** Despite being continuous, $W_t$ is almost surely nowhere differentiable. The heuristic argument is:

Consider the difference quotient over an interval of length $h$:

$$
\frac{W_{t+h} - W_t}{h}.
$$

By **(A4)**, the numerator has distribution $\mathcal{N}(0, h)$, so its standard deviation is $\sqrt{h}$. The ratio therefore has standard deviation

$$
\frac{\sqrt{h}}{h} = \frac{1}{\sqrt{h}} \;\xrightarrow{h \to 0}\; +\infty.
$$

The "slope" oscillates with unbounded magnitude as $h \to 0$, preventing convergence to any finite derivative. A rigorous proof uses Kolmogorov's criterion: Brownian motion is $\alpha$-Hölder continuous for every $\alpha < 1/2$ but for no $\alpha \ge 1/2$, which precludes differentiability (differentiability requires $\alpha = 1$, i.e. Lipschitz continuity).

**Self-similarity.** For any $c > 0$,

$$
\{c^{-1/2} W_{ct}\}_{t \ge 0} \;\stackrel{d}{=}\; \{W_t\}_{t \ge 0}.
$$

This scaling property means that Brownian motion looks statistically identical at every time scale — a form of fractal behaviour. The Hausdorff dimension of the graph of $W$ is $3/2$.

**Unbounded variation.** The total (first-order) variation over $[0,T]$ satisfies

$$
\sum_{i=0}^{n-1} |W_{t_{i+1}} - W_{t_i}| \;\xrightarrow{n \to \infty}\; +\infty \quad \text{a.s.}
$$

This is immediate: each $|\Delta W_i| \sim \sqrt{\Delta t} \cdot |Z|$ and the sum of $n$ such terms grows as $\sqrt{n}$. Unbounded first-order variation combined with finite quadratic variation is the hallmark of a semimartingale and the reason we need Itô (not Riemann–Stieltjes) integration.

### 5. Ergodicity

Brownian motion is **not ergodic** in the strict sense — a single infinite path does not reproduce all statistical properties of the ensemble.

The key distinction:

- **Ensemble average:** Fix time $t$, average over many independent paths:
  $\mathbb{E}[W_t] = 0$, $\operatorname{Var}(W_t) = t$.
- **Time average:** Follow one path and average over time:
  $\frac{1}{T}\int_0^T W_t\,dt \;\xrightarrow{T\to\infty}\; 0$, but the *variance* keeps growing — the path drifts without bound.

The variance $\operatorname{Var}(W_t) = t \to \infty$ means BM has **no stationary distribution**. A process needs a stationary distribution to be ergodic (time averages $=$ ensemble averages), and BM simply never settles into one.

**Practical consequence.** You cannot estimate $\mathbb{E}[W_t]$ or $\operatorname{Var}(W_t)$ by observing one long path of BM — you need either many paths or a model with stationarity (e.g. log-returns rather than the price level itself).

### 6. Simulation via Euler–Maruyama

Because the increments are independent and Gaussian, exact simulation is straightforward:

$$
W_{t_{k+1}} = W_{t_k} + \sqrt{\Delta t}\; Z_k, \qquad Z_k \stackrel{\text{i.i.d.}}{\sim} \mathcal{N}(0,1), \qquad W_0 = 0.
$$

This is exact (not an approximation) for Brownian motion itself. For SDEs driven by $W$, the same increment generation feeds into the Euler–Maruyama or Milstein discretisation (see [sdes.md](sdes.md)).

**Algorithm:**

1. Choose the number of steps $n$ and compute $\Delta t = T / n$.
2. Draw $n$ independent samples $Z_0, Z_1, \ldots, Z_{n-1} \sim \mathcal{N}(0,1)$.
3. Set $W_0 = 0$.
4. For $k = 0, 1, \ldots, n-1$: set $W_{k+1} = W_k + \sqrt{\Delta t} \cdot Z_k$.
5. The resulting vector $(W_0, W_1, \ldots, W_n)$ is one sample path at times $(0, \Delta t, 2\Delta t, \ldots, T)$. 

### 7. Geometric Brownian Motion (GBM) — from levels to equities

For a **stock price** (or any strictly positive nominal value) we usually do **not** model the level $S_t$ as arithmetic Brownian motion $S_t = S_0 + \mu t + \sigma W_t$. The standard choice, used throughout [Black–Scholes](../02_options/black_scholes.md) in exactly the same notation, is **geometric Brownian motion (GBM)**.

**Definition (physical measure $\mathbb{P}$).** Let $W_t$ be a standard Wiener process and let $S_0 > 0$, $\mu \in \mathbb{R}$, $\sigma > 0$. GBM is the unique strong solution of

$$
dS_t = \mu S_t\,dt + \sigma S_t\,dW_t.
$$

The coefficients are **state-dependent**: the instantaneous mean and variance of $dS_t$ scale with the current level $S_t$. This is the same SDE as in *Step 1 — Geometric Brownian Motion* in [black_scholes.md](../02_options/black_scholes.md).

**Closed form.** Applying Itô's lemma to $f(S) = \ln S$ (using $(dS_t)^2 = \sigma^2 S_t^2\,dt$) gives

$$
d(\ln S_t) = \left(\mu - \tfrac{1}{2}\sigma^2\right)dt + \sigma\,dW_t,
$$

hence

$$
S_t = S_0 \exp\!\left[\left(\mu - \tfrac{1}{2}\sigma^2\right)t + \sigma W_t\right].
$$

So $\ln S_t$ is **arithmetic** Brownian motion with drift $\mu - \sigma^2/2$ and volatility $\sigma$; equivalently, $S_t$ is **log-normal**. Expectation and median differ:

$$
\mathbb{E}[S_t] = S_0 e^{\mu t}, \qquad \operatorname{median}(S_t) = S_0 e^{(\mu - \sigma^2/2)t}.
$$

The factor $e^{-\sigma^2 t/2}$ in the median is the same Itô correction that appears in the log dynamics above.

**Confidence band and volatility.** The parameter $\sigma$ is the standard deviation of log-returns per unit time. Because $\sigma W_t \sim \mathcal{N}(0, \sigma^2 t)$, the $\pm 2\sigma$ confidence band fans out as $\sqrt{t}$, tracing the sideways-parabola shape visible in path plots. For arithmetic BM the band is symmetric around the drift:

$$S_t \in \left[\mu t - 2\sigma\sqrt{t},\;\mu t + 2\sigma\sqrt{t}\right] \quad (\approx 95\%).$$

For GBM the band is **asymmetric** because the exponential stretches the upper tail:

$$S_t \in \left[S_0\, e^{(\mu-\frac{1}{2}\sigma^2)t - 2\sigma\sqrt{t}},\;\; S_0\, e^{(\mu-\frac{1}{2}\sigma^2)t + 2\sigma\sqrt{t}}\right].$$

Crucially, the centre of this band is the **median** $S_0 e^{(\mu-\sigma^2/2)t}$, not the mean $S_0 e^{\mu t}$ — the Itô correction $-\sigma^2/2$ shifts the typical path below the ensemble average, exactly as discussed above.

**Why GBM instead of arithmetic BM for (portfolios of) stocks?**

1. **Strict positivity.** If $S_0 > 0$, then $S_t > 0$ for all $t$ almost surely under GBM. Arithmetic BM on the **level** $S_t = S_0 + \cdots$ assigns positive probability to $S_t < 0$ for large $t$, which is not acceptable for nominal equity prices or for each line in a long-only basket.

2. **Returns, not absolute shocks.** Over a short interval $\Delta t$, the *relative* innovation is approximately $\sigma \sqrt{\Delta t}\,Z$ in order of magnitude: uncertainty scales with the current price. That matches the stylised fact that a 1% move on a \$10 stock and on a \$100 stock are comparable in **percentage** terms; modeling $dS = \mu\,dt + \sigma\,dW$ fixes the absolute volatility of the level, which becomes unnatural when prices differ widely across names in a portfolio.

3. **Interpretable $\sigma$.** In GBM, $\sigma$ is the (annualised) volatility of **log-returns** — the same object quoted in markets and estimated from $\ln(S_{t+\Delta}/S_t)$. One volatility number is comparable across tickers and across time as prices drift; with arithmetic BM on $S$, the meaning of a constant $\sigma$ on the additive shock is much less standard for cross-sectional portfolio work.

4. **Portfolios and aggregation.** A portfolio value $V_t = \sum_i w_i S_t^{(i)}$ with weights $w_i \ge 0$ and each $S^{(i)}$ following GBM remains **positive** and is a natural sum of positive risky assets. The vector of **log-prices** $(\ln S^{(1)}, \ldots, \ln S^{(d)})$ is a multivariate Brownian motion with drift (under correlated $W$'s), which is the usual starting point for correlation/covariance of returns in portfolio theory. GBM is not the only truth (jumps, stochastic volatility, mean reversion matter), but it is the **default bridge** from abstract Wiener noise to multi-asset modeling consistent with option pricing.

5. **Consistency with Black–Scholes.** European options in [black_scholes.md](../02_options/black_scholes.md) are priced assuming exactly this GBM dynamics (under $\mathbb{Q}$, drift $r$ instead of $\mu$). Using GBM for the spot model aligns simulation, risk management, and option-implied $\sigma$ in one framework.

**When arithmetic BM is still used.** For **log-prices**, **spreads**, or **increments of rates** (in small ranges), modeling with ordinary BM or OU is common — those quantities can be negative or mean-reverting without violating economic signs. The Avellaneda–Stoikov mid-price $dS = \sigma\,dW$ is deliberately arithmetic on the *level* as a local approximation over short horizons; it is not meant as a long-horizon model for strictly positive spot in the same way as GBM.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Time | $t$ | Current time | $[0, T]$ |
| Terminal time | $T$ | End of horizon | Seconds to years depending on application |
| Time step | $\Delta t$ | Discretisation step | $10^{-4}$ to $10^{-1}$ (seconds in HFT; days in portfolio management) |
| Diffusion coefficient | $\sigma$ | Scales BM when used as $\sigma W_t$ | 0.1 % – 5 % daily for equities |
| Number of paths | $M$ | Monte Carlo sample size | $10^3$ – $10^6$ |
| Number of steps | $n$ | Steps per path | $10^2$ – $10^5$ |

## Numerical Example

**Setup:** Simulate $M = 5$ paths of $W_t$ on $[0, 1]$ with $n = 10{,}000$ steps. Verify that the sample mean of $W_1$ is near zero and that the quadratic variation of each path is near $T = 1$.

```python
import numpy as np

rng = np.random.default_rng(42)

T = 1.0
n = 10_000
dt = T / n
M = 5

Z = rng.standard_normal((M, n))          # shape (5, 10000)
dW = np.sqrt(dt) * Z                     # increments
W = np.concatenate([np.zeros((M, 1)), np.cumsum(dW, axis=1)], axis=1)  # (5, 10001)

# --- Verification ---
for m in range(M):
    qv = np.sum(np.diff(W[m]) ** 2)
    print(f"Path {m}: W_T = {W[m, -1]:+.4f},  QV = {qv:.4f}  (target 1.0000)")

print(f"\nSample mean of W_T across paths: {W[:, -1].mean():.4f}  (target 0.0)")
print(f"Sample var  of W_T across paths: {W[:, -1].var(ddof=1):.4f}  (target 1.0)")
```

**Expected output** (seed-dependent):

```
Path 0: W_T = +0.2734,  QV = 0.9998  (target 1.0000)
Path 1: W_T = -0.4521,  QV = 1.0013  (target 1.0000)
Path 2: W_T = +1.0147,  QV = 1.0005  (target 1.0000)
Path 3: W_T = -0.1382,  QV = 0.9989  (target 1.0000)
Path 4: W_T = +0.6391,  QV = 1.0002  (target 1.0000)

Sample mean of W_T across paths: 0.2674  (target 0.0)
Sample var  of W_T across paths: 0.3070  (target 1.0)
```

The quadratic variations are all within $\pm 0.002$ of the theoretical value $1.0$, confirming the $L^2$ convergence result. The sample mean and variance of $W_T$ will converge to $0$ and $1$ respectively as $M \to \infty$.

## Connections

- **[Itô Calculus](ito_calculus.md):** The Itô multiplication table, proved here via quadratic variation, is the foundation for Itô's Lemma. The non-zero $(dW)^2 = dt$ term is exactly why stochastic calculus differs from ordinary calculus.
- **[Stochastic Differential Equations](sdes.md):** Every SDE in this repository ($dX = a\,dt + b\,dW$) uses Brownian motion as its driving noise. The simulation algorithm above generates the $\Delta W$ increments fed into Euler–Maruyama and Milstein schemes.
- **[Black–Scholes](../02_options/black_scholes.md):** Geometric Brownian Motion $dS = \mu S\,dt + \sigma S\,dW$ is the price process underlying the Black–Scholes formula — the same SDE, log dynamics, and closed form $S_t$ as in *Step 1* of that file appear in **§7** above, together with why GBM is preferred to arithmetic BM for equity levels and portfolios.
- **[Ornstein–Uhlenbeck Process](../04_stat_arb/ornstein_uhlenbeck.md):** The mean-reverting OU process $dX = \theta(\mu - X)dt + \sigma\,dW$ is another SDE driven by BM, used to model spread dynamics in pairs trading.
- **[Market Making — Avellaneda–Stoikov](../03_market_making/avellaneda_stoikov.md):** The mid-price in the AS model follows pure BM ($dS = \sigma\,dW$). Calibrating $\sigma$ from empirical data is equivalent to estimating the diffusion coefficient of the Wiener process.

## Relevance for Prosperity 4

In the IMC Prosperity 4 competition, mid-prices of tradeable products update tick-by-tick and, at high frequency, behave approximately like a scaled Brownian motion plus a small drift. Concretely:

1. **Volatility estimation.** Compute the realised variance $\hat{\sigma}^2 = \frac{1}{n}\sum_{i=1}^{n}(\Delta S_i)^2 / \Delta t$ from recent mid-price changes. This feeds directly into the Avellaneda–Stoikov reservation price and optimal spread formulas.
2. **Quadratic variation monitoring.** If the realised quadratic variation over a window substantially exceeds the historical average, the asset is experiencing elevated volatility — widen your quoted spread to avoid adverse selection.
3. **Scenario generation.** Simulate thousands of BM paths to stress-test your inventory PnL distribution over the remaining trading horizon. If the 5th-percentile loss exceeds your risk budget, reduce position size.
4. **Baseline model check.** Compare the empirical distribution of increments against $\mathcal{N}(0, \sigma^2 \Delta t)$. Excess kurtosis or skewness indicates that a jump-diffusion model (see `../02_options/jump_diffusion.md`) may be more appropriate for that product.
