# 5-Elements of 2D /3D Geometry and Cameral Model

Done?: Done
Select: theory

# 1. Homogeneous Coordinates

## 1.1. Why use Homogeneous coordinates?

When an image is represented in **2D Cartesian coordinates**, important information about the **3D structure** of the scene is lost, such as:

- **Points at infinity**
- The convergence of **parallel lines** in perspective images
- Perspective effects and viewpoint changes

To model these phenomena, we extend Euclidean geometry to **projective geometry**.

## 1.2. Projective Geometry

<aside>
💡

**Projective geometry** is a mathematical framework that generalizes Euclidean geometry by including **perspective and projection**. It extends the Euclidean plane by adding **points at infinity**:

- Each **direction** has its own point at infinity
- All these points together form a **special line** called the **line at infinity**
</aside>

### 1.2.1. Projective Plane

A projective plane is a set of points and lines that satisfy the following properties:

1. Any 2 distinct points lie on exactly one line
2. Any 2 distinct lines meet in exactly one point
3. There exist at least four points such that no three are collinear.

Formally, a projective plane over a **field** $\mathbb{F}$, can be visualized as a set of points in a higher-dimensional space, excluding the origin: $\mathbb{P}^n(\mathbb{F})= \mathbb{F}^{n+1} - \{\vec{0}\}$.

### 1.2.2. Point in the projective plane

- **2D Euclidean → Homogeneous**: $(x, y) \Rightarrow (x, y, 1)$
- **3D Euclidean → Homogeneous**: $(x, y, z) \Rightarrow (x, y, z, 1)$
- **Homogeneous → Euclidean** (only if $w ≠ 0$):
    - $(x, y, w) \Rightarrow (x/w, y/w)$
    - $(x, y, z, w) \Rightarrow (x/w, y/w, z/w)$

In general:

- if $z \ne 0$ , the point is **finite**.
- if $z = 0$, the point is **at infinity**. It does **not** correspond to a finite point in the plane.

**N.B.** Two points are considered equivalent if they differ by a nonzero scalar factor: 

$(x_0, \dots, x_n) \sim (\lambda x_0, \dots, \lambda x_n), \  \forall\lambda \in \mathbb{F} \setminus \{0\}$ (because the homogeneous system is **scale-invariant)**

### 1.2.3. Line in projective plane

Homogeneous coordinates of a line in a projective plane are $[a: b: c]$ that corresponds to $a\cdot x +b \cdot y+ c \cdot z = 0$. Line at infinte is equal to $[0:0:1]$.

- if $c\ne 0$, the line corresponds to a “finite” line. In 3D homogeneous coordinates, the corresponding plane intersects the projective plane in a normal line (you can convert it to Cartesian coordinates by setting z=1).
- if $c = 0$, the line lies **at infinity**. In 3D homogeneous coordinates, the corresponding plane is parallel to the projective plane and **never intersects it** at a finite point.

---

# 2. Image Warping

<aside>
💡

**Filtering vs Warping**

Images can be modified in two main ways: 

- **Filtering** → operates on the **range** of the image, it changes the **value** (the intensity $I$) of a pixel, leaving its **position** $(x,y)$ unaltered: $I'(p) = I(p) * f(p)$.
    
    Here, each pixel intensity is modified by applying a filter function $f(p)$.
    
    *Example:* the image is blurred.
    
- **Warping** → operates on the **domain** of the image, it changes the **position** $(x,y)$ of a pixel, leaving its **value** (intensity $I$) unaltered: $I'(x) = I(f(p))$.
    
    A transformation function $f$ is applied to the coordinates of every pixel. This is often expressed using **homogeneous coordinates** (not possible with Cartesian coordinates): $\begin{bmatrix}x' \\y'\end{bmatrix}=\mathbf{M}\begin{bmatrix}x \\y\end{bmatrix}$
    
    *Example:* the image is rotated.
    
</aside>

## 2.1. Transformation Classes in 2D Geometry

<aside>
💡

**Classes** are group of transformations that behave similarly or share the same properties. Each class of transformations has:

- A certain number of **degrees of freedom (DOF)**, meaning how many independent parameters are needed to describe it
- **Invariant geometric properties** that the transformation preserves (e.g., lengths, angles, parallelism)
- A characteristic **matrix representation**
</aside>

1. **Classe I: Isometry** (It combines translation and rotation)
    - **Preserve:** lengths, angles, area and orientation (if the determinant of the rotation matrix is $+1$).
    - **DOF**: 3 [2 for translation $(d_x,d_y)$, 1 for rotation ($\theta$)].
    
    <aside>
    ✖️
    
    **Matrix representation in homogeneous coordinates:** 
    
    $$
    \begin{bmatrix}
    x' \\
    y' \\
    1
    \end{bmatrix}=
    
    \begin{bmatrix}
    \epsilon cos\theta & -sin\theta & t_x \\
    \epsilon sin\theta & cos\theta & t_y \\
    0 & 0 & 1\end{bmatrix}
    \begin{bmatrix}
     x \\
     y  \\
    1
    \end{bmatrix}
    $$
    
    with:
    
    $\epsilon = 1$, orientation is preserved
    
    $\epsilon = -1$, orientation is reversed
    
    </aside>
    
    1. **Translation →** 
    $\begin{bmatrix}
    x' \\
    y' \\
    1
    \end{bmatrix}=
    
    \begin{bmatrix}
    1 & 0 & t_x \\
    0 & 1 & t_y \\
    0 & 0 & 1
    \end{bmatrix}
    \begin{bmatrix}
    X_1\\
    y_1 \\
    1
    \end{bmatrix}$
    2. **Rotation** → $\begin{bmatrix}
    x' \\
    y' \\
    1
    \end{bmatrix}
    =
    \begin{bmatrix}
    \epsilon \cos \theta & -\sin \theta & 0 \\
    \epsilon \sin \theta & \cos \theta & 0 \\
    0 & 0 & 1
    \end{bmatrix}
    \begin{bmatrix}
    x \\
    y \\
    1
    \end{bmatrix}
    \quad
    \epsilon = \pm 1$
