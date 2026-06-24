# L2. Leonardo

Pages: 1
Status: Done
Type: lab

# 1. **Introduction to HPC Clusters**

<aside>
💡

An HPC (High-Performance Computing) cluster is a collection of separate servers, called nodes, connected via a fast interconnect network.

</aside>

## 1.1. Cluster **Architecture**

A typical HPC cluster is composed of several specialized components:

- **Headnode/Login Node** → the main entry point where users access the system and is usually the only node connected to the internet.
- **Compute nodes** → the nodes where the majority of computational work is executed.
- **GPU Nodes** → specialized compute nodes capable of running computations on both CPU cores and Graphical Processing Units (GPUs).
- **Data transfer nodes** → dedicated nodes for high-volume data movement, used to handle large transfers without overloading the login node.
- **Storage** → typically separated from the compute nodes to provide scalable and efficient data access.

![image.png](L2%20Leonardo/image.png)

---

# 2. Leonardo HPC Cluster Overview (CINECA)

Leonardo is an Italian supercomputer ranked among the top systems in the Top500, which evaluates clusters based on peak performance (PetaFLOPS). Other notable European systems mentioned include LUMI (Finland), MareNostrum (Spain), and JUPITER (Germany).

## 2.1. Architecture **and Partitions**

The system is divided into two main partitions:

- **Booster partition** → optimized for compute-intensive workloads such as deep learning and represents the largest section, with thousands of nodes. Each node includes 32 CPU cores, 512 GB of RAM, and 4 custom NVIDIA A100 GPUs with 64 GB of VRAM.
- **DCGP (Data-Centric General Purpose) partition** → designed for data-intensive CPU workloads, featuring dual-socket processors (112 cores per node), 512 GB of RAM, and **no GPUs**.

## 2.2. Physical Layout and Cooling

The physical structure involves racks containing multiple nodes: the front row (without a letter designation) houses the **login and storage nodes** followed by **Booster racks** and then **DCGP racks**:

![image.png](L2%20Leonardo/image%201.png)

To maximize space, the compute nodes have a custom design. While standard GPU nodes require 4 Rack Units (RU) of vertical space, Leonardo's Booster nodes fit into a single 1 RU slot:

![image.png](L2%20Leonardo/image%202.png)

For the DCGP partition, three individual nodes are squeezed into a single 1 RU drawer:

![image.png](L2%20Leonardo/image%203.png)

The upper part of each rack contains networking and power infrastructure, while the lower section is dedicated to compute and cooling.

<aside>
📌

**Cooling System**

The cluster uses an advanced **water-cooling system**. A floating floor distributes cold and hot water through pipes that connect directly to CPU and GPU heatsinks. This makes the room significantly quieter than air-cooled systems.

</aside>

## 2.3. **Storage Areas**

Storage is divided into three main areas:

- **`$HOME`:** long-term, user-specific storage that is permanently backed up.
- **`$ARCHIVE`**: for long-term data preservation (slower access, used for infrequent retrieval)
- **`$WORK`:** long-term storage linked to specific projects. Files do not auto-expire as long as the project is active.
- **`$SCRATCH`:** a temporary, high-capacity area with no quotas, ideal for writing intermediate data or checkpoints. Files are automatically removed after a certain period (about 40 days).
- **`$PUBLIC`:** A personal area used to share installations or data, visible to all users on the cluster.

![image.png](L2%20Leonardo/image%204.png)

---

# 3. Workload Management with Slurm

## 3.1. Account Hierarchy

Slurm organizes access via **Usernames** (individual identities) and **Accounts** (projects with allocated budgets). Accounts are conceptually organized in a tree structure, with a root level that can have projects and, in some systems, sub-accounts.

However, on clusters like **Leonardo** and the **Modena cluster**, this hierarchy is flat: there is only a root level with direct project accounts, without any sub-account layer.

![image.png](L2%20Leonardo/b0fe6483-1a70-43c4-af21-29a21cd09158.png)

## 3.2. Budget and Billing

