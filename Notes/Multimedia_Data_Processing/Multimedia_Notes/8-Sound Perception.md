# 8-Sound Perception

# 8.1. Human Ear

The human ear acts as a transducer, as it first transforms acoustic energy into mechanical energy and then into electrical energy, so that impulses can be sent to our brain. The auditory system is composed of three main parts:

1. **Outer Ear →** the first part of the ear is called the auricle (pinna), which reflects the sound waves it captures to direct them towards the center, into the ear canal.
2. **Middle Ear →** the ear canal ends at a membrane called the eardrum (tympanic membrane), which vibrates according to the vibrations it receives from outside. On the opposite side, there are three small bones: the hammer (malleus), anvil (incus), and stirrup (stapes), which serve to amplify the vibrations of the eardrum and then transmit them to the cochlea. This amplification is necessary because the eardrum is a thin membrane in contact with air, while the cochlea is filled with a dense fluid.
3. **Inner Ear →** each part of the cochlea resonates more strongly to a specific frequency, which is then converted into electrical energy thanks to the Organ of **Corti**. Inside it are about 4,000 “hair cells” that vibrate according to the fluid movement; these hair cells are grouped (each group is not uniform), and each is connected to its respective nerve endings. This division leads to the concept of the **critical band**.

![image.png](8-Sound%20Perception/image.png)

<aside>
⚠️

**Critical band**

It’s a range of frequencies within which sounds interact closely in our perception. When two sounds fall within the same critical band, our brain struggles to distinguish them as separate sounds; instead, it “mixes” them into a single perceived sound. This phenomenon is known as **masking** and is fundamental in many audio compression algorithms: by representing multiple close frequencies as one, it reduces the amount of sound information to store without noticeably degrading sound quality.

</aside>

---

# 8.2 Anechoic Chamber

We said that we are not able to perceive all sounds in the same way, and for this reason, special rooms have been designed to study sound without any interference. The walls of these rooms are able to absorb all vibrations without reflecting any back → these are called **anechoic chambers**.

![image.png](8-Sound%20Perception/image%201.png)

## 8.2.1 Isophonic Curves

<aside>
💡

**Isophonic curves** were derived from experiments conducted in anechoic chambers, which eliminate sound reflections from walls. They represent the SPL values required at different frequencies for sounds to be perceived as equally loud.

</aside>

Imagine a sound source emitting sine waves at various frequencies but with a constant amplitude—for example, fixed at 80 dB SPL. Listeners perceive low-frequency sounds as much quieter, while higher frequencies appear louder, even though the actual sound pressure level remains the same. This occurs because the human ear’s sensitivity varies depending on frequency.

The unit **phon** is used to describe these loudness levels, linking sounds of different frequencies that are perceived as equally loud to a common reference. For instance, the 40-phon curve corresponds to sounds perceived as equally loud as a 40 dB SPL tone at 1 kHz.

The key insight is that at low frequencies, these curves rise steeply, meaning **significantly higher physical SPLs are needed for low-frequency sounds to be perceived as equally loud as mid-frequency sounds**.

In summary, sounds with different physical SPLs can be perceived as equally loud depending on their frequency.

![image.png](8-Sound%20Perception/image%202.png)

- The lowest isophonic curve of all is called the **audibility threshold** and indicates the smallest pressure change that the ear is able to detect at different frequencies:
    
    ![image.png](8-Sound%20Perception/image%203.png)
    
- **Pain threshold** (120 phons) the ear begins to perceive physical pain and for prolonged exposures non-reversible damage can be generated.

---

# 8.3 Masking

The phenomenon of **masking** occurs when two sounds fall within the same **critical band** and are played simultaneously. Because the frequencies are perceived at the same time and affect the same region of the inner ear, the louder sound tends to **mask** the quieter one. In practice, this **raises the audibility threshold** for the quieter sound, making it harder or even impossible to hear. This effect is known as **Simultaneous Masking:**

![image.png](8-Sound%20Perception/image%204.png)

## 8.3.2 Temporal Masking

There is also another form called **Temporal Masking**, which occurs when a loud sound temporarily makes us “deaf” to similar frequencies that appear shortly **after** it has been heard. This means that **our auditory perception is not instantaneous**, but has a temporal structure — we must consider what happens **before and after** a sound, not just at a single moment in time.

---