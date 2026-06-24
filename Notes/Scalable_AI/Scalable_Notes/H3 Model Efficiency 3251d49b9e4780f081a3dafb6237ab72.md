# H3. Model Efficiency

Pages: 6
Status: Done
Type: theory

# 1. Floating Point Arithmetic

Floating-point numbers are representations of real numbers that allow computers to handle very large or very small values efficiently. Unlike **fixed-point representation**, where the position of the decimal point is fixed, floating-point numbers use a **dynamic scaling mechanism** through an exponent. The value of a floating-point number can be expressed mathematically as:

$$
\text{Value} = (-1)^S\cdot \text{M}\cdot b^{\text{E}}
$$

Where we have:

- **Sign ($S$)** → one bit indicating if the number is positive or negative
- **Exponent ($E$)** → determines the range or scale of the number
- **Mantissa/Significand ($M$)** → determines the precision
- **Base ($b$)** → the numeric base of the system, usually **2** in binary computation

The IEEE 754 standard defines formats like single precision (FP32) and double precision (FP64). Single precision utilizes 32 bits: 1 sign bit, 8 exponent bits, and 23 mantissa bits. 

<aside>
📌

**How it works?**

To represent numbers efficiently, floating-point formats apply two important techniques:

- To avoid negative exponents, the GPU stores the exponent as an **unsigned integer** by adding a **bias** (e.g., 127 for FP32, 7 for FP8).
- To form the **mantissa**, the GPU drops the first $\textcolor{red}{1}$ (implicit bit) to save space and pads with zeros to fill the field width or truncates extra bits if the format is small.

*Example (FP32)* $\left[E=8\ \text{bit},\ M= 23 \text{bit}\right]$:

$$
25 \rightarrow 11001_2 \xrightarrow{\text{normalize}} \textcolor{red}{1}.\textcolor{green}{1001} \cdot 2^{\textcolor{green}{4}} \rightarrow (+1)\cdot (\textcolor{green}{1001} \ 0000\ 0000\ 0000\ 0000\ 0)^{4 + 127} \rightarrow 

  \underbrace{
0}_{S}\  |\ \underbrace{10000011}_{E}\ |\ \underbrace{10010000000000000000000}_{M}
$$

</aside>

<aside>
🚨

**Limitations**

- **Precision Loss** → moving from a higher to a lower precision format (or exceeding the bit limit) causes truncation and information loss.
- **Overflow** → occurs when a number is too large for the representable range, causing different large numbers to map to the same maximum value (or infinity).
- **Underflow** → occurs when a number is too close to zero to be accurately represented, often evaluating to zero.
- **Catastrophic Cancellation** → extreme precision loss when subtracting two nearly equal large numbers.
</aside>

---

# 2. Model Efficiency Techniques

When optimizing machine learning models for production, the goal is to achieve **Pareto optimality**, meaning a model that provides the best trade-off between **high accuracy** and **low inference latency**.

![image.png](H3%20Model%20Efficiency/34ba4889-8e9b-4b77-8684-b72e184a2e8f.png)

A model may perform well during **training** but still be inefficient during **inference**, therefore efficiency must be evaluated across **both phases**:

- **Pruning** → involves in removing unnecessary or redundant parameters from a neural network that do not contribute significantly to the output.
    
    <aside>
    📌
    
    **Pruning Classification**
    
    Pruning can be applied at different levels:
    
    - **Neuron Pruning** → removes entire neurons + all their connections.
    - **Synapse Pruning →** removes individual weights between neurons (all the neurons remain there, but some of them are connected by fewer links).
    
    ![image.png](H3%20Model%20Efficiency/image.png)
    
    It can also be classified based on how parameters are removed:
    
    - **Unstructured Pruning** → removes individual weights across the network, producing sparse and irregular matrices. It usually preserves model accuracy better but is less efficient for hardware acceleration.
    - **Structured Pruning** → removes entire filters, channels, or blocks. This produces more regular matrices and is more efficient for **hardware execution and caching.**
    
    ![image.png](H3%20Model%20Efficiency/image%201.png)
    
    </aside>
    
    <aside>
    📌
    
    **Additional Pruning Strategies**
    
    Pruning often involves additional hyperparameters:
    
    - **Saliency heuristics** to measure parameter importance.
    - **Layer-wise pruning budgets** that define the specific percentage of weights to be removed from each individual layer of the model.
    - **Scheduling**, which decides when pruning occurs during training.
    - **Regrowth mechanisms**, which allow previously removed connections to be restored if necessary.
    </aside>
    
