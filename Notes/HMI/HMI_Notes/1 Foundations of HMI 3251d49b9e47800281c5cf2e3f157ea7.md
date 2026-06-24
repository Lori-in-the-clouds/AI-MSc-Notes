# 1. Foundations of HMI

Pages: 2
Status: Done
Type: theory

# 1. Introduction: UI and HMI

<aside>
📌

**User Interface (UI)**

When developing an application, different layers exist. The surface layer, where the software interacts with the external world (the user), is the interface.

</aside>

<aside>
🔑

**UI vs HMI**

"User Interface" (UI) is the specific boundary between software and user. The term "Human-Machine Interface" (HMI) is often used to describe the same interaction, especially in broader or industrial contexts. Generative AI and Machine Learning are radically changing the nature of HMI. Before designing AI-driven interfaces, it is crucial to understand the standard rules and features of human-machine interaction.

</aside>

---

# 2. Ergonomics

**Ergonomics**, also known as **Human Factors**, comes from the Greek *ergon* (work) and *nomos* (laws). It studies how to design technologies, tools, and environments that are consistent with **human capabilities and limitations**.

Its main objective is to improve the **interaction between humans and technology**. Poorly designed systems or interfaces can lead to misunderstandings, inefficiency, and even accidents. Many events attributed to *human error* are actually caused by systems that do not adequately consider human physical or cognitive limits.

## 2.1 Types of Ergonomics

Ergonomics is typically divided into three areas:

- **Physical ergonomics** → focuses on posture, movement, and environmental conditions.
- **Cognitive ergonomics** →  studies how humans perceive and process information; it is central to **UI/HMI design**.
- **Process ergonomics** → concerns workflows and the organization of complex systems.

---

# 3. **Cognitive Processes in HMI Design**

Designing effective interfaces requires understanding **human cognitive processes** and their limitations. The most relevant processes for HMI design are:

- **perception**
- **attention**
- **memory**
- **thinking**
- **language**
- **emotions**

These processes determine how users interpret information and interact with technological systems.

---

# 4. Perception

**Perception** is the process through which humans interpret sensory information and build a mental representation of what they see. In interface design, unclear or ambiguous elements can lead to misinterpretation.

![Screenshot 2026-03-16 at 11.21.42.png](1%20Foundations%20of%20HMI/Screenshot_2026-03-16_at_11.21.42.png)

<aside>
📌

**Gestalt Theory**

Gestalt theory explains how people naturally **organize visual elements into meaningful patterns** when perceiving an interface. The main principles are:

- **Closure** → the mind fills gaps and perceives incomplete shapes as complete.
- **Proximity** → elements placed close to each other are perceived as a group.
- **Similarity** → elements with similar visual properties (color, shape, size) are grouped together.
- **Continuity** → the eye tends to follow smooth lines or curves, perceiving them as related.
- **Symmetry** → symmetrical elements are perceived as belonging together, creating order and reducing cognitive load.
- **Figure–Ground** → perception distinguishes an object (figure) from its background.
- **Common Fate** → elements moving in the same direction or at the same speed are perceived as belonging to the same group.

![image.png](1%20Foundations%20of%20HMI/image.png)

</aside>

<aside>
📈

**F-Pattern in Visual Scanning**

Eye-tracking studies show that users often scan digital interfaces following an **F-shaped pattern**. Attention is mainly concentrated on the **top area and the left side** of the screen. For this reason, important information should be placed in these areas to ensure rapid detection.

![image.png](1%20Foundations%20of%20HMI/c1e1f5ca-6c35-4cbd-a786-240d6a4f7830.png)

</aside>

## 4.1. Perception and Reasoning

Perception is not only responsible for recognizing objects but also plays a key role in **human reasoning and decision-making**. 

*Example:* a well-known example is a **space shuttle accident**, where poor data visualisation led engineers to misinterpret temperature data. Because the graph lacked a proper interpolation line between testing and flight conditions, the relationship between temperature and failure risk was not clearly visible, contributing to a catastrophic decision.

Effective **data visualisation** can significantly improve reasoning. According to **Dan Roam’s visual thinking framework**, different types of visual representations help answer different questions:

- **Profiles (Who / What)** → describe people or objects
- **Charts (How much)** → represent quantities
- **Maps (Where)** → show spatial information
- **Timelines (When)** → show temporal sequences
- **Flows (How)** → represent processes
- **Multi-variable plots (Why)** → explain relationships or causes

Combining these visual elements helps users understand **complex systems**, such as industrial dashboards or cybersecurity monitoring tools.

![image.png](1%20Foundations%20of%20HMI/image%201.png)

## 4.2. **Visual Recognition and Points of Interest (POI)**

In navigation interfaces, such as indoor maps in hospitals or airports, **Points of Interest (POIs)** must be easily identifiable. These elements are usually highlighted using two visual attributes: **shape and color.** Research in visual perception shows that **color differences are detected faster than shape differences**. For example, identifying a red square among blue squares is much faster than identifying a different shape among objects of the same color:

![image.png](1%20Foundations%20of%20HMI/image%202.png)

For this reason, when highlighting a specific element in a group, designers should **vary only one attribute at a time**, and often **color is the most effective attribute** for rapid recognition.

---

# 5. Attention

<aside>
🔑

**Attention** is the ability to focus on relevant information while ignoring distractions. It is a **limited cognitive resource**, so excessive demands can reduce performance and increase errors. 

</aside>

Two main mechanisms describe how attention operates during multitasking:

- **Task switching:** shifting attention between similar tasks (e.g., visual–visual), which is cognitively demanding.
- **Task sharing:** performing tasks that use different sensory channels (e.g., visual and auditory), which can be processed more efficiently.

Experiments like the **Invisible Gorilla test** show **selective attention**, demonstrating that people may miss visible events when focused on a specific task. For this reason, **HMI design should minimize distractions and prioritise essential information**.

---

# 6. Memory

<aside>
🔑

**Memory** allows humans to **encode, store, and retrieve information**, but it is limited and prone to errors.

</aside>

Three main types are involved:

- **Sensory memory:** briefly retains raw sensory input.
- **Working memory (short-term memory):** temporarily holds and processes information during tasks.
- **Long-term memory:** stores knowledge and experiences over time.

**N.B.** Since working memory is limited, interfaces should not rely on memorization but provide clear visual cues and guidance.

![image.png](1%20Foundations%20of%20HMI/image%203.png)

<aside>
📈

**Forgetting Curve (Ebbinghaus)**

Research by **Hermann Ebbinghaus** shows that humans forget information very quickly:

- About **45% is forgotten after 20 minutes**
- Less than **25% remains after one day**
- Retention stabilizes at a low level after several days

Interfaces used occasionally (for example car systems) should be **intuitive and easy to rediscover**, since users will forget many details between uses.

</aside>

---

# 7. Thinking

Human decision-making is strongly influenced by cognitive processes. According to **Daniel Kahneman**, thinking operates through two different systems:

1. **Fast Thinking** → is automatic, intuitive, and fast. It requires little effort and manages most everyday decisions (about 95%). For example, answering the question *“What is the capital of France?”* immediately activates System 1.
2. **Slow Thinking** → is slower, analytical, and requires conscious effort. It is used for complex reasoning, calculations, or unfamiliar situations. For example, solving a difficult mathematical problem requires System 2.

**N.B.** Because System 2 consumes more mental effort, humans tend to rely on **System 1 whenever possible**.

## 7.1. **Cognitive Biases and Anchoring**

Human reasoning is also influenced by **cognitive biases**, which are systematic errors in judgment caused by mental shortcuts (heuristics). One of the most important biases in interface design is the **Anchoring Bias**.

<aside>
📌

**Anchor Bias**

Anchoring occurs when people rely too heavily on the **first piece of information they receive**, which becomes a reference point for subsequent decisions. For example, if a system suggests a **default donation amount**, users tend to adjust their decision around that value, even if they initially intended to donate a different amount.

**N.B.** For interface designers, this means that **default values and recommended options strongly influence user behavior**. Designers must therefore choose these defaults carefully, since they can guide users toward specific decisions.

</aside>

---