# AI and Interface Design

Pages: 1
Status: Done
Type: theory

# 1. Technologies vs. Smart Technologies

According to Donald Norman in *Things That Make Us Smart*, technologies act as cognitive and physical amplifiers and have evolved to adapt to human behavior since the rise of pervasive computing in the 1990s. From an interaction design perspective, there is a fundamental difference between:

- **Traditional Technologies** → provide a constant and stable interaction behavior. A specific action (e.g., pressing a button) always yields the same result, allowing the user to build a stable and reliable mental model.
- **Smart Technologies (AI)** → introduce **interactional variability**, systems such as autonomous vehicles or Large Language Models adapt to context, users, and inputs, and may not respond the same way every time. This variability makes it harder for users to build stable mental models, so interfaces must clearly communicate system behavior to avoid confusion and anxiety.

---

# 2. Smart Technologies Categories

Intelligent systems can be classified into four main categories based on how they interact with the user:

1. **Automation** (”*instead of the users*”) → performs tasks previously performed by user (e.g., autonomous driving). Automation is never a simple on/off switch; it exists on a spectrum from manual control to full autonomy.
    
    <aside>
    📌
    
    **Design Implication:** 
    
    The interface must manage the "handover" (the transition of control between machine and human) and provide continuous feedback to maintain situational awareness.
    
    </aside>
    
2. **Interpretation** (”*I know who you are*”)→ focuses on understanding the user’s state, context, or behavior (e.g., health monitoring or personalized content).
    
    <aside>
    📌
    
    **Design Implication:** 
    
    Interfaces should present this information in a useful and non-intrusive way, avoiding feelings of surveillance and guiding users appropriately without causing unnecessary alarm. 
    
    For example, if a car detects a fault, it should not immediately trigger a panic-inducing alarm at 130 km/h; the warning must be designed to guide the user safely.
    
    </aside>
    
3. **Prediction** (”*Shouldn't we talk about the weather*”)**→** estimates future events based on data. Since predictions are probabilistic, their value lies in how they influence user behavior.
    
    <aside>
    📌
    
    **Design Implication:** 
    
    Interfaces must clearly communicate uncertainty and present results in a way that encourages informed and constructive actions without causing unnecessary alarm.
    
    For example, the weather forecast: meteorologists often apply a **"wet bias"** (predicting rain when certainty is around 50%) to encourage cautious behavior, like bringing an umbrella.
    
    </aside>
    
4. **Awareness** (”*Extending beyond experience*”) **→ extends human perception beyond natural limits. Through technologies such as AR, VR, and mixed reality, systems can reveal otherwise invisible information, helping users better understand environments, data, or structures.

---

# 3. The UI in the AI

Current generative AI systems, such as Large Language Models, mainly use a **chat-based interface**.

## 3.1. Ergonomic Limitations Chat-based Interface

From a ergonomic perspective, the **chat-based interface** presents significant limitations. A chat stream is not well suited for structuring or sharing complex reasoning, since information is linear and hard to organize. It also creates a workspace imbalance, where the interface gives much more space to the AI’s output than to the user’s input, making it harder to write detailed prompts or develop complex ideas.

![image.png](AI%20and%20Interface%20Design/image.png)

## 3.2. Cognitive Ergonomics: Kahneman's Systems

To understand how users interact with AI, we must look at human cognition. Psychologist Daniel Kahneman divides human thinking into two systems:

- **System 1 (Intuition & Instinct)** → ****is fast, automatic, and intuitive, requiring little effort and driving most everyday decisions.
- **System 2 (Rational Thinking)** → is slower and analytical. It requires high cognitive effort, for this reason, humans tend to avoid it to save cognitive energy.

![image.png](AI%20and%20Interface%20Design/image%201.png)

## 3.3. The AI Design Problem and Cognitive Biases

Because AI interfaces are fast and conversational, users almost exclusively use System 1 to interact with them. Relying on System 1 exposes users to **cognitive biases** (=mental shortcuts that lead to irrational judgments).

<aside>
📌

**Key Cognitive Biases in AI Interaction:**

- **Confirmation Bias:** the tendency to favor information that confirms existing beliefs. Because LLMs are generally designed to please the user, they often agree with the prompt. This creates an echo chamber that exacerbates the user's flawed reasoning instead of challenging it.
- **Availability Heuristic:** overestimating the likelihood of events that are easier to recall, such as rare but vivid scenarios, rather than more common ones (e.g., fearing plane crashes more than common diseases because crashes are more salient in memory).
- **Anchoring Bias:** occurs when decisions are too heavily influenced by the first piece of information received, which sets a reference point for subsequent judgments.
</aside>

<aside>
✅

**Design Solutions**

To prevent users from falling into these cognitive traps, AI interfaces must be redesigned to intentionally trigger **System 2**. This can be done by introducing features like an "Alternative Views Explorer" or an "Automated Devil’s Advocate"—tools that contradict the user, present counter-arguments, and force critical thinking and rational analysis.

![image.png](AI%20and%20Interface%20Design/30c0ce47-6557-4cd9-aca6-09e134ffbc04.png)

</aside>

---