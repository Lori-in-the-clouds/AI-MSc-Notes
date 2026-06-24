# 4. Large Multimodal Models

Pages: 3
Status: Done
Type: theory

# 1. Introduction

**Multimodality** refers to models capable of processing and mapping data across multiple modalities (e.g., visual, text, audio). The input, the output, or both can be multimodal. Key vision-language tasks include:

- **Captioning:** `Image → Text`
- **Retrieval:** `Image ↔ Text` (finding similar images or retrieving a caption that describes an image)
- **Visual Question Answering (VQA):** `Image + Text → Text`
- **Generation:** `Text → Image`
- **Multimodal classification:** `image + text → label`
- **Better understanding/generation:** `image + text → label/text`

<aside>
💡

**Foundation Models**

Instead of training a specialized model for every single task, the modern paradigm relies on **Foundation Models**. These are massive models pre-trained on vast amounts of data, which are then fine-tuned to act as the foundation for multiple downstream tasks.

</aside>

---

# 2. **CLIP (Contrastive Language–Image Pre-training)**

<aside>
💡

CLIP is a **vision–language foundation model** that learns to align images and text in a shared embedding space, enabling tasks like **zero-shot classification** without task-specific training.

</aside>

## 2.1. Architecture

CLIP uses a **dual-encoder** setup:

- An **image encoder** (ResNet or ViT) that maps images into vectors
- A **text encoder** (12-layer Transformer with causal mask, with a context limit of ~77 tokens) that maps sentences into vectors

**N.B.** Each encoder maps inputs into a common feature space.

## 2.2. Contrastive Pre-Training

CLIP uses a **contrastive learning** approach, which brings matching image-text pairs closer together and pushes non-matching pairs apart. Given a batch of $N$ text–image pairs:

1. All $N$ images are passed through the **Image Encoder** to produce a set of $N$ image feature vectors: $[I_1, I_2, . . . , I_N]$.
2. All $N$ text descriptions are passed through the **Text Encoder** to produce a set of $N$ text feature vectors: $[T_1, T_2, . . . , T_N]$.
3. A similarity score (dot product or cosine similarity) is computed between every image vector and every text vector (producing a square similarity matrix of size $N\times N$). 
4. The model is trained to:
    - **maximize similarity** for matching pairs (diagonal)
    - **minimize similarity** for mismatched pairs (off-diagonal)
    
    **N.B.** This is done using a **symmetric cross-entropy loss**.
    

![image.png](4%20Large%20Multimodal%20Models/image.png)

<aside>
📌

**Symmetric Cross-Entropy Loss**

The **symmetric cross-entropy loss** minimizes prediction error **in both directions** simultaneously:

1. **Image-to-Text Loss:**
    - Compute the **softmax across each row** of the similarity matrix. This gives the probability that a given image corresponds to each text.
    - Apply che **cross-entropy**: $\displaystyle Loss_{\text{Image}\rightarrow\text{Text}}=-\sum_iy_i\log(p(y_i))=-\frac{1}{N}\sum_{i=1}^N\log\left(\frac{e^{(\frac{T_i\cdot I_i}{\tau})}}{\sum_{j=1}^N e^{(\frac{I_i\cdot T_j}{\tau})}}\right)$
2. **Text-to-Image Loss:**
    - Compute the **softmax across each column** of the similarity matrix. This gives the probability that a given text corresponds to each image.
    - Apply **cross-entropy**: $\displaystyle Loss_{\text{Text}\rightarrow \text{Image}}=-\sum_iy_i\log(p(y_i))=-\frac{1}{N}\sum_{i=1}^N\log\left(\frac{e^{(\frac{T_i\cdot I_i}{\tau})}}{\sum_{j=1}^N e^{(\frac{I_j\cdot T_i}{\tau})}}\right)$
3. **Total Loss:** the total loss is the average of the two directions:
    
    $$
    Loss = \frac{Loss_{\text{Image}\rightarrow\text{Text}}+Loss_{\text{Text}\rightarrow\text{Image}}}{2}=-\frac{1}{2N}\sum_{i=1}^N\left(
    
    \log\left(
    \frac{e^{(\frac{I_i\cdot T_i}{\tau})}}{
    \sum_{j=1}^N e^{\frac{I_i\cdot T_j}{\tau}}}
    \right)+
    \log\left(
    \frac{e^{(\frac{I_i\cdot T_i}{\tau})}}{
    \sum_{j=1}^N e^{\frac{I_j\cdot T_i}{\tau}}}
    \right)
    
     \right)
    $$
    

