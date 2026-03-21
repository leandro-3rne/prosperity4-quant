# Itô Calculus

> **Core formula:** $$df = \left(\frac{\partial f}{\partial t} + \mu\frac{\partial f}{\partial x} + \frac{1}{2}\sigma^2\frac{\partial^2 f}{\partial x^2}\right)dt + \sigma\frac{\partial f}{\partial x}\,dW_t$$

## Intuition

In ordinary calculus, when you know how a variable $x$ changes and you want to know how $f(x)$ changes, you use the chain rule: $df = f'(x)\,dx$. This works perfectly when $x$ is a smooth function of time, because the second-order term $\tfrac{1}{2}f''(x)(dx)^2$ involves $(dx)^2$, which is infinitesimally small compared to $dx$ and vanishes. But Brownian motion is not smooth — its paths are so jagged that the squared increment $(dW)^2$ does *not* vanish. Instead, as shown in the [quadratic variation proof](brownian_motion.md), $(dW_t)^2 = dt$, which is first-order and must be retained. Itô's Lemma is simply the chain rule corrected for this extra term.

The practical upshot is a systematic "correction factor" that appears whenever you transform a stochastic process. The most famous instance is the drift adjustment $-\sigma^2/2$ that emerges when you take the logarithm of Geometric Brownian Motion — this is not an approximation or a convention but an inevitable consequence of the roughness of Brownian paths. Itô calculus provides the rigorous framework for deriving such corrections and for manipulating stochastic differential equations in general.

For a trader, Itô calculus is the bridge between a model for price dynamics ($dS = \mu S\,dt + \sigma S\,dW$) and the quantities needed for hedging and pricing: the dynamics of a portfolio, the Black–Scholes PDE, Greeks, and risk-neutral valuation. Every formula in this repository that involves transforming or combining stochastic processes relies on Itô's Lemma.

## Mathematical Setup

**Probability space.** We work on $(\Omega, \mathcal{F}, \{\mathcal{F}_t\}_{t \ge 0}, \mathbb{P})$ carrying a standard Wiener process $W_t$.

**Symbols:**

| Symbol | Meaning |
|--------|---------|
| $W_t$ | Standard Wiener process (Brownian motion) |
| $X_t$ | An Itô process satisfying $dX_t = \mu(t, X_t)\,dt + \sigma(t, X_t)\,dW_t$ |
| $f(t, x)$ | A function that is $C^1$ in $t$ and $C^2$ in $x$ |
| $f_t, f_x, f_{xx}$ | Shorthand for $\partial f / \partial t$, $\partial f / \partial x$, $\partial^2 f / \partial x^2$ |
| $\mu(t,x)$ | Drift coefficient of the SDE |
| $\sigma(t,x)$ | Diffusion coefficient of the SDE |
| $\langle W \rangle_t = t$ | Quadratic variation of BM |

**Standing assumptions:** All functions and processes satisfy sufficient regularity (measurability, integrability, growth bounds) for the integrals below to be well-defined.

## Derivation

### 1. Why the Classical Chain Rule Fails for Brownian Motion

Consider a twice-differentiable function $f(x)$ and a smooth deterministic path $x(t)$. Taylor's theorem gives:

$$
f(x + \Delta x) = f(x) + f'(x)\,\Delta x + \frac{1}{2}f''(x)\,(\Delta x)^2 + O(|\Delta x|^3).
$$

For a smooth function, $\Delta x = O(\Delta t)$, so $(\Delta x)^2 = O((\Delta t)^2)$, which is negligible compared to $\Delta x = O(\Delta t)$. Only the first-order term survives:

$$
df = f'(x)\,dx. \qquad \text{(ordinary chain rule)}
$$

Now replace $x(t)$ with $W_t$. Over an interval of length $\Delta t$:

$$
\Delta W = W_{t+\Delta t} - W_t \sim \mathcal{N}(0, \Delta t),
$$

so $\Delta W = O(\sqrt{\Delta t})$ in the sense that $\mathbb{E}[(\Delta W)^2] = \Delta t$. The second-order term becomes:

