# 4-Cognitive systems: keypoints, attention and saliency

Done?: Done
Select: theory

# 1. Understanding Memory and Vision

## 1.1. Kandel and the Physiology of Memory

Eric Kandel, awarded the Nobel Prize in 2000, studied **memory formation at the molecular level** using the sea slug *Aplysia californica*. This simpler organism, with ~20,000 neurons, allowed him to observe how learning modifies neuronal behavior.

<aside>
💡

**Two Types of Memory:**

Kandel highlighted two types of memories:

- **Explicit Memory Storage** → ****complex memory forms requiring the hip-pocampus.
- **Implicit Memory Storage** → simple memory forms, like driving, which become automatic.

He demonstrated that learning modifies the strength of communication between nerve cells in synapses, allowing both short-term and long-term memories to emerge through alterations in gene expression and synaptic growth.

</aside>

## 1.2. Relevance to AI

Kandel’s insights into learning and memory have implications for developing AI, including:

- Understanding brain mechanisms may improve **model explainability**.
- Simpler architectures can lead to **more interpretable systems**.
- Synaptic-like dynamics could support **adaptive, plastic AI**.
- Insights from neuroscience (e.g., filters, attention) can inform neural networks.

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/c481f4c8-a7bd-47ca-b4f7-569f7c00e33b.png)

## 1.3. Vision and Neuroscience

Kandel emphasised that **human vision is complex and distributed**. Vision is not processed by the retina alone but involves multiple stages:

1. **Photoreceptors** → eyes capture the scene using **rods** (night vision) and **cones** (color sensitivity).
2. **Retinal Preprocessing** → the retina processes images, enhances contrast, and performs noise cleaning
3. **Optic Nerve (Thalamus)** → transmits signals to the brain.
4. **Visual Cortex (V1)** → analyses edges, motion, orientation, and color.

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image.png)

<aside>
📌

**Specialized cells:**

- **Magnocellular (M) cells:** detect motion and spatial location.
- **Parvocellular (P) cells:** detect fine texture and color.
</aside>

## 1.4. The Dual Nature of Vision and the Optical Inverse Problem

Vision is difficult due to the **Optical Inverse Problem**, which seeks to determine the characteristics of a subject based on observed optical properties, such as light scattering or reflection patterns. The problem is **ill-posed →** many different 3D scenes can produce the same 2D image.

To resolve this ambiguity, the brain combines sensory input with **prior knowledge** of the world, this is why “*vision is considered a creative process”*.

Vision seen as two different processes that continuously interact with each other:

- **Bottom-up processing (data-driven)** → is based on raw sensory input. It automatically extracts low-level features such as edges, corners, and borders, allowing us to quickly detect shapes, faces, and perspective. These processes are universal and occur even in infants, but they provide only a partial understanding of the scene.
- **Top-down processing (model-driven)** → relies on prior knowledge, memory, and attention. It helps us resolve ambiguities, generate hypotheses, and interpret complex scenes. However, it can also mislead us, as our expectations sometimes shape perception and create optical illusions.

---

# 2. Template Matching Before Detection

## 2.1. What is a template?

A **template** is a small fixed image or a part of an image representing a known pattern. It is typically **semantically neutral** (just raw pixel values). Detection of templates in real-world scenarios is challenging due to variations in color, size, perspective, rotation, etc.

<aside>
📌

**Detection vs Matching**

- **Detection**: identifying or locating a **known shape or object** within an image.
- **Matching**: finding **patterns or structures** in the data, which **may not be previously known**.
</aside>

## 2.2. Template Matching

**Template matching** is a technique to locate a specific template within a larger image by sliding it across the image and computing the similarity at each position. The goal is to find the region where the template best matches the image.

There are 3 types of template matching:

- **Exact Template Matching** → pixel-by-pixel comparison between the template and the image region. This approach is simple and effective in highly controlled environments such as industrial inspection or medical imaging, but it is extremely sensitive to any small variation in appearance, such as lighting or scale.
- **Local Template Matching** → focuses on searching for a specific pattern within a restricted area of the image. It’s sensitive to changes in the appearance of that pattern.
- **Block Matching →** commonly used in video processing, compares larger image regions (blocks) to estimate motion or correspondence between frames. It may sacrifice finer detail for computational efficiency and broader context.

<aside>
🚨

**Limitations of Template Matching** 

- Fails with rotation, scale, deformation, or perspective changes.
- Assumes **rigid templates** and consistent illumination.
</aside>

