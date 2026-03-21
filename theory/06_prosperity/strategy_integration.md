# Strategy Integration

> **Core formula:** $$r_t^{\text{mod}} = S_t - q_t\,\gamma\,\sigma^2(T-t) + \alpha\,z_t\,\sigma$$

## Intuition

A pure market maker quotes symmetrically around the mid-price, collects the bid-ask spread, and tries to keep inventory near zero. A pure stat-arb trader identifies mispricings — assets that are cheap or expensive relative to some equilibrium — and takes directional bets. Each strategy alone leaves money on the table: the market maker ignores predictable price moves, while the stat-arb trader pays the spread on every entry and exit. The idea behind strategy integration is to combine both into a single quoting engine so that the market-making quotes themselves become the vehicle for expressing the directional view.

Think of it like a shopkeeper who adjusts prices based on weather forecasts. On a day when demand will be high (the stat-arb signal says "this asset is cheap and will revert up"), the shopkeeper raises the price at which they are willing to sell and lowers the price at which they are willing to buy — effectively accumulating inventory before the expected price increase. On a day when demand will be low, the shopkeeper does the reverse. The shopkeeper still maintains a spread and still earns the bid-ask margin, but the asymmetry in their quotes is informed by a forecast.

In the Prosperity competition, this combination is particularly powerful. You face position limits, transaction costs, and adversarial bots. A unified system lets you earn spread revenue during calm periods, exploit mispricings when signals fire, and manage inventory risk continuously — all through a single set of bid and ask quotes updated every tick.

## Mathematical Setup

We build on two foundations and fuse them into one decision framework.

**From Avellaneda-Stoikov market making:**

- Mid-price follows arithmetic Brownian motion: $dS_t = \sigma\,dW_t$
- Inventory: $q_t \in \mathbb{Z}$ (units held, target is zero in pure AS)
- Fill intensity at distance $\delta$ from mid: $\lambda(\delta) = A\,e^{-\kappa\,\delta}$
- Reservation price: $r_t = S_t - q_t\,\gamma\,\sigma^2(T - t)$
- Optimal half-spread: $\delta^* = \gamma\,\sigma^2(T - t) + \frac{2}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right)$

**From statistical arbitrage (z-score signals):**

- Spread between two cointegrated assets: $s_t = Y_t - \beta\,X_t$
- Spread follows an OU process: $ds_t = \theta(\mu_s - s_t)\,dt + \sigma_s\,dW_t^s$
- Z-score: $z_t = \frac{s_t - \mu_s}{\sigma_s}$
- Signal interpretation: $z_t < 0$ means asset is cheap (below equilibrium); $z_t > 0$ means expensive

**Assumptions for the combined system:**

1. The mid-price $S_t$ has a predictable component captured by $z_t$ superimposed on random noise.
2. The z-score is a sufficient statistic for short-term directional forecasts.
3. The fill intensity model $\lambda(\delta) = A\,e^{-\kappa\,\delta}$ remains valid for the modified quotes.
4. All parameters ($\gamma$, $\sigma$, $\kappa$, $\alpha$, $\beta$) are re-estimated on a rolling window every $N$ ticks.

## Derivation

### Step 1: Modify the Reservation Price

The standard AS reservation price penalises inventory linearly:

$$r_t = S_t - q_t\,\gamma\,\sigma^2(T - t)$$

We introduce a directional signal by adding a term proportional to the z-score. The modified reservation price is:

$$r_t^{\text{mod}} = S_t - q_t\,\gamma\,\sigma^2(T - t) + \alpha\,z_t\,\sigma$$

where $\alpha > 0$ is a dimensionless signal-strength parameter. The term $\alpha\,z_t\,\sigma$ has units of price (z-score is dimensionless, $\sigma$ has units of price per $\sqrt{\text{time}}$, and we absorb the time-scale into $\alpha$).

**Interpretation:**

- When $z_t < 0$ (asset is cheap), the signal term $\alpha\,z_t\,\sigma < 0$... wait, we want $r$ to increase when the asset is cheap. Let's be precise. If $z_t < 0$ means the spread $s_t$ is below its mean, and we expect it to revert up, then we want to buy. To buy more aggressively, we raise our reservation price (we are willing to pay more). So the sign convention is:

$$r_t^{\text{mod}} = S_t - q_t\,\gamma\,\sigma^2(T - t) + \alpha\,(-z_t)\,\sigma$$

Equivalently, defining the signal as $\phi_t = -z_t$ (positive when we want to go long):

$$r_t^{\text{mod}} = S_t - q_t\,\gamma\,\sigma^2(T - t) + \alpha\,\phi_t\,\sigma$$

For consistency with the convention that $z_t < 0 \Rightarrow$ go long, we write:

$$\boxed{r_t^{\text{mod}} = S_t - q_t\,\gamma\,\sigma^2(T - t) - \alpha\,z_t\,\sigma}$$

When $z_t = -2.5$: the adjustment is $-\alpha \cdot (-2.5) \cdot \sigma = +2.5\,\alpha\,\sigma > 0$, so $r$ increases — we bid more aggressively. Correct.

### Step 2: Compute the Optimal Half-Spread

The AS optimal half-spread does not depend on the reservation price — it depends only on the risk-aversion $\gamma$, volatility $\sigma$, time horizon $T - t$, and order-book depth $\kappa$:

$$\delta^* = \gamma\,\sigma^2(T - t) + \frac{2}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right)$$

This formula remains unchanged in the integrated system. The signal affects *where* we centre our quotes (via $r^{\text{mod}}$), not how wide we quote.

### Step 3: Set Bid and Ask Prices

The raw bid and ask are:

$$p^{\text{bid}} = r_t^{\text{mod}} - \delta^*$$

$$p^{\text{ask}} = r_t^{\text{mod}} + \delta^*$$

### Step 4: Add Directional Skew to the Spread

Beyond shifting the centre via $r^{\text{mod}}$, we can also *asymmetrically* adjust the offsets. Define a skew parameter:

$$\text{skew}_t = \beta_{\text{skew}}\,z_t$$

where $\beta_{\text{skew}} > 0$ controls sensitivity. The skewed quotes become:

$$p^{\text{bid}} = r_t^{\text{mod}} - \delta^* + \text{skew}_t$$

$$p^{\text{ask}} = r_t^{\text{mod}} + \delta^* + \text{skew}_t$$

Equivalently, define directional offsets:

$$\text{bid\_offset} = \delta^* - \text{skew}_t$$

$$\text{ask\_offset} = \delta^* + \text{skew}_t$$

**When the signal says "go long"** ($z_t < 0$, so $\text{skew}_t < 0$):

- $\text{bid\_offset} = \delta^* - \text{skew}_t = \delta^* + |\text{skew}_t|$ — but this *widens* the bid, which is the opposite of what we want. Let's fix the sign.

We want: signal says "go long" → tighten bid (closer to mid, easier to get filled buying), widen ask (further from mid, harder to get filled selling). So:

$$\text{bid\_offset} = \delta^* + \text{skew}_t$$

$$\text{ask\_offset} = \delta^* - \text{skew}_t$$

Now when $z_t < 0$ (go long), $\text{skew}_t = \beta_{\text{skew}} \cdot z_t < 0$:

- $\text{bid\_offset} = \delta^* + (\text{negative}) = \delta^* - |\text{skew}|$ → tighter bid ✓
- $\text{ask\_offset} = \delta^* - (\text{negative}) = \delta^* + |\text{skew}|$ → wider ask ✓

The final quote prices:

$$\boxed{p^{\text{bid}} = r_t^{\text{mod}} - (\delta^* + \text{skew}_t)}$$

$$\boxed{p^{\text{ask}} = r_t^{\text{mod}} + (\delta^* - \text{skew}_t)}$$

where $\text{skew}_t = \beta_{\text{skew}}\,z_t$.

### Step 5: When to Widen the Spread

The base half-spread $\delta^*$ is the theoretical optimum. In practice, we apply multipliers:

**High inventory.** When $|q_t|$ exceeds a threshold $q_{\max}$, we widen to slow accumulation in the dangerous direction:

$$\delta^*_{\text{adj}} = \delta^* \cdot \left(1 + \lambda_q \cdot \max\!\left(0,\;\frac{|q_t| - q_{\text{safe}}}{q_{\max} - q_{\text{safe}}}\right)\right)$$

