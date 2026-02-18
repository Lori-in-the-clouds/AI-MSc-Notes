# 3-Visual Features and Cognitive Systems

Done?: Done
Select: theory

# 1. Marr’s Levels of Vision

David Marr, in his foundational book *Vision* (1980), proposed that vision can be understood at 3 conceptual levels:

- **Computational Level →** defines *what* the system does and *why*. (e.g., object recognition, spatial navigation).
- **Algorithmic/Representational Level**: describes *how* the system works, which representations are used and which algorithms process them.
- **Implementational Level**: focuses on *how* the system is physically implemented. In biology, this refers to the brain’s neural structures.

## 1.1. Marr’s Visual Processing Pipeline

Marr described vision as a sequence of stages that transform a 2D image into a 3D scene representation:

1. A **2D image** is captured, containing raw intensity values at each pixel.
2. Next, a **primal sketch** is generated, identifying basic visual features such as edges, textures, and regions.
3. Then, a **2.5D sketch** of the scene is created. Depth, surface orientation, and texture from the viewer’s perspective.
4. Finally, a **3D model** is constructed, providing a object-centered representation of the full scene.

**N.B.** These are conceptual stages, not concrete algorithmic implementations.

## 1.2. Feature Extraction Example

Feature extraction stage **is highly dependent on the goal we aim to achieve.** For each task, we can design or use different networks that focus on extracting the most useful features for that purpose. For example, if we want to know *how many objects there are*, *where they are*, and *what they are*:

1. **Low level vision:** detect edges, find borders, apply segmentation to define regions.
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image.png)
    
2. **Labeling and feature extractions**: assign labels to objects and compute descriptive features.
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%201.png)
    
3. **Camera calibration knowledge**: use camera data to estimate dimensions, relative positions, etc.
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/7a9f1f71-f44d-48c3-aa56-82f78686103c.png)
    
4. **Classification**: select the most useful features and classify objects.
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%202.png)
    

**N.B.** Nowadays, instead of designing all features manually, we often start from **foundation models** (=unsupervised models that acquire general knowledge and require fine-tuning to specialise) and fine-tune them for specific tasks.

![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%203.png)

---

# 2. Labeling

After segmentation, we obtain a **binary image** (background = 0, foreground = 1). The next step is **labelling**, whose goal is to assign different labels to distinct connected objects in the foreground.

## 2.1. Pixel Connectivity

Two pixels are considered part of the same component if they are **connected**. Connectivity is defined through **adjacency**, which depends on how distances are measured:

- **4-neighborhood** → **

$N_4(p) = \{q \in I \ | \ D_4(p,q) \leq 1\}$** → ****(only horizontal and vertical neighbors are considered)
- **8-neighborhood** → 

$N_8(p) = \{q \in I \ | \ D_8(p,q) \leq 1\}$ → (diagonal neighbors are also included)

<aside>
🚨

**Be aware:** different definitions of neighborhood produce different labeling results:

![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%204.png)

</aside>

## 2.2. Connected Component Labelling (CCL)

Connected Component Labelling (CCL) takes as input a binary image and outputs a **symbolic image**, where all pixels belonging to the same connected region share the same label. Labelling algorithms fall into 3 main families:

1. **Multiscan Algorithms** → ****repeated scans until labels stabilize.
    
    <aside>
    
    **Algorithm**
    
    1. First scan: assign a new label to each foreground pixel.
    2. Then, repeat alternating scans:
        - **Top-down scan** (top-left → bottom-right) using a mask to propagate labels.
        - **Bottom-up scan** (bottom-right → top-left) using a complementary mask.
    3. Continue until no further label changes occur.
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%205.png)
    
    **Pro**: simple concept.
    
    **Contro**: very slow due to many iterations.
    
    </aside>
    
2. **Two-Scan Algorithms** → the image is scanned twice, with an **equivalence list** maintained to handle label conflicts.
    
    <aside>
    
    **Rosenfeld (1966)**
    
    - First scan (raster order):
        - if pixel is background, skip it
        - if pixel has no neighbors, a new label is assigned to it
        - if pixel is neighbor to a labelled pixel, takes the same label
        - if pixel is neighbor to more than one pixel that have different labels, takes the same label of one pixel and equivalence list is updated (equivalence between the labels of the neighbors)
    - Second scan: resolve equivalences and update labels accordingly.
    
    **Pro**: efficient, well-suited for parallel implementation.
    
    **Contro**: high memory usage (label image + equivalence list) and sorting overhead.
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%206.png)
    
    </aside>
    
    <aside>
    
    **Haralick (1981)**
    
    It is an optimization of Rosenfeld, in particular to use less memory. It doesn’t use any equivalences table, but iteratively performes forward and backward scan passes to solve the equivalences exploiting only local neighborhood
    
    **Pro**: less memory usage.
    **Contro**: higher computational cost.
    
    </aside>
    
    <aside>
    
    **Samet (1986) – Union-Find**
    
    Equivalences are resolved immediately using a **Union-Find** data structure:
    
    - $\text{find}(x)$ → returns the root of label $x$.
    - $\text{union}(x,y)$ → merges two label sets.
    
    **Pro**: avoids costly sorting step, more efficient.
    
    </aside>
    
    <aside>
    
    **He (2008)**, **Grana (2010)**
    
    Introduced optimizations using decision trees (*He*) or decision tables (*Grana*) to reduce neighborhood checks and merges.
    
    </aside>
    
