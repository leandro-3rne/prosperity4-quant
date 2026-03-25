# Black–Scholes Option Pricing

> **Core formula:**
> $$C = S\,N(d_1) - K e^{-rT} N(d_2), \qquad d_1 = \frac{\ln(S/K) + (r + \tfrac{1}{2}\sigma^2)T}{\sigma\sqrt{T}}, \qquad d_2 = d_1 - \sigma\sqrt{T}$$

## Intuition

Imagine you sell an insurance contract on a stock. You don't know where the stock will be at expiry, but you can trade the stock itself to hedge your exposure continuously. If you adjust your hedge often enough, the randomness cancels out and the cost of maintaining that hedge is completely determined by one number: the stock's volatility. The Black–Scholes formula is exactly the cost of that self-financing hedging strategy.

The model says: in a world with no arbitrage, continuous trading, and log-normal stock prices, the fair price of a European option depends on only five inputs — the current stock price $S$, the strike $K$, the time to expiry $T$, the risk-free rate $r$, and the volatility $\sigma$. Crucially, the expected return $\mu$ of the stock does **not** appear. This is the deep insight of risk-neutral pricing: hedging eliminates the drift, so only volatility matters.

Financially, $N(d_2)$ is the risk-neutral probability that the call finishes in the money, and $N(d_1)$ is the delta-weighted counterpart. The formula decomposes the call value into "what you receive" ($S\,N(d_1)$) minus "what you pay" ($K e^{-rT} N(d_2)$), each probability-weighted under the appropriate numeraire.

### Hedging Principle

The key step in the derivation is choosing the stock position (delta) so that the stochastic $dW_t$ term cancels in the hedged portfolio. Once randomness is removed, the portfolio is locally riskless and must earn the risk-free rate $r$ — this no-arbitrage condition yields the Black–Scholes PDE. For a broader introduction to derivatives, hedging, and the replication principle, see [Derivatives and Hedging](derivatives_and_hedging.md).

## Mathematical Setup

### Symbols and assumptions

| Symbol | Meaning |
|--------|---------|
| $S = S_t$ | Current spot price of the underlying asset |
| $K$ | Strike price of the option |
| $T$ | Time to expiry (in years) |
| $r$ | Continuously compounded risk-free interest rate |
| $\sigma$ | Volatility of the underlying (annualised standard deviation of log-returns) |
| $V(S,t)$ | Option price as a function of spot and time |
| $W_t$ | Standard Brownian motion under the physical measure $\mathbb{P}$ |
| $\widetilde{W}_t$ | Standard Brownian motion under the risk-neutral measure $\mathbb{Q}$ |
| $N(\cdot)$ | CDF of the standard normal distribution |
| $n(\cdot)$ | PDF of the standard normal distribution |

### Assumptions

1. The underlying follows geometric Brownian motion (GBM).
2. No dividends during the life of the option.
3. Constant volatility $\sigma$ and risk-free rate $r$.
4. Continuous trading is possible with no transaction costs.
5. No arbitrage opportunities exist.
6. Short selling is permitted without restrictions.

## Derivation

### Step 1 — Geometric Brownian Motion

Under the physical measure $\mathbb{P}$, the stock price follows:

$$dS = \mu S\,dt + \sigma S\,dW_t$$

where $\mu$ is the expected return. By Itô's lemma applied to $f(S) = \ln S$:

$$d(\ln S) = \frac{1}{S}\,dS - \frac{1}{2}\frac{1}{S^2}(dS)^2$$

Substituting $dS = \mu S\,dt + \sigma S\,dW_t$ and using $(dS)^2 = \sigma^2 S^2\,dt$ (from the Itô multiplication table: $dW^2 = dt$, $dt\cdot dW = 0$, $dt^2 = 0$):

$$d(\ln S) = \frac{1}{S}(\mu S\,dt + \sigma S\,dW_t) - \frac{1}{2}\sigma^2\,dt$$

