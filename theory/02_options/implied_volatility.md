# Implied Volatility

> **Core formula:**
> $$\sigma_{\text{impl}} = \text{BS}^{-1}(V_{\text{market}}) \quad\Longleftrightarrow\quad C_{\text{BS}}(S, K, T, r, \sigma_{\text{impl}}) = C_{\text{market}}$$

## Intuition

If the Black–Scholes model were perfectly correct, every option on the same underlying would produce the same implied volatility regardless of strike or expiry. In practice, they don't — and the pattern of implied vols across strikes and maturities contains a wealth of information about market expectations and risk preferences.

Implied volatility is the market's "consensus forecast" of future uncertainty, embedded in the price of an option. Unlike historical (realised) volatility, which looks backward at what the stock actually did, implied volatility looks forward at what the market thinks the stock might do. When traders say "vol is 25%," they typically mean implied vol. It has become the universal language for quoting option prices — saying "the 100-strike put trades at \$5.20" is less informative than saying "it trades at 22 vol," because the latter immediately conveys whether the option is cheap or expensive relative to its risk.

The procedure is straightforward in concept: take the observed market price of an option, plug it into the BS formula alongside all the known inputs ($S$, $K$, $T$, $r$), and solve for the one unknown: $\sigma$. Since the BS formula has no closed-form inverse in $\sigma$, this requires a numerical root-finding algorithm — most commonly Newton-Raphson, which converges very quickly because the BS price is smooth and monotone in $\sigma$.

## Mathematical Setup

### The inversion problem

Given a market-observed call price $C_{\text{market}}$, find $\sigma_{\text{impl}}$ such that:

$$f(\sigma) \equiv C_{\text{BS}}(S, K, T, r, \sigma) - C_{\text{market}} = 0$$

where:

$$C_{\text{BS}} = S\,N(d_1) - Ke^{-rT}N(d_2)$$

$$d_1 = \frac{\ln(S/K) + (r + \sigma^2/2)T}{\sigma\sqrt{T}}, \qquad d_2 = d_1 - \sigma\sqrt{T}$$

### Existence and uniqueness

**Monotonicity:** Vega $= \partial C_{\text{BS}}/\partial\sigma = S\,n(d_1)\sqrt{T} > 0$ for all $\sigma > 0$. The BS price is strictly increasing in $\sigma$.

**Limits:**

$$\lim_{\sigma \to 0^+} C_{\text{BS}} = \max(S - Ke^{-rT}, 0) \quad\text{(intrinsic value)}$$

$$\lim_{\sigma \to \infty} C_{\text{BS}} = S \quad\text{(the call approaches the stock price)}$$

Therefore, for any market price $C_{\text{market}}$ satisfying:

$$\max(S - Ke^{-rT}, 0) < C_{\text{market}} < S$$

there exists a **unique** $\sigma_{\text{impl}} > 0$ solving the equation. If $C_{\text{market}}$ falls outside this range, the price violates no-arbitrage bounds and no implied vol exists.

### Symbols

| Symbol | Meaning |
|--------|---------|
| $\sigma_{\text{impl}}$ | Implied volatility |
| $\sigma_{\text{realised}}$ | Realised (historical) volatility, computed from past returns |
| $C_{\text{market}}$ | Market-observed option price |
| $C_{\text{BS}}(\sigma)$ | BS theoretical price as a function of $\sigma$ |
| $\mathcal{V}(\sigma)$ | Vega: $\partial C_{\text{BS}}/\partial\sigma = S\,n(d_1)\sqrt{T}$ |

## Derivation

### Newton-Raphson algorithm

Newton-Raphson finds the root of $f(\sigma) = C_{\text{BS}}(\sigma) - C_{\text{market}} = 0$ by iterating:

$$\sigma_{n+1} = \sigma_n - \frac{f(\sigma_n)}{f'(\sigma_n)}$$

Since $f'(\sigma) = \partial C_{\text{BS}}/\partial\sigma = \mathcal{V}(\sigma)$ (Vega), this becomes:

$$\boxed{\sigma_{n+1} = \sigma_n - \frac{C_{\text{BS}}(\sigma_n) - C_{\text{market}}}{\mathcal{V}(\sigma_n)}}$$

**Why Vega as the denominator:** Newton-Raphson requires the Jacobian (first derivative) of the function whose root we seek. Since we are inverting $C_{\text{BS}}(\sigma)$, the Jacobian is $\partial C_{\text{BS}}/\partial\sigma = \mathcal{V}$. This is the natural step size: if the price is too high by $\delta C$, we adjust $\sigma$ by $\delta\sigma \approx \delta C / \mathcal{V}$.

**Convergence criterion:**

$$|C_{\text{BS}}(\sigma_n) - C_{\text{market}}| < \varepsilon$$

where $\varepsilon$ is a small tolerance (e.g., $\varepsilon = 10^{-8}$). Equivalently, one can check $|\sigma_{n+1} - \sigma_n| < \varepsilon_\sigma$ for a volatility tolerance (e.g., $10^{-6}$).

**Starting value:** A reasonable initial guess is $\sigma_0 = 0.20$ (typical equity vol) or, for a better approximation, use the Brenner–Subrahmanyam ATM formula:

$$\sigma_0 \approx \frac{C_{\text{market}}}{S}\sqrt{\frac{2\pi}{T}}$$

**Convergence speed:** Newton-Raphson has **quadratic convergence** for this problem (because Vega is smooth and positive near the solution). Typically 3–6 iterations suffice for machine precision.

### Potential issues and safeguards

1. **Vega near zero:** For deep in-the-money or deep out-of-the-money options, Vega is very small, making the Newton step $\delta\sigma = (C_{\text{BS}} - C_{\text{market}})/\mathcal{V}$ enormous. Safeguard: clamp the step size, e.g., $|\delta\sigma| \le 1.0$.

2. **Negative implied vol:** If the iteration produces $\sigma_n < 0$, reset to a small positive value (e.g., $\sigma_n = 0.001$).

3. **Non-convergence:** If the market price violates no-arbitrage bounds ($C_{\text{market}} \le 0$ or $C_{\text{market}} \ge S$), no solution exists. Check bounds before iterating.

4. **Near-zero time to expiry:** As $T \to 0$, the BS function becomes discontinuous at $\sigma = 0$. Use a minimum $T$ floor (e.g., $T \ge 1/365$) or switch to intrinsic-value pricing.

5. **Alternative methods:** For extreme strikes, Brent's method (bisection + secant hybrid) is more robust than Newton-Raphson because it doesn't require Vega and is guaranteed to converge within the bracketing interval $[\sigma_{\min}, \sigma_{\max}]$.

### Second-order correction (Halley's method)

For even faster convergence, use Halley's method which includes the second derivative (Volga = $\partial\mathcal{V}/\partial\sigma$):

$$\sigma_{n+1} = \sigma_n - \frac{f(\sigma_n)}{f'(\sigma_n)}\cdot\frac{1}{1 - \frac{f(\sigma_n)\,f''(\sigma_n)}{2\,[f'(\sigma_n)]^2}}$$

where $f''(\sigma) = \mathcal{V}(\sigma)\cdot d_1 d_2/\sigma$ (the "Volga" or "Vomma"). This converges cubically but is rarely needed in practice since Newton-Raphson is already fast enough.

## Volatility smile, skew, and surface

### Volatility smile

For a fixed expiry $T$, plot $\sigma_{\text{impl}}(K)$ as a function of strike $K$. In a pure BS world, this would be a flat horizontal line. In reality:

- **Equity indices:** The curve is a **skew** — implied vol decreases as $K$ increases. OTM puts ($K < S$) have higher implied vol than ATM options, which in turn have higher implied vol than OTM calls ($K > S$). This reflects the market's fear of crashes (demand for downside protection pushes up put prices).

- **Foreign exchange:** The curve is a **smile** — implied vol is higher for both low and high strikes relative to ATM. This reflects the symmetric uncertainty in exchange rates (large moves in either direction are more likely than GBM predicts).

- **Commodities:** Often exhibit a smile or a "smirk" depending on supply/demand dynamics.

### Volatility skew

The systematic tilt of the smile is called the **skew**. It is typically measured as:

$$\text{Skew} = \sigma_{\text{impl}}(K_{\text{25-delta put}}) - \sigma_{\text{impl}}(K_{\text{25-delta call}})$$

For equity indices, this is typically 3–8 vol points. A steeper skew means the market is pricing more crash risk.

**Why the skew exists:**

1. **Crash risk:** Large downward moves occur more frequently than GBM predicts (fat left tail). The leverage effect amplifies this: when stocks fall, companies become more levered, increasing volatility.
2. **Supply/demand:** Portfolio insurers systematically buy OTM puts, driving up their prices and hence their implied vol.
3. **Jump risk:** As shown in [jump diffusion](jump_diffusion.md), adding downward jumps ($\mu_J < 0$) to GBM generates exactly this skew pattern.

### Volatility surface

