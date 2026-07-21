# 4. Model Free Prediction

Num of pages: 2
Status: Done
Type: theory

# 1. Introduction

**Model-Free Learning** is a branch of Reinforcement Learning where the agent learns **without a model of the environment**. In particular, **Model-Free Prediction** aims to estimate the **value function** $v_\pi$ of an unknown Markov Decision Process (MDP). 

---

# 2. Monte Carlo Learning

Monte Carlo (MC) methods are **model-free** approach that learn from **complete episodes**. ****They estimate state values from observed returns **without bootstrapping**.

## 2.1. How it works?

The value of each state is estimated by averaging the returns observed across all episodes:

1. For a given state $s$, track:
    - $N(s)$ → number of times the state is visited
    - $S(s)$ → sum of returns observed from those visits
2. The estimated value is the mean return: $V(s)=\frac{S(s)}{N(s)}$
3. By the law of large numbers, this estimate converges to the true value function as the number of visits approaches infinity: $V(s)\rightarrow v_\pi(s)\ \text{as}\ N(s)\rightarrow \infty$

<aside>
📌

**MC Variations:**

There are two common variations depending on how multiple visits per episode are handled:

- **First-Visit MC** → computes the return $G_t$ using **only the first time** a state is visited in an episode. Any subsequent visits to that state in the same episode are ignored.
- **Every-Visit MC** →  computes a separate return $G_t$ for **every single visit** to a state within the episode, treating each pass as a new data point.
</aside>

## 2.2. Incremental Updates and Learning Rates

Instead of storing all returns and re-computing the mean at the end, MC methods use an **incremental mean update**. This allows the value estimate to be updated efficiently after each episode:

$$
\mu_k = \frac{1}{k} \sum_{j=1}^{k} x_j =\frac{1}{k} \left( x_k + \sum_{j=1}^{k-1} x_j \right) = \frac{1}{k} \left( x_k + (k-1)\mu_{k-1} \right)= \mu_{k-1} + \frac{1}{k}(x_k - \mu_{k-1})
$$

This gives us the general update rule: 

$$
\text{New\_Estimate}=\text{Old\_Estimate} + \text{StepSize}\cdot(\text{Target}- \text{Old\_Estimate})
$$

For MC, the **Target** is the total return $G_t$. The choice of **StepSize** defines two different update rules:

- **Update with Decreasing Learning Rate** → the estimate is updated using a step size that decreases with the number of visits:
    
    $$
    V(S_t)\leftarrow V(S_t)+\frac{1}{N(S_t)}(G_t-V(S_t))
    $$
    
- **Update with Fixed Learning Rate ($\alpha$)** → in non-stationary environments, a constant step size gives more weight to recent returns:
    
    $$
    V(S_t)\leftarrow V(S_t)+\alpha(G_t-V(S_t))
    $$
    
    <aside>
    🔑
    
    **Exponential Decay of Past Returns**
    
    This update is a form of **Exponentially Weighted Moving Average**. More recent returns have a larger influence, while older returns gradually become less important because their weights decrease **exponentially**. Let's consider three returns over time: $G_1$ (the oldest), $G_2$, $G_3$ (the most recent):
    
    - After $G_1$ → $V_1 = \alpha G_1$
    - After $G_2$ → $V_2= (1-\alpha)V_1+\alpha G_2=\alpha(1-\alpha) G_1+\alpha G_2$
    - After $G_3$ → $V_3 = (1-\alpha) \cdot V_2 + \alpha \cdot G_3 = (1-\alpha) \cdot [\alpha (1-\alpha) G_1 + \alpha G_2] + \alpha G_3 = \textcolor{red}{\alpha(1-\alpha)^2}\cdot G_1+\textcolor{orange}{\alpha(1-\alpha)}\cdot G_2+\textcolor{green}{\alpha} \cdot G_3$
    
    The most recent return, $G_3$, has the largest weight $\textcolor{green}\alpha$, while the oldest return, $G_1$, has its weight reduced by a factor of $\textcolor{red}{(1−\alpha)^2}$.
    
    </aside>
    

---

# 3. Temporal Difference Learning

**Temporal-Difference (TD)** methods improve sample efficiency by learning **online** from **incomplete episodes**. TD methods **bootstrap**, meaning they update value estimates using an estimate of the next state’s value.

## 3.1. TD(0) Algorithm

