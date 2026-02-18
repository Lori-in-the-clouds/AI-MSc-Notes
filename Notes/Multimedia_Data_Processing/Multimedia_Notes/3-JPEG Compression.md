# 3-JPEG Compression
The **JPEG standard** was introduced because previous image compression techniques could achieve, at best, only a **50% reduction**, which is not sufficient given the large number of images we produce today. JPEG was specifically designed to handle **photographic images** with a continuous color spectrum, rather than computer-generated graphics.

This technique incorporates several components we’ve already encountered, such as **DCT (Discrete Cosine Transform)**, **RLE (Run-Length Encoding)**, and **Huffman Encoding** (a form of entropy coding).

The JPEG standard operates in **four different modes**:

- **Lossless JPEG**: an obsolete and rarely used version.
- **Baseline JPEG**: the standard, widely used sequential version.
- **Progressive JPEG**: a variation of the Baseline that improves loading for web images.
- **Hierarchical JPEG**: another variation of the Baseline, designed for scalability.

# 3.1 Lossless JPEG

**Lossless JPEG** is a compression mode within the JPEG family that **preserves all image data**. Exploits a linear prediction technique:

1. For each pixel $x$, the encoder tries to **predict** its value using the neighboring pixels. These neighbors typically are:
    
    
    - a = left
    - b = above
    - c = upper-left (diagonal)
    
    ![image.png](3-JPEG%20Compression/image.png)
    
    Lossless JPEG defines **8 possible prediction formulas** (predictors)
    

$$
y = 0 \newline y = a \newline y = b 
\newline y = c
$$

$$
y = a+b-c
$$

$$
y = a+\frac{b-c}{2}
$$

$$
y = b +\frac{a-c}{2}
$$

$$
y = \frac{a+b}{2}
$$

1. Calculate **residual value:** $Residual = x-y$, this creates a *residual image* made of small differences, typically close to zero.
2. Finally, the residuals are compressed using **Huffman coding**. Since these residual values are usually small and statistically centered near 0, the data exhibits low entropy. Huffman coding efficiently compresses such data, achieving high compression ratios without loss of information.
    
    <aside>
    🔑
    
    Example:
    
    </aside>
    

---

# 3.2 Baseline JPEG (Sequential JPEG)

This compression standard is considered **color-blind**, meaning it does not take color into account when developing compression techniques. Instead, it aims to compress matrices of bits per pixel, regardless of their meaning. For color images, this means we must first **separate the color channels** and process each one individually.

## 3.2.1 Steps

