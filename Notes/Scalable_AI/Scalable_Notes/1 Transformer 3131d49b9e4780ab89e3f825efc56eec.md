# 1. Transformer

Pages: 3
Status: Done
Type: theory

# 1. Transformer Architecture

The original Transformer architecture, initially used for translation tasks, is built from repeated layers organized into two main parts:

- **Encoder** → processes the input sequence using **self-attention**. Each layer includes:
    - A **self-attention** mechanism
    - An **Add & Norm** step (residual connection followed by normalization)
    - A **feed-forward network** (linear layers applied independently at each time step)
    
    Several of these layers are stacked together to build the full Encoder.
    
- **Decoder** → generates the output sequence step by step. Each layer includes:
    - **Masked Self-Attention** that is similar to the encoder’s self-attention, but it prevents the model from looking at future tokens during training.
    - **Cross-attention**, queries come from the decoder’s sequence, while keys and values come from the encoder’s outputs. This allows the decoder to attend to the most relevant parts of the input sequence.
    - **Feed-Forward + Add & Norm**, ****same structure as in the encoder, applied after each attention block.

![image.png](1%20Transformer/image.png)

---

# 2. Data Representation and Embeddings

1. Language models process text by first converting it into **tokens**, which are elements of a vocabulary usually represented as integers. For a mini-batch of size $n$ and sequence length $T$, the input tensor has shape $(n, T)$.
    
    Since sequences can have different lengths, we use **padding** by adding special **`<PAD>` tokens** so that all sequences in the same mini-batch have the same length $T$.
    
    <aside>
    📌
    
    **Sub-word Tokenisation (BPE)**
    
    Modern tokenisation does not assign one token per whole word. Instead, it uses methods like **Byte Pair Encoding (BPE)**, which break words into sub-word units and even single letters. This helps the model handle typos and process words that are not in the vocabulary.
    
    </aside>
    
2. The **Embedding Layer** converts integer tokens into **dense vectors** using a lookup table. Each token corresponds to a row in a weight matrix of size $(V, d)$, where $V$ is the vocabulary size and $d$ is the embedding dimension.
3. After the embedding step, the input tensor changes from shape $(n, T)$ to $(n, T, d)$, allowing the model to represent the semantic meaning of each token as a point in a high-dimensional space.

![image.png](1%20Transformer/image%201.png)

---

# 3. Positional Encoding

Self-attention is **permutation-invariant**: if we shuffle the tokens, it produces the same output. This is problematic for sequences or images where order matters. To use **order information**, we can inject absolute or relative positional information by adding positional encoding to the input representations.

![image.png](1%20Transformer/image%202.png)

Positional encoding can be:

- **Learned** → trainable parameters
- **Fixed** → sinusoidal functions (originally used in the Transformer), suboptimal
    
    <aside>
    📌
    
    **Sinusoidal functions:**
    
    Given an input representation $X\in \mathbb{R}^{n\times d }$ , the positional encoding outputs $X +P$ using a a positional embedding matrix $P \in \mathbb{R}^{n\times d }$ of the same shape, whose element on the $i^{th}$ row and the $(2j)^{th}$ or the $(2j+1)^{th}$ column is:
    
    $$
    p_{i,2j}=\sin\left(\frac{i}{10000^{2j/d}}\right)\quad p_{i,2j+1}=\cos\left(\frac{i}{10000^{2j/d}}\right)
    $$
    
    ![image.png](1%20Transformer/image%203.png)
    
    </aside>
    

**N.B.** While the input representation changes depending on the mini-batch being loaded, the **positional encoding remains constant**. It is not data-dependent, meaning it does not change when loading new textual data. However, the positional encoding must match the fundamental nature and shape of the data being processed.

---

# 4. Self-Attention

## 4.1. Attention operator

<aside>
💡

**Attention** is an operator that allows a model to focus on the most relevant parts of an input.

</aside>

## 4.2. How it works?

Suppose that we have a **query** $q\in \mathbb{R}^q$ and $m$ **key-values** pairs $(k_1,v_1),...,(k_m,v_m)$, where any $k_i\in \mathbb{R}^k$ and any $v_ì\in \mathbb{R}^v$:

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
    
    f\left(q, (k_1,v_1)..., (k_m,v_m)\right) = \sum_{i=1}^m \alpha(q,k_i)\cdot v_i \in \mathbb{R}^v
    $$
    

![image.png](1%20Transformer/image%204.png)

**N.B.** This operation transforms each query into a new representation enriched by the most relevant information from the input.

<aside>
📌

**Similarity Functions**

Different methods can be used to compute similarity between queries and keys:

- **Dot-Product Attention** → **

$a(q, k) = \frac{q^T k}{\sqrt{d}}$**
- **Additive Attention** → **

