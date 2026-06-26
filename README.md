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
dS_t = \bar{\mu} S_t dt + \sqrt{v_t} S_t dz_t^1 \\ 
dv_t = \alpha(\gamma - v_t) dt + \sigma \sqrt{v_t} dz_t^2 
\end{cases}
$$
With instantaneous correlation defined by:
$$ d\langle z^1, z^2 \rangle_t = \rho dt $$

### Parameter Definitions
* **Asset Price Dynamics ($S_t$):** Governed by the drift $\bar{\mu}$ under the physical measure $\mathbb{P}$.
* **Variance Process ($v_t$):** Follows a mean-reverting square-root process (Cox-Ingersoll-Ross type dynamics).
* **Mean-Reversion Speed ($\alpha$):** Determines how fast the variance returns to its long-term mean.
* **Long-Term Variance ($\gamma$):** The baseline level towards which the variance process drifts.
* **Volatility of Volatility ($\sigma$):** Represents the instantaneous standard deviation coefficient of the variance process.
* **Correlation ($\rho$):** Captures the leverage effect, where $d\langle z^1, z^2 \rangle_t = \rho dt$.

### The Role of Girsanov's Theorem in Asset Pricing

To transition from the physical measure $\mathbb{P}$ to an equivalent martingale measure $\mathbb{Q}$, we invoke **Girsanov's Theorem**. Since the Heston model features two coupled Brownian motions ($z_t^1, z_t^2$), we define a two-dimensional market price of risk vector:

$$ \mathbf{\Lambda}_t = \begin{bmatrix} q_1(S_t, v_t, t) \\ q_2(S_t, v_t, t) \end{bmatrix} $$

Where $q_1$ is dictated by the market risk premium of the underlying asset to eliminate arbitrage ($q_1 = \frac{\bar{\mu} - r}{\sqrt{v_t}}$), and $q_2(S_t, v_t, t) = q(S_t, v_t, t)$ represents the independent market price of volatility risk. 

By Girsanov's theorem, the transformed processes defined below are standard Brownian motions under $\mathbb{Q}$:

$$ dz_t^{1,\mathbb{Q}} = dz_t^1 + q_1(S_t, v_t, t)dt $$
$$ dz_t^{2,\mathbb{Q}} = dz_t^2 + q(S_t, v_t, t)dt $$
