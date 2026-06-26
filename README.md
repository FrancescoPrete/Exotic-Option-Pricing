# Exotic-Option-Pricing
An open-source quantitative framework for exotic options pricing (Monte Carlo &amp; Finite Differences). Project under expansion to incorporate stochastic volatility models.
## Key features

* **Stochastic volatility integration:** Implementation of Heston model to capture the empirical volatility smile and leverage effects.
* **Numerical pricing methods:**
  * Vectorized Monte Carlo simulation
  * Finite Differences to solve the pricing PDEs.
* **Exotic payoffs supported:** Asian (arithmetic and geometric average), barrier and binary options.

## Mathematical framework

### The benchmark for the options pricing: Black-Scholes model
The asset price $S_t$ is first modeled under standard geometrical Brownian motion with constant volatility $\sigma$:

$$dS_t= \mu S_t+ \sigma dW_t$$

While computationally efficient for path-dependent exotic options under constant $\sigma$ assumption, this limitation fails to capture the empirical volatility smile observed in live option chains. 
<img width="3044" height="1568" alt="Implied volatility" src="https://github.com/user-attachments/assets/d4509184-d6c6-4c0f-8480-e4e6e12769b3" />