$a(q, k) = w^T \tanh(w_q q + w_k k)$**

These scoring functions determine how much attention each key-value pair receives when computing the final weighted sum.

</aside>

<aside>
🚨

**Constraints in the parameters:**

- The number of key vectors $k$ must be equal to the number of value vectors $v$.
- The dimensionality of the query $q$ must match the dimensionality of the keys $k$ → 

$\dim(q) = \dim(k)$
</aside>

### 4.2.1. What is the real meaning of $q$, $k$, $v$?

- **Query ($q$):** the question that a token/patch asks to the others (e.g., *“Who else has a similar color?”*).
- **Key ($k$):** the “answer” or identifier of a token/patch, telling whether it matches a query.
- **Value ($v$):** the actual content or information that a token/patch provides to others.

Through training, the model learns the optimal transformations $W_q$, $W_k$, $W_v$ that map inputs into queries, keys, and values.

### 4.2.2. Matrix Formulation

Since all the operation inside the attention layers are linear we can use matrices to process many queries in parallel. Given a input sequence of length $T$ we get:

- $Q \in \mathbb{R}^{T \times d_k}$ → matrix of queries
- $K \in \mathbb{R}^{T \times d_k}$ → matrix of keys
- $V \in \mathbb{R}^{T \times d_v}$ → matrix of values

The attention mechanism can then be written compactly as:  $Attention(Q, K, V) = softmax(similarity(Q,K))\cdot V$

**N.B.** **Where did the $\sum$ go?** It’s implicit in the matrix multiplication.

## 4.3. Convolution vs Self-attention

**Convolution** 

- It has a **fixed receptive field**, meaning it focuses on local information
- **Weights** in convolution are **shared** across the input
- Computational cost is **lower**, making it efficient for local patterns
- It has a **varying path length** between distant positions
- It’s more **complex** to parallelize **because it depends** on sequential operations

![image.png](1%20Transformer/image%205.png)

**Self-attention**

- It has a **global receptive field**, allowing to to capture long-range dependencies
- It uses **different weights** for each interaction
- It’s more **expensive** but enables direct connections between all elements
- It maintains a **constant path length**, making information flow more efficient
- It’s easier to **parallelize** due to its reliance on **matrix operations**

![image.png](1%20Transformer/image%206.png)

## 4.4. Multi-Head Attention

The core idea behind multi-head attention is to allow the model to simultaneously focus on different aspects of the input data. Below we describe the mechanism **for a single batch element**:

1. **Linear Projections** → let’s suppose we have an input of length $T$, where each element has dimension $d_m$. We project the input into $h$ **different sets of Queries, Keys, and Values** using different weight matrices. For each head, this gives us: $Q_i \in \mathbb{R}^{T \times d_k}$, 

$K_i \in \mathbb{R}^{T \times d_k}$

, 

$V_i \in \mathbb{R}^{T \times d_v}$

. So we obtain:
    - $Q\in \mathbb{R}^{h\times T \times d_k}$
    - $K\in \mathbb{R}^{h\times T \times d_k}$
    - $V\in \mathbb{R}^{h\times T \times d_v}$
2. **Parallel attention** → each head computes its own attention:
    
    $$
    
    head_i = Attention(Q_i, K_i, V_i) = \text{softmax}\left(\text{similarity}(Q_i,K_i)\right)V_i \in \mathbb{R}^{T\times d_v}
    
    $$
    
3. **Concatenation** → the outputs of all $h$ heads are concatenated along the feature dimension: 

$Concat(head_1,\dots,head_h) \in \mathbb{R}^{T \times (h\cdot d_v)}$

4. **Final linear projection** → to bring the concatenated result back to the original input dimension $d_m$, we apply a final projection with a matrix $W_O$:
    
    $$
    
    MultiHead(Q,K,V) = Concat(head_1,\dots,head_h)W_O \in \mathbb{R}^{T \times d_m}
    
    $$
    

![image.png](1%20Transformer/image%207.png)

**N.B.** The number of head $h$ is a hyperparameter. **More heads** → **More parameters to learn** (because each head has its own set of weights) → **More expressive power** (since each head captures different aspects of the input).

<aside>
📌

**Distributed Training Across GPUs**

When training the model across multiple GPUs (e.g., splitting the sequence across two GPUs), specific operations have different communication overheads:

- Extraction of keys, values, and queries → Local into the GPU.
- Division by the scaling dimension → Local into the GPU.
- The dot product between sequences → Requires cross-GPU communication.
- `softmax` operation → Requires cross-GPU communication.

Since the network connection is considerably slower than the internal PCI bus, minimizing this communication is a core challenge in scalable AI.

</aside>

