# L3.3D Convolution

Done?: Done
Select: lab

# 4. Convolution (3D)

While 2D convolution slides a kernel over spatial dimensions $(H,W)$, **3D convolution** slides the kernel over three dimensions: *Height, Width,* and ***Depth** (or Time*).

In 3D convolution, the kernel depth $kD$ is typically smaller than the input depth $iD$, allowing the kernel to slide along the depth axis as well. The operation is defined as:

$$
(V*K)[x,y,z]=\sum_{(i,j,k)\in kernel}V[x+i,y+j,z+k]\cdot K[i,j,k]
$$

## 4.1. Shapes of 3D Convolutional Layer

The tensors now include a dimension $D$:

- **Input shape** → $(n,iC,iD,iH,iW)$
- **Kernel shape** → $(oC,iC,kD,kH,kW)$
- **Output shape** → $(n,oC,oD,oH,oW)$
    
    The output dimensions are calculated similarly to 2D, but with an added formula for the depth axis:
    
    $$
    oD = \left\lfloor \frac{{iD} + 2 \times \text{padding}[1] - \text{dilation}[1] \times (\text{kernel\_size}[1] - 1) - 1}{\text{stride}[1]} \right\rfloor + 1
    $$
    

<aside>
🚨

**Difference between 2D Convolution on Video and 3D Convolution:**

- **2D Convolution on Video** → treats frames as independent images. It captures spatial features but loses temporal relationships (motion) between frames.
- **3D Convolution** → slides over the time axis. It captures **spatiotemporal features**, creating a volume of output features.
</aside>

## 4.2. Parameters (3D)

- **Stride, Padding, Dilation** → they become 3-dimensional tuples (e.g., `stride=(s_depth, s_height, s_width)`)
- **Number of parameters** → $\text{Parameters}=(kD×kH×kW×iC×oC)+oC$

## 4.3. Pooling Layer (3D)

Pooling 3D reduces the spatial and temporal/depth dimensions. It takes an input of size $(n,iC,iD,iH,iW)$ and applies the operation (Max or Avg) over a 3D volume $kD×kH×kW$. The output dimension, but with an added formula for the depth axis:

$$
oD = \frac{(iD-kD)}{s}+1
$$

---