## 2.3. Similarity Measures

Let $t(i,j)$ be the template and $I(x,y)$ the input image:

- **Mean Square Error (MSE)** → measures squared difference between pixels:  $\displaystyle D_{i,j}(x,y) = \sqrt{\sum_i \sum_j [I(i+x,j+y)-t(i,j)]^2}$
    - Suitable for grayscale/color images, but still lighting-sensitive.
- **Mean Absolute Difference (MAD) →** simpler and faster alternative to MSE:  $\displaystyle MAD_{i,j}(x,y)=\frac1N\sum_i \sum_j |I(i+x,j+y)-t(i,j)|$
    - More efficient (fewer operations) → for a $8\times 8$ block, MSE requires 64 products and 192 sums, while MAD only requires 128 sums, making it computationally advantageous.
- **Cross-Correlation** → we start from the MSE formula and make some approximations:
    1. We remove the square root and we expand the square: 
        
        $D(x, y) = {\sum_i \sum_j (I(x+i, y+j) - t(i,j))^2}$
        
        $D(x, y) = {\sum_i \sum_j (I(x+i, y+j)^2 -2 \cdot I(x+i, y+j) \cdot  t(i,j) + t(i,j)^2}$
        
    2. The square of the template is constant for every sample therefore we can rule it out:
        
        $D(x, y) = {\sum_i \sum_j (I(x+i, y+j)^2 -2 \cdot I(x+i, y+j) \cdot  t(i,j)}$
        
    3. Also if we assume that the average value of intensity over the image is pretty much constant, we can say that $I(x+i, y+i)^2$ is constant as well, and we can remove it. This approximation is often not true, but it’s useful:
        
        $D(x, y) = {\sum_i \sum_j -2 \cdot I(x+i, y+j) \cdot  t(i,j)}$
        
    4. We can take out $-2$, thus we are no longer try to minimize a distance, now we are trying to **maximize** the remaining value:
        
        $$
        D(x, y) = {\sum_i \sum_j I(x+i, y+j) \cdot  t(i,j)}
        $$
        
        This formula corresponds to cross-correlation.
        
- **Normalized Cross-Correlation (NCC)** → it’s defined as cross-correlation normalized by the mean values of both the image patch and the template. It is defined as: $\displaystyle NCC(x,y)=\frac{\sum_j \sum_i I(i+x,j+y)\cdot t(i,j)}{\sqrt{\sum_j \sum_i I(i+x,j+y)^2}\cdot \sqrt{\sum_j \sum_i t(i,j)^2}}$
    
    NCC returns a similarity score between $-1$ and $1$, making it robust to illumination changes. A value near $1$ indicates a strong match.When both the template and the patch are treated as vectors, NCC is equivalent to **cosine similarity**:  
    
    $$
    \text{NCC} = \cos(\theta) = \frac{a\cdot b}{|a||b|}=\frac{\sum_i a_i b_i}{\sqrt{\sum_i a_i^2} \cdot \sqrt{\sum_i b_i^2}}
    $$
    
    To further improve robustness, matching is done between image and template **normalized by their average gray levels**, making the comparison invariant to average gray level:
    
    ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%201.png)
    
    So we obtain that: $\displaystyle NCC(f,g)=C_{f,g}(\hat{f},\hat{g})=\sum_{[i,j] \in \R}\hat{f}(i,j)\hat{g}(i,j)$
    

---

# 3. Keypoints, Detectors and Descriptors

<aside>
💡

**Keypoint**

It’s a distinctive image point that stands out visually and remains stable under transformations such as illumination changes, noise, or affine transformations. There are two possible types of keypoints:

- **Perceptual keypoints:** are local, visually distinctive points, such as corners, detected directly from pixels. They can be either hand-crafted, like the Harris detector, or learned through unsupervised or self-supervised methods.
- **Landmark keypoints**, on the other hand, have a specific semantic meaning, like the corners of the eyes, the mouth, or body joints. They are usually detected using supervised learning.

---

**Detectors**

A detector is an algorithm or process whose goal is to find specific keypoints os saliency areas in an image.

- Keypoints detectors
- Saliency detectors

---

**Descriptors**

**Descriptors** are numerical representations (vectors) of image content. It could be a:

- **Keypoints** (local features)
- **Saliency regions** (areas of interest)
- **Object ROIs** (object descriptors)
- **Entire images** (global descriptors) → can be handcrafted (e.g., SIFT, HOG) or deep learned (mostly).
</aside>

## 3.1. Steps to Handle Local Features

1. **Detect** significant points or regions in the image
2. **Describe** a patch around the point with a vector of features (e.g. using a patch of pixel values, gradients, etc.).

## 3.2. Harris Keypoint Detector

The **Harris detector** is designed to find *“good”* keypoints. The most **distinctive** points are the ones where changes occurs in **all directions**. 

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%202.png)