## 4.5. Performance Comparison

- **Complexity per Layer:**
    - **RNN $O(n\cdot d^2)$:** operates strictly sequentially. For each of the $n$ tokens, it updates a hidden state of dimension $d$. This update requires multiplying a $d$-dimensional vector by a $d\times d$ recurrent weight matrix, yielding $d^2$ operations per step.
    - **CNN $O(k\cdot n\cdot d^2)$:** slides a filter of kernel size $k$ across the $n$ tokens. Transforming the $d$-dimensional vectors involves a $d\times d$ weight matrix for every token within that kernel window.
    - **Self-Attention $O(n^2\cdot d)$:** computes attention scores for every possible pair of tokens in the sequence ($n^2$), multiplied by the representation dimension $d$.
- **Sequential Operations:**
    - **RNN $O(n)$:** strict bottleneck. The model must process tokens step-by-step.
    - **CNN $O(1)$:** full parallelization. Kernels compute all sequence positions simultaneously.
    - **Self-Attention $O(1)$:** full parallelization. All tokens are processed at once.
- **Maximum Path Length:**
    - **RNN $O(n)$:** signal passes through all steps.
    - **CNN $O(log_k(n))$:** in the first layer, each token sees only its local neighbors (defined by kernel size $k$). As more layers are added, the receptive field grows, allowing information from distant tokens to be combined in fewer steps.
    - **Self-Attention $O(1)$:** direct connection. A signal travels between the first and last token instantly without degradation.

<aside>
📌

**Restricted Self-Attention**

To reduce the $O(n^2$) memory cost of standard self-attention, Restricted Self-Attention limits each token to a local window of size $r$, instead of the full sequence.

- **Complexity per Layer $O(r\cdot n\cdot d)$:** linear in $n$, since attention is computed only within a fixed window $r$. This makes very long sequences feasible.
- **Sequential Operations $O(1)$:** all windows are still computed simultaneously on the GPU.
- **Maximum Path Length $O(n/r)$:** by limiting attention to a small local window, the instantaneous global connection $O(1)$ is lost. For information to travel from one end of the sequence to the other, it must be passed through overlapping windows, requiring a number of steps proportional to the total sequence length divided by the window size.
</aside>

---

# 5. Other Components

## 5.1. Add & Norm Layer

In modern Transformer architectures, every sublayer (Multi-Head Attention or Feed-Forward Network) is wrapped inside an **Add & Norm** block with **Dropout**. This structure is essential for stable training of deep networks. The complete formula is:

$$
\text{Output} = \text{LayerNorm}(x+ \text{Dropout}(F(x)))
$$

**Step-by-Step:**

- **Dropout** → it’s applied to $F(x)$, randomly setting some activations to zero. This acts as a regulariser, improving robustness and reducing overfitting.
- **Residual Connection** → this shortcut connection allows gradients to flow directly through the network during backpropagation, helping prevent the vanishing gradient problem: $\tilde{x} = F(x) + x$
- **Norm** → finally LayerNorm is applied: $\hat{x} = \frac{\tilde{x} - \mu}{\sigma} \cdot \gamma + \beta$ where:
    - $\mu$ and $\sigma$: mean and standard deviation computed across the layer.
    - $\gamma$ and 
    $\beta$
    : learnable parameters that scale and shift the normalized values.
    
    **N.B.** This structure helps prevent both **vanishing** and **exploding gradients**, ensuring stable training in deep networks.
    

<aside>
📌

**Add & Norm: LayerNorm vs. ResNet**

- **Batch Normalization (ResNet)** →  normalizes  across the batch dimension for each feature.
    
    *Problem with text:* sequences of different lengths (e.g., 5 vs. 50 tokens with padding) make the batch mean unreliable, causing instability.
    
- **LayerNorm (Transformer)** → normalizes across the features of a single token. This ensures that the internal activations remain stable across many layers.

---

Example:

![image.png](1%20Transformer/9725dce1-f0e5-42fb-b540-53d4cb55173c.png)

</aside>

### 5.1.1. **Pre-Normalization vs. Post-Normalization**

The position of **LayerNorm** strongly affects training stability, especially in deep Transformers:

- **Post-Normalization (Original Transformer):** the formula is $\text{LayerNorm}(x+\text{Sublayer}(x))$. This was used in the original Transformer paper but can lead to vanishing or exploding gradients in very deep architectures.
- **Pre-Normalization (Modern Standard)**: the formula is $x+\text{Sublayer}(\text{LayerNorm}(x))$. Here, the normalization is applied **before** the Attention or Feed-Forward sublayer. It ensures that the original input $x$ is preserved and passed along without being transformed by a normalization layer. This improves **gradient flow**, increases training stability.

