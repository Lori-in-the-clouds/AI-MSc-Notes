# 15. Comparing Systems in Simulation

Num of pages: 1
Status: Done
Type: theory

# 1. The Statistical Challenge

When comparing two different system configurations, simply comparing the results of single simulation runs is incorrect because simulations are random processes.

<aside>

*Example:*

Consider a bank simulation with a customer arrival rate of $1$ per minute:

- **System A** → an $M/M/1$ queue with a mean service time of $0.9$ minutes ($\rho=0.9$).
- **System B** → An $M/M/2$ queue with a mean service time of $1.8$ minutes ($\rho=0.9$).
- **Performance measures of interest** → the expected average delay in the queue for the first $100$ customers.

If an analyst performs only **one simulation run** of 100 customers for each system, using independent random numbers to decide which system performs better:

- **True Values** → the actual expected delays are $d_B(100)=3.70$ and $d_A(100)=4.13$, meaning System $B$ is truly better.
- **Experimental Result** → repeating this single-run comparison 100 times shows that System B is correctly identified as better ($d_B < d_A$) only **48% of the time.**

![Screenshot 2025-12-26 at 16.12.53.png](15%20Comparing%20Systems%20in%20Simulation/Screenshot_2025-12-26_at_16.12.53.png)

So, in **52% of the cases** the analyst would make the wrong recommendation based on a single run. Therefore, comparing systems by subtracting performance measures obtained from single simulation runs is **statistically invalid**.

</aside>

---

# 2. Statistical Methods for Comparing Two Systems

To compare the performance of two systems’ random variables metrics, it is **not statistically correct** to evaluate each metric separately and then subtract them. Instead, the correct procedure is to analyze the **difference between the two metrics as a single random variable** and construct **one confidence interval for this difference**:

- **Paired-t confidence interval ($n_1=n_2$):**
    1. Pair the observations $X_{1j}$ and $X_{2j}$ to define $Z_j=X_{1j}−X_{2j}$ for each replication $j$. The $Z_j$ values are IID random variables.
    2. Compute the sample mean $\bar{Z}(n)$ and sample variance $Var[\bar{Z}(n)]$ using standard formulas.
    3. Construct the confidence interval:
        
        $$
        \bar{Z}(n)\pm t_{n-1,1-\alpha/2}\sqrt{Var[\bar{Z}(n)]}
        $$
        
- **Welch Confidence Interval ($n_1\ne n_2$):** if the number of replications differs, observations cannot be paired. We must assume that $X_{ij}$’s are normally distributed and that $X_{1j}$ and  $X_{2j}$ are independent.
    1. Compute sample means $\bar{X}_i(n_i)$ and variances $S_i^2(n_i)$ separately for each system:
        
        $$
        \bar{X}_i(n_i)=\frac{\sum_{j=1}^{n_i}X_{ij}}{n_i} \qquad S_i^2(n_i)=\frac{\sum_{j=1}^{n_i}[X_{ij}-\bar{X}_i(n_i)]^2}{n_i-1}
        $$
        
    2. Compute the effective degrees of freedom $\hat{f}$:
        
        $$
        \displaystyle \hat{f}=\frac{\left[\frac{S_1^2(n_1)}{n_1}+\frac{S_2^2(n_2)}{n_2}\right]^2}{\frac{S_1^2(n_1)/n_1}{n_1-1} + \frac{S_2^2(n_2)/n_2}{n_2-1}} = \frac{[A + B]^2}{\frac{A^2}{n_1 - 1} + \frac{B^2}{n_2 - 1}}
        
        $$
        
    3. Construct the confidence interval:
        
        $$
        \bar{X}_1(n_1)-\bar{X}_2(n_2)\pm t_{\hat{f},1-\frac{\alpha}{2}}\sqrt{\frac{S_1^2(n_1)}{n_1}+\frac{S_2^2(n_2)}{n_2}}
        $$
        

---

# 3. Multi-System Comparison

When comparing more than two systems, the error probability accumulates. We use the **Bonferroni Inequality** to adjust the confidence level. To guarantee a global confidence level of $1 - \alpha$ across a total of $c$ simultaneous comparisons, we must split the total error budget $\alpha$ equally. For this reason, each individual confidence interval must be constructed using a confidence level of $1 - \alpha/c$.

- **Comparison with a Standard** → to compare $k$ systems where one is a "standard" (System 1) and the others are variants:
    1. **Number of Comparisons ($c$):** construct $k−1$ confidence intervals for the differences ($μ_i−μ_1$).
    2. **Adjusted alpha:** $\alpha_s=\alpha/(k−1)$.
    3. **Interpretation:**
        - If the interval **contains 0** the system is statistically similar to the standard.
        - If the interval **misses 0** the system is significantly different from the standard.
- **All Pairwise Comparisons** → to compare every system against every other system:
    1. **Number of Comparisons ($c$):** construct $k(k−1)/2$ confidence intervals for the differences ($\mu_i-\mu_j$).
    2. **Adjusted alpha:**  $\alpha_s=\frac{\alpha}{[k(k−1)/2]}$.
    3. **Interpretation:**
        - If the interval **contains 0** the two systems being compared are statistically similar.
        - If the interval **misses 0** the two systems are significantly different.

---

# 4. Variance Reduction: Common Random Numbers (CRN)

We want to be sure that we are detecting differences due to different configurations of system and not variability of the stochastic process.

**Common Random Numbers (CRN)** achieves this by using the **same random seeds and random streams** for corresponding events (e.g., arrivals and service times) in both systems. This creates a **positive correlation** between the observations. Let $X_{1j}$ and $X_{2j}$ be the performance measures from replication $j$, and let $Z_j = X_{1j} - X_{2j}$. The variance of the estimator of the mean difference is:

$$
Var[\bar{Z}(n)] = \frac{Var(Z_j)}{n} = \frac{Var(X_{1j}) + Var(X_{2j}) - 2Cov(X_{1j}, X_{2j})}{n}
$$

Because CRN makes $Cov(X_{1j},X_{2j}) > 0$, the variance of the difference decreases. As a result, **confidence intervals become narrower**, allowing for a more precise and reliable comparison between systems.

---