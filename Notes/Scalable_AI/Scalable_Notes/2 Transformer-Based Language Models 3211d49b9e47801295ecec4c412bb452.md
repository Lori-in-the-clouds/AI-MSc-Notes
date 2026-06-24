# 2. Transformer-Based Language Models

Pages: 2
Status: Done
Type: theory

# 1. Architecture vs Model

Architecture alone does not make a model: 

$$
\text{Architecture} + \text{Training} = \text{Model}
$$

A model expresses different properties depending on how it is trained, not just on its structure. In modern scalable AI, the fundamental architecture remains largely consistent across different models. Unlike Convolutional Neural Networks (CNNs), which utilized highly varied structures, current large models differentiate themselves primarily through their scale and the number of learnable parameters.

---

# 2. BERT (Bidirectional Encoder Representations from Transformers)

<aside>
💡

**BERT** stands out in the NLP landscape for its unique encoder-only architecture and its use of the pre-training and fine-tuning paradigm. Unlike autoregressive models, it uses full bidirectional self-attention, allowing each token to attend to both left and right context.

</aside>

## 2.1. Self-Supervised Pre-training

BERT is trained in a fully **self-supervised** way: labels are automatically generated from the input text itself. This makes large-scale training feasible without manual annotation. The pre-training includes two objectives:

1. **Masked Language Modeling (MLM)** → about $15\%$ of tokens are randomly masked. The model predicts the missing tokens using **bidirectional context** (both left and right).
2. **Next Sentence Prediction (NSP)** → given two sentences ($A$ and $B$), the model predicts whether $B$ truly follows $A$. A special `[CLS]` token is added at the beginning, and its final hidden state is used for binary classification.

![image.png](2%20Transformer-Based%20Language%20Models/image.png)

<aside>
📌

**Input Representation**

Each input token is represented as the sum of three embeddings:

- **Token Embeddings** → represent the token itself.
- **Positional Embeddings** → indicate the position of the token in the sequence.
- **Segment Embeddings (A/B)** →  a learned embedding (A or B) that helps the model distinguish between the first and second sentences (it’s not strictly necessary).

![image.png](2%20Transformer-Based%20Language%20Models/image%201.png)

</aside>

## 2.2. Fine-Tuning

After the self-supervised pre-training, the model is fine-tuned on a specific, smaller, labeled dataset for a downstream task (e.g., sentiment analysis, question answering). The pre-trained weights provide a powerful starting point, allowing for high performance with less task-specific data. 

**N.B.** This paradigm marked a major shift in NLP, moving from task-specific models to a **pre-trained + fine-tuned approach**.

---

# 3. The GPT Family **(Generative Pre-trained Transformer)**

<aside>
💡

 **GPT** is a **decoder-only Transformer** that generates text by predicting the next word in a sequence, given the context.

</aside>

## 3.1. Training Strategy

GPT models are trained using **next-token prediction**, meaning the model learns to predict the next word in a sequence given all previous words. This simple objective enables the model to learn grammar, semantics, and even reasoning patterns directly from large text corpora.

<aside>
📌

**In-Context Learning**

One of the most important properties of GPT models is **in-context learning**. Instead of updating model parameters for each new task, GPT can adapt by using the examples provided in the input prompt. This allows it to perform tasks in:

- **zero-shot** (no examples)
- **one-shot** (one example)
- **few-shot** (a few examples)

without any additional training or gradient updates. The first that introduces zero-shot was GPT-3, it represents a major **advance** in LLM capabilities.

</aside>

## 3.2. Evolution of the GPT Series

Across the GPT family, the **core architecture remains largely unchanged**. Most improvements come from **scaling**, both in terms of model size (e.g., GPT-3 with 175 billion parameters) and the quality and quantity of training data.

![image.png](2%20Transformer-Based%20Language%20Models/image%202.png)

Over time, the paradigm has shifted from simple **text completion** to **instruction following**, where models are trained to understand and execute tasks described in natural language prompts.

## 3.3. Codex

**Codex** is a specialized variant of GPT designed for **programming tasks**:

- **Training** → it’s a GPT model pre-trained on a large corpus of public source code. Code is often easier to model than natural language because it is structured and unambiguous.
- **Evaluation** → models are measured by **functional correctness**: the generated code is run against unit tests to verify it behaves as expected. The same principle applies to tasks involving math or logical reasoning.
    
    <aside>
    📌
    
    **pass@k metric:**
    
    Functional correctness can be evaluated using the $\text{pass}@k$ metric. It measures the fraction of problems for which **at least one out of $k$ generated samples** passes all test cases:
    
    $$
    
    \text{pass@k} = 1 - (1 - \text{pass@1})^k
    $$
    
    Where:
    
    - $\text{pass}@1$ → probability that a single generated solution is correct (first-try accuracy).
    - $k$ → number of independent solutions generated for the same problem.
    
    *Example:* $\text{pass}@5$ represents the probability that **at least one of five generated solutions** is correct.
    
    </aside>
    

---

# 4. Trends in Large-Scale Learning

Modern LLM development is driven by a few dominant trends:

- **Extreme scaling** of training data and model parameters.
- **Minimal manual feature engineering**, with representations learned directly from data.
- **Low interpretability**, leading to “black-box” behavior.
- **Heavy computational dependence**, requiring specialized hardware and large infrastructure.

**N.B.** As models scale, engineering priorities shift from architectural changes ****to data pipelines, distributed training, and hardware optimization.

![image.png](2%20Transformer-Based%20Language%20Models/image%203.png)

<aside>
📌

**Training Cost** 

It’s quantified by multiplying the time the training runs by the number of hardware resources utilized:  $\text{Training Cost}= \text{runtime}\times \text{hardware usage}$

</aside>

---

# 5. Open-Source LLM Architectures

## 5.1. **The LLaMA Family (Large Language Model Meta AI)**

The **LLaMA (Large Language Model Meta AI)** family, developed by **Meta**, focuses on achieving high performance through **efficient design choices and high-quality data curation**, rather than relying only on massive parameter scaling.

### 5.1.1. **Architecture and Key Improvements (LLaMa 2)**

LLaMA is built on a **decoder-only Transformer architecture** and introduces several architectural enhancements that improve both **training stability, model capacity, and inference efficiency**:

- **RMSNorm (RMS Normalization)** → stabilizes training by normalizing activations in a simpler way than LayerNorm. Unlike **Layer Normalization**, which subtracts the mean and divides by the standard deviation, **RMSNorm scales activations using only the root mean square (RMS)**, without centering.
    
    <aside>
    🔢
    
    **Formulas**
    
    Given $x = (x_1, \dots, x_d)$:
    
    LayerNorm:
    
    $$
    
    \mu = \frac{1}{d} \sum_{i=1}^{d} x_i, \quad
    \sigma^2 = \frac{1}{d} \sum_{i=1}^{d} (x_i - \mu)^2 
    
    \quad
    
    \text{LayerNorm}(x_i) = \gamma \cdot \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta
    
    $$
    
    RMSNorm:
    
    $$
    
    \text{RMS}(x) = \sqrt{\frac{1}{d} \sum_{i=1}^{d} x_i^2}\quad
    
    \text{RMSNorm}(x_i) = \gamma \cdot \frac{x_i}{\text{RMS}(x) + \epsilon}
    $$
    
    </aside>
    
    **N.B.** RMSNorm works because controlling the magnitude of activations is enough for stable training. Centering is not strictly necessary, since the network can learn to handle mean shifts through its weights.
    
- **SwiGLU Activation Functions** → enhance the expressiveness of feed-forward layers, allowing the model to capture more complex patterns.
- **Rotary Positional Embeddings (RoPE)** → encode **relative positional information** by rotating token representations instead of adding positional values. Each token embedding is rotated by an angle proportional to its position:
    - position $0$ → no rotation
    - position $1$ → rotation of angle $\theta$
    - position $m$ → rotation of angle $m\theta$
    
    During the attention computation, what matters is not the absolute position, but the **difference between angles**:  $(m\cdot \theta) - (n\cdot \theta)=(m - n)\cdot \theta$.
    
    This means the model naturally captures **relative distances between tokens**, not their absolute positions. As a result, it generalizes better to long sequences, since relationships remain consistent regardless of where tokens appear.
    
- **KV Cache** → during autoregressive decoding, previously computed keys and values are stored to **avoid redundant computations**, significantly speeding up inference at the cost of higher memory usage.
- **Grouped-Query Attention (GQA)** → reduces computation and memory usage by allowing multiple **query heads** to share the same **key and value heads**. It can be seen as a compromise between **Multi-Head Attention (MHA)** and **Multi-Query Attention (MQA)**. This design significantly **reduces the size of the KV cache**, leading to faster inference and lower memory consumption.
    
    ![image.png](2%20Transformer-Based%20Language%20Models/image%204.png)
    