3. **Label Propagation Algorithms** 
    1. Scan in raster order.
    2. If pixel is background or already labelled → skip.
    3. If pixel is unlabelled foreground → assign a new label and **propagate** it recursively (or iteratively) to its neighbors.
    
    **Pro**: requires no extra memory for equivalence tables.
    
    **Contro**: inherently sequential, not suitable for parallelization.
    

---

# 3. Feature Extraction

Feature extraction transforms an image into **n-dimensional vectors** that describe meaningful visual properties. This step reduces raw pixels into a compact representation useful for analysis.

This is essential for:

- Reduces computational cost.
- Avoids overfitting (generalization).
- Provides input for recognition, retrieval, and tracking task.

## 3.1. Approaches

- **Bottom-up** → start from pixels → extract edges/colors → build up to objects detection and classification.
- **Top-down** → start from task → choose which features are needed.

Both produce **feature vectors** that compress information (e.g., turning a 224×224 image into a smaller vector).

## 3.2. Designing Robust Features

According to the **Gestalt principles** of perception, our brain organizes visual information using:

- **Invariant properties**: robust under transformations like scaling, rotation, reflection, etc.
- **Subjective relevance**: robust against lighting changes, occlusions, or background distractions.

So, **a good visual feature** should satisfy:

- **Perceptual similarity**: be invariant and relevant to what we perceive as meaningful.
- **Computational stability**:
    1. **Discriminant** → features must be significantly different for objects belonging to different classes.
    2. **Reliable** → they must remain similar for objects belonging to the same class.
    3. **Independent** → values in the vector should not be redundant (If we have two values with the same information, one can be discarded)
    4. **Compact** → as short as possible (minimum cardinality)
- **Continuity**: maintain consistency across space and time (e.g., in videos or sequences).

## 3.3. Types of Features

- **Handcrafted features** → designed manually for specific tasks. Examples:
    - Color histograms
    - Texture descriptors (e.g., entropy)
    - Histograms of gradients (HOG)
- **Generic perceptual features** → inspired by human vision (e.g., keypoints, saliency maps)
- **Learned features** → extracted by neural networks (CNNs, Transformers).

<aside>
🔑

**Color histrogram**

One of the most widely used feature representations is the **color histogram**, which captures the **distribution of colors** within an image.

**Advantages**: it’s faster to compute compared to other invariants.
**Disadvantages**: It depends only on colors, ignoring the shape and texture of the image.

---

**Spatial color histogram**

To preserve some **spatial structure**, an image can be divided into blocks, and a histogram is computed per block. This is known as a **spatial color histogram**.

![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%207.png)

</aside>

---

# 4. Distance Functions for Feature Similarity

To compare features, we use a **distance function**. A distance function $d: S \times S \to \mathbb{R}$ is a **metric** if it satisfies:

1. **Self-identity**: $d(x, x) = 0$
2. **Positivity**: $d(x, y) \ge 0$
3. **Symmetry**: $d(x, y) = d(y, x)$
4. **Triangle inequality**: $d(x, z) \le d(x, y) + d(y, z)$

<aside>
📌

**Variants:**

- **Semi-metric** → no triangle inequality.
- **Pseudo-metric** → no positivity.
- **Non-metric** → breaks more than one rule.
</aside>

## 4.1. Common Metrics

- **Minkowski distance $L_p$ →** relevant in truly Euclidean spaces, especially $L_1$ and $L_2$:  $d(x, y) = \left(\sum_i |x_i - y_i|^p\right)^{1/p}$
    
    <aside>
    🚨
    
    **Common values of** $p$
    
    It’s a valid metrics only for $p≥1$ (otherwise, the triangle inequality fails). There can be different variants:
    
    - $p=1$ → **Manhattan Distance** (visualized by the red and yellow paths on the grid).
    - $p=2$ → **Euclidean Distance** (visualized by the blue diagonal line).
    - $p=\infty$ → $d=max_i​∣x_i​−y_i​∣$  → **Chessboard Distance** (represents the minimum moves for a king on a chessboard).
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%208.png)
    
    </aside>
    
