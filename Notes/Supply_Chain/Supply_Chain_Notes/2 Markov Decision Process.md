# 2. Markov Decision Process

Num of pages: 2
Status: Done
Type: theory

# 1. The Markov Framework: From Process to Decision

The environment is considered **fully observable**, meaning the agent has all necessary information in the current state to determine future probabilities (the **Markov Property**). Almost all RL problems can be formalized as MDPs; even partially observable problems can be converted into them. MDPs with a single state are **bandited**. The framework is built in a progression that add complexity one step at a time:

1. **Markov Process (MP)**: a memoryless random process defined by a tuple $\langle \mathcal{S}, \mathcal{P} \rangle$ where:
    - $\mathcal{S}$ → finite set of **states**
    - $\mathcal{P}$ → **state transition probability matrix**, **$\mathcal{P}_\mathcal{SS'}=\mathbb{P}[S_{t+1}=s'|S_t=s]$** (= the probability of moving to a next state, $s'$ depends only on the current state $s$, not on the history)
    
    **N.B.** It has **no actions** and **no rewards**.
    
    For example, consider the Student Markov Chain, the probability of moving from "*Class 1*" to "*Class 2*" is always 0.5, regardless of whether you just came from "*TikTok*" or from another activity. The process only cares about your current state to determine the next state:
    
    ![image.png](2%20Markov%20Decision%20Process/image.png)
    
    <aside>
    💡
    
    **Goal:** predict how the system evolves over time (=**Predicting movement)**.
    
    </aside>
    
2. **Markov Reward Process (MRP)**: extends a MP by adding value judgments. It's defined as a tuple $\langle \mathcal{S}, \mathcal{P},
\textcolor{red}{\mathcal{R}},\textcolor{red}{\gamma} \rangle$, where:
    - $\mathcal{S}$ → finite set of **states**
    - $\mathcal{P}$ → **state transition probability matrix**, **$\mathcal{P}_\mathcal{SS'}=\mathbb{P}[S_{t+1}=s'|S_t=s]$**
    - $\textcolor{red}{\mathcal{R}}$ → **reward function**, **$\mathcal{R}_\mathcal{s}=\mathbb{E}[R_{t+1}|S_t=s]$**
    - $\textcolor{red}{\gamma}$ → **discount factor**, **$\gamma\in[0,1]$**
    
    We can compute:
    
    1. **Return $G(t)$** → the total discounted reward from a specific point in time): $G_t = R_{t+1}+\gamma R_{t+2}+...=\sum_{k=0}^\infty \gamma^kR_{t+k+1}$.  The discount factor prevents the return from becoming infinite in processes that don't have a defined end:
        - If $\gamma$ is close to 0, the agent is "**myopic**" and only cares about immediate rewards.
        - If $\gamma$ is close to 1, the agent is "**far-sighted**" and considers long-term rewards almost as important as immediate ones.
        
        The calculation of the Return is based only on future rewards:
        
        - $C1\rightarrow C2 \rightarrow C3 \rightarrow \text{Pass} \rightarrow \text{Sleep}$, with $\gamma=0.5$: $\ \ G_1 = -2\cdot1 + -2\cdot \frac12+-2\cdot\frac14+10\cdot\frac18=-2.25$
        - $C1 \rightarrow TK \rightarrow TK \rightarrow C1 \rightarrow C2 \rightarrow \text{Sleep}$, with $\gamma=0.5$: $\ \ G_1 = -2\cdot1 -1\cdot \frac12-1\cdot\frac14-2\cdot\frac{1}{8} -2\cdot\frac{1}{16}=-3.125$
        
        ![image.png](2%20Markov%20Decision%20Process/image%201.png)
        
    2. **State value function $v(s)$** → the expected return starting from state $s$:  $v(s)=\mathbb{E}[G_t|S_t=s]$
        
        ![IMG_35A26E01889F-1.jpeg](2%20Markov%20Decision%20Process/IMG_35A26E01889F-1.jpeg)
        
    
    **N.B.** An MRP is a **passive model** where the agent cannot choose actions; transitions are purely probabilistic.
    
    <aside>
    💡
    
    **Goal:** evaluate the value of states (=**Evaluation**).
    
    </aside>
    
