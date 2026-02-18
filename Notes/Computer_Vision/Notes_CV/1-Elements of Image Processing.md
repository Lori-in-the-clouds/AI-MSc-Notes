# 1-Elements of Image Processing

Done?: Done
Select: theory

<aside>
💡

Image processing is the science of studying how to automatically process images with a computer to produce other images, modified or enhanced in some visual properties. An image can be defined as:

- A discrete 2D continuous function, $I(x, y)$, making image processing a 2D extension of 1D signal processing
- A matrix of pixels, where each element corresponds to a digital values manipulated through mathematical functions and algorithms
</aside>

# 1. Image Representation and Color Models

1. **RGB and Grayscale representation**
    - **RGB:** each pixel is defined by 3 values that correspond to Red, Green and Blue channels, forming a 3D array, where pixels values range from 0 to 255 (for 8 bit images) or are normalized in [0,1].
    - **Grayscale images**: contain only luminance (intensity), represented by a single channel. Even if the human eye distinguishes 256 levels, fewer levels may be enough for automatic analysis.
2. **Palette based representation:** each pixel is a pointer to a corresponding row of a palette table (→ each row has 3 columns → RGB). This saves space but break signal continuity.
3. **Mathematical representation:** an image can be represented as a function: $I(x)=f(x,y):R^2 \rightarrow R$ where each point $(x,y)$ has an associated intensity value, which depends on the sensor capturing the image. For color images, the function becomes **vector-valued**:
    
    $$
    f(x,y)=
    
    \begin{bmatrix}
    r(x,y) \\
    g(x,y) \\
    b(x,y)
    \end{bmatrix}
    
    $$
    
    **N.B.** In practical applications, images are defined over a finite square interval and the value of the function is sampled and digitalised to be processed on the computers.
    

---

# 2. Images and Videos as Signals

A digital image is essentially a discretised signal, meaning that it’s obtained by sampling a continuous function. Even though images are discrete, certain operations assume they belong to a contiguous domain, allowing the use of mathematical tools like differentiation, gradient and convolution. Video extend this concept by adding a time dimension, making them 3 dimensional signals: $I(x,y,t)=l(x,y,t)$.

Signal can be classified based on their length. They are considered infinite if they extend across all the possibile value of $n$, while there are finite if they are 0 outside a specific interval.

<aside>

**Signal energy and $L_2$  metrics**

The energy of a signal can be computed as: $E = \sum_{-\infty}^{\infty}|l[n]|^2$

When we compare 2 signals, we compare their energy, and the simplest metrics is $L_2$ norm, which corresponds to the euclidian distance: $D = \frac{1}{N}\sum_{n=0}^{N-1}|l_1[n]-l_2[n]|^2$

</aside>

---

# 3. Image histogram

A histogram represents the distribution of pixel intensities in a image and can be seen as a discrete approximation of a probability density function (PDF), when pixel intensities are identically distributed (i.i.d.).

For a image $I$ of size $N\times M$  with $L$  possibile intensity levels (ranging form 0 to $maxrange$), the histogram $H(j)$ is defined as: $H(j)=\#\{x:I(x)=j\}$. To obtain a probability distribution, the histrogram must be normalized, ensuring that the sum of all probabilities are equal to 1: $\sum_{j=0}^{L-1}p_I(j)=1$.

## 3.1. Type of histogram

- **Normalized histogram:** a normalized histogram represents the probability of each pixel intensity to appear in the image. Given a grayscale image $I(x)$, the probability of an occurrence of a pixel of level $i$ in the image is: $(i) = \#\frac{\{x:I(x)=i\}}{n}$, $0 \leq i < L$ where $n$ is the total number of pixel in the image.
- **Cumulative histogram:** represents the accumulated probability of pixel intensities less than or equal to a given level **$i$:  $cdf_x(i)=\sum_{j=0}^{i}p_x(j)$.**

<aside>
✖️

**Statistical properties of the histogram**

Several statistical measures can be computed from the histogram to analyze an image:

