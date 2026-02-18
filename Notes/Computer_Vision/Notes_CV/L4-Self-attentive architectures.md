# L4-Self-attentive architectures

Done?: Done
Select: lab

# 1. Attention operator

<aside>
💡

**Attention** is an operator that allows a model to focus on the most relevant parts of an input.

</aside>

## 1.1. How it works?

Attention requires three components: **queries ($Q$)**, **keys ($K$)**, and **values ($V$)**.

1. Each **query** is compared with all the **keys** using a similarity function. This produces a set of scores that measure how relevant each key is to the query:
    
    $$
    
    a(q, k_i) = \text{similarity}(q, k_i)
    $$
    
2. These scores are normalized with a **softmax** function, producing attention weights that sum to $1$:
    
    $$
    
    \alpha(q,k_i) = \frac{e^{a(q,k_i)}}{\sum_j e^{a(q,k_j)}}
    $$
    
3. Each **value vector** $v_i$ is multiplied by its corresponding weight $\alpha(q,k_i)$.
4. The weighted values are summed to produce the **output vector** for the query:
    
    $$
    
    \text{Attention}(q, K, V) = \sum_{i=1}^m \alpha(q,k_i) v_i
    $$
    

**N.B.** This operation transforms each query into a new representation enriched by the most relevant information from the input.

![image.png](L4-Self-attentive%20architectures/image.png)

<aside>
📌

**Similarity Functions**

Different methods can be used to compute similarity between queries and keys:

- **Cosine Similarity** → 

$a(q,k) = \frac{q^T k}{|q||k|}$
- **Additive Attention** → **

$a(q, k) = w^T \tanh(w_q q + w_k k)$**
- **Dot-Product Attention** → **

$a(q, k) = \frac{q^T k}{\sqrt{d}}$**

These scoring functions determine how much attention each key-value pair receives when computing the final weighted sum.

</aside>

<aside>
🚨

**Constraints in the parameters:**

- The number of key vectors $k$ must be equal to the number of value vectors
- The dimensionality of the query $q$ must match the dimensionality of the keys $k$ → 

$\dim(q) = \dim(k)$
</aside>

## 1.2. Matrix Formulation

Since all the operation inside the attention layers are linear we can use matrices to process many queries in parallel. Given a input sequence of length $T$ we get:

- $Q \in \mathbb{R}^{T \times d_k}$ → matrix of queries
- $K \in \mathbb{R}^{T \times d_k}$ → matrix of keys
- $V \in \mathbb{R}^{T \times d_v}$ → matrix of values

The attention mechanism can then be written compactly as:  $Attention(Q, K, V) = softmax(similarity(Q,K))\cdot V$

**N.B.** **Where did the $\sum$ go?** It’s implicit in the matrix multiplication.

## 1.3. What is the real meaning of $q$, $k$, $v$?

- **Query ($q$):** the question that a token/patch asks to the others (e.g., *“Who else has a similar color?”*).
- **Key ($k$):** the “answer” or identifier of a token/patch, telling whether it matches a query.
- **Value ($v$):** the actual content or information that a token/patch provides to others.

Through training, the model learns the optimal transformations $W_q$, $W_k$, $W_v$ that map inputs into queries, keys, and values.

---

# 2. Convolution vs Self-attention

**Convolution** 

- It has a **fixed receptive field**, meaning it focuses on local information
- **Weights** in convolution are **shared** across the input
- Computational cost is **lower**, making it efficient for local patterns
- It has a **varying path length** between distant positions
- It’s more **complex** to parallelize **because it depends** on sequential operations

![image.png](L4-Self-attentive%20architectures/image%201.png)

**Self-attention**

- It has a **global receptive field**, allowing to to capture long-range dependencies
- It uses **different weights** for each interaction
- It’s more **expensive** but enables direct connections between all elements
- It maintains a **constant path length**, making information flow more efficient
- It’s easier to **parallelize** due to its reliance on **matrix operations**

![image.png](L4-Self-attentive%20architectures/image%202.png)

---

# 3. Multi-Head Attention

## 3.1. Intuition

The core idea behind multi-head attention is to allow the model to simultaneously focus on different aspects of the input data. 

Example: imagine you have a sentence, and you want to understand the relationships between words. A single attention mechanism might only capture one type of relationship. Multi-head attention, however, enables the model to capture multiple relationships at once, such as grammatical dependencies, semantic similarities etc.

## 3.2. How it works?

1. **Linear Projections** → let’s suppose we have an input of length $T$, where each element has dimension $d_m$. We project the input into $h$ **different sets of Queries, Keys, and Values** using separate weight matrices. For each head, this gives us: $Q_i \in \mathbb{R}^{T \times d_k}$, 