**N.B.** Contrastive loss requires a large number of **negative pairs** to work effectively. Large batch sizes (e.g., 32,768) provide enough negatives.

</aside>

## 2.3. Zero-Shot Capability

Because CLIP is not trained on a closed set of classes but on the general alignment between text and images, it can perform **zero-shot classification**, ****this means that CLIP can perform classification on **categories never seen before** without retraining or fine-tuning.

1. Convert text labels into text prompts (e.g., *“a photo of a dog”*)
2. Compare the image embedding with all text embeddings
3. Select the most similar one

![image.png](4%20Large%20Multimodal%20Models/image%201.png)

## 2.4. Dataset and Performance

- **Dataset** → the success of models like CLIP depends heavily on large-scale, high-quality data. A major breakthrough came with **LAION-5B**, an open-source dataset of 5 billion filtered image-text pairs. Earlier internet-scraped data was noisy, so LAION-5B provided much cleaner, curated pairs. Over time, smaller subsets with slightly longer captions were also released, further improving quality.
- **Performance** → it defines the "Foundation Model" paradigm by delivering zero-shot performance that matches or exceeds traditional models trained on 16 labeled examples per class, leveraging massive multimodal pre-training to bypass the need for task-specific data.
    
    ![image.png](4%20Large%20Multimodal%20Models/image%202.png)
    

<aside>
🚨

**Limitations**

One limitation of CLIP is its **text encoder**, which has a **maximum context length of 77 tokens**. Captions shorter than 77 tokens are padded, but longer captions are truncated, which constrains the amount of text information the model can process.

</aside>

---

# 3. CLIP Variants and Improvements

Given CLIP's success, several variants have been developed to address its computational bottlenecks and improve its scale:

- **SigLIP** → a key limitation of standard **CLIP** is its **softmax-based contrastive loss**, which requires extremely large batch sizes (e.g., 32,768) to provide enough negative pairs. This results in a huge $N \times N$ similarity matrix and high memory/computation requirements. **SigLIP** addresses this by reformulating the task as a **binary classification problem** using a **sigmoid loss**:
    
    $$
    L_{\text{SigLIP}} = -\frac{1}{|\mathcal{B}|} \sum_{i=1}^{|\mathcal{B}|} \sum_{j=1}^{|\mathcal{B}|} \log \left[ \sigma\big(z_{ij} (-t \, I_i \cdot T_j + b)\big) \right]
    =
     -\frac{1}{|\mathcal{B}|} \sum_{i=1}^{|\mathcal{B}|} \sum_{j=1}^{|\mathcal{B}|} \log \left[ \frac{1}{1 + e^{\,z_{ij}(-t \, I_i \cdot T_j + b)}} \right]
    $$
    
    Where:
    
    - $z_{ij}$ → is the label for a given image and text input, $z_{ij} = 1$ for matching pairs, $-1$ otherwise
    - $\sigma(\cdot)$ → is the sigmoid function
    - $|\mathcal{B}|$ → is the batch size
    - $b$ → a bias term used to handle the extreme class imbalance between positive and negative pairs. For example, in a batch of $N = 32,768$, there is typically only **1 positive pair** and **32,767 negative pairs**. Without this bias, a similarity score of 0 would correspond to a probability of **50%**, causing the model to overestimate matches. By setting b to a large negative value (e.g., $-10$), the sigmoid output is shifted so that pairs are considered **non-matching by default**, and only very high similarity scores are classified as matches.
        
        ![image.png](4%20Large%20Multimodal%20Models/c68f1051-92b4-40f9-93fd-4a56522e9b49.png)
        
    - $t$ → is the temperature, acting as a “scaling factor” for similarity. Multiplying the similarity by $t$ adjusts the steepness of the sigmoid:
        - a high $t$ amplifies differences, making the model more confident
        - a low $t$ flattens the curve, increasing uncertainty
    
    <aside>
    📌
    
    **Why SigLIP is more efficient?**
    
    SigLIP is more memory-efficient than standard CLIP because it removes the **global normalization term**:
    
    $$
    \displaystyle Loss_{\text{Image}\rightarrow\text{Text}}=-\sum_iy_i\log(p(y_i))=-\frac{1}{N}\sum_{i=1}^N\log\left(\frac{e^{(\frac{T_i\cdot I_i}{\tau})}}{
    \textcolor{orange}{
    \sum_{j=1}^N e^{(\frac{I_i\cdot T_j}{\tau})}}}\right)
    $$
    
    The key improvement is that SigLIP applies the sigmoid independently to each image–text pair. Unlike CLIP’s softmax, which places the sum inside the logarithm, SigLIP moves the sum outside, making the total loss simply the sum of many small, independent losses. This allows each pair to be processed individually, greatly reducing memory usage and enabling simpler batching.
    
    </aside>
    
