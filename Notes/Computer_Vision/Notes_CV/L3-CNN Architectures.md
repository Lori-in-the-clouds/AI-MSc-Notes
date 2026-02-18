# L3-CNN Architectures

Done?: Done
Select: lab

# 1. Introduction

<aside>
🔑

**Motivations for CNNs**

We want a neural network capable of processing images:

1. In image processing, it does not make sense for every neuron to be connected to every pixel. A neuron that processes a pixel in the upper part of an image is probably modeling something very different from a neuron that processes a pixel at the bottom. To address this, we introduce the concept of a **receptive field,** the portion of the input that a neuron is allowed to “see.”
2. Image inputs are typically very large, and in a fully connected network the number of parameters grows extremely quickly as we go deeper. To solve this problem, we need an operator that can **gradually reduce the dimensionality** of the input.
</aside>

CNNs solve both of these problems by applying **convolution**.

# 2. Convolution (2D)

The core operation involves sliding a kernel across the input tensor. At each position, the element-wise multiplication between the kernel and the corresponding input patch is performed, and all resulting products are summed up to produce a scalar. For a 2D input $I$ and kernel $K$, the convolution is defined as:

$$
(I * K)[x,y]=\sum_{(i,j) \in kernel} I[x-i, y-j]\cdot K[i,j]
$$

In deep learning convolution is equivalent to **cross-correlation** (operation preferred in CV):

$$

(I * K)[x,y]=\sum_{(i,j) \in kernel} I[x+i, y+j]\cdot K[i,j]
$$

![image.png](L3-CNN%20Architectures/image.png)

<aside>
💡

**Are convolution and cross-correlation equivalent?**

They are mathematically equivalent if the kernel is **real and symmetric**, because cross-correlation is equal to convolution with a flipped kernel.

**Why are they equivalent in deep learning?**

In deep learning, the kernels are **learned**. If we swap convolution and cross-correlation, the network can simply learn the flipped version of the kernel.

**Why is cross-correlation preferred?**

Cross-correlation avoids the extra flipping operation required in convolution, making it slightly faster to compute.

</aside>

## 2.1. Input and Output Shapes (of a 2D Convolution)

- **2D shaped input:**
    - Input shape → $(H,W)$
    - Kernel shape → $(kH,kW)$
    - Output shape → $(H – (kH-1), W – (kW – 1))$

- **3D shaped input:**
    - Input shape → $(iC,H,W)$
    - Kernel shape → $(iC,kH,kW)$
    - Output shape → $(1, H-(kH-1), W-(kW-1))$
    
    **N.B.** This is still 2D Convolution, because the kernel still moves over 2 axes!
    

<aside>
🚨

**Beware**: in 3D, the kernel must have the same number of **channels** as the input.

</aside>

---

# 3. CNN Layers

## 3.1. Convolutional Layer

The convolutional layer receives $iC$ input feature maps and applies $oC$ distinct convolution filters (or kernels) to produce $oC$ output feature maps.

### 3.1.1. Parameters of the Convolutional Layer

- **Stride $(S)$ →** how many pixels the kernel shifts each step. Larger stride results in smaller output (downsampling). Stride can have different values on the $x$ and $y$ axes.
- **Padding** $(P)$ → number of extra pixels added around the input. A typical choice is $k-1$ (where $k$ is the kernel size), so that the output shape matches the input shape on spatial axes. By default is $0$ unless specified.
- **Dilation** → expands the kernel by inserting zeros between its elements, increasing the receptive field without adding parameters. Given a dilation factor $l$, it is defined as follows:
    
    $$
    (I * K)[x,y]=\sum_{(i,j) \in kernel} I[x+l\cdot i, y+l\cdot j]\cdot K[i,j]
    $$
    
    In the figure below, red dots are the application points of increasingly dilated kernels at each layer, green areas represent the exponentially increasing receptive field of the sequence of convolutions.
    
    ![image.png](L3-CNN%20Architectures/image%201.png)
    
- **Grouping** → input channels can be split into groups, reducing parameters and computation while performing convolutions in parallel.
    
    <aside>
    
    Example:
    
    - $\text{groups} = 1$ → Equivalent to standard convolution: all input channels are connected to all filters.
    - $\text{groups} = 2$ → The input channels are split into 2 groups, and each group is processed by half of the filters. This is **like having two separate convolutional layers running in parallel** (the output of these two groups are concatenated).
    - $\text{groups} = \text{in\_channels}$ → Each input channel is **connected to its own individual set of filters**.
    </aside>
    
    **N.B.** This is useful because you can reduce the number of parameters by a factor of $g$ (=number of groups).
    

