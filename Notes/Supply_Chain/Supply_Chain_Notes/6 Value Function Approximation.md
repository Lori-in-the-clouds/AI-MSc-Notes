# 6. Value Function Approximation

Num of pages: 2
Status: Done
Type: theory

# 1. The Need for Scaling

**Tabular methods** (using look-up tables for $Q$ or $V$ values) are unusable for large-scale problems due to two main issues:

1. There are too many states and/or actions to store in memory.
2. It’s too slow to learn the value of each state individually.

<aside>
✅

**Solution: Value Function Approximation (VFA)**

The solution is to estimate the value function with a **parametric function approximator** that generalizes experience from seen states to **unseen states**:

- **State-Value Function** → ****$v_π(s)≈\hat{v}_{\pi}(s,\mathbf{w})$
- **Action-Value Function** → ****$q_π(s,a)≈\hat{q}(s,a,\mathbf{w})$

Here, $\mathbf{w}$ is the **weight vector** (or parameters) that the function approximator must learn.

</aside>

---

# 2. Learning the Parameters $\mathbf{w}$ with Gradient Descent

The primary goal is to define the optimal weights $\mathbf{w}$ that minimize the error between our estimate $\hat{v}{(s,w)}$ and the true value $v_π(s)$ by minimizing the **Mean Squared Error (MSE)**:

$$
J(\mathbf{w}) = \mathbb{E}_{\pi} \left[ \left( v_{\pi}(S) - \hat{v}(S, \mathbf{w}) \right)^2 \right] 
$$

## 2.1. The Theoretical Method: Gradient Descent

To find the optimal weights $\mathbf{w}$ that minimize $J(\mathbf{w})$,  we use a method called **Gradient Descent**.

- **What is the gradient?** → ****the gradient $∇J(\mathbf{w})$ is a vector that points in the direction of the *steepest ascent* (maximum increase) of the cost function.
- **How to Minimize** → to *reduce* the cost, we must simply move in the **opposite** (negative) direction of the gradient.

The theoretical update rule for Gradient Descent is:

$$
\mathbf{w}←\mathbf{w}−α∇J(\mathbf{w})
$$

Where $\alpha$ is the "step size" (or *learning rate*) that controls how fast we move.

## 2.2. The Practical Method: Stochastic Gradient Descent (SGD)

<aside>
🚨

**Problem:**

The formula for $J(\mathbf{w})$ contains the expectation operator $\mathbb{E}_\pi$. Calculating this expected value would require averaging over *all possible states* in the environment, which is almost always computationally impossible.

</aside>

Instead of calculating the gradient over the entire expectation, we use **Stochastic Gradient Descent (SGD)**. SGD "samples" the gradient by using a single experience (or a small *batch*) at a time.

- The **Expected Update** ($\Delta\mathbb{w}$) using the full gradient is:
    
    $$
    \Delta \mathbf{w} = -\frac{1}{2} \alpha \nabla_{\mathbf{w}} J(\mathbf{w}) = 
    
    -\frac{1}{2} \alpha \nabla_{\mathbf{w}}\left[
    \mathbb{E}_{\pi} \left( \left( v_{\pi}(S) - \hat{v}(S, \mathbf{w}) \right)^2 \right)
    
    \right]=
    
     \alpha \mathbb{E}_{\pi} \left[ \left( v_{\pi}(S) - \hat{v}(S, \mathbf{w}) \right) \nabla_{\mathbf{w}} \hat{v}(S, \mathbf{w}) \right]
    $$
    
- The **Stochastic Update** ($\Delta\mathbb{w}$) removes the expectation operator:
    
    $$
    \Delta \mathbf{w} = \alpha \underbrace{\left( v_{\pi}(S) - \hat{v}(S, \mathbf{w}) \right) }_{\text{Prediction Error}}\underbrace{\nabla_{\mathbf{w}} \hat{v}(S, \mathbf{w})}_{\text{Direction }}
    $$
    
    The term $v_π(S)−\hat{v}(S,\mathbf{w})$ is the **Prediction Error**, it's the difference between the true value and our estimate.
    

## 2.3. A simple Case: Linear VFA

The simplest type of function approximator is a **linear** one, where the value is a weighted sum of the state's features $x(S)$:

$$
\hat{v}(S,\mathbf{w})=x(S)^T\mathbf{w}
$$

Using a linear model provides two mathematical advantages:

1. **Stability** → the loss function $J(\mathbf{w})$ is *convex* (it's shaped like a "bowl"). This means SGD is **guaranteed** to find the single, global minimum (the optimal $\mathbf{w}$).
2. **Simplicity** → the gradient of the estimate  $∇_\mathbf{w}\hat{v}(S,\mathbf{w})$ is simply the feature vector itself, $\mathbf{x}(S)$.

As a result, the SGD update rule for linear VFA becomes incredibly simple:

$$
Δ\mathbf{w}=\alpha[v_π(S)−x(S)^T\mathbf{w}]\cdot x(S)
$$

<aside>
📌

**Table Lookup**

Tabular methods are a special case of linear VFA using **one-hot encoding** as features:

$$
\mathbf{x}^{table}(S) = \begin{pmatrix}    \mathbf{1}(S = s_1) \\    \vdots \\    \mathbf{1}(S = s_n)\end{pmatrix} \Rightarrow 

\hat{v}(S, \mathbf{w}) =  \begin{pmatrix}
    \mathbf{1}(S = s_1) \\
    \vdots \\
    \mathbf{1}(S = s_n)
\end{pmatrix} ^T \cdot
\begin{pmatrix}
    w_1 \\
    \vdots \\
    w_n
\end{pmatrix}
$$

</aside>

---

# 3. How do we apply SGD in RL?

## 3.1. Incremental (Online) Methods

Until now, we have assumed access to the true value function $v_\pi(s)$, as if it were provided by an oracle. But in RL there is no supervisor, only rewards. The true value must be replaced by a **Target** derived from experience:

- For MC, the target is the episode return $G_t$ → $\Delta \mathbf{w} = \alpha \left( \textcolor{red}{G_t} - \hat{v}(S, \mathbf{w}) \right) \nabla_{\mathbf{w}} \hat{v}(S_t, \mathbf{w})$
- For TD(0)  the target is the online TD target → $\Delta \mathbf{w} = \alpha \left( \textcolor{red}{R_{t+1} + \gamma \hat{v}(S_{t+1}, \mathbf{w})} - \hat{v}(S_t, \mathbf{w}) \right) \nabla_{\mathbf{w}} \hat{v}(S_t, \mathbf{w})$
- For 𝑇𝐷(𝜆), the target is the $\lambda$-return $G_t^\lambda$ → $\Delta \mathbf{w} = \alpha \left( \textcolor{red}{G_t^{\lambda}} - \hat{v}(S, \mathbf{w}) \right) \nabla_{\mathbf{w}} \hat{v}(S_t, \mathbf{w})$

<aside>
📌

**Study of $\lambda$: Should We Bootstrap?**

- **Bootstrapping efficiency** → performance is best for intermediate values ($\lambda \approx 0.8–0.9$), while $\lambda = 1$ (Monte Carlo) has higher variance. This shows that bootstrapping improves sample efficiency.
- **Accumulating traces instability** → with accumulating traces ($E_t \leftarrow E_{t-1} + 1$), repeated visits can make traces grow too large, amplifying TD errors and leading to instability, especially for high $\lambda$. Replacing traces ($E_t \leftarrow 1$) limit the credit assigned to repeated visits, reducing variance and making learning more stable.

![image.png](6%20Value%20Function%20Approximation/image.png)

</aside>

<aside>
🚨

**Problem in $TD(\lambda)$: Stability and Convergence Issues**

In standard Gradient Descent, the target is fixed. In TD learning, instead, the parameter vector $\mathbf{w}$ appears in both the prediction and the target:

$$
\Delta \mathbf{w} = \alpha \left( \textcolor{red}{R_{t+1} + \gamma \hat{v}(S_{t+1}, \mathbf{w})} - \hat{v}(S_t, \mathbf{w}) \right) \nabla_{\mathbf{w}} \hat{v}(S_t, \mathbf{w})
$$

This creates a chain reaction:

1. **Cause (Moving target)** → the TD target depends on $\mathbf{w}$, so every update changes both the prediction and the target.
2. **The Effect (Semi-Gradient Approximation)** → computing the true gradient of a moving target is mathematically intractable online. To bypass this, TD learning ignores the target's dependency on  $‭\mathbf{w}$‬ during differentiation, computing the derivative only with respect to the current prediction  $\hat{v}(S_t, \mathbf{w})‬‭‬‭‬‭‬$‭‬‭‬.
3. **The Consequence (Instability & Divergence)** → because the update follows a **semi-gradient** rather than the true gradient of an objective function, the method loses a proper optimization direction. As a result, learning can become unstable and may diverge.

While this approximation remains stable in simpler scenarios, it can lose control and diverge when combined with:

- Off-policy learning
- Non-linear function approximation

![image.png](6%20Value%20Function%20Approximation/image%201.png)

</aside>

<aside>
✅

**Solution: Gradient TD**

Unlike standard TD, Gradient TD follows the true gradient of the **projected Bellman error.** As shown in its convergence table, it converges even in **off-policy non-linear:**

![image.png](6%20Value%20Function%20Approximation/image%202.png)

</aside>

## 3.2. Batch (Offline) Methods

<aside>
🚨

**Problem with Incremental**

Incremental methods are "not sample efficient". They use a piece of data *once* and then throw it away.

</aside>

<aside>
✅

**Solution:**

1. Collect a large "batch" of experience $\mathcal{D}$ consisting of `<state, value>` pairs: $\mathcal{D}=\{<s_1,v_1^\pi>,<s_2,v_2^\pi>,...,<s_T,v_T^\pi>\}$.
2. Fit the parameter vector $\mathbf{w}$ to the entire dataset at once.
3. Choose the parameters that minimize the **Least-Squares (LS) error** over the whole batch:
    
    $$
    LS(\mathbf{w})=\sum_{t=1}^{T}(v_t^π−\hat{v}(s_t,\mathbf{w}))^2= \mathbb{E}_\mathcal{D}\left[(v^\pi-\hat{v}(s,\mathbf{w}))^2 \right]
    $$
    
</aside>

---

# 4. Advanced Algorithms: DQN and LSPI

## 4.1. Deep Q-Networks (DQN)

The core idea of DQN is to use a deep neural network as a powerful, non-linear function approximator to learn the action-value function $\hat{q}(s,a,\mathbf{w})$, updating its weights via Stochastic Gradient Descent (SGD). Combining an off-policy algorithm like Q-Learning with a non-linear approximator like a neural network is notoriously unstable. To achieve stability we can use two key techniques:

1. **Experience Replay** → stores transitions in a memory buffer $\mathcal{D}$. Instead of learning from consecutive experiences, the agent trains on random mini-batches sampled from the buffer. This **breaks the correlation** in the sequential data, solving the non-i.i.d. problem for neural networks.
2. **Fixed Q-Targets (Target Network)** → in Q-learning the parameter vector $\mathbf{w}$ appears in both the prediction and the target, creating a **moving target** that can make learning unstable. To solve this, DQN uses a separate **target network** with frozen parameters $\mathbf{w}_{\text{old}}$ to compute the TD target:
    
    $$
    \mathcal{L}(\mathbf{w}) = \mathbb{E}_{s, a, r, s' \sim \mathcal{D}} \left[ \left( r + \gamma \max_{a'} Q(s', a'; \textcolor{green}{\mathbf{w}_{\text{old}}}) - Q(s, a; \mathbf{w}) \right)^2 \right]
    $$
    
    **N.B.** The target network $\mathbf{w}_{\text{old}}$ is updated periodically by copying the weights of the main Q-network: $\mathbf{w}_{\text{old}} \leftarrow \mathbf{w}$.
    

