# 9. Emergent Problems and Limitations in Multimodal LLMs

Pages: 2
Status: Done
Type: theory

# 1. Weak Visual Grounding

**Weak Visual Grounding** occurs when an MLLM relies more on its **language priors** than on actual visual evidence. This often leads to **visual hallucinations**, where the model produces fluent but incorrect image descriptions.

<aside>
📌

**Main Causes:**

- **Noisy image–text data** in training pairs.
- **Limited diversity and confirmation bias**, causing the model to favor likely answers even when visual evidence is weak.
- **Weak visual features**, where the image encoder does not extract information strong enough to influence the language model.
</aside>

## **1.1. Information Flow in MLLMs**

Studies show that visual information propagates through the model in stages:

- **Early layers** → visual tokens mainly interact with each other (or across frames in video models).
- **Middle layers** → visual information begins to influence text tokens.
- **Final layers** → visual and textual information are merged into the final token used for prediction.

![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image.png)

<aside>
📌

**Analysis Techniques**

- **Attention Knockout** → selectively removes attention between visual and text tokens. Results show that disabling visual attention in later layers has little impact, suggesting that most visual information is integrated in the middle layers.
    
    ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/d8fdd25b-43a3-4333-a2a4-6d6016926396.png)
    
- **LogitLens** → applies the output head to intermediate layers, allowing inspection of the model’s internal predictions before the final layer.
    
    ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%201.png)
    
</aside>

## 1.2. Mitigation

A common solution is to include **negative training examples**, encouraging the model to rely more on visual evidence and reducing confirmation bias.

---

# 2. Limited Fine-Grained Perception

**Limited Fine-Grained Perception** refers to the model’s difficulty in capturing fine visual details such as object counting, small text, or precise spatial locations.

<aside>
📌

**Main effects:**

- Inaccurate object counting
- Poor OCR performance on small text
- Difficulty distinguishing similar objects
- Weak spatial localization and bounding box prediction
</aside>

## 2.1. Attention Sink

A key cause of this issue is the **attention sink** phenomenon. In self-attention, the **softmax** forces attention weights to sum to $1$ for each token. When a token does not strongly attend to other tokens, it cannot distribute attention everywhere, so the model learns to direct excess attention to low-information tokens such as **`BOS`**, punctuation, or newline tokens. These become **attention sinks**.

![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%202.png)

**N.B.** Attention sinks can be identified by strong and persistent activations across many layers, often matching the activation patterns of the `BOS` token.

<aside>
📌

**Impact on tasks:**

- **Global understanding tasks** → attention sinks can help aggregate global context.
- **Fine-grained tasks** → they reduce attention on important visual details, lowering performance.
</aside>

## 2.2. Papers

- **Visual Attention Sink in Large Multimodal Models (2025):** first paper to identify this hidden property of visual attention sink (massive activation in specific dimensions). They propose to re-distribute the attention weights on non-sink tokens.
    
    ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%203.png)
    
- **Aligning What Vision-Language Models See and Perceive with Adaptive Information Flow (2026):** detects irrelevant tokens using entropy in attention maps. Tokens with unstable attention across layers are marked as distractors and masked.
    
    ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%204.png)
    
- **Learning to See What You Need: Gaze Attention for Multimodal Large Language Models (2026):**
    1. They divide the image in regions, each described by a descriptor: mean pooling of all values vector.
    2. Each generated token is matched with the most relevant region via similarity.
    3. Attention to other regions is masked.
    
    ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/39b4ed56-537a-4b53-86d6-40b85c7b3724.png)
    
- **Look Twice: Training-Free Evidence Highlighting in Multimodal Large Language Models (UNIMORE Lab, 2026):**
    
    
    1. The model performs a first pass and generates a single token to extract its attention maps.
    2. Using these attention maps, it identifies the image regions and text passages most relevant to the query.
    3. Irrelevant high-attention areas (e.g., punctuation, background regions, or attention sinks) are filtered out.
    4. The clean attention map is used to create a **bounding box** around the relevant image object and selecting the most relevant text snippets.
    5. The model processes the highlighted evidence in a second pass to generate a more accurate answer.
    
    ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/d2d326cb-361c-4f3c-8889-7e640b71bcef.png)
    

---

# 3. Lack of Up-to-Date Knowledge

**Lack of Up-to-Date Knowledge** refers to a model’s inability to answer questions about events, facts, or entities that appeared after its **training data ends.** When asked about information outside its training data, the model may produce outdated, incorrect, or hallucinated responses in an attempt to answer the query.

---

# 4. Unsafe Content Generation

**Unsafe Generations** refer to the tendency of models to produce inappropriate, violent, sexual, or hateful content. In MLLMs, this issue is amplified by **visual jailbreaks** and **document injection**, where malicious instructions are hidden inside images or screenshots.

<aside>
📌

**Main Causes:**

- **Garbage in, garbage out** → large-scale training datasets often contain toxic or unfiltered content.
- Even with filtering, there is no formal guarantee of safety.
</aside>

