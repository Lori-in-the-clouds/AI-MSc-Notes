# H1. Hardware and architectural foundations for large-scale

Pages: 2
Status: Done
Type: theory

# 1. Introduction

<aside>
💡

Artificial Intelligence has become a **computationally intensive field**, requiring large-scale infrastructure. As **Moore’s Law** reached its physical limits due to power and heat constraints, single-core performance improvements slowed. This motivated the adoption of **accelerated and parallel computing**, such as GPUs and multi-core systems, to handle modern AI workloads.

</aside>

Training large models means minimizing a loss function in a very high-dimensional space (e.g., GPT-3 with 175 billion parameters corresponds to a 175-billion-dimensional optimization problem). Although a neural network is mathematically defined, it is too complex to solve analytically, so iterative methods like **Stochastic Gradient Descent (SGD)** use mini-batches to approximate the gradient and reach a local minimum.

<aside>
📌

**Core Operations in Deep Learning**

The core operations in deep learning are **matrix multiplication and accumulation**, as dense layers propagate information and capture features through successive layers.

</aside>

**N.B.** Achieving **100% training accuracy** usually indicates **overfitting**, meaning the model memorizes instead of generalizing. For exact lookup tasks, a **hash map** is far more efficient than a neural network.

---

# 2. CPU vs GPU

- **CPU (Host)** → is composed of a small number of cores designed for general-purpose, serial tasks. CPU core is complex:
    - ALU and FPU (Floating Point Unit)
    - Control Unit (including branch prediction and instruction scheduling)
    - L1, L2, and often shared L3 cache
    - many registers
    
    N.B. Its architecture focuses on **minimizing latency** within each thread.
    
- **GPU (Device)** → is composed of hundreds of cores that can run thousands of threads at the same time. CUDA core is simple:
    - ALU and FPU
    - Registers (fewer than in CPU cores)
    - L1 and L2 caches and shared memory
    
    **N.B.** It is specialized for operations like **matrix multiplications** and is designed for **high throughput**. 
    
    In GPUs, **Tensor Cores** can also be used. These are specialized GPU cores designed to speed up **matrix multiplication and accumulation**, which are the main mathematical operations used in deep learning and AI.
    
    ![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image.png)
    

![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%201.png)

<aside>
🔑

**Performance Metrics**

- **Latency**: time required for a single data packet to travel from source to destination. The goal is typically to **minimize latency**.
- **Bandwidth**: the theoretical maximum data transfer rate (e.g., 900 GB/s with NVLink).
- **Throughput**: the amount of work completed per unit of time.

![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%202.png)

</aside>

## 2.1. Execution Time

Performance also depends on libraries:

- `numpy` → mono-thread
- `torch` → multi-thread

When evaluating performance, different time measurements must be considered:

- **Wall Time (Wall-clock time)** → the total real-world time elapsed from start to finish, including system calls, data movement, printing outputs, and returning the final result.
- **CPU Time** → the exact amount of time the CPU actually spent exclusively processing the instruction.
- **User Time vs Sys Time** → on Linux platforms, execution is split between:
    - **User Time** → time spent in user space (application logic).
    - **System Time (Sys Time)** → time spent in kernel space due to system calls (e.g., hardware access, file I/O).

![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%203.png)

---

# 3. System Architecture

## 3.1. Flynn's Taxonomy

**Flynn’s Taxonomy** classifies computer architectures based on the flow of information through a processor, identifying two main streams:

- **Instruction Stream** → the sequence of commands executed by the processor.
- **Data Stream** → the flow of data between memory and the processor.

According to this model, the **instruction** and **data streams** can each be single or multiple, resulting in **four main architectural types**.

- **SISD:** Single Instruction, Single Data.
- **MISD:** Multiple Instruction, Single Data.
- **SIMD:** Single Instruction, Multiple Data (historically the most famous parallel architecture type).
- **MIMD:** Multiple Instruction, Multiple Data.

![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%204.png)

<aside>
📌

**SIMT (Single Instruction, Multiple Threads)**

**SIMT** (Single Instruction, Multiple Threads) is like SIMD for GPUs: multiple threads execute the same instruction (a kernel) on different data at the same time. Unlike SIMD, which uses fixed hardware vector units, SIMT uses independent software threads, offering more **flexibility** and easier **scalability**.

</aside>

## 3.2. **CUDA Execution Hierarchy**

1. **Thread** → the smallest execution unit, like a single worker performing one operation.
2. **Warp** → a group of 32 threads that execute together at the same time.
3. **Thread Block** → a collection of multiple warps (up to 1024 threads). Threads inside a block can cooperate, share data, and synchronize. Each block runs on a single SM.
4. **Wave** → a group of thread blocks executed simultaneously on the available SMs.
5. **Grid** → the collection of all thread blocks launched for a kernel. It represents the entire workload assigned to the GPU.

## 3.3. Processing Flow