$$
\frac{1}{2}f''(W_t)(\Delta W)^2 = O(\Delta t),
$$

which is the **same order** as the first-order term $f'(W_t)\Delta W = O(\sqrt{\Delta t})$. In fact, $(\Delta W)^2 \to \Delta t$ deterministically (quadratic variation), so this term contributes a non-random drift of $\frac{1}{2}f''(W_t)\,dt$ that cannot be discarded.

**The Itô Multiplication Table**

When manipulating differentials in stochastic calculus we keep terms up to order $dt$ and discard anything of higher order. The rules are:

$$
\begin{array}{c|cc}
\times & dt & dW_t \\
\hline
dt & 0 & 0 \\
dW_t & 0 & dt
\end{array}
$$

Written out:

$$
(dW_t)^2 = dt, \qquad dt \cdot dW_t = 0, \qquad (dt)^2 = 0.
$$

**Why $(dW_t)^2 = dt$.**
This is precisely the quadratic variation result proved above. Over an infinitesimal interval $[t, t+dt]$, we have $(\Delta W)^2 \approx \Delta t$ with variance $2(\Delta t)^2 \to 0$, so the squared increment is *deterministic* at leading order and equals $dt$.

**Why $dt \cdot dW_t = 0$.**
The product $dt \cdot dW_t$ is of order $\Delta t \cdot \sqrt{\Delta t} = (\Delta t)^{3/2}$, which vanishes faster than $\Delta t$ and is therefore negligible compared to both $dt$ and $(dW_t)^2 = dt$.

**Why $(dt)^2 = 0$.**
The product $dt \cdot dt$ is of order $(\Delta t)^2$, which is even smaller and clearly negligible.

These rules are the engine that drives Itô's Lemma: the second-order Taylor term $\tfrac{1}{2}f''(W_t)(dW_t)^2$ survives because $(dW_t)^2 = dt$ is first-order, not zero as it would be for a smooth driving function.

Higher-order terms: $(\Delta W)^3 = O((\Delta t)^{3/2})$ is negligible compared to $dt$, so we truncate after the second-order term. This is the key insight.

### 2. Itô's Lemma — Simple Form: $f(t, W_t)$

**Theorem.** Let $f(t, x)$ be $C^1$ in $t$ and $C^2$ in $x$. Then:

$$
df(t, W_t) = \frac{\partial f}{\partial t}\,dt + \frac{\partial f}{\partial x}\,dW_t + \frac{1}{2}\frac{\partial^2 f}{\partial x^2}\,dt.
$$

Or more compactly:

$$
df = f_t\,dt + f_x\,dW_t + \tfrac{1}{2}f_{xx}\,dt = \bigl(f_t + \tfrac{1}{2}f_{xx}\bigr)\,dt + f_x\,dW_t.
$$

**Proof sketch.** Write the second-order Taylor expansion:

$$
\Delta f = f_t \,\Delta t + f_x \,\Delta W + \frac{1}{2}f_{tt}\,(\Delta t)^2 + f_{tx}\,\Delta t\,\Delta W + \frac{1}{2}f_{xx}\,(\Delta W)^2.
$$

Apply the Itô multiplication table:

- $(\Delta t)^2 \to 0$ (higher order than $dt$)
- $\Delta t \cdot \Delta W \to 0$ (order $(\Delta t)^{3/2}$, negligible)
- $(\Delta W)^2 \to dt$ (quadratic variation)

Substituting:

$$
\Delta f \;\to\; f_t\,dt + f_x\,dW_t + \frac{1}{2}f_{xx}\,dt.
$$

Collecting the $dt$ terms:

$$
df = \bigl(f_t + \tfrac{1}{2}f_{xx}\bigr)dt + f_x\,dW_t. \qquad \blacksquare
$$

### 3. Itô's Lemma — General Form: $f(t, X_t)$

**Theorem.** Let $X_t$ be an Itô process satisfying

