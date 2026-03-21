# Prosperity 3 Analysis

> **Core formula:** $$\text{Total PnL} = \underbrace{\sum_k \tfrac{1}{2}(p_k^{\text{ask}} - p_k^{\text{bid}})}_{\text{Spread Revenue}} - \underbrace{\sum_t q_t\,\Delta S_t}_{\text{Inventory Cost}} - \underbrace{\sum_k c_k}_{\text{Transaction Costs}} + \underbrace{\sum_t \alpha_t\,\Delta S_t}_{\text{Alpha P\&L}}$$

## Intuition

Every algorithmic trading competition is, at its core, a repeated multi-agent game played on a simulated exchange. Prosperity 3 was IMC's third iteration of this format, and it generated a rich corpus of public strategy write-ups from top-performing teams. Studying these results is not about copying strategies — the market microstructure changes between editions — but about extracting structural lessons: what classes of approaches are robust, what failure modes recur, and how to decompose PnL into components that can be independently optimised.

The central lesson from Prosperity 3 is that the competition rewards *robust simplicity* over *fragile complexity*. The winning teams implemented clean versions of well-known models — Avellaneda-Stoikov for market making, cointegration-based pairs trading for alpha, and exponential-window volatility estimation for risk management. They won not because their models were novel, but because their implementations were fast, correct, and adaptive. Overly complex approaches — deep learning on limited data, multi-factor alpha models without proper regularisation — consistently underperformed.

The second major insight is that Prosperity is a *game theory* problem as much as a quant finance problem. Your PnL depends not only on the true price process but on the behaviour of other bots. Understanding that most opponents likely run AS-like market-making strategies lets you anticipate their quotes and position accordingly. Teams that modelled the opponent population — even crudely — gained a measurable edge.

## Mathematical Setup

We decompose trading PnL into four additive components. For a market maker who executes $K$ round-trip trades over a session:

**Total PnL:**

$$\Pi = \Pi_{\text{spread}} - \Pi_{\text{inventory}} - \Pi_{\text{txn}} + \Pi_{\text{alpha}}$$

**Spread Revenue.** Each round-trip (buy at bid, sell at ask) captures the half-spread on each side:

$$\Pi_{\text{spread}} = \sum_{k=1}^{K} \frac{1}{2}\left(p_k^{\text{ask}} - p_k^{\text{bid}}\right)$$

In practice this is the *realised spread*: the difference between the execution price and the mid-price at the time of execution, summed over all fills.

**Inventory Cost.** Holding inventory $q_t$ while the mid-price moves by $\Delta S_t$ creates mark-to-market gains or losses:

$$\Pi_{\text{inventory}} = -\sum_{t=1}^{T} q_{t-1}\,\Delta S_t$$

This is a cost when inventory moves against you (long and price drops), and a gain when it moves in your favour. On average, for a mean-zero price process, $E[\Pi_{\text{inventory}}] = 0$ but $\text{Var}[\Pi_{\text{inventory}}] = \sigma^2 \sum_t q_t^2$. Large inventory = large variance = large risk.

**Transaction Costs.** Per-trade fees, if any:

$$\Pi_{\text{txn}} = \sum_{k=1}^{K} c_k$$

In Prosperity, explicit transaction fees may be zero, but there are implicit costs: crossing the spread when taking liquidity, and slippage from position limit constraints.

**Alpha P&L.** The directional component — PnL from correctly predicting price moves:

$$\Pi_{\text{alpha}} = \sum_{t=1}^{T} \alpha_t\,\Delta S_t$$

where $\alpha_t$ is the intended directional position driven by signals (as opposed to $q_t$, which includes unintended inventory from market-making fills).

## Derivation

### What Worked in Prosperity 3

#### 1. Aggressive Market Making

Top teams ran tight bid-ask spreads calibrated to realised volatility. The spread formula most teams converged on was a variant of Avellaneda-Stoikov:

$$\text{spread}_t = 2\,\delta^*_t = 2\left[\gamma\,\hat{\sigma}_t^2\,(T-t) + \frac{2}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right)\right]$$