1. **Data Transfer (CPU → GPU)** → input data is copied from CPU memory (host) to GPU memory (device) through the PCIe bus. Since data transfer is relatively slow, minimizing memory movement is important for performance.
2. **Kernel Execution** 
    1. When a kernel is launched, the GPU receives a **Grid** of work composed of multiple **Thread Blocks**.
    2. The **GigaThread Engine** distributes these thread blocks across the available **Streaming Multiprocessors (SMs)**.
        
        <aside>
        📌
        
        **Streaming Multiprocessor (SM)**
        
        It’s the management unit that coordinates resources and schedules warps. Each SM is composed of:
        
        - **CUDA Cores** → execution units inside the SM that perform arithmetic and logic operations. They support different instruction types depending on the operation being executed:
            - **INT32** is for whole numbers (integer) operations
            - **FP32** (Single Precision) is the standard for AI and graphics
            - **FP64** (Double Precision) is reserved for high-accuracy scientific simulations
        - **Warp Manager**
        - **Cache L1** → for thread communication
        - **Register file** → used to store intermediate values during computation (idle warps)
        
        ![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%205.png)
        
        </aside>
        
        **N.B.** A thread block cannot be split across multiple SMs.
        
    3. Once an SM receives one or more thread blocks:
        - The hardware immediately divides each thread block into **Warps** (groups of 32 threads).
        - A **Warp Scheduler** manages execution. Since memory access is much slower than computation, the scheduler continuously switches between warps: if one warp is waiting for data, another ready warp is executed instead. This mechanism is called **latency hiding**.
    4. Now execution begins: the GPU follows the **SIMT (Single Instruction Multiple Threads)** model, where all threads in a warp execute the same instruction simultaneously on different data.
        
        <aside>
        🚨
        
        **Warp Divergence:**
        
        If threads in the same warp take different paths (e.g., an `if-else`), execution becomes **serial** (one after another), which greatly reduces performance. This happens because the GPU can generally execute **only one operation at a time** per warp, so different threads must wait their turn.
        
        ---
        
        **Waves and Tail Effect**
        
        Since the GPU has a limited number of SMs, blocks are executed in multiple waves. If the final wave contains only a few blocks, many SMs remain idle, causing the **tail effect** and reducing hardware utilization.
        
        </aside>
        
3. **Result Transfer (GPU → CPU)** → once execution is completed, the results are copied back from GPU memory to CPU memory.

---

# 4. Multithreading vs Multiprocessing

**Multithreading** and **multiprocessing** are both used for parallelism, but they differ in implementation and resource management:

- **Multithreading** → a single process creates multiple threads that share the same memory space. This approach is highly efficient for **I/O-bound tasks**, where the system spends time waiting for external resources.
    
    <aside>
    📌
    
    **The Global Interpreter Lock (GIL)**
    
    Python is often effectively single-threaded due to the **Global Interpreter Lock (GIL)**, which allows only one thread to execute at a time. As a result, multithreading is suitable mainly for **I/O-bound tasks**, not **CPU-bound tasks**. 
    
    ![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%206.png)
    
    **N.B.** For true parallelism in CPU-intensive workloads, the **multiprocessing** module is recommended, as each process has its own memory space and interpreter, bypassing the GIL.
    
    </aside>
    
- **Multiprocessing** → multiple processes run in parallel, each with its own memory space and threads.  It is particularly well-suited for heavy **CPU and GPU-bound operations**, where maximizing raw processing throughput is the primary goal. This architecture is primarily categorized into two types:
    - **SMP (Symmetric Multiprocessing):** all processes perform the same identical role, sharing uniform operating system privileges and memory. In an SMP system, a single OS manages all cores simultaneously and can dynamically schedule any process on any core
    - **AMP (Asymmetric Multiprocessing):** processes have dedicated, distinct tasks (e.g., one core handles the OS, while others handle I/O or computations). Common in embedded systems, AMP can be more cost-effective when a single CPU is sufficient for specific tasks.
    
    **N.B.** In general a core executes one thread at time.
    

## 4.1. **How we can achieve Parallelism?**

Modern computing achieves parallelism at three hardware levels:

- **Multiprocessor Systems** → multiple **physical CPUs** on a single motherboard. In **SMP (Symmetric Multiprocessing)**, all CPUs are identical and share the same privileges. Each CPU is installed in its own dedicated socket.
- **Multicore Architectures** → multiple CPU cores in a **single chip**, sharing a slot or socket. Most multicore processors use the **SMP** model, with each core acting as a peer.
- **Hardware Multithreading** (=Hyper-Threading for intel) → a single CPU core manages multiple **execution threads** simultaneously. These “virtual cores” share the core’s functional units and caches, keeping the processor busy even when one thread waits for memory or I/O.

![Screenshot 2026-02-25 at 13.35.57.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/Screenshot_2026-02-25_at_13.35.57.png)

**N.B.** Hardware levels of parallelism are **nested**: a system can be **multiprocessor** (multiple chips), where each chip is **multicore** (multiple units), and each core performs **hardware multithreading** (multiple threads per unit).

---

# 5. Parallelism

<aside>
💡

**Parallel Computer**

A **parallel computer** is a collection of processing elements that cooperate to solve large problems efficiently. Designing such systems requires addressing three key aspects:

- **Resource Allocation** → determine the number and power of processing elements.
- **Data Access, Communication, and Synchronisation** → define how elements exchange data, coordinate tasks, and maintain consistency.
- **Performance and Scalability** → evaluating system speed and how well performance improves as more processing elements are added.
</aside>

## 5.1. Serial vs Parallel Processing

The distinction between **serial** and **parallel** processing is fundamental to system performance:

- **Serial Processing:** a process in which its sub-processes happen sequentially in time. Speed depends only on the rate at which each sub-process will occur (e.g. processing unit clock speed). Execution speed is determined only by the frequency at which each sub-task is completed.
- **Parallel Processing:** multiple sub-processes run simultaneously. System performance depends on:
    1. The speed of each processing unit
    2. The degree of concurrency (how many tasks run at the same time)
    
    By distributing work across multiple units, parallel processing overcomes the physical limits of serial execution.
    

## 5.2. Classification of Parallel Architectures

Parallel architectures can be classified according to different design aspects:

1. **Flynn’s Taxonomy**
2. **Memory Configuration:** distinguishes systems by how memory is organized and accessed:
    
    
    - **Shared Memory (Tightly Coupled)** → all processors access a common global memory space. Processors are closely integrated and share resources.  There is no direct processor-to-processor communication at the programming level, instead processors communicate implicitly by reading from and writing to specific locations in the shared memory.
        
        Depending on the memory interconnection structure, shared memory systems are classified into:
        
        - **UMA (Uniform Memory Access):** a shared memory is accessible by all processors through an interconnection in the same way a single processor accesses its memory. Access times are uniform regardless of the origin of the request.
        - **NUMA (Non-Uniform Memory Access):** memory is physically distributed but logically shared. Each processor has part of the shared memory attached, but the memory has a single address space.  The memory access time of processors differs depending on which region of the main memory is accessed.
        
        ![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%207.png)
        
        <aside>
        📌
        
        **NUMA Variant: cc-NUMA**
        
        In the cc-NUMA cache coherence is maintained among the caches of various processors. The main advantage of a cc-NUMA system is that it can deliver effective performance at higher levels of parallelism.
        
        </aside>
        
    
    ![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%208.png)
    
    - **Distributed Memory (Loosely Coupled)** → each processor has private memory. Communication occurs via a network.
        
        <aside>
        📌
        
        **RDMA (Remote Direct Memory Access)**
        
        Normally, copying memory across nodes requires data to pass through the OS, the network protocol stack, and the CPU. RDMA bypasses CPU and OS to fetch data directly from the memory of another separate computer system over the network, vastly reducing latency.
        
        ![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%209.png)
        
        </aside>
        
    - **Hybrid Architecture** → combines both models. Processors are grouped into **nodes**:
        - Cores within a node share local memory (shared memory).
        - Different nodes communicate through a distributed memory network.
        
        **N.B.** This model is common in modern supercomputers.
        
3. **Interconnection Topology:** the performance of a **distributed memory architecture** strongly depends on how processors are connected. Directly connecting every processor to all others with separate cables is impractical, except for systems with a very small number of processors. A common approach is to use specialized **bus or network topologies**, allowing each processor to communicate with any other in the system. Common network topologies include:
    
    
    - **2D and 3D Mesh Networks (a)**
    - **Tree Networks (b)**
    - **Hypercube Networks (c)**
    
    ![image.png](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/image%2010.png)
    
    <aside>
    📌
    
    **Hypercube Addressing and Routing**
    
    An $n$-dimensional hypercube consists of $2^n$ nodes, where each node is assigned a unique **$n$-bit binary address.**
    
    - **Connectivity Rule** → two nodes are directly connected if and only if their binary addresses differ by **exactly one bit**.
    - **The XOR Mask** → routing is managed by generating a **mask** using a bitwise **XOR** operation between the source and destination addresses:
        
        $$
        \text{Source } (000) \oplus \text{Destination } (101) = \text{Mask } (101)
        $$
        
    - **Routing Logic** → the mask indicates which "paths" (dimensions) the message must traverse:
        - **1** at a position: the bit at that position **must be flipped** (move to the next dimension).
        - **0** at a position: the bit remains the same (**no move** required in that dimension).
    
    ![IMG_5008435A7CD9-1.jpeg](H1%20Hardware%20and%20architectural%20foundations%20for%20larg/IMG_5008435A7CD9-1.jpeg)
    
    </aside>
    
4. **Classification based on characteristic nature of PEs (=Processing Element):**
    - **CISC (Complex Instruction Set Architecture):** large instruction set, complex multi-step operations. Optimized for peak performance (common in PCs, servers, HPC). Example: x86.
    - **RISC (Reduced Instruction Set Architecture):** simpler instructions, fast execution, high energy efficiency (mobile and low-power systems). Example: ARM.
    - **Vector Processors and DSPs (Digital Signal Processors):** specialized for mathematical and signal-processing workloads.
    - **Homogeneous vs Heterogeneous Parallel Architectures:**
        - **Homogeneous:** all processing elements are of the same type (common in traditional supercomputers).
        - **Heterogeneous:** Combine different types of processors (e.g., CPU + GPU).

---