$$
dX_t = \mu(t, X_t)\,dt + \sigma(t, X_t)\,dW_t,
$$

and let $f(t, x)$ be $C^1$ in $t$ and $C^2$ in $x$. Then $Y_t = f(t, X_t)$ satisfies:

$$
dY_t = \left(\frac{\partial f}{\partial t} + \mu\frac{\partial f}{\partial x} + \frac{1}{2}\sigma^2 \frac{\partial^2 f}{\partial x^2}\right)dt + \sigma\frac{\partial f}{\partial x}\,dW_t.
$$

**Full proof sketch via Taylor expansion.**

**Step 1.** Write the second-order Taylor expansion of $f(t + \Delta t,\, X_t + \Delta X)$ around $(t, X_t)$:

$$
\Delta f = f_t\,\Delta t + f_x\,\Delta X + \frac{1}{2}f_{tt}\,(\Delta t)^2 + f_{tx}\,\Delta t\,\Delta X + \frac{1}{2}f_{xx}\,(\Delta X)^2.
$$

**Step 2.** Substitute $\Delta X = \mu\,\Delta t + \sigma\,\Delta W$:

$$
(\Delta X)^2 = (\mu\,\Delta t + \sigma\,\Delta W)^2 = \mu^2(\Delta t)^2 + 2\mu\sigma\,\Delta t\,\Delta W + \sigma^2(\Delta W)^2.
$$

**Step 3.** Apply the Itô multiplication table to $(\Delta X)^2$:

- $\mu^2(\Delta t)^2 \to 0$
- $2\mu\sigma\,\Delta t\,\Delta W \to 0$
- $\sigma^2(\Delta W)^2 \to \sigma^2\,dt$

So $(\Delta X)^2 \to \sigma^2\,dt$.

**Step 4.** The cross-term $f_{tx}\,\Delta t\,\Delta X = f_{tx}\,\Delta t\,(\mu\,\Delta t + \sigma\,\Delta W)$. Both pieces are $O((\Delta t)^2)$ or $O((\Delta t)^{3/2})$, hence negligible. Similarly $f_{tt}(\Delta t)^2 \to 0$.

**Step 5.** Collect surviving terms:

$$
\Delta f \;\to\; f_t\,dt + f_x\,(\mu\,dt + \sigma\,dW_t) + \frac{1}{2}f_{xx}\,\sigma^2\,dt.
$$

**Step 6.** Group the $dt$ and $dW_t$ terms:

$$
\boxed{df = \left(f_t + \mu f_x + \frac{1}{2}\sigma^2 f_{xx}\right)dt + \sigma f_x\,dW_t.} \qquad \blacksquare
$$

### 4. Worked Derivation: $d[\ln S_t]$ from GBM

This is the single most important application of Itô's Lemma in quantitative finance. Starting from:

$$
dS_t = \mu S_t\,dt + \sigma S_t\,dW_t,
$$

we want the dynamics of $Y_t = \ln S_t$.

**Step 1 — Identify the function and its derivatives.**

Set $f(S) = \ln S$. Then:

$$
f'(S) = \frac{1}{S}, \qquad f''(S) = -\frac{1}{S^2}.
$$

(There is no explicit time dependence, so $f_t = 0$.)

**Step 2 — Apply Itô's Lemma.**

With $dS = \mu S\,dt + \sigma S\,dW$, the general formula gives:

$$
d(\ln S) = f'(S)\,dS + \frac{1}{2}f''(S)\,(dS)^2.
$$

**Step 3 — Compute $f'(S)\,dS$.**

$$
f'(S)\,dS = \frac{1}{S}\bigl(\mu S\,dt + \sigma S\,dW_t\bigr) = \mu\,dt + \sigma\,dW_t.
$$

**Step 4 — Compute $(dS)^2$.**

$$
(dS)^2 = (\mu S\,dt + \sigma S\,dW_t)^2.
$$

Expanding and applying the Itô table:

