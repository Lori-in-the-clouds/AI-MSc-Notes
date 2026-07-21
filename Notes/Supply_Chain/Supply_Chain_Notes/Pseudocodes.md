# Pseudocodes

Num of pages: 0
Status: Done
Type: theory

# 1. First-Visit Monte Carlo Prediction

<aside>

1. Initialize $V(s)=0,\ \forall  s\in S$
2. For each episode:
    1. Generate an episode following policy $\pi$: $\{\mathbf{S_0, A_0}, R_1, \mathbf{S_1, A_1}, R_2, \dots, \mathbf{S_{T-1}, A_{T-1}}, R_T, S_T\}$
    2. For each step of episode $t=T-1, T-2,\dots,0$:
        1. $G_t \leftarrow R_{t+1} + \gamma G_{t+1}$
        2. If $S_t$  does not occur in $S_0, S_1, \dots, S_{t-1}$:
            1. $V(S_t) \leftarrow V(S_t) + \alpha (G_t - V(S_t))$
</aside>

# 2. Every-visit Monte Carlo Prediction

<aside>

1. Initialize $V(s)=0,\ \forall  s\in S$
2. For each episode:
    1. Generate an episode following policy $\pi$: $\{\mathbf{S_0, A_0}, R_1, \mathbf{S_1, A_1}, R_2, \dots, \mathbf{S_{T-1}, A_{T-1}}, R_T, S_T\}$
    2. For each step of episode $t=T-1, T-2,\dots,0$:
        1. $G_t \leftarrow \gamma G_{t+1} + R_{t+1}$
        2. $V(S_t) \leftarrow V(S_t) + \alpha (G_t - V(S_t))$
</aside>

---

# 3. Tabular TD(0) Prediction

<aside>

1. Initialize $V(s)=0,\ \forall  s\in S$
2. For each episode:
    1. Initialize $S_0$
    2. While $S_t\ne S_{end}$:
        1. Choose an action $A_t \sim \pi(\cdot |S_t)$ and observe $R_{t+1}$ and $S_{t+1}$
        2. $V(S_t) \leftarrow V(S_t) + \alpha \left( R_{t+1} + \gamma V(S_{t+1}) - V(S_t) \right)$
        3. $t \leftarrow t + 1$
</aside>

# 4. Tabular TD($\lambda$) Prediction (Backward View)

<aside>

1. Initialize $V(s)=0,\ \forall  s\in S$
2. For each episode:
    1. Initialize $S_0$
    2. $E(s)=0$, $\forall s \in S$
    3. While $S_t\ne S_{end}$:
        1. Choose an action $A_t \sim \pi(\cdot |S_t)$ and observe $R_{t+1}$ and $S_{t+1}$
        2. Compute TD-error: $\delta_t=R_{t+1} + \gamma V(S_{t+1}) - V(S_t)$
        3. $E(S_t) \leftarrow E(S_t) + 1$
        4. For each $s \in \mathcal{S}$:
            1. $V(s)\leftarrow V(s) + \alpha\delta_tE(s)$
            2. $E(s)= \gamma  \lambda  E(s)$
        5. $t \leftarrow t + 1$
</aside>

---

# 5. On-Policy Monte Carlo Control

<aside>

1. 

$$Initialize $Q(s,a)=0,\ \forall  s\in \mathcal{S}\ \text{and}\  \forall a \in \mathcal{A}(s)$
2. 

$$Initialize $\pi(s)$ to a random policy, $\forall s\in \mathcal{S}$
3. For each episode:
    1. Initialize $S_0$
    2. Generate an episode by acting $\epsilon-$greedy with respect to $\pi$: $\{\mathbf{S_0, A_0}, R_1, \mathbf{S_1, A_1}, R_2, \dots, \mathbf{S_{T-1}, A_{T-1}}, R_T, S_T\}$
    3. For each transition $(S_t, A_t, R_{t+1})$ in reverse order $t = T-1, T-2, \dots, 0$:
        1. $G_t \leftarrow  R_{t+1} + \gamma G_{t+1}$
        2. $Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left( G_t - Q(S_t, A_t) \right)$
    4. $\pi(s) = \arg\max_{a} Q(s, a), \quad \forall s \in \mathcal{S}$
</aside>

---

# 6. SARSA

<aside>

1. 

