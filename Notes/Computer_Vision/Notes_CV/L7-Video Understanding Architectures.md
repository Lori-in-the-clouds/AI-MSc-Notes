# L7-Video Understanding Architectures

Done?: Done
Select: lab

# 1. Introduction

<aside>
💡

**Video**

A video can be seen as a 3D signal with three coordinates $(x, y, t)$, where $(x, y)$ represent the spatial dimensions and $t$ is the temporal dimension. If we fix $t = t_0$ we obtain an image

</aside>

Video understanding presents several challenges: 

- Videos are much larger in size compared to images
- Video has often lower quality, and their quality can vary across frames due to issues like resolution changes, motion blur, or occlusions
- The network requires a huge amount of data to train

---

# 2. Traditional Approaches

Before deep learning, video understanding relied mainly on modeling motion. There are two common representations:

- **Optical Flow representation** → motion is measured at a **fixed pixel location** across consecutive frames. For example, at point $p_1$ we compute displacement vectors:
    - $d_\tau(p_1)$ → how $p_1$ moves from frame $\tau$ to frame $\tau+1$
    - $d_{\tau+1}(p_1)$ → how $p_1$ moves from frame $\tau+1$ to frame $\tau +2$
    - $d_{\tau+2}(p_1)$ → how $p_1$ moves from frame $\tau+2$ to frame $\tau+3$
    
    This representation captures how each pixel moves over time.
    

![image.png](L7-Video%20Understanding%20Architectures/image.png)

- **Trajectory Stacking Representation** → instead of looking at a fixed pixel location you follow the actual trajectory of the point across frames. At each step we compute displacement at the new location:
    - $d_\tau(p_1)$ → motion at $p_1$
    - $d_{\tau+1}(p_2)$ → motion at $p_2$ which is the new location of $p_1$
    - $d_{\tau+2}(p_3)$ = motion at $p_3$ which is the new location of $p_2$
    
    This captures motion along the **path of the moving object** rather than at a static location.
    

![image.png](L7-Video%20Understanding%20Architectures/image%201.png)

---

# 3. Deep Learning Approaches

## 3.1. Using CNN

- **Single Frame Models** → each frame is passed independently through a CNN, and then a **combination head** merges the outputs to produce the final prediction.
    
    **Disadvantages:** this method ignores the temporal order of the frames.
    
- **Multiple Frame models** → try to incorporate temporal information in the network using a process called **fusion.** Fusion is a general term that means “merging features”, there are many ways to perform it:
    - Concatenation
    - Element-wise sum
    - Element-wise max
    - Element-wise mean
    - Learnable Fusion $\rightarrow$ small neural network
    
    Not only there are many ways to perform fusion, there are also many places where you can perform it:
    
    - **Late Fusion** → process frames independently and merge them at the end, where fusion happens at the semantic level (e.g., “person,” “bat,” “grass” features). This approach can also combine information extracted at different scales, such as high and low resolution.
    - **Early Fusion** → combine raw inputs at the start, for example by stacking frames together: instead of feeding 3 frames with 3 channels each, feed a single frame with 9 channels. This allows the CNN to capture temporal relations directly at the pixel level.
    - **Slow Fusion** → merge features gradually across network layers, balancing early and late fusion.
    
    ![image.png](L7-Video%20Understanding%20Architectures/image%202.png)
    

<aside>
🚨

**Limitations of CNN-based Fusion**

- Models only consider a finite **window $L$** of frames.
- Temporal relations inside the window are captured, but **long-term dependencies are lost**.
- Increasing $L$ drastically increases parameters, making the network heavy and inefficient.
</aside>

## 3.2. 2D CNN + RNN

Recurrent Neural Networks are really good at processing sequences because they can remember long-term dependencies.

**Limitation:** RNNs are inherently **sequential** and cannot be parallelized. Even if we parallelize CNNs and use late fusion, the final LSTM must still process sequentially, making training slow and less GPU-friendly.

![image.png](L7-Video%20Understanding%20Architectures/image%203.png)

## 3.3. Two-Stream 2D CNNs

The **two-stream architecture** combines two complementary pipelines:

- **Spatial stream:** focuses on appearance (single RGB frame as input).
- **Temporal stream:** focuses on motion (optical flow).

In the temporal stream, each input has 3 channels corresponding to the optical flow from the **previous, current, and next frame**, stacked together. This parallel design lets the model capture both **spatial** and **temporal** information.

![image.png](L7-Video%20Understanding%20Architectures/image%204.png)

## 3.4. 3D CNNs (State-of-the-art) (C3D)

3D CNNs extend the convolution operation into the temporal dimension, capturing both spatial and temporal features. It’s based on 3D Convolution and 3D max-pooling. The video that need to be processed needs to be divided into clips (usually 16 frames).

### 3.4.1. Inflated 3D CNN (I3D)

**Idea**: adapt powerful 2D CNNs for video by “inflating” their filters into 3D.

**Steps**:

1. Start with a strong pre-trained 2D CNN (e.g., VGG, ResNet).
2. Convert each $k \times k$ filter into $k \times k \times k$ by replicating it across the temporal axis and normalizing.

**N.B.** This works well because the model begins with robust spatial representations (edges, textures, shapes) and extends them into the temporal domain.

## 3.5. Two Streams Inflated 3D CNN

![image.png](L7-Video%20Understanding%20Architectures/image%205.png)

## 3.6. Pseudo 3D CNN

The idea is to use spatial convolution + temporal convolution separately. It’s generally used in the inception module:

![image.png](L7-Video%20Understanding%20Architectures/image%206.png)

## 3.7. Non Local Networks

Overcome the limitation of convolution (which is a **local operator**) to capture relationships between distant pixels or frames. It applies **Self-Attention** (like Transformers). It computes the response at a position as a weighted sum of features at **all positions** in space and time. 

**Advantage:** Directly captures **long-range dependencies** immediately, without needing deep stacks of convolutional layers.

---