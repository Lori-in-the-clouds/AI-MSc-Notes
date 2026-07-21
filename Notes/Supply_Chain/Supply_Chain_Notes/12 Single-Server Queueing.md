# 12. Single-Server Queueing

Num of pages: 1
Status: Done
Type: theory

# 1. State Variables

To build a simulation, specific components of the real system must be modeled mathematically:

- **Inter-arrival times** → represent the time between one arrival and the next. They are typically modeled using probability distributions and are often assumed to be **IID random variables**, meaning each arrival is independent of the previous one (memoryless behavior). In some settings, however, such as manufacturing, arrivals may follow a fixed, deterministic schedule (e.g., one item every 5 minutes).
- **Service/Processing time** → duration a customer spends being processed by the server. It is typically modeled as an **IID random variable**. Although in real systems service speed might vary (e.g., a worker becoming slower when tired), assuming independence is a practical and common simplification for basic modeling.
- **Queue discipline** → it defines the rule by which customers are selected for service. The most common is **FIFO**, where customers are served in the order they arrive.

![image.png](12%20Single-Server%20Queueing/image.png)

<aside>
🔑

**Simulation Initialization** 

Simulation typically begins in an **Empty and Idle** state at $t = 0$. This means that there are zero customers and the server is inactive. Consequently, the first arrival must occur *after* time zero.

Since real-world initial conditions are unknown, starting empty provides a **neutral and consistent baseline**. This prevents the simulation from being distorted by arbitrary guesses about the initial queue size (bias).

</aside>

---

# 2. Performance Metrics (PKI)

The simulation focuses on three key performance indicators:

- **Expected Average Delay in Queue $d(n)$** → measures the system performance from a **customer’s point of view** and is computed as the average delay of the first $n$ customers who have completed service:
    
    $$
    \hat{d}(n)=\frac{\sum_{i=1}^n D_i}{n}
    $$
    
    where $D_1$, $D_2$, …. are delays of customers, they can be zero too.
    
    **N.B.** Unlike inter-arrival times, delays are **not IID**: they are correlated, since the delay of first customer $D_1$ influences the delay of the second customer $D_2$ and so on. 
    
- **Average Queue Size** $q(n)$ → measures the average number of customers waiting in the queue over time. It is useful from the **manager’s perspective**, since it reflects the level of congestion in the system:
    
    $$
    q(n)=\sum_{i=0}^\infty i\cdot p_i \quad \text{where } \displaystyle \ \hat{p}_i = \frac{T_i}{T(n)}
    $$
    
    Where:
    
    - **$i$**: ****number of customers in the queue.
    - $T_i$: total time the queue remained at length $i$.
    - $T(n)$: total time required to observe $n$ completed delays.
    - $\hat{p}_i$: fraction of time the queue had length $i$.
    
    Equivalently, in integral time form:
    
    $$
    q(n)=\frac{1}{T(n)}\int_0^{T(n)} Q(t)\,dt
    $$
    
    Where $Q(t)$ is the queue length at time $t$.
    
    <aside>
    
    Example:
    
    ![image.png](12%20Single-Server%20Queueing/image%201.png)
    
    </aside>
    
- **Expected Server Utilization $u(n)$** → measures efficiency from the **manager's perspective** (cost efficiency). It represents the **fraction of time the server is busy $B(t)=1$** during the observation period:
    
    $$
    \hat{u}(n)=\frac{\int_{0}^{T(n)}B(t)\ dt}{T(n)}
    $$
    
    <aside>
    📌
    
    **Interpretation:**
    
    - **< 90%** → ****excess capacity (or underutilization).
    - **~ 95%** → ideal utilization; well-used but can handle spikes.
    - **~ 100%** → the server is a **bottleneck**. The entire system's output is limited by this server's rate.
    </aside>
    
    <aside>
    
    Example:
    
    ![image.png](12%20Single-Server%20Queueing/image%202.png)
    
    </aside>
    

---

# 3. Calculation Method: Sampling vs Time-Integration

When computing time-dependent metrics like queue size $q(n)$ or utilization $u(n)$, the methodology is crucial:

- **Discrete Sampling** → pausing the simulation periodically (e.g., every 5 minutes) to record the state.
    
    *Risk:* If the sampling frequency does not match the system dynamics, events are missed. For example, if the queue fills and empties between two sample points, the simulation might incorrectly conclude the queue was always empty:
    
    ![image.png](12%20Single-Server%20Queueing/image%203.png)
    
- **Continuous Time Integration (Correct)** → track exactly how long the system stays in each state. We compute the area under the curve:
    - for $q(n)$, we sum $\text{Queue Length} \times \text{Duration}$
    - for $u(n)$, we sum the busy durations
    
    This yields mathematically correct, time-weighted averages.
    

---