- **EVA** →  an improved and scaled-up version of CLIP, reaching around **1B parameters** by leveraging larger and higher-quality datasets. It adopts **Masked Image Modeling (MIM)**,  a **self-supervised learning** **method** (similar to BERT) where the model learns to understand an image by trying to predict the missing parts of it.

---

# 4. Generative Multimodal Models

<aside>
🔑

**Evolution:**

- **Early Multimodal Architectures  (e.g. CLIP):**
    - They are **encoder-only and discriminative**: they learn a shared embedding space to measure similarity between images and text, without any ability to generate content.
    - These models use **late fusion**, encoding each modality independently and combining them only at the final stage. This design is highly efficient for retrieval tasks (e.g., text-to-image search), as image features can be precomputed and reused during inference.
- **Modern architectures (e.g., LMMs):** they can be either **encoder–decoder** or **decoder-only**, and can both understand multimodal inputs and generate text, enabling tasks such as image captioning, visual question answering, and multimodal dialogue.
</aside>

## 4.1. **SimVLM (2021)**

SimVLM is one of the first fully generative multimodal models, marking the transition from discriminative approaches (e.g., CLIP) to **image-to-text generation**.

### **4.1.1. Architecture**

It is an **encoder–decoder Transformer** that jointly processes image patches (from a convolutional stage) and text tokens using **early fusion**. The model is trained **end-to-end from scratch** on large-scale image–text pairs with a **cross-entropy loss** for autoregressive text generation, without relying on pre-trained encoders.

It demonstrated that a single model can effectively combine visual understanding and text generation, representing a key step toward modern multimodal generative models.

**N.B.** SimVLM generates coherent image descriptions but is not designed for dialogue. 

![image.png](4%20Large%20Multimodal%20Models/a6e6e529-6e61-4739-9ad7-42d19aef73cf.png)

## 4.2. **CoCa (Contrastive Captioner, 2022)**

CoCa is a multimodal architecture that combines **discriminative** (CLIP-like) and **generative** capabilities in a single model. It can measure image–text similarity and generate captions at the same time.

### 4.2.1. Architecture

- **Image Encoder (ViT):** extracts visual features from the image.
    
    <aside>
    📌
    
    **Attention Pooling**
    
    It’s a operator that collapses a variable number of image patch features into a fixed number of output vectors. This is done via a **multi-head attention** mechanism where:
    
    - A set of **learnable query vectors** acts as the queries.
    - The image patch features serve as the **keys and values**.
    
    The output is a single, fixed-dimensional vector that summarizes the visual content for contrastive alignment.
    
    </aside>
    
    In particular, attention pooling is applied twice to produce two different outputs:
    
    - **Contrastive output ($n_{\text{query}}=1$)** → uses a single learnable query to produce a **global image embedding** (similar to a CLS token).
    - **Generative output ($n_{\text{query}}=256$)** → uses 256 learnable queries to produce a set of visual tokens, which are fed into the multimodal decoder as **cross-attention inputs**.
- **Unimodal Text Decoder:** processes text independently and outputs a **CLS token** for contrastive learning.
- **Multimodal Text Decoder:** generates text using **cross-attention** over visual features.

