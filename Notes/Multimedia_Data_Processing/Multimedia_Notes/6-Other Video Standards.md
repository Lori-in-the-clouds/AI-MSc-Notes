# 6-Other Video Standards

After examining the H.261 standard in detail, let’s now take a look at other video encoding algorithms that are commonly used.

# **6.1 MPEG Standard**

The first version of this standard, **MPEG-1**, was designed for Video CDs and aimed to simulate VHS quality. It was intended for **progressive frames only** and did **not support interlaced video**, which is typical in television applications.

For this reason, **MPEG-2** was introduced shortly after. It proved to be much more effective and supported a **bitrate range from 4 to 9 MB/s**, compared to the **1.5 MB/s** of **MPEG-1**.

## 6.1.1 Differences with H.261

The first fundamental concept is the **key frame**, which allows the transmission to restart as if it were the beginning of the video. This is essential for **broadcasting**, enabling viewers to join the stream at any time. These frames are called **I-frames**, where the “I” stands for **INTRA**—they are encoded independently, without references to previous frames, allowing decoding from any point in the stream. The other type of frame is the **P-frame**, where “P” stands for **Predicted**. These frames contain data that references to the previously transmitted frames.

Thanks to the progress made with JPEG, **quantization** is now applied using two distinct quantization matrices:

![image.png](6-Other%20Video%20Standards/image.png)

As we can see, for blocks in **INTRA mode**, it is not a good idea to maintain the same quantization level throughout the entire image. On the other hand, for **INTER blocks**, no compelling reason was found to vary the quantization values, so a much simpler approach was adopted.

Another important point is that the **amount of quantization** can be adjusted using a **scaling factor** in the range $(0, 1]$.

## 6.1.2 MPEG Novelties

Based on what has been said so far, we are in a situation similar to the following:

![image.png](6-Other%20Video%20Standards/image%201.png)

Unfortunately, there are cases, in fact, most cases, where this method of prediction does not work, since it is very likely that we have no information about certain parts of the previous frame. However, if we look at the following frame, that information becomes available.

Therefore, **B-frames (*Bidirectional Frames*)** are introduced. These perform a search using both past and future references. Note that these references don’t necessarily have to be the immediately preceding or following frames, but can be any previous and any subsequent frame.

![image.png](6-Other%20Video%20Standards/image%202.png)

<aside>

**Video Layer and B-Frames**

A video in the MPEG standard is divided into a series of layers designed to perform and manage various tasks. The first layer is called the **Video Sequence Layer**, which represents a complete video, independent of what comes before or after. The second layer is the **GOP (Group of Pictures)**, which consists of a combination of $I$, $B$, and $P$ frames.

Next, we have the **Picture Layer**, where each picture is divided into **slices**, and each slice is a sequence of **macroblocks**, which in turn are made up of **blocks**, very similar to the structure used in the H.261 standard.

Each level has its own **32-bit byte-aligned start code**.

By introducing forward prediction, things become significantly more complex. Let’s look at this example:

![image.png](6-Other%20Video%20Standards/image%203.png)

Since each frame is of a different type, before decoding frames 1 and 2, I will need to work on frame 3 first, because $B$-frames require information from the following $P$
-frame.
This introduces a delay and also requires memory within the devices, at a time when hardware wasn’t as powerful as it is today.
What was done, then, was to change the order in which the frames were transmitted:

![image.png](6-Other%20Video%20Standards/image%204.png)

In this way, we can start decoding as early as possible while still receiving information; an interesting but very complex idea. The **problem** with this approach is, as we can see, that some frames belonging to $GOP_0$ become part of $GOP_1$.

</aside>

<aside>

**Bidirectional Prediction in B-Frames**

To make things even more complicated, it is possible to calculate the difference with respect to two different references, one in the past and one in the future → **2 motion vectors**:

![image.png](6-Other%20Video%20Standards/image%205.png)

</aside>

## 6.1.3. More Differences with H.261