$$Initialize $Q(s,a)=0,\ \forall  s\in \mathcal{S}\ \text{and}\  \forall a \in \mathcal{A}(s)$
2. For each episode:
    1. Choose $S_0$ and $A_0$ (according to an $\epsilon$-greedy policy derived from $Q(S_0, \cdot)$)
    2. While $S_t\ne S_{end}$: 
        1. Get $(S_t,A_t,R_{t+1})$
        2. Choose $A_{t+1}$ according to an $\epsilon$-greedy policy derived from $Q(S_{t+1},\cdot)$
        3. $Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left( R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) \right)$
        4. $t \leftarrow t + 1$
</aside>

# 7. SARSA($\lambda$)

<aside>

1. Initialize $Q(s,a)=0,\ \forall  s\in \mathcal{S}\ \text{and}\  \forall a \in \mathcal{A}(s)$
2. For each episode:
    1. Choose $S_0$ and $A_0$ (according to an $\epsilon$-greedy policy derived from $Q(s_0, \cdot)$)
    2. $E(s,a)=0$, $\forall s\in S$, $\forall a\in \mathcal{A}$
    3. While $S_t\ne S_{end}$: 
        1. Get $(S_t,A_t,R_{t+1})$
        2. Choose $A_{t+1}$ according to an $\epsilon$-greedy policy derived from $Q(S_{t+1},\cdot)$
        3. $\delta_t \leftarrow R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)$
        4. $E(S_t, A_t) \leftarrow E(S_t, A_t) + 1$
        5.  For each $s \in \mathcal{S}, a \in \mathcal{A}(s)$:
            1. $Q(s, a) \leftarrow Q(s, a) + \alpha \delta_t E_t(s,a)$
            2. $E(s,a)= \gamma  \lambda  E(s,a)$
        6. $t \leftarrow t + 1$
</aside>

---

# 8. Q-Learning

<aside>

1. Initialize $Q(s,a),\ \forall  s\in \mathcal{S}\ \text{and}\  \forall a \in \mathcal{A}(s)$
2. For each episode:
    1. Initialize starting state $S_0$ 
    2. While $S_t\ne S_{end}$: 
        1. Choose action $A_t$ from state $S_t$ using an $\epsilon$-greedy policy derived from $Q(S_t, \cdot)$ and observe $R_{t+1}$ and $S_{t+1}$
        2. $Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left( R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a) - Q(S_t, A_t) \right)$
        3. $t \leftarrow t + 1$
</aside>

# 9. DQN

<aside>

1. Initialize $D$ to capacity $N$
2. Initialize $Q(s,a,\mathbf{w})$ with random weights $\mathbf{w}$
3. Initialize target action-value function $Q(s,a,\mathbf{w}_{old})$
4. For each episode:
    1. Initialize starting state $S_0$
    2. While $S_t\ne S_{end}$: 
        1. Choose action $A_t$ from state $S_t$ using an $\epsilon$-greedy policy with respect to $Q(s,a,\mathbf{w})$
        2. Execute action $A_t$, observe reward $R_{t+1}$ and next state $S_{t+1}$
        3. Store transition $(S_t, A_t, R_{t+1}, S_{t+1})$ in $\mathcal{D}$
        4. Sample a random mini-batch of transitions $(S_j, A_j, R_{j+1},S_{j+1})$,  from  $\mathcal{D}$
        5. Set target $y_j$ for each mini-batch transition:
            
            $$
            y_j = \begin{cases} R_{j+1} & \text{if } S_{j+1} \text{ is terminal} \\ R_{j+1} + \gamma \max_{a'} Q(S_{j+1}, a'; \mathbf{w}_{\text{old}}) & \text{otherwise} \end{cases}
            $$
            
        6. Perform a gradient descent step on the loss function with respect to the network parameters $\mathbf{w}$:
            
            $$
            \mathcal{L}(\mathbf{w}) = \left( y_j - Q(S_j, A_j; \mathbf{w}) \right)^2
            $$
            
        7. Every $C$ steps, update target network weights: $\mathbf{w}_{\text{old}} \leftarrow \mathbf{w}$
        8. $t \leftarrow t + 1$
</aside>

---

# 9. REINFORCE

<aside>

1. Initialize $\mathbf{w}$ arbitrarily 
2. For each episode:
    1. Generate an episode by sampling actions from $\pi_\mathbf{w}$: $\{S_0, A_0, R_1, S_1, A_1, R_2, \dots, S_{T-1}, A_{T-1}, R_T, S_T\}$
    2. For each step of episode $t=T-1, T-2,\dots,0$:
        1. $G_t \leftarrow \gamma G_{t+1} + R_{t+1}$
        2. $\mathbf{w} \leftarrow \mathbf{w} + \alpha  \nabla_\mathbf{w} \ln \pi_\mathbf{w}(s_t,a_t)G_t$
