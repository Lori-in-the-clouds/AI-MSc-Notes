# L5-Object Detection Architectures

Done?: Done
Select: lab

# 1. Computer Vision Tasks

- **Image Classification →** assign a single class label to an input image. The output is typically a fixed-size vector of class scores.
- **Classification + Localization** → predict the class of an object and also its precise location within the image, usually represented by a bounding box. This is typically for a single object in the image.
- **Object Detection** → the model must identify and locate multiple objects within an image. For each detected object, a class label and a bounding box are provided.
- **Semantic Segmentation** → this task involves classifying each pixel in an image into a predefined category, such as "*road*", "*sky*", or "*person*".
- **Instance Segmentation** → it combine elements of object detection and semantic segmentation. It requires the model to detect each individual object instance in an image, predict its class and bounding box, and also generate a precise pixel-level mask for each instance.

![image.png](L5-Object%20Detection%20Architectures/image.png)

---

# 2. Object Detection

The real challenge in object detection is how to create an architecture able to produce a **variable size output**. This is needed because the number of objects we have to detect in each image is variable.

## 2.1. Foundational Approaches to Object Detection

At the beginning, people tried to solve object detection by using methods made for image classification. Here are the main ideas:

- **Detection as a Regression Problem** → the first idea was to train a model to directly predict bounding box coordinates $(x, y, w, h)$ for a fixed number of objects in each class. This approach fails because the number of objects in an image can vary, but a fixed-output model cannot handle more or fewer objects than expected.
- **Detection as a Classification Problem (The Sliding Window)** → a fixed-size window is moved across the image at different scales and aspect ratios. At each position, a Convolutional Neural Network (CNN) classifies the content within the window as either an object of interest or background. The limitation of this approach is that it is computationally prohibitive.
    
    ![image.png](L5-Object%20Detection%20Architectures/41632691-4afd-44d0-a1fd-ffc3c8ed496c.png)
    
- **Region Proposals** → ****instead of checking every possible location, first generate *candidate regions* that are likely to contain objects, and then process these regions.
    
    ![image.png](L5-Object%20Detection%20Architectures/c103fe19-6f62-467b-bcdd-e8483bb9d8b2.png)
    
    <aside>
    💡
    
    **Selective Search Algorithm (a common region proposal method)**
    
    1. Split the image into many small regions
    2. Similar neighboring regions are merged based on things like color, texture, and shape
    3. Repeat merging in a hierarchical way to get regions at different scales
    4. Each region is then surrounded by a bounding box to suggest where an object might be
    
    ![image.png](L5-Object%20Detection%20Architectures/image%201.png)
    
    **N.B.** Despite its simplicity, this method can generate around 2000 proposals in a few seconds on a standard CPU.
    
    </aside>
    

---

# 3. The Three Main Families of Object Detection Architectures

Modern object detection architectures can be categorized into three families: 

- Region-Based Object Detection (Two stages)
- Single-Shoot Detection (one stage)
- Transformer-based Detection

## 3.1. Region-Based Object Detectors (Two-Stage)

Two-stage detectors work in two main steps: first, they propose candidate object regions, and then they classify and refine these proposals.

- **R-CNN (Region-based CNN)** → it was one of the first methods to successfully combine convolutional neural networks with region proposals for object detection. The pipeline works as follows:
    
    
    1. **Region Proposal** → about 2000 candidate object regions are generated using an external method, usually Selective Search.
    2. **Feature Extraction** → each region is resized (e.g., to $224×224$) and passed separately through a pre-trained CNN like AlexNet to get a feature vector.
    3. **Classification** → for each class, a dedicated linear Support Vector Machine (SVM) is trained to classify the feature vector extracted from each region.
    4. **Bounding Box Regression** → a linear regression model is trained to improve the accuracy of the bounding box coordinates.
    
    **N.B. Learnable parameters**: CNN fine-tuning, SVM classifier, Bounding box regressor
    
    ![image.png](L5-Object%20Detection%20Architectures/image%202.png)
    
    <aside>
    🚨
    
    **Limitations:**
    
    - **The training process is not unified**; it involves several stages, each with different loss functions and models, for example, fine-tuning the CNN with one loss, training the SVMs with another, and using a third for bounding box regression. This makes optimization difficult.
    - **Inference is very slow**, around 47 seconds per image, because the CNN processes each of the 2000 region proposals separately, resulting in a lot of repeated computation.
    </aside>
    