1. In H.261, motion estimation was performed using only the previous frame. In more recent standards, it is possible to use reference frames that are further away in time. However, as the temporal distance between frames increases, the likelihood of larger motion in the image also increases. As a result, the motion search range needs to be expanded, for example from ±15 to ±32 pixels.
2. Motion vectors (MVs) are no longer limited to integer-pixel displacements, they can now indicate motion at half-pixel precision (e.g., 5.5, 7.0, 3.5…).
With half-pixel accuracy, motion prediction becomes more precise, which reduces the prediction error (residual) and improves compression efficiency. **However, half a pixel doesn’t physically exist!** → Therefore, we need to interpolate between the nearest pixels to obtain an intermediate value, ypically using a weighted average or similar interpolation method.
3. A positive consequence of the way frames are organized is that video can be navigated more easily. I-frames (intra-coded frames) act like complete, self-contained images. This means we can jump from one I-frame to the next to perform fast-forward operations without needing to decode all the intermediate frames. As a result, no additional hardware power is required, since there’s no need to reconstruct the entire sequence.

---

# 6.2 H.264/AVC

Here is a list of what has been introduced in this new standard:

- One of the first things to note is the added support for encoding **INTRA pictures**.
- Smaller blocks have been introduced to replace the fixed-size 16 × 16 macroblocks.
- While previously we had half-pixel precision, here we reach **quarter-pixel precision** through weighted interpolation.
- **Motion vectors** can now go beyond the boundaries of the frame, allowing better prediction for objects entering the scene.
- **Weighted prediction** has been introduced, assigning different importance to past and future frames.
- The traditional **8 × 8 DCT** has been replaced by a slightly different **4 × 4 transform**.
- **Arithmetic entropy coding** is used instead of Huffman coding.

## 6.2.1 Prediction for INTRA Pictures

**Intra prediction** aims to predict the texture of the current block by using pixel samples from **neighboring blocks**. In **H.264**, intra prediction is supported for both **4×4** and **16×16** block sizes.

![image.png](6-Other%20Video%20Standards/image%206.png)

**N.B**. Intra prediction for 16×16 blocks uses fewer methods than for 4×4 blocks because larger blocks typically have simpler textures and require less detailed prediction.

![4×4 prediction](6-Other%20Video%20Standards/image%207.png)

4×4 prediction

![16×16 prediction](6-Other%20Video%20Standards/image%208.png)

16×16 prediction

## 6.2.2 Variable Block Size

In H.264, there are four different possible block sizes:

- 16 × 16
- 16 × 8
- 8 × 16
- 8 × 8

How do we choose which size to use?

The idea is to start with the largest block size and calculate a variance measure with respect to the corresponding block in the previous frame. Then, we try a horizontal subdivision. If the variance in one of the two parts is significantly lower, this separation makes sense, and we proceed accordingly:

![image.png](6-Other%20Video%20Standards/image%209.png)

We observe that in static parts of the image, the block size remains the classic 16 × 16, while in other areas, we perform a finer subdivision to improve prediction.

Where we use 8 × 8 blocks, we can further partition them into:

- 8 × 4
- 4 × 8
- 4 × 4

Applying the same criterion as before, we note that at most we can have 16 different motion vectors if we select the 4 × 4 mode. This implies more vectors to send, so we must carefully balance the block size with the actual improvement gained.

## 6.2.3. Different DTC for Smaller Size

Since our block size can be as small as 4 × 4, we need to find a way to apply the DCT to this new dimension. There is a very good approximation that allows us to use integer weights:

$$

H = \begin{bmatrix}
1 & 1 & 1 & 1 \\
2 & 1 & -1 & -2 \\
1 & -1 & -1 & 1 \\
1 & -2 & 2 & -1
\end{bmatrix}
$$

The fractional parts have been shifted into a post-scaling matrix, so that the fundamental operations can be performed using only integers, making the transform very fast. . The post-scaling matrix is applied afterward to correct for the approximations introduced by using integer weights, ensuring precision is maintained.

To apply this to larger blocks, we can first divide them into sub-blocks, apply the transform, take the DCTs of each mini-block, and then recompose a new matrix.

---