$$
(dS)^2 = \mu^2 S^2\underbrace{(dt)^2}_{=\,0} + 2\mu\sigma S^2\underbrace{dt\,dW_t}_{=\,0} + \sigma^2 S^2\underbrace{(dW_t)^2}_{=\,dt} = \sigma^2 S^2\,dt.
$$

**Step 5 — Compute $\frac{1}{2}f''(S)(dS)^2$.**

$$
\frac{1}{2}\left(-\frac{1}{S^2}\right)\sigma^2 S^2\,dt = -\frac{1}{2}\sigma^2\,dt.
$$

**Step 6 — Combine.**

$$
d(\ln S_t) = \mu\,dt + \sigma\,dW_t - \frac{1}{2}\sigma^2\,dt = \left(\mu - \frac{\sigma^2}{2}\right)dt + \sigma\,dW_t.
$$

$$
\boxed{d(\ln S_t) = \left(\mu - \frac{\sigma^2}{2}\right)dt + \sigma\,dW_t.}
$$

**Step 7 — Integrate both sides from $0$ to $t$.**

$$
\ln S_t - \ln S_0 = \left(\mu - \frac{\sigma^2}{2}\right)t + \sigma W_t.
$$

$$
\boxed{S_t = S_0 \exp\!\left[\left(\mu - \frac{\sigma^2}{2}\right)t + \sigma W_t\right].}
$$

**Where does the $-\sigma^2/2$ come from?** It is entirely due to the second-derivative (Itô correction) term $\frac{1}{2}f''(S)(dS)^2 = -\frac{1}{2}\sigma^2\,dt$. Because $f(S) = \ln S$ is concave ($f'' < 0$), the stochastic fluctuations systematically reduce the growth rate of the logarithm relative to the arithmetic growth rate $\mu$. In financial terms, the *median* path of GBM grows at rate $\mu - \sigma^2/2$ (the geometric mean return) while the *expected* path grows at rate $\mu$ (the arithmetic mean return). This wedge is the "volatility drag" that erodes compounded returns.

### 5. The Itô Integral — Definition as an $L^2$ Limit

The Itô integral $\int_0^T f(t)\,dW_t$ is defined as the $L^2$ (mean-square) limit of a sequence of simple approximations.

**Step 1 — Simple processes.** A simple (step) process on $[0, T]$ is a function of the form

$$
f_n(t) = \sum_{i=0}^{n-1} \phi_i \,\mathbf{1}_{[t_i, t_{i+1})}(t),
$$

where $\Pi_n = \{0 = t_0 < t_1 < \cdots < t_n = T\}$ is a partition and each $\phi_i$ is $\mathcal{F}_{t_i}$-measurable (adapted — it uses only information available at the *left* endpoint $t_i$, not at $t_{i+1}$; this is the crucial non-anticipating property).

**Step 2 — Integral of a simple process.** Define

$$
I_n(f_n) = \sum_{i=0}^{n-1} \phi_i \bigl(W_{t_{i+1}} - W_{t_i}\bigr).
$$

This is a finite sum of random variables and is well-defined.

**Step 3 — Extension to general integrands.** For a general adapted process $f(t)$ with $\mathbb{E}\!\left[\int_0^T f(t)^2\,dt\right] < \infty$, there exists a sequence of simple processes $f_n$ such that

$$
\mathbb{E}\!\left[\int_0^T |f(t) - f_n(t)|^2\,dt\right] \;\xrightarrow{n \to \infty}\; 0.
$$

**Step 4 — Define the integral as the $L^2$ limit.**

$$
\int_0^T f(t)\,dW_t \;:=\; \lim_{n \to \infty} I_n(f_n) \quad \text{in } L^2(\Omega).
$$

The key fact that makes this limit exist and be unique is the **Itô isometry** (below): the map $f \mapsto I(f)$ preserves the $L^2$ norm, so convergence in the function space implies convergence of the integrals.

**Important properties of the Itô integral:**

