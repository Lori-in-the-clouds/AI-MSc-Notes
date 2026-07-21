# 14. Output Data Analysis

Num of pages: 2
Status: Done
Type: theory

# 1. Introduction

In simulation modeling, the goal is often to evaluate changes aimed at improving performance, such as increasing throughput or reducing waiting times. However, observing the output of a single simulation run is not sufficient.

Because simulations rely on random inputs, their outputs are also **random variables**. A single run represents only one realization of a stochastic process. Therefore, **statistical analysis is required** to determine whether observed performance differences are truly significant or simply due to random variability.

## 1.1. Output Data is not IID

A fundamental challenge in output analysis is that raw simulation metrics are **not IID** data:

1. **Not Identical (Non-Stationary)** → the data distribution changes over time.
    - Simulations typically start with the system in an "idle and empty" state (e.g., queue size = 0). Consequently, initial metrics (waiting times, queue sizes) are significantly different from those observed when the system is fully operational.
    - The system only acquires stable, stationary behavior after a **warm-up period.**
2. **Not Independent (Autocorrelated)** → successive observations are strongly correlated with one another.
    
    *Example:* If a system contains 4 clients at time $t$, it is highly probable that at time $t+1$ it will contain a similar number (e.g., 3, 4, or 5). It is statistically unlikely to suddenly jump to 0 or 100 clients.
    

So to analyze output data correctly, we cannot use standard statistics immediately. We must:

- **Discard initial values**, as they are non-stationary and correlated to unrealistic initial conditions (the "startup problem").
- Use **correct statistical techniques** (like batch means or replication methods) that account for autocorrelation.

---

# 2. Statistics Recap

- **Variance** → samples sparsity with respect to mean value:  $Var(X)=\mathbb{E}\left[(X-\mathbb{E}[X])^2\right]$
- **Covariance** → measures how two random variables vary together:  $Cov(X,Y)=\mathbb{E}\left[(X-\mathbb{E}[X])(Y-\mathbb{E}[Y])\right]$
- **Correlation** → normalized covariance, indicating the strength of linear dependence:  $\displaystyle \rho(X,Y)=\frac{Cov(X,Y)}{\sqrt{Var(X)}\cdot\sqrt{Var(Y)}}$
- **Autocorrelation** → correlation between observations of the same process at different time lags:  $\displaystyle\rho_j=\frac{Cov(X_t,X_{t+j})}{Var(X_t)}$

---

# 3. Simulation as Stochastic Processes

A simulation output is a collection of random variables ordered over time, known as a **stochastic process**. To apply standard statistical analysis, it is common to assume that this process is **covariance-stationary**.

<aside>
💡

**Covariance-Stationary Conditions**

A stochastic process is covariance-stationary if it satisfies the following conditions:

1. **Stable mean**: $\mu_i=\mu\quad\text{for}\ \  i=1,2,...\ \ \text{and}\ -\infty<\mu<\infty$
2. **Stable covariance**:  $\sigma_i^2=\sigma^2\quad\text{for}\ \  i=1,2,...\ \ \text{and}\ \sigma^2<\infty$
3. **Stationary Autocovariance**: the covariance between two observations depends only on their time separation (lag), not on the absolute time:
    
    $$
    C_{i,i+j}=Cov(X_i,X_{i+j})\ \text{is independent of} \ i\ \text{for}\ j=1,2,...
    $$
    
    where $j$ is the **lag** (it is a $\Delta t$).
    
</aside>

---

# 4. Types of Simulations

Simulations are generally classified into two categories based on their objectives:

- **Terminating Simulation** → has a specific, natural event $E$ that defines the end of the run. To estimate performance measures over that specific finite period.
- **Non-Terminating Simulation** → has no natural end event. The objective is to study the long-run behavior of the system.
    
    **N.B.** These simulations require the system to reach a **steady state**, where the distribution of the output process becomes independent of the initial conditions.
    

---

# 5. Output Analysis for Terminating Simulations

To analyze terminating simulations, we use the method of **Independent Replications**. Since observations within a single run are autocorrelated, statistical independence is achieved **across multiple runs**.

1. Run $n$ replications of the simulation. Use the same initial conditions and terminal events for each replication. We used use **different random number seeds** (e.g., $C_{42}$, $C_{21}$) for each replication to ensure they are statistically independent.
2. For each replication $j$, compute a summary measure $X_j$ (e.g., the average delay for that specific run). The collection $X_1,X_2,...,X_n$ forms an **IID sample**, allowing the calculation of a sample mean $\bar{X}(n)$ and sample variance $S^2(n)$.