### 3.2.1. Theoretic Algorithm

1. Take a pixel $q(x,y)$ and a surrounding window  $\Omega(q)$.
2. Shift $\Omega(q)$ by a small vector $\delta = (u,v)$.
3. Compare the new region $\Omega'(q)$ with $\Omega(q)$.
4. The change/difference between $\Omega'(q)$ and $\Omega(q)$ is called information content $D_q(d)$.
5. Find the points $q(x,y)$ with maximum information content in every possible direction.

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%203.png)

<aside>
📌

**Information Content formula** 

- The change after the shift can be computed as a sum of square differences between the original window and the shifted one, for every $r$ pixel $\in \Omega(q)$:
    
    $$
    D_q(d) = \sum_{r \in \Omega(q)} (I(r+d)-I(r))^2
    $$
    
- Now we can approximate the difference using the first order Taylor expansion:
    
    $$
    I(r+d)\approx I(r) + d^T\nabla I(r) \Longrightarrow I(r+d) -  I(r) \approx d^T\nabla I(r)
    $$
    
- Now we can substitute in the original SSD formula to get the standard information content formula:
    
    $$
    D_q(d) = \sum_{r \in \Omega(q)}\left[d^T\nabla I(r)\right]^2
    $$
    
    An equivalent formulation is: $D_q(d) = \sum_{x,y} w(x,y)\cdot \left[d^T\nabla I(r)\right]^2$ where $w(x,y)$ is a window function that is used to select pixels $(x,y)$ and can have different shapes:
    
    ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%204.png)
    
</aside>

### 3.2.2. From theoretic to practical algorithm

We need a criterion that is independent from the direction $\vec d$:

- We take the information content second formulation and we expand it:
    
    $$
    
    D_q(d) = \sum_{x,y} w(x,y)\cdot \left[d^T\nabla I(r)\right]^2  = \sum_{x,y} w(x,y)\left[(u,v)\cdot(I_x, I_y)\right]^2  = \sum_{x,y} w(x,y)\left[u\cdot I_x + v \cdot I_y\right]^2 = \sum_{x,y} w(x,y) \begin{pmatrix} u & v \end{pmatrix}\begin{bmatrix}I_x I_x & I_x I_y \\I_x I_y & I_y I_y\end{bmatrix}\begin{pmatrix} u \\ v \end{pmatrix} \ = [u, v] ~M~\begin{bmatrix} u\\v\end{bmatrix}
    $$
    
    Where: 
    $M = \sum_{x,y} w(x,y) \begin{bmatrix}I_x I_x & I_x I_y \\I_x I_y & I_y I_y\end{bmatrix} = \begin{bmatrix}\sum_{x,y} I_x I_x & \sum_{x,y}I_x I_y \\\sum_{x,y}I_x I_y & \sum_{x,y}I_y I_y\end{bmatrix}$
    
    <aside>
    🚨
    
    **Where did $w(x,y)$ go?**
    
    To keep things simple we assume that: 
    $w(x,y) = \begin{cases} 1, 	\text{~~if~~pixel~~is~~selected} \\ 0, \text{~~else}\end{cases}$
    
    </aside>
    
- $M$ is independent from $(u,v)$, which is what we need. Since $M$ is symmetric positive semi-definite the spectral theorem tells us that we can diagonalize it, with eigenvalues $\lambda_1$, $\lambda_2$ representing intensity variation in orthogonal directions:
    
    $$
    M = V \begin{bmatrix} \lambda_{max} & 0 \\0 & \lambda_{min} \end{bmatrix} V^T
    $$
    
    Where $V = [v_{max}, v_{min}]$ is the matrix of the eigenvectors of $M$.
    
    - If $\lambda_1 \gg \lambda_2$ or $\lambda_1 \ll \lambda_2$ → **Edge**
    - If $\lambda_1 \approx \lambda_2$ and both large → **Corner**
    - If both small → **Flat region**
    
    ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%205.png)
    
    So, to find keypoints we only need the eigenvalues of $M$, which is independet from $d=(u,v)$, therefore we don’t need to compute $D_q(d)$ for every possible direction. In particular we can compute the **corner response function**:
    
    $$
    R = det(M)-k\cdot(trace(M))^2 = \lambda_{max}\cdot\lambda_{min} - k(\lambda_{max}+\lambda_{min})^2
    $$
    
    Where:
    
    - $R > 0$ → **Corner**
    - $R < 0$ → **Edge**
    - $R\ \text{is small}$ **→ Flat region**
    
    ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%206.png)
    