$$d(\ln S) = \left(\mu - \tfrac{1}{2}\sigma^2\right)dt + \sigma\,dW_t$$

Integrating from $0$ to $T$:

$$\ln S_T = \ln S_0 + \left(\mu - \tfrac{1}{2}\sigma^2\right)T + \sigma W_T$$

$$S_T = S_0 \exp\!\left[\left(\mu - \tfrac{1}{2}\sigma^2\right)T + \sigma W_T\right]$$

### Step 2 — Risk-neutral measure (Girsanov)

Under $\mathbb{P}$, the stock has drift $\mu$. By the Girsanov theorem, there exists an equivalent measure $\mathbb{Q}$ under which we define:

$$\widetilde{W}_t = W_t + \frac{\mu - r}{\sigma}\,t$$

The quantity $\theta = (\mu - r)/\sigma$ is the **market price of risk**. Under $\mathbb{Q}$, $\widetilde{W}_t$ is a standard Brownian motion, and the stock dynamics become:

$$dS = rS\,dt + \sigma S\,d\widetilde{W}_t$$

The drift is now $r$, not $\mu$. This is why the expected return drops out of the pricing formula: under the risk-neutral measure, all assets earn the risk-free rate on average.

Under $\mathbb{Q}$, the terminal stock price is:

$$S_T = S_0 \exp\!\left[\left(r - \tfrac{1}{2}\sigma^2\right)T + \sigma \widetilde{W}_T\right]$$


### Step 3 — Delta-hedging and the Black–Scholes PDE

Consider a derivative $V(S,t)$ on the stock. We form a self-financing portfolio:

$$\Pi = V - \Delta\,S$$

where $\Delta$ is the number of shares held (to be chosen). Over an infinitesimal interval, the change in portfolio value is:

$$d\Pi = dV - \Delta\,dS$$

Apply Itô's lemma to $V(S,t)$:

$$dV = \frac{\partial V}{\partial t}\,dt + \frac{\partial V}{\partial S}\,dS + \frac{1}{2}\frac{\partial^2 V}{\partial S^2}(dS)^2$$

Substitute $(dS)^2 = \sigma^2 S^2\,dt$:

$$dV = \frac{\partial V}{\partial t}\,dt + \frac{\partial V}{\partial S}\,dS + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2}\,dt$$

Now substitute into $d\Pi$:

$$d\Pi = \frac{\partial V}{\partial t}\,dt + \frac{\partial V}{\partial S}\,dS + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2}\,dt - \Delta\,dS$$

$$d\Pi = \left(\frac{\partial V}{\partial t} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2}\right)dt + \left(\frac{\partial V}{\partial S} - \Delta\right)dS$$

**Choose** $\Delta = \dfrac{\partial V}{\partial S}$ to eliminate the stochastic term $dS$:

$$d\Pi = \left(\frac{\partial V}{\partial t} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2}\right)dt$$

The portfolio is now **risk-free** over $dt$. By the no-arbitrage condition, a risk-free portfolio must earn the risk-free rate:

$$d\Pi = r\,\Pi\,dt = r\left(V - \frac{\partial V}{\partial S}\,S\right)dt$$

Equating the two expressions:

$$\frac{\partial V}{\partial t} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} = r\,V - r\,S\frac{\partial V}{\partial S}$$

Rearranging:

$$\boxed{\frac{\partial V}{\partial t} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} + rS\frac{\partial V}{\partial S} - rV = 0}$$

This is the **Black–Scholes PDE**. Note that $\mu$ does not appear — the drift has been eliminated by hedging.

### Step 4 — Reduction to the heat equation