![image.png](4%20Large%20Multimodal%20Models/01b5e1c9-14e8-46fe-9bbb-19030e4e924a.png)

### 4.2.2. Training

The model is trained jointly with two losses:

- **Contrastive Loss** → applied between the image representation and the unimodal text CLS token to align image–text pairs (maximize similarity for correct pairs, minimize it for incorrect ones).
- **Captioning Loss** → applied to the multimodal decoder to learn how to generate coherent text by predicting the next token:
    
    $$
    \mathcal{L}_{\text{Cap}} = -\sum_{t=1}^{T} \log P_{\theta}(y_t | y_{<t}, x)
    $$
    
    Where: 
    
    - **$x$** → the image features
    - $y_t$ → the target token at time $t$
    - $y_{<t}$ → all previous tokens (context)

**N.B.** Joint training improves performance compared to training the tasks separately. If the generative part is removed, the model behaves similarly to CLIP.

### 4.2.3. Pseudo-Code

![image.png](4%20Large%20Multimodal%20Models/image%203.png)

## 4.3. FLAVA (2022 – Case Study in Limited Adoption)

FLAVA is a multimodal model trained on **three types of data**:

- images alone
- text alone
- image–text pairs

**N.B.** It adopts a **late fusion-oriented approach**, which improves scalability. Separate image and text embeddings can be precomputed, making retrieval tasks (e.g., image search) more scalable. Pure early fusion would require recomputing the entire network for every image-text pair, which is computationally expensive.

### 4.3.1. Architecture

FLAVA uses a **hybrid fusion** approach, features are first encoded independently and then fused at an intermediate stage, combining ideas from both CLIP and SimVLM:

- **Image Encoder (ViT)** → extracts visual features from the image.
- **Text Encoder** → extracts textual features from the input text.
- **Multimodal Encoder** → ****fuses the visual and textual representations.

<aside>
🚨

**Limited Adoption**

Despite its innovative approach, training all components together proved less practical. The industry shifted toward **pre-training unimodal encoders separately** and then integrating them, achieving similar benefits more efficiently.

</aside>

![image.png](4%20Large%20Multimodal%20Models/9a1ef3ed-3388-429d-bf7c-a0fdd014f5b8.png)

<aside>
📌

**Losses**

FLAVA is trained with two groups of losses:

1. **Unimodal Losses (before fusion):**
    - **Masked Image Modeling  $L_{MIM}$** → applied to the **image encoder**. The model masks some image patches and learns to reconstruct them.
    - **Masked Language Modeling $L_{MLM}$** → applied to the **text encoder**. The model masks some words and predicts them.
    - **Global Contrastive Loss  $L_{GC}$** → ****aligns image and text representations at a **global level**. It compares the `[CLS]` token from the image encoder with the `[CLS]` token from the text encoder, encouraging matching pairs to be similar and non-matching pairs to be different.
2. **Multimodal Losses (after fusion):**
    - **Multimodal Masked Modeling $L_{MMM}$** → applied to the **multimodal encoder**. The model reconstructs missing parts of one modality using information from the other, learning fine-grained cross-modal relationships.
        
        *Example:* ****If the “*head of a cat*” is masked in the image, but the text says *“A cat is jumping”*, the model uses the text to infer what is missing in the image.
        
    - **Image–Text Matching $L_{ITM}$** → a **binary classification loss**. Given an image–text pair, the model predicts whether they match (1) or not (0), using the multimodal `[CLS]` representation.
</aside>

## 4.4. Frozen

The Frozen approach shows that a **language model can remain completely fixed (frozen)** while only training a vision encoder. Visual features are transformed and **concatenated to the text embeddings**, so the model treats them like words. This supports the Platonic Representation Hypothesis.

<aside>
📌

**Platonic Representation Hypothesis**

Large models trained on different modalities (vision and text) be likely to learn **compatible latent spaces**, which can be aligned with a simple projection (e.g., a linear layer or small MLP).

</aside>

### **4.4.1. Architecture**

- **Frozen LLM:** weights remain unchanged
- **Trainable vision encoder:** extracts visual features. It learns to map images into a space **compatible with the LLM’s text embeddings**, allowing the frozen LLM to understand visual inputs.