- **Quantization** → reducing the **numerical precision** of parameters (e.g., from FP32 to 8-bit integers) in order to **reduce memory usage and accelerate inference**. The goal is to compress the model while preserving the **original data distribution** as much as possible. It can be:
    
    
    - **Symmetric Quantization:** the range is centered around zero. A symmetric interval $[-R,R]$ is mapped to an integer range (e.g., $[−127,127]$).
        
        $$
        \text{Quantized Value}=round\left(\frac{FP32\_{Value}}{\text{Scale factor}}\right)\quad \text{where} \quad \text{Scale factor}=\frac{R}{\text{Max Integer Value}}
        $$
        
        **N.B.** It requires storing the **scaling factor** to perform dequantization later.
        
    - **Asymmetric Quantization:** the range is not centered around zero. The observed minimum ($min$) and maximum ($max$) are mapped to the integer range 
    $[0,Q_{max}]$ (e.g., $[0,255]$):
        
        $$
        \text{Quantized Value}=round\left(\frac{FP32\_{Value} - min}{\text{Scale factor}}\right)
        \quad \text{where} \quad \text{Scale factor}=\frac{max -min}{Q_{max}}
        $$
        
        **N.B.** It requires storing both the **scaling factor** and the **min**‬ value to handle the offset.
        
    
    ![image.png](H3%20Model%20Efficiency/image%202.png)
    
    <aside>
    🔢
    
    **Example:**
    
    Suppose we have a **fully connected layer** in a neural network whose weights are represented as **32-bit floating-point numbers (FP32)**. The weights range from $[−0.5,0.7]$, and we want to **quantize them to 8-bit integers (INT8).**
    
    **Symmetric** 
    
    - **Quantization:**
        1. Determine the **representable range of the target format**. For signed **INT8** integers: 
        
        $[-128,127]$.
        2. Compute the **absolute maximum value** of the floating-point range: $\text{AbsMax}=\max(∣−0.5∣,∣0.7∣)=0.7$
        3. Compute the **scale factor**:
            
            $$
            \text{scale\_factor} = \frac{\text{AbsMax}}{127}=\frac{0.7}{127}\approx 0.00551
            $$
            
        4. Quantize the value $x=0.3$:
            
            $$
            q = \frac{x}{\text{scale\_factor}}\approx 54.44
            $$
            
        5. Round to the nearest integer: $\text{round}(54.44)= 54$
    - **Dequantization:**
        
        $$
        x_{\text{dequant}}=q\cdot \text{scale} = 54 \cdot 0.00551 \approx 0.2975 \approx 0.3
        $$
        
    
    **Asymmetric** 
    
    - **Quantization:**
        1. Determine the **representable range of the target format**. For unsigned **INT8** integers: $[0,255]$.
        2. Compute the **scale factor**:
            
            $$
            
            \text{scale factor} = \frac{\text{max value} - \text{min value}}{\text{quantization range}}=\frac{0.7-(-0.5)}{255}\approx 0.00471
            $$
            
        3. Subtract the minimum value $(−0.5)$ so that the range becomes $[0, 1.2]$**:  $0.3-(-0.5) = 0.8$**
        4.  ****Map to the integer range:
            
            $$
            q = \frac{0.8}{0.00471}\approx169.85
            $$
            
        5. Round to the nearest integer: **
        
        $\text{round}(169.85)= 170$**
    - **Dequantization:**
        
        $$
        x_{\text{dequant}} = q \cdot \text{scale} + \text{min}= 170 \cdot 0.00471 + (-0.5)=0.3007\approx 0.3
        $$
        
    </aside>
    
    <aside>
    📌
    
    **Fake Quantization**: exposing the model to both real and quantised values during training so it learns to handle quantization loss before inference.
    
    </aside>
    

- **Knowledge Distillation** → a training technique where a smaller and more efficient **student model** learns to mimic the behavior of a larger **teacher model**. The **teacher** is a large, pre-trained model that produces **soft labels** (=probability distributions over classes that contain richer information than standard labels). The **student** model is trained using both:
    - the **soft labels** produced by the teacher
    - the **true labels (hard labels)** from the dataset
    
    This allows the student to capture the knowledge of the teacher while remaining **smaller, faster, and more efficient for deployment**.
    

![image.png](H3%20Model%20Efficiency/2e9c6856-496f-4042-be43-32a4809bfb0b.png)

- **Low-Rank Factorization (e.g., LoRA)** → a technique used to **efficiently fine-tune large neural networks** by updating only a small number of additional parameters instead of modifying the full weight matrices. The main idea is that the **updates required during fine-tuning often lie in a low-rank subspace**. Therefore, instead of updating the full weight matrix $W$, LoRA keeps the original weights **frozen** and learns a small low-rank update:
    
    
    $$
    W_\text{final} = W_\text{frozen} + Y  Z^T
    $$
    
    Here, $W_{\text{frozen}}$ represents the original pre-trained weights, which remain fixed. The update $\Delta W = Y Z^T$ is computed using two smaller matrices:
    
    - $Y \in \mathbb{R}^{m \times k}$
    - $Z^T \in \mathbb{R}^{k \times n}$
    
    Where $k$ is the **rank hyperparameter**, typically very small (e.g., 8, 16, or 32). Since $k \ll m,n$, the number of trainable parameters becomes extremely small compared to the full matrix. 
    
    ![image.png](H3%20Model%20Efficiency/image%203.png)
    
    <aside>
    📌
    
    **SVD (Singular Value Decomposition)**
    
    It’s a mathematical technique used to **decompose a matrix** $A$ into the product of three matrices:
    
    $$
    
    A = U S V^{T}
    $$
    
    where:
    
    - $U$ and $V^{T}$ are **orthogonal matrices** representing the principal directions (basis vectors) of the data.
    - $S$ is a **diagonal matrix** containing the **singular values**, which indicate the importance of each component.
    
    ![image.png](H3%20Model%20Efficiency/image%204.png)
    
    SVD is often used for **dimensionality reduction and matrix compression**. By keeping only the **largest $k$ singular values** in $S$ and discarding the smaller ones, we obtain the **best low-rank approximation** of the original matrix while preserving most of the important information.
    
    However, computing SVD for models with **billions of parameters** is computationally very expensive. For this reason, techniques such as **LoRA** approximate low-rank updates by **learning small matrices directly during training**, instead of explicitly computing the SVD.
    
    </aside>
    
- **Data Augmentation** → a technique used in machine learning to **artificially increase the size of training set** by generating modified versions of existing data. The goal is to **improve model generalization and robustness** while reducing the need for collecting and labeling large amounts of new data. By exposing the model to different variations of the same samples, data augmentation helps **reduce overfitting** and allows the model to better handle real-world variability.
    
    ![image.png](H3%20Model%20Efficiency/image%205.png)
    

- **Sensitivity-Based Layer Dropping (SBLD)** → a technique used to **accelerate the training of large transformer models** by selectively skipping certain layers during training. The method starts with a **pre-analysis phase**, where the model is trained for a few iterations to measure the **sensitivity of each layer** with respect to changes in the training loss. This analysis identifies which layers are more critical for model performance. Based on this information, each layer is assigned a **drop probability**, meaning that less sensitive layers can be skipped during some training steps, while the most important layers remain active.
    
    To control this process, SBLD uses several strategies:
    
    - **Layer-wise sensitivity analysis** to determine how much each layer can be dropped.
    - **Block/Allow lists** to ensure that highly sensitive layers are never skipped.
    - A **variance-based scheduler** that dynamically adjusts the probability of dropping layers during training.