2. **Class II: Similarity Transformation** (It combines **uniform** **2D scaling**, **rotation**, and **translation**)
    - **Preserve**: shapes, angles, and proportions.
    - **DOF**: 4 [1 for scale ($s$), 1 for rotation ($\theta$), 2 for translation ($t_x,t_y$)].
    - **Orientation:** determinant > 0 preserves orientation, determinant < 0 reverses orientation.
    
    <aside>
    ✖️
    
    **Matrix representation in homogeneous coordinates:**
    
    $$
    \begin{bmatrix}
    x’ \\
    y’ \\
    1
    \end{bmatrix}=
    
    \begin{bmatrix}
    s\cos\theta & -s\sin\theta & t_x \\
    s\sin\theta & s\cos\theta & t_y \\
    0 & 0 & 1
    \end{bmatrix}
    \begin{bmatrix}
    x \\
    y \\
    1
    \end{bmatrix}
    $$
    
    </aside>
    
    1. **Scaling →** $\begin{pmatrix}
    x' \\
    y' \\
    1
    \end{pmatrix}
    =
    \begin{pmatrix}s_x & -s_y & 0 \\
    s_x & s_y & 0 \\
    0 & 0 & 1
    \end{pmatrix}
    \begin{pmatrix}
    x \\
    y \\
    1
    \end{pmatrix}$
3. **Class III: Affine Transformations** (It combines **translation**, **rotation**, **anisotropic scaling)**
    - **Preserve**: parallelism of line, ratios of distances along parallel lines, ratio of areas (not the area itself).
    - **DOF:** 6 ****[2 for scale, 2 for rotation, 2 for translation]
    
    <aside>
    ✖️
    
    **Matrix representation in homogeneous coordinates:**
    
    $$
    \begin{bmatrix}
    x’ \\
    y’ \\
    1
    \end{bmatrix}
    
    \begin{bmatrix}
    a_{11} & a_{12} & t_x \\
    a_{21} & a_{22} & t_y \\
    0 & 0 & 1
    \end{bmatrix}
    \begin{bmatrix}
    x \\
    y \\
    1
    \end{bmatrix}
    $$
    
    This can be written compactly as: $x′ = H_Ax$,  where $H_A$ is the homogeneous matrix for affine transformation: $H_A = \begin{bmatrix}A & t \\ 0^T & 1\end{bmatrix}$
    
    </aside>
    
4. **Class IV: Projective Transformations** → these are the most general 2D transformations and allow modeling of **perspective effects** (e.g., lines that converge at a vanishing point). 
    - **Preserve**: cross-ratio of four collinear points
    - **DOF**: 8 [2 for rotation, 2 for translation, 2 for scale, 2 related to the line at infinity]
    
    <aside>
    ✖️
    
    **Matrix representation in homogeneous coordinates**
    
    $$
    \begin{bmatrix}x' \\y'\\w'\end{bmatrix}=\begin{bmatrix}h_{11} & h_{12} & h_{13} \\ h_{21} & h_{22} & h_{23} \\h_{31} & h_{32} & h_{33}\end{bmatrix}\begin{bmatrix}x \\y \\1\end{bmatrix}, ~h_{33} \ne  0
    
    $$
    
    We can normalize with respect to $h_{33}$:
    
    $$
    
    \begin{bmatrix}x' \\y' \\w'\end{bmatrix}=\begin{bmatrix}h_{11} & h_{12} & h_{13} \\ h_{21} & h_{22} & h_{23} \\h_{31} & h_{32} & 1\end{bmatrix}\begin{bmatrix}x\\y\\1\end{bmatrix}
    $$
    
    These coordinates are no longer euclidean, to switch back to them we need to normalize $x'$ and $y'$ with $w'$.
    
    </aside>
    

## 2.2. Transformation Classes in 3D

1. **Classe I: Isometry**
    1. **Translation:**
        
        $$
        \begin{bmatrix}
        X_2 \\
        Y_2 \\
        Z_2 \\
        1
        \end{bmatrix}=
        
        \begin{bmatrix}
        1 & 0 & 0 & dx \\
        0 & 1 & 0 & dy \\
        0 & 0 & 1 & dz \\
        0 & 0 & 0 & 1
        \end{bmatrix}
        \begin{bmatrix}
        X_1 \\
        Y_1 \\
        Z_1 \\
        1
        \end{bmatrix} = T\cdot \begin{bmatrix}
        X_1 \\
        Y_1 \\
        Z_1 \\
        1
        \end{bmatrix}
        $$
        
    2. **Rotation:**
        
        $$
        R_X(\alpha) =
        \begin{bmatrix}
        1 & 0 & 0 & 0 \\
        0 & \cos \alpha & -\sin \alpha & 0 \\
        0 & \sin \alpha & \cos \alpha & 0 \\
        0 & 0 & 0 & 1
        \end{bmatrix} \quad R_Y(\alpha) =
        \begin{bmatrix}
        \cos \alpha & 0 & \sin \alpha & 0 \\
        0 & 1 & 0 & 0 \\
        -\sin \alpha & 0 & \cos \alpha & 0 \\
        0 & 0 & 0 & 1
        \end{bmatrix}
        \quad
        R_Z(\alpha) =
        \begin{bmatrix}
        \cos \alpha & -\sin \alpha & 0 & 0 \\
        \sin \alpha & \cos \alpha & 0 & 0 \\
        0 & 0 & 1 & 0 \\
        0 & 0 & 0 & 1
        \end{bmatrix}
        $$
        
