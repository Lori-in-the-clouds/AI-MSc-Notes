# 13. Queueing Models

Num of pages: 2
Status: Done
Type: theory

# 1. Introduction

Queueing models are mathematical frameworks used to analyze systems where customers/jobs arrive, wait for service, are processed by one or more servers, and then leave. The primary goal of using these models (or simulation) is to predict system performance measures, specifically:

- **Server utilization** → the portion of time the server is busy.
- **Queue length →** the number of customers waiting.
- **Delays →** time spent by customers in the system.

**N.B.** A standard queueing model assumes a **single queue**, meaning all customers wait in one line before being served.

---

# 2. State Variables

- **Calling population** → refers to the population of potential customers. We can divided them in:
    - **Infinite calling population:** it assumes that there are so many potential customers that the arrival rate is unaffected by how many are already in the system, new arrivals happen with constant probability.
    - **Finite calling population:** **it is used when only a small number of customers exist. Here, the arrival rate depends on the system state: as customers enter the system, fewer remain outside to generate new arrivals, so the arrival rate decreases.
        
        *Example:* in a hospital ward with only 5 patients, if all 5 are currently requesting attention, no further requests can arrive.
        

![image.png](13%20Queueing%20Models/image.png)

- **System Capacity** → defines how many customers can be in the waiting line:
    - **Infinite Capacity:** the queue can grow indefinitely (the standard assumption).
    - **Finite Capacity:** when the system is full, new arrivals are blocked and must leave or return to the calling population.
        - Arrival rate $\lambda$: the rate at which customers *try* to arrive.
        - Effective Arrival Rate $\lambda_e$: the number of customers who actually enter the system.
        
        **N.B.** When capacity is finite, $\lambda_e < \lambda$ because blocked customers cannot join the system.
        
- **Arrival Process** → this variable characterizes how potential customers enter the system.
    1. **Types of Arrivals** → arrivals generally occur:
        - At scheduled times
        - Randomly (modeled by a probability distribution)
            
            <aside>
            📌
            
            **The Poisson Arrival Process (Infinite Population)**
            
            This is the standard model for **random arrivals** from a large (infinite) population. It assumes customers make **independent decisions about when to arrive**. Given an arrival rate $\lambda$:
            
            - The inter-arrival time between two consecutive customers follows an **Exponential Distribution**: $A_n \sim \text{Exponential}(\lambda)$
            - The number of arrivals $N$ in an interval  $\Delta t$ follows a **Poisson Distribution**:  $N(\Delta t) \sim \text{Poisson}(\lambda\cdot \Delta t)$
            
            **N.B.** This process is "memoryless”, the probability of a new arrival is independent of how much time has passed since the last one.
            
            </aside>
            
    2. **Single vs Batch Arrivals** → arrival can occur:
        - One at a time: customers arrive one by one (most common model)
        - In batches: customers arrive in groups. The group size can be fixed or variable (random)
    3. **Impact of Calling Population** → the nature of the arrival process depends heavily on the population size:
        - Infinite Calling Population: the arrival rate $\lambda$ is constant. The number of customers already in the system **does not** influence new arrivals.
        - Finite Calling Population: the arrival rate is variable. It depends on the number of customers currently "outside" the system (i.e., the *pending customers*).
            
            <aside>
            📌
            
            **Pending Customer and Runtime of a Customer**
            
            - **Pending Customer** → a customer currently in the calling population (not in the queue or being served).
            - **Runtime** → the time a customer spends outside the system before returning.
            
            ---
            
            *Example (Machine-Repair Problem):*
            
            Machines are the "customers". A machine running smoothly is a "pending customer". Its **Runtime** is the **Time-to-failure** (often modeled with Exponential, Weibull, or Gamma distributions).
            
            </aside>
            
- **Queue Discipline** → refers to the logical ordering of customers in the queue, determining who is served next:
    - **FIFO (First-In-First-Out):** customers are served in order of arrival (standard policy)
    - **LIFO (Last-In-First-Out):** last arrived is served first
    - **SIRO:** service in random order
    - **SPT:** shortest processing time first
    - **PR:** priority service (e.g. emergencies)
- **Queue Behaviour** → refers to the specific actions customers may take while waiting:
    - **Balking:** a customer sees a long queue and decides not to join (leaves immediately).
    - **Reneging:** a customer joins the queue but leaves after waiting for a certain amount of time without being served.
    - **Jockeying:** a customer moves from one line to another (e.g., switching lanes in traffic).
        
         **N.B.** This can be only applied to systems with multiple queues, not the standard single-queue model.
        
- **Service Centers** → a **service center** consists of $c$ **parallel servers** that share a **single common queue**. Customers wait in one line and always move to the **first available server**. A center may have:
    - Single server **→** $c = 1$
    - Multiple servers → $1 < c < \infty$
    - Infinite servers → $c = \infty$, meaning there is **no waiting line**: every customer is served immediately. A typical example is a **self-service system**, where each user can start service without waiting for others.

---

# 3. Queueing Notation

Kendall’s notation system is $A / B / c / N / K$, where:

- $A$ → inter-arrival time distribution
- $B$ → service time distribution
- $c$ → number of parallel servers
- $N$ → system capacity (omitted if infinite)
- $K$ → size of calling population (omitted if infinite)

<aside>
📌

**Common Distribution Symbols (for A and B):**

- **$M$ (Markovian)** → exponential distribution (Memoryless)
- **$D$ (Deterministic)** → constant times
- **$G$ (General)** → arbitrary distribution
</aside>

<aside>

Examples:

- **M/M/1:** exponential arrivals, exponential service, 1 server (Infinite capacity/population assumed).
- **M/M/1/5/5:** the nurse example (1 server, capacity 5, population 5).
</aside>

---

# 4. Evaluating Metrics

To analyze a queueing system effectively, we use specific metrics to measure performance. These metrics help determine if a system is stable, efficient, or congested.

## 4.1. Time-Based Metrics

- **Service Time $S$** → time required to process a customer *at the server*. It **does not** include waiting time in the queue.
- **Service rate $\mu$** → number of customers the server can process per time unit. It is the inverse of the average service time: $E[S]=\frac{1}{\mu}$.
- **Waiting Time $W_Q$** → average time spent in the queue.
- **Total Time Spent in a system $W$** → average time in the system, which is the sum of waiting time and service time. For the $n$-th customer: $W_n^{(i)} = W_{Qn}^{(i)} + S_n^{(i)}$.
- **Server Utilization** $\rho$ → represents the proportion of time in which the server is busy. For a single server ($c=1$):
    
    $$
    \rho=\lambda E[S]=\frac\lambda\mu
    $$
    
    In reality, a system can contain more than one server $c$. In this general case, the utilization is averaged across all servers:
    
    $$
    \rho = \frac{\lambda}{c\mu}
    $$
    
    <aside>
    📌
    
    **Stability Condition**
    
    We compare the arrival rate $\lambda$ with the service capacity $\mu$:
    
    - if $\lambda < c\mu$ ($\rho<1$) → the system is **stable.**
    - if $\lambda > c\mu$ → the server is saturated. The queue will grow indefinitely at a rate of $(\lambda−c\mu)$ customers per time unit.
    </aside>
    

## 4.2. Count-Based Metrics

- **Average Number of Customers in System** $L$:
    
    $$
    \hat{L}=\frac1T\sum_{i=0}^\infty iT_i = \frac1T\int_0^TL(t)\ dt \quad \hat{L} \rightarrow L \quad \text{as}\quad T \rightarrow \infty
    $$
    
    Where $T_i$ is the total time during which the system contained exactly $i$ customers and $L(t)$ is total number of customers inside the system at a specific time $t$.
    

![image.png](13%20Queueing%20Models/7de4f2b7-9530-446c-b5db-da46728404c1.png)

- **Average Number of Costumers in queue** $L_Q$:
    - For a single server ($c=1$):
        
        $$
        \hat{L}_{Q}= \frac1T\sum_{i=0}^\infty i\cdot T_i^Q=\frac1T\int_0^TL_Q(t)\ dt
        \quad \hat{L}_Q \rightarrow L_Q \quad \text{as}\quad T \rightarrow \infty
        
        \qquad
        
        L_{Q}(t) = \begin{cases}    0 & \text{if } L(t) = 0 \\   L(t) - 1 & \text{if } L(t) \ge 1 \end{cases}
        $$
        
    - For $c$ servers:
        
        $$
        L_{Q}(t) = \begin{cases}    0 & \text{if } L(t) < c \\   L(t) - c & \text{if } L(t) \ge c \end{cases}
        $$
        
- **Average Number of Customers Being Served $L_s$:** (average number of busy servers)
    
    $$
    \hat{L_s}=\frac1T\int_0^T\left[L(t)-L_Q(t)\right]dt
    $$
    
- **Average Time Spent in the System $w$:**
    
    $$
    \hat{w}=\frac1N\sum_{i=1}^N W_i
    \quad \hat{w} \rightarrow w \quad \text{as}\quad N \rightarrow \infty
    $$
    
- **Average Time Spent in the Queue** $w_Q$:
    
    $$
    \hat{w}_Q=\frac1N\sum_{i=1}^N W_i^Q \quad \hat{w}_Q \rightarrow w_Q \quad \text{as}\quad N \rightarrow \infty
    $$
    

## 4.3. Little’s Law/Conservation Equation

Little’s Law states that the **average number of customers in the system** ($L$) is equal to the **average number of arrivals per time unit** ($\lambda$), times the **average time spent in the system** ($W$):

$$
L=\lambda W
$$

This result remains valid ****for **any stable queueing system**, regardless of arrival/service distributions, number of servers, or queue discipline.

<aside>
🔢

**Derivation:**

Little’s Law start from the fact that the **total time spent** by all customers (=horizontal sum) is equal to the **area under the curve** $L(t)$:

1. Total time spent by $N$ customers: $\displaystyle\sum_{i=1}^N W_i$
2. This is equal to the area under $L(t)$:  $\displaystyle\sum_{i=1}^N W_i = \int_0^T L(t)\, dt$
3. Divide both sides by the observation time $T$:
    
    $$
    \frac{1}{\textcolor{orange}T}\sum_{i=1}^NW_i=\frac{1}{\textcolor{orange}T}\int_0^TL(t)dt\quad \Rightarrow \quad
    
    \textcolor{green}{N}\cdot\frac{1}{\textcolor{orange}T }\left(\frac{1}{\textcolor{green}{N}}\sum_{i=1}^NW_i\right)=\frac{1}{\textcolor{orange}T}\int_0^TL(t)dt
    $$
    
4. Rewrite using empirical estimates:
    
    $$
    \underbrace{\frac{N}{T}}_{\hat{\lambda}} \cdot \underbrace{\left( \frac{1}{N} \sum_{i=1}^{N} W_i \right)}_{\hat{W}} = \underbrace{\frac{1}{T} \int_{0}^{T} L(t) \, dt}_{\hat{L}}
    $$
    

![image.png](13%20Queueing%20Models/b4ecbf54-f2e8-4197-8474-0a195c8c179c.png)

</aside>

---