<aside>
🔢

**Practical Algorithm**

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%207.png)

</aside>

<aside>
📌

**Problem**

Harris detector is:

- Invariant to rotation
- Invariant to illumination
- Not invariant to scale changes

To overcome this limitation, more advanced descriptors are needed (**SIFT**).

</aside>

## 3.2. SIFT (Scale Invariant Feature Transform)

SIFT, introduced in 2004 by David Lowe, is a powerful algorithm that detects and describes distinctive keypoints in images. It is designed to be invariant to: **Scale**, **Rotation** and **Illumination**. 

SIFT has 2 stages:

- **Detection of keypoints:**
    1. Create the scale-space → to achieve **scale invariance**, we create levels of images where each level contains images of the same size blurred with Gaussians with increasing $\sigma$: $L(x, y, \sigma) = G(x, y, \sigma) * I(x, y)\quad \text{with}\quad 
    
    G(x, y, \sigma) = \frac{1}{2\pi\sigma^2} e^{-\frac{x^2 + y^2}{2\sigma^2}}$
        
        ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%208.png)
        
    2. Compute Difference of Gaussians (DoG) → for each scale-level we subtract consecutive Gaussian-blurred images: $D(x, y, \sigma) =
    (G(x, y, k\sigma) - G(x, y, \sigma)) * I(x, y)
    = L(x, y, k\sigma) - L(x, y, \sigma)$
        
        ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%209.png)
        
        **N.B.** DoG approximates the **Laplacian of Gaussian (LoG)**, highlighting strong intensity changes.
        
    3. Detect local extrema
        
        
        1. For each pixel $p$, compare it with **8 neighbors in the same scale**.
        2. If $p$ is a local extremum, compare it with **9 neighbors above + 9 below** → **26 neighbors total**.
        3. Pixels greater or smaller than all neighbors are **keypoint candidates**.
        
        ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%2010.png)
        
    4. Filter out candidate keypoints with low contrast
    5. Filter out candidate keypoints if they are on edges → an edge is characterized by:
        - **High curvature in one direction** (across the edge)
        - **Low curvature in the perpendicular direction** (along the edge)
        
        To distinguish edges from corners, we analyze the **second-order derivatives** of the image using the **Hessian matrix** with DoG values:
        
        $$
        H = \begin{bmatrix}
        D_{xx} & D_{xy} \\
        D_{xy} & D_{yy}
        \end{bmatrix}
        $$
        
        We evaluate:
        
        - $Tr(H) = I_{xx}+I_{yy} = \lambda_1 + \lambda_2$
        - $Det(H) = I_{xx} \cdot I_{yy} - (I_{xy}^2) = \lambda_1 \cdot \lambda_2$
        
        where $\lambda_1$ and  $\lambda_2$ are eigenvalues. We know that a region is an edge when one eigenvalue is much larger than the other. Thus when $\frac{Tr(H)}{Det(H)}$ is high. We want to filter out edges, for these reason we look for points with low $\frac{Tr(H)}{Det(H)}$.
        
- **Description of keypoints:**
    1. **Assign orientation** → to make the keypoints **rotation invariant**, we assign an orientation to each one. For each keypoints:
        1. Compute gradients in a local window:  $L_x=L(x+1,y)-L(x-1,y) \quad L_y=L(x,y+1)-L(x,y-1)$
        2. Compute gradient magnitude: $m(x,y) = \sqrt{L_x^2 + L_y^2}$
        3. Create a **36-bin histogram** (10° per bin) of gradient directions. Each pixel contribution is weighted using a Gaussian filter centered on the keypoint. This weighting gives more importance to pixels near the keypoint center and makes the orientation assignment more robust to small localization errors.
        4. The **peak of the histogram** defines the **canonical orientation**. Additional orientations (within 80% of the peak) → create **duplicate keypoints** at the same location.
    2. **Generate descriptor**
        
        
        - Take a **16×16 pixel window** around the keypoint.
        - Divide it into **4×4 subregions**.
        - For each subregion, compute an **8-bin histogram** of gradient orientations.
        - Concatenate all histograms → **128-dimensional vector** (4 × 4 × 8).
        
        ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/9fde42d1-a1e3-400b-8bc3-605c7740fc2a.png)
        