Supercomputers use **Standard Hours** as an internal currency to normalize costs across different hardware. For example, 1 GPU hour may cost 8 Standard Hours on CINECA and 6 on the Modena cluster. Billing is based on the **actual execution time**, not the requested wall time. This means that if a job scheduled for 24 hours stops after 10 minutes, only those 10 minutes are charged. However, **allocated resources are always billed in full**: if 4 GPUs are requested but only 1 is used, all 4 are still charged because they were reserved.

To encourage consistent usage, the priority algorithm may divide the total budget by the project's duration in months, evaluating consumption against a monthly average instead of the total allocation.

<aside>
📌

**Enforcement**

Slurm typically checks budgets before execution, preventing users from exceeding their quota and keeping usage just below 100%. Instead of checking before the job starts, CINECA uses external scripts that check usage every **24 hours**. As a result, if the budget is exhausted during execution, jobs may continue running for some time before being stopped, allowing usage to slightly exceed 100%.

</aside>

---

# 4. Lab & Demo

1. **Connecting to the Cluster** → users access Leonardo via SSH (`ssh user@login.leonardo.cineca.it`). After login, the system prints hardware information (Booster/DCGP nodes), maintenance notices, and storage warnings. It also highlights that the `$PUBLIC` directory is visible to all users and shows the status of `$SCRATCH` cleaning.
    
    ![image.png](L2%20Leonardo/34916975-71a9-4cb3-acf9-d4c8f761ecbb.png)
    
2. **Cluster Status** → the command `sinfo -s` shows an overview of the cluster, including partitions (GPU Booster and CPU DCGP) and node states such as Active (`A`), Idle (`I`), Off (`O`), and Total (`T`). Because the system is large, the output includes thousands of nodes per partition.
    
    ![Screenshot 2026-05-04 at 13.44.56.png](L2%20Leonardo/048a5e76-427b-4c82-a2f3-2eb20a499faa.png)
    
3. **Software Modules and Profiles** →  software is managed through modules that must be loaded in each session. Users can list available modules with `module av`, load specific tools (e.g., CUDA) using `module load <module>`, or activate full environments via profiles (e.g., deep learning profiles with datasets like ImageNet). The system also supports Spack for custom environments.
    
    ![image.png](L2%20Leonardo/image%205.png)
    
4. **Interactive Sessions** → compute nodes can be accessed using:
    - **`srun`:** executes commands directly on a node.
    - **`salloc`**: allocates resources while keeping the session on the login node; commands must then be run with `srun`
    
    **N.B.** Login nodes do not have GPUs. If a user runs a GPU-specific command (like `nvidia-smi`) on the login node, it will throw an error; 
    
    ![image.png](L2%20Leonardo/image%206.png)
    
5. **Job Submission and Management** → jobs are submitted with `sbatch script.sh`. The script includes `#SBATCH` directives specifying account, wall time, partition, number of nodes, and GPUs.

<aside>
📌

**Distributed Machine Learning Experiment**

A standard single-GPU training pipeline includes a model, a dataloader, a loss function, and a training loop. In a distributed setting, this setup is extended with additional components to coordinate multiple GPUs and nodes:

- **Environment Variables:** you must define variables for the distributed environment, including `local_rank`, `global_rank`, and `world_size`.
- **Synchronization:** to handle synchronization between processes we can use protocols like `nccl` (for NVIDIA GPUs) or `gloo` (for CPUs).
- **Model and Data Distribution:** the model is wrapped in `DistributedDataParallel`, and the dataset is partitioned across GPUs using distributed samplers.
- **I/O Handling:** to avoid concurrent prints and simultaneous file writes that could corrupt data, only "`Rank 0`" (the master process) should be allowed to print outputs and save model checkpoints.

**N.B.** On Leonardo’s Booster nodes, each node has 32 CPU cores and 4 GPUs, meaning there are **8 CPU cores per GPU** ($32 /4 = 8$). For this reason, the dataloader should use **8 workers per GPU** to maximize data loading efficiency.

</aside>

---