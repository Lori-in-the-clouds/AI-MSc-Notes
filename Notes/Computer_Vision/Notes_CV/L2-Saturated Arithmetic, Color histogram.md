# L2-Saturated Arithmetic, Color histogram

Done?: Done
Select: lab

# 1. Working with images

Images are typically stored in compressed formats (JPG, PNG, BMP). Compression can be **lossy** (e.g., JPG) or **lossless** (e.g., PNG).

<aside>
💡

**Image representation**

- Grayscale images are stored as $(H, W)$ tensors.
- Color images are stored as $(3,H, W)$ tensors, where the first axis represents color channels (RGB or BGR).

**N.B.** Pixel values are usually **8-bit unsigned integers** (0-255) but are often converted to **32-bit floats** for processing (e.g. Deep learning). In image processing we alway must clamp values between 0 and 255.

</aside>

---

# 2. Reading Images with OpenCV

To read (and write) images, we will use the routines implemented inside OpenCV, a very popular Computer Vision library:

- **`cv2.imread(path, cv2.IMREAD_GRAYSCALE)`**: reads a grayscale image as an $(H, W)$ numpy array.
- **`cv2.imread(path, cv2.IMREAD_COLOR)`**: reads a color image as an $(H, W, 3)$ numpy array (BGR format).
- **`im.swapaxes(0,2)`**: to convert $(H,W,3)$ ****images to the common $(3, H, W)$ format.
- To display images using matplotlib, they need to be converted to **RGB** format.

![image.png](L2-Saturated%20Arithmetic,%20Color%20histogram/image.png)

---

# 3. Compute a histogram

For computing a histrogram you can use:

- **`hist, bin_edges = torch.histogram(input, bins=10, range=None, weight=None, density=False)`** → Returns **`hist`**(=tensor containing the number of elements in each bin) and **`bin_edges`** (the edges defining the bin ranges).
- **`hist = torch.histc(input, bins=10, min=0, max=255)`**→ Return only the histogram counts, not the bin edges. Requires specifying a fixed range (min and max), unlike **`torch.histogram`** which can determine this automatically.

---

# 4. Compute a normalized histrogram

- Compute a histrogram
- Divide each bin with the total number of pixels in the image: $p_i = \frac{\#I(x)=i}{n}$

**N.B.** The **normalized histogram** represents a **probability distribution**, meaning that the sum of all probabilities equals 1: $\sum_{i=0}^{255}p_i=1$

---