2. **Class II: Similarity**
    1. **Scaling** → can be non-uniform along each axis, with factors $S_x$, $S_y$, $S_z$:
        
        $$
        \begin{bmatrix}
        X_2 \\
        Y_2 \\
        Z_2 \\
        1
        \end{bmatrix}=
        
        \begin{bmatrix}
        S_x & 0 & 0 & 0 \\
        0 & S_y & 0 & 0 \\
        0 & 0 & S_z & 0 \\
        0 & 0 & 0 & 1
        \end{bmatrix}
        \begin{bmatrix}
        X_1 \\
        Y_1 \\
        Z_1 \\
        1
        \end{bmatrix} = S\cdot \begin{bmatrix}
        X_1 \\
        Y_1 \\
        Z_1 \\
        1
        \end{bmatrix}
        $$
        
3. **Class III: Affine Transformation** → the property that parallel lines remain parallel holds true. The transformation can be represented by a $3\times 4$ matrix (when applied to 3D points in homogeneous coordinates):
    
    $$
    
    x' =
    \begin{bmatrix}
    a_{00} & a_{01} & a_{02} & a_{03} \\
    a_{10} & a_{11} & a_{12} & a_{13} \\
    a_{20} & a_{21} & a_{22} & a_{23} 
    \end{bmatrix}\cdot x
    $$
    
    This matrix can be interpreted as a general linear transformation (scaling, shear, rotation) followed by a translation.
    
4. **Class IV: Projective Transformation** → it allows for realistic visual effects such as **foreshortening** (objects appearing smaller as they move farther away) and **convergence of parallel lines**, which are common in real-world camera images. This transformation is represented by a full **4×4 matrix** in homogeneous coordinates:
    
    
    $$
    \tilde{x}'=\tilde{H}\tilde{x}\quad\text{where}\quad H=\begin{bmatrix}
    \textcolor{green}{m_1}
     & \textcolor{orange}{m_{5}}   & \textcolor{red}{m_{9}} & m_{13} \\
    \textcolor{green}{m_2} & \textcolor{orange}{m_{6}} & \textcolor{red}{m_{10}} & m_{14} \\
    \textcolor{green}{m_3} & \textcolor{orange}{m_{7}} & \textcolor{red}{m_{11}}& m_{15} \\
    \textcolor{gray}{m_4} & \textcolor{gray}{m_8} & \textcolor{gray}{m_{12}} & \textcolor{gray}{m_{16} }\\\end{bmatrix}
    $$
    
    ![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image.png)
    
    - The entries $m_{13},$ $m_{14}$, $m_{15}$ represent the **translation component**
    - The bottom row ($m_4$, $m_8$, $m_{12}$, $m_{16}$) introduces the **perspective effect** by modifying the **homogeneous coordinate** $w$. If all these values are 0 except $m_{16} = 1$, then the transformation becomes affine.

## 2.3. Composite Transformation

A **composite transformation** is created by **combining multiple individual transformations** (e.g., translation, rotation, scaling) into one single operation by **matrix multiplication**.

Let’s say you want to apply a sequence of transformations: $T_1$, $T_2$, $T_3$, $T_4$ → the **composite transformation** matrix is: $T_c = T_4 \cdot T_3 \cdot T_2 \cdot T_1$

<aside>
🚨

**Beware:** matrix multiplication is **not commutative**, meaning: 

$A \cdot B \neq B \cdot A$

</aside>

## 2.4. Warping Algorithm

Once you have the **transformation matrix**, you need to decide how to apply it to the image:

- **Forward Warping** → for each pixel $(x,y)$ in the source image:
    1. Compute the destination coordinates $(x’, y’) = h(x, y)$, where $h$ is the warping function.
    2. Copy the pixel value from $(x, y)$ to $(x’, y’)$ in the destination image.
    
    Contro:
    
    - $(x’, y’)$ may not be integer, requiring additional processing such as **splatting** to distribute the value to neighboring pixels.
    - Some pixels in the destination image may remain **undefined** if no source pixel maps exactly to them.
- **Inverse Warping** → for each pixel $(x’, y’)$ in the destination image:
    1. Compute the corresponding source location $(x, y) = h^{-1}(x’, y’)$, using the inverse of the warping transformation.
    2. Copy the pixel value from $(x', y')$ in the destination to $(x’, y’)$ in the source.
    
    Pro:
    
    - If $(x,y)$ is not an integer, interpolation can be easily applied to determine it.
    - No holes in occur in the destination image, since every pixel is explicitly assigned a value.

## 2.5. Focus on Homographic transformation (= Projective Transformation)

<aside>
💡

A **homographic transformation** is a specific type of warping that maps one planar image onto another. It applies only to objects lying on the **same plane** in 3D space. There are 2 main challenges:

1. How to provide a parametric transformation
2. **How to compute the homography matrix $H$** (= $3×3$ matrix that represents the transformation in homogeneous coordinate)
</aside>

### 2.5.1. Estimating the homography matrix $H$