## 5.1. Confidence Intervals (CI)

A point estimate alone (e.g., *the average delay is 99*) does not show how reliable the result is. A **Confidence Interval (CI)** provides a range that contains the true mean with a specified probability:

$$
\bar{X}(n) \pm\ t_{n-1,1-\alpha/2}\cdot \sqrt{\frac{S^2(n)}{n}}
$$

Where:

- $\displaystyle \bar{X}(n) = \frac{\sum_{i=1}^n X_i}{n}$ is the sample mean
- $\displaystyle S^2(n) = \frac{\sum_{i=1}^n [X_i - \bar{X}(n)]^2}{n-1}$ is the sample variance
- $1-\alpha$ is the confidence level:
    - **High Confidence** (e.g., $α=0.05→95\%$) ****→ we are 95% confident that the true value lies within the interval. This results in a **wider** interval.
    - **Lower Confidence** (e.g., $α=0.30→70\%$) ****→ we are less confident, but the interval is **smaller** (narrower).
- $t_{n−1,1−α/2}$ is critical value from the Student’s *t*-distribution, obtained from a table using:
    - $n-1$: degree of freedom (related to the number of independent observations used to estimate the variance).
    - $1-\alpha/2$: cumulative probability. The division by 2 is used to split the error probability $\alpha$ equally between the lower and upper tails of the distribution.

<aside>
🔑

**Normal vs. T-Student Distribution**

While large samples approximate a Normal distribution, simulation replications often have a sample size $n<30$. In these cases, the **Student's t-distribution** must be used. It has heavier tails than the Gaussian distribution, producing wider and more conservative confidence intervals that account for the uncertainty of small samples.

![image.png](14%20Output%20Data%20Analysis/d64a6fea-3824-4578-99ec-abc120142232.png)

</aside>

## 5.2. Controlling Precision

The main limitation of fixing the sample size $n$ is that you cannot control the **precision** of your results. Since you do not know the variance before running the simulation, you risk obtaining a confidence interval that is too wide to be useful (e.g., an estimate of $99±50$). To guarantee a specific precision, meaning the absolute error $|\bar{X}−\mu|$ is at most a target value $\beta$, an **iterative (sequential) procedure** is required, allowing additional replications until the confidence interval is sufficiently narrow.

<aside>
📌

**Sequential Procedure**

We treat the sample size $n$ as a variable that increases until the precision requirement is reached:

1. Start with an initial number of replications $n_0$ (e.g., $n_0\ge10$)
2. Run the simulation and compute the sample mean $\bar{X}(n)$ and sample variance $S^2(n)$
3. Compute the current confidence interval half-width: $\delta(n,\alpha)=t_{n-1,1-\alpha/2}\sqrt{\frac{S^2(n)}{n}}$
4. Check Condition:
    - If $\delta(n,α)\le\beta$, the precision is satisfied. The process stops, and $\bar{X}(n)$ is used as the point estimate.
    - If $\delta(n,\alpha)>\beta$, the interval is too wide. Increment $n$ by 1, perform an additional replication, and repeat the evaluation.

**N.B.** Alternatively, to avoid checking after every single run, one can estimate the required optimal sample size $n^*$ by solving the inequality $t_{i-1, 1-\alpha/2} \sqrt{S^2(n)/i} \le \beta$ for $i$, assuming the variance remains stable.

</aside>

---

# 6. Output Analysis for non-terminating Simulations

In non-terminating simulations, our goal is often to estimate a **steady-state parameter $\phi$** (such as the average delay $ν=E[Y]$). We assume that as time tends to infinity ($i\rightarrow\infty$), the system's output distribution $F_i(y)$ converges to a **steady-state distribution** $F(y)$.

## 6.1. Transient vs Steady-State

To understand why standard analysis fails initially, we must distinguish between two phases of the process:

- **Transient Distribution $F_i(y∣I)$** → depends on the initial conditions $I$ and time $i$. It governs the system during the initial "warm-up" phase.
- **Steady-State Distribution $F(y)$** → as $i\rightarrow\infty$, the distribution becomes independent of the initial conditions. The process acquires a stable probabilistic behavior.

## 6.2. **Startup problem (Initial Transient)**

A major difficulty in steady-state analysis is that the initial observations $Y_1,Y_2,…$ are governed by the **transient distribution**, which depends heavily on the initial conditions $I$ rather than the steady-state distribution.

