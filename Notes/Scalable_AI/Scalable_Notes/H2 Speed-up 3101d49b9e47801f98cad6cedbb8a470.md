# H2. Speed-up

Pages: 2
Status: Done
Type: theory

# 1. Speed-up

A key metric in the context of **parallel computing** is **speedup**, which measures the improvement in execution time achieved through parallelization. There are two main types of speedup:

- **Parallel speedup →** compares the execution time of a program running sequentially (usually on a single CPU core) with its execution in a parallel environment (multi-core CPU or GPU). It measures the absolute performance gain from parallelization and how effectively additional hardware resources are used.
- **Comparative speedup** → evaluates the performance difference between two parallel implementations or hardware configurations. Rather than using a serial baseline, it compares, for example, an MPI-based CPU implementation with a GPU-optimized version on the same node. This metric helps determine which parallel strategy or architecture is more efficient for a specific computational task.

## 1.1. Strong Scaling vs Weak Scaling

Performance scalability in parallel computing is evaluated by how workload changes as the number of processors increases:

- **Strong Scaling** → we measure how the **execution time decreases** when more resources are used for the **same problem size.** **[Equal problem size, more resources]**
- **Weak Scaling** → we measures how a system performs when the problem size increases at the same rate as the resources. The goal is to increase throughput by solving larger problems in approximately the same time. **[Bigger problem size, equal time]**

![image.png](H2%20Speed-up/image.png)

## 1.2. Amdahl's Law (Strong Scalability)

If the size of the problem doesn’t grow with the number of processors, the speedup is expressed as:

$$
S=\frac{T_1}{T_n}=\frac{s + p}{s+\frac p n}   =\frac{\textcolor{red}{1}}{(1-p) + \frac{p}{n}}
$$

Where:

- $p$ → is the proportion of a program that can be made parallel
- $s$ → serial fraction of the program
- $n$ → number of processors

<aside>
🚨

Since the problem size remains fixed, we normalize the original execution time to $\textcolor{red}{1}$ (the numerator) in order to measure how much the execution time decreases as more processors are added.

</aside>

**N.B.** This formula states that the maximum improvement in speed of a process is limited by the proportion of the program that can be made parallel.

## 1.3. Gustafson-Barsis's Law (Weak Scalability)

If the size of the problem grows proportionally to the number of processors, the speedup is now expressed as:

$$
S = \frac{T_1}{T_n}=\frac{s + p\cdot N}{\textcolor{blue}{1}} = s+(1-s)\cdot N = N + (1-N)\cdot s
$$

- $N$ is the number of processors
- $s$ and $p$ are the fractions of time spent executing the serial parts and the parallel parts of the program on the parallel system ($s + p = 1$).

<aside>
🚨

Since time remains fixed, we normalize the parallel execution time to $\textcolor{blue}{1}$ (e.g., 1 hour of work) and calculate how long a single processor would need to complete the same amount of work alone.

</aside>

**N.B.** The result is a larger problem that can be solved in the same time by using more processors.

---

# 2. Affinity

When a thread is assigned to a specific core (e.g., Core 1), it caches local memory in that core. If the OS dynamically shifts that thread to a different core, the entire cached memory block must be copied over, causing a severe performance overhead penalty. 

<aside>
💡

**Affinity** is the deliberate practice of "pinning" a thread to a specific core to guarantee it never jumps, securing localized memory efficiency.

</aside>

In multi-socket systems, scheduling policies determine how tasks are mapped onto hardware:

- **Compact Scheduling** → fills one socket before using another. It maximizes cache sharing and data locality but may create thermal hotspots.
- **Round Robin Scheduling** → alternates tasks across sockets. It improves load balancing and memory bandwidth usage, but may increase inter-socket latency.
- **Random (Stupid) Scheduling** → no structured allocation. It can overload some cores while leaving others underutilised, causing performance degradation.

![image.png](H2%20Speed-up/image%201.png)

---

# 3. Collective Communications

In parallel and distributed systems, processes need to exchange data efficiently during operations. This takes palace exploiting different communication patterns:

- **Master and Slaves Operations:**
    - **Broadcast** → a root node sends the same data to all other nodes.
        
        ![Screenshot 2026-03-11 at 11.14.31.png](H2%20Speed-up/Screenshot_2026-03-11_at_11.14.31.png)
        
    - **Scatter** → the root node splits data into chunks and sends one portion to each process.
        
        ![image.png](H2%20Speed-up/image%202.png)
        
    - **Gather** → a root node collects data from all processes and concatenates them into a single vector.
        
        ![image.png](H2%20Speed-up/image%203.png)
        
    - **Reduce** → a root node collects data from all nodes and applies an operation (e.g., sum, max, or average).
        
        ![image.png](H2%20Speed-up/image%204.png)
        