![image.png](H3%20Model%20Efficiency/image%206.png)

---

# 3. Evaluating Training Curves

Training and validation curves are essential tools for diagnosing how well a model is learning and generalizing:

1. **Ideal Fitting** → both the training and validation curves decrease smoothly together and stabilize (plateau) at a low, healthy margin above zero (around $0.4$), the model is learning efficiently and generalizing well.
    
    ![image.png](H3%20Model%20Efficiency/image%207.png)
    
2. **Underfitting** → occurs when the model fails to capture the underlying patterns in the data. Training loss remains slightly higher than validation loss, while both curves are still decreasing and far from convergence. The model is still learning but has not fully converged.
    
    ![Screenshot 2026-03-18 at 21.35.47.png](H3%20Model%20Efficiency/Screenshot_2026-03-18_at_21.35.47.png)
    
    **Solution:** increase the number of training epochs. This is the only scenario where hyperparameter tuning can help.
    
3. **Overfitting** → occurs when the model memorizes the training data but fails to generalize. The **training loss continues to decrease**, while the **validation loss starts increasing or diverging**.
    
    ![image.png](H3%20Model%20Efficiency/image%208.png)
    
    **Solution:** apply early stopping at the epoch where validation loss starts increasing, use regularization techniques like dropout, or review the training dataset.
    

<aside>
🚨

**Dataset-Related Issues**

Some curve patterns indicate problems with the data rather than the model:

- **Shaking Validation Curve:** the training loss decreases normally, but the validation loss oscillates wildly. This indicates that the validation set does not represent the training data. The datasets are heavily imbalanced or mismatched (e.g., the training set is full of dogs, but the validation set is full of cats).
    
    ![image.png](H3%20Model%20Efficiency/3cab22b3-3a52-42c2-b223-484b421e0771.png)
    
    **Solution:** rebalance the datasets or apply data augmentation to create a more representative validation set.
    
- **Validation Dataset Too Easy:** the validation loss starts significantly lower than the training loss and remains almost completely flat throughout the entire training process. This is not standard underfitting (otherwise it would dive to zero rapidly). For instance, training a model on complex calculus problems but validating it only on basic multiplication tables,  it will appear to perform perfectly on validation while failing to generalize.
    
    ![image.png](H3%20Model%20Efficiency/image%209.png)
    
</aside>

**N.B.** The way you manipulate and curate your data accounts for 80% of your success. If your curves look fundamentally wrong from the start, hyperparameter tuning will not save you. **Good data is required for a good model**.

---

# 4. LLM Development Stages

The lifecycle of a Large Language Model can be divided into distinct functional phases:

- **Pre-training (unsupervised)** → the model learns general language patterns by predicting the next word, without any specific instructions. A variant, *continuous pre-training*, is used to specialize a model in a specific domain (e.g., medical or legal) by continuing the original learning process on specialized data without changing the training objective.
- **Fine-tuning (supervised)** → the model is trained with explicit instructions (e.g., summarization, question answering) to perform specific tasks.
- **Inference and Reasoning** → during the inference phase, advanced models can apply reasoning by generating intermediate steps, process known as Chain-of-Thought. This allows the model to decompose complex problems into manageable sub-tasks, and refining its answers. The more compute is allocated to “reasoning,” the more accurate the result can be; however, this introduces the risk of overthinking.
    
    <aside>
    🚨
    
    **Overthinking**
    
    Overthinking occurs when increasing the computational time and the length of the Chain-of-Thought stops yielding benefits and instead begins to degrade the quality of the response.
    
    </aside>
    

---

<aside>
💡

**Computational budget**

The computational budget is the total amount of compute used over time (e.g., GPU-hours). Using more machines makes training faster, but the total compute cost stays roughly the same.

</aside>

---

# 5. **Computational Constraints**

## 5.1. GPU Memory occupation

During training, the total GPU memory usage is determined by four main components:

1. **Model Weights** → depend on the chosen precision (e.g., 4 bytes/parameter for FP32; 6 bytes/parameter for mixed precision, which keeps a master copy in FP32 and an active copy in FP16).
2. **Gradients** → typically kept in higher precision (e.g., FP32) because they represent the "direction" and "magnitude" of the update needed for the model's weights. If this information is not precise, the entire training process can fail.
3. **Forward Activations** → required for backpropagation; size scales with sequence length, hidden dimension, and batch size.
4. **Optimizer States** → depend on the optimizer. Optimizers update the model’s weights to reduce the **loss**. Backpropagation computes the gradients (the direction), while the optimizer decides **how big the step is and how to apply it**.
    - **SDG (Stochastic Gradient Descent):** the most basic optimizer. It updates weights based strictly on the current gradient and a regularization term:
        - Gradient Calculation → $‭g_t = \frac{\partial L}{\partial \theta_t} + \lambda \theta_t$  (Weight Decay $\lambda \theta_t$: penalizes large weights to prevent overfitting).
        - Update Rule → $\theta_{t+1} = \theta_t - \eta g_t‬$
        
        **Memory:** 0 bytes per weight (No internal states are stored).
        
    - **SGD with momentum:** adds "inertia" to the update to accelerate through plateaus:
        - Gradient Calculation → $‭g_t = \frac{\partial L}{\partial \theta_t} + \lambda \theta_t$
        - Update Rule → $\theta_{t+1} = \theta_t - m_t$
        - Momentum State (‭$m_t‬$) → ****a moving average of previous updates using a friction factor $\gamma$:  
          $m_t = \gamma m_{t-1} + \eta g_t‬‭$
        
        **Memory:** 4 bytes per weight (stores 1 state: Momentum in FP32).
        
    - **Adam (Adaptive Moment Estimation):** adapts the learning rate for every individual parameter by tracking two different statistics:
        - Gradient Calculation → $‭g_t = \frac{\partial L}{\partial \theta_t} + \lambda \theta_t$
        - Update Rule → $\theta_{t+1} = \theta_t -   \frac{\eta}{\sqrt{v_t} + \epsilon}\cdot  m_t$
        - Momentum State (‭$m_t‬$) → tracks the direction/speed of the gradient:  $m_t = \beta_1 m_{t-1} + (1 - \beta_1) g_t$
        - Variance ($v_t$) →  tracks the "volatility" or scale of the gradient ($g_t^2$):  $v_t=\beta_2v_{t-1}+(1-\beta_2)\cdot g_t^2$
        
        **Memory:** 8 bytes per weight (stores 2 states: Momentum + Variance in FP32).
        
    - **AdamW (Weight Decay Decoupled):** standard Adam applies weight decay by adding $\lambda \theta_t$ directly to the gradient $g_t$ **before** computing the adaptive moments. Because Adam divides the update by the root of the variance $\sqrt{v_t}$, the weight decay also gets divided by it:
        - If a parameter has large gradients (high variance), the weight decay is **suppressed**.
        - If a parameter has small gradients, the weight decay becomes **disproportionately strong**.
        
        AdamW fixes this by removing the weight decay from the gradient calculation and applying it **directly to the weight update** at the very end:
        
        - Gradient Calculation → $‭g_t = \frac{\partial L}{\partial \theta_t}$
        - Update Rule → $\theta_{t+1} = \theta_t - \eta \left( \frac{1}{\sqrt{v_t} + \epsilon} m_t + \textcolor{red}{\lambda} \theta_t \right)$
        
        **Memory:** 8 bytes per weight (stores 2 states: Momentum + Variance in FP32).
        
    - **8-bit AdamW:** same as AdamW but the 2 states are stored compressed form FP32 to int8. During updates are decompressed and used as FP32.
        
        **Memory:** 2 bytes per weight.
        

## 5.2. Arithmetic Intensity

**Arithmetic intensity** is defined as the ratio of **computations performed** to the **amount of data moved:** 

$$
\frac{FLOPs}{Byte}
$$

 So it defines the threshold at which an operation shifts from being memory-bound to compute-bound:

- **Compute Bound (High Intensity)** → the bottleneck is the processor's calculation speed (=more calculations per byte).
- **Memory-bound (Low Intensity)** → the bottleneck is the memory bandwidth (=more data movement per calculation). In deep learning, large matrix operations move data across multiple levels (storage → main memory → CPU → GPU and back), and this transfer can become the main bottleneck even with optimizations. GPUs are often memory-bound because they compute much faster than data movement.

<aside>
🔑

*Example:*

An NVIDIA **A100** has $\sim 312\ \text{TFLOPs}$ of compute and $\sim 2039\ \text{GB/s}$ of memory bandwidth. The threshold is $\frac{312.000\ \text{FLOPs}}{ 2039\ \text{GB/s}}=143\ \text{flops/B}$. Operations below this value are memory-bound, while those above are compute-bound. 

</aside>

## 5.3. Transformer Operations

- **Tensor contractions (compute bound)** → linear layers and multi-head attention mainly perform batched matrix–matrix multiplications.
- **Statistical normalization (memory bound)** → operations like Softmax and LayerNorm involve reductions and are less compute-intensive than matrix multiplications.
- **Element-wise operations (memory bound)** → include biases, activations, dropout, and residual connections, applied independently to each element.

**N.B.** Transformers are highly parallelizable architectures. Most computation (~99% of FLOPs) is concentrated in tensor contractions that are compute-bound.

![image.png](H3%20Model%20Efficiency/image%2010.png)

## 5.4. Hardware **Alignment**

The performance of deep learning models is heavily dependent on how well tensor dimensions align with the underlying GPU architecture.

<aside>
🔑

**Tiling Mechanism**

GPUs process large tensors by splitting them into smaller blocks called **tiles**. Maximum efficiency is achieved when tensor dimensions are perfectly divisible by the hardware’s execution units. Misaligned sizes (e.g., $17\times17$ instead of $16×16$) force the system to use padding or perform extra iterations, causing a significant performance penalty known as the **Tile Quantization effect**.

![image.png](H3%20Model%20Efficiency/c3365f5f-f38e-475c-83c8-dd04b3670b1b.png)

---

*Example:*

On the NVIDIA A100, we can notice that performance suffers when the input dimensions deviate from optimal multiples, such as 128:

![image.png](H3%20Model%20Efficiency/a107e08b-9191-4076-84f2-932febbe4995.png)

</aside>

### 5.4.1. Scaling Guidelines

- **Batch size**, **Channels**, and **Features** must follow specific divisibility rules based on the numerical precision used:
    
    
    | **Precision Format** | **Standard Divisibility** | **Optimized for NVIDIA A100** |
    | --- | --- | --- |
    | **TF32** | Multiples of 4 | Multiples of 32 |
    | **FP16** | Multiples of 8 | Multiples of 64 |
    | **INT8** | Multiples of 16 | Multiples of 128 |
- **For Convolutional Layers:**
    - Input and output channels should be aligned with the hardware requirements. For example, RGB images have 3 channels, but padding them to 8 channels can enable faster Tensor Core kernels.
    - On NVIDIA GPUs, **Channels Last (NHWC)** is often faster than **Channels First (NCHW)** because it better matches Tensor Core memory access patterns.
    - Enable **auto-tuning** so the framework can automatically choose the most efficient convolution algorithm.
- **For Fully Connected Layers:**
    - When model parameters are small, the data movement overhead increases. For this reason, proper dimension alignment is critical for efficient GPU tiling. Dimensions should be divisible by **64**, and ideally by **256**, to maximize hardware utilization.
    - If the workload is too small, the GPU becomes memory-bound, spending more time on data transfer than computation. As a rough guideline for A100 and V100 GPUs, keep **batch sizes** and **neuron counts** greater than **128** to maintain compute-bound performance.