- **Mahalanobis Distance (Quadratic)** → It is applicable when features are correlated, meaning the covariance matrix satisfies:
    - $w_{ij} = w_{ji}$ (symmetry)
    - $w_{ii} = 1$ (typically standardised features)
    
    It can be computed as: 
    
    $d(x, y) =\sqrt{(x - y)^T S^{-1}(x - y)}$ where $S$ is the covariance matrix. 
    
    Using Euclidean distance can be misleading: two points may appear equally distant from the cluster center, but one may lie in a region where data is naturally more spread out:
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%209.png)
    
    Mahalanobis distance instead measures distance **relative to the data distribution:**
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%2010.png)
    
- **Weighted Mean Variance** → $D(A,B)=\frac{|\mu_A-\mu_B|}{|\sigma(\mu_{distribution})|}+ \frac{|\sigma_A-\sigma_B|}{|\sigma(\sigma_{distribution})|}$ where:
    - $\sigma(\mu_{distribution})$: standard deviation of the **means** calculated across *all* images in the entire dataset.
    - $\sigma(\sigma_{distribution})$: standard deviation of the **standard deviations** calculated across *all* images in the entire dataset.
- **Cosine Distance** → measures the cosine of the angle between two vectors $x$ and $y$:  

$d(x, y) = 1 - \frac{x \cdot y}{\|x\| \|y\|}$ (often used neural features).
- **Correlation Distance** → measures the **linear dependence** between two feature vectors $A$ and $B$, quantifying how strongly they vary together regardless of their absolute magnitude:
    
    $$
    correlation(A,B)=\frac{cov(A,B)}{\sigma_A\cdot \sigma_B}
    $$
    
    Where:
    
    - $cov(A,B)=\frac1n\sum_{i=1}^n(A_i-\mu_A)\cdot(B_i-\mu_B)$
    - $\sigma_X=\sqrt{\frac1n\sum_{i=1}^n(x_i-\mu_X)^2}$
- **Kullback-Leibler Divergence (KL)** → considers histograms as distributions and measures their similarity by calculating the relative entropy: 

$D_{KL}(P \| Q) = \sum_i P(i) \log \frac{P(i)}{Q(i)}$.
- **Histogram Intersection** → compares each element of the histogram and takes the minimum: $d_\cap (H,K)=1-\frac{\sum_i min(h_i,k_i)}{\sum_ik_i}$
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%2011.png)
    
- **Earth Movers Distance →** quantifies the effort required to transform one distribution (e.g., histogram) into another. It’s based on the idea of computing the **minimum “*work*”** needed to shift “*mass*” from one configuration to match the other. It’s especially useful when comparing distributions that may be similar but slightly shifted. Given 2 ordered histograms $A$  and $B$:
    
    $$
    EMD(A,B)=\frac{\sum_{i=1}^n|F_i|}{\sum_{i=1}^na_i}
    $$
    
    Where:
    
    - $F_i = \sum_{j=1}^i(a_j-b_j) \cdot 1$ → represents the **cumulative amount of mass** that must be moved up to bin $i$, and the factor $1$ is the **distance between consecutive bins** (since the bins are ordered). So, $|F_i|$ corresponds to the **work** required to shift the histogram mass up to that point.
    - $\sum_{i=1}^n a_i$ → normalizes the total work by the **total amount of earth (=mass) to be moved**, making the metric scale-invariant.
    
    ![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%2012.png)
    

---

# 5. Before Deep Learning: Handcrafted Features

Before the rise of deep learning, **feature vectors were manually engineered.** One widely used method was **HOG (Histogram of Oriented Gradients)**:

1. We compute the gradient orientation for each pixel in the image
2. We divide the image in a series of patches
3. For each patches, a histogram is created to record the distribution of the gradient orientation

**N.B.** The final features vector describe the object’s shape. 

HOG helped transition from raw pixel representations to mid-level features, anticipating the role of feature learning in deep neural networks.

![image.png](3-Visual%20Features%20and%20Cognitive%20Systems/image%2013.png)

---

# 6. Retrieval

Given a **query image** (an image, a video frame, or a feature description), retrieval finds the most similar items from a database. When the query is an image, the process is called **Content-Based Image Retrieval (CBIR)**.

## 6.1. Retrieval Algorithm: Nearest Neighbor Strategy

1. Extract a feature vector from each item in the dataset.
2. Extract a feature vector from the query item.
3. Compare the query vector with all dataset vectors using a distance function.
4. Select the $N$ **items** with the smallest distance values (most similar).

---