# 7-The Sound


<aside>
💡

Sound is considered a cyclic variation of pressure that reaches our ears, and it requires a medium to propagate, such as air, water, and so on.

</aside>

But how does this variation travel through the air? It takes advantage of the multiple collisions between its particles. This is how loudspeakers work: their membrane moves back and forth, **compressing** and **decompressing** the air in front of them. Obviously, this movement is not instantaneous for all the air particles in front of the speaker; depending on certain physical properties, these variations will be perceived at a certain distance after a given amount of time → about 344 meters in one second.

![image.png](7-The%20Sound/image.png)

Obviously, it won’t be the same particle that has to travel the whole distance; in fact, it will move only until it collides with the next one, and so on. For this reason, after a certain distance, the pressure variation becomes so faint that it is no longer perceptible.

What makes sound audible is the fact that, right after starting to push some particles forward, others are pulled back, causing a decompression; this structure can be seen as a wave that changes its pressure value over time.

![image.png](7-The%20Sound/image%201.png)

---

# 7.1. Sound Waves

Sound waves are variations in pressure that travel through space as a sequence of compressions and rarefactions (decompressions). This movement carries energy outward from the source. However, the particles of the medium through which the wave propagates do not move permanently; instead, they oscillate back and forth around a fixed average position.

Sound waves are therefore called **longitudinal waves**, because the compressions and rarefactions occur in the same direction as the wave’s propagation.

<aside>
📌

### Properties

Thanks to the **Fourier Transform**, we can describe a sound as a combination of sinusoids. A single sinusoid can be described by the following equation:

$$
y = Asin(2\pi ft)
$$

Where:

- $A$ is the **amplitude** of the wave → it represents the amount of motion, or how much the air is being pushed and pulled. It gives us a sense of the **intensity** or **loudness** of the sound.
- $\sin(2\pi f t)$ describes the wave’s periodic behavior:
    - The $2\pi$ ensures that the wave completes one full cycle per second for a frequency of $1\ Hz$.
    - $f$ is the **frequency**, which tells us how many complete cycles occur per second. Frequency governs our **perception of pitch**: higher frequencies sound higher in pitch, while lower frequencies sound deeper. **Frequency** is measured in **Hertz ($Hz$)**, which has the physical unit $s^{-1}$, in other words, 1 Hz = 1 cycle per second.

### Other properties

- **Wave Length →** refers to the distance, measured in meters, that corresponds to one **period** of a wave. In other words, the space between two consecutive peaks (or troughs). It is given by the following formula:
    
    $$
    \lambda = \frac{c}{f}
    $$
    
    Where:
    
    - **$\lambda$** is the wavelength, in meters (m)
    - $c$ is the speed of sound in the medium
    - $f$ is the frequency, in Hertz (Hz)
    
    As we can see, everything depends on the **speed of sound** in the propagation medium. For example, in air, the speed of sound is about **344 m/s**, so a wave with a frequency of **1 Hz** will travel **344 meters** in one second.
    
    **N.B.** The human ear starts perceiving sound from around **20/30 Hz**, which corresponds to wavelengths of about **17 to 11 meters**.
    
- **Phase** → is the time shift between two waves of the same frequency — it tells us how much one wave is delayed compared to another. The **phase difference** is the angular distance between two points rotating at the same speed (same frequency) but starting from different positions. It’s measured in **radians** and given by:
    
    $$
    
    \Phi = 2\pi f \Delta t
    $$
    
    **N.B.** This property is essential for phenomena such as **sound amplification** and is at the core of **resonance** in musical instruments.
    
</aside>

---

# 7.2. Sound Pressure

What we care about in describing sound is **pressure**, since sound is a variation in **sound pressure**. Even in silence, we are always under pressure  (atmospheric pressure ~10⁵ Pa) but we don’t notice it because it compresses us **equally in all directions**. We define **silence** (zero sound) as a condition where there’s **no variation** from atmospheric pressure.

When a sound occurs, pressure **changes rapidly**, so to describe how strong the pressure is over time, we use the **RMS (Root Mean Square)**, which gives us the **Effective Sound Pressure** p → The **higher** the RMS value, the **louder** the sound.

However, our **perception of loudness** is not linear, so instead of using $p$ directly, we define the **Sound Pressure Level (SPL)** $L_p$, which is a **logarithmic** measure:

$$
L_p = 20 \log_{10} \left( \frac{p}{p_0} \right) \quad [\text{dB}]
$$

Where:

- $p$ is the effective sound pressure (RMS)
- $p_0$ is the reference sound pressure, about  $20 \mu Pa$, which is the **threshold of human hearing**

<aside>
📌

**SPL examples**

![image.png](7-The%20Sound/image%202.png)

</aside>

---

# 7.3. Microphones

To measure sound, we need to convert **acoustic energy** into **electrical energy**. Microphones do exactly that. There are two common types:

- **Electrodynamic Microphone →** works through the motion of a **diaphragm** attached to a **magnet** or a **coil**. When sound waves move the diaphragm, a magnetic element moves inside a **coil of wire**, generating a **voltage** via electromagnetic induction.
    
    These devices are very **resistant**, making them perfect for capturing **low kicks from drums** or very loud sounds. By construction, they are well suited for capturing **not too fast** vibrations, since they are made of pieces that must move **physically**, being subject to the laws of **momentum**, which limits the speed at which we can change their direction of motion. Therefore, they are suitable for **loud** but **limited-frequency** sounds.
    

![image.png](7-The%20Sound/image%203.png)

- **Condenser Microphone (Electrostatic Microphone) →** uses a **capacitor** instead of a moving electromagnetic part. The **diaphragm** and a **backplate** form a capacitor. When sound moves the diaphragm, the **capacitance C** changes, and since:
    
    $$
    V = \frac{Q}{C}
    $$
    
    a change in **C** causes a change in **voltage V**.
    
    We mentioned that we first need to apply a voltage to the capacitor, called **Phantom Power**. This type of microphone is **very sensitive** and reproduces sound **faithfully**, but care must be taken because they are **not as durable** as dynamic ones.
    
    <aside>
    💡
    
    **Polar Patterns**
    
    Regardless of the type of microphone, each of them has what is called a Polar Pattern, which indicates the areas from which the microphone is able to capture sound:
    
    ![image.png](7-The%20Sound/image%204.png)
    
    ![image.png](7-The%20Sound/image%205.png)
    
    </aside>
    
    <aside>
    🚨
    
    **Shotgun Microphones**
    
    These specialized microphones feature a diaphragm at the end of a slitted tube. Sounds not originating from the pointing direction enter through the slits and undergo reflections that largely cancel each other out. Sounds from the intended direction pass unimpeded. This design allows shotgun microphones to capture specific sound sources, even at considerable distances.
    
    ![image.png](7-The%20Sound/image%206.png)
    
    </aside>
    

## 7.3.1. **Microphones Parameters**

- **Sensitivity**: its the ability to convert sound into electricity. If the voltage variation produced is small, it must be amplified, which also results in amplification of any errors.
    
    To measure this parameter, a $1\ kHz$ tone with an SPL of $92\ dB$ is generated at a distance of 1 meter. The microphone is placed in front of the source, and the voltage produced (in mV) is measured, then converted into **dBV** using the following formula:
    
    $$
    Sensitivity_{dBV}= 10log_{10}(\frac{Sensitivity_{mV/Pa}}{1000mV/Pa})\quad [dBV]
    $$
    
    **N.B.** Since it is impossible to exceed the reference value in the denominator, this will always result in a negative value.
    
- **Frequency Response**: a microphone's sensitivity is not constant across all frequencies. A **Bode plot**, with a logarithmic frequency axis and a $dB$ vertical axis, is used to describe its behavior. The sensitivity at $1\ kHz$ is typically set as the $0\ dB$ reference. The human ear generally doesn't perceive sounds outside the $20-20000\ Hz$ range.
    
    ![image.png](7-The%20Sound/image%207.png)
    

---

# 7.4. Sound Reproduction

Loudspeakers convert electrical signals into acoustic waves. They operate using a **coil attached to a membrane** placed within a **circular magnet**. When an electrical signal passes through the coil, it creates a magnetic field that interacts with the magnet’s field, causing the membrane to move. A **positive half-wave** pushes the membrane forward, while a **negative half-wave** pulls it back.

## 7.4.1. **Loudspeaker Characteristics**

- **Resonant Frequency**: When the input frequency approaches the system’s natural resonance, **undesirable 180° phase shifts** can occur, distorting the audio signal.
- **Efficiency**: Indicates how well electrical energy is converted into sound. Efficiency is typically **low (1–8%)**, since much energy is lost as heat. **Cone-shaped membranes** are more effective at low frequencies than flat ones.
- **Sensitivity**: Measured in dB SPL, it reflects the sound pressure level at 1 meter with 1 Watt input.
- **Maximum Power**:
    - Peak Power → maximum instantaneous power the speaker can handle without damage.
    - RMS Power → average power it can handle over time before overheating.
- **Loudspeaker Types** (based on membrane size and mass, affecting resonant frequency):
    - Woofers: For low frequencies, move large volumes of air
    - Subwoofers: For very low frequencies (20–40 Hz)
    - Midrange: For mid frequencies
    - Tweeters: Small membranes for high frequencies

---

# 7.5. Digital Audio

Digital audio converts continuous analog signals into discrete digital signals through **sampling**: values are taken at regular time intervals (sampling frequency). 

First of all, we recall **Nyquist’s Theorem**, which states that the **sampling frequency must be at least twice the bandwidth of the signal**, usually 44 kHz. To have a bit more flexibility, we typically use **44,100 Hz**. If we **don’t follow this rule**, we get **aliasing**, which means **false or distorted sounds**.

![image.png](7-The%20Sound/image%208.png)

---