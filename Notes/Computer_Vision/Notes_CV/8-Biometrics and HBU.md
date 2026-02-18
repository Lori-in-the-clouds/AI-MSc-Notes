# 8-Biometrics and HBU

Done?: Done
Select: theory

# 1. Introduction

<aside>
💡

**Biometrics**

It refers to the precise measurement of human identity based on unique human features. It allows identification and authentication of individuals.

---

**Human Behavior Understanding (HBU)**

HBU aims to recognize and understand human actions and behaviours from video data. It enables tasks such as emotion analysis, tracking, and re-identification.

</aside>

## 1.1. Constraints in Biometrics and HBU systems

Systems in this domain needs to be compliant with regulations such as the AI Act and often require additional properties:

- Explainability and interpretability
- Privacy Compliance

---

# 2. Sub-problems of HBU in video

- **Action Recognition** → the process of classifying a full video based on the action it contains.
    - Input: complete video
    - Output: single action label

![image.png](8-Biometrics%20and%20HBU/image.png)

- **Action Prediction** → the process of classifying an action that has not finished yet, using a partial observation of the video. There are different types:
    - Action Anticipation**:** the observed part does not include the action itself, therefore we can only use contextual information
    - Early Action Prediction: a portion of the action is already visible in the observation
    
    **Output:** predicted single class
    

![image.png](8-Biometrics%20and%20HBU/image%201.png)

- **Temporal Action Proposals** → the goal is to identify the start and end times of actions in untrimmed videos.
    
    Output: time intervals of actions (no classification)
    

![image.png](8-Biometrics%20and%20HBU/image%202.png)

- **Temporal Action Localization/Detection** → this combines temporal action proposals with action recognition.
    
    Output: time intervals of actions and classification
    

![image.png](8-Biometrics%20and%20HBU/image%203.png)

- **Spatio-Temporal Action Proposals** → similar to temporal action proposals but also includes spatial localization of actions. The output is often represented as a “tube” (3D bounding box).
    
    Output: time intervals + localization (no classification)
    

![image.png](8-Biometrics%20and%20HBU/image%204.png)

- **Spatio-Temporal Action Localization/Detection** → this combines spatio-temporal proposals with action recognition.
    
    Output: time intervals + spatial localization + classification
    

![image.png](8-Biometrics%20and%20HBU/image%205.png)

- **Action Description** → generates natural language descriptions of detected actions within a video.

![image.png](8-Biometrics%20and%20HBU/image%206.png)

## 2.1. Graph-Based Representations

In HBU, understanding the semantic relationships between humans (actors) and objects is crucial. These relationships can be modeled as a graph:

- **Nodes:** detected people and objects
- **Edges:** represent affordances

<aside>
🔑

**Affordance**

All the possible actions that an object can perform given human interaction. 

</aside>

**N.B.** Graph-based modeling can be added on top of any backbone architecture.

---

# 3. Anomaly Detection

<aside>
💡

**Anomaly Detection**

Detects rare or unusual events that deviate significantly from the majority of the data. Training data may contain both normal samples and outliers.

---

**Novelty Detection**

Detects new or unseen patterns not present during training. Deviations at test time are considered novel.

</aside>

## 3.1. Approaches for anomaly detection

- **Autoencoder** → learns to reconstruct normal behavior. High reconstruction error indicates anomalous behavior.
    
    Output: reconstruction error
    
- **Estimator** → learns the probability distribution of latent representations to evaluate the “surprisal” of an action.
    
    Output: surprisal score
    

**N.B.** Reconstruction error and surprisal score can be combined in a loss function.

---

# 4. Human Pose Estimation

## 4.1. Multiple-Person Pose Estimation Approaches

1. **Top-Down** → detect individuals first, then estimate poses within their bounding boxes.
2. **Bottom-Up** → ****detect keypoints first and then link them together to determine the human pose.

## 4.2. Some Pose Estimation Algorithm

- **CPM (Convolutional Pose Machines):** multi-stage CNN producing 2D belief maps for keypoint locations; intermediate supervision is applied at each stage.
- **OpenPose:** extension of CPM capable of multi-person pose estimation.

## 4.3. Datasets

Synthetic datasets are commonly used to increase labeled data availability. Notable example: **MotSynth**.

---

# 5. Biometrics

Biometrics studies human characteristics:

- **Behavioural:** movement patterns while speaking, walking, typing, or writing.
- **Physiological:** fingerprints, face, eyes, etc.

**Some models:** MTCNN (2016), YOLO5Face, FaceNet (2015), VGGFace, DeepFace

**Datasets:** CelebA (200K+ celebrity images), YouTube Faces (600K images)

---