The critical insight specific to Prosperity: **other bots frequently trade at suboptimal prices**, creating spread opportunities that would not exist on a real exchange. This is because many competing teams run simpler strategies — fixed spreads, pure trend-following, or naive mean-reversion — that leak value to sophisticated market makers.

The consequence: teams that captured the most volume (highest $K$ in the spread revenue formula) while maintaining a positive per-trade edge (positive $\frac{1}{2}(p^{\text{ask}} - p^{\text{bid}}) - \text{adverse selection cost}$) accumulated the most PnL. The law of large numbers applies: with thousands of trades per session, even a small per-trade edge compounds reliably.

Position limits in Prosperity forced inventory discipline. Teams that managed inventory well — using quick mean-reversion via the $q\,\gamma\,\sigma^2(T-t)$ penalty, hedging across correlated products, or hard-capping inventory at 80% of the limit — outperformed. Teams that hit position limits were effectively taken out of the game for that product until inventory naturally decayed.

#### 2. Volatility Estimation

Accurate volatility estimation was the single most impactful parameter calibration. Three approaches were used:

**Realised volatility from rolling windows:**

$$\hat{\sigma}_{\text{RV}}^2 = \frac{1}{n-1}\sum_{i=1}^{n} (r_i - \bar{r})^2, \quad r_i = S_i - S_{i-1}$$

with $n = 50$–$100$ ticks. Simple and robust, but slow to adapt.

**Exponentially weighted moving variance (EWMA):**

$$\hat{\sigma}_{\text{EW},t}^2 = \lambda\,\hat{\sigma}_{\text{EW},t-1}^2 + (1 - \lambda)\,(S_t - S_{t-1})^2$$

with $\lambda = 0.94$–$0.97$. Faster adaptation to regime changes, which is critical in Prosperity where volatility can shift abruptly between rounds.

**Parkinson (high-low) estimator:**

$$\hat{\sigma}_{\text{P}}^2 = \frac{1}{4\ln 2}\,\frac{1}{n}\sum_{i=1}^{n}(\ln H_i - \ln L_i)^2$$

where $H_i$, $L_i$ are the high and low prices in window $i$. This estimator is approximately $5\times$ more efficient than close-to-close volatility for the same sample size.

**Yang-Zhang estimator** combines overnight (open-to-close) and intraday (Parkinson) components for the most efficient estimate under realistic conditions. In Prosperity, where there is no "overnight" gap, the Parkinson or EWMA estimators are most relevant.

The operational impact: teams that correctly estimated $\sigma$ could set tighter spreads during calm periods (higher fill rate, more spread revenue) and wider spreads during volatile periods (less adverse selection loss). Teams with fixed or slowly-adapting $\sigma$ estimates either bled on adverse selection in volatile regimes or lost volume in calm regimes.

#### 3. Cointegration on Correlated Products

Several Prosperity 3 products were cointegrated — baskets, ETF-like structures, or products with a common underlying factor. Top teams exploited this in three steps:

**Step 1 — Identify the relationship.** Run Engle-Granger on a rolling window: regress $Y_t$ on $X_t$, obtain $\hat{\beta}$, and test the residuals $\hat{s}_t = Y_t - \hat{\beta}\,X_t$ for stationarity via the ADF test.

**Step 2 — Compute the spread and z-score.** If cointegration is confirmed:

$$s_t = Y_t - \hat{\beta}\,X_t, \quad z_t = \frac{s_t - \hat{\mu}_s}{\hat{\sigma}_s}$$

where $\hat{\mu}_s$ and $\hat{\sigma}_s$ are the rolling mean and standard deviation of the spread.

**Step 3 — Trade the mean-reversion.** Enter when $|z_t| > z_{\text{entry}}$ (typically 2.0), exit when $|z_t| < z_{\text{exit}}$ (typically 0.5). The position is hedged: go long $Y$ and short $\hat{\beta}$ units of $X$ (or vice versa), so directional risk is eliminated.

The critical implementation detail: **the hedge ratio $\hat{\beta}$ must be re-estimated periodically** (every 200–500 ticks). A stale $\hat{\beta}$ introduces unhedged directional exposure that masquerades as spread risk — a subtle but account-destroying error.