Consequently, an estimator based on all observations like the simple sample mean $\bar{Y}(m)$ is **biased** for finite sample sizes. The expected value of the sample mean does not equal the true steady-state mean $E[\bar{Y}(m)]\neν$ because the initial "cold" period drags the average down (or up).

![image.png](14%20Output%20Data%20Analysis/image.png)

## 6.3. Warming Up the Model

To correct this bias, the standard technique is **"warming up"** the model. We define a cutoff point $l$ (the length of the warm-up period) and **discard** all data collected prior to this point. We estimate the mean using only the remaining $m−l$ observations:

$$
\bar{Y}(m,l)=\frac{\sum_{i=l+1}^m Y_i}{m-l}
$$

<aside>
📌

**Choosing the Warm-Up Period ($l$)**

Choosing the correct $l$ involves a trade-off:

- **If $l$ is too small** → the bias remains because we are still including transient data ($E[\bar{Y}(m)]\neν$).
- **If $l$ is too large** → we discard valid steady-state data, reducing the sample size. This unnecessarily increases the **variance** of the estimator (making the confidence interval wider).
</aside>

## 6.4. Welch’s Procedure

Welch’s procedure is a graphical method used to determine the optimal $l$ by visualizing when the system "flattens out" to its steady state. Since a single simulation run is too "noisy" to judge visually, we average across multiple runs.

<aside>

**Algorithm:**

1. Perform $n$ replications ($n\ge5$) of the simulation, each of length $m$ (where $m$ is large). Let $Y_{ji}$ be the value at observation (=time step) $i$ in replication $j$.
2. Compute the average $\bar{Y}_i$ for each time step $i$ across all $n$ replications:
    
    $$
    \bar{Y}_i=\sum_{j=1}^n \frac{Y_{ji}}{n}
    $$
    
    This produces a single averaged trajectory with variance reduced by a factor of $n$:  $Var(\bar{Y}_i)=Var(Y_{i})/n$.
    
3. Apply a **moving average** using a window of size $w$ to smooth fluctuations:
    
    $$
    \overline{Y}_i(w) = \begin{cases}     \frac{\sum_{s=-w}^{w} \overline{Y}_{i+s}}{2w + 1} & \text{if } i = w+1, \dots, m-w \\[15pt]    \frac{\sum_{s=-(i-1)}^{i-1} \overline{Y}_{i+s}}{2i + 1} & \text{if } i = 1, \dots, w\end{cases}
    $$
    
    The smoothed value $\bar{Y}_i(w)$ is the average of observations from $i−w$ to $i+w$.
    

![image.png](14%20Output%20Data%20Analysis/image%201.png)

![image.png](14%20Output%20Data%20Analysis/image%202.png)

1. Plot $\overline{Y}_i(w)$ against time $i$, and choose $l$ as the point after which the curve appears to have **converged or stabilized**.
    
    ![image.png](14%20Output%20Data%20Analysis/1e546716-ff79-47b6-9adc-2c18c92ffd80.png)
    
</aside>

---

# 7. Output Analysis for Terminating Episodes with Multiple Measures of Performance

Real-world simulations often track multiple performance metrics simultaneously (e.g., queue length, wait time, utilization). If we build a separate confidence interval for each metric with confidence level $1 - \alpha$, the probability that **all intervals are simultaneously correct** becomes smaller than $1 - \alpha$. To maintain an **overall** confidence level, **Bonferroni’s Inequality** established the probability that all the $k$ true measures falls into the corresponding confidence interval:

$$
P(\mu_S\in I_S\ \forall s=1,2,...,k)\ge1-\sum_{s=1}^k\alpha_s
$$

<aside>
📌

**How to choose $\alpha$?**

Starting from Bonferroni’s inequality: 

$$
1-\sum_{s=1}^k \alpha_s\ge1-\alpha
$$

to guarantee an **overall confidence level** of $1 - \alpha$, we must set the individual $\alpha_s$ such that:

$$
\sum_{s=1}^k \alpha_s\le\alpha
$$

Therefore, we first **choose the desired overall significance level** $\alpha$ and then **allocate it across the** k **confidence intervals** by selecting appropriate values of $\alpha_s$. A common and simple choice is to assign them uniformly:

$$
\alpha_s=\frac{\alpha}{k},\quad s=1,...,k
$$

**N.B.** This method is practical only for a small number of measures ($k\le10$).

</aside>

---