where $\lambda_q \in [0.5, 2.0]$ controls how aggressively we widen, and $q_{\text{safe}}$ is the inventory level below which no widening is applied.

**High volatility.** When estimated $\hat{\sigma}$ is elevated relative to its rolling average:

$$\delta^*_{\text{adj}} = \delta^* \cdot \left(1 + \lambda_\sigma \cdot \max\!\left(0,\;\frac{\hat{\sigma} - \bar{\sigma}}{\bar{\sigma}}\right)\right)$$

**Low signal confidence.** When $|z_t|$ is small (near zero), there is no directional conviction. We widen to earn pure spread:

$$\delta^*_{\text{adj}} = \delta^* \cdot \left(1 + \lambda_z \cdot \mathbb{1}_{|z_t| < z_{\text{thresh}}}\right)$$

All three multipliers can be combined multiplicatively.

### Step 6: Risk Budget Allocation Across Instruments

In Prosperity, you trade $n$ products simultaneously. Define the total risk budget constraint:

$$\sum_{i=1}^{n} \gamma_i \cdot \text{Var}(q_i \cdot \Delta S_i) \leq R_{\text{total}}$$

Since $\text{Var}(q_i \cdot \Delta S_i) = q_i^2\,\sigma_i^2\,\Delta t$:

$$\sum_{i=1}^{n} \gamma_i\,q_i^2\,\sigma_i^2\,\Delta t \leq R_{\text{total}}$$

**Priority ranking for risk allocation.** Assign more risk budget (lower $\gamma_i$, meaning looser inventory control and larger positions) to instruments with:

1. Higher spread opportunity: $\frac{\text{quoted spread}_i}{\sigma_i}$ is large (high Sharpe from market making)
2. Lower adverse selection: fills are mostly noise traders, not informed flow
3. Stronger mean-reversion signals: $|z_{i,t}| > 2$ and the OU half-life is short

**Correlation handling:**

- If instruments $i$ and $j$ are cointegrated, trade the spread $s_t = S_i - \beta_{ij}\,S_j$ as a single virtual instrument. The risk budget for the spread replaces the individual budgets.
- If uncorrelated, treat independently: $\gamma_i$ and $\gamma_j$ are set in isolation.
- If positively correlated but not cointegrated, reduce total exposure: $q_i$ and $q_j$ should not both be large and same-signed.

**Dynamic reallocation:**

- Increase $\gamma_i$ (tighten risk) when instrument $i$ shows signs of regime change: sudden volatility spike, breakdown of cointegration, or anomalous order flow.
- Decrease $\gamma_i$ (loosen risk) on instruments that remain stable, mean-reverting, and liquid.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Risk aversion | $\gamma$ | Controls inventory penalty; higher $\gamma$ = faster mean-reversion of $q$ toward 0 | $0.01$ – $1.0$ |
| Volatility | $\sigma$ | Mid-price volatility per $\sqrt{\text{tick}}$; estimated from recent returns | Asset-dependent |
| Order-book depth | $\kappa$ | Decay rate of fill probability with distance from mid | $0.5$ – $5.0$ |
| Signal strength | $\alpha$ | Weight of the z-score signal in the reservation price | $0.0$ – $1.0$ |
| Skew sensitivity | $\beta_{\text{skew}}$ | How much to asymmetrically shift bid/ask offsets per unit z-score | $0.0$ – $0.5\,\sigma$ |
| Z-score threshold | $z_{\text{thresh}}$ | Below this $|z|$, no directional signal; widen spread and collect | $0.5$ – $1.0$ |
| Inventory widening | $\lambda_q$ | Multiplier for spread widening at high inventory | $0.5$ – $2.0$ |
| Safe inventory | $q_{\text{safe}}$ | Inventory level below which no widening is applied | $0$ – $0.5\,q_{\max}$ |
| Volatility widening | $\lambda_\sigma$ | Multiplier for spread widening when $\hat{\sigma} > \bar{\sigma}$ | $0.5$ – $2.0$ |
| Total risk budget | $R_{\text{total}}$ | Maximum acceptable portfolio variance per tick | Strategy-dependent |