### 3.1.2. Shapes of Convolutional Layer

- **Input shape → $(n, iC, iH, iW)$** with $n$ = batch size (=number of images processed at once)
- **Kernel shape** → 

$(oC, iC, kH, kW)$ with $oC$ = number of output channels (i.e., the number of different filters applied)
- **Bias shape** → $(oC)$ shifts the output values of each channel after convolution
- **Output shape** → $(n, oC, oH, oW)$ where:
    - $oH = \left\lfloor \frac{iH + 2 \times \text{padding}[1] - \text{dilation}[1] \times (\text{kernel\_size}[1] - 1) - 1}{\text{stride}[1]} \right\rfloor + 1$
    - $oW = \left\lfloor \frac{{iW} + 2 \times \text{padding}[2] - \text{dilation}[2] \times (\text{kernel\_size}[2] - 1) - 1}{\text{stride}[2]} \right\rfloor + 1$

<aside>
📌

**Number of learnable parameters**

The number of learnable parameters in a convolutional layer with $OC$ output channels, $IC$ input channels, and kernel size $(kH × kW )$ is:

$$
\text{Parameters} = (kH × kW × IC × oC) + oC
$$

---

**How to compute the size of receptive field given the kernel size and the level of depth**

The general formula to compute the size of receptive field give kernel size $k\times k$ and number of level $n$ is:  $R = 1+ (k-1)\cdot n$

</aside>

## 3.2. Pooling Layer (2D)

Pooling reduces spatial resolution without learnable parameters. It takes an input tensor of size $(n, iC, iH, iW)$ and applies an operation, such as **max** or **average pooling** inside a neighbourhood**.** Differently form convolution, we do not perform a reduction over input channel axes, the operation is applied independently on all input channels. 

As for convolution we can set: stride, padding and dilation. The output dimensions are calculated as:

- $oH = \frac{(iH - kH)}{s} + 1$
- $oW = \frac{(iW - kW)}{s} + 1$
- $oC = iC$

![image.png](L3-CNN%20Architectures/image%202.png)

## 3.3. Normalization Layer

Normalization helps mitigate **covariate shift**, stabilizing training.

- The first step is to normalize the sample using the mean and the std of the current batch, these values are computed as the training is happening:  $\hat x = \frac{x-\mu_{batch}}{\sigma_{batch}}$
- Since the mean and the std are computed during training they are not perfect values, they are an approximation. To correct this approximation error we introduce two learnable parameters, gamma and beta:  $\text{final output} = \gamma\cdot\hat x + \beta$

---

# 4. Historical context of CNN architectures

## 4.1. AlexNet (2012)

Introduced by Krizhevsky in 2012, it was the first deep CNN to achieve top performance on ImageNet. It featured 8 layers and was trained across two GPUs due to hardware limitations at the time. Notable innovations included the use of ReLU activations, local response normalization, and dropout to prevent overfitting. The network used large $11×11$ convolutional kernels, it was the first and the last architecture to use these enormous kernels. 

## 4.2. VGG

The core idea behind VGG is that stacking multiple small filters is more effective than using one large filter.

- **Same Receptive Field, Smaller Filters** → a stack of three **3×3** convolutions has the **same receptive field** as a single **7×7** convolution:
    - $1\times[7\times 7]$ → $1+(7-1)\cdot 1 = 7$
    - $3\times[3\times 3]$ → $1-(3-1)\cdot 3= 7$
- **More Non-Linearities** → each convolutional layer is followed by a **ReLU** activation. Replacing one **7×7** convolution with **three 3×3** convolutions introduces 3 ReLUs  instead of one which increases the network’s ability to model complex functions and extract richer features.
- **Less Parameters** → smaller filters significantly reduce the number of parameters:
    - $1\times(C_{out},C_{in},7,7)$ → $\text{n° of param}=7\cdot 7\cdot C_{in}\cdot C_{out}= 49C^2$
    - $3\times(C_{out},C_{in},3,3)$ → $\text{n° of param}=(3\cdot 3\cdot C_{in}\cdot C_{out})\cdot 3= 27C^2$

![image.png](L3-CNN%20Architectures/image%203.png)

## 4.3. GoogleNet and the Inception Module (2014)

In 2014, Google introduced **GoogLeNet**, which was revolutionary thanks to the **Inception module**. The core idea was to design a *mini-network* and use it as a layer within the larger network. This allowed the model to apply **kernels of different sizes in parallel**, capturing information at multiple scales.