$K_i \in \mathbb{R}^{T \times d_k}$

, 

$V_i \in \mathbb{R}^{T \times d_v}$

.
2. **Parallel attention** → each head computes its own attention:
    
    $$
    
    head_i = Attention(Q_i, K_i, V_i) = \text{softmax}\left(\text{similarity}(Q_i,K_i)\right)V_i
    
    $$
    
    Output per head: a matrix of shape $(T, d_v)$.
    
3. **Concatenation** → the outputs of all $h$ heads are concatenated along the feature dimension: 

$Concat(head_1,\dots,head_h) \in \mathbb{R}^{T \times (h\cdot d_v)}$

4. **Final linear projection** → to bring the concatenated result back to the original input dimension $d_m$, we apply a final projection with a matrix $W_O$:
    
    $$
    
    MultiHead(Q,K,V) = Concat(head_1,\dots,head_h)W_O \in \mathbb{R}^{T \times d_m}
    
    $$
    

![image.png](L4-Self-attentive%20architectures/image%203.png)

**N.B.** The number of head $h$ is a hyperparameter. **More heads** → **More parameters to learn** (because each head has its own set of weights) → **More expressive power**  (since each head captures different aspects of the input).

## 3.3. Positional Encoding

Self-attention is **permutation-invariant**: if we shuffle the tokens, it produces the same output. This is problematic for sequences or images where order matters.

To use **order information**, we can inject absolute or relative positional information by adding positional encoding to the input representations. Positional encoding can be:

- **Learned** → trainable parameters
- **Fixed** → sinusoidal functions (originally used in the Transformer), suboptimal

## 3.4. Add & Norm Layer

After multi-head attention, we use an **Add & Norm** layer:

1. Add the residual connection → 

$\tilde{x} = F(x) + x$
2. Apply layer normalization → 

$\hat{x} = \frac{\tilde{x} - \mu}{\sigma} \cdot \gamma + \beta$ (where $\gamma$ and $\beta$ are learnable parameters)

**N.B.** This ensures stability and preserves gradient flow.

## 3.5. Position-wise Feed Forward Network (FNN)

Finally, we enrich each token with **non-linearity**: 

$FFN(x) = ReLU(xW_1 + b_1)W_2 + b_2$

The FFN is applied **independently to each token**, with shared parameters across positions.

---

# 4. Transformer model

The transformer consists of two main components:

- **Encoder** → processes the **input** sequence. It consists of:
    - Applies **Self-Attention** to capture relationships within the input.
    - Uses **Add & Norm** → residual connection + layer normalization.
    - Includes a **Position-wise Feed-Forward Network (FFN)** → enriches each token representation independently.
    
    **N.B.** In image processing, the encoder is more commonly used than the decoder.
    
- **Decoder** → generates the **output** sequence. It consists of:
    - **Masked Self-Attention** → similar to the encoder’s self-attention, but it prevents the model from looking at future tokens during training.
    - **Cross-Attention** → queries come from the decoder’s sequence, while keys and values come from the encoder’s outputs. This allows the decoder to attend to the most relevant parts of the input sequence.
    - **Feed-Forward + Add & Norm** → same structure as in the encoder, applied after each attention block.
    
    **N.B.** The decoder is primarily used in NLP models
    

![image.png](L4-Self-attentive%20architectures/image%204.png)

**N.B.** Transformer is a **Isotropic architecture** because it doesn’t change the dimension across all layers.

---

# 5. Vision Transformer (ViT)

## 5.1. How it works?

- The image is divided into fixed-size patches (e.g., 16×16 pixels).
- Each patch is flattened into a vector and **embedded**. Positional **encodings** are added to these patch embeddings to retain spatial information.
- These embedded patches are then fed into a standard Transformer encoder.
- For image classification, a **classification head** (usually a simple linear layer) is added on top of the encoder’s output.

![image.png](L4-Self-attentive%20architectures/image%205.png)

<aside>

**Patch Dimensions**

The dimension of the input patches is a hyperparameter:

- **Smaller patches** → **longer output** → **better results** (but memory limitations, so we can’t divide each pixel in a batch)
</aside>

## 5.2. Data-efficient Vision Transformers

Vision Transformers (**ViTs**) usually need **very large datasets** (e.g., **JFT-300M**) to perform well. On smaller datasets, they risk **overfitting** and poor generalization.

To improve **data efficiency**, one common approach is **knowledge distillation**: a ViT (*student*) learns to **mimic a pre-trained CNN (teacher)**, allowing it to learn effectively with less data.

---