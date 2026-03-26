# Derivatives and Hedging

> **Key payoffs at expiry $T$:**
> $$\text{Call: } \max(S_T - K,\,0), \qquad \text{Put: } \max(K - S_T,\,0), \qquad \text{Forward: } S_T - F$$

## Intuition

A derivative is a contract whose value depends on the price of something else — the underlying asset. Rather than owning the asset directly, you own a claim on it under specific conditions. This allows market participants to transfer risk: a farmer who fears falling wheat prices can lock in a price today with a forward; an investor who fears a stock crash can buy a put as insurance.

Hedging is the practice of offsetting risk by taking a position in a related instrument. The fundamental insight driving all of modern options theory is that, under certain conditions, a derivative's payoff can be **replicated** by dynamically trading the underlying. If replication is possible, the derivative's fair price is exactly the cost of that replicating portfolio — any other price creates an arbitrage.

**Arbitrage** is the ability to make a certain profit with no initial capital and no risk. In practice: if Gold trades at \$2,500 in New York and \$2,505 in London simultaneously, buying in New York and selling in London is a risk-free gain. No-arbitrage is the key pricing axiom: if two portfolios produce identical future payoffs in every scenario, they must have the same price today.

## Common Derivative Instruments

### Forwards and Futures

A **forward contract** obliges both parties to trade the underlying at a fixed price $F$ (the forward price) at a future date $T$. No money changes hands at inception; the payoff at expiry is $S_T - F$ for the long side.

Under continuous compounding and no dividends, no-arbitrage pins the forward price to:

$$F = S_0\,e^{rT}$$

because any other price lets you replicate the forward with stock and bonds at a different cost, creating arbitrage.

**Futures** are exchange-traded forwards with daily mark-to-market settlement (variation margin). The economics are equivalent but the cash flows differ: gains and losses are settled daily rather than at expiry.

### Options: Calls and Puts

An option gives its buyer the **right but not the obligation** to trade the underlying at the strike price $K$.

| Instrument | Right of holder | Payoff at expiry |
|------------|----------------|-----------------|
| Long call  | Buy at $K$     | $\max(S_T - K, 0)$ |
| Short call | Obligation to sell at $K$ | $-\max(S_T - K, 0)$ |
| Long put   | Sell at $K$    | $\max(K - S_T, 0)$ |
| Short put  | Obligation to buy at $K$ | $-\max(K - S_T, 0)$ |

#### Long vs. Short

**Long** means you **buy** the option. You pay the premium today and receive the **positive** expiry payoff from the table above.

**Short** means you **sell** the option. You receive the premium today, but at expiry you must pay the long payoff — so the short payoff is exactly the negative of the long payoff. Consequence: a long position has the right-hand “asymmetry” (for calls typically limited downside), while a short position has the opposite risk profile (for calls it can be unbounded to the upside; for puts it is bounded by the strike in the model).

The call buyer profits when $S_T > K$ (the stock rises above strike); the put buyer profits when $S_T < K$ (the stock falls below strike). Options are **nonlinear** because of the $\max(\cdot,0)$ kink — this nonlinearity is the source of their richness and complexity.

An option is:
- **in the money (ITM)** if immediate exercise would be profitable ($S > K$ for call, $S < K$ for put),
- **at the money (ATM)** if $S \approx K$,
- **out of the money (OTM)** if immediate exercise would not be profitable.

**Option premium** is what the buyer pays upfront to acquire the right. It has two components:

$$\text{Premium} = \underbrace{(S - K)^+}_{\text{intrinsic value}} + \underbrace{\text{time value}}_{\text{optionality from remaining life}}$$

### European vs. American Options

| Feature | European | American |
|---------|---------|---------|
| Exercise timing | Only at expiry $T$ | Any time up to $T$ |
| Pricing | Closed-form BS formula | Requires numerical methods (binomial tree, PDE) |
| Put-call parity | Holds exactly | Holds only as inequality |

European options are the standard in theoretical work because they admit the Black–Scholes closed form. Most equity index options are European; most single-stock options are American.

For a European call on a non-dividend-paying stock, early exercise is never optimal (the time value is always positive), so American and European calls have the same price. This does not hold for puts.

### Put-Call Parity

For European options with the same strike $K$ and expiry $T$:

$$\boxed{C - P = S - Ke^{-rT}}$$

Conceptually, this means the portfolio **long call + short put** has the same expiry payoff as **long stock + short discounted strike**. So call and put premiums are linked by no-arbitrage: once you know one of them, the other is essentially forced up to the model-independent identity.

**Proof:** A portfolio that is long one call and short one put has payoff $S_T - K$ in all scenarios (in the money or out of the money). A forward with delivery price $K$ (long one share, short $Ke^{-rT}$ in bonds) replicates this exactly. By no-arbitrage, both portfolios must have the same value today.

Put-call parity is model-independent — it holds regardless of the assumed price process. Violations are pure arbitrage and exploitable immediately.

## Hedging

### General Principle: Replication and No-Arbitrage

If two portfolios have identical future payoffs in every state of the world, they must have the same price today. This is the **replication principle** and is a direct consequence of no-arbitrage.

For derivatives, the idea is:
1. Construct a portfolio of tradable assets (stock, bonds, other options) that replicates the derivative's payoff.
2. The cost of building that portfolio is the derivative's fair value.

If the fair value differs from the market price, you buy the cheap version, sell the expensive version, and lock in a risk-free profit — which is arbitrage. Competition among traders therefore drives the market price toward fair value.

### Static vs. Dynamic Hedging

**Static hedging:** Set up a portfolio once and do not trade until expiry. Works when a payoff can be replicated exactly by a fixed combination of instruments (e.g., put-call parity lets you replicate a forward statically). Requires no assumptions about price dynamics.

**Dynamic hedging:** Continuously rebalance a portfolio of the underlying and cash so that the portfolio tracks the option value at every instant. This is the Black–Scholes approach: the key assumption is that you can trade **continuously** and **without transaction costs**.

Dynamic hedging is more powerful (can replicate any smooth payoff) but requires a model for price dynamics (typically GBM) and is costly in practice due to transaction costs and discrete rebalancing.
The standard implementation is **delta hedging**: how many shares to hold against an option so the book is locally neutral to small moves in $S$, how to rebalance as Gamma moves Delta, and what P&L discrete hedging generates. See [greeks.md — Delta hedging](greeks.md#delta-hedging). The continuous-time argument $\Pi = V - \Delta S$ with $\Delta = \partial V/\partial S$ that removes the Brownian term and yields the **Black–Scholes PDE** is in [Black–Scholes](black_scholes.md), Step 3.

## Connections

- [Black–Scholes](black_scholes.md) — Full derivation of the BS PDE via delta-hedging (Step 3) and the closed-form pricing formula for European calls and puts.
- [Greeks](greeks.md) — All hedge ratios ($\Delta, \Gamma, \mathcal{V}, \Theta, \rho$) as partial derivatives of the BS formula; their interpretation, formulas, and use in portfolio risk management.
- [Implied volatility](implied_volatility.md) — The BS formula inverted to extract market-implied $\sigma$ from observed option prices; Vega is the key sensitivity.
- [Jump diffusion](jump_diffusion.md) — Merton's extension: when prices can jump, delta-hedging no longer eliminates all risk and the BS PDE gains a jump term.
- [Avellaneda–Stoikov](../03_market_making/avellaneda_stoikov.md) — When market-making options, you accumulate Greek exposures that need to be hedged; AS inventory risk is supplemented by delta and gamma hedging.

## Relevance for Prosperity 4

In Prosperity 4, options-like instruments appear with known expiries and tradable underlyings. The relevant applications are:

- **Fair value:** Use the Black–Scholes formula (see [black_scholes.md](black_scholes.md)) to price any European-style instrument from the current mid-price, estimated volatility, and time to expiry.
- **Delta-hedging inventory:** See [greeks.md — Delta hedging](greeks.md#delta-hedging); hedge option exposure with the underlying so portfolio Delta stays near zero.
- **Put-call parity:** Check $C - P = S - Ke^{-rT}$ in real time. Violations are immediate arbitrage — buy the underpriced side, sell the overpriced side, and hedge the remaining position.
- **Greek-based risk limits:** Monitor net Delta, Gamma, and Vega across your entire book. Large Gamma means rapid delta changes; large Vega means sensitivity to volatility shifts. The full formulas are in [greeks.md](greeks.md).