## Numerical Example

**Setup.** A single product trades at $S = 100$, with estimated volatility $\sigma = 0.5$ per $\sqrt{\text{tick}}$, risk aversion $\gamma = 0.1$, order-book depth $\kappa = 1.5$, time remaining $T - t = 100$ ticks. The current z-score from a cointegration signal is $z_t = -2.5$. Our signal strength is $\alpha = 0.3$ and skew sensitivity $\beta_{\text{skew}} = 0.1$. Current inventory $q = 2$.

**Step 1 — Reservation price:**

$$r^{\text{mod}} = S - q\,\gamma\,\sigma^2(T-t) - \alpha\,z_t\,\sigma$$

$$r^{\text{mod}} = 100 - 2 \cdot 0.1 \cdot 0.25 \cdot 100 - 0.3 \cdot (-2.5) \cdot 0.5$$

$$r^{\text{mod}} = 100 - 5.0 + 0.375 = 95.375$$

The inventory penalty pulls $r$ down by $5.0$ (we are long 2 units and want to reduce). The signal pushes $r$ up by $0.375$ (z-score says the asset is cheap, so we tolerate being long a bit more).

**Step 2 — Optimal half-spread:**

$$\delta^* = \gamma\,\sigma^2(T-t) + \frac{2}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right)$$

$$\delta^* = 0.1 \cdot 0.25 \cdot 100 + \frac{2}{0.1}\ln\!\left(1 + \frac{0.1}{1.5}\right)$$

$$\delta^* = 2.5 + 20 \cdot \ln(1.0\overline{6})$$

$$\delta^* = 2.5 + 20 \cdot 0.06454 = 2.5 + 1.291 = 3.791$$

**Step 3 — Skew:**

$$\text{skew}_t = \beta_{\text{skew}} \cdot z_t = 0.1 \cdot (-2.5) = -0.25$$

**Step 4 — Quote prices:**

$$p^{\text{bid}} = r^{\text{mod}} - (\delta^* + \text{skew}_t) = 95.375 - (3.791 + (-0.25)) = 95.375 - 3.541 = 91.834$$

$$p^{\text{ask}} = r^{\text{mod}} + (\delta^* - \text{skew}_t) = 95.375 + (3.791 - (-0.25)) = 95.375 + 4.041 = 99.416$$

**Interpretation.** The bid at $91.83$ is tighter relative to $r^{\text{mod}}$ than the ask at $99.42$ — we are making it easier for counterparties to sell to us (we want to buy, because the signal says cheap). The total quoted spread is $99.416 - 91.834 = 7.582$, which is approximately $2\delta^* = 7.582$. The spread width is preserved; only the *centering* and *asymmetry* changed.

**Economic logic:** We are already long 2 units, which the standard AS model penalises heavily ($-5.0$). But the mean-reversion signal says the asset is $2.5\sigma$ below fair value, so we partially offset the inventory penalty. The net effect: we still want to reduce inventory, but less urgently than pure AS would suggest.

```python
import numpy as np

S, sigma, gamma, kappa = 100.0, 0.5, 0.1, 1.5
T_minus_t, q, z_t = 100.0, 2.0, -2.5
alpha, beta_skew = 0.3, 0.1

r_mod = S - q * gamma * sigma**2 * T_minus_t - alpha * z_t * sigma
delta_star = gamma * sigma**2 * T_minus_t + (2 / gamma) * np.log(1 + gamma / kappa)
skew = beta_skew * z_t

bid = r_mod - (delta_star + skew)
ask = r_mod + (delta_star - skew)

print(f"Reservation price (modified): {r_mod:.3f}")
print(f"Optimal half-spread:          {delta_star:.3f}")
print(f"Skew:                         {skew:.3f}")
print(f"Bid:                          {bid:.3f}")
print(f"Ask:                          {ask:.3f}")
print(f"Total spread:                 {ask - bid:.3f}")
```

## Connections