**TD(0)** (or one-step TD) is the simplest TD algorithm. It updates the value function $V(S_t)$ towards an **estimated return** based on the immediate reward and the estimated value of the *next* state $V(S_{t+1})$:

$$
V(S_t)←V(S_t)+\alpha[R_{t+1}+\gamma V(S_{t+1})−V(S_t)]
$$

Where:

- $R_{t+1}​+\gamma V(S_{t+1}​)$ is the **TD target**, it’s the **best one-step hypothesis** that the agent can make about the true value of state $S_t$.
- $R_{t+1}​+\gamma V(S_{t+1}​)-V(S_t)$ is the **TD error**, it ****computes the **difference** between what the agent **actually observed** in that single step and what it **expected to obtain**.
- $\alpha$ is the **learning rate**.

---

# 4. MC vs TD

|  | **MC** | **TD** |
| --- | --- | --- |
| **Core Method** | Learns only from **complete episodes** | Learns from **incomplete episodes** (online, step-by-step). |
| **Bootstrapping** | **No**. It updates a value based on the final, true return. | **Yes**. It updates a value estimate using a subsequent *estimate*. |
| **Update Target** | Depends on the **full actual return $G_t$** | Depends on the **estimated next state value** $V(S_{t+1})$ |
| **Bias** | **Zero Bias** ($G_t$ is an unbiased estimate) | **Some Bias**. The TD target is a biased estimate because it relies on the current, imperfect estimate $V(S_{t+1})$. |
| **Variance** | **High Variance**. Returns depend on many random transitions and actions | **Lower Variance**. The TD target depends on only one random reward and transition. |
| **Sample Efficiency** | **Sample Inefficient**. Often requires hundreds of thousands of episodes for accurate estimates. | **More Sample-Efficient**. The lower variance allows it to learn more quickly from each step. |
| **Environments** | Only works for **episodic** (terminating) MDPs. | Works for both episodic and **continuing** (non-terminating) MDPs. |
| **Markov Property** | Does not fully exploit it (and can be used in non-Markov environments). | **Exploits the Markov property** (making it more efficient in Markov environments). |

**N.B.** It’s more easy to solve the bias than the variance

---

# 5. The Spectrum of the Backup Methods

<aside>
💡

**BootStrapping** 

Bootstrapping is when an update involves an estimate. Both TD and DP use bootstrapping, while MC does not.

</aside>

<aside>
💡

**Backup**

The **Backup** operation is the core mechanism by which all Reinforcement Learning (RL) algorithms propagate value information from one state to its preceding states. It determines *how* the value of a state is updated based on the values of its successor states and the observed rewards.

</aside>

The **Unified View of Reinforcement Learning** places different algorithms on a spectrum based on two dimensions:

- **Backup Type (Full vs. Sample)**:
    - **Full Backups:** are methods that compute an exact value by considering **all** possible next states and actions (e.g., Dynamic Programming).
    - **Sample Backups:** are methods that compute a value using a **single branch/trajectory** from the environment (e.g., Monte Carlo).
- **Backup Depth (Shallow vs. Deep)**:
    - **Shallow Backups:** are methods that bootstrap from an estimate one step ahead. They don't wait for a final outcome (e.g., TD Learning).
    - **Deep Backups:** are methods that don't bootstrap. They wait until the end of an episode to get the final outcome before they update their value (e.g., Monte Carlo).

![image.png](4%20Model%20Free%20Prediction/image.png)

<aside>
📌

**Distinction between Methods**

![image.png](4%20Model%20Free%20Prediction/image%201.png)

</aside>

## 5.1. N-step Prediction

N-step prediction offers a **compromise between the high variance of MC and the high bias of TD(0)**. It represents a spectrum that ranges from TD(0) to MC. 

The **N-step return $G_t^{(n)}$** combines $n$ actual rewards with a bootstrapped value from the $n$-th step. The value function is then updated towards this return.

- The generalized N-step return is: $G_t^{(n)}=\overbrace{R_{t+1}+γR_{t+2}+⋯+γ^{n−1}R_{t+n}}^{n\ \text{actual rewards}}+γ^n\overbrace{V(S_{t+n})}^{n-th\ \text{value}}$
- The N-step temporal-difference learning update is: $V(S_t)←V(S_t)+α(G_t(n)−V(S_t))$

Here's how different values of $n$ correspond to different methods:

- When $n=1$, the return is the **TD Target** $G_t^{(1)}=R_{t+1}+γV(S_{t+1})$.
- When $n=\infty$, the return is the **full MC return** $G_t^{(\infty)}$.