- **Fast R-CNN** → improves R-CNN by running the CNN only once on the whole image to produce a shared feature map. The pipeline works as follows:
    
    
    1. **Feature Extraction** → ****the whole image is passed once through a CNN to get a convolutional feature map.
    2. **Region Proposals** → generate ~2000 **candidate regions (ROIs)** on the original image and project them on the feature map.
    3. **RoI Pooling** →  is an operation that allows for the extraction of **fixed-size feature maps** from each ROI (=Region of Interest).
    4. Each pooled ROI is passed through **FC layer** for further processing.
    5. The output of the FC layer are passed to a **Softmax Classifie**r in order to predict the label, and to a BB-Reg in order to adjust the bounding box coordinates.
    
    ![image.png](L5-Object%20Detection%20Architectures/5729157f-c731-49c0-a7e6-5164e5d90483.png)
    
    **N.B. Learnable parameters**: final Classifier, BB-regressor, RoI-Head (The FC after RoI-Pooling), Finetuning of CNN
    
    <aside>
    📌
    
    **Focus on RoIPooling Layer**
    
    Region proposals are generated based on statistics from the **original image**, not the **intermediate feature map**. When we project these ROIs onto the feature map, coordinates often become **fractional** due to resolution downsampling. So for this reason, they cannot be used directly to extract precise features.
    
    The solution to this problem is ROI-Pooling:
    
    1. We snap the coordinates of the proposal to the nearest integers (=fractional coordinates are rounded):
        
        ![image.png](L5-Object%20Detection%20Architectures/image%203.png)
        
    2. Since we also want to bring the shape of the proposal to a standard shape we also divide it into a standard number of portions an perform maxpooling on each:
        
        ![image.png](L5-Object%20Detection%20Architectures/image%204.png)
        
    
    **Problem**: since we are snapping the coordinates to the nearest integer the actual pixels we are taking are slightly misaligned
    
    </aside>
    
    <aside>
    ✅
    
    **Solution: ROI-Align**
    
    RoIAlign was introduced to fix the alignment problems caused by RoIPooling. This is especially important for tasks that need high precision, like object segmentation.
    
    1. Divide the proposal in a standard number of regions (e.g. $7\times7$)
    2. In each region we take a standard number of equidistant points and we compute their values exploiting 2D interpolation: 
    $P(x,y)_{interpolated} = \sum_{i,j = 1}^2 f_{i,j} \cdot(1-|x-x_i|)\cdot(1-|y-y_i|)$
        
        **N.B.** The sum iterates over the four nearest pixels to $P(x,y)$
        
    3. Apply **max pooling** (or average pooling) over the interpolated points in each subregion for every channel to obtain a **fixed-size feature map**.
    
    ![image.png](L5-Object%20Detection%20Architectures/9b5a2174-eebf-408c-8830-00c27b1cdec5.png)
    
    </aside>
    
    <aside>
    🚨
    
    **Limitations:**
    
    - Still relatively slow due to separate region proposal computation. A better solution would be to share convolutional operations between proposal generation and detection.
    - The pipeline remains complex and not fully unified. End-to-end training is preferred to simplify the process.
    </aside>
    

- **Faster R-CNN (Real-Time Detection)** → improves over Fast R-CNN by **removing the external region proposal step** and replacing it with a **Region Proposal Network (RPN)**. This allows the network to generate region proposals **directly from the shared feature map** produced by the backbone CNN, enabling **end-to-end training** and **real-time detection**.
    
    <aside>
    📌
    
    **Focus on Region Proposal Network**
    
    The RPN slides over the feature map. At each location, it evaluates $K$ anchor boxes (with various scales and aspect ratios) per pixel. For each anchor, it predicts:
    
    - An objectness score (is it an object or not?)
    - 4 box refinements (with the use of **BB-Reg**): $Δx, Δy, Δw, Δh$ to refine the anchor box’s position and size to better fit a potential object
    
    Example:
    
    ![image.png](L5-Object%20Detection%20Architectures/image%205.png)
    
    </aside>
    
    **N.B.** Final Classifier (x2), BB-Regressor (x2), Region Proposal Network, Finetuning of CNN
    
    <aside>
    🚨
    
    **Problems:**
    
    Faster R-CNN still uses **three separate networks**: backbone, RPN, and RoI head. This adds complexity and slows inference. This problem is the exact motivation for single-stage detectors (YOLO).
    
    </aside>
    