### 5.4.2. Accurate Profiling in PyTorch

Because PyTorch executes GPU operations asynchronously, standard timers will return inaccurate results. You must explicitly invoke synchronization events (e.g., `torch.cuda.synchronize()`) before and after the operation to measure execution time correctly:

```python
start = torch.cuda.Event(enable_timing=True)
end = torch.cuda.Event(enable_timing=True)

start.record()
# code to be measured
…
end.record()

torch.cuda.synchronize()

elapsed_time_in_ms = start.elapsed_time(end)
```

---

# 6. Optimization Techniques

To balance the mismatch between available memory and computational power, several key techniques are used:

| **Method** | **Improves Speed?** | **Improves Memory?** |
| --- | --- | --- |
| **Gradient Accumulation** | No | Yes |
| **Gradient Checkpointing** | No | Yes |
| **Mixed Precision training** | Yes | (No) |
| **Batch Size** | Yes | Yes |
| **Optimizer Choice** | Yes | Yes |
| **DataLoader** | Yes | No |
| **DeepSpeed ZeRO (Sharding)** | No | Yes |

## 6.1. Gradient Accumulation

It enables training with large effective batch sizes when memory is limited. Data is split into smaller mini-batches processed sequentially, and gradients are accumulated across steps before performing a single optimizer update (`optimizer.step()`). 

This reduces memory usage because fewer activations need to be stored at each step, while preserving the effect of a larger batch size. However, it does **not** reduce the total amount of computation and may slightly increase training time.

![image.png](H3%20Model%20Efficiency/6985a09f-0cbd-4286-b102-0130118fc7fe.png)

## 6.2. Gradient Checkpointing

In Transformers, the attention matrix grows **quadratically ($L^2$)** with sequence length, making it highly memory-intensive. Gradient Checkpointing reduces memory usage by storing only a subset of activations (checkpoints) during the forward pass and recomputing the missing ones during backpropagation. This significantly lowers VRAM consumption at the cost of increased computation time and training overhead.

![image.png](H3%20Model%20Efficiency/image%2011.png)

<aside>
📌

**Selective Activation Recompute with Megatron-LM**

Unlike standard checkpointing, which saves activations every $N$ layers, selective recompute identifies which activations are better to store or recompute based on memory and compute cost. Instead of treating all layers equally, it distinguishes between:

- **Save (Checkpoint):** activations that use **little memory** but are **expensive to recompute** (e.g., large matrix multiplications in linear layers).
- **Recompute:** activations that use **a lot of memory** but are **cheap to recompute** (e.g., Dropout, Softmax, LayerNorm).

The histogram below shows the impact of selective recomputation: the green portion (activations) is drastically reduced, keeping total memory below the red line (NVIDIA A100 80GB limit), while the blue portion (parameters and optimizer states) remains constant.

![image.png](H3%20Model%20Efficiency/fd585f02-4073-4f33-92fc-065257cb6258.png)

</aside>

## 6.3. Automatic Mixed Precision (AMP)

It accelerates training by performing most operations in FP16 for high-speed computation, while maintaining a master copy of weights in FP32 and using dynamic loss scaling to preserve numerical stability and accuracy.

<aside>
📌

**Data Formats: Precision vs Dynamic Range**

In deep learning, the ability to represent very large or very small values (**dynamic range**) is often more important than high numerical precision:

- **FP32 (Full Precision)** → 8-bit exponent (wide range), 23-bit mantissa (high precision)
- **FP16 (Half Precision)** → 5-bit exponent, 10-bit mantissa; limited range can cause *underflow* (values collapsing to zero).
- **TF32 (Tensor Float 32)** → an NVIDIA-specific format that combines the best of both:
    - **8-bit exponent (like FP32)** → preserves wide dynamic range and avoids loss scaling.
    - **10-bit mantissa (like FP16)** → enables fast computation on Tensor Cores.
    
    **N.B.** Internally, values are stored as FP32 but truncated to lower precision during computation, requiring no code changes.
    

![image.png](H3%20Model%20Efficiency/bafcd933-d53c-4295-b76f-f12fa3b3f6ec.png)

</aside>

### 6.3.1. Core Algorithm

1. Maintain a primary "Master Copy" of all weights in **FP32**.
2. Initialize scaling factor $S$ to a large value.
3. For each iteration:
    1. Create a temporary FP16 copy of the weights
    2. Execute Forward propagation using FP16 to leverage the speed of Tensor Cores.
    3. Multiply the loss by a scaling factor $S$, this "pushes" small gradients into a range that FP16 can represent, preventing them from becoming zero.
    4. Execute Backward propagation using FP16 to leverage the speed of Tensor Cores.
    5. If there is an $\text{Inf}$ or $\text{NaN}$ in weight gradients:
        1. Reduce $S$
        2. Skip the weight update and move to the next iteration
    6. Convert gradients back to FP32. Multiply by $\frac{1}{S}$‬ ****to restore the original scale.
    7. Update the **FP32 Master Weights**.
    8. (Optional) If no overflows occur for $N$‬ steps, increases $S$ in order to store gradients in FP16 more precisely. 

![image.png](H3%20Model%20Efficiency/image%2012.png)

### 6.3.2. How to use it in PyTorch?

To implement Automatic Mixed Precision, PyTorch provides two main components:

- `torch.autocast` (to handle the precision switch)
- `torch.cuda.amp.GradScaler` (to handle loss scaling)

```python
# 1. Initialize the Gradient Scaler
scaler = torch.cuda.amp.GradScaler()

for epoch in epochs:
    for input, target in data:
        optimizer.zero_grad()

        # 2. Forward pass with autocasting (FP16/FP32 mix)
        with torch.autocast(device_type='cuda'):
            output = model(input)
            loss = loss_fn(output, target)

        # 3. Scale the loss and call backward()
        # Gradients are computed in FP16 but "pushed" to avoid underflow
        scaler.scale(loss).backward()

        # 4. Step the scaler
        # This unscales gradients and calls optimizer.step() if they are valid
        scaler.step(optimizer)

        # 5. Update the scale factor for the next iteration
        scaler.update()
```