The full volatility surface $\sigma_{\text{impl}}(K, T)$ is a function of both strike and expiry. Key features:

- **Strike dimension:** Smile/skew as described above.
- **Expiry dimension:** The **term structure** of ATM implied vol. Typically upward-sloping in calm markets (longer-dated options are more uncertain) and inverted in crisis periods (near-term uncertainty dominates).
- **Smile flattening:** The smile is steepest for short-dated options and flattens with increasing maturity. This is because jumps affect short-dated options most, and over long horizons the central limit theorem makes the distribution more Gaussian.

The surface must satisfy no-arbitrage constraints:

1. **No butterfly arbitrage:** $\partial^2 C/\partial K^2 \ge 0$ (the density must be non-negative).
2. **No calendar arbitrage:** Total implied variance $\sigma_{\text{impl}}^2 T$ must be non-decreasing in $T$.

## Implied vs. realised volatility

| | Implied volatility | Realised volatility |
|---|---|---|
| **Direction** | Forward-looking | Backward-looking |
| **Source** | Option market prices | Historical price data |
| **Includes** | Risk premium, supply/demand effects | Only past price moves |
| **Typical relation** | $\sigma_{\text{impl}} > \sigma_{\text{realised}}$ on average | — |

The difference $\sigma_{\text{impl}} - \sigma_{\text{realised}}$ is called the **variance risk premium** (VRP). On average, implied vol exceeds realised vol by 2–4 percentage points for equity indices. This means option sellers earn a systematic premium for bearing volatility risk — analogous to the equity risk premium for bearing stock-market risk.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Implied vol | $\sigma_{\text{impl}}$ | The BS-inversion output | 0.05–1.00+ |
| Tolerance | $\varepsilon$ | Convergence threshold for price match | $10^{-8}$ to $10^{-6}$ |
| Initial guess | $\sigma_0$ | Starting point for Newton-Raphson | 0.15–0.30 |
| Max iterations | $N_{\max}$ | Safety limit to prevent infinite loops | 50–100 |
| Step clamp | $\Delta\sigma_{\max}$ | Maximum allowed $|\delta\sigma|$ per iteration | 0.5–2.0 |
| Vega floor | $\mathcal{V}_{\min}$ | Minimum Vega to avoid division by near-zero | $10^{-10}$ |

## Numerical Example

**Setup:** $S = 100$, $K = 100$, $T = 0.25$, $r = 0.05$. Suppose the market call price is $C_{\text{market}} = 5.50$.

**Goal:** Find $\sigma_{\text{impl}}$ such that $C_{\text{BS}}(100, 100, 0.25, 0.05, \sigma_{\text{impl}}) = 5.50$.

**Iteration 0:** Start with $\sigma_0 = 0.20$.

$$d_1 = \frac{\ln(1) + (0.05 + 0.02)\times 0.25}{0.20 \times 0.5} = \frac{0.0175}{0.10} = 0.175$$

$$d_2 = 0.175 - 0.10 = 0.075$$

$$C_{\text{BS}}(0.20) = 100\times N(0.175) - 100\times e^{-0.0125}\times N(0.075)$$

$$= 100\times 0.5694 - 98.76\times 0.5299 = 56.94 - 52.33 = 4.61$$

$$\mathcal{V}(0.20) = 100\times n(0.175)\times 0.5 = 100\times 0.3932\times 0.5 = 19.66$$

$$\sigma_1 = 0.20 - \frac{4.61 - 5.50}{19.66} = 0.20 + 0.0453 = 0.2453$$

**Iteration 1:** $\sigma_1 = 0.2453$.

$$d_1 = \frac{0 + (0.05 + 0.5\times 0.2453^2)\times 0.25}{0.2453\times 0.5} = \frac{0.0175 + 0.00752}{0.12265} = \frac{0.02002}{0.12265} = 0.2041$$

$$d_2 = 0.2041 - 0.12265 = 0.08145$$

$$C_{\text{BS}}(0.2453) = 100\times N(0.2041) - 98.76\times N(0.08145)$$

$$= 100\times 0.5809 - 98.76\times 0.5325 = 58.09 - 52.57 = 5.52$$

$$\mathcal{V}(0.2453) = 100\times n(0.2041)\times 0.5 = 100\times 0.3910\times 0.5 = 19.55$$

$$\sigma_2 = 0.2453 - \frac{5.52 - 5.50}{19.55} = 0.2453 - 0.0010 = 0.2443$$

**Iteration 2:** $\sigma_2 = 0.2443$. At this point, $|C_{\text{BS}}(\sigma_2) - 5.50| < 0.001$, so we have converged.