![There are 4 losses. They flow in different ways according to what I need to update](L5-Object%20Detection%20Architectures/9449ecf6-7cb0-4119-a1f8-21e85af1af1b.png)

There are 4 losses. They flow in different ways according to what I need to update

- **Mask R-CNN** → ****extends Faster R-CNN to perform instance segmentation, a more challenging task that requires pixel-accurate object masks.
    
    
    **Architecture**:
    
    Mask R-CNN builds directly on Faster R-CNN with two main modifications:
    
    1. RoIAlign is used instead of RoIPooling to preserve spatial accuracy.
    2. A third branch is added to the network head, running in parallel with the classification and bounding box regression branches. This branch is a small Fully Convolutional Network (FCN) that predicts a binary mask (e.g., $28×28$ pixels) for each ROI, representing the object's precise shape.
    
    ![image.png](L5-Object%20Detection%20Architectures/3d22e5f8-50b3-4769-a59b-cdeb2c8c6783.png)
    

## 3.2. Single-Shot Detectors (One-Stage)

Single-shot detectors focus on speed by performing object detection in a single forward pass, eliminating the need for a separate region proposal stage.

- **YOLO (You Only Look Once) →** is a **single-stage object detector** that combines feature extraction and detection in one network.
    
    
    1. The **input image** is divided into an $S\times S$ grid of cells
    2. For each grid cell, the network predicts:
        - **$B$ bounding boxes**, each with:
            - Center coordinates $(x_c, y_c)$
            - Width and height $(w, h)$
            - **Confidence score for the class** → $P(class)\cdot P(object)$
        - **$C$ class probabilities**, assuming an object exists in the cell
    
    ![image.png](L5-Object%20Detection%20Architectures/e9dda210-0b8a-45fa-bb89-f54b4666b497.png)
    

**N.B.** The final output tensor for YOLO has the shape  $S × S × (B × 5 + C)$, where $C$ is the number of classes.

- **CenterNet** → takes a different approach: instead of predicting bounding boxes directly, it detects objects by identifying their center points:
    
    
    1. A keypoint estimation network is used to predict a heatmap. Peaks in this heatmap correspond to the center points of objects.
    2. For each detected center point, the network directly regresses other object properties, such as the object’s width ($w$) and height ($h$).
    
    This approach often reduces the need for Non-Maximum Suppression, simplifying the pipeline and avoiding some of the post-processing complexity of other detectors.
    
    ![image.png](L5-Object%20Detection%20Architectures/image%206.png)
    

## 3.3. Detection Transformers

Detection Transformers (DETR) aim to use **transformers for object detection**, replacing traditional components like anchor boxes and NMS. Transformers naturally produce a **sequence**, whereas object detection requires a **set of objects**. DETR resolves this mismatch using **object queries**.

![image.png](L5-Object%20Detection%20Architectures/image%207.png)

**How it works:**

1. The input image is processed by a CNN backbone to produce a feature map, which is then flattened into a feature vectors with positional encoding added to create the input embeddings.
2. The features are passed into a Transformer encoder.
3. A Transformer decoder then receives a fixed number of learned positional embeddings, called *object queries*, and iteratively refines them to predict the bounding box coordinates and class for each object.
4. Each output is then passed through a FFN prediction head to get as output either the class and the bbox or “no object”.

<aside>
📌

**Focus on Object Queries**

The problem with transformers is that you get an output of the same length of the input, so if my image is divided into 9 patches I will always get as output 9 bounding boxes. To avoid this problem we feed to the decoder a list of vectors called object queries, while the embeddings of the patches are fed via cross attention. In this way the number of patches and final outputs are completely independent.

</aside>

<aside>

**Pro/Contro**

- Simple design, fewer heuristic components, easier to extend to other tasks
- Needs more training time and large datasets to work well. Can be unstable during training
</aside>

---