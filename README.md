# Prosperity 4 — Quantitative Finance Toolkit

A structured learning and implementation repository for the IMC Prosperity 4 algorithmic trading competition, covering stochastic calculus, options pricing, market making, statistical arbitrage, and game theory.

Built by a first-year Electrical Engineering & Information Technology student at ETH Zürich 
as a self-study project to develop the theoretical foundations of quantitative finance from 
the ground up to bridge the gap between the mathematics of stochastic processes and their 
practical application in algorithmic trading.

![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue)
![License MIT](https://img.shields.io/badge/License-MIT-green)
![Status Active](https://img.shields.io/badge/Status-Active-brightgreen)

---

## Table of Contents

- [Topic Areas](#topic-areas)
  - [1. Stochastic Foundations](#1-stochastic-foundations)
  - [2. Options Pricing](#2-options-pricing)
  - [3. Market Making](#3-market-making)
  - [4. Statistical Arbitrage](#4-statistical-arbitrage)
  - [5. Game Theory](#5-game-theory)
  - [6. Prosperity Strategy](#6-prosperity-strategy)
- [Directory Structure](#directory-structure)
- [Getting Started](#getting-started)
- [Notebook Order](#notebook-order)
- [Resources](#resources)

---

## Topic Areas

### 1. Stochastic Foundations

The mathematical bedrock for everything that follows. Brownian motion defines the noise model for asset prices, Itô calculus provides the chain rule for stochastic processes, and SDEs give the language for writing down price dynamics. The key result is the Itô correction: for geometric Brownian motion $`dS = \mu S\,dt + \sigma S\,dW`$, the log-price drifts at $`\mu - \sigma^2/2`$ rather than $`\mu`$, because the exponential function's convexity interacts with the non-zero quadratic variation of Brownian motion.

$$S_t = S_0 \exp\!\bigl((\mu - \tfrac{1}{2}\sigma^2)\,t + \sigma W_t\bigr)$$

### 2. Options Pricing

Black-Scholes pricing, the Greeks, jump-diffusion extensions, and implied volatility. The BS model derives a closed-form option price by constructing a self-financing delta-hedging portfolio and invoking no-arbitrage. The five Greeks (Delta, Gamma, Vega, Theta, Rho) quantify an option's sensitivity to each input parameter and are essential for risk management. Jump-diffusion (Merton 1976) extends the model to capture fat tails and volatility skew.

$$C = S\,N(d_1) - K e^{-rT} N(d_2), \quad d_1 = \frac{\ln(S/K) + (r + \sigma^2/2)\,T}{\sigma\sqrt{T}}$$

### 3. Market Making

The Avellaneda-Stoikov (2008) framework for optimal market making under inventory risk. The market maker quotes a bid and ask around a reservation price that adjusts for current inventory exposure. The model balances two objectives: capturing spread revenue from the bid-ask spread and managing the risk of holding inventory that may lose value. This is the core strategy engine for Prosperity.

$$r_t = S_t - q_t\,\gamma\,\sigma^2(T-t), \quad \delta^* = \frac{\gamma\sigma^2(T-t)}{2} + \frac{2}{\gamma}\ln\!\left(1 + \frac{\gamma}{\kappa}\right)$$

### 4. Statistical Arbitrage

Mean-reversion trading via the Ornstein-Uhlenbeck process, cointegration testing, and z-score signals. When two assets share a long-run equilibrium, their spread is stationary and mean-reverting. The spread's z-score provides entry and exit signals: go long when the spread is unusually low, short when unusually high, and exit when it reverts. The OU half-life $`t_{1/2} = \ln 2 / \theta`$ determines whether mean-reversion is fast enough to trade within a given horizon.

$$dX_t = \theta(\mu - X_t)\,dt + \sigma\,dW_t$$

### 5. Game Theory

Nash equilibrium, auction theory, and repeated games applied to multi-agent trading. In Prosperity, multiple bots compete on the order book simultaneously. Nash equilibrium analysis predicts where competitors will quote. Auction theory (Vickrey's dominant-strategy truthful bidding, first-price bid shading) applies to pricing rounds. Repeated-game theory (folk theorem, tit-for-tat) explains how cooperation or competition can emerge across multiple rounds.

$$u_i(s_i^*, s_{-i}^*) \ge u_i(s_i, s_{-i}^*) \quad \forall\, s_i \in S_i$$

### 6. Prosperity Strategy

Integration of all the above into a unified competition strategy. Combines AS market making with stat arb directional signals, allocates risk budgets across instruments, and incorporates lessons from Prosperity 3. The modified reservation price $`r_t = S_t - q\gamma\sigma^2(T-t) + \alpha z_t \sigma`$ adds a mean-reversion signal to the standard AS formula, skewing quotes toward profitable direction when the z-score is extreme.

$$\pi_t = \pi_t^{\text{spread}} + \pi_t^{\text{inventory}} + \pi_t^{\text{alpha}}$$

---

## Directory Structure

```
prosperity4-quant/
├── README.md
├── requirements.txt
├── notebooks/
│   ├── 01_stochastic_foundations.ipynb
│   ├── 02a_black_scholes_greeks.ipynb
│   ├── 02b_jump_diffusion.ipynb
│   ├── 03_market_making_avellaneda_stoikov.ipynb
│   ├── 04_stat_arb_mean_reversion.ipynb
│   └── 05_game_theory_applied.ipynb
├── theory/
│   ├── 01_stochastic/
│   │   ├── brownian_motion.md
│   │   ├── ito_calculus.md
│   │   └── sdes.md
│   ├── 02_options/
│   │   ├── black_scholes.md
│   │   ├── greeks.md
│   │   ├── jump_diffusion.md
│   │   └── implied_volatility.md
│   ├── 03_market_making/
│   │   ├── avellaneda_stoikov.md
│   │   ├── inventory_risk.md
│   │   └── bid_ask_spread.md
│   ├── 04_stat_arb/
│   │   ├── ornstein_uhlenbeck.md
│   │   ├── cointegration.md
│   │   └── zscore_signals.md
│   ├── 05_game_theory/
│   │   ├── nash_equilibrium.md
│   │   ├── auction_theory.md
│   │   └── repeated_games.md
│   └── 06_prosperity/
│       ├── strategy_integration.md
│       └── prosperity3_analysis.md
├── plots/
│   ├── 01_stochastic/
│   ├── 02_options/
│   ├── 03_market_making/
│   ├── 04_stat_arb/
│   └── 05_game_theory/
└── references/
│   ├── avellaneda_stoikov_2008.pdf
│   ├── black_scholes_1973.pdf
│   ├── chakraborty_kearns_2011.pdf
│   ├── merton_jump_diffusion_1976.pdf
│   └── osborne_rubinstein_game_theory_1994.pdf
```

---

## Getting Started

1. **Clone and install dependencies:**

```bash
cd prosperity4-quant
pip install -r requirements.txt
```

2. **Run the notebooks in order** (each builds on the previous):

| Order | Notebook | Topic |
|-------|----------|-------|
| 1 | `01_stochastic_foundations.ipynb` | Brownian motion, GBM, Itô's lemma |
| 2a | `02a_black_scholes_greeks.ipynb` | BS pricing, Greeks, implied volatility |
| 2b | `02b_jump_diffusion.ipynb` | Merton model, fat tails, jump pricing |
| 3 | `03_market_making_avellaneda_stoikov.ipynb` | AS market making, inventory control |
| 4 | `04_stat_arb_mean_reversion.ipynb` | OU process, cointegration, pairs trading |
| 5 | `05_game_theory_applied.ipynb` | Nash equilibria, auctions, repeated games |

3. **Read the theory files** in `theory/` for full derivations and mathematical background. Each notebook links to its corresponding theory files.


---

| # | Reference | Description |
|---|-----------|-------------|
| 1 | **Black, F. & Scholes, M.** (1973). *The pricing of options and corporate liabilities.* Journal of Political Economy, 81(3), 637–654. | Original Black-Scholes paper. Derivation of the BS PDE via delta-hedging, closed-form option pricing formula. |
| 2 | **Merton, R.C.** (1976). *Option pricing when underlying stock returns are discontinuous.* MIT Sloan Working Paper No. 787. | Extends BS with Poisson-distributed jumps. Closed-form series solution for jump-diffusion pricing. |
| 3 | **Avellaneda, M. & Stoikov, S.** (2008). *High-frequency trading in a limit order book.* Quantitative Finance, 8(3), 217–224. | The canonical market making model. Reservation price and optimal spread derived via stochastic control (HJB equation). Core strategy reference for Prosperity. |
| 4 | **Chakraborty, T. & Kearns, M.** (2011). *Market making and mean reversion.* Proc. 12th ACM Conference on Electronic Commerce. | Connects market making with mean-reversion signals. Analyses adverse selection, temporary vs. permanent price impact, and optimal trading speed. |
| 5 | **Osborne, M.J. & Rubinstein, A.** (1994). *A course in game theory.* MIT Press. | Full-text freely available. Covers Nash equilibrium, mixed strategies, extensive games, repeated games, and auction theory with full proofs. |

### Additional resources

- **timodiehm — Prosperity 3 write-up** (2nd place globally, 9000+ teams):
  `https://github.com/TimoDiehm/imc-prosperity-3`
- **CarterT27 — Prosperity 3** (9th place globally):
  `https://github.com/CarterT27/imc-prosperity-3`
- **Ben Polak — ECON 159: Game Theory** (Open Yale, free video lectures):
  `https://oyc.yale.edu/economics/econ-159`