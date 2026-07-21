# 8. Integrating Learning and Planning

Num of pages: 2
Status: Done
Type: theory

# 1. Spectrum of RL: Model-Free vs. Model-Based

RL methods exist on a spectrum defined by how they use (or don't use) a **model** of the environment. A model is an approximation of the environment's transitions $\mathcal{P}$ and rewards $\mathcal{R}$.

- **Model-Free (MF) RL** → this is what we have studied so far (MC, TD, Actor-Critic). The agent has **no knowledge** of the environment's dynamics. It learns the value function and/or policy *directly from experience* (trial and error by interacting with the "real world").
- **Model-Based (MB) RL** → this is a different approach. The agent first **learns a model** (an approximation) of $\mathcal{P}$ and $\mathcal{R}$ from its experience. Then, it uses this learned model to **plan** a policy or value function (e.g., by interacting with its *own simulation* instead of the real world).
- **Integrated Architectures** → these methods, like **Dyna**, sit in the middle. They learn a model *and* learn from real experience simultaneously, using the model to "dream" and augment their real-world data.

---

# 2. Model-Based RL

## 2.1. Model-Based Loop

The process follows a distinct loop:

1. **Act** → the agent takes an action in the real environment and gathers **experience**.
2. **Model Learning** → the agent uses this experience to update its internal **model** (its approximation of the environment).
3. **Planning** → the agent uses this internal model (or simulation) to find the best **value/policy**.
4. This updated policy is then used for *Act* (Step 1).

![Screenshot 2025-11-07 at 10.49.13.png](8%20Integrating%20Learning%20and%20Planning/Screenshot_2025-11-07_at_10.49.13.png)

## 2.2. What is a model?

A model $\mathcal{M}_\eta$ is a parametric representation of the MDP, containing approximations of the transitions and rewards:

- **Transition Model: $\mathcal{P}_\eta(S'∣S,A)\approx \mathcal{P}$**
- **Reward Model:** $\mathcal{R}_\eta(R∣S,A) \approx \mathcal{R}$

## 2.3. Model Learning as Supervised Learning

Learning the model is a standard **supervised learning problem.** We collect a dataset $\mathcal{D}$ of transitions $(S_t,A_t,R_{t+1},S_{t+1})$ and then:

- **Learn $\mathcal{R}_\eta$**  is a **regression** problem (predict the scalar reward $R$ from the input $(S,A)$).
- **Learn $\mathcal{P}_\eta$** is a **density estimation** problem (predict the *distribution* over the next state $S'$ from the input $(S,A)$).

<aside>
💡

**Table Lookup Model**

A very simple, non-parametric approach is the **Table Lookup Model**:

- Store all experienced tuples $(S,A,R,S')$ in a large memory buffer.
- To "sample" from the model (e.g., for state $S$ and action $A$), find all transitions in memory that match $(S,A,⋅,⋅)$ and pick one of the resulting $(R,S')$ pairs at random.
</aside>

---

# 3. Planning with a Model

Once we have a model $\mathcal{M}_\eta$, we can use it to "plan".

- **Classical Planning** → we can solve the approximate MDP $⟨\mathcal{S},\mathcal{A},\mathcal{P}_\eta,\mathcal{R}_\eta⟩$ using traditional planning algorithms like Value Iteration or Policy Iteration.
- **Sample-Based Planning** → this is a simpler and often more powerful approach. We use the model *only to generate "fake" (simulated) samples* of experience. We can then apply any **model-free** algorithm (like Sarsa or Q-learning) to this stream of simulated data.

---

# 4. Integrated Architecture: Dyna-Q Algorithm

The **Dyna** architecture integrates Model-Free and Model-Based learning. It learns from *both* real experience and simulated experience. The agent repeats a **3-step cycle** at every time-step:

1. **Direct RL** → the agent interacts with the real environment ($s, a \rightarrow r, s'$) and updates its $Q(s,a)$ table via classical Q-learning.
2. **Model Learning** → the agent records its real-world experience, to build an internal model of the environment.
3. **Planning (”Dreaming”)** → uses the learned model to generate $n$ simulated experiences and performs additional Q-learning updates on them.

![image.png](8%20Integrating%20Learning%20and%20Planning/image.png)

<aside>
🔢

**Algorithm:**

![image.png](8%20Integrating%20Learning%20and%20Planning/image%201.png)

</aside>

<aside>
📌

**Performance:**

The maze example shows that adding **planning** ($n>0$) greatly speeds up learning compared to using only real experience ($n=0$):

![image.png](8%20Integrating%20Learning%20and%20Planning/130f0e19-8ef1-44b1-b315-55b04e90403c.png)

</aside>

**N.B.** However, this planning strategy is **naive** because it samples random past experiences. Ideally, planning should focus on the **current state** rather than random transitions from the past. E.g. When playing chess *now*, you don't want to dream about a game from 10 years ago; you want to "dream" (plan) about the *current* board state.

---

# 5. Simulation-Based Search (Smarter Planning)

A sub-category of sample-based planning where, instead of sampling (”dreaming”) random states, the agent performs a **forward search** from the current state $S_t$ (= it simulates possible future actions and selects the one with the highest expected performance).

## 5.1. Simple Monte-Carlo Search

![image.png](8%20Integrating%20Learning%20and%20Planning/image%202.png)

Given a state we simulate the future by taking in consideration all the possible action that we can choose. We simulate many possible trajectories of the tree and then we select the most promising one. 

1. Given a model $\mathcal{M}_v$ and a **simulation policy $\pi$** (usually a random policy).
2. For each action $a \in \mathcal{A}$:
    1. Simulate $K$ episodes from current state $S_t$: $\{s_t, a, R_{t+1}^k, S_{t+1}^k, A_{t+1}^k,...S_T^k\}_{k=1}^K \sim \mathcal{M}_v, \pi$
    2. Evaluate actions by computing the **mean return** over all episodes:
        
        $$
        Q(s_t,a) = \frac{1}{K}\sum_{k=1}^KG_t^k \overset{P}{\to}q_{\pi}(s,a)
        $$
        
        **N.B.** By the Law of Large Numbers, as $K \to \infty$, this converges to the true expected return $q_\pi(s,a)$.
        
3. After search is finished, select current real action with maximum value in tree search:
    
    $$
    a_t = \argmax_{a \in \mathcal{A}}Q(S_t, a)
    $$
    

## 5.2. Monte-Carlo Tree Search (MCTS)

MCTS is a powerful, sample-based search algorithm (famously used by AlphaGo). It repeats $K$ simulations from the current state $S_t$. The simulation process relies on two policies and two conceptual phases:

- **In-Tree Phase** → uses the **Tree Policy** (which improves) to choose actions that **maximize $Q(s,a)$** (exploitation).
- **Out-of-Tree Phase** → uses the **Default Policy** (which is fixed) to choose actions randomly or simply (exploration).

### 5.2.1. The Simulation Loop

The following four steps are repeated for every single simulation, showing how the policies above are executed and how the tree is updated:

1. **Selection** → starting from the root (=current state), the agent follows a tree policy (e.g., UCT) to choose actions until it reaches a node that is not fully explored meaning that at least one available action has not yet been added to the tree.
2. **Expansion** → one or more new child nodes are added, extending the explored portion of the tree.
3. **Simulation (Rollout)** → from the new node, the agent plays until the end of the episode using a simple *Default Policy* (often random actions), obtaining a return $G_t$.
4. **Backpropagation** → the return is propagated back to the root, updating visit counts and $Q(s,a)$ values along the path, improving the *tree policy* (which will be used in the next Selection phase).

![image.png](8%20Integrating%20Learning%20and%20Planning/image%203.png)

<aside>
🚨

**MC Search vs MC Tree Search**

- **Simple MC Search** → evaluates immediate actions by running random simulations to the end, planning only **one step ahead** with no memory of the future [ $K_{tot} = |\mathcal{A}|\cdot K$].
- **Monte Carlo Tree Search** → plans **multiple steps ahead** by dynamically building and storing a search tree in memory. It uses previous simulation results to focus future searches on the most promising paths [$K_{tot}=K$].
</aside>

### 5.2.2. MCTS Formulas and Convergence

MCTS is the application of **Monte-Carlo control** to simulated experience and converges on the optimal search tree, $Q(S,A)→q_*(S,A)$. The core evaluation formula (Backpropagation) is:

$$
Q(s,a)=\frac{1}{N(s,a)}\sum_{k=1}^K\sum_{u=t}^T \mathbf{1}(S_u,A_u=s,a)G_u
$$

**N.B.** $\mathbf{1}(S_u,A_u=s,a)$ is an indicator function that acts like an **if statement**. It is equal to $1$ only when the simulation visits the state-action pair $(s,a)$, and $0$ otherwise.

After running $K$ simulations, the agent selects the **real action** at with the maximum value in the search tree:

$$
a_t=\argmax_{a\in\mathcal{A}}Q(s_t,a)
$$

---