**Result:** $\sigma_{\text{impl}} \approx 0.2443$ (24.43%).

**Verification:** The BS call at $\sigma = 0.2443$ gives $C \approx 5.500$. $\checkmark$

```python
import numpy as np
from scipy.stats import norm

def bs_call(S, K, T, r, sigma):
    d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
    d2 = d1 - sigma*np.sqrt(T)
    return S*norm.cdf(d1) - K*np.exp(-r*T)*norm.cdf(d2)

def bs_vega(S, K, T, r, sigma):
    d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
    return S * norm.pdf(d1) * np.sqrt(T)

def implied_vol_newton(S, K, T, r, C_market, sigma0=0.20, tol=1e-8, max_iter=100):
    """Newton-Raphson implied volatility solver."""
    intrinsic = max(S - K*np.exp(-r*T), 0.0)
    if C_market <= intrinsic or C_market >= S:
        raise ValueError(f"Price {C_market:.4f} outside no-arb bounds [{intrinsic:.4f}, {S:.2f}]")

    sigma = sigma0
    for i in range(max_iter):
        price = bs_call(S, K, T, r, sigma)
        diff = price - C_market
        if abs(diff) < tol:
            return sigma
        vega = bs_vega(S, K, T, r, sigma)
        if vega < 1e-12:
            vega = 1e-12
        step = diff / vega
        step = np.clip(step, -1.0, 1.0)
        sigma -= step
        sigma = max(sigma, 1e-6)
    raise RuntimeError(f"Did not converge after {max_iter} iterations (sigma={sigma:.6f})")

S, K, T, r = 100, 100, 0.25, 0.05
C_market = 5.50

iv = implied_vol_newton(S, K, T, r, C_market)
print(f"Implied vol = {iv:.4f} ({iv*100:.2f}%)")
print(f"Verification: BS({iv:.4f}) = {bs_call(S, K, T, r, iv):.4f}, target = {C_market}")
```

## Connections

- [Black–Scholes](black_scholes.md) — Implied volatility is defined as the inverse of the BS pricing function. The BS formula provides the forward map $\sigma \mapsto C$; implied vol is the backward map $C \mapsto \sigma$.
- [Greeks](greeks.md) — Vega ($\partial C/\partial\sigma$) is the key ingredient in Newton-Raphson for implied vol. When Vega is large (ATM, long-dated), the solver converges fastest. When Vega is small (deep OTM/ITM), convergence is fragile.
- [Jump diffusion](jump_diffusion.md) — The Merton model generates a volatility smile/skew when its prices are re-expressed in BS implied vol. Specifically, downward jumps ($\mu_J < 0$) produce an equity-like skew where OTM put IV exceeds OTM call IV.
- [Ornstein–Uhlenbeck](../04_stat_arb/ornstein_uhlenbeck.md) — Implied volatility for pairs of cointegrated assets can be used to detect regime changes: a sudden spike in IV signals increased uncertainty about the spread, which may warrant wider trading bands.
- [Avellaneda–Stoikov](../03_market_making/avellaneda_stoikov.md) — When market-making options, the $\sigma$ input to the AS model should be the *implied* volatility, not historical volatility, because implied vol is the market's consensus and determines your hedging costs.

## Relevance for Prosperity 4

Implied volatility is one of the most actionable signals in Prosperity 4:

- **Real-time vol estimation:** Compute implied vol from every observed option trade on the order book. This gives you a continuously updated view of the market's volatility expectation, which is more informative than historical vol alone (especially in a short competition where history is limited).
- **Detecting mispricing:** If two options on the same underlying with different strikes imply very different $\sigma_{\text{impl}}$, one of them is likely mispriced. Buy the cheap-vol option and sell the expensive-vol option, then delta-hedge the package. Your profit depends on the vol spread narrowing.
- **Vol surface arbitrage:** If the competition has multiple expiries, check for calendar arbitrage: total implied variance $\sigma^2 T$ must be increasing in $T$. Violations are free money.
- **The Newton-Raphson solver** should be part of your core trading infrastructure. Pre-compile it (e.g., with Numba) so that you can compute implied vol in microseconds for every tick update. The solver above (3–6 iterations of simple arithmetic) is fast enough for real-time use even without compilation.
- **Vol as a quoting parameter:** Instead of quoting options in price space, think in vol space. Convert your fair-value estimate to an implied vol, add/subtract a vol spread (e.g., bid at fair vol $-$ 1%, offer at fair vol $+$ 1%), and convert back to prices. This ensures your spread is consistent across strikes.