1. Pixels are scaled: $x'=x-2^{n-1}$
2. The image is divided into **non-overlapping 8 × 8 blocks**
3. DCT (Discrete Cosine Similarity) is applied to each block
    
    <aside>
    🔑
    
    **DCT**
    
    The Discrete Cosine Transform is a variant of the more well-known **Fourier Transform**, but it operates **only on real-valued coefficients,** nothing imaginary. This operation can be thought of as a **change of basis**: it allows us to observe our values from a new perspective, in a different space compared to the canonical one, making it possible to analyze their **frequency content**. **DCT coefficients** represent spatial frequency (= pixel variation rate along a direction).
    
    Example:
    
    Suppose we have the following values expressed in the standard (canonical) basis:
    
    $$
    
    v = \begin{bmatrix} 3 \\ 4 \end{bmatrix}, \quad B = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}
    $$
    
    The same values can be expressed in a different way, for example, using their **mean** and **half of their difference**. The basis changes accordingly:
    
    $$
    
    v = \begin{bmatrix} 3.5 \\ -0.5 \end{bmatrix}, \quad B = \begin{bmatrix} 0.5 & 0.5 \\ 0.5 & -0.5 \end{bmatrix}
    $$
    
    Now let’s look at the actual DCT. This is a **bidirectional transform**, meaning it is invertible. It is defined as follows:
    
    $$
    
    S_{uv} = \frac{1}{4} C_u C_v \sum_{x=0}^{7} \sum_{y=0}^{7} s_{xy} \cdot \cos\left( \frac{(2x+1)u\pi}{16} \right) \cdot \cos\left( \frac{(2y+1)v\pi}{16} \right)
    \quad \text{for } 0 \leq u, v \leq 7 \newline s_{xy} = \frac{1}{4} \sum_{u=0}^{7} \sum_{v=0}^{7} C_u C_v S_{uv} \cos\left( \frac{(2x+1)u\pi}{16} \right) \cos\left( \frac{(2y+1)v\pi}{16} \right) \quad \text{for } 0 \leq u, v \leq 7
    $$
    
    Where:
    
    - $s_{xy}$ is the input image block (typically 8×8 pixels),
    - $S_{uv}$ is the output DCT coefficient matrix,
    - $C_i$ is defined as:
        
        $$
        
        C_i =
        \begin{cases}
        \frac{1}{\sqrt{2}} & \text{if } i = 0 \\
        1 & \text{otherwise}
        \end{cases}
        $$
        
    - $v$ and $u$ represent rispectively the number of variation cycles along rows and colums:
        
        
        ![image.png](3-JPEG%20Compression/image%201.png)
        
        For each value of u (from $0$ to $7$), the graphs show how the cosine values vary within an 8-element block.
        
        For example, when $u = 0$, the function is nearly constant, representing the **lowest frequency component,** similar to the **average** of the signal.
        
        As $u$ increases (as seen in the subsequent plots up to $u = 7$), the basis functions exhibit faster oscillations, capturing the **higher frequency components** of the signal.
        
        **So theDCT thus projects the signal onto these different basis functions to obtain the coefficients** $S_{uv}$.
        
    
    So we can represent the input and output blocks as:
    
    $$
    
    s =
    \begin{bmatrix}
    s_{00} & s_{01} & \cdots & s_{07} \\
    s_{10} & s_{11} & \cdots & s_{17} \\
    \vdots & \vdots & \ddots & \vdots \\
    s_{70} & s_{71} & \cdots & s_{77}
    \end{bmatrix},
    \quad
    S =
    \begin{bmatrix}
    S_{00} & S_{01} & \cdots & S_{07} \\
    S_{10} & S_{11} & \cdots & S_{17} \\
    \vdots & \vdots & \ddots & \vdots \\
    S_{70} & S_{71} & \cdots & S_{77}
    \end{bmatrix}
    $$
    
    As we can see, **each value $S_{uv}$** receives a contribution from **every value $s_{xy}$** in the input block, this is the definition of a **scalar product**: 
    
    $S_{uv} = \frac{1}{4} C_{uv} \cdot (s_{xy} \cdot k_{uv})$
    
    This transform allows us, by changing the values of $u$ and $v$, to observe the **frequency** of the input data, how rapidly the values change.
    
    The result of the DCT can be visualized as a 2D matrix of frequency components, and this gives us the **motivation behind the zig-zag ordering**:
    
    - Along the diagonals, we find components with similar frequency.
    - The frequency increases as we move toward the **bottom-right** of the matrix.
    
    ![image.png](3-JPEG%20Compression/image%202.png)
    
    </aside>
    
4. CDT coefficients are quantized (integer division): ****studies show that the human eye is **more sensitive to low-frequency information**. For this reason:
    - The **low-frequency components** of the image are preserved **with higher precision** (=less aggressively compressed)
    - The **high-frequency components** are **more heavily compressed**
    
    In particular CDT coefficients are quantized executing pairwise integer division/multiplication with a **quantization matrix**:
    
    ![image.png](3-JPEG%20Compression/image%203.png)
    
    So in this way we can proportionally  increasing or decreasing the level of compression:
    
5. CDT coefficients are sorted using zig-zag path: we create a vector of pixels ordered according to frequency (ascending). To not prioritize $u$ frequencies over $v$ frequencies or vice versa we use **zig-zag** path:
    
    ![image.png](3-JPEG%20Compression/image%204.png)
    