#### 4. Options Pricing

Some Prosperity 3 rounds included option-like payoffs (or products whose value was a nonlinear function of observables). Teams that could price these using Black-Scholes — or better, by calibrating implied volatility to the observable option prices — had a structural edge.

The two-part advantage: (1) **pricing** — knowing the fair value of the option means you can quote a spread around it and profit, and (2) **risk management** — understanding the Greeks (especially delta) lets you hedge the directional exposure from option fills. Teams that sold options without delta-hedging were exposed to unlimited directional risk.

The BS call price for reference:

$$C = S\,N(d_1) - K\,e^{-rT}N(d_2)$$

$$d_1 = \frac{\ln(S/K) + (r + \sigma^2/2)T}{\sigma\sqrt{T}}, \quad d_2 = d_1 - \sigma\sqrt{T}$$

In Prosperity, the key is not the formula itself — it is the ability to solve $\sigma_{\text{impl}}$ from observed market prices via Newton-Raphson and then use the calibrated $\sigma_{\text{impl}}$ to price other strikes or maturities consistently.

### What Did NOT Work

#### 1. Pure Directional Bets

Without strong, demonstrable alpha signals, pure directional trading was a losing strategy. The expected PnL of a directional bet is:

$$E[\Pi_{\text{dir}}] = \alpha \cdot E[\Delta S] - \text{spread paid} - \text{market impact}$$

Unless $\alpha \cdot E[\Delta S]$ exceeds the execution costs (typically $\geq 1$ tick of spread + impact), the strategy bleeds. In Prosperity's environment, mid-prices are designed to be approximately efficient on short horizons, so $E[\Delta S] \approx 0$ without specific structural signals. Teams that bet on direction without hedging inventory risk absorbed the full variance $\sigma^2 \sum_t q_t^2$ as realised losses in adverse scenarios.

#### 2. Static Strategies

Fixed-spread strategies — quoting a constant $\delta$ regardless of volatility, inventory, or market conditions — failed in two complementary ways:

- **During high-volatility regimes:** the fixed spread was too tight, and fills were dominated by informed flow (bots that trade *because* of the price move). Adverse selection losses exceeded spread revenue.
- **During low-volatility regimes:** the fixed spread was too wide, and fill rates dropped to near zero. Other teams with adaptive spreads captured all the volume.

The mathematical argument: the optimal spread $\delta^*(\sigma)$ is an increasing function of $\sigma$. A fixed $\delta$ crosses the optimal curve at exactly one point and is suboptimal everywhere else:

$$\text{Suboptimality} \propto \int_0^T \left(\delta_{\text{fixed}} - \delta^*(\hat{\sigma}_t)\right)^2 dt$$

#### 3. Over-Complex Models

Several teams reported using neural networks, reinforcement learning, or high-dimensional factor models. The failure mode was consistent: **overfitting to the training data**. Prosperity's training environment has limited historical data (typically a few thousand ticks across a handful of rounds), which is insufficient to train models with thousands of parameters.

The bias-variance tradeoff is stark:

$$\text{MSE} = \text{Bias}^2 + \text{Variance}$$

Simple models (AS, linear cointegration) have some bias — they assume constant parameters, Gaussian returns, etc. — but very low variance because they have few parameters. Complex models can reduce bias but explode in variance given limited data. In a competitive environment where the data-generating process shifts between rounds (different products, different bot populations), low-variance models win.

The empirical evidence from Prosperity 3 write-ups is unambiguous: the top 10 teams overwhelmingly used clean implementations of AS + pairs trading, not ML.

## Key Parameters