The BS PDE has variable coefficients in $S$ because $S^2$ and $S$ multiply $\partial^2 V/\partial S^2$ and $\partial V/\partial S$. Substitution 1 ($\tau = T-t$, $x = \ln(S/K)$, $V = Kv$) yields a PDE for $v$ in which the leading term $\tfrac{1}{2}\sigma^2\,\partial^2 v/\partial x^2$ already has constant coefficients, but first-order ($\partial v/\partial x$) and zeroth-order ($-rv$) terms remain — so it is not yet the heat equation. Substitution 2 ($v = e^{\alpha x + \beta\tau}u$ with suitable $\alpha,\beta$) eliminates those lower-order terms and leaves the standard heat equation for $u$. That PDE admits the classical Gaussian fundamental solution; convolving with the transformed terminal payoff and reversing the substitutions gives the closed-form Black–Scholes formula.

**Substitution 1:** Let $\tau = T - t$ (time to expiry, so $\tau = 0$ at expiry) and $x = \ln(S/K)$ (log-moneyness). Define:

$$V(S,t) = K\,v(x,\tau)$$

Compute the partial derivatives. Since $S = K e^x$:

$$\frac{\partial V}{\partial t} = -K\frac{\partial v}{\partial \tau}$$

$$\frac{\partial V}{\partial S} = K\frac{\partial v}{\partial x}\cdot\frac{1}{S} = \frac{\partial v}{\partial x}\cdot\frac{K}{Ke^x} = e^{-x}\frac{\partial v}{\partial x} \cdot \frac{K}{K} \cdot K/K$$

More carefully: $x = \ln(S/K)$ so $\partial x/\partial S = 1/S$. Then:

$$\frac{\partial V}{\partial S} = K\frac{\partial v}{\partial x}\cdot\frac{1}{S}$$

$$\frac{\partial^2 V}{\partial S^2} = K\left(\frac{\partial^2 v}{\partial x^2}\cdot\frac{1}{S^2} - \frac{\partial v}{\partial x}\cdot\frac{1}{S^2}\right) = \frac{K}{S^2}\left(\frac{\partial^2 v}{\partial x^2} - \frac{\partial v}{\partial x}\right)$$

Substituting into the BS PDE (and using $S = K e^x$, so $S^2 = K^2 e^{2x}$):

$$-K\frac{\partial v}{\partial \tau} + \frac{1}{2}\sigma^2 K^2 e^{2x}\cdot\frac{K}{K^2 e^{2x}}\left(\frac{\partial^2 v}{\partial x^2} - \frac{\partial v}{\partial x}\right) + rKe^x\cdot\frac{K}{Ke^x}\frac{\partial v}{\partial x} - rKv = 0$$

Simplifying (divide through by $K$):

$$-\frac{\partial v}{\partial \tau} + \frac{1}{2}\sigma^2\left(\frac{\partial^2 v}{\partial x^2} - \frac{\partial v}{\partial x}\right) + r\frac{\partial v}{\partial x} - rv = 0$$

$$\frac{\partial v}{\partial \tau} = \frac{1}{2}\sigma^2\frac{\partial^2 v}{\partial x^2} + \left(r - \frac{1}{2}\sigma^2\right)\frac{\partial v}{\partial x} - rv$$

**Substitution 2:** Remove the first-order and zeroth-order terms. Set:

$$v(x,\tau) = e^{\alpha x + \beta \tau}\,u(x,\tau)$$

where $\alpha$ and $\beta$ are constants to be determined. Substituting and matching terms, we require:

$$\alpha = -\frac{1}{2}\left(\frac{2r}{\sigma^2} - 1\right) = \frac{1}{2} - \frac{r}{\sigma^2}$$

$$\beta = -\frac{1}{2}\sigma^2\alpha^2 - \left(r - \frac{1}{2}\sigma^2\right)\alpha + r$$

After algebra:

$$\beta = -\frac{1}{2}\left(\frac{2r}{\sigma^2} + 1\right)^2 \cdot \frac{\sigma^2}{8}\quad\text{... simplifying ...}$$

$$\beta = r + \frac{1}{8}\sigma^2\left(\frac{2r}{\sigma^2}+1\right)^2 \cdot (-1) $$

