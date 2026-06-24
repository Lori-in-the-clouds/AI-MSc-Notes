# 6. Retrieval Augmentation

Pages: 3
Status: Done
Type: theory

# 1. Introduction to Retrieval

<aside>
💡

**Retrieval** is the task of finding relevant documents or information from a large collection in response to a specific query.

</aside>

To be effective, retrieval systems must be both functional and computationally efficient, especially when operating on a large scale. There are three main types of retrieval:

- **Unimodal retrieval** → operates within the same modality (e.g., `text-to-text` or `image-to-image`, as in early methods like SIFT).
- **Cross-modal retrieval** → works across different modalities, such as `image-to-text` or `text-to-image`, popularized by architectures like CLIP.
- **Multimodal/Universal retrieval** → handle complex scenarios where both the query and the external knowledge base are multimodal.

![image.png](6%20Retrieval%20Augmentation/image.png)

---

# 2. Text-Text Retrieval Models

While multimodal models like CLIP can be used for text-to-text retrieval, they are limited by a short context length (77 tokens), making them inefficient for long documents. For this reason, specialized text-to-text architectures are preferred:

- **Qwen3 Embedding** → a model specifically designed for text embedding and re-ranking. Re-ranking is the step where retrieved documents are ordered by relevance.
- **Contriever** → a model designed for dense document retrieval using unsupervised contrastive learning.
    - **Architecture:** it uses a bi-encoder Transformer architecture (specifically BERT, which allows for longer context lengths) where queries and documents are encoded independently and compared using dot product similarity.
    - **Training:** it is trained with **unsupervised contrastive learning**, which keep similar texts close together and different texts far apart, without needing labeled data.
        - **Creating Positive Pairs:**
            - Inverse Cloze Task: given a text passage, one sentence is randomly removed and used as the query, while the remaining text acts as the context; the model is trained to match the query with its original context.
            - Independent Cropping / Random Sampling: taking random segments ("views") from the same document to teach the model that the overall meaning remains consistent across different parts.
        - **Handling Negatives:**
            - Negative within a Batch: for a given query, only one document is the positive sample, while other samples in the batch act as negatives (requires large batch sizes).
            - Negative Across Batches: stores past representations from previous batches and using them as negatives in subsequent batches, enabling smaller batch sizes.

---

# 3. LLM as Retriever

Traditional retrieval systems use separate models (encoders and rerankers) for different modalities and struggle with complex multimodal queries. **LamRA** uses a single **Large Multimodal Model (LMM)** adapted with **LoRA** to act as both retriever and reranker, avoiding full fine-tuning. The system is built as a two-stage pipeline:

- **LamRA-Ret (Retrieval)** → maps text, images, or multimodal queries into a shared embedding space:
    1. The query is processed by an LMM equipped with LoRA adapters.
    2. A special `<emb>` token is added to the end of the query. Through self-attention, it acts as a collector that absorbs and compresses the meaning of the entire request into its **hidden state**. This single vector is extracted as the final embedding, while text generation is stopped.
    3. Relevant documents are retrieved using **cosine similarity**.
    4. Training uses **contrastive learning** with positive and negative pairs, followed by **instruction tuning** to adapt the model to retrieval tasks.

![image.png](6%20Retrieval%20Augmentation/image%201.png)

- **LamRA-Rank (Reranking)** → reranks the top-$k$ candidates retrieved by LamRA-Ret:
    - The LMM jointly processes the query and retrieved candidates.
    - A LoRA module predicts their relevance using:
        - **Pointwise reranking** → score each candidate independently.
        - **Listwise reranking** → evaluate multiple candidates together.
    - Training uses a **supervised ranking loss** to produce the final ranking.

![image.png](6%20Retrieval%20Augmentation/image%202.png)

---

# 4. Multimodal Retrieval Fusion Techniques

## 4.1. UniIR (Universal Information Retriever)

UniIR builds on pre-trained encoders like CLIP and BLIP (=Bootstrapping Language-Image Pre-training) to process multimodal queries. To handle missing modalities (e.g., when a query is only text but the database is multimodal), UniIR passes empty inputs, such as a black image, to bridge the gap. The final representation is obtained through two possible fusion strategies:

