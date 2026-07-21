# 11. Discrete Event Simulation

Num of pages: 1
Status: Done
Type: theory

# 1. Introduction

<aside>
💡

**Simulation**

It’s is the process of designing a model of a real system and conducting experiments for the purpose of:

- **understanding the behavior** of the *system*
- **evaluate different operational strategies** and identify the optimal one
</aside>

## 1.1. When to Simulate

Simulation is particularly useful for systems that are:

- **Non-existent (Future Systems)** → allows studying systems that do not yet exist.
- **Expensive or Dangerous** → enables testing without the cost or risk of experimenting on the real system.
- **Complex** → helps understand complex internal dynamics that are mathematically too difficult to solve analytically.

## 1.2. When not to Simulate

Simulation is a good tool to study complex systems, anyway it is not a good solution when:

- direct experiments are less expensive
- simulation model is not validated
- the level of details required to answer the questions is too complex

---

# 2. System Definitions

- **System** → group of entities that are joined together in some regular interaction or interdependence toward the accomplishment of some purpose. In particular:
    - **Entity:** an object of interest within the system.
    - **Attributes:** properties or characteristics of the entity.
    - **Activity:** is an action or task performed over a period of time.
    - **Event:** an instantaneous occurrence that may change the system state. It can be exogenous (external) or endogenous (internal) to the system.
- **State** → set of variables needed to describe the system at a particular moment.
- **Environment** → everything outside the system that can affect it.
- **Boundary** → the separation between the system and its environment.

---

# 3. Model Definition

A **model** is a representation of a system for the purpose of studying it. The level of detail should match the goal: useful models simplify reality to provide insight, while overly complex models risk being as hard to understand as the real system, defeating the purpose of simulation.

<aside>
🚨

**System vs Model**

The **system** is the actual, complex reality, instead the **model** is a **simplification** of that reality created to understand or test it.

</aside>

## 3.1. Models Classification

A model can be:

- **Physical or Mathematical**
- **Static or Dynamic**
    - *Static →* represents a system at a specific point in time, time is irrelevant.
    - *Dynamic →* represents a system evolving over time.
- **Deterministic or Stochastic**
    - *Deterministic →* no random variables. Identical inputs always produce the exact same output.
    - *Stochastic →* includes random variables. Most real-world systems are stochastic.
- **Discrete or Continuous**
    - *Discrete →* state variables change at discrete points in time.
    - *Continuous* → state variables change continuously over time.

![image.png](11%20Discrete%20Event%20Simulation/dfa6a0b6-097c-4174-a20d-41c625d2a185.png)

**N.B.** This course focuses on **Discrete Event Simulation (DES)**, which is dynamic, stochastic, and discrete.

---

# 4. Discrete Event Simulation (DES)

**Discrete Event Simulation (DES)** models a system where state variables change instantaneously **at discrete points in time**, driven by specific **events**. A DES study consists of the following steps:

1. **Discovery and Orientation:** define the problem, set objectives, and determine available resources.
2. **Model Building:** this phase focuses on defining and implementing the simulation model:
    1. Conceptualisation ****→ start with a simple model and gradually enrich it until it provides a useful approximation of the real system.
    2. Data Collection → gather all necessary data to parameterize the model. It may require a lot of time and can be expensive.
    3. Translation and Verification → implement the model (e.g., in Python/SimPy) and ensure it correctly reflects the conceptual model without errors.
    4. Validation → ****critically check that the model accurately represents the system’s behavior. An invalid model leads to unreliable decisions.
3. **Run Phase:** simulate different designs or policies. Each simulation is characterised by:
    - warm-up period
    - a defined run length
    - multiple independent replications to obtain unbiased, low-variance estimates
4. **Implementation Phase:** evaluate and compare results, generate reports, and select the best design for deployment.

![image.png](11%20Discrete%20Event%20Simulation/image.png)

---