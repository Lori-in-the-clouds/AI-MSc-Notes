# 5-H.261 Standard


Introduced in 1990, **H.261** was the first modern standard for video compression. Its original title is “*Video Codec for Audiovisual Services at p×64 kbit/s”*, where “p×64 kbit/s” refers to **ISDN** transmission lines.

<aside>
💡

**ISDN characteristics**

- Each ISDN line allows **bidirectional traffic** at **64 kbps per channel**. By combining multiple lines in parallel (up to 30), higher bitrates could be achieved.
- **No byte alignment/no reference points for random file access**, since it was designed for **real-time communication**, not for file storage.
- Supports **two different frame sizes**:
    - CIF (Common Intermediate Format): 352 × 288 pixels
    - QCIF (Quarter CIF): 176 × 144 pixels
- Uses the $YC_bC_r$ color space:
    - **Y** (luminance): ranges from **black (16)** to **white (235)**
    - **Chrominance** ($C_b$, $C_r$): values range from **16 to 240**
    - Subsampling: for every **16×16 block** of luminance (**Y**), there is **one 8×8 block** of $C_b$ and one of $C_r$
</aside>

# 5.1. H.261 Structure

Below is a diagram representing the encoder structure for the **H.261** standard:

![Intra Encoding Path](5-H%20261%20Standard/image.png)

Intra Encoding Path

![Inter Encoding Path](5-H%20261%20Standard/image%201.png)

Inter Encoding Path

<aside>
🚨

The fundamental unit of this standard is not the frame but the **macroblock**. In fact, there are no actual INTRA or INTER frames; instead, **within the same frame**, some blocks can be transmitted as **INTRA**, while others as **INTER.**

</aside>

Now, let’s analyse the blocks:

- **Selectors**: allow selections to be made on the input (INTER / INTRA).
- **$T$, $T^{-1}$**: these are the transform and inverse transform operations.
- **$Q$, $Q^{-1}$**: represent the quantization and dequantization stages.
- **$P$, $F$**: used to generate the predicted image and apply filtering to slightly correct reconstruction errors. The **low-pass filter** aims to smooth variations within the block and is similar to a Gaussian filter, but simplified. It has the following form:
    
    $$
    
    \begin{bmatrix}
    1/16 & 1/8 & 1/16 \\
    1/8  & 1/4 & 1/8  \\
    1/16 & 1/8 & 1/16
    \end{bmatrix}
    $$
    
    Applying this filter reduces the entropy by a small amount, nothing exceptional, but still helpful.
    

<aside>
📌

**Additional Mechanism Rules:**

1. It is possible to skip some frames during transmission. In particular, up to **3 consecutive frames** can be skipped, meaning that, out of 4 real-time frames, only 1 may be transmitted. This mechanism allows for **framerate reduction**.
2. To prevent excessive quality degradation, each macroblock must be transmitted as **INTRA** at least once every 132 times it is sent. There are no strict rules on when to transmit **INTER** or **INTRA** frames, or when to skip frames, this decision is left to the programmer. This flexibility encourages future improvements in the standard and promotes **competition between companies**.
</aside>

---

# 5.2. Bitstream Structure (most significant bit first)

There are 4 hierarchical levels within the bitstream, each identified by a specific header that allows us to determine which section of the structure we are in:

1. **Picture:** 
    1. The **start code (PSC)** consists of **20 bits**: $0000\ 0000\ 0000\ 0001\ 0000$
    2. **Temporal Reference (TR)**, which identifies the frame number. It’s incremented with each new frame and is useful for handling **frame losses** or **intentional frame skipping**
    3. **Type Information (PTYPE),** it’s composed of 6 bits, but only **bit 4** is relevant, indicating whether the image is in **QCIF** (0) or **CIF** (1) format (each frame can have different quality for communication performance).
    4. **Extra Insertion Information (PEI)** is composed of **1 bit**, which specifies whether an additional byte is present in the header. If the bit is set to **1**, the additional byte is called **PSUPP** (Payload Supplement), and it is followed by another **PEI** bit. This mechanism was introduced to allow programmers to add extra information in a flexible and extensible way.
2. **Group of Blocks:** each **Picture** is divided into:
    
    
    - **12 GOBs** in the case of **CIF**
    - **3 GOBs** in the case of **QCIF** (1,3,5)
    
    Each GOB is represented as:
    
    1. **16-bit Group of blocks start code (GBSC): $0000\ 0000\ 0000\ 0001$**
    2. A **4-bit value** called **Group Number (GN)**, which identifies the GOB index within the picture (useful for skipping specific GOBs), in particular:
        - 1-12 represnet GOB number
        - 13-15 are never used
    3. A **5-bit field** called **Quantizer Information (GQUANT)**, indicating the quantization parameter used for all macroblocks within the GOB
    4. A **1-bit flag** called **Extra Insertion Information (GEI)**, similar to PEI, used to signal the presence of an extra byte.
    
    ![image.png](5-H%20261%20Standard/image%202.png)
    