## 4.1. Safety Mitigation in LLMs / MLLMs (Text Modality)

Safety is enforced through multiple post-training techniques:

- **Constitutional AI** → replaces human feedback with AI feedback guided by a set of safety rules (a “constitution”). A well-known real-world example is Claude.
- **SFT and RLHF** → teach the model to follow safe behavior and refuse harmful requests.
- **DPO (Direct Preference Optimization)** → directly optimizes preference data without training a separate reward model.
- **Red Teaming** → evaluates the model using adversarial prompts:
    - Black-box attacks: ****the attacker repeatedly tries different jailbreak prompts and observes which ones succeed.
    - White-box attacks: ****the attacker uses gradient information to search for prompt suffixes that maximize the probability of unsafe outputs.
    
    Real systems often include lightweight safety classifiers (e.g., LlamaGuard-style models) that run in real time.
    
    <aside>
    📌
    
    **Model possible reactions:**
    
    - **Refusal:** the model declines to answer a harmful request.
    - **Over-refusal:** the model refuses a request that is actually safe.
    - **Under-refusal:** the model answers a request it should have refused.
    
    ---
    
    **Attack Success Rate:** measure of how often adversarial prompts successfully make the model fail.
    
    </aside>
    

## 4.2. Safety Mitigation in Diffusion Models (Vision Modality)

<aside>
💡

**Diffusion Model**

Diffusion models generate images by gradually removing noise from a random latent representation. Starting from pure Gaussian noise, the model learns to denoise the image step by step until a clear image is formed. The denoising process is usually implemented with a **U-Net** architecture. The model is conditioned by a textual input via **cross-attention**.

![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/7d5cffc1-2e82-4a6c-9492-26959b3a09b3.png)

</aside>

Safety is enforced at multiple stages:

![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/51752140-d236-4f69-9a35-25116cf1d784.png)

1. **Textual prompt:**
    - **Input filtering** (can be bypassed using prompts that avoid explicit unsafe words but keep the same meaning)
        
        ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%205.png)
        
    - **Prompt rewriting**
    - **Policy classifiers** → secondary, specialized AI models that analyze the semantic meaning and context of a prompt to detect harmful intent.
2. **Text encoder:**
    - **Safe CLIP alignment** → map unsafe embeddings toward safe ones.
    - **Inference-time steering** → if an embedding falls into an unsafe region, it is shifted toward a safe direction.
        
        ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%206.png)
        
3. **Embeddings:**
    - **Embedding sanitization** → removes or reduces unsafe semantic information in the representation. This is done by subtracting from the embedding the components that lead to unsafe concepts.
        
        ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/dab44c4a-6f42-4971-a2e2-310579437b59.png)
        
4. **Diffusion process:** (prevents unsafe visual concepts during image generation)
    - **Inference-time steering** → guides the denoising process away from unsafe concepts.
        
        ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%207.png)
        
    - **Model editing** → modifies model weights to reduce or erase unsafe concepts:
        - The model is trained with a target concept to remove (Van Gogh-style paintings) and a safe/neutral concept as reference (general paintings).
        - The loss encourages the model to move away from the unsafe concept and follow the neutral one during denoising instead of the unsafe direction.
        
        ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%208.png)
        
5. **Decoder:**
    - **Watermarking** → techniques such as *Stable Signature* embed invisible watermarks in generated images to enable traceability and provenance detection.
        
        ![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/image%209.png)
        

---

## 4.3. Focus on Unlearning

**Unlearning** is the ability to remove or forget specific information from a model without changing performance on the rest. Main approaches include:

- **Exact Unlearning** → completely removes the target data from the dataset and retrains the model from scratch.
- **Empirical Unlearning (known examples)** → fine-tuning the model using a known set of examples that need to be removed.
- **Empirical Unlearning (unknown concepts)** → removing or suppressing a concept or type of knowledge rather than specific samples.
- **Differential Privacy-based Unlearning** → during training, the algorithm adds controlled statistical noise to the optimization process. This noise allows the model to learn general patterns while preventing it from memorizing highly specific details from individual data points.

### 4.3.1. Safe-CLIP

Simply filtering unsafe data from a dataset (e.g., CLIP) and retraining the model is not enough. Even after filtering, models can still reconstruct unwanted concepts through correlated information in the data.

**Safe-CLIP** addresses this by explicitly modifying the embedding space using a 4-encoder setup (two fixed reference encoders and two trainable ones). The method balances two losses:

- **Content Redirection Loss** → forces embeddings of unsafe text to move toward safe embeddings, effectively removing the semantic distinction of harmful concepts.
- **Structure Preservation Loss** → preserves the global structure of the embedding space, preventing the model from forgetting valid concepts while unlearning unsafe ones.

![image.png](9%20Emergent%20Problems%20and%20Limitations%20in%20Multimodal%20/c9307170-b0e0-450b-9dbd-263cb5d9b0dd.png)

---