- **All-to-All Operations:**
    - **Reduce-Scatter** → data from all nodes is first **combined through a reduction operation**, and the resulting aggregated data is **split and distributed across all nodes**, so each node receives only a portion of the final result.
        
        ![image.png](H2%20Speed-up/image%205.png)
        
    - **All-Gather** → every node receives the data from all other nodes, resulting in each node having a complete copy of the global dataset.
        
        ![image.png](H2%20Speed-up/image%206.png)
        
    - **All-to-All** → each node sends a different block of data to every other node. This operation is equivalent to a **matrix transpose** and is commonly used in **self-attention mechanisms**.
        
        ![image.png](H2%20Speed-up/image%207.png)
        
    - **All-Reduce** → all nodes sends its data and each node receives the final aggregated result. This is the **most widely used collective in distributed deep learning**, especially for **gradient synchronisation**.
        
        ![image.png](H2%20Speed-up/image%208.png)
        

## 3.1. Ring-Based All-Reduce

All-Reduce is often implemented using a **ring algorithm**, which works well in fast distributed systems. It has two main phases:

1. **Reduce Scatter** → GPUs are arranged in a logical ring. During each step:
    1. A GPU sends a chunk of its data to its **right neighbor**.
    2. It simultaneously receives a chunk from its **left neighbor**.
    3. The received data is combined with the local data and forwarded.
    
    After $N − 1$ **steps**, each GPU holds one piece of the final reduced result.
    
2. **All-Gather** → each GPU sends its partial result to others in the ring, after $N − 1$ **steps**, every GPU has the full final result.

![image.png](H2%20Speed-up/image%209.png)

<aside>
📌

**Neighbor computation**

For a GPU at position $pos$:

- Right neighbor → $(pos + 1)\ \%\ N$
- Left neighbor → $(pos - 1 + N)\ \% \ N$

---

**Bandwith Optimality**

Ring All-Reduce is considered **bandwidth-optimal** because:

- **Full utilization:** each GPU continuously sends and receives data during the $2(N-1)$ communication rounds.
- **Data segmentation:** the tensor is split into $N$ chunks, so as $N$ increases, each chunk becomes smaller, keeping the total communication per GPU roughly constant.
</aside>

## 3.2. **Communication Optimizations**

- **Non-blocking collectives** allow GPUs to continue computation while communication is in progress, improving overall efficiency.
- **Ghost Cell:** a technique used to **reduce communication latency** by sending data **in advance**, so the latency cost is paid only once and **amortized** across multiple computations.

<aside>
📌

**NCCL (Accelerating multi-GPU collective communications)**

**NCCL** is a library that accelerates **collective communications in multi-GPU systems**, providing **topology-aware, high-performance collectives** that scale efficiently.

</aside>

---

# 4. **PyTorch Distributed**

PyTorch provides tools to run **distributed training** across multiple GPUs and machines. The standard approach is **Distributed Data Parallel (DDP)**, where each GPU runs a separate process and gradients are synchronized during training.

<aside>
🔑

**Key Concepts:**

- **Node** → a physical machine participating in the training. A node can contain multiple GPUs.
- **World Size** → total number of processes (usually equal to the total number of GPUs) across all nodes.
- **Global Rank** → unique identifier of each process in the entire distributed system, ranging from $0$ to 

$\text{world\_size} - 1$ .
- **Local Rank** → identifier of a process within its node, ranging from $0$  to $(\text{GPUs\_per\_node} - 1)$.
- **Environment Variables** → ranks and configuration parameters are typically passed to the program through environment variables (e.g., `RANK`, `LOCAL_RANK`, `WORLD_SIZE)`.

**N.B.** Applications can use the **rank** to assign different roles. For example, the process with $\text{global\_rank} = 0$ often handles tasks such as logging or checkpointing.

![image.png](H2%20Speed-up/image%2010.png)

</aside>

## 4.1. Launcher

It’s a utility that starts the distributed training job by **spawning processes and assigning environment variables**. These variables determine the role of each process and how data is distributed. Examples include:`torchrun`, `mpirun` (recommended), `srun`.

![image.png](H2%20Speed-up/image%2011.png)

## 4.2. PyTorch Parallelism Modes

- **DataParallel (DP) [`torch.nn.DataParallel`]**  →  an older **single-process, multi-thread** method for training on multiple GPUs within a single node. It is now **deprecated and inefficient**.
- **DistributedDataParallel (DDP) [`torch.nn.parallel.DistributedDataParallel`]**  → the **recommended standard**. Each GPU runs its own process, and gradients are synchronized using **All-Reduce** operations.
- **Torch Elastic [`torch.distributed.elastic`]** → a more advanced framework that allows **dynamic scaling and fault tolerance**, but it is rarely used in standard setups.

---