- **Martingale property:** $M_t = \int_0^t f(s)\,dW_s$ is a martingale: $\mathbb{E}[M_t \mid \mathcal{F}_s] = M_s$ for $s \le t$.
- **Zero expectation:** $\mathbb{E}\!\left[\int_0^T f(t)\,dW_t\right] = 0$.
- **Non-anticipating:** The integrand $f(t)$ is evaluated at the left endpoint of each subinterval, which ensures causality (no peeking into the future).

### 6. The Itô Isometry

**Theorem.** For any adapted process $f$ with $\mathbb{E}\!\left[\int_0^T f(t)^2\,dt\right] < \infty$:

$$
\boxed{\mathbb{E}\!\left[\left(\int_0^T f(t)\,dW_t\right)^{\!2}\right] = \mathbb{E}\!\left[\int_0^T f(t)^2\,dt\right].}
$$

**Proof for simple processes.** Let $f_n(t) = \sum_i \phi_i \mathbf{1}_{[t_i, t_{i+1})}(t)$. Then

$$
I_n = \sum_{i=0}^{n-1} \phi_i \,\Delta W_i.
$$

Square this:

$$
I_n^2 = \sum_{i=0}^{n-1} \sum_{j=0}^{n-1} \phi_i \phi_j \,\Delta W_i \,\Delta W_j.
$$

Take expectations. For $i \ne j$, say $i < j$: $\phi_i \phi_j \Delta W_i$ is $\mathcal{F}_{t_j}$-measurable and $\Delta W_j$ is independent of $\mathcal{F}_{t_j}$ with $\mathbb{E}[\Delta W_j] = 0$. Therefore

$$
\mathbb{E}[\phi_i \phi_j \,\Delta W_i \,\Delta W_j] = \mathbb{E}[\phi_i \phi_j \Delta W_i] \cdot \mathbb{E}[\Delta W_j] = 0.
$$

Only diagonal terms survive:

$$
\mathbb{E}[I_n^2] = \sum_{i=0}^{n-1} \mathbb{E}[\phi_i^2 \,(\Delta W_i)^2] = \sum_{i=0}^{n-1} \mathbb{E}[\phi_i^2]\,\Delta t_i = \mathbb{E}\!\left[\int_0^T f_n(t)^2\,dt\right].
$$

The extension to general $f$ follows by taking $L^2$ limits. $\blacksquare$

**Why it matters:**

1. **Variance computation.** If $dX = \sigma\,dW$, then $\operatorname{Var}(X_T - X_0) = \mathbb{E}[\sigma^2 T] = \sigma^2 T$. More generally, for time-varying $\sigma(t)$, you integrate $\sigma(t)^2$.
2. **Convergence guarantee.** The isometry shows that the Itô integral map $f \mapsto \int f\,dW$ is an isometry from $L^2([0,T] \times \Omega)$ to $L^2(\Omega)$. This is what makes the $L^2$ limit in the definition well-defined.
3. **Monte Carlo error analysis.** The variance of a stochastic integral (e.g. a hedging error) is computed via the isometry without needing to evaluate the integral pathwise.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Drift | $\mu$ | Deterministic trend in the SDE | Annualised: $-0.1$ to $0.3$ for equities |
| Diffusion | $\sigma$ | Volatility coefficient | Annualised: $0.1$ to $1.0$ |
| Time horizon | $T$ | Integration endpoint | Seconds (HFT) to years (options) |
| Risk-free rate | $r$ | Replaces $\mu$ under risk-neutral measure | $0$ to $0.06$ |
| Itô correction | $-\sigma^2/2$ | Drift adjustment in log-coordinates | Depends on $\sigma$; e.g. $-0.02$ for $\sigma = 0.2$ |

## Numerical Example

**Task:** Verify the Itô correction numerically. Simulate $M = 100{,}000$ paths of GBM with $\mu = 0.10$, $\sigma = 0.30$, $S_0 = 100$, $T = 1$ year, and check that the mean of $\ln(S_T / S_0)$ is close to $(\mu - \sigma^2/2)T = (0.10 - 0.045) = 0.055$, not $\mu T = 0.10$.