6. Full image encoding: 
    
    The coefficients obtained from the DCT do not all have the same meaning:
    
    - The **first coefficient**, located at position **(0,0)** of each 8×8 block, is called the **DC coefficient**, and it is **proportional to the average pixel value** within the block.
    - The remaining coefficients are called **AC (alternating current) coefficients**, and they represent the **variations within the block**.
    
    The **DC coefficient** is compressed differently from the others because it is typically **similar to the DC coefficient of neighboring blocks**.
    
    <aside>
    📌
    
    **DC Encoding**
    
    How many bits are needed for DC coefficients? 
    
    If pixels are represented with 8 bits, each pixel value ranges from 0 to 255. An 8×8 block contains 64 pixels. The **DC coefficient** ($S_{00}$) is essentially the **average** of these 64 pixel values, scaled by certain DCT factors. Its **maximum value can exceed 255**.
    
    For instance, if all pixels are 255, the DC coefficient will not be 255, but rather something like: 
    
    $255 \times 8 \times 8 \times \text{(some scaling factor)} = 255 \times 64 \times \frac{1}{8} = 2040 \quad \text{(approx.)}$
    
    Typically, for **8×8 blocks** with **8-bit pixels**, the DC coefficient may range approximately from **−1024 to +1023,** for this reason **11 bits** is used to be represented.
    
    If a DC coefficient needs, say, **11 bits** to be represented (since 2^{10} = 1024, and values go up to ~2047), then the **difference between two DC values** (which is what’s actually encoded) may require **up to 12 bits.**
    
    How do we perform the encoding?
    
    If the values were **uniformly distributed**, we would need **12 bits** to represent them. However, in practice, they are **not uniformly distributed**. As usual, we make the assumption that **small differences are more likely** than large ones:
    
    ![image.png](3-JPEG%20Compression/image%205.png)
    
    With:
    
    - SSSS → indicate the number of bits needed to represent the difference
    - DC difference → difference between the current and previous DC
    
    Then category is encoded using **Huffman** so in this way we have a huffman table and a binary coding for the effective values:
    
    ![image.png](3-JPEG%20Compression/image%206.png)
    
    ---
    
    Example:
    
    Let’s take **class 2** as an example and see how the values are encoded:
    
    $$
    -3 = 00,\quad -2 = 01,\quad 2 = 10,\quad 3 = 11
    $$
    
    In practice, the encoding is performed using **1’s complement,** we **invert the bits** of the positive value to obtain its **negative representation**. This works because the **intermediate values** (e.g., 0, ±1) have already been encoded in the **previous classes**.
    
    </aside>
    
    <aside>
    📌
    
    **AC Encoding**
    
    The idea is similar to the encoding of DC coefficients, but this time we are **not encoding a difference between two values**, rather we encode the **value as it is**, with one key difference: **zeros are not encoded directly**, but instead handled through **Run-Length Encoding (RLE):**
    
    ![image.png](3-JPEG%20Compression/image%207.png)
    
    This time, for **each non-zero AC coefficient**, we associate it with the **number of preceding zeros**. It was decided to use **4 bits** to represent this count, which gives us **16 possibilities (0 to 15)**. In addition, we also need to encode the **size class** (i.e., the number of bits needed to represent the non-zero value). For this, there are **10 possible size categories**.
    
    So, the **Huffman table** will have: $16 \times 10 + 2 = 162 \text{ combinations}$
    
    The **+2 combinations** account for two special cases:
    
    - **ZRL (Zero Run Length)**: used when we have **16 consecutive zeros**
    - **EOB (End of Block)**: used when **all remaining coefficients are zero**
    </aside>
    
    <aside>
    📌
    
    **Byte stuffing**
    
    JPEG files contain *binary **encoded data* + *byte markers*** to determine sections of the file. 
    
    Markers consits of two byte: $FF\_\_$ (← section byte). To distinguish $FF$ of a marker by $FF$ binary encoded data when during encoding a $FF$ is produced, a $00$ is added.
    
    </aside>
    

---

# 3.3. Sequential JPEG recap

![image.png](3-JPEG%20Compression/image%208.png)

---

# 3.4 Color Management

When we work with color images we have 2 possibile approach:

1. **Encode each channel separately** 
    
    ![image.png](3-JPEG%20Compression/image%209.png)
    
2. **JFIF format** (JPEG file interchange format) → we apply **two different encoding strategies,**  one for the $Y$ **(luminance)** values, which are encoded **as they are**, since the human eye is much more sensitive to this component; and a different one for the $C_b$ **and $C_r$ (chrominance)** values, which undergo **subsampling** before compression, reducing of $\frac{1}{4}$ the  resolution of the original images.
    
    ![image.png](3-JPEG%20Compression/image%2010.png)
    

---