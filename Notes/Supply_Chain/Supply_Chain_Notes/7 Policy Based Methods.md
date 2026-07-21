# 7. Policy Based Methods

Num of pages: 2
Status: Done
Type: theory

# 1. Value-Based vs. Policy-Based Methods

Previously, we studied **Value-Based methods**, where we learned the action-value function $Q(s,a)$ and derived a policy from it (e.g., using an $\epsilon$-greedy strategy).

Now, we introduce **Policy-Based methods**, which learn the policy directly: $\pi_{\mathbf{w}}(a|s)=P(a|s,\mathbf{w})$. Instead of estimating action values, the model directly learns which action to take in each state. These two approaches can be combined in **Actor-Critic methods**, where:

- **Actor** → learns the policy.
- **Critic** → learns a value function used to evaluate and improve the policy.

![image.png](7%20Policy%20Based%20Methods/image.png)

<aside>
📌

**Advantages:**

- **Better convergence properties:**
    - Value-Based (`argmax`): small changes in $Q(s,a)$ can suddenly switch the selected action, causing unstable training.
    - Policy-based methods: Optimize the policy directly with gradient ascent, producing smooth changes in action probabilities and more stable convergence.
- **Continuous Action Spaces:** value-based methods (like Q-learning) cannot handle continuous action spaces because you cannot perform an `argmax` over an infinite number of actions. Policy-based methods work naturally by outputting the parameters of a continuous distribution (like a Gaussian).
- **Can Learn Stochastic Policies**

**Disadvantages:**

- They typically converge to a **local optimum**, not necessarily the global one.
- The evaluation of a policy is often inefficient and has **high variance**, which makes learning slow.
</aside>

---

# 2. The Need for Stochastic Policies

Value-based methods almost always produce a deterministic policy (i.e., "in this state, always do *this*") . Policy-based methods can learn **stochastic policies** (i.e., "in this state, 50% chance of A, 50% chance of B"), which are essential in two key scenarios:

- **Exploitable Opponents (Game Theory)** → in a game like Rock-Paper-Scissors, any deterministic policy (e.g., "always play Rock") is easily exploited. The optimal solution is the **Nash Equilibrium**, which is a uniform random policy (1/3 Rock, 1/3 Paper, 1/3 Scissors).
- **Aliased States (POMDPs)** → this occurs when the agent cannot distinguish between two or more states (e.g., two identical-looking grey corridor squares). If the optimal action in one state is "*Go Left*" and in the other is "*Go Right*," a deterministic policy will get stuck . The optimal stochastic policy is "50% Left, 50% Right," which allows the agent to eventually escape.
    
    ![image.png](7%20Policy%20Based%20Methods/image%201.png)
    

---

# 3. Policy Optimization and Gradient Ascent

## 3.1. The Objective Function $J(\mathbf{w})$

Our goal is to find the parameters $\mathbf{w}$ that maximize an **objective function $J(\mathbf{w})$**, which measures the quality of our policy $\pi_\mathbf{w}$. We can measure the quality of $\pi_\mathbf{w}$ in different ways:

- In episodic environments we can use:
    - **Start value**, the expected value from the start state → **$J(\mathbf{w})=V^{π_\mathbf{w}}(s_1)$**
- In continuing environments we can use:
    - **Average value** of all states weighted by how often you visit them → $J​(\mathbf{w})=\sum_s ​d^{π_\mathbf{w}}​(s)V^{π_\mathbf{w}}​(s)$
    - **Average reward per time-step** *→ $J(\mathbf{w})=\sum_s ​d^{π_\mathbf{w}}​(s)\sum_a π_\mathbf{w}​(s,a)R_S^a$,* where $d^{\pi_\mathbf{w}}(s)$ is the probability to be in state $s$. This is the meaning of a stationary distribution for a Markov Chain $\pi_{\mathbf{w}}$.

## 3.2. Policy Optimization with Gradient Ascent

The objective is to maximize $J(\mathbf{w})$. Therefore, we update the policy parameters using **gradient ascent**, moving $\mathbf{w}$ **in the direction of the gradient**:

$$
Δ\mathbf{w}=\alpha \nabla_\mathbf{w}J(\mathbf{w}) \quad \text{where}\ \nabla_{\mathbf{w}}J(\mathbf{w}) = \begin{pmatrix} \frac{\partial J(\mathbf{w})}{\partial w_1} \\ \vdots \\ \frac{\partial J(\mathbf{w})}{\partial w_n} \end{pmatrix}

$$

<aside>
🚨

**Problem:** 

Directly computing $\nabla_\mathbf{w}J(\mathbf{w})$ is difficult because we need to compute also the gradient of 

$d^{\pi_\mathbf{w}}(s)$ that depend on the transition model $\mathcal{P}_{ss'}^a$ which is unknown. This problem will be resolved with the **Policy Gradient Theorem**.

</aside>

To rewrite the policy gradient $\nabla_\mathbf{w}\pi_\mathbf{w}(a|s)$ in a form that can be estimated from sampled trajectories, we use the **Likelihood Ratio Trick**, it rewrites the policy gradient using the logarithmic derivative identity $\frac{d}{dx}​\log(f(x))=\frac{1}{f(x)}​⋅f'(x)$:

$$
\nabla_\mathbf{w}\pi_\mathbf{w}(s,a)=\pi_\mathbf{w}(s,a)\frac{\nabla_\mathbf{w}\pi_\mathbf{w}(s,a)}{\pi_\mathbf{w}(s,a)}=\pi_\mathbf{w}(s,a)\nabla_\mathbf{w}\log(\pi_\mathbf{w}(s,a))
$$

Where $\nabla_\mathbf{w}\log(\pi_\mathbf{w}(s,a))$ is the **score function**.

<aside>
📌

**Why Likelihood Ratio Trick is important?**

Without the Likelihood Ratio Trick, computing the policy gradient would require knowing the effect of **all possible actions**. This is impractical because the agent only observes the reward of the action it actually selects, not the rewards of all other possible actions. The Likelihood Ratio Trick allows the agent to estimate the gradient directly from its own experience:

1. The agent samples an action according to its policy: $a\sim\pi_\mathbf{w}(a|s)$
2. The environment returns a reward $R_t$
3. The agent updates the policy using: $\nabla\log\pi(a|s)R_t$

If the action produces a high reward, its probability is increased. If it produces a low reward, its probability is reduced.

</aside>

In a simplified one-step MDP, where an action immediately produces a reward and the episode terminates, the gradient becomes:

$$
\nabla_{\mathbf{w}}J(\mathbf{w}) = \sum_{s \in \mathcal{S}} d(s) \sum_{a \in \mathcal{A}} \pi_{\mathbf{w}}(s,a) \nabla_{\mathbf{w}} \log (\pi_{\mathbf{w}}(s,a))R(s,a) = \mathbb{E}_{\pi_{\mathbf{w}}} \left[ \nabla_{\mathbf{w}} \log \pi_{\mathbf{w}}(s,a) \, R_t \right]
$$

<aside>
✅

**Policy Gradient Theorem:**

The previous derivation considers only a one-step MDP. For an $n$-step MDP, the immediate reward is replaced by the expected future return, represented by the action-value function $Q^\pi(s,a)$:

$$
∇_\mathbf{w}J(\mathbf{w})=\mathbb{E}_{π_{\mathbf{w}}}[∇_\mathbf{w}\log\pi_\mathbf{w}(s,a)\textcolor{red}{Q_\pi(s,a)}]
$$

The important result is that the gradient no longer depends explicitly on $\nabla_\mathbf{w}d^\pi(s)$ and only requires quantities that can be estimated from sampled trajectories.

</aside>

---

# 4. Common Policy Parametrizations

How we define $\pi_\mathbf{w}$ depends on the action space:

- **Discrete Actions (Softmax Policy)** → we use a **softmax function** on top of our features:
    
    $$
    \pi_{\mathbf{w}}(s, a) = \frac{e^{\mathbf{x}(s, a)^T \mathbf{w}}}{\sum_{a' \in \mathcal{A}} e^{\mathbf{x}(s, a')^T \mathbf{w}}}
    $$
    
    This outputs a valid probability distribution (all outputs sum to 1) for a discrete set of actions. The score function is:
    
    $$
    {\color{blue}\nabla_\mathbf{w}	\ln(\pi_\mathbf{w}(a,s))}=
    \nabla_\mathbf{w}\left[ \ln(e^{\mathbf{x}(s, a)^T \mathbf{w}})-\ln\left(\sum_{a' \in \mathcal{A}} e^{\mathbf{x}(s, a')^T \mathbf{w}}\right)\right] 
    
     =\mathbf{x}(s,a)^T -\frac{1}{\sum_{a' \in \mathcal{A}} e^{\mathbf{x}(s, a')^T \mathbf{w}}}\cdot \nabla_\mathbf{w}\left(\sum_{a' \in \mathcal{A}} e^{\mathbf{x}(s, a')^T \mathbf{w}}\right)= 
    
    \mathbf{x}(s,a)^T -\frac{1}{\sum_{a' \in \mathcal{A}} e^{\mathbf{x}(s, a')^T \mathbf{w}}}\cdot \left(\sum_{a' \in \mathcal{A}} e^{\mathbf{x}(s, a')^T \mathbf{w}}\mathbf{x}(s,a')^T\right)= 
    
    \\
    \
    \\
    
    =\mathbf{x}(s, a)^T - \sum_{a'\in \mathcal{A} } \left( \frac{e^{\mathbf{x}(s, a')^T \mathbf{w}}}{\sum_{a''} e^{\mathbf{x}(s, a'')^T \mathbf{w}}} \right) \mathbf{x}(s, a')^T = 
    
     \mathbf{x}(s, a)^T-\sum_{a'}\pi_\mathbf{w}(s,a')\cdot\mathbf{x}(s,a')^T={\color{blue}\mathbf{x}(s, a)^T - \mathbb{E}_{a'\sim \pi}\left[\mathbf{x}(s,a')^T\right]}
    
    $$
    
- **Continuous Actions (Gaussian Policy)** → we use our function approximator (e.g., a neural network) to output the parameters ($\mu$ and $\sigma$) of a **Gaussian distribution**:
    
    $$
    \pi_\mathbf{w}(a,s)=\frac{1}{\sigma \sqrt{2\pi}}\cdot e^{-\frac{\left(a-\mu(s)\right)^2}{2\sigma^2}}
    $$
    
    Where the mean is defined as a linear combination of state features: $\mu(s)=\mathbf{x}(s)^T\mathbf{w}$ and the variance $\sigma^2$ can be fixed or learned. The policy then *samples* an action from this distribution $a∼N(μ(s),σ^2)$, a higher variance leads to more exploration. The score function is:
    
    $$
    \nabla_\mathbf{w}	\ln(\pi_\mathbf{w}(a,s))= 
    
    \nabla_\mathbf{w}\left[\ln\frac{1}{\sigma\sqrt{2\pi}}-\frac{(a-\mu(s))^2}{2\sigma^2}\right]=
    
    0-\frac{1}{2\sigma^2}\left[2(a-\mathbf{x}(s)^T\mathbf{w})\cdot(-\mathbf{x}(s))\right]=\frac{(a-\mathbf{x}(s)^t\mathbf{w})\cdot\mathbf{x}(s)}{\sigma^2}
    $$
    

---

# 5. Policy Gradient Algorithms

## 5.1. REINFORCE (Monte-Carlo Policy Gradient)

REINFORCE is the simplest implementation of **Policy Gradient theorem**. It runs an episode and uses the **full episodic return $G_t$** as an unbiased s*ample* of $Q_\pi(s_t,a_t)$. It states that for any of the objective functions, the policy gradient is:

$$
\Delta\mathbf{w}_t=\alpha\nabla_\mathbf{w}\log\pi_\mathbf{w}(s_t,a_t)\textcolor{red}{G_t}
$$

<aside>
🔢

**Algorithm:**

![image.png](7%20Policy%20Based%20Methods/image%202.png)

</aside>

**Problem:** although the return $G_t$ provides an unbiased estimate, its **high variance** makes the Monte-Carlo policy gradient learning process slow and inefficient.

## 5.2. Actor-Critic Policy Gradient

To fix the high variance of REINFORCE, we introduce **Actor-Critic** methods. We learn *two* sets of parameters:

1. **The Actor ($\pi_\theta$)** → this is the policy. It updates its parameters $\theta$ in the direction suggested by the critic.
2. **The Critic ($Q_\mathbf{w}$)** → this is a value-function approximator. It learns and estimates $Q_\mathbf{w}$ to "criticise" the actor's actions.

The critic's job is just policy evaluation, which we can solve using MC, TD(0), or TD(λ). The use of TD makes the $Q$-value estimate $Q_\mathbf{w}$ a *biased* but much *lower-variance* signal than the full $G_t$ return. It follows an **approximate** policy gradient:

$$
\nabla_\theta J(\theta)\approx\mathbb{E}_{\pi_\theta}[\nabla_\theta\log\pi_\theta(s,a)\textcolor{red}{Q_\mathbf{w}(s,a)}]\qquad \Delta\theta=\alpha\nabla_\theta\log\pi_\theta(s,a)\textcolor{red}{Q_\mathbf{w}(s,a)}
$$

<aside>
🔢

**Algorithm:**

![image.png](7%20Policy%20Based%20Methods/image%203.png)

</aside>

---

# 6. Reducing Variance: Baselines and Advantage

Policy gradient methods can have **high variance**, which makes learning unstable. A common solution is to use **baselines** and **advantage estimation**.

## 6.1. Baseline

We can subtract a **baseline function $B(s)$** from our Q-value. This can reduce variance without changing the expectation:

$$

{\color{blue}\mathbb{E}_{\pi_\theta}\left[\nabla_\theta\log\pi_\theta(s,a)\left(Q(s,a)-B(s)\right)\right]}=  
\mathbb{E}_{\pi_\theta}\left[\nabla_\theta\log\pi_\theta(s,a)Q(s,a) \right]-
\mathbb{E}_{\pi_\theta}\left[\nabla_\theta\log\pi_\theta(s,a)B(s) \right]= 

\mathbb{E}_{\pi_\theta}\left[\nabla_\theta\log\pi_\theta(s,a)Q(s,a) \right] - \sum_{s\in \mathcal{S}} d^{\pi_\theta}(s)\sum_a\pi_\theta(s,a)\nabla_\theta\log\pi_\theta(s,a)B(s) =
\\ \ \\

=\mathbb{E}_{\pi_\theta}\left[\nabla_\theta\log\pi_\theta(s,a)Q(s,a) \right] - \sum_{s\in \mathcal{S}} d^{\pi_\theta}(s)\sum_a\nabla_\theta\pi_\theta(s,a)B(s)= \mathbb{E}_{\pi_\theta}\left[\nabla_\theta\log\pi_\theta(s,a)Q(s,a) \right]-

\sum_{s\in \mathcal{S}} d^{\pi_\theta}(s)B(s)\nabla_\theta

\underbrace{\sum_a\pi_\theta(s,a)}_{=1}= 
\color{blue}\mathbb{E}_{\pi_\theta}\left[\nabla_\theta\log\pi_\theta(s,a)Q(s,a) \right]-0
$$

## 6.2. Advantage

Using the state-value function as baseline leads to the **Advantage Function $A(s,a)$:**

$$
A(s,a)=Q^{\pi_\theta}(s,a)-V^{\pi_\theta}(s)
$$

The advantage tells us *how much better* (or worse) an action is compared to the *average* action from state $s$. Our new, lower-variance policy gradient is:

$$
\nabla_\theta J(\theta)\approx\mathbb{E}_{\pi_\theta}[\nabla_\theta\log\pi_\theta(s,a)A^{\pi_\mathbf{w}}(s,a)]
$$

<aside>
📌

**Approximation with TD Error**

Computing $A(s,a)$ exactly is expensive. We could learn two critics (one for $Q$ and one for $V$), but a simpler approach is to use the **TD error** as an estimate of the advantage:

$$
\delta_v=r+\gamma V_v(s')-V_v(s)\approx A(s,a)
$$

This allows us to use **only one critic** (the $V_v(s)$ function). The final Actor-Critic update becomes:

$$
\Delta\theta=\alpha\nabla_\theta\log\pi_\theta(s,a)\delta_v
$$

This is the standard **efficient Advantage Actor-Critic (A2C) update rule**.

</aside>

---