- mean: $\displaystyle \mu =\sum_{j=0}^{L-1}jP(j)$ (→ represents the average intensity)
- standard deviation: $\displaystyle\sigma^2=\sum_{j=0}^{L-1}(j-\mu)P(j)$ (→ measure the spread of intensity values, indicating constrast variation)
</aside>

## 3.2. Entropy and image information

Entropy is a measure of uncertainly in the image values, indicating how much information is needed to encode an image. It’s calculated as: 

$$
Entropy(image)= -\sum_{j=0}^{L-1}P(j)\cdot \log_2(P(j))
$$

Where:

- High entropy → indicates greater complexity and more information content in the image
- Low entropy → means the image is more predictable (e.g. uniform images, highly compressed images)

![image.png](1-Elements%20of%20Image%20Processing/image.png)

---

# 4. Image processing Operators

Image processing uses some operators on image pixels which transform images into other images:

1. Point operators: modify each pixel independently, based only on that pixel → $I'(x,y)=h(I(x,y))$ (e.g. thresholding)
2. Local operators: modify a pixel based on its neighboring pixels → $I'(x,y)=h((x,y),\N(I(x,y))$ (e.g. filters)
3. Global operators: if the values of each pixels depends on all the pixels of the original images → $I'(x)=h(\int \int I(x,y))$ (e.g. Fourier transform)

---

# 5. Linear Systems

<aside>
🔑

A system is lineare if satisfies 2 properties:

- **Superposition principle**: $h(f_0+f_1)=h(f_0)+h(f_1)$ (← applying the operator to a sum of functions is the same as applying it separately and summing the result)
- **Scalar linearity**: $h(a\cdot f) = a\cdot h(f)$ (← applying the operator to a scaled function preserves the scaling factor)

![image.png](1-Elements%20of%20Image%20Processing/image%201.png)

</aside>

## 5.1. Linear point operator

<aside>
💡

A point operator is linear if: $g(x) = h(f(x))=sf(x)+k$ where: 

- $s$ is the scale factor (=gain/contrast)
- $k$ is the offset (=bias/brightness)
</aside>

Below there are 3 common examples:

- **Luminance Variation** → ****it’s a linear variation where pixels values are proportionally increased and decreased: $I'(x)= h(I(x))=sI(x)+k$.
    
    *Example*: increasing brightness by $10\%$ → $s= 1.1$, $k=0$. 
    
    ![image.png](1-Elements%20of%20Image%20Processing/image%202.png)
    
    **N.B**. Creating a negative image is also a linear transformation: $s=-1,k=255$ which inverts pixel intensities, making dark areas and vice versa.
    
- **Linear Blending** → it’s a linear pixel-level operator that combines 2 images together: $g(x)=h(f_0(x)+f_1(x))=(1-a)f_0(x)+af_1(x)$ (with $a$ = blending factor).
    
    ![image.png](1-Elements%20of%20Image%20Processing/image%203.png)
    
- **Contrast-Stretching** → it’s the expansion of the grey (or color) levels of pixels in a dynamic range given the histogram:
    
    
    $$
    
    \begin{cases}
    I'(x)=min1 \quad \quad if\ I(x) \le min\\
    I'(x)=max1 \quad \quad if\ I(x) \ge max\\ 
    I'(x)=(I(x)-min)\cdot (\frac{max1-min1}{max-min})+min1 \quad \quad if\ (I(x) > min)\ and \ (I(x)< max)
    \end{cases}
    $$
    
    ![image.png](1-Elements%20of%20Image%20Processing/268988b9-f2ee-482d-bc8c-2299b1bc6475.png)
    

---

# 6. Equalization (not linear operator)

Equalization is a non-linear image processing technique used to enhanced contrast by redistributing pixel intensity values. This method modifies the histogram so that pixel intensities are more spread across the full range (e.g. 0-255 in an 8-bit image):

$$

I'(x) = round(~(L-1) \cdot cdf(I(x))~)
$$

![image.png](1-Elements%20of%20Image%20Processing/image%204.png)

<aside>
📌

**Properties**