| Parameter | Symbol | Meaning | Typical range |
|-----------|--------|---------|---------------|
| Rolling window for $\sigma$ | $n$ | Number of ticks for volatility estimation | 50 – 100 |
| EWMA decay | $\lambda$ | Exponential decay for EWMA volatility | 0.94 – 0.97 |
| Z-score entry threshold | $z_{\text{entry}}$ | Trigger for entering mean-reversion trades | 1.5 – 2.5 |
| Z-score exit threshold | $z_{\text{exit}}$ | Trigger for closing mean-reversion trades | 0.3 – 0.75 |
| Hedge ratio | $\hat{\beta}$ | Cointegrating coefficient for pairs trades | Regression-estimated |
| Hedge ratio update frequency | $N_\beta$ | How often to re-estimate $\hat{\beta}$ | 200 – 500 ticks |
| Risk aversion | $\gamma$ | Controls inventory penalty in AS | 0.01 – 1.0 |
| Inventory hard limit | $q_{\max}$ | Absolute position cap per product | Set by competition rules |
| Inventory soft limit | $q_{\text{safe}}$ | Level at which spread widening begins | 50% – 80% of $q_{\max}$ |

## Numerical Example

**PnL decomposition for a single session.** A market maker trades one product for 1000 ticks. Suppose we observe the following aggregates:

- 200 round-trip trades completed
- Average half-spread captured: $0.15$ per trade per side
- Average inventory held: $\bar{q} = 3.2$ units
- Volatility: $\sigma = 0.4$ per $\sqrt{\text{tick}}$
- Net directional signal PnL: $+2.1$ (from correctly timing entries via z-score)
- Transaction costs: $0$ (Prosperity has no explicit fees in most rounds)

**Spread Revenue:**

$$\Pi_{\text{spread}} = 200 \times 2 \times 0.15 = 60.0$$

(200 trades, half-spread of 0.15 captured on each of the buy and sell sides.)

**Inventory Cost (expected variance, not realised):**

$$E\!\left[\Pi_{\text{inventory}}^2\right] = \sigma^2 \sum_{t} q_t^2 \approx \sigma^2 \cdot T \cdot \bar{q}^2 = 0.16 \cdot 1000 \cdot 10.24 = 1638.4$$

$$\text{Std dev of inventory cost} = \sqrt{1638.4} \approx 40.5$$

The realised inventory cost in any single session is a draw from a distribution with standard deviation $\approx 40.5$. This is the dominant source of PnL variance and explains why inventory management is the #1 differentiator.

**Suppose the realised inventory cost this session was $-12.3$ (inventory moved against us on net).**

**Alpha P&L:**

$$\Pi_{\text{alpha}} = +2.1$$

**Total PnL:**

$$\Pi = 60.0 - 12.3 - 0 + 2.1 = 49.8$$

**Decomposition percentages:** Spread revenue accounts for $60.0 / 49.8 = 120\%$ of total PnL, inventory cost consumed $-12.3 / 49.8 = -25\%$, and alpha contributed $2.1 / 49.8 = 4\%$. This is typical: spread revenue is the engine, inventory cost is the main risk, and alpha is a smaller but consistent contributor.

```python
import numpy as np

n_trades = 200
avg_half_spread = 0.15
sigma = 0.4
T = 1000
avg_q = 3.2
alpha_pnl = 2.1
txn_cost = 0.0

spread_rev = n_trades * 2 * avg_half_spread
inv_cost_std = sigma * np.sqrt(T) * avg_q
realised_inv_cost = 12.3  # example draw

total_pnl = spread_rev - realised_inv_cost - txn_cost + alpha_pnl

print(f"Spread Revenue:     {spread_rev:+.1f}")
print(f"Inventory Cost:     {-realised_inv_cost:+.1f}  (std ≈ {inv_cost_std:.1f})")
print(f"Transaction Costs:  {-txn_cost:+.1f}")
print(f"Alpha P&L:          {alpha_pnl:+.1f}")
print(f"{'─' * 36}")
print(f"Total PnL:          {total_pnl:+.1f}")
```

## Transferable Lessons for Prosperity 4

### Lesson 1: Start with AS Market Making as the Baseline

Avellaneda-Stoikov is the workhorse. It is robust across market regimes, always provides positive expected PnL from spread capture (as long as $\sigma$ is estimated reasonably), and degrades gracefully when conditions worsen (the spread widens automatically). Build this first, verify it works, then layer on enhancements.

### Lesson 2: Layer Statistical Arbitrage Signals for Alpha