## 3.5. Saliency and Visual Attention

<aside>
💡

**Saliency** is an attentional mechanism that consists in identifying the most relevant parts of the visual scene. Saliency mechanism is useful for learning/understanding.

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%2011.png)

Given an image, we can produce:

- **Saliency map** → that represents relevant regions basing on low level visual features such brightness, colour, motion,..
- **Priority map** → that represents relevant regions basing on low level visual features + goal directed (task) information
</aside>

### 3.5.1. Saliency use in Computer Vision

Studying saliency may be useful in computer vision for:

- **Model explainability** → we can track important regions with respect to CNN behavior in order to understand what the model is evaluating.
- **To understand fixation movements** (the small, involuntary movements of the eyes that occur even when an individual is trying to maintain a steady gaze at a target).

### 3.5.2. Attention Mechanisms

- **Pre-selection**: attention is spread across the scene in a **graded way**, assigning different levels of importance to different regions → Graded attention representation.
- **Post-selection**: the system selects a **specific target** to focus on → Attentional target representation.
- **Overt Attention**: refers to **actual eye movement** toward the target. This is the **physical shift of gaze** to focus on what has been selected.

### 3.5.3. Itti and Koch Saliency Model (Optional)

It’s a foundational bottom-up model for saliency detection developed by Laurent Itti and Christof Koch. The process follows several key steps:

1. **Input Image** → the system starts with a visual scene as the input for analysis.
2. **Multiscale Feature Extraction** → the image is decomposed into several **low-level feature maps** at different spatial scales:
    - **Color channels** are separated into **Red-Green** and **Blue-Yellow** opponency.
    - **Intensity** (luminance) is processed through On/Off channels, mimicking retinal processing.
    - **Orientation** information is extracted using **Gabor filters** at multiple angles (0°, 45°, 90°, and 135°), detecting edges and contours.
    - Optionally, other features such as **motion**, **depth**, or **shape** can also be integrated.
3. **Center-Surround Mechanism** → inspired by the receptive fields of neurons in the visual system, this step involves comparing **fine-scale (center)** and **coarse-scale (surround)** representations. This emphasizes local contrasts, for example, a bright object on a dark background, and generates **conspicuity maps** for each feature channel.
4. **Normalization and Integration** → the conspicuity maps are normalized to balance their influence (ensuring no single feature dominates), and then **summed together** to form a final **saliency map**. This map highlights the most visually prominent regions in the image.
5. **Winner-Take-All (WTA) Selection →** a **WTA neural network** is used to simulate the selection of the most salient location, the region with the highest response on the saliency map. This mechanism models how attention “jumps” to the most conspicuous item in the scene.
6. **Inhibition of Return →** after a region has been attended to, it is **temporarily suppressed** to prevent attention from returning there immediately. This allows attention to **shift dynamically** across the scene, simulating exploratory behavior.
    
    ![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%2012.png)
    
7. **Top-down Modulation (Extension)** → while the original model was purely bottom-up, later extensions introduced **top-down influences**, allowing cognitive factors (like goals or tasks) to modulate the feature weights and bias the saliency map toward goal-relevant information.

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%2013.png)

---

### 3.5.4. Deep Learning-Based Saliency (Optional)

Recent deep learning models have significantly improved the prediction of visual saliency by learning from large annotated datasets and by incorporating both **spatial** and **temporal** information. Two notable architectures are **SAM** and **DR(eye)VE**, each designed for different domains.

<aside>
📌

**SAM (Saliency Attentive Model)**

SAM focuses on predicting **human eye fixations** in **static images**. It combines classical attention mechanisms with deep learning components to model how humans visually explore scenes. Its architecture includes:

- **Feature Extraction** → the model utilizes a Dilated Convolutional Network (based on VGG-16 or ResNet-50) to extract **high-resolution spatial features** from the input image while preserving context.
- **Attentive ConvLSTM Module** → iteratively refines the attention map over several steps, mimicking the way human attention adjusts and refocuses.
- **Learning the Center Bias with Gaussian Priors** → a set of learned spatial priors, modeled as multiple 2D Gaussian maps, is used to capture the center bias typically observed in human visual attention, that is, the natural tendency for viewers to focus near the center of an image. The parameters of these Gaussians, namely the means $(\mu_x, \mu_y)$ and standard deviations $(\sigma_x, \sigma_y)$, are learned directly from data, allowing the model to flexibly adapt to the statistical distribution of human gaze patterns.
    
    $$
    \displaystyle f(x,y)=\frac{1}{2\pi\sigma_x\sigma_y}\cdot e^{-\frac{(x-\mu_x)^2}{2\sigma_x^2}-\frac{(y-\mu_y)^2}{2\sigma_y^2}}
    $$
    
- **Final Saliency Map Prediction** → final predictions are produced by combining the attention-refined features and the spatial priors through additional **convolutional and upsampling layers**.
- **Training and Loss Function** → the model is trained using a composite loss function, combining three standard metrics to ensure robust prediction quality:
    
    $$
    L(\tilde{y},y_{den},y_{fix})=\alpha L_1(\tilde{y},y_{fix})+\beta L_2(\tilde{y},y_{den})+ \gamma L_3(\tilde{y},y_{den})
    $$
    
    where $\tilde{y}$ is the predicted saliency map, $y_{den}$ is the ground truth density map, and $y_{fix}$ is the ground truth fixation map. This combination of NSS, Linear Correlation Coefficient (CC), and Kullback-Leibler Divergence (KL-Div) loss functions ensures a comprehensive evaluation and optimization of the model.
    
</aside>

<aside>
📌

**DR(eye)VE: Driver Attention Prediction**

The DR(eye)VE project focuses on predicting driver attention in real-world driving scenarios. This model uses a multi-path deep learning architecture designed to predict where a driver is likely to look in different driving contexts.

Key components of the architecture include:

- **Multiple input modalities**: the model takes as input RGB video frames (for appearance), **optical flow** (for motion detection), and **semantic segmentation** maps (for understanding the scene layout and object categories).
- A **3D ConvNet backbone** (specifically, the C3D network) that captures **spatiotemporal features** from sequences of frames (typically 16-frame clips), learning how attention evolves over time.
- A **two-stream training strategy**:
    - One stream uses **random spatial crops** to force the model to learn **local details**.
    - The other processes **resized full-frame inputs** to retain **global scene context**.
- A **refinement module** sharpens the prediction, while a **fusion layer** combines the outputs from all branches into a final **probabilistic fixation map**, which predicts the driver’s gaze distribution for the last frame in the clip.

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%2014.png)

![image.png](4-Cognitive%20systems%20keypoints,%20attention%20and%20salie/image%2015.png)

</aside>

### 3.5.5. Model Comparison: SAM vs DR(eye)VE (Optional)

| Aspect | SAM | DR(eye)VE |
| --- | --- | --- |
| **Domain** | General-purpose saliency prediction. Predicts human eye fixations on static images across diverse scenes | Designed specifically for driver visual attention prediction in real-world driving scenarios (videos) |
| **Input** | Single RGB image | RGB Video + Optical Flow + Semantics |
| **Temporal Processing** | Uses **Attentive ConvLSTM** to refine attention maps over iterations. Models recurrence and focus, but not true temporal motion | Uses **3D Convolutional Networks (C3D)** to explicitly model **spatio-temporal dynamics** over video clips (e.g., 16 frames) |
| **Architecture** | **Single-stream** CNN | **Multi-stream** (3 branches) network, each processing a different modality (RGB, motion, semantics), fused at the final stage |
| **Center Bias** | Learned Gaussian Priors | Two-stream data strategy: one stream uses full images, the other random crops to encourage learning both global and local cues
 |
| **Training** | Supervised on human fixation datasets, trained with multiple losses: **NSS, CC, KL divergence** to reflect gaze behavior | Trained using **KL-divergence** loss on real driving data (video sequences and corresponding fixation maps from real drivers |
| **Output** | **Static saliency map**: probability distribution of eye fixations on an image | **Dynamic fixation map**: predicts where the driver is likely to look in the **last frame** of a video clip |
| **Use Case Examples** | Image understanding, gaze modeling, visual attention in general vision systems, explainable AI | Driver monitoring systems, ADAS (Advanced Driver-Assistance Systems), autonomous vehicles, and attention-aware in-vehicle interfaces |

---