</aside>

---

# 10. Q-Learning Actor-Critic

<aside>

1. Initialize actor parameters $\theta$ and critic parameters $w$
2. Foreach episode:
    1. Initialize starting state $S_0$ and $A_0$ (by sampling the current policy $\pi_\theta(\cdot \vert{} S_0)$)
    2. While $S_t\ne S_{end}$: 
        1. Get $(S_t,A_t,R_{t+1})$
        2. Choose $A_{t+1}$ by sampling from $\pi_\theta$
        3. $\theta \leftarrow \theta + \alpha_\theta \nabla_\theta \ln \pi_\theta(A_t \vert{} S_t) Q_w(S_t, A_t)$
        4. $w \leftarrow w + \alpha_w \left( R_{t+1} + \gamma Q_w(S_{t+1}, A_{t+1}) - Q_w(S_t, A_t) \right) \nabla_w Q_w(S_t, A_t)$
        5. $t \leftarrow t + 1$
</aside>

---

# 11. Advantage Actor Critic (A2C)

<aside>

1. Initialize actor parameters $\theta$ and critic parameters $w$
2. Foreach episode:
    1. Initialize starting state $S_0$ and $A_0$ (by sampling the current policy $\pi_\theta(\cdot \vert{} S_0)$)
    2. While $S_t\ne S_{end}$: 
        1. Get $(S_t,A_t,R_{t+1})$
        2. Choose $A_{t+1}$ by sampling from $\pi_\theta$
        3. Compute $\delta_t = R_{t+1} + \gamma V_\mathbf{w}(S_{t+1}) - V_\mathbf{w}(S_t)$
        4. $\theta \leftarrow \theta + \alpha_\theta \nabla_\theta \ln \pi_\theta(A_t \vert{} S_t) \delta_t$
        5. $w \leftarrow w + \alpha_w \delta_t \nabla_w V_w(S_t)$
        6. $t=t+1$
</aside>

---

# 12. Dyna-Q

<aside>

1. Initialize $\mathcal{M}(s,a)$, $Q(s,a)$ $\  \forall s \in \mathcal{S}, a \in \mathcal{A}(s)$
2. For each episode:
    1. Initialize $S_0$
    2. While $S_t\ne S_{end}$: 
        1. Get $(S_t,A_t,R_{t+1})$ using an $\epsilon$-greedy policy derived from $Q(S_t, \cdot)$
        2. $Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left( R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a) - Q(S_t, A_t) \right)$
        3. Record the real transition in the model $\mathcal{M}$:  $M(S_t, A_t) \leftarrow (R_{t+1}, S_{t+1})$
        4. Repeat $n$ times:
            1. Take $(S,A)$ randomly sampled from $\mathcal{M}$
            2. Query the model $M$ to get the simulated reward and next state: $(R, S') \leftarrow M(S, A)$
            3. $Q(S, A) \leftarrow Q(S, A_) + \alpha \left( R + \gamma \max_{a} Q(S', a) - Q(S, A) \right)$
        5. $t=t+1$
</aside>

---

# 13. MC Search

<aside>

1. Given $\mathcal{M}$ and the simulation policy $\pi$ (random)
2. For each episode:
    1. Initialize $S_0$
    2. While $S_t\ne S_{end}$: 
        1. For each action $a \in \mathcal{A}(S_t)$: 
            1. Simulate $K$ episode: $\{ S_t, a, R_{t+1}^k, S_{t+1}^k, A_{t+1}^k, R_{t+2}^k, \dots, S_T^k \} \sim M(S_t, a), \quad \forall k \in \{1, \dots, K\}$
            2. Evaluate it by computing the mean return over all episode: $Q(S_t, a) \leftarrow \frac{1}{K} \sum_{k=1}^K G_t^k$
        2. Select the best action greedily: $A_t \leftarrow \arg\max_{a} Q(S_t, a)$
        3. $t=t+1$
</aside>

---

# Detailed Helper Functions

### Backpropagate

Updates the statistics ($N$ visits and $Q$ total reward) for all nodes on the path from the leaf back to the root.

```jsx
function Backpropagate(Node, Reward):
    While Node is not NULL:
        Node.Visits <- Node.Visits + 1
        Node.Reward <- Node.Reward + Reward
        Node <- Node.Parent
```

---