- Select 4 points that all lie on the same plane and are not collinear (i.e., do not lie on a single straight line).
- Each point correspondence provides 2 equations, so 4 points give 8 equations
- The matrix $H$ has **8 degrees of freedom** (since scale is arbitrary, one parameter can be fixed, e.g., $h_{22} = 1$)
- Solve a linear system with these 8 equations to find $H$

<aside>
🔢

**Mathematical point of view:**

Let’s suppose we have 4 points in the **first image** ($p_0,p_1,p_2,p_3$) and their corresponding points in a **second image** ($p_0',p_1',p_2',p_3'$). For each point in the first image we have:

$$
\begin{bmatrix}
x_i'\\
y_i'\\
1
\end{bmatrix}=
\begin{bmatrix}
h_{00} & h_{01} & h_{02}\\
h_{10} & h_{11} & h_{12}\\
h_{20} & h_{21} & 1
\end{bmatrix}
\begin{bmatrix}
x_i\\
y_i\\
1
\end{bmatrix}
$$

For each correspondence $(x_i, y_i) \leftrightarrow (x_i’, y_i’)$, the homography is defined by:

$$

\begin{aligned}
x’_i &= h_{00} x_i + h_{01} y_i + h_{02} \quad \quad
y’_i = h_{10} x_i + h_{11} y_i + h_{12}
\end{aligned} \quad \quad 1 = h_{20}x_i + h_{21}y_i + 1
$$

Divide $x_i'$ and $y_i'$ with the third equation:

$$

\begin{aligned}
x’_i &= \frac{h_{00} x_i + h_{01} y_i + h_{02}}{h_{20} x_i + h_{21} y_i + 1}
\\ y’_i &= \frac{h_{10} x_i + h_{11} y_i + h_{12}}{h_{20} x_i + h_{21} y_i + 1}
\end{aligned}\quad \Rightarrow 

\begin{cases}
h_{00} x_i + h_{01} y_i + h_{02} -h_{20}x_1x_1'-h_{21}y_ix_i'=x_i' \\
h_{10} x_i + h_{11} y_i + h_{12} -h_{20}x_1y_1'-h_{21}y_iy_i'=y_i'
\end{cases}

$$

Since each point correspondence provides 2 equations, and this process must be repeated for all 4 points, we end up with a system of 8 equations and 8 unknowns. In matrix form:

$$
\begin{bmatrix}x_0 & y_0 & 1 & 0 & 0 & 0 & -x_0 x'_0 & -y_0 x'_0 \\0 & 0 & 0 & x_0 & y_0 & 1 & -x_0 y'_0 & -y_0 y'_0 \\x_1 & y_1 & 1 & 0 & 0 & 0 & -x_1 x'_1 & -y_1 x'_1 \\0 & 0 & 0 & x_1 & y_1 & 1 & -x_1 y'_1 & -y_1 y'_1 \\x_2 & y_2 & 1 & 0 & 0 & 0 & -x_2 x'_2 & -y_2 x'_2 \\0 & 0 & 0 & x_2 & y_2 & 1 & -x_2 y'_2 & -y_2 y'_2 \\x_3 & y_3 & 1 & 0 & 0 & 0 & -x_3 x'_3 & -y_3 x'_3 \\0 & 0 & 0 & x_3 & y_3 & 1 & -x_3 y'_3 & -y_3 y'_3 \\\end{bmatrix}\begin{bmatrix}h_{00} \\h_{01} \\h_{02} \\h_{10} \\h_{11} \\h_{12} \\h_{20} \\h_{21} \\\end{bmatrix}=\begin{bmatrix}x'_0 \\y'_0 \\x'_1 \\y'_1 \\x'_2 \\y'_2 \\x'_3 \\y'_3 \\\end{bmatrix}
$$

---

**Case with N points (N > 4):**

When more than 4 points are available, the system is **overdetermined** (more equations than unknowns → there is no exact solution that satisfies all the equations simultaneously), so a **least-squares solution** (e.g., via SVD) is used to find the best-fitting homography.

$$
\begin{bmatrix}x_0 & y_0 & 1 & 0 & 0 & 0 & -x_0x_0' & -y_0x_0' \\0 & 0 & 0 & x_0 & y_0 & 1 & -x_0y_0' & -y_0y_0' \\x_1 & y_1 & 1 & 0 & 0 & 0 & -x_1x_1' & -y_1x_1' \\0 & 0 & 0 & x_1 & y_1 & 1 & -x_1y_1' & -y_1y_1' \\\vdots & \vdots & \vdots & \vdots & \vdots & \vdots & \vdots & \vdots \\x_{N-1} & y_{N-1} & 1 & 0 & 0 & 0 & -x_{N-1}x_{N-1}' & -y_{N-1}x_{N-1}' \\0 & 0 & 0 & x_{N-1} & y_{N-1} & 1 & -x_{N-1}y_{N-1}' & -y_{N-1}y_{N-1}'\end{bmatrix}\begin{bmatrix}h_{00} \\ h_{01} \\ h_{02} \\ h_{10} \\ h_{11} \\ h_{12} \\ h_{20} \\ h_{21}\end{bmatrix}=\begin{bmatrix}x_0' \\ y_0' \\ x_1' \\ y_1' \\ \vdots \\ x_{N-1}' \\ y_{N-1}'\end{bmatrix}
$$

</aside>

### 2.5.2. Applications

The main applications of homographic transformations include:

