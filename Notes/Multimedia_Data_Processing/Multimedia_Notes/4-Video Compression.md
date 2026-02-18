# 4-Video Compression

From a technical point of view, the video is a sequence of images, obtained by temporal sampling.

Each country has its own standard; in Europe, the goal is to transmit **25 frames per second**. Since encoding any video, no matter how short or low in quality, into a single stream would require an **unsustainable amount of memory**, new techniques had to be developed to **optimize the process**.

# 4.1 Type of video encoding

## 1. **INTRA Frame Encoding**

In this technique, each frame is compressed **independently** from the others, meaning no external information is used beyond what is contained within the frame itself. Even so, despite the significant reduction in size, the **memory requirements remain unsustainable**.

This type of encoding is known as **M-JPEG (Motion JPEG)**.

## **2. INTER Frame Encoding**

It uses the **previous frame to predict the current one.** 

<aside>
⏰

**Time Difference**

Since it is very likely that there are small variations from one frame to the next, we can predict the values of the upcoming pixels based on the current ones. Obviously, we can make errors, so we need to add to the prediction the difference between the two values. To perform the encoding, it will then be necessary to have only the difference between the two frames, since having the first frame we can find the second through this difference.

To **visualize** this difference, we create a **difference image**. Since pixel values range from $0$ to $255$, their differences span $[-255, 255]$. To map this to a visible range $[0, 255]$, we use:

$$
\text{diff}(x, y) = \left\lfloor \frac{\text{B}(x, y) - \text{A}(x, y)}{2} \right\rfloor + 127
$$

- **127** represents no change,
- **<127** means a decrease,
- **>127** means an increase.

**Why is useful?**

- The distribution of difference values is often **narrow and skewed**, meaning most differences are close to zero.
- The **entropy** of such a distribution can be as low as **4.06 bits**, allowing efficient **lossless compression** through variable-length coding.

![image.png](4-Video%20Compression/image.png)

---

Example:

![Frame A](4-Video%20Compression/image%201.png)

Frame A

![Frame B](4-Video%20Compression/image%202.png)

Frame B

![Difference Image](4-Video%20Compression/image%203.png)

Difference Image

</aside>

<aside>
🏃🏻

**The motion:**

So far, we have seen how two frames captured by a fixed camera on static objects turn out to be very similar to each other. Things change quite a bit if the camera starts to move: the introduction of motion causes our prediction to be wrong. The same happens if the object moves in the scene instead of the camera.

We need a way to improve our forecast: we could send how much each pixel moves from one frame to the next, but that would require more data than before and therefore doesn’t make sense.

The first solution was to send **motion vectors** for groups of pixels: we take a set of pixels that move in the same way and send information about where they were previously. The chosen group size was 16 × 16 and was called a **macroblock**. (16 × 16 was chosen to relate to JPEG, which uses 8 × 8 blocks.

---

**Motion Vector:**

We used the term **motion vector**, but what exactly is it? It is a pair of numbers $(x,y)$ that indicates where the given point was located in the previous image (in reality, it points to the most similar position). To measure similarity, we need to compare all the blocks present in the two frames with each other, which is a computationally heavy operation if we use a brute force approach.

Fortunately, there are approximations and heuristics to perform this operation practically in real time.

We said that having the previous frame and the motion vector, we can reconstruct the image composed of the macroblocks from that previous frame (= **Prediceted image**); obviously, however, the vectors we use also take up space, so we need to find a compromise.

**SAD: Sum of Absolute Differences** → With this technique, we consider the information provided by the motion vector only if the SAD of the block in question is greater than a certain threshold: we only keep the vectors related to large movements.

---

Example:

![Frame A](4-Video%20Compression/image%204.png)

Frame A

![Frame B](4-Video%20Compression/image%205.png)

Frame B

![Difference Image](4-Video%20Compression/image%206.png)

Difference Image

![Motion Vector of B Compared to A](4-Video%20Compression/image%207.png)

Motion Vector of B Compared to A

![Predicted Image](4-Video%20Compression/image%208.png)

Predicted Image

</aside>

<aside>
🔑

**Lossy Compression**

1. The real gain happens when we discard part of the information; it is not necessary to send the entire difference between two frames but something similar is enough. Just like in JPEG, we use the DCT followed by quantization to understand which parts of the image are useful to keep. Depending on the division we perform, we can obtain different types of resolution.
2. As we can see, something is wrong; in fact, there are some connections that we practically cannot have. When we reach the end of the second frame, at point $\~{x_2}$, we should use information coming from the original frame, which is not available at the decoder. What happens, then, is that since we don’t have $x_1$ but only its reconstruction.
3. However, it is easy to notice that the further we go, the more errors accumulate, causing our image to become very noisy after just a few frames. This phenomenon is called **error propagation**, and we therefore need to find a solution. The problem is that, since we don’t have $x_1$, we use two different references between the upper and lower parts of the scheme. This is what needs to be fixed: we must use the same reference in both cases.

![1)](4-Video%20Compression/image%209.png)

1)

![2)](4-Video%20Compression/image%2010.png)

2)

![3)](4-Video%20Compression/image%2011.png)

3)

</aside>

---

# 4.2 Reference Schema for an Encoder

Before going into the specifics of the first standard, let’s look at a general scheme of an encoder:

![image.png](4-Video%20Compression/d53d7e59-9da5-4bc6-bfa7-19dc397b37f4.png)

We notice that the INTRA Frame coding part is not shown, but at least for the first frame we need it, or it is useful during transmission errors or other special cases.

---