- **Score-level fusion** → the image and text are processed by separate encoders. The resulting vectors are merged via a weighted sum or average.
- **Feature-level fusion** → modalities are merged during the encoding phase using a merge layer (e.g., an MLP or cross-attention). This layer allows all tokens from different modalities to interact and share information, producing a single unified multimodal representation.

![image.png](6%20Retrieval%20Augmentation/cd88a22b-cf7e-4128-a484-c60e2f23acf7.png)

## 4.2. ReT (Recurrence Meets Transformers)

Standard CLIP encoders produce a single global representation at the final layer, which may lose fine-grained details. ReT addresses this by extracting features layer-by-layer instead of relying only on the last embedding.

It introduces a **recurrent mechanism** (inspired by LSTM) that operates over encoder layers rather than time steps. At each layer, a gating function combines the current visual and textual features with an internal state, deciding what information to keep or discard depending on its relevance. Early layers contribute fine details, while deeper layers provide global context.

![image.png](6%20Retrieval%20Augmentation/image%203.png)

<aside>
📌

**Fine-grained Score**

Instead of producing a single global embedding, ReT outputs a **feature matrix**, where each rows represents the different levels of information, from fine details to global context, extracted through the recurrent mechanism. We define:

- **Query matrix** → **$Q= ReT_Q(q) \in \mathbb{R}^{k\times d}$**
- **Document matrix** → $D= ReT_D(d)^T \in \mathbb{R}^{d\times k}$

where $k$ is the number of layers of information that the model extracts (typically 32 in the original formulation) and $d$ is the embedding dimension. The relevance score is computed as follows:

1. For each query vector $q_i$, compute its similarity with **all** document vectors $d_j$.
2. Keep only the **maximum similarity** for that query vector.
3. Repeat for all query vectors.
4. Sum all the best matches to obtain the final score: $\displaystyle s(Q,D)=\sum_{i=1}^{k}\max_{j=1...k} Q_i \cdot D_j$
</aside>

---

# 5. RAG (Retrieval-Augmented Generation)

<aside>
💡

**RAG** is a method that combines an external retrieval system with a Large Language Model (LLM) to generate more accurate answers. Instead of relying only on internal model parameters, the LLM uses relevant external documents retrieved at query time.

</aside>

It consists of three main steps: 

1. **Indexing** → documents are embedded and stored in an external database. 
2. **Retrieval** → the system searches for information relevant to the user query.
3. **Generation** → LLM produces an answer using both the query and the retrieved context.

<aside>
📌

**FAISS (Indexing Implementation)**

Computing exact similarities (e.g., dot product or L2 distance) over millions or billions of vectors is not scalable. **FAISS (Facebook AI Similarity Search)** is a library designed to efficiently index and search dense vectors at scale. It provides different indexing strategies depending on the trade-off between accuracy and efficiency:

- **IndexFlatL2 / IndexFlatIP:** perform exact search without compression, ensuring high accuracy but with high memory usage and slow performance on large datasets.
- **HNSW (Hierarchical Navigable Small World):** organizes vectors into hierarchical clusters, enabling fast approximate search by limiting comparisons to relevant regions, significantly reducing computation and memory cost.
</aside>

## 5.1. Why use RAG?

Instead of increasing the parameter size of a model or constantly retraining it, RAG connects the model to dynamic external knowledge. This solves several issues:

- **Knowledge staleness** → the model can access updated information without retraining.
- **Capacity constraints** → knowledge is stored externally, reducing the need for large internal parameters.
- **Domain-specific tasks** → it can handle specialized queries by retrieving relevant expert sources.
- **Hallucinations** → by using retrieved documents as support, the model is less likely to invent incorrect facts based on internal training biases, and users can check the exact source of the answer.

## 5.2. RAG Paradigms

A retrieval system typically requires a query, an external document database, an indexing mechanism and a retriever model. RAG architectures can be categorized into two main paradigms:

- **Naive RAG** → a basic approach where the user's prompt and the retrieved documents are simply concatenated and passed to a frozen LLM to generate a response.
- **Advanced RAG** → introduces optimization steps before and after retrieval to improve retrieval quality and generation accuracy:
    - **Pre-retrieval:** the goal is to refine the query to make it more useful for the retriever, such as isolating a specific subject from a long text or a large image. Common techniques include:
        - Query Rewriting: rephrasing the query to better match the vocabulary of the stored documents.
        - Query Expansion: adding related words or synonyms to reduce vocabulary mismatch.
        - Query Routing: instead of searching everywhere, it dynamically directs the query to the most suitable database.
    - **Post-retrieval:** retrieved documents are further processed through operations such as re-ranking, summarization, or fusion before being provided to the LLM.

![Screenshot 2026-05-12 at 18.38.04.png](6%20Retrieval%20Augmentation/Screenshot_2026-05-12_at_18.38.04.png)

## 5.3. How to Fuse Retrieved Knowledge with the LLM

After retrieval, the retrieved documents must be integrated with the LLM. There are three main strategies:

- **Text Concatenation** → retrieved documents are appended directly to the prompt.
    - **Pros:** simple and does not require retraining.
    - **Cons:** limited by the context window, and excessive context may introduce noise and reduce performance.
- **Cross-Attention** → retrieved documents are injected directly into the model's cross-attention layers.
    - **Pros:** handles longer contexts more effectively.
    - **Cons:** computationally expensive and requires architectural changes and retraining.
- **Output Interpolation** → the output probabilities of the language model are mathematically combined with the retriever's scores.
    - **Pros:** does not require retraining.
    - **Cons:** strongly depends on the quality of both the retriever and the LLM.

![Screenshot 2026-05-12 at 18.51.19.png](6%20Retrieval%20Augmentation/Screenshot_2026-05-12_at_18.51.19.png)

---

# 6. **RAG Architectures**

## 6.1. RETRO (Retrieval-Enhanced Transformer)

RETRO is an architecture that uses **cross-attention fusion**. Its main idea is to reach the performance of very large models by scaling the external retrieval database instead of increasing the number of model parameters.

![image.png](6%20Retrieval%20Augmentation/98a4f16a-f42a-424e-bb6c-9e326e73327f.png)

### 6.1.1. How it works?

1. **Database Encoding** → the database stores encoded chunks (**keys**) and their following text (**values**). All keys are pre-encoded with a frozen BERT model for efficient similarity search.
2. **Input Chunking** → the input sequence is divided into fixed-size chunks $C_i$, and each chunk is treated as an independent query.
3. **Neighbor Retrieval** → for each chunk $C_i$, the system uses a **frozen BERT retriever** to generate a query embedding. This embedding is compared against the database to find the **Top-K nearest neighbors**.
4. **Context Encoding** → the retrieved passages are processed through a **Trainable Bi-directional Encoder**, together with the query chunk, to generate context-aware representations.
5. **Chunked Cross Attention** → the processed chunks are injected into the decoder through cross-attention layers. To preserve autoregressive generation, chunk $C_i$ can only attend to information retrieved from previous chunks $C_{i-1}$, avoiding access to future context.

![image.png](6%20Retrieval%20Augmentation/image%204.png)

<aside>
📈

**Performance:**

RETRO achieves performance gains comparable to scaling the parametric model by roughly $10×$, and performance generally improves as the retrieval database size and the number of retrieved neighbors increase, until reaching a saturation point.

![image.png](6%20Retrieval%20Augmentation/image%205.png)

</aside>

## 6.2. RAG for Knowledge-Intensive NLP Tasks

This architecture was proposed by Lewis et al. (2020) from Meta AI to resolve **knowledge-intensive tasks**, a specific problems where a model cannot answer using only its internal knowledge.

### **6.2.1. Hybrid Memory**

The model combines two types of memory:

- **Parametric Memory** → knowledge stored in the model weights. It is difficult to update, hard to inspect and may produce hallucinations.
- **Non-Parametric Memory** → external knowledge stored in a retrieval system. It can be easily updated and allows direct inspection of the retrieved sources.

### 6.2.2. **Components**

The system follows a fine-tuning strategy for Sequence-to-Sequence (seq2seq) models using two main components:

- **Retriver $p_{\eta}(z|x)$** (non-parametric) → retrieves the top-$k$ relevant passages $z$ for a query $x$.
- **Generator $p_{\theta}(y_i|x,z,y_{1:i-1})$** (parametric) → generates the next token $y_i$ using the query $x$, retrieved passages $z$, and previous tokens.

### 6.2.3. Two Types of RAG Models

The authors studied two different ways to combine the retrieved documents with the generated sequence:

- **RAG-Sequence** → the entire output sequence is generated using the same retrieved document. It marginalizes the probability over each of the top-‭k:
    
    $$
       p_{RAG-Sequence}(y|x) \approx \sum_{z \in top-k(p(\cdot|x))} p_{\eta}(z|x) \prod_{i}^{N} p_{\theta}(y_i|x, z, y_{1:i-1})‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‭‬‭‬ ‭‬ 
    $$
    
- **RAG-Token** → the model can use a **different document to generate each individual token** in the sequence, allowing the model to combine information from multiple sources:
    
    $$
       p_{RAG-Token}(y|x) \approx \prod_{i}^{N} \sum_{z \in top-k(p(\cdot|x))} p_{\eta}(z|x) p_{\theta}(y_i|x, z, y_{1:i-1})‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬‭‬ ‭‬‭‬ ‭ 
    $$
    

**N.B.** RAG-Token usually achieves better performance.

### 6.2.4. **Implementation**

The system uses **DPR (Dense Passage Retriever)** with BERT encoders to encode queries and documents into embeddings. Similarity search is performed over a FAISS index using the **HNSW** algorithm for efficient nearest-neighbor retrieval.

1. **Query Encoding:** the query $x$ is encoded into a vector representation $q(x)$.
2. **Retrieval:** the retriever finds the most relevant passages $z_1, z_2, …$ from the indexed database.
3. **Generation:** the retrieved passages ($z$) are concatenated with the original query ($x$) and passed to a pre-trained **BART-large** generator, which produces the final response $y$.

<aside>
📌

**What is Fact Verification?**

Fact Verification is one of the main tasks of RAG models. It consists of checking whether a specific statement (claim) is true or false based on evidence taken from an external source. Usually, fact verification has three possible outcomes:

- **SUPPORTED:** the retrieved documents confirm the claim.
- **REFUTED:** the retrieved documents prove the claim is false.
- **NOT ENOUGH INFO:** the retrieved evidence is insufficient to verify the claim.
</aside>

![image.png](6%20Retrieval%20Augmentation/image%206.png)

## 6.3. REPLUG (Retrieval-Augmented Black-Box Language Models)

REPLUG is designed for situations where the language model is a **black-box API**, meaning its internal parameters cannot be accessed or fine-tuned. Instead of concatenating all retrieved documents (which can exceed the context window), REPLUG processes each document separately. 

1. The query $x$ is combined with one document at a time and sent through the LLM in multiple forward passes. 
2. The model produces one output per document, and the final prediction is obtained by averaging or combining all output probabilities.

**N.B.** This allows parallelization and only requires the context window to fit $x$ plus the largest single document.

<aside>
💡

**REPLUG-LSR:**

An extension where the retriever is trained using the LLM’s uncertainty (perplexity) as a signal, so it learns to retrieve documents that help the model make more confident predictions.

</aside>

![image.png](6%20Retrieval%20Augmentation/38462fde-37f7-43db-90c6-cbf92bafac2f.png)

## 6.4. Self-RAG (Self-Reflective RAG)

Retrieval is computationally expensive and not always necessary. While standard RAG might force the LLM to incorporate irrelevant or noisy retrieved documents into its answer, Self-RAG improves this aspect by introducing a mechanism that lets the model decide when to retrieve information and how to evaluate its own outputs. The model is trained to generate special **reflection tokens** that control the process:

| **Token Name** | **Input** | **Output**  | **Definitions** |
| --- | --- | --- | --- |
| **`Retrieve`** | Query (x), Context (y) | {Yes, No, Continue} | Decides if external knowledge is required. If "No", the model relies on its parametric memory to save compute. |
| **`IsRel`** | Query (x), Document (d) | {Relevant, Irrelevant} | Evaluates if a retrieved document d actually provides useful information to solve the query x. |
| **`IsSup`** | Query (x), Document (d), Segment (y) | {Fully, Partially, No support} | Verifies if the generated segment y is supported by the evidence found in d. |
| **`IsUse`** | Query (x), Generation (y) | {Score 1-5} | The model measures how helpful and relevant the final response y is in relation to the original prompt x. |

### 6.4.1. Inference Process

During inference, the model generates multiple candidate continuations based on different retrieved documents, then uses the reflection tokens to critique these outputs and applies a segment-level beam search to select the best one.

![image.png](6%20Retrieval%20Augmentation/image%207.png)

**N.B.** Self-RAG improves accuracy and controllability, but it requires complex training and annotated data to learn the reflection behavior correctly.

---

# 7. Retrieval for Multimodal LLMs

Standard RAG pipelines are mainly designed for text, while multimodal systems must handle multiple modalities such as images and text together. A key task in this area is **Knowledge-based Visual Question Answering (KB-VQA)**, which answers questions about an image using external knowledge.

<aside>
💡

**Comparison of Approaches:**

- **Standard MLLMs:** `Image` + `Question` → `Answer`
- **MLLMs with RAG:** `Image` + `Question` + `Retrieved Passages` → `Answer`

![image.png](6%20Retrieval%20Augmentation/image%208.png)

</aside>

## 7.1. Wiki-LLaVA

Wiki-LLaVA is an early multimodal RAG architecture designed to integrate external knowledge from Wikipedia into a multimodal LLM through a hierarchical retrieval pipeline, where retrieval is not done in a single step but instead filters through two sequential levels of granularity to efficiently handle the large-scale knowledge base.

1. **Document Retrieval (Coarse Search)** → ****the input image is encoded using CLIP, and its embedding is compared with Wikipedia page embeddings (titles + summaries) to retrieve relevant documents.
2. **Passage Retrieval (Fine Search)** → retrieved documents are split into passages. A Contriever model computes the similarity between each passage and the user question to select the most relevant ones.
3. **Generation** → the top-ranked passages are concatenated with the image and the question, then passed to the multimodal LLM to generate the final answer.

<aside>
🚨

**Limitations**

- CLIP often struggles to retrieve the correct documents in KB-VQA tasks.
- Irrelevant retrieved passages can confuse the LLM and reduce answer quality.

**N.B.** Oracle experiments show that when the correct passages are manually provided, performance improves significantly, indicating that the main limitation comes from the retrieval stage rather than the generation model.

</aside>

![image.png](6%20Retrieval%20Augmentation/05651198-45a2-40aa-b243-8841717243db.png)

## 7.2. ReflectiVA (Multimodal Self-Reflection)

ReflectiVA improves multimodal RAG by adding self-reflection and re-ranking directly inside the LLM, reducing noise from irrelevant retrieved passages:

- **Self-reflection** → given an image and a question, the model first decides whether retrieval is needed using special tokens (**`[RET]`** or **`[NORET]`**).
- **Re-ranking** (to solve noise injection) → if retrieval is used, each passage is evaluated and labeled as relevant **`[REL]`** or not relevant **`[NOREL]`**. Only relevant passages are kept for generation, reducing the impact of noisy or irrelevant information.

![image.png](6%20Retrieval%20Augmentation/44bce33d-7571-4e51-95ef-616ae8c12f0c.png)

## 7.3. ReAG: Reasoning-Augmented Generation

ReAG improves standard multimodal RAG by combining reasoning and filtering to reduce noisy retrieval and low recall:

1. **Multi-level retrieval** → starts with a coarse global view of the image and then focuses on finer, more specific visual regions.
2. **Filtering** → a dedicated critic model removes irrelevant retrieved samples before they reach the generator.
3. **Reasoning with RL** → the model is trained with reinforcement learning (GRPO) to produce explicit reasoning steps (e.g., `<THINK>` tokens) before the final `<ANSWER>`, improving interpretability and better use of retrieved knowledge.

![image.png](6%20Retrieval%20Augmentation/4db34a33-f5c2-4c63-b91f-4757b653b185.png)

---