## 6.4. Batch Size Trade-off (p**erformance vs optimization**)

Large and small batch sizes present different trade-offs:

| **Feature** | **Small Batch** | **Large Batch** |
| --- | --- | --- |
| Efficiency | Lower → GPU is not fully utilized due to limited parallelism | Higher → better parallelism, allowing to faster hardware utilization |
| Generalization | Better → gradient noise acts as implicit regularization, helping to prevent overfitting | Worse → reduced noise can lead to sharper solutions and worse generalization |
| Memory | Lower VRAM usage → fewer activations stored per step | Higher VRAM usage → require more memory to store the intermediate values for gradient computation during backpropagation |
| Gradients | Noisy → can make training less stable but helps exploration | Stable → more accurate gradient estimates and smoother convergence |

### 6.4.1. Learning-rate/batch size

Increasing the batch size reduces the number of updates per epoch (  $\text{Updates per Epoch} = \frac{\text{Total Dataset Size (N)}}{\text{Batch Size (B)}}‬$). To compensate and maintain convergence speed, the Learning Rate (LR) must be adjusted:

- **Linear Scaling Rule (Most Common)** → if you multiply the batch size by $k$ ‬, multiply the learning rate by $k$.
- **Square Root Scaling Rule** → some researchers suggest scaling by $\sqrt{k}$‬ to keep the variance in the gradient expectation constant.

## 6.5. Optimizer Efficiency

Standard optimizers like Adam are memory-intensive because they store two FP32 states (momentum and variance) per parameter. For a 7B model, this can require ~28GB of VRAM. **8-bit Adam** reduces memory usage by ~75% through quantization tricks:

- **Dynamic quantization** → preserves accuracy for both large and small values.
- **Block-wise scaling (absmax)** → states are divided into small blocks, each with its own scaling factor (typically based on *absmax*). This prevents outliers in one region from reducing the precision of the entire tensor.
- **Stable embeddings** → embedding layers are often kept in higher precision because they tend to have high gradient variance, making them more sensitive to quantization errors.

### 6.5.1. How 8-bit Adam works?

1. **Quantization (Storage):**
    1. Start from FP32 optimizer states
    2. Split them into small blocks and normalize each block using its **absmax**
    3. Map each normalized value to the closest **8-bit index (0–255)** using ****symmetric quantization and a non-linear lookup table (LUT).
    4. Store only the 8-bit index values (+ **absmax values** for each block, typically stored in FP32) → large memory savings.
2. **Dequantization (Calculation):**
    1. When it's time to update the weights, retrieve the stored indices.
    2. Convert them back using the LUT.
    3. Rescale them using the corresponding **absmax.**
    4. Perform the weight update in higher precision.

![image.png](H3%20Model%20Efficiency/image%2013.png)

### 6.5.2. **Implementation (bitsandbytes)**

In PyTorch, the `bitsandbytes` (`bnb`) library is the standard for this. It is a "drop-in" replacement, meaning you only need to change a few lines of code.

```python
import bitsandbytes as bnb

# --- Standard Adam (High VRAM) ---
# optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

# --- 8-bit Adam (Low VRAM) ---
optimizer = bnb.optim.Adam8bit(model.parameters(), lr=1e-3)

# For NLP models, it is recommended to use Stable Embeddings, to reduce gradient variance and keep training stable.
model.embed = bnb.nn.StableEmbedding(vocab_size, embed_dim)
```

## 6.6. Efficient Data Loader

The goal is to ensure the GPU is never idle. Data loading and augmentation should happen in parallel with GPU computation. The main strategies are:

- **In-memory storage** → if possible, load the dataset directly into GPU memory to avoid transfer overhead.
- **Pinned Memory (Page-locked)** → use `pin_memory=True` in the `DataLoader`. Data transfer from Host (CPU) to Device (GPU) is much faster when memory is "pinned," as it avoids intermediate staging areas in the OS.
    
    ![image.png](H3%20Model%20Efficiency/image%2014.png)
    
- **Multiple workers** → use `num_workers > 0` to load data in parallel, so the next batches are ready while the GPU processes the current one.

### 6.6.1. Pre-fetching & Overlapping

By setting a `prefetch_factor` (the number of batches prepared in advance per worker), the DataLoader loads data into a queue before it is needed. Without prefetching, the GPU experiences idle gaps while waiting for data. With prefetching and multiple workers, data loading overlaps with computation, enabling continuous execution.

![image.png](H3%20Model%20Efficiency/image%2015.png)

---

# 7. **Parallelism Strategies for Scaling Deep Learning Models**

To train very large models, weights and data are split across devices using different techniques. The main scaling methods are **pipeline parallelism** and **tensor parallelism**, often combined in hybrid setups.

## **7.1. Pipeline Parallelism (Inter-Layer)**

This approach splits consecutive layers of the model across different GPUs (e.g., layers 0–2 on GPU 1, layers 3–5 on GPU 2). The goal is to improve GPU utilization within a single node by maximizing parallel execution of pipeline stages.

![image.png](H3%20Model%20Efficiency/image%2016.png)

<aside>
🚨

**Bubbles problem:**

Pipeline parallelism introduces *bubbles*, where some GPUs remain idle while waiting for others to finish their computation:

![image.png](H3%20Model%20Efficiency/c08311d7-15df-4275-ada8-8a642cd52f09.png)

</aside>

<aside>
✅

**Solutions:**

The bubbles problem can be attenuated with:

- **Micro-batches** → the input batch is split into smaller **micro-batches** that are processed in an overlapping way across GPUs:
    
    ![image.png](H3%20Model%20Efficiency/dbf8f70a-2b6e-4cc9-8588-0e1474c6d831.png)
    
    Here, the **number** (e.g., $1$, $2$) represents the global batch (training iteration), while the **letter** ($a$, $b$, $c$, …) indicates the micro-batch (a subdivision of the batch). Without micro-batches, GPUs would remain idle for long periods. With micro-batches, execution becomes pipelined: each GPU processes a portion of the layers on one micro-batch while simultaneously starting the next micro-batch. For example:
    
    - $\text{GPU}\_1$ processes micro-batch $1a$ on its layers (e.g., layers 0–2)
    - Once done, sends the partial result to $\text{GPU}\_2$ (layers 3–5)
    - At the same time, $\text{GPU}\_1$ does not wait and starts processing micro-batch $1b$ on its own layers

- **Interleaved Stages** → in a standard pipeline, each GPU handles a single contiguous block of layers (e.g., GPU 1 handles Layers 1–4). In an Interleaved approach:
    - Layers are split into smaller **chunks**.
    - Each GPU is assigned **multiple non-contiguous chunks**.
    
    *Example:* In a 16-layer model with 4 GPUs, instead of GPU 1 taking layers 1–4, it might take layers 1–2 and 9–10.
    
    N.B. It acts on layer partitioning rather than data.
    

![image.png](H3%20Model%20Efficiency/image%2017.png)

</aside>

## 7.2. Tensor Parallelism (Intra-Layer)

Instead of splitting layers, this method divides computations *within* each layer across multiple GPUs, which compute different parts of the same operation in parallel.

![image.png](H3%20Model%20Efficiency/4fe28a20-ab42-42f1-8a66-350198a03465.png)

It is typically implemented in two ways:

- **Row parallelism** → the matrix is split by rows, so each GPU computes a partial sum; the results are then combined using all-reduce.
- **Column parallelism** → the weight matrix is split by columns, so each GPU computes a portion of the output; the partial results are then gathered using all-gather.

<aside>
📌

**Conjugate Operators $f$**‬ **and $g$:**

In Tensor Parallelism, communication is managed by two special conjugate operators, $f$‬ and $g$‬, which are placed at the boundaries of the parallelized block. They are called "*conjugate*" because their behavior is swapped between the forward and backward passes:

- **Operator $f$ (at the block input)** → acts as an **identity** in the forward pass, simply forwarding the input $X$ to each GPU. During the backward pass, it performs an **all-reduce** to sum gradients across GPUs before propagating them to earlier layers.
- **Operator $g$ (at the block output)** → performs the opposite role. In the forward pass, it applies an **all-reduce** to combine partial outputs into the final result $Y$. In the backward pass, it behaves as an identity, sending gradients back without communication.
</aside>

![image.png](H3%20Model%20Efficiency/image%2018.png)

### 7.2.1. MLP Partitioning

The MLP block in a Transformer typically consists of two linear layers (matrix multiplications) separated by a non-linear activation function (GeLU). To minimize communication overhead, the optimized implementation applies a different parallelism strategy to each layer within the same block:

- **First Layer ($h \to 4h$): Column Parallelism** → the weight matrix $A\in \mathbb{R}^{h\times 4h}$ is split by columns. The input $X$‬ is provided to each GPU via an identity operator $f$:
    
    $$
    A = [A_1, A_2]  \implies [Y_1,Y_2] = \left[\text{GeLU}(XA_1),\ \text{GeLU}(XA_2)\right]
    $$
    
    Each GPU computes its output slice (e.g., $XA_1, XA_2$) and applies GeLU locally, with no communication needed.
    
- **Second Layer (**‭$4h ‬\to h$‬**): Row Parallelism** → the weight matrix $B \in \mathbb{R}^{4h\times h}$ is split by rows. It operates directly on the already partitioned outputs $(Y_1,Y_2)$ from the previous layer:
    
    $$
    B = \begin{bmatrix} B_1 \\ B_2 \end{bmatrix} \implies Z = Y_1B_1+Y_2B_2
    $$
    
    The partial results are finally combined using an **all-reduce** operator $g$.
    

![image.png](H3%20Model%20Efficiency/image%2019.png)

<aside>
📌

**Why this combination works?**

The two layers are designed to match each other’s data layout: the first produces partitioned activations that the second layer can consume directly. This removes the need for an intermediate all-gather between layers. Only a final all-reduce is required at the end of the MLP block to merge partial results into the final output.

</aside>

### 7.2.2. Self-Attention Tensor Partitioning

The same principles used in MLP tensor parallelism are applied to self-attention:

1. The projection matrices for **Query $Q$**, **Key $K$**, and **Value $V$** are split by columns, assigning a subset of attention heads to each GPU (e.g., if you have 16 heads and 2 GPUs, each GPU handles 8 heads).
2. The input $X$ is passed through operator $f$ (identity in the forward pass) and it’s replicated across GPUs.
3. Each device then computes attention independently on its assigned heads, including local Softmax and Dropout, producing partial outputs (e.g., $Y_1$, $Y_2$).
4. The final linear projection follows a **row-parallel** scheme: the weight matrix $B$ is split by rows (e.g., $B = [B_1,\ B_2]$) and applied directly to the already partitioned outputs.

![image.png](H3%20Model%20Efficiency/image%2020.png)

## 7.3. Data Parallelism

**Data parallelism** distributes a large dataset across multiple workers (GPUs), while **each worker keeps an identical copy of the model**.

1. Each worker processes its **own subset of data**, performing a forward pass and a backward pass to compute gradients.
2. The **gradients are then synchronized and averaged** across all workers so that every model copy updates in the same way before the next training step.

<aside>
📌

**Gradient Synchronisation Approaches:**

Two main approaches are used to synchronize gradients:

- **Parameter Server** → a central server collects gradients from all workers (master–slaves structure), averages them, and distributes the updated weights back. This creates a single point of failure  and a communication bottleneck.
- **All-Reduce** → a decentralized approach where all workers communicate directly with each other to share and average their gradients. This requires $n−1$ communication rounds but removes the central bottleneck.
</aside>