![image.png](4%20Model%20Free%20Prediction/image%202.png)

---

# 6. TD($\lambda$)

TD($\lambda$) is a powerful method that unifies all $N$-step approaches by using a weighted average. The $\lambda$**-return $G_t^{\lambda}$** is a weighted average of all n-step returns, where the weights decay geometrically based on a parameter $λ∈[0,1]$:

$$
G_t^{\lambda}=(1-\lambda)\sum_{n=1}^\infty \lambda^{n-1}G_t^{(n)}
$$

 Where:

- $(1-\lambda)$ ensures that the sum of all weights is $1$ [since $\sum_{n=1}^\infty \lambda^{n-1}=\frac{1}{(1-\lambda)}$ (→ geometric series)].
- $\lambda^{n-1}$ makes the weight of a return decrease exponentially as $n$ increases.

<aside>
📌

**The Role of the value** $\lambda$:

The value of $\lambda$ determines the balance between bootstrapping and relying on the final outcome:

- if $\lambda=0$, the weight is 1 only for $n=1$, and zero for all other N-step returns. The formula reduces to TD(0) alone.
- If $\lambda=1$, the weight does not decay, and the formula combines all returns. The result is equivalent to a Monte Carlo approach.
- For a value of $\lambda$ between 0 and 1, the agent gives weight to all experiences, but it trusts the rewards and transitions closer in time more.
</aside>

## 6.1. Forward-View vs Backward-View

There are two views for implementing TD($\lambda$):

- **Forward-View TD($\lambda$)** → this is the **theoretical view**. It computes the $\lambda$-return by looking into the future to the end of the episode. Like MC, it can only be applied offline using complete episodes.
    
    ![image.png](4%20Model%20Free%20Prediction/image%203.png)
    
- **Backward-View TD($\lambda$)** → this is the **practical**, **online mechanism**. It provides an online algorithm to update every step using incomplete sequences. This is achieved using **eligibility traces**.
    
    ![image.png](4%20Model%20Free%20Prediction/image%204.png)
    

---

# 7. Credit Assignment Problem and Eligibility Traces

<aside>
🚨

**Credit Assignment Problem**

The challenge of determining which specific past action(s) are responsible for a received reward. Rewards are often **sparse** (rewards are rare or infrequent) and **delayed.**

</aside>

## 7.1. Eligibility Traces

**Eligibility Traces** are the mechanism used to solve this problem. An eligibility trace $E_t(s)$ is a temporary record, assigned to each state $s$. This counter tracks how "eligible" a state is to receive credit for a future reward. This mechanism simultaneously implements two key heuristics:

- **Recency Heuristic** → assigns credit to the most recent states.
- **Frequency Heuristic** → assigns credit to the most frequent states.

### 7.1.1. How It works?

The eligibility trace for every state $s$ is initialized to zero $E_0(s)=0$ and is updated at every time step $t$ using the following formula:

$$
E_t(s)=γλE_{t−1}(s)+\mathbb{1}(S_t=s)
$$

This formula is composed of two parts:

- **Decay** → the $γλE_{t−1}(s)$ term. It multiplies the previous trace value $E_{t−1}(s)$  by a discount factor $\gamma$ and a trace decay parameter $\lambda$. Since both $\gamma$ and $\lambda$ are less than 1, the traces of **all** states decay exponentially by a factor of $\gamma \lambda$ at every single time step, implementing the **recency heuristic**.
- **Increment** → the $\mathbb{1}(S_t=s)$ term. This is an indicator function that equals 1 *only* for the state $s$ that is the current state $S_t$. This assign credit to the most frequent states, implementing the **frequency heuristic**.

### 7.1.2. Application: Backward-View TD($\lambda$)

The backward-view TD($\lambda$) algorithm update the value of **every** state in proportion to both the TD-error and its eligibility trace. This allows a single TD error to be "broadcast" back to recent and frequent states, providing a practical mechanism for the credit assignment problem:

$$
\delta_t=R_{t+1}+\gamma V(S_{t+1})−V(S_t)\qquad
V(s)←V(s)+α\delta_t\textcolor{orange}{E_t(s)}
$$

Where $\delta_t$ is the **TD error**:

- $\delta_t > 0$‬‭‬ → the outcome was **better than expected** (**positive surprise**).
- $\delta_t < 0$ → ****the outcome was **worse than expected** (**negative surprise**).

---