Once the base market maker is running, add directional signals from cointegration, mean-reversion, and volatility arbitrage. The integration framework in [strategy_integration.md](./strategy_integration.md) shows exactly how to modify the reservation price and skew quotes. The alpha PnL component is small but consistent and compounds over thousands of ticks.

### Lesson 3: Manage Inventory Obsessively

The PnL decomposition makes this clear: inventory cost has the highest variance of any component. The difference between a good team and a great team is not the alpha model — it is how aggressively and intelligently they manage $q_t$. Practical rules:

- Never let $|q|$ exceed 80% of the position limit
- Widen spreads asymmetrically when inventory is high (see [Inventory Risk](../03_market_making/inventory_risk.md))
- Hedge across correlated products whenever possible
- If in doubt, flatten inventory — missed alpha is less costly than a blow-up

### Lesson 4: Adapt Parameters Online

Fixed parameters lose to adaptive ones. Re-estimate at least every $N$ ticks:

- $\hat{\sigma}_t$: EWMA with $\lambda = 0.94$, or rolling window of 50 ticks
- $\gamma_t$: increase when inventory is dangerously high, decrease when positions are flat
- $\hat{\beta}_t$: re-run cointegration regression every 200–500 ticks
- $\kappa_t$: track fill rates and infer effective order-book depth

### Lesson 5: Keep It Simple

The bias-variance argument applies to strategy design, not just statistical models. A clean, well-tested AS implementation with one stat-arb overlay beats a spaghetti codebase with five alpha signals, three ML models, and custom execution logic. In a timed competition, bugs in complex code are the #1 killer. Ship correct code, not clever code.

### Lesson 6: Study Other Bots (Game Theory)

Prosperity is a multi-agent game. If you know (or can infer) that most opponents run AS-like strategies, you can:

- Predict their quotes: if $\hat{\sigma}$ just spiked, their spreads will widen, creating opportunities for you to provide liquidity at tighter spreads and capture volume.
- Avoid being adversely selected: if an opponent suddenly tightens their bid, they may have a signal that the price is going up — do not sell to them cheaply.
- Coordinate implicitly: in repeated rounds with the same bots, consistent quoting behaviour can lead to cooperative equilibria where all market makers earn positive PnL (see [Repeated Games](../05_game_theory/repeated_games.md)).

## PnL Decomposition Framework

The decomposition formula from the top of this file is the single most important diagnostic tool for post-session analysis. Here is how to measure each component from trade data:

**Spread Revenue** — for each fill $k$ at price $p_k$ and side $s_k \in \{+1, -1\}$ (buy = $+1$, sell = $-1$):

$$\Pi_{\text{spread}} = \sum_{k=1}^{K} s_k \cdot (S_{t_k}^{\text{mid}} - p_k)$$

If you bought below mid or sold above mid, this term is positive.

**Inventory Cost** — mark-to-market the running position:

$$\Pi_{\text{inventory}} = \sum_{t=1}^{T} q_{t-1} \cdot (S_t - S_{t-1})$$

Compute this from the time series of positions and mid-prices.

**Transaction Costs** — sum of explicit fees:

$$\Pi_{\text{txn}} = \sum_{k=1}^{K} c_k$$

**Alpha P&L** — isolate the intentional directional component. If your target position (from signals) at time $t$ is $\alpha_t$, and your actual position is $q_t$, then:

$$\Pi_{\text{alpha}} = \sum_{t=1}^{T} \alpha_t \cdot (S_t - S_{t-1})$$

$$\Pi_{\text{noise}} = \sum_{t=1}^{T} (q_t - \alpha_t) \cdot (S_t - S_{t-1})$$

The noise PnL captures the unintended inventory exposure from market-making fills. In a good strategy, $\text{Var}[\Pi_{\text{noise}}]$ is small relative to $\Pi_{\text{spread}}$.

**Diagnostic ratios:**