```python
import numpy as np

rng = np.random.default_rng(7)

S0, mu, sigma, T = 100.0, 0.10, 0.30, 1.0
n_steps = 1000
dt = T / n_steps
M = 100_000

# Simulate GBM via Euler-Maruyama
S = np.full(M, S0)
for _ in range(n_steps):
    Z = rng.standard_normal(M)
    S = S + mu * S * dt + sigma * S * np.sqrt(dt) * Z

log_returns = np.log(S / S0)

theory_mean = (mu - 0.5 * sigma**2) * T     # 0.055  (Itô correction)
theory_var  = sigma**2 * T                   # 0.09
wrong_mean  = mu * T                         # 0.10   (ignoring correction)

print(f"Empirical mean of ln(S_T/S0): {log_returns.mean():.4f}")
print(f"Itô-corrected theory:         {theory_mean:.4f}")
print(f"Naive (no correction):         {wrong_mean:.4f}")
print(f"Empirical variance:            {log_returns.var():.4f}")
print(f"Theory variance:               {theory_var:.4f}")
```

**Expected output:**

```
Empirical mean of ln(S_T/S0): 0.0551
Itô-corrected theory:         0.0550
Naive (no correction):         0.1000
Empirical variance:            0.0901
Theory variance:               0.0900
```

The empirical mean matches $\mu - \sigma^2/2 = 0.055$, not $\mu = 0.10$, confirming that the Itô correction is real and non-negligible for $\sigma = 0.30$.

## Connections

- **[Brownian Motion](brownian_motion.md):** Itô calculus is built entirely on top of the properties of BM — in particular the quadratic variation $\langle W \rangle_T = T$ and the Itô multiplication table derived there.
- **[Stochastic Differential Equations](sdes.md):** Itô's Lemma is the main tool for solving SDEs. The exact solutions for GBM and OU processes are obtained by applying it to appropriate transformations.
- **[Black–Scholes](../02_options/black_scholes.md):** The Black–Scholes PDE is derived by applying Itô's Lemma to a portfolio of the option and the underlying, then imposing the no-arbitrage condition. The $\frac{1}{2}\sigma^2 S^2 V_{SS}$ term in the PDE is the Itô correction.
- **[Greeks](../02_options/greeks.md):** Delta, Gamma, Theta and other Greeks are the partial derivatives that appear in Itô's Lemma applied to the option price function $V(t, S)$. Gamma ($V_{SS}$) matters precisely because the Itô correction $\frac{1}{2}\sigma^2 S^2 V_{SS}\,dt$ is non-zero.
- **[Implied Volatility](../02_options/implied_volatility.md):** The Itô correction $-\sigma^2/2$ explains why implied volatility (which enters the BS formula) differs from realised drift: the market prices the *geometric* growth rate, not the arithmetic one.

## Relevance for Prosperity 4

In Prosperity 4, Itô calculus underpins several concrete tasks:

1. **Log-return modelling.** When estimating expected returns from mid-price data, use $\hat{\mu}_{\text{log}} = \bar{r}_{\text{log}} + \frac{1}{2}\hat{\sigma}^2$ to recover the arithmetic drift from the sample mean of log-returns. Ignoring the Itô correction underestimates the true expected growth rate by $\sigma^2/2$.
2. **Options pricing.** If the competition includes options or option-like payoffs (e.g. vouchers with convex payoffs), the entire pricing machinery — from the BS formula through Greeks-based hedging — relies on Itô's Lemma to derive the replicating portfolio dynamics.
3. **Inventory risk.** The Avellaneda–Stoikov reservation price $r = S - q\gamma\sigma^2(T-t)$ is derived via Itô calculus applied to the market maker's exponential utility. Understanding the derivation lets you extend the model (e.g. adding drift or jumps) when the standard AS assumptions break down.
4. **Model validation.** Checking whether the $-\sigma^2/2$ correction matches empirical log-return means is a quick diagnostic for whether the diffusion model is well-calibrated. A persistent discrepancy suggests drift, jumps, or microstructure effects that your strategy should account for.
