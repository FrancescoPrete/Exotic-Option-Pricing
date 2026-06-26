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
<img width="3344" height="1768" alt="Implied volatility" src="https://github.com/user-attachments/assets/d4509184-d6c6-4c0f-8480-e4e6e12769b3" />

### The Heston stochastic volatility model
To resolve the previous limitation of a constant sigma, we use the Heston model, where the joint dynamics of asset price $S_t$ and its instantaneous variance $v_t$ are governed by the following SDEs:

$$
\begin{cases} 
dS_t = \bar{\mu} S_t dt + \sqrt{\,v_t^{\vphantom{2}}\,} \, S_t \, dz_t^1 \\ 
dv_t = \alpha(\gamma - v_t) dt + \sigma \sqrt{\,v_t^{\vphantom{2}}\,} \, dz_t^2 
\end{cases}
$$
With instantaneous correlation defined by:
$$ d\langle z^1, z^2 \rangle_t = \rho dt $$