- Spread capture ratio: $\frac{\Pi_{\text{spread}}}{\Pi_{\text{total}}}$ — target $> 80\%$ for a market-making strategy
- Inventory risk ratio: $\frac{\text{Std}[\Pi_{\text{inventory}}]}{\Pi_{\text{spread}}}$ — target $< 1.0$; above 1.0 means inventory risk dominates
- Alpha efficiency: $\frac{\Pi_{\text{alpha}}}{\text{Std}[\Pi_{\text{alpha}}]}$ — this is the Sharpe ratio of the alpha signal; target $> 0.5$

## Connections

- [Strategy Integration](./strategy_integration.md) — the operational framework for combining the lessons from this file into a live trading system.
- [Avellaneda-Stoikov](../03_market_making/avellaneda_stoikov.md) — the core market-making model that was the backbone of top Prosperity 3 strategies.
- [Inventory Risk](../03_market_making/inventory_risk.md) — the theoretical foundation for why inventory management was the #1 differentiator.
- [Bid-Ask Spread](../03_market_making/bid_ask_spread.md) — understanding why adaptive spreads outperformed fixed spreads.
- [Cointegration](../04_stat_arb/cointegration.md) — the statistical framework for identifying tradeable relationships between Prosperity products.
- [Z-Score Signals](../04_stat_arb/zscore_signals.md) — the signal generation framework that top teams used for alpha.
- [Ornstein-Uhlenbeck](../04_stat_arb/ornstein_uhlenbeck.md) — the mean-reversion dynamics underlying pairs trading PnL.
- [Black-Scholes](../02_options/black_scholes.md) — the options pricing framework used in rounds with option-like products.
- [Implied Volatility](../02_options/implied_volatility.md) — the calibration technique for extracting $\sigma_{\text{impl}}$ from option prices.
- [Nash Equilibrium](../05_game_theory/nash_equilibrium.md) — the game-theoretic lens for understanding multi-agent competition.
- [Repeated Games](../05_game_theory/repeated_games.md) — cooperative equilibria across repeated Prosperity rounds.

## Relevance for Prosperity 4

Everything in this file is directly transferable to Prosperity 4 with one caveat: the specific products and bot population will change. The structural lessons — adaptive AS market making, cointegration-based alpha, obsessive inventory management, online parameter estimation, simplicity, and game-theoretic awareness — are invariant across editions because they derive from fundamental principles (mean-variance optimisation, stationarity, law of large numbers) rather than competition-specific artefacts. When building your Prosperity 4 bot, use this file as a checklist: (1) can your bot estimate $\sigma$ adaptively? (2) does it manage inventory with a penalty term? (3) does it detect and exploit cointegration? (4) is the codebase simple enough to debug under time pressure? If any answer is "no", fix that before adding new features. The PnL decomposition framework should be run after every backtest and every live round to diagnose where PnL is coming from and where risk is leaking.

## Python Snippet

```python
import numpy as np

def decompose_pnl(mid_prices, fill_prices, fill_sides, fill_times,
                   positions, target_positions=None, fees=None):
    """
    Decompose session PnL into spread, inventory, transaction, and alpha components.

    Parameters
    ----------
    mid_prices : array of shape (T,) — mid-price at each tick
    fill_prices : array of shape (K,) — execution price of each fill
    fill_sides : array of shape (K,) — +1 for buy, -1 for sell
    fill_times : array of shape (K,) — tick index of each fill
    positions : array of shape (T,) — inventory at each tick
    target_positions : array of shape (T,) or None — intended directional position
    fees : array of shape (K,) or None — per-trade fees
    """
    spread_pnl = np.sum(fill_sides * (mid_prices[fill_times] - fill_prices))

    delta_s = np.diff(mid_prices)
    inventory_pnl = np.sum(positions[:-1] * delta_s)

    txn_cost = np.sum(fees) if fees is not None else 0.0

    alpha_pnl = 0.0
    if target_positions is not None:
        alpha_pnl = np.sum(target_positions[:-1] * delta_s)

    total = spread_pnl + inventory_pnl - txn_cost

    return {
        'total': total,
        'spread': spread_pnl,
        'inventory': inventory_pnl,
        'transaction': -txn_cost,
        'alpha': alpha_pnl,
        'inv_risk_std': np.std(positions[:-1] * delta_s) * np.sqrt(len(delta_s)),
    }
```