These visual features are then projected and concatenated with text embeddings, allowing the LLM to process them as if they were tokens.

![image.png](4%20Large%20Multimodal%20Models/image%204.png)

## 4.5. Flamingo

Flamingo is one of the first MLLMs to robustly support multimodal dialogue with images and **video**. Its main components are:

- **Vision Encoder** → a pre-trained visual feature extractor.
- **Perceiver Resampler** → converts variable-length visual inputs into a fixed number of visual tokens. It uses multi-layer attention, where visual features are keys and values, and a set of learnable query vectors extracts a compact representation.
- **Frozen LLM** → a large pre-trained language model kept frozen, which receives visual and textual inputs through specially inserted cross-attention modules.

![image.png](4%20Large%20Multimodal%20Models/image%205.png)

<aside>
📌

**Key Mechanisms:**

- **Gated Cross-Attention:** integrates visual information into the LLM while preserving its pre-trained linguistic knowledge. Cross-attention layers are inserted between self-attention blocks and controlled by **learnable tanh gates initialized at zero**, which gradually open during training to smoothly incorporate visual features.
- **Interleaving Images and Text:** enables few-shot prompting by mixing visual and textual examples in the input.
</aside>

## 4.6. BLIP-2

BLIP-2 is an efficient multimodal model that connects vision and language using a **two-stage pre-training strategy**, while keeping both the **image encoder and the LLM frozen**. It shows that you don’t have to retrain the whole “brain” (the LLM) to understand images, you just need a ****efficient “translator” that provides the right informations.

### **4.6.1. Architecture**

- **Frozen Image Encoder →** a pre-trained visual model (e.g., ViT) that extracts image features.
- **Q-Former** → a trainable module that connects vision and language. It uses learnable query vectors that look at image features (as keys and values) and compress them into a fixed-size representation with the most important visual information. This output is then combined with text embeddings and sent to the LLM.
- **Frozen LLM** → a pre-trained language model kept unchanged.

![image.png](4%20Large%20Multimodal%20Models/image%206.png)

### 4.6.2. Training

The  Q-Former is trained using a two-stage pre-training strategy:

1. **Teaches the Q-Former what to see** → the **LLM is not involved**, the goal is to force the Q-Former to learn how to extract visual feature that are highly relevant to the text. The Q-Former is trained with three objectives:
    
    
    - **Image-Text Matching (ITM)** → a binary classification task that predicts where an image and a text match, capturing fine-grained details.
    - **Image-Grounded Text Generation (ITG)** → trains the model to generate text based on visual features.
    - **Image-Text Contrastive (ITC)** → aligns image and text representations by bringing matching pairs closer and pushing mismatched ones apart.
    
    ![image.png](4%20Large%20Multimodal%20Models/image%207.png)
    
    <aside>
    📌
    
    **Self-Attention Mask Strategies**
    
    Different attention masks are used to support each objective:
    
    - **Bi-directional [for ITM]:** image queries and text tokens can fully see to each other.
    - **Causal multimodal [for ITG]:** text tokens can see visual queries and to previous text tokens, but not to future text tokens. This allows the model to generate text autoregressively while using image information.
    - **Uni-modal [for ITC]:** image and text are kept separate, forcing alignment only at the representation level.
    
    ![image.png](4%20Large%20Multimodal%20Models/image%208.png)
    
    </aside>
    
2. **Teaches the Q-Former how to speak to the LLM** → the goal is to align the output of the Q-Former with the input space of the LLM.

---

# **5. Instruction-Tuned Multimodal Models**

Instruction tuning teaches models to follow human commands, enabling strong zero-shot performance.

## **5.1. LLaVA (Large Language-and-Vision Assistant)**

LLaVA connects a **frozen CLIP ViT-L/14 vision encoder** with a **frozen LLM (e.g., Vicuna)** using a simple projection layer. To overcome the lack of human-annotated "instruction-image" pairs, it uses **GPT-4** to generate synthetic data from datasets like COCO.

### 5.1.1. **Architecture**

- **Frozen LLM** → pre-trained language model
- **Frozen Vision Encoder** → CLIP ViT-L/14
- **MLP Projection** → maps visual features to the LLM’s embedding space and concatenates them with text inputs