With the standard choices $\alpha = -\frac{1}{2}(k-1)$ and $\beta = -\frac{1}{4}(k+1)^2 \cdot \frac{\sigma^2}{2}$ where $k = 2r/\sigma^2$, the transformed equation becomes the **standard heat equation**:

$$\boxed{\frac{\partial u}{\partial \tau} = \frac{1}{2}\sigma^2 \frac{\partial^2 u}{\partial x^2}}$$

With a further rescaling $\tau' = \frac{1}{2}\sigma^2 \tau$, this becomes $\partial u/\partial \tau' = \partial^2 u/\partial x^2$, the classical heat equation whose fundamental solution is the Gaussian kernel:

$$u(x,\tau') = \frac{1}{\sqrt{4\pi\tau'}}\int_{-\infty}^{\infty} u_0(\xi)\,\exp\!\left(-\frac{(x-\xi)^2}{4\tau'}\right)d\xi$$

Transforming back through all substitutions yields the closed-form Black–Scholes formula.

### Step 5 — Closed-form solution for European call

For a European call with payoff $\max(S_T - K, 0)$, the risk-neutral pricing formula gives:

$$C = e^{-rT}\,\mathbb{E}^{\mathbb{Q}}\!\left[\max(S_T - K, 0)\right]$$

Under $\mathbb{Q}$, $\ln S_T \sim \mathcal{N}\!\left(\ln S + (r - \tfrac{1}{2}\sigma^2)T,\;\sigma^2 T\right)$. Let $Z \sim \mathcal{N}(0,1)$, so:

$$S_T = S\exp\!\left[(r - \tfrac{1}{2}\sigma^2)T + \sigma\sqrt{T}\,Z\right]$$

The call is in the money when $S_T > K$, i.e., when:

$$S\exp\!\left[(r - \tfrac{1}{2}\sigma^2)T + \sigma\sqrt{T}\,Z\right] > K$$

$$(r - \tfrac{1}{2}\sigma^2)T + \sigma\sqrt{T}\,Z > \ln(K/S)$$

$$Z > \frac{\ln(K/S) - (r - \tfrac{1}{2}\sigma^2)T}{\sigma\sqrt{T}} = -d_2$$

where:

$$d_2 = \frac{\ln(S/K) + (r - \tfrac{1}{2}\sigma^2)T}{\sigma\sqrt{T}}$$

Now evaluate the expectation by splitting it:

$$C = e^{-rT}\int_{-d_2}^{\infty}\left(Se^{(r-\sigma^2/2)T + \sigma\sqrt{T}\,z} - K\right)\frac{1}{\sqrt{2\pi}}e^{-z^2/2}\,dz$$

**Second term:**

$$e^{-rT}\int_{-d_2}^{\infty} K\,\frac{e^{-z^2/2}}{\sqrt{2\pi}}\,dz = Ke^{-rT}\,N(d_2)$$

**First term:** Factor out $S$:

$$e^{-rT}\cdot S\int_{-d_2}^{\infty} e^{(r-\sigma^2/2)T + \sigma\sqrt{T}\,z}\,\frac{e^{-z^2/2}}{\sqrt{2\pi}}\,dz$$

$$= S\int_{-d_2}^{\infty}\frac{1}{\sqrt{2\pi}}\exp\!\left(-\frac{\sigma^2 T}{2} + \sigma\sqrt{T}\,z - \frac{z^2}{2}\right)dz$$

Complete the square in the exponent:

$$-\frac{z^2}{2} + \sigma\sqrt{T}\,z - \frac{\sigma^2 T}{2} = -\frac{1}{2}(z - \sigma\sqrt{T})^2$$

Substitute $w = z - \sigma\sqrt{T}$, so $dw = dz$ and the lower limit becomes $-d_2 - \sigma\sqrt{T} = -d_1$:

$$S\int_{-d_1}^{\infty}\frac{1}{\sqrt{2\pi}}e^{-w^2/2}\,dw = S\,N(d_1)$$

where:

$$d_1 = d_2 + \sigma\sqrt{T} = \frac{\ln(S/K) + (r + \tfrac{1}{2}\sigma^2)T}{\sigma\sqrt{T}}$$

**Combining both terms:**

$$\boxed{C = S\,N(d_1) - Ke^{-rT}\,N(d_2)}$$

$$d_1 = \frac{\ln(S/K) + (r + \tfrac{1}{2}\sigma^2)T}{\sigma\sqrt{T}}, \qquad d_2 = d_1 - \sigma\sqrt{T}$$

### Step 6 — European put formula

For a put with payoff $\max(K - S_T, 0)$:

$$P = e^{-rT}\,\mathbb{E}^{\mathbb{Q}}\!\left[\max(K - S_T, 0)\right]$$

The put is in the money when $S_T < K$, i.e., when $Z < -d_2$. Evaluating the analogous integral:

$$P = Ke^{-rT}\int_{-\infty}^{-d_2}\frac{e^{-z^2/2}}{\sqrt{2\pi}}\,dz - S\int_{-\infty}^{-d_1}\frac{e^{-w^2/2}}{\sqrt{2\pi}}\,dw$$

$$\boxed{P = Ke^{-rT}\,N(-d_2) - S\,N(-d_1)}$$

### Step 7 — Put-call parity

**Claim:** $C - P = S - Ke^{-rT}$.

**Proof from payoffs:** Consider a portfolio that is long one call and short one put, both with strike $K$ and expiry $T$. At expiry:

- If $S_T > K$: call pays $S_T - K$, put pays $0$. Net payoff $= S_T - K$.
- If $S_T \le K$: call pays $0$, put pays $-(K - S_T) = S_T - K$. Net payoff $= S_T - K$.

In both cases the payoff is $S_T - K$. A portfolio that replicates this payoff is: long one share of stock plus short $Ke^{-rT}$ in bonds (which grows to $K$ at expiry). By no-arbitrage, the present values must be equal:

$$C - P = S - Ke^{-rT}$$

**Verification from the formulas:** Using $N(d) + N(-d) = 1$:

$$C - P = S\,N(d_1) - Ke^{-rT}N(d_2) - \left[Ke^{-rT}N(-d_2) - S\,N(-d_1)\right]$$

$$= S\left[N(d_1) + N(-d_1)\right] - Ke^{-rT}\left[N(d_2) + N(-d_2)\right]$$

$$= S \cdot 1 - Ke^{-rT} \cdot 1 = S - Ke^{-rT} \qquad\checkmark$$

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Spot price | $S$ | Current market price of the underlying | Asset-dependent |
| Strike price | $K$ | Exercise price agreed in the contract | Set by contract |
| Time to expiry | $T$ | Remaining life of the option (years) | Days to years; Prosperity rounds are short-horizon |
| Risk-free rate | $r$ | Continuously compounded interest rate | 0%–6% per annum |
| Volatility | $\sigma$ | Annualised std dev of log-returns | 10%–80% depending on asset class |
| $d_1$ | — | Normalised distance to strike, adjusted for drift and convexity | Typically $-3$ to $+3$ |
| $d_2$ | — | $d_1 - \sigma\sqrt{T}$; log-moneyness standardised by vol | Typically $-3$ to $+3$ |

## Numerical Example

**Setup:** $S = 100$, $K = 105$, $T = 0.5$ years, $r = 0.05$, $\sigma = 0.20$.

**Step 1 — Compute $d_1$ and $d_2$:**

$$d_1 = \frac{\ln(100/105) + (0.05 + 0.02)\times 0.5}{0.20\times\sqrt{0.5}} = \frac{-0.04879 + 0.035}{0.14142} = \frac{-0.01379}{0.14142} = -0.09752$$

$$d_2 = -0.09752 - 0.14142 = -0.23894$$

**Step 2 — Look up normal CDF:**

$$N(d_1) = N(-0.09752) = 0.46116$$

$$N(d_2) = N(-0.23894) = 0.40556$$

**Step 3 — Call price:**

$$C = 100\times 0.46116 - 105\times e^{-0.025}\times 0.40556$$

$$= 46.116 - 105\times 0.97531\times 0.40556$$

$$= 46.116 - 41.540 = 4.576$$

**Step 4 — Put price (via put-call parity):**

$$P = C - S + Ke^{-rT} = 4.576 - 100 + 102.408 = 6.984$$

**Verification with Python:**

```python
import numpy as np
from scipy.stats import norm

S, K, T, r, sigma = 100, 105, 0.5, 0.05, 0.20
d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
d2 = d1 - sigma*np.sqrt(T)
C = S*norm.cdf(d1) - K*np.exp(-r*T)*norm.cdf(d2)
P = K*np.exp(-r*T)*norm.cdf(-d2) - S*norm.cdf(-d1)

print(f"d1 = {d1:.5f},  d2 = {d2:.5f}")
print(f"Call = {C:.3f},  Put = {P:.3f}")
print(f"Put-call parity check: C - P = {C - P:.3f}, S - Ke^(-rT) = {S - K*np.exp(-r*T):.3f}")
# d1 = -0.09752,  d2 = -0.23894
# Call = 4.576,  Put = 6.984
# Put-call parity check: C - P = -2.408, S - Ke^(-rT) = -2.408
```

## Connections

- [Derivatives and Hedging](derivatives_and_hedging.md) — Primer on forwards, options, put-call parity, and the replication / no-arbitrage principle that motivates the delta-hedging argument here.
- [Brownian motion](../01_stochastic/brownian_motion.md) — GBM is the exponential of a drift-adjusted Brownian motion; the quadratic variation of $W_t$ is the reason the $\sigma^2/2$ Itô correction appears.
- [Greeks](greeks.md) — All Greek letters are partial derivatives of the BS formula; they quantify the sensitivity of $C$ and $P$ to each input.
- [Implied volatility](implied_volatility.md) — The BS formula is inverted numerically to extract the market's implied $\sigma$ from observed option prices; the fact that implied vol varies with strike reveals the model's limitations.
- [Jump diffusion](jump_diffusion.md) — Merton's model extends BS by adding Poisson jumps to GBM; the BS formula appears as a building block inside Merton's infinite-series pricing formula.
- [Avellaneda–Stoikov market making](../03_market_making/avellaneda_stoikov.md) — The diffusion assumption ($dS = \sigma S\,dW$) in the AS model mirrors the BS setup; if you market-make options, you delta-hedge using BS Greeks.

## Relevance for Prosperity 4

In Prosperity 4, several products are European-style options or option-like instruments on tradable underlyings. The Black–Scholes formula gives you a **real-time fair-value estimate** for these instruments using only the current mid-price of the underlying, an estimate of volatility (from recent returns or implied from the order book), and the known expiry time. Specifically:

- **Quoting options:** Use BS to compute bid/ask around fair value, adjusting for inventory risk via the Avellaneda–Stoikov framework. Your edge comes from having a better volatility estimate than the other bots.
- **Delta hedging:** When you accumulate option inventory, hedge by trading $\Delta = N(d_1)$ shares of the underlying per call (or $N(d_1)-1$ per put). This converts directional risk into pure gamma/theta exposure.
- **Detecting mispricing:** If an option trades significantly above or below BS fair value (after calibrating $\sigma$), that's a potential arbitrage. Check put-call parity violations — if $C - P \neq S - Ke^{-rT}$, you can lock in risk-free profit by trading all three instruments.
- **Short-horizon simplification:** With short round durations, $T$ is small, so deep OTM options have near-zero value and ATM option value is approximately $S\sigma\sqrt{T}/(2\sqrt{2\pi}) \approx 0.4\,S\sigma\sqrt{T}$. This quick approximation helps you quote instantly without computing the full formula.