- Equalisation maximize entropy → it redistributes pixel intensities, increasing the variability of pixel values.
- It provides more “*information*” → enhances contrast, making details more distinguishable.
- It introduces non-linear changes to pixel values
- It distorts original intensity relationship, which might be crucial for computer vision

![image.png](1-Elements%20of%20Image%20Processing/image%205.png)

</aside>

---

# 7. Thresholding and clustering (Non-linear operator)

Thresholding is a segmentation technique that converts a greyscale image into a binary image. The task consists in the selection of a value $T$ capable of dividing the image into 2 regions of pixels with intensity greater or less than $T$: 

$$
I'(x)=\begin{cases}
1 \quad if\ I(x)\ge T \\
0 \quad otherwise

\end{cases}
$$

The threshold $T$ can be:

- **Global:** used across the whole image.
- **Adaptive:** it changes depending on the local area of the image. It’s calculated within a local window $W(i,j)$. The window size depends on the problem and the variability of the data. There are many algorithms to computes $T$ locally:
    
    
    1. $T=mean(W)$
    2. $T = median(W)$
    3. $T = mean(W)-C$, with $C$ as a correction constant that is used to correct some luminance problem in the acquisition (the best common choice is 
    
    $C = 7$ and $W=7\times 7$ )
    
    ![image.png](1-Elements%20of%20Image%20Processing/image%206.png)
    

The threshold can be determined:

- **Manually**
- **Automatically**, using automatic thresholding methods. Automatic thresholding assumes the histogram is **bimodal** (two distinct peaks for object and background).

<aside>
📌

**Otsu’s method (Automatic Global Thresholding)**

1. For each possible threshold $t$:
    1. Computes prior probabilities of both groups: $q_1(t)=\sum_{i=1}^t P(i),\  q_2(t)=\sum_{i=t+1}^L P(i)$
    2. Computes mean intensities of both groups: $\mu_1(t)=\sum_{i=1}^t \frac{iP(i)}{q_1(t)},\  \mu_2(t)=\sum_{i=t+1}^L \frac{iP(i)}{q_2(t)}$
    3. Computes variance of both groups: $\sigma_1^{2}(t)=\sum_{i=1}^t \frac{[i-\mu_1(t)]^2 \cdot P(i)}{q_1(t)},\ \sigma_2^{2}(t)=\sum_{i=t+1}^L \frac{[i-\mu_2(t)]^2 \cdot P(i)}{q_2(t)}$
2. Find the threshold $T$ that maximizes the **within-group variance:**
$\sigma_W^2(t)=q_1(t)\sigma_1^2(t)+q_2(t)\sigma_2^2(t)$

![image.png](1-Elements%20of%20Image%20Processing/image%207.png)

</aside>

<aside>
📌

**Linda Shapiro’s algorithm (Automatic Global Thresholding)**

1. Initialized an initial threshold $T$ (randomly or according to any other method)
2. Split the pixels into:
    - $G_1 = \{f(p):f(p)>T\}$ (object pixels)
    - $G_2 = \{f(p):f(p)\le T\}$ (background pixels)
3. Computes the mean intensity of both regions $m1$ and $m2$
4. Update the threshold as: $T' = \frac{m1+m2}{2}$
5. Repeat steps 2-4 until converge has been reached

**N.B.** This method is a special case of *K-means* clustering applies in one dimension.

</aside>

## 7.1. Nearest neighbour clustering classifier

This method allows gray-level segmentation using clustering:

1. Computes the cluster centers: $m_1 = \frac{1}{N_1}\sum_{i=1}^{N_1}g_1(i)$,  $m_2 = \frac{1}{N_2}\sum_{i=1}^{N_2}g_2(i)$  (→ $g_1(i)$ and $g_2(i)$ represent the intensity values of the pixels currently assigned to cluster $G_1$ and $G_2$)
2. Reassign each pixel to the nearest cluster: if a pixel intensity is closer to $m_1$ than to $m_2$, assign it to $G_1$; otherwise, assign it to $G_2$
3. Repeat it until none of the pixel labels changes anymore

---