1. **Transfer an image into another plane** (e.g., perspective rectification)
2. **Image Stitching (Panorama Creation) →** combine several overlapping images to create a single, wide-view image, such as a panorama.
3. **Template Matching Under Geometric Variations →** given a template and a target image, robustness can be improved by applying all possible Euclidean, affine, and homographic transformations of the template before matching. This increases tolerance to geometric distortions but is computationally expensive.
4. **Augmented Reality**

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%201.png)

## 2.6. 3D to 2D Transformations

- **Orthographic Projection:** projects a 3D point directly into a 2D plane by **ignoring one coordinate**, typically the depth ($z$):
    
    $$
    \textbf{P} =
    \begin{bmatrix}
    x \\
    y \\
    z
    \end{bmatrix}
    \quad \xrightarrow{\text{projection}}  \quad
    \textbf{P'} =
    \begin{bmatrix}
    x \\
    y \\
    \end{bmatrix}
    $$
    
    <aside>
    
    Properties:
    
    - **Parallel lines remain parallel**
    - **Object size does not change** with distance [suitable for long focal lengths or distant objects (e.g., satellite imagery, technical drawings]
    - Preserves scale and angles, but **loses depth information**
    </aside>
    
    In matrix form for a point $p$:
    
    $$
    
    \mathbf{x} =
    \begin{bmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \end{bmatrix} p
    \quad
    \text{or in homogeneous coordinates: }\ 
    \mathbf{\tilde{x}} =
    \begin{bmatrix}
    1 & 0 & \textcolor{orange}{0} & 0 \\
    0 & 1 & \textcolor{orange}{0} & 0 \\
    0 & 0 & \textcolor{orange}{0} & 1
    \end{bmatrix}
    \tilde{\mathbf{p}} = \begin{bmatrix}
    1 & 0 & {0} & 0 \\
    0 & 1 & {0} & 0 \\
    0 & 0 & {0} & 1
    \end{bmatrix}\cdot \begin{bmatrix}
    x   \\
    y   \\
    z\\	
    1  
    \end{bmatrix} = 
    \begin{bmatrix}
    x   \\
    y   \\  
    1
    \end{bmatrix}
    $$
    

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%202.png)

- **Perspective Transformation:** objects appear **smaller** as they move **farther away**. For a 3D point $p = [x, y, z]^T$, the 2D projection is: $\textbf{x} =
\begin{bmatrix}
x / z \\
y / z \\
\end{bmatrix}$.
    
    <aside>
    
    Properties:
    
    - **Foreshortening:** objects shrink with distance
    - **Convergence**: parallel lines may appear to meet (e.g., train tracks).
    - More **realistic** and commonly used in computer vision and graphics.
    </aside>
    
    In homogeneous coordinates:
    
    $$
    \tilde{x} =
    \begin{bmatrix}
    1 & 0 & 0 & \textcolor{orange}{0} \\
    0 & 1 & 0 & \textcolor{orange}{0} \\
    0 & 0 & 1 & \textcolor{orange}{0}
    \end{bmatrix}
    \tilde{p} = \begin{bmatrix}
    1 & 0 & 0 & {0} \\
    0 & 1 & 0 & {0} \\
    0 & 0 & 1 & {0}
    \end{bmatrix}\cdot
    \begin{bmatrix}
    x \\
    y \\
    z \\
    1 \\
    \end{bmatrix} = \begin{bmatrix}
    x \\
    y \\
    z \\
    \end{bmatrix} 
    
    \xrightarrow{\text{Normalize by}\ z}
    \begin{bmatrix}x/z\\y/z\\1 \end{bmatrix}
    $$
    
    **N.B.** This transformation captures depth cues but introduces non-linearity due to the division by $z$. 
    

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%203.png)

---

# 3. The Camera Model

An image is more than a matrix of pixel values; it digitally represents the physical signals captured by a camera. Light illuminates the scene, objects reflect it, and the camera sensor records the reflected radiation onto a 2D image plane. This process can be modeled mathematically as a projection of the 3D world onto the 2D plane, which forms the basis for camera models and geometric transformations in computer vision.

## 3.1. The Pinhole Camera Model

A **pinhole camera** is the simplest device to project a 3D scene into a 2D surface. It works by allowing light rays to pass through a small aperture, forming an (inverted) image on the image plane, a principle known as **camera obscura**.

<aside>
📌

**Real vs Virtual Images**

- A **real image** is formed at the point of intersection of real light rays.
- A **virtual image** dos not exist in reality but is formed on the extension of reflected rays  (e.g., in a mirror). The image plane in computer vision typically refers to this virtual image plane.
</aside>

### 3.1.1. Pinhole Camera with Perspective Projection

The essential parameters of a pinhole camera system are:

- **Optical center ($C$)** → the center through which all light rays pass
- **Image plane ($I$)** → the surface where the image is projected
- **Focal length ($f$)** → the distance from the optical center to the image plane
- **Optical axis** → the straight line that passes through the optical center and is perpendicular to the image plane
- **Principal point ($c$)** → the intersection of the optical axis with the image plane
- **Focal plane ($F$)**:  a plane parallel to the image plane that contains $C$

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/76dfb5a2-b9e3-4292-bbe4-eefd3400ddd2.png)

To simplify computations, we use a **virtual image plane** placed **in front of** the optical center at distance $f$, producing an **upright image** without changing the projection geometry.

<aside>
🔢

**Projection form 3D to 2D**

To map a 3D point $P = (X, Y, Z)$ to a 2D point $p = (x, y)$:

- Set the **optical center $C$** at the origin $(0, 0, 0)$
- Place the **virtual image plane** at distance $f$ along the $Z$-axis
- Using **similar triangles** (=the ratio between the heights and the bases of the two triangles is the same), the projection equations are:
    - For the **x-coordinate**: 
    
    $\frac{X}{Z}=\frac{x}{f}  \quad \Rightarrow \quad x = f \cdot \frac{X}{Z}$
    - For the **y-coordinate**: 
    
    $\frac{Y}{Z}=\frac{y}{f}  \quad \Rightarrow \quad y = f \cdot \frac{Y}{Z}$

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%204.png)

</aside>

<aside>
🔑

**Characteristics of the Model**

- **Non-linear Equations →** the division by $Z$ introduces non-linearity, **which causes objects to appear smaller** as they move farther from the camera.
- **Depth Ambiguity** → since $f$ and $Z$ are often unknown from a single 2D image, $Z$ cannot be recovered from a single 2D image without extra information.
- **Generative model** → the pinhole camera model is considered a generative model. It describes the probability $Pr(x|w)$ of observing a feature at position $x = [x,y]^T$ in the image, given a 3D world point $w = [X, Y, Z]^T$.
</aside>

## 3.2. Normalized Camera Model

The **normalized camera model** is a simplified version of the pinhole model, often used to make mathematical formulations easier, especially when working with **homogeneous coordinates**. It establishes a canonical projection scenario by making specific assumptions:

- **Focal length $f=1$** → this defines a ”normalized image plane” where the projection of 3D points occurs
- **Principal point at the origin** → the origin of the 2D image coordinate system $(x,y)$ is centered at the optical axis, removing the need to account for translation offsets.

So, we have that:

- For the **x-coordinate**: 

$\frac{X}{Z}=\frac{x}{1}  \quad \Rightarrow \quad x =\frac{X}{Z}$
- For the **y-coordinate**: 

$\frac{Y}{Z}=\frac{y}{1}  \quad \Rightarrow \quad y =\frac{Y}{Z}$

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%205.png)

## 3.3. Camera Calibration

<aside>
💡

Camera calibration is the process of estimating both the **intrinsic** and **extrinsic** parameters of a camera, enabling the mapping between 3D world coordinates and 2D image coordinates.

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%206.png)

</aside>

### 3.3.1. Intrinsic Parameters

These parameters describe the **internal characteristics** of the camera and how it maps 3D points in the camera coordinate frame to pixel coordinates:

- **Focal Length** $(f_x, f_y)$ → measures how strongly the camera converges light, possibly differing in the $x$ and $y$ directions when pixels aren’t square.
- **Principal Point** $(x_c, y_c)$ → the point where the optical axis intersects the image plane. It may not be at the center.
- **Skew** $(s)$ → describes the angle between the image’s $x$ and $y$ axes. It is usually $0$ for modern digital cameras.

The intrinsic matrix $K$ combines these parameters: 

$K =
\begin{bmatrix}
f_x & s & x_c \\
0 & f_y & y_c \\
0 & 0 & 1
\end{bmatrix}$

### 3.3.2. Extrinsic Parameters

These define the **position and orientation** of the camera in the world coordinate system.

- A **rotation matrix** $R$ ($3\times 3$) to align the camera axes with the world
- A **translation vector** $t$ ($3 \times 1$) to locate the camera in space

 A 3D point in the world $\mathbf{X}_{\text{world}} = [X, Y, Z]^T$ is transformed into the camera coordinate frame by:  $\mathbf{X}_{\text{camera}} = R \mathbf{X}_{\text{world}} + t$

This transformation is often written compactly as a **single $3\times 4$ matrix**: 

$$
[R|t]=\begin{bmatrix}
r_{11} & r_{12} & r_{13} & t_x \\
r_{21} & r_{22} & r_{23} & t_y \\
r_{31} & r_{32} & r_{33} & t_z
\end{bmatrix}

$$

<aside>
📌

**Rotation Matrices**

Camera orientation can be represented through **Euler angles**, which decompose the total rotation into 3 successive rotations:

- **Pitch** (rotation around the **x-axis**, angle $\alpha$):  

$R_x(\alpha) =
\begin{bmatrix}
1 & 0 & 0 \\
0 & \cos\alpha & -\sin\alpha \\
0 & \sin\alpha & \cos\alpha
\end{bmatrix}$
- **Roll** (rotation around the **y-axis**, angle $\beta$):  

$R_y(\beta) =
\begin{bmatrix}
\cos\beta & 0 & \sin\beta \\
0 & 1 & 0 \\
-\sin\beta & 0 & \cos\beta
\end{bmatrix}$
- **Yaw** (rotation around the **z-axis**, angle $\gamma$): 

$R_z(\gamma) =
\begin{bmatrix}
\cos\gamma & -\sin\gamma & 0 \\
\sin\gamma & \cos\gamma & 0 \\
0 & 0 & 1
\end{bmatrix}$

The final rotation matrix is the product of these three, applied in a specific order (typically $z$–$y$–$x$):  

$R = R_z(\gamma) \cdot R_y(\beta) \cdot R_x(\alpha)$

</aside>

### 3.3.3. Full Camera Projection Model (Intrinsic and Extrinsic combined)

By combining intrinsic and extrinsic parameters, a 3D point $\mathbf{X} = (X, Y, Z, 1)^T$ can be projected onto a 2D image point  $\mathbf{x} = (x, y, 1)^T$ through the **projection matrix** $P$:

$$

\mathbf{x} \sim P \mathbf{X} = K [R | t] \mathbf{X}
$$

So, the full expression becomes:

$$

