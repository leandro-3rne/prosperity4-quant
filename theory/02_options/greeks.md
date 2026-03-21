# The Greeks

> **Core formula:**
> $$\Delta = \frac{\partial V}{\partial S}, \quad \Gamma = \frac{\partial^2 V}{\partial S^2}, \quad \mathcal{V} = \frac{\partial V}{\partial \sigma}, \quad \Theta = \frac{\partial V}{\partial t}, \quad \rho = \frac{\partial V}{\partial r}$$

## Intuition

Options are nonlinear instruments — their value curves with respect to every input, rather than moving in a straight line. The Greeks measure how the option price responds to small changes in each input, one at a time. Think of them as the "dashboard gauges" of a car: Delta tells you how fast you're going (directional exposure), Gamma tells you how quickly your steering responds (convexity), Theta is fuel consumption (time decay), Vega measures sensitivity to road conditions (volatility), and Rho captures the cost of financing.

For a trader, the Greeks translate abstract partial derivatives into concrete dollar risks. If your portfolio Delta is +500, a \$1 move in the underlying gains you \$500. If your Gamma is +50, that Delta increases by 50 for every additional dollar move, making gains accelerate (or losses, if you're short gamma). This nonlinearity is what makes options trading fundamentally different from trading the underlying.

The most important tradeoff in options is between Gamma and Theta: buying options gives you positive Gamma (convexity — you benefit from large moves) but negative Theta (you bleed value every day). Selling options gives you the opposite profile. Understanding this tradeoff is the foundation of every options strategy.

## Mathematical Setup

All formulas below are derived from the Black–Scholes call and put prices:

$$C = S\,N(d_1) - Ke^{-rT}\,N(d_2)$$

$$P = Ke^{-rT}\,N(-d_2) - S\,N(-d_1)$$

where:

$$d_1 = \frac{\ln(S/K) + (r + \tfrac{1}{2}\sigma^2)T}{\sigma\sqrt{T}}, \qquad d_2 = d_1 - \sigma\sqrt{T}$$

and $N(\cdot)$ is the standard normal CDF and $n(\cdot) = N'(\cdot) = \frac{1}{\sqrt{2\pi}}e^{-x^2/2}$ is the standard normal PDF.

**Useful identities** (needed repeatedly in the derivations):

$$\frac{\partial d_1}{\partial S} = \frac{\partial d_2}{\partial S} = \frac{1}{S\sigma\sqrt{T}}$$

$$S\,n(d_1) = Ke^{-rT}\,n(d_2)$$

The second identity follows from:

$$\frac{n(d_1)}{n(d_2)} = \exp\!\left(-\frac{d_1^2 - d_2^2}{2}\right) = \exp\!\left(-\frac{(d_1-d_2)(d_1+d_2)}{2}\right)$$

Since $d_1 - d_2 = \sigma\sqrt{T}$ and $d_1 + d_2 = \frac{2\ln(S/K) + 2rT}{\sigma\sqrt{T}} - \sigma\sqrt{T} + \sigma\sqrt{T}$... more directly:

$$d_1^2 - d_2^2 = (d_1+d_2)(d_1-d_2) = (2d_1 - \sigma\sqrt{T})(\sigma\sqrt{T})$$

$$= 2d_1\sigma\sqrt{T} - \sigma^2 T = 2\ln(S/K) + 2rT$$

Therefore:

$$\frac{n(d_1)}{n(d_2)} = e^{-\ln(S/K) - rT} = \frac{Ke^{-rT}}{S}$$

which gives $S\,n(d_1) = Ke^{-rT}\,n(d_2)$. $\square$

## Derivation

### Delta — $\Delta = \partial V/\partial S$

**Call Delta:**

$$\Delta_{\text{call}} = \frac{\partial C}{\partial S} = N(d_1) + S\,n(d_1)\frac{\partial d_1}{\partial S} - Ke^{-rT}\,n(d_2)\frac{\partial d_2}{\partial S}$$

Since $\partial d_1/\partial S = \partial d_2/\partial S = \frac{1}{S\sigma\sqrt{T}}$ and $S\,n(d_1) = Ke^{-rT}\,n(d_2)$, the last two terms cancel:

$$\boxed{\Delta_{\text{call}} = N(d_1)}$$

Since $0 < N(d_1) < 1$, call Delta is always between 0 and 1.

**Put Delta:** Using $P = C - S + Ke^{-rT}$ (put-call parity):

$$\Delta_{\text{put}} = \Delta_{\text{call}} - 1 = N(d_1) - 1$$

Using the identity $N(x) - 1 = -N(-x)$:

$$\boxed{\Delta_{\text{put}} = -N(-d_1)}$$

Since $0 < N(-d_1) < 1$, put Delta is always between $-1$ and $0$.

**Interpretation:** Delta measures the equivalent stock position. A call with $\Delta = 0.6$ behaves like holding 0.6 shares for small price moves. Delta also approximates the probability (under $\mathbb{Q}$) that the option finishes in the money, though this approximation is only exact for digital options.

### Gamma — $\Gamma = \partial^2 V/\partial S^2 = \partial\Delta/\partial S$

**Call Gamma:**

$$\Gamma_{\text{call}} = \frac{\partial}{\partial S}N(d_1) = n(d_1)\frac{\partial d_1}{\partial S} = \frac{n(d_1)}{S\sigma\sqrt{T}}$$

**Put Gamma:** Differentiate $\Delta_{\text{put}} = N(d_1) - 1$:

$$\Gamma_{\text{put}} = \frac{\partial}{\partial S}\left[N(d_1) - 1\right] = n(d_1)\frac{\partial d_1}{\partial S} = \frac{n(d_1)}{S\sigma\sqrt{T}}$$

$$\boxed{\Gamma = \frac{n(d_1)}{S\sigma\sqrt{T}} \quad \text{(same for calls and puts)}}$$

Gamma is always positive for long options. It is largest when the option is at-the-money ($S \approx K$) and near expiry ($T$ small), because that's when Delta changes most rapidly with the stock price.

**Interpretation:** Gamma is the convexity of the option payoff. High Gamma means your Delta changes rapidly, so you benefit from large moves in either direction. This is why long Gamma positions profit from volatility.

### Vega — $\mathcal{V} = \partial V/\partial\sigma$

**Call Vega:**

$$\mathcal{V}_{\text{call}} = S\,n(d_1)\frac{\partial d_1}{\partial\sigma} + \left[-Ke^{-rT}\,n(d_2)\frac{\partial d_2}{\partial\sigma}\right] + \text{direct terms from } N$$

A cleaner approach: differentiate $C = S\,N(d_1) - Ke^{-rT}N(d_2)$ with respect to $\sigma$:

$$\mathcal{V}_{\text{call}} = S\,n(d_1)\frac{\partial d_1}{\partial\sigma} - Ke^{-rT}n(d_2)\frac{\partial d_2}{\partial\sigma}$$

Compute:

$$\frac{\partial d_1}{\partial\sigma} = -\frac{d_2}{\sigma}, \qquad \frac{\partial d_2}{\partial\sigma} = -\frac{d_1}{\sigma}$$

Wait — let us compute carefully. We have $d_1 = \frac{\ln(S/K) + (r+\sigma^2/2)T}{\sigma\sqrt{T}}$. Let $A = \ln(S/K) + rT$. Then $d_1 = \frac{A + \sigma^2 T/2}{\sigma\sqrt{T}} = \frac{A}{\sigma\sqrt{T}} + \frac{\sigma\sqrt{T}}{2}$.

$$\frac{\partial d_1}{\partial\sigma} = -\frac{A}{\sigma^2\sqrt{T}} + \frac{\sqrt{T}}{2} = -\frac{d_1}{\sigma} + \frac{\sigma\sqrt{T}}{\sigma} + \frac{\sqrt{T}}{2}$$

This gets messy. The standard shortcut uses the identity $S\,n(d_1) = Ke^{-rT}n(d_2)$ and the fact that $d_1 - d_2 = \sigma\sqrt{T}$:

$$\mathcal{V} = S\,n(d_1)\frac{\partial d_1}{\partial\sigma} - Ke^{-rT}n(d_2)\frac{\partial d_2}{\partial\sigma}$$

$$= S\,n(d_1)\left(\frac{\partial d_1}{\partial\sigma} - \frac{\partial d_2}{\partial\sigma}\right)$$

$$= S\,n(d_1)\frac{\partial}{\partial\sigma}(d_1 - d_2) = S\,n(d_1)\frac{\partial}{\partial\sigma}(\sigma\sqrt{T})$$

$$= S\,n(d_1)\sqrt{T}$$

$$\boxed{\mathcal{V} = S\,n(d_1)\sqrt{T} \quad \text{(same for calls and puts)}}$$

Vega is always positive for long options. This makes sense: more uncertainty (higher $\sigma$) always increases the value of optionality.

**Put Vega** is identical because put-call parity gives $C - P = S - Ke^{-rT}$, which does not depend on $\sigma$, so $\partial C/\partial\sigma = \partial P/\partial\sigma$.

**Interpretation:** Vega tells you how much the option price changes per 1-unit change in volatility. In practice, traders quote Vega per 1 percentage point change in vol: if $\mathcal{V} = 15$ and vol rises by 1%, the option gains \$0.15 (since $\sigma$ is quoted in decimal, multiply by 0.01).

### Theta — $\Theta = \partial V/\partial t$

Note: $\Theta$ is the derivative with respect to calendar time $t$, not time to expiry $\tau = T - t$. Since option value generally decreases as time passes, $\Theta$ is usually negative for long options.

**Call Theta:** Differentiate $C = S\,N(d_1) - Ke^{-rT}N(d_2)$ with respect to $t$ (holding $S$ fixed). We need $\partial d_1/\partial t$ and $\partial d_2/\partial t$. Using $\tau = T-t$ (so $\partial\tau/\partial t = -1$):

$$\frac{\partial d_1}{\partial t} = \frac{-(r+\sigma^2/2)\sigma\sqrt{\tau} - \left[\ln(S/K)+(r+\sigma^2/2)\tau\right]\cdot(-\sigma/(2\sqrt{\tau}))}{\sigma^2\tau}$$

After careful algebra (and using the cancellation identity $S\,n(d_1) = Ke^{-rT}n(d_2)$), the result is:

$$\boxed{\Theta_{\text{call}} = -\frac{S\,n(d_1)\,\sigma}{2\sqrt{T}} - rKe^{-rT}N(d_2)}$$

The first term is always negative (time decay from shrinking optionality). The second term is also negative for calls (you lose the interest on the deferred strike payment as time passes).

**Put Theta:** From put-call parity $P = C - S + Ke^{-rT}$, differentiating with respect to $t$:

$$\Theta_{\text{put}} = \Theta_{\text{call}} + rKe^{-rT}$$

$$= -\frac{S\,n(d_1)\,\sigma}{2\sqrt{T}} - rKe^{-rT}N(d_2) + rKe^{-rT}$$

$$\boxed{\Theta_{\text{put}} = -\frac{S\,n(d_1)\,\sigma}{2\sqrt{T}} + rKe^{-rT}N(-d_2)}$$

For puts, the second term is positive because you earn interest on the strike you will receive. Deep in-the-money puts can have positive total Theta.

**Interpretation:** Theta is "time decay" — the price you pay for holding optionality. At-the-money options have the largest (most negative) Theta because that's where the option value is most sensitive to remaining time.

### Rho — $\rho = \partial V/\partial r$

**Call Rho:** Differentiate $C$ with respect to $r$. The dominant term comes from the discount factor $e^{-rT}$:

$$\rho_{\text{call}} = S\,n(d_1)\frac{\partial d_1}{\partial r} - Ke^{-rT}n(d_2)\frac{\partial d_2}{\partial r} + KTe^{-rT}N(d_2)$$

The first two terms cancel by the same identity ($S\,n(d_1) = Ke^{-rT}n(d_2)$ and $\partial d_1/\partial r = \partial d_2/\partial r$):

$$\boxed{\rho_{\text{call}} = KTe^{-rT}N(d_2)}$$

**Put Rho:** From put-call parity $P = C - S + Ke^{-rT}$:

$$\rho_{\text{put}} = \rho_{\text{call}} - KTe^{-rT} = KTe^{-rT}N(d_2) - KTe^{-rT} = -KTe^{-rT}N(-d_2)$$

$$\boxed{\rho_{\text{put}} = -KTe^{-rT}N(-d_2)}$$

**Interpretation:** Rho measures interest-rate sensitivity. Call values increase with $r$ (higher rate reduces the PV of the strike you pay), put values decrease with $r$ (higher rate reduces the PV of the strike you receive). Rho is the least important Greek for short-dated options because $T$ is small.

## Sign Analysis

| Greek | Long Call | Short Call | Long Put | Short Put |
|-------|-----------|------------|----------|-----------|
| $\Delta$ | $+$ (0 to 1) | $-$ ($-1$ to 0) | $-$ ($-1$ to 0) | $+$ (0 to 1) |
| $\Gamma$ | $+$ | $-$ | $+$ | $-$ |
| $\mathcal{V}$ | $+$ | $-$ | $+$ | $-$ |
| $\Theta$ | $-$ (usually) | $+$ (usually) | $-$ (usually) | $+$ (usually) |
| $\rho$ | $+$ | $-$ | $-$ | $+$ |

**Reading the table:** Long options (calls or puts) always have positive Gamma and Vega — you benefit from volatility and large moves. The cost is negative Theta: time works against you. Short options have the mirror image.

## Delta Hedging

### Concept

A delta-neutral portfolio has $\Delta_{\text{portfolio}} = 0$, meaning it is locally insensitive to small moves in the underlying. To hedge a short call position with $\Delta_{\text{call}} = 0.6$:

$$\text{Shares to buy} = \Delta_{\text{call}} \times \text{number of calls sold}$$

If you sold 100 calls, buy 60 shares. The portfolio is now delta-neutral.

### Rebalancing

Delta changes as $S$ moves (because of Gamma), so you must rebalance. After $S$ moves by $\delta S$:

$$\Delta_{\text{new}} \approx \Delta_{\text{old}} + \Gamma \cdot \delta S$$

You trade $\Gamma \cdot \delta S$ additional shares to restore neutrality.

**Rebalancing frequency tradeoff:**

| Rebalance often | Rebalance rarely |
|-----------------|------------------|
| Better hedge (closer to BS assumptions) | Lower transaction costs |
| Higher transaction costs | Larger hedging error |
| P&L closer to theoretical Theta | P&L more volatile |

In the BS world with continuous trading and no costs, continuous rebalancing is optimal. In practice (and in Prosperity), you rebalance discretely — typically whenever $|\Delta_{\text{portfolio}}|$ exceeds a threshold.

### Discrete hedging P&L

Over one rebalancing interval, the P&L of a delta-hedged short call position is approximately:

$$\text{P\&L} \approx -\frac{1}{2}\Gamma(\delta S)^2 + \Theta\,\delta t$$

$$= \frac{1}{2}\Gamma\left[\sigma^2 S^2\,\delta t - (\delta S)^2\right]$$

This is the **Gamma-Theta P&L**: you earn Theta but pay whenever realised moves $|\delta S|$ exceed what the implied volatility predicted ($\sigma S\sqrt{\delta t}$).

## Gamma-Theta Tradeoff

From the BS PDE evaluated at a delta-hedged portfolio:

$$\Theta + \frac{1}{2}\sigma^2 S^2 \Gamma = rV$$

For near-zero rates ($r \approx 0$):

$$\Theta \approx -\frac{1}{2}\sigma^2 S^2 \Gamma$$

This is the **fundamental tradeoff**:

- **Long Gamma ($\Gamma > 0$):** You profit from large moves (your Delta automatically adjusts favourably). The cost is $\Theta < 0$: you lose money every day from time decay.
- **Short Gamma ($\Gamma < 0$):** You collect Theta (time decay) but suffer on large moves because your Delta is working against you.

The breakeven daily move for a delta-hedged position is:

$$|\delta S|_{\text{breakeven}} = \sigma S \sqrt{\delta t}$$

If the stock moves more than this, long Gamma wins. If it moves less, short Gamma (Theta collection) wins.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Spot price | $S$ | Current underlying price | Asset-dependent |
| Strike price | $K$ | Option exercise price | Near $S$ for ATM |
| Time to expiry | $T$ | Years remaining | $0$ to $2+$ |
| Volatility | $\sigma$ | Annualised implied volatility | 0.10–0.80 |
| Risk-free rate | $r$ | Continuous compounding rate | 0.00–0.06 |
| Normal PDF | $n(x)$ | $\frac{1}{\sqrt{2\pi}}e^{-x^2/2}$ | Peaks at 0.3989 |
| Normal CDF | $N(x)$ | $\int_{-\infty}^x n(z)\,dz$ | 0 to 1 |

## Numerical Example

**Setup:** $S = 100$, $K = 100$ (ATM), $T = 0.25$, $r = 0.05$, $\sigma = 0.20$.

**Step 1 — Compute $d_1$, $d_2$:**

$$d_1 = \frac{\ln(1) + (0.05 + 0.02)\times 0.25}{0.20\times 0.5} = \frac{0 + 0.0175}{0.10} = 0.175$$

$$d_2 = 0.175 - 0.10 = 0.075$$

**Step 2 — Normal values:**

$$N(0.175) = 0.5694, \quad N(0.075) = 0.5299$$

$$n(0.175) = \frac{1}{\sqrt{2\pi}}e^{-0.175^2/2} = 0.3932$$

**Step 3 — Greeks for the call:**

| Greek | Formula | Value |
|-------|---------|-------|
| $\Delta_{\text{call}}$ | $N(d_1)$ | $0.5694$ |
| $\Gamma$ | $\frac{n(d_1)}{S\sigma\sqrt{T}} = \frac{0.3932}{100\times 0.20\times 0.50}$ | $0.03932$ |
| $\mathcal{V}$ | $S\,n(d_1)\sqrt{T} = 100\times 0.3932\times 0.50$ | $19.66$ |
| $\Theta_{\text{call}}$ | $-\frac{100\times 0.3932\times 0.20}{2\times 0.50} - 0.05\times 100\times e^{-0.0125}\times 0.5299$ | $-7.864 - 2.616 = -10.48$ per year $= -0.0287$ per day |
| $\rho_{\text{call}}$ | $100\times 0.25\times e^{-0.0125}\times 0.5299$ | $13.09$ |

**Step 4 — Greeks for the put:**

| Greek | Formula | Value |
|-------|---------|-------|
| $\Delta_{\text{put}}$ | $N(d_1) - 1$ | $-0.4306$ |
| $\Gamma$ | Same as call | $0.03932$ |
| $\mathcal{V}$ | Same as call | $19.66$ |
| $\Theta_{\text{put}}$ | $\Theta_{\text{call}} + rKe^{-rT}$ | $-10.48 + 4.938 = -5.54$ per year |
| $\rho_{\text{put}}$ | $-KTe^{-rT}N(-d_2)$ | $-11.66$ |

**Verification with Python:**

```python
import numpy as np
from scipy.stats import norm

S, K, T, r, sigma = 100, 100, 0.25, 0.05, 0.20

d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
d2 = d1 - sigma*np.sqrt(T)

delta_call = norm.cdf(d1)
delta_put  = norm.cdf(d1) - 1
gamma      = norm.pdf(d1) / (S * sigma * np.sqrt(T))
vega       = S * norm.pdf(d1) * np.sqrt(T)
theta_call = -S*norm.pdf(d1)*sigma/(2*np.sqrt(T)) - r*K*np.exp(-r*T)*norm.cdf(d2)
theta_put  = theta_call + r*K*np.exp(-r*T)
rho_call   = K*T*np.exp(-r*T)*norm.cdf(d2)
rho_put    = -K*T*np.exp(-r*T)*norm.cdf(-d2)

print(f"d1 = {d1:.4f}, d2 = {d2:.4f}")
print(f"Delta call = {delta_call:.4f}, put = {delta_put:.4f}")
print(f"Gamma      = {gamma:.5f}")
print(f"Vega       = {vega:.2f}")
print(f"Theta call = {theta_call:.2f}/yr ({theta_call/365:.4f}/day)")
print(f"Theta put  = {theta_put:.2f}/yr")
print(f"Rho   call = {rho_call:.2f}, put = {rho_put:.2f}")
```

## Connections

- [Black–Scholes](black_scholes.md) — The Greeks are all partial derivatives of the BS formula. The BS PDE itself encodes the Gamma-Theta-Rho relationship: $\Theta + \frac{1}{2}\sigma^2 S^2\Gamma + rS\Delta = rV$.
- [Implied volatility](implied_volatility.md) — Vega is the denominator in the Newton-Raphson iteration for implied vol; it measures how sensitive the BS price is to $\sigma$, which determines how quickly the root-finder converges.
- [Avellaneda–Stoikov](../03_market_making/avellaneda_stoikov.md) — When market-making options, your optimal quotes depend on your Greek exposure. The inventory risk parameter $\gamma$ in AS should be calibrated to your net Gamma and Vega risk.
- [Jump diffusion](jump_diffusion.md) — Under jump-diffusion, the Greeks change: Gamma spikes near the strike due to the possibility of jumps landing just ITM/OTM, and Vega includes sensitivity to both diffusive and jump volatility.

## Relevance for Prosperity 4

In Prosperity 4, the Greeks are your primary risk-management toolkit:

- **Delta hedging:** After trading options, immediately compute your portfolio Delta and hedge with the underlying. In a fast-moving market, prioritise getting Delta close to zero before worrying about other Greeks. The formula $\Delta_{\text{call}} = N(d_1)$ is computationally cheap and should be evaluated on every tick.
- **Gamma scalping:** If you accumulate long Gamma positions (by buying options cheaply), you profit by delta-hedging frequently — each rebalance locks in a small gain when the stock moves. The total profit is approximately $\frac{1}{2}\Gamma(\sigma_{\text{realised}}^2 - \sigma_{\text{implied}}^2)S^2 T$, so you win if realised vol exceeds implied vol.
- **Theta awareness:** In short rounds, Theta can be significant for ATM options. If you're long options approaching expiry, the time decay accelerates. Plan to exit or hedge before the final minutes.
- **Vega for mispricing:** If you believe the market over-estimates or under-estimates future volatility, take a Vega position. Long Vega (buy options) if you think vol will rise; short Vega (sell options) if you think vol will fall. Your edge depends on your volatility forecast being better than the market's.