## **7.4. Hybrid Model Parallelism**

Training large language models at scale requires combining **Tensor, Pipeline, and Data Parallelism** simultaneously. 

<aside>
📌

**Example: Invidia NeMo (=Neural Modules)**

We consider a 4-layer neural network distributed across two nodes, each with 8 GPUs (16 GPUs in total), using a hybrid parallelism setup:

- **Tensor Parallelism ($TP =2$)** → splits computations *within* each layer across small groups of nearby GPUs connected by high-speed links (e.g., NVLink/NVSwitch).
- **Pipeline Parallelism ($PP=4$)** → the 4 layers of the model are distributed sequentially across GPUs so that data flows sequentially through the network during forward and backward passes.
- **Data Parallelism ($DP = 2$)** → two full replicas of the model (Model 0 and Model 1) are created to process different data batches in parallel.

**N.B.** These three dimensions interact naturally: $TP$ reduces intra-layer compute cost, $PP$ allows the model to exceed single-device memory limits, and $DP$ scales training by increasing the effective batch size. Together, they form a hybrid scheme where the total number of GPUs is the product of the three parallelism factors: $TP\cdot PP\cdot DP = 2 \cdot 4\cdot 2=16$.

![image.png](H3%20Model%20Efficiency/f3b24e64-cf9d-4519-b043-bc8737d0fc7b.png)

</aside>

---

# 8. Data Formats

Data is generally categorized into:

- **ASCII Formats** → text-based formats including JSON, XML, and CSV.
- **Binary Formats** → formats like JPEG images, which require decoding into a raw structure before the data can be processed. To represent binary data in an ASCII file, encoding schemes like Base64 are used. UTF-8 and UTF-16 are common encodings for representing text in binary form.

<aside>
🚨

**Data Structure Issues:**

- **File system overhead:** disks are split into blocks managed by allocation tables. Deep learning training often uses data shuffling to create random access patterns, which causes significant overhead due to the constant opening and closing of file inodes.
- **CSV limitation:** it stores data row by row, so to analyze a single column you must read each full line. Converting data to a column-based layout significantly makes it much faster to read and process specific columns.
</aside>

## 8.1. Optimized Formats for Deep Learning

To optimize input pipelines, several specialized data formats are used:

- **HDF5** → combines an ASCII header containing structural metadata with a binary payload, making it highly suitable for large datasets.
- **TFRecord** → binary format developed by Google that is optimized for sequential reading and deep learning I/O performance.
- **Parquet** → a columnar storage format that is highly efficient for data analytics.

<aside>
✅

**Best Practices for Performance**

- Avoid using numerous small JPEG files, as they create massive file system overhead. If possible, merge them into larger files, use optimized formats, and reduce batch size.
- If storage is slow and becomes a bottleneck, disable shuffling, store data in formats like HDF5, and lower the batch size.
- On Linux, `/dev/shm` allows you to store datasets in RAM, acting as a very fast filesystem. Since RAM is volatile, always save checkpoints to persistent storage.
</aside>

---

# 9. Memory Layouts in CV: NCHW vs NHWC

Computer vision tensors generally contain five dimensions:

- $N$ → batch size (number of images processed together)
- $C$ → channels (e.g., RGB)
- $H$ → height (Y-axis measurement)
- $W$ → width (X-axis measurement)
- $D$ → depth (distance from the camera to a pixel, optional)

Because memory is allocated sequentially, these multi-dimensional tensors must be unrolled. The order of unrolling heavily impacts hardware efficiency:

- **NCHW (Channel-First):** stores all values of one channel contiguously before moving to the next channel. It is the default in **PyTorch** and is well optimized for GPU execution. However, it can be less efficient for operations that compare values across channels.
- **NHWC (Channel-Last):** stores all channel values of each pixel consecutively (e.g., R, G, B for one pixel, then the next pixel). It is the default layout in **TensorFlow** and is highly cache-friendly for CPU implementations.
    
    **N.B.** Deep learning frameworks like TensorFlow will often transpose NHWC data to NCHW internally before calling GPU convolution kernels, which introduces processing overhead.
    

![image.png](H3%20Model%20Efficiency/image%2021.png)

---

# 10. Mixture of Experts (MoE)

**Mixture of Experts (MoE)** increases model capacity without proportionally increasing training or inference time by activating only a subset of the model.

<aside>
💡

**Core Idea:**

An MoE layer consists of multiple specialized sub-networks (**experts**) and a **gating network (router)** that assigns each input token to the most relevant experts. The router is critical and must be trained to make accurate routing decisions.

![image.png](H3%20Model%20Efficiency/ec30eb2b-7ba3-4d8f-ba18-130ce4e4cf89.png)

**N.B.** Empirically, routing each token to **1–2 experts** is usually sufficient to achieve strong performance.

</aside>

## 10.1. MoE in Transformers

MoE layers can be integrated directly into the Transformer architecture, replacing or augmenting the standard Feed-Forward Network (FFN) layer. This results in **Switch Transformers**. Because experts operate independently and do not need to communicate with one another, they are highly parallelizable.

![image.png](H3%20Model%20Efficiency/1c2453cf-bdf8-4aff-8d9a-b30df0d46414.png)

<aside>
📌

**GShard**

GShard enables scaling large models by distributing computation across multiple devices (GPU/TPU):

- Standard Transformer layers (e.g., Multi-Head Attention) are duplicated on each device
- Experts (FFN) are distributed across devices (each device hosts a subset of experts)

Since experts are spread across devices, tokens must be routed through the network:

1. **All-to-All Dispatch** → when an expert is not available on the local GPU, the router sends the token from their local GPU to the device where the chosen expert is located.
2. **All-to-All Combine** → after processing, outputs are sent back to the original GPU and merged to continue the forward pass.

![image.png](H3%20Model%20Efficiency/b9ea56fa-3ebd-4c0e-aab4-1115257d1db3.png)

</aside>

---