# 1. Introduction to Reinforcement Learning

Num of pages: 1
Status: Done
Type: theory

# 1. Introduction

Reinforcement Learning (RL) is a paradigm where agents learn through interaction with their environment. Agents learn without examples of optimal behavior, instead they optimize a **reward signal** to maximize cumulative rewards over time. The main characteristics are:

- No supervisor, only a reward signal
- Feedback is often delayed, not instantaneous
- Time matters, as data is sequential and **not** independent and identically distributed (**not** i.i.d.)
- The agent's actions influence the subsequent data it receives

---

# 2. Lexicon

- **Agent →** the entity that performs actions and interacts with the environment. It can have one or more of the following components:
    - **Policy ($\pi$):** the agent's behavior function. It's a map from a state to an action. It can be:
        - **Deterministic policy**: $a=\pi(s)$
        - **Stochastic policy**: $\pi(a|s)=\mathbb{P}[A_t=a|S_t=s]$
    - **Value Function ($v_\pi(s)$):** a prediction of future reward. It's used to evaluate the goodness or badness of states and to help select actions. It's defined as the expected cumulative reward starting from a given state:
        
        $$
        v_\pi(s)= \mathbb{E}[R_{t+1}+\gamma R_{t+2}+\gamma^2R_{t+3}+...|S_t=s]
        $$
        
        Where $\gamma$ is the **Discount Factor**, a value between 0 and 1 that trades off the importance of immediate rewards versus long-term rewards.
        
    - **Model**: the agent's internal representation of how the environment works. A model predicts what the environment will do next:
        - **Transition Function →** predicts the next state: **$P_{SS'}^a=\mathbb{P}[S_{t+1}=s'|S_t=s,A_t=a]$.**
        - **Reward Function** → predicts the next reward: $R_S^a=\mathbb{E}[R_{t+1}|S_t=s,A_t=a]$.
    
    **N.B.** All three of these components are functions that can be learned using neural networks, a field known as **Deep Reinforcement Learning.** However, this violates the assumptions of supervised learning (i.e., i.i.d. data), so specific tools are needed to handle this.
    
- **Reward ($R_t$)** → scalar feedback signal indicating how well the agent is performing at a given step.
- **History ($H_t$)** → the sequence of all past observations, actions, and rewards up to time $t$. Formally: $O_1,R_1,A_1,...,A_{t-1},O_t,R_t$.
- **State ($S_t$)** → ****the information used to determine what happens next. Formally, it's a function of the history: $S_t​=f(H_t​)$. We can distinguish two types of states:
    - **Environment State ($S_t^e$):** the environment's private, internal representation. This state is usually not visible to the agent and can be too complex to know.
    - **Agent State ($S_t^a$):** the agent's internal representation, which is the data it uses to choose its next action. It can be any function of the history.
    
    <aside>
    📌
    
    **Information State (Markov State)**
    
    A state is Markov if the future is independent of the past, given the present:
    
    $$
    \mathbb{P}[S_{t+1}|S_t]=\mathbb{P}[S_{t+1}|S_1,...,S_t]
    $$
    
    This means that once the current state is known, the rest of the history can be discarded, significantly reducing memory requirements. The environment state ($S_t^e$) and the history ($H_t$) are both considered Markov.
    
    </aside>
    
- **Environment** → the context with which the agent interacts. It provide observations and rewards in response to the agent's actions. There are two different types:
    - **Markov Decision Process (MDP):** a **fully observable environment** where the agent's observation is a direct and complete representation of the environment's state ($O_t=S_t^a=S_t^e$). In this case a single observation is a **Markov state**, because it contains all the information needed for optimal decision-making.
    - **Partially Observable Markov Decision Process (POMDP): a partially observable environment** where the agent's state is not equal to the environment's state ($S_t^a\ne S_t^e$). Because of this, a single observation is **not a Markov state**. The agent cannot make optimal decisions based on the current observation alone and needs more information.
    
    <aside>
    📌
    
    **POMDP → MDP**
    
    With RL we cannot solve POMDP directly, we have to converting it into MDP. Agent can construct its own state representation $S_t^a$ in 3 main ways:
    
    1. Using Complete History: $S_t^a=H_t$.
    2. Using Beliefs about the Environment: the agent maintains a **probability distribution** over all possible environment states: $S_t^a=(\mathbb{P}[S_t^e=s_1],...,\mathbb{P}[S_t^e=s_n])$.
    3. Using a Recurrent Neural Network: **$S_t^a=\sigma(S_{t-1}^aW_s+O_tW_o)$**
    
    **N.B.** You managed to complete the transaction if $S_t^a$ become a markovian state.
    
    </aside>
    

---

# 3. How an agent interacts in the environment?

In RL, an **agent** interacts with an **environment**. At each time step ($t$), the agent:

- Receives an observation ($O_t$)
- Receives a scalar reward ($R_t$)
- Executes an action ($A_t$)

The environment then receives the action, and emits a new observation ($O_{t+1}$) and reward ($R_{t+1}$).

<aside>
💡

The agent's primary objective is to maximize its **cumulative reward**. A key challenge in RL is **sequential decision-making**, where actions can have long-term consequences. This leads to the **credit assignment problem.**

</aside>

<aside>
🚨

**Credit Assignment problem:** the challenge of determining which specific past action(s) are responsible for a received reward. Rewards can also be delayed, meaning it may be necessary to sacrifice an immediate reward for a larger, long-term gain.

</aside>

---

# 4. Taxonomy

RL agents can be classified based on whether they use an internal model of the environment:

- **Model-Free Agents** → learn without building a model of the environment. They directly learn a policy and/or a value function from their experiences (trial and error).
- **Model-Based Agents** → build or use an explicit **model** of the environment. They learn a policy using its internal model to test actions and make mistakes safely offline before acting in the external environment.

RL agents can also be categorized based on which components they learn or use:

- **Value-Based Agents** → primarily focus on learning a **value function**, which estimates the goodness of a state or action. The policy is not learned directly; it's implicitly derived from the value function.
- **Policy-Based Agents** → directly learn a **policy** function. They don't need a value function to operate.
- **Actor-Critic Agents** → combine both approaches. They have a **policy** (the "actor") that chooses actions and a **value function** (the "critic") that evaluates the actions chosen by the actor.

![image.png](1%20Introduction%20to%20Reinforcement%20Learning/210f1a81-d270-4d1b-bc71-01e74973e73b.png)

---

# 5. Core Problems and Challenges in RL

- **Prediction vs Control**:
    - **Prediction** → given a fixed policy ($\pi$), the goal is to determine how good that policy is by calculating its value function.
    - **Control** → this problem gives you no policy and challenges you to find the **best policy ($\pi_*$).**
- **Exploration vs Exploitation (central dilemma in RL):**
    - **Exploration** → finds more information about the environment.
    - **Exploitation** → exploits known information to maximise reward.
    
    **N.B.** A good RL agent must find the best trade-off between these two to find a good policy without wasting too much reward.
    
- **Planning vs Learning:** this distinction classifies the source of experience used to compute value functions and optimize the policy. Both planning and learning are methods for solving the core problems of prediction and control.
    - **Learning** → improving the policy by interacting with the real environment (trial and error).
    - **Planning** → improving the policy without touching the real environment, by computing internal simulations inside a generative model.

![image.png](1%20Introduction%20to%20Reinforcement%20Learning/image.png)

---