## 4.2. Least-Squares Policy Iteration (LSPI)

LSPI is a powerful **batch control** algorithm built on the classic "Policy Iteration" framework. This means it works in a simple two-step cycle:

- **Policy Improvement** → make the policy greedy: $π'(s)←\argmax_a Q(s,a)$
- **Policy Evaluation** → instead of slowly learning $Q(s,a)$ with incremental updates, it uses **LSTDQ** (Least-Squares TD for Q-values) which computes the best-fit parameter vector $\mathbf{w}$ from the entire dataset $\mathcal{D}$ in a single batch computation.
    
    <aside>
    📌
    
    **LSTDQ (Least Square Q-Learning)**
    
    The core challenge for any batch *control* algorithm is that the experience $\mathcal{D}$ was gathered using many different behavior policies, not just the single policy $\pi$ we are currently trying to evaluate. 
    
    Therefore, to evaluate $\pi$, the algorithm **must learn off-policy**. LSTDQ achieves this by adopting the central idea of Q-learning: it updates the value of the action $A_t$ (taken by an old policy) by comparing it to the value of the action $A'$ that the *new* policy $\pi$ would take in the *next* state $S_{t+1}$.
    
    - First, consider the **incremental update**:
        - TD Error → $\delta=R_{t+1}+\gamma\hat{q}\left(S_{t+1},\pi(S_{t+1}),\mathbf{w}\right)-\hat{q}(S_t,A_t,\mathbf{w})$
        - Weight Update → $\Delta\mathbf{w}= \alpha\delta\mathbf\nabla_{\mathbf{w}}\hat{q}(S_t, A_t, \mathbf{w})=
        
        \alpha\delta\mathbf{x}(S_t,A_t)^T\mathbf{w}=
        
        \alpha\delta\mathbf{x}(S_t,A_t)$
    - **LSTDQ algorithm** finds the single best $\mathbf{w}$ by solving for the point where the **total update** (the sum of all $\Delta\mathbf{w}$ over the entire batch $\mathcal{D}$) is zero, meaning the model no longer needs corrections:
        
        $$
        0=
        
        \sum_{t=1}^T \alpha \delta_t \mathbf{x}(S_t, A_t)
        
        =\sum_{t=1}^T\alpha\left[R_{t+1}+\gamma\hat{q}(S_{t+1},\pi(S_{t+1}),\mathbf{w})-\hat{q}(S_t,A_t,\mathbf{w})\right]\mathbf{x}(S_t,A_t)
        $$
        
        By substituting the linear approximation $\hat{q}(S_t,A_t,\mathbf{w})=\mathbf{x}(S_t,A_t)^T\mathbf{w}$ and algebraically isolating $\mathbf{w}$, we obtain the final **closed-form solution** for the LSTDQ parameters:
        
        $$
        0
        
        =\sum_{t=1}^T\cancel{\alpha}\left[R_{t+1}+\gamma\hat{q}(S_{t+1},\pi(S_{t+1}),\mathbf{w})-\hat{q}(S_t,A_t,\mathbf{w})\right]\mathbf{x}(S_t,A_t) \
        
        \\
        0=
        \sum_{t=1}^T \left[ R_{t+1} + \gamma \mathbf{x}(S_{t+1}, \pi(S_{t+1}))^T\mathbf{w} - \mathbf{x}(S_t, A_t)^T\mathbf{w} \right] \mathbf{x}(S_t, A_t) \\
        
        0=\left[\sum_{t=1}^T \mathbf{x}(S_t, A_t)R_{t+1}\right] + \left[\sum_{t=1}^T \gamma \mathbf{x}(S_{t+1}, \pi(S_{t+1}))^T\mathbf{w} \cdot \mathbf{x}(S_t, A_t)\right] - \left[\sum_{t=1}^T \mathbf{x}(S_t, A_t)^T\mathbf{w} \cdot \mathbf{x}(S_t, A_t)\right]\\
        
        \sum_{t=1}^T \mathbf{x}(S_t, A_t)R_{t+1} = \left[\sum_{t=1}^T \mathbf{x}(S_t, A_t)\mathbf{x}(S_t, A_t)^T\mathbf{w}\right] - \left[\sum_{t=1}^T \gamma \mathbf{x}(S_t, A_t)\mathbf{x}(S_{t+1}, \pi(S_{t+1}))^T\mathbf{w}\right] \\
        
        \sum_{t=1}^T \mathbf{x}(S_t, A_t)R_{t+1} = \left(\sum_{t=1}^T \mathbf{x}(S_t, A_t)\mathbf{x}(S_t, A_t)^T - \sum_{t=1}^T \gamma \mathbf{x}(S_t, A_t)\mathbf{x}(S_{t+1}, \pi(S_{t+1}))^T\right)\cdot \mathbf{w}\\
        
        \mathbf{w}=\left(\sum_{t=1}^T \mathbf{x}(S_t,A_t)\left[\mathbf{x}(S_t,A_t)-\gamma\mathbf{x}\left(S_{t+1},\pi(S_{t+1})\right)\right]^T  \right)^{-1}\cdot \sum_{t=1}^T\mathbf{x}(S_t,A_t)R_{t+1}
        
        $$
        
    
    **N.B.** This formula allows the algorithm to evaluate the policy in a single, highly **data-efficient** step, making it ideal for the **Policy Evaluation** phase of LSPI.
    
    </aside>
    

---