## 5.2. Position-wise Feed Forward Network (FNN)

After each self-attention block, the model includes a **position-wise Feed-Forward Network (FFN)**. This is a simple MLP that is identical for all tokens and applied independently to each position in the sequence. While self-attention allows tokens to interact, the FFN processes the information within each individual token.

<aside>
🔑

**Why?**

- It exploits the contextual information produced by the self-attention layer.
- It increases the model’s depth and expressive power by adding a non-linear transformation (ReLU).
- It increases resolution by expanding the vector into a much larger hidden space $d_{ff}$, making it easier to separate and extract features that were previously overlapping in the smaller model dimension.
</aside>

### 5.2.1. How It Works?

The FFN consists of two linear (fully connected) layers with a ReLU activation in between. It first projects the representation to a higher-dimensional hidden space, then back to the original dimension:

$$
FNN(x)=\max(0,W_1x+b_1)W_2+b_2
$$

Where:

- $W_1\in\mathbb{R}^{d\times d_{ff}}$: expands from $d$ to a larger hidden dimension $d_{ff}$ (typically $4\times$ larger).
- $W_2\in\mathbb{R}^{d_{ff} \times d }$: projects back to $d$.

For computational efficiency, the input tensor of shape $(n, T, d)$ is reshaped to $(n \cdot T, d)$, allowing one large matrix multiplication over all tokens. Then it’s reshaped back to  $(n, T, d)$:

![image.png](1%20Transformer/6561d289-925f-4626-997f-0b2aeddda48b.png)

---

# 6. Training Phase vs. Prediction Phase

## 6.1. Decoder’s Training

In the Transformer architecture, the training phase of the Decoder is significantly different from inference. During training, the Transformer Decoder follows these steps:

1. **Shift the Target Sequence** → during training, the target sentence is shifted one position to the right and fed to the decoder. For example:
    - Decoder Input: `<SOS>` un uomo corre `⟨EOS⟩`
    - Target: un uomo corre `⟨EOS⟩` `⟨PAD⟩`
    
    At each position, the model learns to predict the next token while receiving the correct previous tokens (*teacher forcing*). A **causal (triangular) mask** prevents attention to future positions, ensuring that token $t$ can only attend to tokens $1$ to $t$. 
    
    **N.B.** Since the entire shifted sequence is available during training, all positions can be processed in parallel, significantly speeding up training.
    
2. **Perform Cross-Attention with the Encoder** → the Decoder attends to the Encoder outputs (keys and values). This allows it to extract relevant context from the source sentence to guide prediction.
3. **Project to Vocabulary Space** → a final Linear (Fully Connected) layer maps the hidden representation from dimension $d$ to vocabulary size $V$. The output tensor has shape $(n, T, V)$, where $n$ is the batch size.
4. **Apply Softmax and Compute Prediction** → each position produces logits (raw scores) over the vocabulary. Softmax converts these into probabilities, and the highest-probability token is selected as the prediction.

![image.png](1%20Transformer/b8b446dc-dfb0-43bd-9cbe-e1fa575df199.png)

<aside>
📌

**Attention Mask: Why $-\infty$ ?**

Masking tokens (for padding or future words) with $0$ **before Softmax** is incorrect: since $e^0 = 1,$ the masked token still gets a non-zero probability, so the model continues to attend to it. Instead, using a very small value like $-\infty$ (or $−1e^9$) gives $e^{-\infty} = 0$, so Softmax completely ignores these positions, assigning them $0\%$ probability.

</aside>

## 6.2. Decoder’s Prediction

During inference, the Decoder works **auto-regressively**: each predicted token is fed back as input for the next step. Since the ground-truth sentence is unknown, the parallelization used in training is no longer possible.

1. **Starting Point** → the process begins with the special **Start of Sentence (`⟨SOS⟩`)** token. The Encoder processes the entire input sentence (e.g., *“A man is running”*) and its outputs (Keys and Values) are **cached** and reused at every step by the Decoder’s cross-attention layer, reducing computation.
2. **Step-by-Step Generation** → the Decoder then follows a sequential loop:
    1. It receives the tokens generated so far (initially just `⟨SOS⟩`).
    2. It performs a forward pass and produces a probability distribution over the vocabulary $V$ for the next token.
    3. The next token is chosen, either by taking the highest-probability word (greedy decoding) or using strategies like Beam Search.
    4. The selected token is appended to the sequence and fed back into the Decoder for the next step.
    
    **N.B.** To make this repeated operation efficient, it is necessary to *cache* the tokens and states that have already been processed; otherwise, the model would have to recalculate the entire sequence from scratch at every step.
    

![image.png](1%20Transformer/f935e765-34a1-447b-b978-3213def66474.png)

---