![image.png](2%20Transformer-Based%20Language%20Models/image%205.png)

### 5.1.2. LLaMA-2 Variant

LLaMA-2 highlights the distinction between a **base model** and an **instruction-tuned model**, which are obtained through two separate training stages:

- **LLaMA 2 (Base Model):** the foundational version, trained with a standard next-word prediction objective (cross-entropy loss) on large-scale internet data. Its purpose is simple text completion. Since the training data is raw and unfiltered, the base model may produce unsafe or unaligned outputs.
- **LLaMA 2-Chat (Instruction-Tuned Model):** built on top of the base model through an additional training phase focused on instruction following, dialogue, and safety alignment. The training process consists of two main steps:
    - **Supervised Fine-Tuning (SFT):** the model is trained on curated **prompt–response pairs**, teaching it to follow instructions and produce coherent, contextually appropriate outputs.
    - **Reinforcement Learning from Human Feedback (RLHF):** a **reward model**, trained on human preference data, guides the model to prioritize outputs that are **helpful, safe, and fluent**. During this phase, the model learns to optimize its responses according to the human-aligned reward signals.

![image.png](2%20Transformer-Based%20Language%20Models/image%206.png)

### **5.1.3. LLaMA 3 Improvements**

The latest versions, such as **LLaMA 3**, further extend these ideas with a larger and more expressive tokenizer (up to **128K tokens**), a longer context window (around **8192 tokens**), and training on extremely large datasets (up to **15 trillion tokens**). These improvements push performance beyond what is typically predicted by standard scaling laws.

## 5.2. Mistral & Qwen

- **Mistral 7B:** a **decoder-only Transformer model** with 7 billion parameters, designed for high efficiency. Regardless of its relatively small size, it achieves **strong performance comparable to much larger models**, thanks to optimized architecture and training strategies. They release 2 version, the pre-trained one and the instruct one (chat-one).
    
    ![image.png](2%20Transformer-Based%20Language%20Models/image%207.png)
    
- **Qwen Family:** a series of Transformer models developed by **Alibaba**, trained on **large-scale multilingual data** (up to 18 trillion tokens). These models support **long context windows** and are available in specialized variants, including:
    - **Code-Qwen** → optimized for programming tasks
    - **Qwen-VL** → designed for **vision-language (multimodal)** applications

---

# 6. Beam Search & Temperature

## 6.1. Beam Search

<aside>
🚨

**The Search Space Problem**

During **inference**, the model converts its internal representations into text by selecting one token at a time. If the vocabulary size is $V$ and we generate a sequence of length $n$, the number of possible sequences is $V^n$. This exponential space makes exact search infeasible, so models rely on approximate methods to select high-probability outputs.

</aside>

Rather than exploring every sequence, **Beam Search** keeps track of the best $k$ candidates after every token expansion. It is more accurate than **greedy search**, which chooses the most likely word at each step, but by looking only at the next word, it can miss better sentences where a less likely start leads to a better final result.

<aside>
🔢

**How it works?**

1. Start with an empty sequence, compute the probability distribution over the vocabulary $V$, and keep only the **top-$k$** candidate tokens.
2. Feed these $k$ partial sequences back into the model to obtain $k$ new probability distributions, each generates $V$ probabilities, creating $k \cdot V$ total paths.
3. Compute the score of each candidate by combining the probability of the new token with the cumulative probability of its sequence.
4. Rank all $k \cdot V$ candidates and keep only the **top-$k$** sequences.
    - **Pruning** → occurs when none of branch's extensions are good enough to remain in the global top-$k$ set.
    - **Splitting** → occurs when multiple extensions **from the same branch** are good enough to remain in the global top-$k$ set.
5. Repeat until an `<EOS>` token or maximum length is reached, then output the best sequence.

![image.png](2%20Transformer-Based%20Language%20Models/70423720-3630-42e7-9af8-7a72f95cefc9.png)

</aside>

## 6.2. Temperature

The **Temperature** parameter adjusts the Softmax probabilities before choosing a token, balancing accuracy and linguistic creativity:

$$
P(y_i)= \frac{e^{\frac{z_i}{T}}}{\sum_{j=1}^{V}e^{\frac{z_j}{T}}}
$$

- **Low Temperature ($T\rightarrow 0$):** sharpens the distribution, making the model more deterministic and favouring the most likely tokens, useful for tasks requiring correctness, like code generation.
- **High Temperature ($T>1$):** flattens the distribution, increasing the chance of selecting less likely tokens and producing more diverse or creative text.

---