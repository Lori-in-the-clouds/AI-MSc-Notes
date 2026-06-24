# 5. Parameter Efficient Fine-Tuning

Pages: 1
Status: Done
Type: theory

# 1. Introduction

Standard (full) fine-tuning updates all parameters of a pre-trained model, making it computationally expensive in both time and memory. **Parameter Efficient Fine-Tuning (PEFT)** techniques aim to reduce the number of trainable parameters during fine-tuning. This offers advantages in:

- **Memory** → reduced storage requirements for optimizers and gradients.
- **Time** → faster training due to fewer parameters being updated.
- **Generalization** → updating fewer parameters can help mitigate the risk of overfitting.

---

# 2. PEFT Approaches

Several techniques exist, categorized by their approach:

- **Subset**
- **Adapters**
- **Prompt Tuning & Prefix Tuning**
- **LoRa**

## 2.1. Subset

It consists of fine-tuning only a subset of layers (typically the top ones) while keeping the rest of the network frozen. This means:

1. Keep all parameters fixed except for the top $K$ layers
2. Gradients flow only through these $K$ layers instead of the full network ($K+L$ layers)

Efficiency**:** the optimizer only needs to track gradients and update parameters in the selected upper layers, saving memory and computation time for the frozen lower layers.

**N.B.** However, the full model still needs to be stored in memory for the forward pass, including frozen layers.

![image.png](5%20Parameter%20Efficient%20Fine-Tuning/db4ae137-7ec1-4788-a40b-1206cea857a7.png)

## 2.2. Adapters

It introduce small, trainable modules between the layers of a pre-trained network, while keeping the original weights fixed.

### 2.2.1. Structure

Instead of using a single large matrix $(d \times d )$, the adapter uses two smaller matrices:

1. **Feed-Forward down-projection (matrix $d\times r$)** → maps the high-dimensional input $d$ to a smaller bottleneck dimension $r$.
2. **Non-linearity** → is applied in this reduced space. Without a non-linear activation function, $W_{down}$ and $W_{up}$ would collapse in a single linear transformation.
3. **Feed-Forward up-projection (matrix $r\times d$)**  → restores the original dimension $d$ and the result is added to the input via a residual connection.

**N.B.** The output dimension must match the input dimension to maintain compatibility with the original layer.

<aside>
📌

**Why it saves parameters?**

The total parameters of the two matrices $(d \times r + r \times d)$ are much smaller than a single square matrix $(d \times d)$.

*Example:* 

If $d = 1024$ and $r = 64$:

- Standard layer → $1024 \times 1024 \approx 1M \ \text{parameters}$
- Adapter → $1024 \times 64 + 64 \times 1024 \approx 131K\ \text{parameters}$
</aside>

![image.png](5%20Parameter%20Efficient%20Fine-Tuning/f4b620d3-ce9a-4ab1-b8a5-fcae0fff88a0.png)

Efficiency: by choosing an appropriate bottleneck dimension $r 

\ll

d$, adapters can constitute only $0.5\%$ to $8\%$ of the total network parameters, achieving performance close to full fine-tuning.

## 2.3. Prompt Tuning & Prefix Tuning

It adapt a pre-trained Transformer by adding trainable vectors (“virtual tokens”) to the input while keeping all original model parameters frozen:

- **Prompt Tuning** → a single set of trainable tokens is prepended at the input embedding layer and then processed through all layers. It requires very few parameters and is lightweight to store, but it performs well mainly on large models and simpler tasks.
    
    ![image.png](5%20Parameter%20Efficient%20Fine-Tuning/image.png)
    
- **Prefix Tuning** → extends this idea by adding a distinct set of trainable prefix vectors at every Transformer layer. At each layer, a new prefix is injected while previous prefix activations are discarded, preventing sequence growth and allowing layer-specific control.
    
    ![image.png](5%20Parameter%20Efficient%20Fine-Tuning/image%201.png)
    
    <aside>
    🔢
    
    **Trainable Parameters**
    
    The number of trainable parameters in Prefix Tuning is computed as:
    
    $$
    
    \text{Parameters} = \text{N. of prefix Token} \times \text{N. of  Layer} \times (2 \times \text{Hidden dimension})
    $$
    
    The factor of 2 comes from the fact that, at each layer, separate prefix vectors are learned for both the key and value projections in the attention mechanism.
    
    </aside>
    

**N.B.** Prompt tuning uses one shared set of embeddings for the entire model, whereas prefix tuning learns separate embeddings for each layer.

<aside>
🚨

**Limitations**

Both methods may converge more slowly than full fine-tuning, reduce available context length, have limited interpretability, and require tuning of the prefix size.

</aside>

## 2.4. LoRA

It’s one of the most widely used PEFT techniques, based on the idea that fine-tuning updates lie in a low-dimensional subspace. The original weights are kept frozen, and only a low-rank update is learned.

### 2.4.1. How it works?

For a weight matrix $W_0 \in \mathbb{R}^{d \times k}$, LoRA learns an additive update $\Delta W = BA$, where $B \in \mathbb{R}^{d \times r}$ and $A \in \mathbb{R}^{r \times k}$ with $r \ll \min(d,k)$. The forward pass becomes:

$$
z = (W_0 + BA)x
$$

The matrix $A$ is randomly initialized, while $B$ is initialized to zero so that $\Delta W = 0$ at the start, preserving the original model behavior. The computation can be seen as the sum of a fixed branch ($W_0\cdot  x$) and a trainable low-rank branch ($B\cdot A\cdot x$), without introducing additional non-linearities since the update is linear.

**N.B.** You can also keep the architecture divided, maintaining both the fixed branch and the low-rank branch as separate components instead of merging them.

![image.png](5%20Parameter%20Efficient%20Fine-Tuning/99666eda-f823-4434-9089-40f994b82ea6.png)

<aside>
💡

**Application to Transformers**

LoRA is applied by replacing linear layers within the Transformer architecture (e.g., in attention mechanisms and feed-forward networks).

![image.png](5%20Parameter%20Efficient%20Fine-Tuning/36c588cf-8cdc-4a91-a2e5-cd4ac76fb512.png)

In Transformers, applying LoRA only to the attention weight matrices (e.g., $W_q$ and $W_v$) can achieve performance comparable to, or even better than, full fine-tuning while using far fewer parameters. In practice, very low ranks (e.g. $r = \{2,4,8,16\}$) are often sufficient. LoRA variants have also been successfully applied to Vision Transformers, sometimes outperforming full fine-tuning in computer vision tasks.

</aside>

<aside>
🚨

**Adapters vs LoRa**

- **Adapters** → small trainable modules inserted directly into the Transformer layers. Since they add extra operations to the forward pass, they increase inference latency.
- **LoRA** → because the update is purely linear, after training the matrices can be merged into the original weights ($W_0 + BA$). As a result, the model performs a single matrix multiplication at inference time, introducing **no additional latency**.
</aside>

---