\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
\sim
\begin{bmatrix}
f_x & s & x_c \\
0   & f_y & y_c \\
0   & 0   & 1
\end{bmatrix}
\cdot
\begin{bmatrix}
r_{11} & r_{12} & r_{13} & t_x \\
r_{21} & r_{22} & r_{23} & t_y \\
r_{31} & r_{32} & r_{33} & t_z
\end{bmatrix}
\cdot
\begin{bmatrix}
X \\
Y \\
Z \\
1
\end{bmatrix}

$$

**N.B.** This projection matrix $P$ has **11 degrees of freedom** (since it is defined up to scale).

### 3.3.4. Camera Model Variations

Depending on the camera setup and characteristics, the model can take several forms, each refining the intrinsic matrix $K$ to better reflect reality:

1. **Ideal Pinhole Model →** the **camera center $C$** is the origin of the world coordinate system, and the **principal point** is located at $(0, 0, f)$ on the image plane.
    
    $$
    \begin{bmatrix}
    x \\
    y \\
    1
    \end{bmatrix}
    \sim
    \begin{bmatrix}
    f & 0 & 0 & 0\\
    0 & f & 0 & 0 \\
    0 & 0 & 1& 0
    \end{bmatrix}
    \begin{bmatrix}
    X  \\
    Y  \\
    Z \\
    1
    \end{bmatrix}\quad \Rightarrow \quad \begin{bmatrix}
    f & 0 & 0\\
    0 & f & 0  \\
    0 & 0 & 1
    \end{bmatrix}
    \begin{bmatrix}
    X/Z  \\
    Y/Z  \\
    1
    \end{bmatrix}
    
    $$
    
    Here, $f$ represents the **focal length**, the first and only intrinsic parameter in this model.
    
2. **Translated Principal Point** → if the origin of the image plane is **not aligned with the principal point**, which now lies at $(x_0, y_0)$, a translation must be included in the model. The intrinsic matrix becomes:
    
    $$
    
    \begin{bmatrix}
    x \\
    y \\
    1
    \end{bmatrix}
    \sim
    \begin{bmatrix}
    f & 0 & x_0 \\
    0 & f & y_0 \\
    0 & 0 & 1
    \end{bmatrix}
    \begin{bmatrix}
    X / Z \\
    Y / Z \\
    1
    \end{bmatrix}
    $$
    
    The model introduces **2 additional intrinsic parameter**:  $x_0$ , $y_0$ (principal point coordinates).
    
3. **Non-Square Pixels (Pixel Aspect Ratio)** → in many digital cameras, especially in video, pixels are **not perfectly square**. As a result, the focal length differs along the $x$ and $y$ axes, leading to two values: $f_x$ and $f_y$. These are obtained by scaling a common focal length $f$ by the pixel dimensions $(\alpha_x,\alpha_y)$ → 

$f_x = f \alpha_x,\ f_y = f \alpha_y$. The intrinsic matrix becomes:
    
    $$
    
    \begin{bmatrix}
    x \\
    y \\
    1
    \end{bmatrix}
    \sim
    \begin{bmatrix}
    f_x & 0 & x_0 \\
    0 & f_y & y_0 \\
    0 & 0 & 1
    \end{bmatrix}
    \begin{bmatrix}
    X / Z \\
    Y / Z \\
    1
    \end{bmatrix}
    $$
    
    This model introduces **4 intrinsic parameters**: $f_x$, $f_y$, $x_0$, $y_0$.
    
4. **General Intrinsic Matrix with Skew** → the most general model also includes the **skew** parameter $s$, which accounts for the non-orthogonality between the camera’s $x$ and $y$ axes. While skew is usually negligible in modern digital cameras (often set to $0$), the full intrinsic matrix is:
    
    $$
    
    K =
    \begin{bmatrix}
    f_x & s & x_c \\
    0 & f_y & y_c \\
    0 & 0 & 1
    \end{bmatrix}
    
    $$
    
    This model introduces **5 intrinsic parameters**: $f_x$, $f_y$, $s$, $x_c$, $y_c$.
    
    **N.B.** This general model assumes **no lens distortion**, which is a common simplification in theoretical analysis.
    

### 3.3.5. Camera Calibration Approaches

1. **Multi-Image-Chessboard Method** → It estimates intrinsic and extrinsic parameters capturing multiple images of a known calibration pattern (e.g., a chessboard) and extracting the corner points (e.g. Zhang’s Method).
    
    <aside>
    
    **Why 2 images with 4 points each are enough**
    
    Assuming skew $s = 0$, there are $4$ intrinsic parameters and $6$ extrinsic parameters ($3$ angles, $3$ translations). For $N$ images with $M$ corners each:
    
    - Number of parameters: $4 + 6N$
    - Number of constraints: $2NM$ (2 coordinates for each of $M$ corners in $N$  images)
    
    We need $2NM ≥4 + 6N$, which implies $N≥ \frac{2}{M−3}$ . Thus, $2$ images with $4$ points could be enough, but over-constrained solutions with more images are typically used for better accuracy.
    
    ![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%207.png)
    
    </aside>
    