- [Avellaneda-Stoikov](../03_market_making/avellaneda_stoikov.md) — provides the base reservation price and optimal spread that this file modifies with directional signals.
- [Inventory Risk](../03_market_making/inventory_risk.md) — the $q\,\gamma\,\sigma^2(T-t)$ penalty term and the spread-widening logic at high inventory both derive from inventory risk management.
- [Bid-Ask Spread](../03_market_making/bid_ask_spread.md) — the mechanics of how spread width affects fill rates and adverse selection.
- [Z-Score Signals](../04_stat_arb/zscore_signals.md) — the $z_t$ signal that drives the directional overlay comes from the z-score framework on cointegrated spreads.
- [Ornstein-Uhlenbeck](../04_stat_arb/ornstein_uhlenbeck.md) — the mean-reversion dynamics that justify why $z_t \neq 0$ is a tradeable signal.
- [Cointegration](../04_stat_arb/cointegration.md) — how to identify cointegrated pairs and estimate the hedge ratio $\beta$ used to construct the spread.
- [Nash Equilibrium](../05_game_theory/nash_equilibrium.md) — understanding opponents' strategies (likely also AS-based) informs our parameter choices, especially $\gamma$ and $\kappa$.
- [Prosperity 3 Analysis](./prosperity3_analysis.md) — empirical evidence from the previous competition that this combined approach works.

## Relevance for Prosperity 4

In IMC Prosperity 4 you submit a single `Trader` class that quotes on multiple products each tick. The strategy integration framework maps directly to this structure: at each tick, (1) estimate $\sigma_i$ from the last 50–100 mid-price returns for each product $i$, (2) compute $z_{i,t}$ from any identified cointegration relationships between products, (3) compute $r_i^{\text{mod}}$ and $\delta_i^*$ per the formulas above, (4) apply skew and widening adjustments, and (5) submit bid/ask orders.

The key Prosperity-specific considerations are:

- **Position limits** act as hard caps on $|q_i|$. When approaching the limit, override $\gamma_i$ upward to prevent breaching. Hitting a position limit means you cannot hedge or rebalance — a critical failure mode.
- **Discrete tick sizes** mean you must round quotes to valid price levels. Always round bids down and asks up to avoid giving away edge.
- **Multiple products** create both risk and opportunity. If two Prosperity products are cointegrated (common in baskets, ETF-like structures), trade the spread rather than each leg independently. The risk budget allocation framework above tells you how much capital to assign to each.
- **Adversarial bots** mean your fill model $\lambda(\delta) = A\,e^{-\kappa\delta}$ is approximate. Monitor fill rates empirically and update $A$ and $\kappa$ online. If a competing bot consistently takes your resting orders at bad times, you are experiencing adverse selection — widen spreads.
- **Parameter sensitivity**: start with conservative parameters ($\gamma = 0.1$, $\alpha = 0.1$, $\beta_{\text{skew}} = 0.05$) and tune from there. In a competition, robustness beats optimality — a strategy that works in 90% of regimes beats one that is optimal in 60% and blows up in 40%.

## Python Snippet

```python
import numpy as np

def integrated_quotes(S, q, sigma, gamma, kappa, T_minus_t, z_t,
                      alpha=0.2, beta_skew=0.1,
                      q_safe=5, q_max=20, lam_q=1.0,
                      sigma_bar=None, lam_sigma=1.0,
                      z_thresh=0.5, lam_z=0.3):
    """Compute bid and ask prices from the integrated AS + stat-arb framework."""
    r_mod = S - q * gamma * sigma**2 * T_minus_t - alpha * z_t * sigma

    delta = gamma * sigma**2 * T_minus_t + (2 / gamma) * np.log(1 + gamma / kappa)

    inv_mult = 1.0 + lam_q * max(0, (abs(q) - q_safe) / max(1, q_max - q_safe))
    vol_mult = 1.0
    if sigma_bar is not None and sigma_bar > 0:
        vol_mult = 1.0 + lam_sigma * max(0, (sigma - sigma_bar) / sigma_bar)
    sig_mult = 1.0 + lam_z * (1.0 if abs(z_t) < z_thresh else 0.0)
    delta_adj = delta * inv_mult * vol_mult * sig_mult

    skew = beta_skew * z_t

    bid = r_mod - (delta_adj + skew)
    ask = r_mod + (delta_adj - skew)
    return bid, ask, r_mod, delta_adj
```