### 4.3.1. Naive Design

- Input is processed in parallel with convolution layers and pooling.
- Outputs are concatenated along the channel dimension.

<aside>
🚨

**Problem with this design**

- The intermediate feature maps grow very large
- Convolutions on high-dimensional inputs are computationally expensive
</aside>

<aside>
✅

**Solution**

We add a bottleneck layer that use **1×1 convolutions** before the larger ones to **reduce the number of input channels**.

![image.png](L3-CNN%20Architectures/image%204.png)

</aside>

### 4.3.2. Problem: The Vanishing Gradient

A major issue in **GoogLeNet** (and in deep networks in general) is the **vanishing gradient**:

![image.png](L3-CNN%20Architectures/image%205.png)

As the network becomes deeper, gradients become extremely small while propagating backward, making it difficult for early layers to learn. The solution is to add 2 additional MLPs to inject loss function value also in deep layers.

## 4.3. ResNet and the Residual Connection (2015)

From **AlexNet** to **VGG**, the trend was to build deeper networks with smaller kernels. But, in 2015 a study showed that simply increasing the depth of a network does not always improve performance. In fact, very deep networks often perform **worse** than shallower ones.

### 4.3.1 Why does this happen?

You might expect that a deeper network should perform at least as well, since, in principle, the extra layers could just learn the identity function and let the earlier layers do the real work. It turns out that learning the **identity function**, for a network, it’s actually really **hard**. For a single layer with ReLU non-linearity, the transformation is:

$$
f_{layer} = ReLU(W\cdot x+b)
$$

If we want to learn the identity function it is not enough to learn $W=I$ and $b = 0$, I also need to compensate the non linearity introduced by the ReLU. But it is not just that, learning a coordinated identity function across multiple layers is nearly impossible if the layers do not cooperate.

<aside>
📌

**Idea: Residual Learning**

To solve this, ResNet introduces an additional **shortcut connection**. Instead of forcing the network to learn the identity, we let the input 

$x$ “skip” a few layers and be added directly to their output:

$$

H(x) = F(x) + x
$$

In this way, the network only needs to learn the **residual function** $F(x)$, which is often easier than learning the full mapping $H(x)$.

![image.png](L3-CNN%20Architectures/145fc82b-8d42-46d6-94b0-cd005205e432.png)

</aside>

<aside>
🔢

**Why it works**

Consider the output: $y=F(x)+x$, during backpropagation, the gradient of the loss $L$ with respect to the input $x$ is:

$$

\frac{\partial L}{\partial x} = \frac{\partial L}{\partial y} \cdot \frac{\partial y}{\partial x}
$$

If we compute the partial derivative of $y$ in $x$ we get:

$$

\frac{\partial L}{\partial x} = \frac{\partial L}{\partial y} \cdot\left(\frac{\partial F(x)}{\partial x}+ \frac{\partial x}{\partial x}\right) = \frac{\partial L}{\partial y} \cdot\left(\frac{\partial F(x)}{\partial x}+ I\right)
$$

This result shows that if the residual weights go to zero ($F(x) \approx 0$), the block behaves like an **identity mapping,** without the need to coordinate with later layers, because $x$ will be passed forward by the residual connection:

$$

\frac{\partial L}{\partial x} \approx \frac{\partial L}{\partial y} ~~\text{for} ~~\frac{\partial F(x)}{\partial x} \longrightarrow 0
$$

Another important aspect, is that the residual connection also mitigates the problem of the **vanishing gradient** because even if the gradient of $F(x)$ gets very small (vanishes), the skip connection ensures that gradients can still propagate backward without loss.

</aside>

<aside>
🚨

**One Remaining Problem**

To sum $F(x)$ and $x$, their shapes must match, this means that the residual block cannot change the shape of the input.

**Solution:** ResNet introduces a projection function $G(x)$ (typically an convolution $1 \times1$) that has the only purpose of taking $x$ and reshape it so that it will match the same shape of $F(x)$.
For example, in cases where input and output dimensions differ (due to strides), a convolution with stride 2 is applied to the shortcut path to match shapes:

![image.png](L3-CNN%20Architectures/image%206.png)

</aside>

---

# 5. CNN Performance Comparison

|  | **Computational Load** | **Memory Usage** | **Accuracy** |
| --- | --- | --- | --- |
| **AlexNet** | light | heavy | 55% |
| **VGG** | heavy | heavy | 70/75% |
| **GoogleNet** | light | light | 70% |
| **ResNet** | medium | medium | 75/80% |

---