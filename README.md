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

### Risk-Neutral Measure & Girsanov's Theorem ($\mathbb{P} \to \mathbb{Q}$)

To price exotic options using Monte Carlo simulations or Finite Difference Methods, the system must be modeled under the risk-neutral measure $\mathbb{Q}$. By invoking **Girsanov's Theorem**, the drift of the underlying asset is shifted from $\bar{\mu}$ to the risk-free interest rate $r$. 

Since volatility is not a traded asset, the market is incomplete, requiring the definition of the market price of volatility risk, $q(S_t, v_t, t)$. To maintain mathematical parsimony we assume the market price of volatility risk to be zero:

$$q(S_t, v_t, t) = 0$$

Under this assumption, the structural parameters of the variance process ($\alpha, \gamma, \sigma$) remain identical under both the physical measure $\mathbb{P}$ and the risk-neutral measure $\mathbb{Q}$. The resulting system under $\mathbb{Q}$ is formulated as:

$$
\begin{cases} 
dS_t = r S_t dt + \sqrt{\,v_t\,} \, S_t \, dz_t^{1,\mathbb{Q}} \\ 
dv_t = \alpha(\gamma - v_t) dt + \sigma \sqrt{\,v_t\,} \, dz_t^{2,\mathbb{Q}} 
\end{cases}
$$

Where $d\langle z^{1,\mathbb{Q}}, z^{2,\mathbb{Q}} \rangle_t = \rho dt$, with $\rho < 0$ to capture the empirical leverage effect.

---

## 🔬 Numerical Methods Framework

To implement the pricing engine in Python, the continuous risk-neutral SDEs are discretized. This framework utilizes two distinct numerical paradigms: **Path Simulation (Monte Carlo)** and **Grid Discretization (Finite Differences)**.

### 1. Vectorized Monte Carlo Simulation
Under Monte Carlo pricing, the expected payoff is computed by simulating $num\_paths$ independent asset trajectories over a discrete time grid $0 = t_0 < t_1 < \dots < t_n = T$ with a constant time step $\Delta t = T/n$.

The simulation is **fully vectorized across all paths** to leverage NumPy's low-level optimizations, executing the spatial updates simultaneously at each temporal step $t$.

#### Variance Path Discretization (Full Truncation Scheme)
To handle the non-negativity constraint of the variance process $v_t$ under Euler-Maruyama discretization, a **Full Truncation Scheme** is applied. This prevents numerical instability and domain errors inside the square-root coefficient:

$$v_{t+1} = \max\left( v_t + \alpha(\gamma - v_t)\Delta t + \sigma \sqrt{\max(v_t, 0)} \sqrt{\Delta t} \, \epsilon_t^2, \; 0 \right)$$

Where $\epsilon_t^2 \sim \mathcal{N}(0,1)$ represents the random shock driving the volatility process.

#### Asset Path Discretization
The underlying asset price $S_t$ is discretized linearly on price levels matching the lecture framework:

$$S_{t+1} = S_t + r S_t \Delta t + \sqrt{\max(v_t, 0)} S_t \sqrt{\Delta t} \, \epsilon_t^1$$

Where the asset shock $\epsilon_t^1 \sim \mathcal{N}(0,1)$ is correlated to the variance shock via the leverage parameter $\rho$:

$$\epsilon_t^1 = Z_t^1$$
$$\epsilon_t^2 = \rho Z_t^1 + \sqrt{1 - \rho^2} Z_t^2$$

With $Z_t^1, Z_t^2 \stackrel{iid}{\sim} \mathcal{N}(0,1)$.

#### Monte Carlo Estimator for Exotic Payoffs
The fair price of a path-dependent exotic option with an arbitrary payoff function $\Phi(\{S_t\}_{t=0}^n)$ is evaluated via the risk-neutral discounted sample average across all simulated paths:

$$\hat{C}_0 = e^{-rT} \frac{1}{num\_paths} \sum_{i=1}^{num\_paths} \Phi\left( \{S_t^{(i)}\}_{t=0}^n \right)$$

---

### 2. Finite Difference Method (PDE Approach)
By the Feynman-Kac theorem, the pricing function $U(S, v, t)$ for any derivative under the risk-neutral Heston dynamics satisfies the following 2D partial differential equation (PDE):

$$\frac{\partial U}{\partial t} + rS \frac{\partial U}{\partial S} + \alpha(\gamma - v) \frac{\partial U}{\partial v} + \frac{1}{2} v S^2 \frac{\partial^2 U}{\partial S^2} + \rho \sigma v S \frac{\partial^2 U}{\partial S \partial v} + \frac{1}{2} \sigma^2 v \frac{\partial^2 U}{\partial v^2} - rU = 0$$

#### Grid Discretization & ADI Schemes
To solve this 2D PDE numerically, we define a localized spatial grid $(S, v) \in [0, S_{\max}] \times [0, v_{\max}]$. Due to the cross-derivative term $\frac{\partial^2 U}{\partial S \partial v}$ introduced by the correlation $\rho$, standard implicit methods become computationally expensive. 

To maintain stability and speed, the engine implements an **Alternating Direction Implicit (ADI)** scheme (such as the Craig-Sneyd or Hundsdorfer-Verwer splitting method), breaking the 2D problem into successive 1D tridiagonal systems at each time step.
