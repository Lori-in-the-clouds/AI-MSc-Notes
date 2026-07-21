# 3. Learning in MDP (Planning)

Num of pages: 1
Status: Done
Type: theory

# 1. What is Dynamic Programming

**Dynamic Programming (DP)** is a general method for solving complex problems. It consist in:

1. Divide problems in sub-problems
2. Find partial solutions
3. Combine solutions to subproblems

It can be applied to problems with the following properties:

- **Principle of optimality** → optimal solution can be decomposed in recursive way in partial optimal solutions.
- **Overlapping subproblems** → sub-problems’s solutions are used multiple times.

MDP satisfy these properties thanks to Bellman equation. So It is used for **planning** in an MDP.

---

# 2. The Main DP Algorithms

DP provides several algorithms for planning in MDPs:

1. **Iterative Policy Evaluation** → this algorithm solves the **prediction** problem. Given a **fixed** policy $\pi$, the goal is to find its true value function $v_\pi$. The algorithm works by iteratively applying the Bellman Expectation Equation. At each iteration $k+1$, it updates the value of every state using the value function from the previous iteration, $v_k$:
    
    $$
    v_{k+1}(s)=\sum_{a\in \mathcal{A}}\pi(a|s)\overbrace{(\mathcal{R}_s^a+\gamma\sum_{s'\in \mathcal{S}}\mathcal{P}_{ss'}^av_k(s'))}^{q_\pi(s,a)}\quad \Rightarrow \quad v^{k+1}=\mathcal{R}^\pi+\mathcal{P}^\pi v^k
    
    $$
    
    This is a **synchronous backup** method, where all states are updated in parallel. The process is guaranteed to converge to the true value function $v_\pi$.
    
2. **Policy Iteration** → this algorithm solves the **control** problem by alternating between two steps, given a policy $\pi$:
    1. **Policy Evaluation**: the algorithm estimates the true value function $v_\pi$ for the current policy $\pi$ using Bellman Expectation Equation.
    2. **Policy Improvement**: it generates a new, improved policy $\pi'$ by acting greedily with respect to the current value function $v_\pi$.
    
    ![image.png](3%20Learning%20in%20MDP%20(Planning)/image.png)
    
    **N.B.** This process is guaranteed to converge to the optimal policy $\pi_*$.
    
    <aside>
    📌
    
    **Policy Improvement** 
    
    First, we consider a deterministic policy $a=\pi(s)$, and define a new improved policy $\pi'$ by choosing the action that maximizes the action-value function, $q_\pi(s,a)$:
    
    $$
    \pi'(s)=\argmax_{a\in \mathcal{A}}q_\pi(s,a)
    $$
    
    The action chosen by the new greedy policy is guaranteed to have an action-value that is greater than or equal to the value of the old policy:
    
    $$
    q_\pi(s,\pi'(s))=\max_{a\in \mathcal{A}}q_\pi(s,a){\color{red}\ge} q_\pi(s,\pi(s))=v_\pi(s)
    $$
    
    If improvements stop:  $q_\pi(s,\pi'(s))=\max_{a\in \mathcal{A}}q_\pi(s,a)=q_\pi(s,\pi(s))=v_\pi(s)$.
    
    Then the Bellman optimality equation has been satisfied:  $\max_{a\in \mathcal{A}}q_\pi(s,a)=v_\pi(s)$.
    
    ---
    
    **Proof of Policy Improvement:**
    
    This one-step improvement extends to the long-term, proving that the new policy's value function is also better or equal to the old one, i.e., $v_{\pi'}(s)\ge v_\pi(s)$. This is demonstrated through the following chain of inequalities:
    
    $$
    \begin{align*}\textcolor{orange}{v_{\pi}(s)} & {\color{red}\leq} q_{\pi}(s, \pi'(s))= \mathbb{E}_{\pi'} \big[ R_{t+1} + \gamma v_{\pi}(S_{t+1}) \mid S_t = s \big] \\&\leq \mathbb{E}_{\pi'} \big[ R_{t+1} + \gamma q_{\pi}(S_{t+1}, \pi'(S_{t+1})) \mid S_t = s \big] \\&\leq \mathbb{E}_{\pi'} \big[ R_{t+1} + \gamma R_{t+2} + \gamma^{2} q_{\pi}(S_{t+2}, \pi'(S_{t+2})) \mid S_t = s \big] \\&\leq \mathbb{E}_{\pi'} \big[ R_{t+1} + \gamma R_{t+2} + \dots \mid S_t = s \big] \\&= \textcolor{orange}{v_{\pi'}(s)}\end{align*}
    $$
    
    </aside>
    
    <aside>
    🧑‍💻
    
    **Pseudocode**
    
    1. Initialize $\pi(a|s)$
    2. While !($\pi(a|s)$ stable):
        1. While !($v_{\pi}(s)$ stable):
            1. $v_{k+1}(s) = \sum_{a \in \mathcal{A}} \pi(a|s)\left[\mathcal{R}_s^a + \gamma \sum_{s' \in \mathcal{S}} \mathcal{P}_{ss'}^a v_k(s')\right] \quad \forall s \in S$
        2. $\pi(a|s) = \argmax_a q_{\pi}(s,a)=\argmax_a \left[\mathcal{R}_s^a + \gamma \sum_{s' \in \mathcal{S}} \mathcal{P}_{ss'}^a v_{\pi}(s')\right]\quad \forall s \in S$
    </aside>
    
3. **Value Iteration** → is a variant of Policy Iteration where **each individual policy evaluation phase** is truncated after just one iteration, rather than running to full convergence for that specific policy. It directly applies the Bellman Optimality Equation to find the optimal value function $v_*$. The policy is **not explicitly** represented or improved during the process. The optimal policy $\pi_*$ is extracted from the final value function at the end of the algorithm.
    
    ![image.png](3%20Learning%20in%20MDP%20(Planning)/image%201.png)
    
    <aside>
    🧑‍💻
    
    **Pseudocode**
    
    1. Random $\pi(a|s)$
    2. While !($v_{k+1}(s)$ stable):
        1. $v_{k+1}(s) = \max_a [\mathcal{R}_s^a + \gamma \sum_{s' \in \mathcal{S}} \mathcal{P}_{ss'}^a v_k(s')]\quad \forall s \in S$
    3. $\pi(a|s) = \argmax_a q_{\pi}(s,a) = \argmax_a [\mathcal{R}_s^a + \gamma \sum_{s' \in \mathcal{S}} \mathcal{P}_{ss'}^a v_{\pi}(s')] \quad \forall s \in S$
    </aside>
    
    <aside>
    💡
    
    **Why the evaluation phase is truncated after just one iteration?**
    
    Because of the $\max_a$ operator, it is impossible to perform multiple evaluation iterations on the same policy. After each update, the greedy action may change, which means the policy itself changes immediately. Instead, the policy $\pi$ is explicitly frozen and held constant during the evaluation phase, in that way you achieve $V^\pi$.
    
    </aside>
    

---