3. **Markov Decision Process (MDP)**: adds the concept of actions to an MRP. It’s a tuple $\langle \mathcal{S},\textcolor{red}{\mathcal{A}}, \mathcal{P}, \mathcal{R},\gamma \rangle$ where:
    - $\mathcal{S}$ → finite set of states
    - $\textcolor{red}{\mathcal{A}}$ → finite set of actions
    - $\mathcal{P}$ → state transition probability matrix, defined by $\mathcal{P}_{\mathcal{s}\mathcal{s}′}^{\textcolor{red}{a}}=\mathbb{P}[\mathcal{S}_{t+1}=s'∣\mathcal{S}_t=s,\textcolor{red}{A_t=a}]$
    - $\mathcal{R}$ →reward function, defined by 
    
    $\mathcal{R}_s^a = \mathbb{E}[ R_{t+1} \mid S_t = s,\, \textcolor{red}{A_t = a} ]$
    - $\gamma$ → discount factor, $\gamma \in [0,1]$
    
    The goal of an MDP is no longer just to calculate the reward, but to find the **optimal policy**. In this case we can compute:
    
    - **State Value function** → $v_{\textcolor{red}{\pi}}(s) = \mathbb{E}_{\textcolor{red}{\pi}}[G_t|S_t=s]$
    - **Action-value function** → 
    
    $q_{\textcolor{red}{\pi}}(s,a) = \mathbb{E}_{\textcolor{red}{\pi}} [ G_t \mid S_t = s,\, A_t = a ]$
    
    <aside>
    💡
    
    **Goal:** introducing actions to find the absolute best strategy $\pi^*$ (=**Control**).
    
    </aside>
    

---

# 4. The Bellman Equations

The Bellman Equations provide the fundamental **recursive decomposition** of the value function. They state that the value of any state is the immediate reward plus the discounted value of the next state.

## 4.1. Bellman Expectation Equations (Linear) → [Policy Evaluation]

These equations define the value of a state or action for a **fixed policy $\pi$** and are used in **Policy Evaluation**. These are systems of **linear equations**.

- **For MRP ($v(s)$):** the value of a state is the expected immediate reward plus the discounted expected value of the next state $S_{t+1}$:
    - **Recursive Decomposition:**
        
        $$
        \textcolor{orange}{v(s)}= \mathbb{E}[G_t|S_t=s]=\mathbb{E}[R_{t+1}+\gamma R_{t+2}+\gamma^2 R_{t+3}+...|S_t=s]=\mathbb{E}[R_{t+1}+\gamma(R_{t+2}+\gamma R_{t+3}+...)|S_t=s] =\mathbb{E}[R_{t+1}+\gamma G_{t+1}|S_t=s]=\textcolor{orange}{\mathbb{E}[R_{t+1}+\gamma v(S_{t+1})|S_t=s]}
        $$
        
    - **Decomposition by Successor States:** (→ it relates the value of a state to the values of all possible successor states)
        
        $$
        \displaystyle v(s) = R_s + \gamma\sum_{s'\in \mathcal{S}}\mathcal{P}_{ss'}\cdot v(s')
        $$
        
    - **Matrix Form:**
        
        $$
        v= \mathcal{R}+\gamma\mathcal{P}v \rightarrow v = (I-\gamma\mathcal{P})^{-1}\mathcal{R}
        $$
        
- **For MDP ($v_\pi(s)$ and $q_\pi(s,a)$):**
    - **State-Value Function:**
        1. **Definition** → $v_\pi(s) = \mathbb{E}_\pi[G_t|S_t=s]$
        2. **Recursive Decomposition → $v_\pi(s)=\mathbb{E}_\pi[R_{t+1}+\gamma v_\pi(S_{t+1})|S_t=s]$**
        3. **Decomposition by Action (using $q_\pi$):** the value of the state is the sum of the values of all possible actions, weighted by the probability of taking that action according to the policy:
            
            $$
            \displaystyle
            
             v_\pi(s)=\color{red}\sum_{a \in \mathcal{A}} \pi(a \mid s)q_\pi(s,a)
            $$
            
    - **Action-Value Function:**
        1. **Definition → $q_\pi(s,a) =\mathbb{E}_\pi [ G_t \mid S_t = s,\, A_t = a ]$**
        2. **Recursive Decomposition** → $q_\pi(s,a) = \mathbb{E}_\pi [ R_{t+1} + \gamma \, q_\pi(S_{t+1}, A_{t+1}) \mid S_t = s,\, A_t = a ]$
        3. **Decomposition by Next State (using $v_\pi$)**: the value is the sum of the immediate reward and the discounted value of all possible next states, weighted by their transition probabilities: 
            
            $$
            \displaystyle
            q_\pi(s,a)={\color{blue}\mathcal{R}_{s}^a+\gamma \sum_{s' \in \mathcal{S}} \mathcal{P}_{ss'}^av_{\pi}(s')}=\mathcal{R}_s^a + \gamma \sum_{s' \in \mathcal{S}} \mathcal{P}_{ss'}^a {\color{red} \sum_{a' \in \mathcal{A}} \pi(a'|s')q_{\pi}(s',a')}
            
            $$
            

<aside>
✅

**Full Bellman Expectation Equation**

$$
q_\pi(s,a)= \mathcal{R}_s^a + \gamma \sum_{s' \in \mathcal{S}} \mathcal{P}_{ss'}^a \sum_{a' \in \mathcal{A}} \pi(a'|s')q_{\pi}(s',a') \qquad
v_{\pi}(s) = \sum_{a \in \mathcal{A}} \pi(a|s)\left[{\color{blue}\mathcal{R}_{s}^a+\gamma \sum_{s' \in \mathcal{S}} \mathcal{P}_{ss'}^av_{\pi}(s')}\right]
$$

The Bellman Expectation Equations can be solved **directly** via matrix inversion. However, this direct solution is computationally expensive with complexity  $O(n^3)$ for $n$ states, making it intractable for large problems.

</aside>

## 4.2. Bellman Optimality Equations (non-linear) → [Policy Improvement]

<aside>
💡

**Principle of Optimality**

A policy ‭$\pi(a|s)$‬‭‬‭‬‭‬‭‬‭‬ achieves the optimal value from state ‭$s$‬ $\left[v_{\pi}(s) = v_{*}(s)\right]$ ‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬if and only if, for any state ‭$s'$‬ reachable from ‭$s$‬, ‭$\pi$‬ achieves the optimal value from state ‭$s'$$\left[v_{\pi}(s') = v_{*}(s')\right]$‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬.

**N.B.** An optimal policy can be broken down into optimal sub-policies. It mathematically proves that you can update current state's value by looking just one step ahead, assuming optimal behavior in all future successor states.

</aside>

The ultimate goal of Reinforcement Learning is to find the **optimal policy $\pi_*$** and corresponding **optimal value functions** ($v_*$ and $q_*$):

- The **optimal state-value function $v_*(s)$** is the maximum of the optimal action-value functions:
    
    $$
    v_*(s)=\max_aq_*(s,a)=\max_a[R_s^a + \gamma\sum_{s' \in \mathcal{S}} \mathcal{P}^a_{ss'}v_*(s')]  
    $$
    
- The **optimal action-value function $q_*(s,a)$** is the expected immediate reward plus the discounted value of the optimal next state:
    
    $$
    q_*(s,a)= \mathcal{R}_s^a +\gamma\sum_{s'\in\mathcal{S}} \mathcal{P}_{ss'}^av_*(s')= \mathcal{R}_s^a +\gamma\sum_{s'\in\mathcal{S}} \mathcal{P}_{ss'}^a\max_{a'}q_*(s',a')
    $$
    

<aside>
💡

**When an MDP is considered solved?**

An MDP is considered **solved** when we find the **optimal policy** $\pi_*$ that can be derived from the optimal value function:

- Knowing $q_*(s,a)$ directly solves the MDP, since the optimal action in each state is simply the one with the highest action value.
- Knowing $v_*(s)$ solves the MDP only if the agent also knows the environment model (P and R). In this case, the agent can look one step ahead, evaluate the possible next states, and choose the optimal action.
</aside>

<aside>
🚨

**Solving the Optimality Equations**

The **Bellman Optimality Equations** are non-linear due to the max operator for this reasons cannot be solved directly. They are solved using iterative methods like Value Iteration, Policy Iteration and Q-Learning.

</aside>

---

# 5. How to find the Optimal Policy

If we define a partial ordering over policies: $\pi \ge \pi'\ \ \text{if}\ \ v_\pi(s) \ge v_{\pi'}(s),\ \forall s$. For any MDP, there exists an deterministic optimal policy $\pi_*$ that is better than or equal to all other policies, $\pi_* \ge \pi,\ \forall \pi$.

An optimal policy can be found by maximizing over $q_*(s,a)$:

$$
\pi_*(a \mid s) =
\begin{cases}
1 & \text{if } a = \displaystyle \arg\max_{a \in \mathcal{A}} q_*(s,a) \\
0 & \text{otherwise}
\end{cases}
$$

**N.B.** All optimal policies achieve the same optimal state-value function $v_*(s)$ and optimal action-value function $q_*(s,a)$.

![image.png](2%20Markov%20Decision%20Process/image%202.png)

---