2. **Using Different Geometric Structures of the Scene** → this approach uses known **geometric constraints** in the scene, such as:
    - **Parallel lines** (leading to vanishing points)
    - **Orthogonality assumptions**
    
    to estimate calibration parameters from a single image.
    
    Limitation: Only applicable to structured, man-made environments.
    
    <aside>
    📌
    
    **Direct Linear Transformation - DLT**
    
    The **Direct Linear Transformation (DLT)** method estimates the camera projection matrix $M \in \mathbb{R}^{3 \times 4}$ using a set of known **3D points** ($X_i, Y_i, Z_i$) and their corresponding **2D image points** ($u_i$, $v_i$). The projection equation is:
    
    $$
    
    \lambda_i
    \begin{bmatrix}
    u_i \\
    v_i \\
    1
    \end{bmatrix}
    =
    M \cdot
    \begin{bmatrix}
    X_i \\
    Y_i \\
    Z_i \\
    1
    \end{bmatrix} = 
    
    \begin{bmatrix}m_{00} & m_{01} & m_{02} & m_{03} \\m_{10} & m_{11} & m_{12} & m_{13} \\m_{20} & m_{21} & m_{22} & 1\end{bmatrix}\cdot \begin{bmatrix}
    X_i \\
    Y_i \\
    Z_i \\
    1
    \end{bmatrix}
    
    \quad \text{with } \lambda \in \mathbb{R}
    $$
    
    $M$ has **12 elements**, but only **11 degrees of freedom**, because it is defined up to a scale factor. After eliminating the scale factor $\lambda$, we obtain 2 equation per point:
    
    $$
    
    \begin{aligned}
    u_i(m_{20}X_i + m_{21}Y_i + m_{22}Z_i + m_{23}) &= m_{00}X_i + m_{01}Y_i + m_{02}Z_i + m_{03} \\
    v_i(m_{20}X_i + m_{21}Y_i + m_{22}Z_i + m_{23}) &= m_{10}X_i + m_{11}Y_i + m_{12}Z_i + m_{13}
    \end{aligned}
    $$
    
    These can be written as a linear system $Am = 0$, where $m$ is a vector containing the 12 elements of $M$:
    
    $$
    
    \begin{bmatrix}
    X_i & Y_i & Z_i & 1 & 0 & 0 & 0 & 0 & -u_i X_i & -u_i Y_i & -u_i Z_i & -u_i \\
    0 & 0 & 0 & 0 & X_i & Y_i & Z_i & 1 & -v_i X_i & -v_i Y_i & -v_i Z_i & -v_i
    \end{bmatrix}m=0\quad \text{with}\ 
    m =
    \begin{bmatrix} m_{00} & m_{01} & m_{02} & m_{03} & m_{10} & m_{11} & m_{12} & m_{13} & m_{20} & m_{21} & m_{22} & m_{23} \end{bmatrix}^T
    
    $$
    
    Since each 3D–2D correspondence provides **2 equations**, with at least **6 non-coplanar points**, the matrix $M$ can be solved using **least-squares techniques**.
    
    ---
    
    **Non linear optimization approach**
    
    After obtaining an initial estimate of $M$ via DLT, we can refine it using **non-linear optimization** to minimize the **reprojection error**:
    
    $$
    
    \sum_{i=1}^{N} 
    \left( u_i - \frac{m_{00}X_i + m_{01}Y_i + m_{02}Z_i + m_{03}}{m_{20}X_i + m_{21}Y_i + m_{22}Z_i + 1} \right) ^2
    + 
    \left( v_i - \frac{m_{10}X_i + m_{11}Y_i + m_{12}Z_i + m_{13}}{m_{20}X_i + m_{21}Y_i + m_{22}Z_i + 1}\right)^2
    $$
    
    where $u_i$ are the observed 2D image points and $(X_i,Y_i,Z_i)$ are the 3D world points.
    
    **N.B.** A **practical issue** with 3D calibration is that measuring 3D feature positions accurately can be difficult.
    
    </aside>
    
3. **Camera Self-Calibration** → estimates the intrinsic camera parameters from **video sequences** captured with a moving camera, using **epipolar geometry** and **corresponding feature points**, and can also estimate lens distortion.
4. **Deep Learning–Based Calibration** → recent learning-based methods aim to estimate camera parameters from single images. 
    
    Limitations:
    
    - Require **large training datasets**
    - Typically estimate **incomplete calibration** (e.g., only $f_x$ or only $f_y$ orientation)
    - Accuracy not yet comparable to classical geometric methods

---

## 3.4. Distortion

An important intrinsic parameter in camera calibration is **lens distortion**, particularly **radial distortion.** Radial distortion is caused by imperfections in the lens and happens when light rays bend more as they move away from the center of the lens. There are **several forms** in which it can appear:

- **Barrel Distortion:** common in wide-angle lenses, where lines appear to bulge outward (with $k_1 > 0$).
- **Pincushion Distortion:** seen in telephoto lenses, where lines appear to pinch inward (with $k_1<0$).

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%208.png)

<aside>
🔢

**Modeling Distortion**

Distortion can be modeled mathematically in the following way: $\begin{pmatrix}x'\\y'\end{pmatrix} =  L(r) \cdot \begin{pmatrix}x\\y\end{pmatrix} + \begin{pmatrix}dx\\dy\end{pmatrix}$

Where:

- $L(r)$ is the **radial distortion** function, which depends on the distance $r$ of a pixel from the image center: $L(r) = r \left(1 + k_1 r^2 + k_2 r^4 \right)$. Where $k_1$ and $k_2$ are the **radial distortion coefficients**. These, along with other parameters (e.g., tangential distortion terms), are typically estimated using **non-linear least squares**.
- $\begin{pmatrix}dx\\dy\end{pmatrix}$ represents **tangential distortion**, which depends on the distance of the pixel from the center of the image.

![image.png](5-Elements%20of%202D%203D%20Geometry%20and%20Cameral%20Model/image%209.png)

</aside>

---