<aside>
📌

**Multimodal Input Pipeline**

1. **Visual Feature Extraction** → an image $X_v$ is processed by a pre-trained vision encoder:
    
    $$
    Z_v = \text{VisionEncoder}(X_v)
    $$
    
    where $Z_v$ is a set of patch feature vectors in the visual space, with dimension $D_v$.
    
2. **Projection (MLP Adapter)** → since the LLM operates in a different embedding space $D_q$, the visual features are projected using a learnable mapping $W$ (MLP):
    
    $$
    
    H_v = W \cdot Z_v
    $$
    
    The resulting $H_v$ has the same dimensionality as the LLM’s word embeddings and is treated as **visual tokens**.
    
3. **Multimodal Concatenation** → given a text input $X_q$, its embeddings $H_q$ are computed and concatenated with visual tokens: $X_{\text{input}} = [H_v, H_q]$

![image.png](4%20Large%20Multimodal%20Models/image%209.png)

</aside>

<aside>
📌

**Training Strategy**

1. **Projection pre-training:** train only the MLP (adapter) on image-caption pairs to align modalities.
2. **Fine-tuning (Instruction following):** the MLP adapter and the LLM are trained on image–dialogue data. This helps the model follow human instructions and perform more complex multimodal reasoning.

![image.png](4%20Large%20Multimodal%20Models/cdaf0395-a36b-42ea-829a-4bd898510954.png)

</aside>

<aside>
🚨

**Limitations:**

- Input length grows with image resolution, increasing cost
- Requires fixed image resolution
</aside>

---

---

# **4.7. Recap: Evolution of Multimodal Architectures**

The development of Large Multimodal Models (LMMs) shows a progression from aligning images and text to reasoning and generating content.

**1. Discriminative Alignment (Late Fusion)**

- **CLIP:** Dual encoder maps images and text into a shared space using contrastive loss. Excellent for retrieval but cannot generate text.
- **SigLIP:** Improves CLIP by replacing softmax with a sigmoid loss, making training more memory-efficient and enabling smaller batch sizes.

**2. Generative Approaches (Early & Hybrid Fusion)**

- **SimVLM:** Introduces encoder-decoder generative paradigm with early fusion, trained from scratch to predict text from images.
- **CoCa:** Combines contrastive loss (like CLIP) and captioning loss in a single model using attention pooling.
- **FLAVA:** Uses a triple-encoder setup (image, text, multimodal) and hybrid fusion, training on both single modalities and image-text pairs simultaneously.

**3. Frozen Model Paradigm (Bridges and Connectors)**

- **Frozen:** Demonstrates that a simple linear projection suffices for mapping visual tokens into a frozen LLM’s space (Platonic Representation Hypothesis).
- **Flamingo:** Adds Gated Cross-Attention with tanh gates and the Perceiver Resampler to integrate images and video into the LLM.
- **BLIP-2:** Introduces the Q-Former, a lightweight module that extracts only essential visual features, acting as an efficient bridge to the LLM.

**4. Instruction Tuning and Adaptivity**

- **LLaVA:** Applies instruction tuning using a simple MLP to connect a frozen LLM with a visual encoder, enabling complex visual reasoning.
- **Qwen-VL:** Introduces dynamic resolution, allowing the model to handle images of any size without losing spatial detail.

![image.png](4%20Large%20Multimodal%20Models/image%2010.png)

**1. SimVLM (2022)** – Early fusion model enabling image-to-text generation, unifying visual understanding and text generation without pre-trained encoders.

**2. CoCa (2022)** – Combines discriminative (CLIP-like) and generative (captioning) capabilities; uses attention pooling to compress variable visual features into fixed representations.

**3. FLAVA (2022)** – Trains on images, text, and image-text pairs simultaneously; employs late fusion for scalable retrieval tasks.

**4. Frozen (2021)** – Keeps LLM fully frozen, training only a vision encoder; demonstrates Platonic Representation Hypothesis: different modalities learn compatible latent spaces aligned via simple projection.

**5. Flamingo (2022)** – Enables few-shot multimodal prompting with interleaved image-text examples; uses Gated Cross-Attention with tanh gates to inject visual features without disrupting frozen LLM knowledge.

---