3. **Macroblock:** each **GOB** is divided into **11 × 3 macroblocks**, each of size **16 × 16:**
    
    ![image.png](5-H%20261%20Standard/image%203.png)
    
    All information at this level is encoded in a space-efficient way. Unlike previous layers, which had fixed-size fields for each piece of information, here we need to transmit a large amount of data, so **bit economy** becomes crucial.
    
    Each macroblock is represented as:
    
    1. **MBA (macroblock address):** is encoded using a **variable-length code**, because it is easy to predict which macroblock will come next. Instead of sending the absolute value, **a difference from the previous macroblock** is transmitted: how many macroblocks forward do we need to move?
        - For the **first macroblock** in a GOB, the absolute address is sent
        - For **subsequent macroblocks**, only the **difference** from the previous one is transmitted
        - If we reach the end of the GOB, the stream continues with the next GOB or Picture.
        
        ![image.png](5-H%20261%20Standard/image%204.png)
        
        **N.B.** There is also something called **MBA Stuffing**, a varaibile code length used to insert more data and reduce bitrate.
        
    2. **MTYPE (Macroblock Type Information)** [variable length code], a macroblock can be encoded in the following ways:
        - **Intra** → compressed with DCT coefficients + quantization
            
            ![image.png](5-H%20261%20Standard/image%205.png)
            
            **N.B. MQUANT** is a 5-bit field that represents the quantization level. In this case, it is not a grid (as in JPEG), but a **scalar** value. It is transmitted **only when it changes**.
            
        - **Inter without motion control** → compressed as the difference with the previous frame (prediction), followed by DCT and quantization (there is no motion in the macroblock, so **no motion compensation** is used via motion vectors).
            
            ![image.png](5-H%20261%20Standard/image%206.png)
            
            - **MQUANT** is  transmittend only when it changes.
            - **CBP (Coded Block Pattern)** is a 6-bit field that indicates **which blocks** have been transmitted. Each macroblock consists of **6 blocks** ($4\ Y$,  $1\ C_b$,  $1\ C_r$), and they are transmitted only if they differ from the previous frame.
                
                <aside>
                🔢
                
                The blocks are numbered like this:
                
                ![image.png](5-H%20261%20Standard/image%207.png)
                
                </aside>
                
        - **Inter with motion control** → compressed using the previous frame with motion control.
            
            ![image.png](5-H%20261%20Standard/image%208.png)
            
            Motion control determines **one motion vector per macroblock**. Motion vectors are transmitted as the **difference** with the motion vector of the previous macroblock. This is called **MVD (Motion Vector Difference)**. The idea is that motion vectors of **adjacent macroblocks are similar**, so prediction works well.
            
            <aside>
            
            **Example:**
            
            - Macroblock 1 has **no previous macroblock**.
            - Macroblocks 12 and 23 have a previous macroblock, but it is **not adjacent**.
            - If the previous macroblock is **intra** and has **no motion vector**, then **(0, 0)** is used as the default motion vector.
            
            ![image.png](5-H%20261%20Standard/image%209.png)
            
            </aside>
            
            <aside>
            
            **Example of use:**
            
            *(See the VLC table for MVDs in recap)*
            
            Let’s suppose that the $x$ component of the previous motion vector is $-5$, and we receive a difference encoded as:
             $0000 0111$ → this could represent either $-7$ or $25$. 
            
            However, since $−5 + 25 = 20$, which is outside the valid range  $[−15, +15]$, we deduce that it must be $-7$. → −5 + (−7) = −12, which will be the current value of the $x$ component.
            
            </aside>
            
        - **Inter with motion control + loop filter**
            
            ![image.png](5-H%20261%20Standard/image%2010.png)
            
        
        <aside>
        🔑
        
        **Recap:**
        
        ![image.png](5-H%20261%20Standard/image%2011.png)
        
        ![VLC table for CBPs](5-H%20261%20Standard/image%2012.png)
        
        VLC table for CBPs
        
        ![VLC table for MVDs](5-H%20261%20Standard/image%2013.png)
        
        VLC table for MVDs
        
        </aside>
        
4. **Blocks: w**e have now reached the final level, which contains the actual **DCT coefficients** of the blocks indicated by the **CBP**. These coefficients are encoded following the **zig-zag ordering**. If the macroblock is of type **INTRA**, then **all individual blocks** will be present.
    
    Each **AC coefficient**, whether for **INTER** or **INTRA**, is represented by a pair **(run, level)**:
    
    - **Run** indicates the number of **zeros** before a non-zero coefficient,
    - **Level** represents the **quantized value** of the coefficient.
    
    Below are shown the **most probable (run, level) pairs**:
    
    ![image.png](5-H%20261%20Standard/image%2014.png)
    
    If the pair is **not present in the table**, it will be encoded using **20 bits** in the following format:
    
    $$
    escape: 0000 01 + 6-bit run + 8-bit level
    $$
    
    At the end of each block, a special **10-bit code 10 (EOB – End of Block)** is used to indicate that **all remaining coefficients are zero**. The **first coefficient** in **INTRA blocks** is encoded using **8 bits**, while for **INTER blocks**, the same table used for the other coefficients is applied, with a small optimization: **EOB is not allowed** for the first coefficient (as it’s assumed to be non-zero).
    

---

# 5.3 Quantization

The last missing step is how to reconstruct the original number used for computing the transform, given the **level**. There are two different formulas depending on whether the **quantization level (QUANT)** is **odd or even**:

$$

\text{QUANT Dispari}
\begin{cases}
\text{REC}=\text{QUANT} \cdot (2 \cdot \text{level} + 1) & \text{if } \text{level} > 0 \\
\text{REC}=\text{QUANT} \cdot (2 \cdot \text{level} - 1) & \text{if } \text{level} < 0 \\
0 & \text{if } \text{level} = 0
\end{cases} \newline 

\text{QUANT Pari}
\begin{cases}
\text{REC}=\text{QUANT} \cdot (2 \cdot \text{level} + 1) - 1 & \text{if } \text{level} > 0 \\
\text{REC}=\text{QUANT} \cdot (2 \cdot \text{level} - 1) + 1 & \text{if } \text{level} < 0 \\
0 & \text{if } \text{level} = 0
\end{cases}

$$

*This separation is introduced in order to balance the error introduced toward both negative and positive values.*

---