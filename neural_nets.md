# Personalized Framework for Neural Networks

## Helpful Terms To Keep Top of Mind
- **Forward Pass**: The process of passing input through the network to generate an output.
- **Backpropagation**: A method used for training the network by adjusting weights based on errors.
- **Hyperparameters**: Configurations that define the model's structure and training process.
- **Optimization Functions**: Algorithms that minimize loss functions during training.
- **Loss Functions**: A function that measures how well the model's predictions match the expected result.
- **Gradients**: Measures of how much a change in input affects the output.

---

## Neural Net Architecture

The neural network consists of layers, weights, and connections that allow information to flow and be processed.

### Forward Pass
In this step, the input is passed through the network, and each layer transforms the data before producing the output.

#### Inference Mode
In inference mode, the model processes data to make predictions without adjusting weights.

#### Training Mode
During training mode, weights are updated using backpropagation to reduce the loss.

---

## **General Forward Pass Flow for Different Task Types in Neural Networks**

Below is a step-by-step breakdown of the forward pass for different task types, including how the input and output vectors are represented and how they translate into meaningful predictions.

---

### **1. Binary Classification**
**Example Task:** Determining if an email is spam (1) or not spam (0).  

**Input Representation:** A feature vector representing email characteristics, such as word frequencies, sender reputation, and presence of specific phrases. 

![Vector X](https://latex.codecogs.com/png.latex?X%20%3D%20%5B0.2%2C%200.5%2C%200.8%2C%200.1%2C%200.6%5D)

**Output Representation:** A single probability value between 0 and 1, indicating how likely the email is spam.  

![Equation](https://latex.codecogs.com/png.latex?y%20%3D%20%5Csigma(WX%20%2B%20b)%2C%20%5Cquad%20%5Csigma(x)%20%3D%20%5Cfrac%7B1%7D%7B1%20%2B%20e%5E%7B-x%7D%7D)
- **WX + b** is the raw, unbounded score (logit).
- **σ(x)** transforms the logit into a probability.
- **y** is the final probability output of the neural network, which determines classification (e.g., spam vs. not spam).

💡 **Personal Helper Note: Why use the Sigmoid Function (σ(x))?**
    - **Monotonic and Asymptotic Behavior.** 
        - Sigmoid is **monotonic,** meaning as input ![x](https://latex.codecogs.com/png.latex?x) increases, the output σ(x) never decreases. 
        - It is **asymptotic** as it approaches **1** as ![Sigmoid Behavior](https://latex.codecogs.com/png.latex?x%5Cto%5Cinfty) and **0** as ![x → −∞](https://latex.codecogs.com/png.latex?x%20%5Cto%20-%5Cinfty).
    - **Smooth and differentiable**
        - Sigmoid is a smooth and continuous function, meaning it allows gradient-based optimization (backpropagation) to work effectively.
        - Since it has a well-defined derivative: ![Sigmoid Derivative](https://latex.codecogs.com/png.latex?%5Csigma%27%28x%29%20%3D%20%5Csigma%28x%29%281%20-%20%5Csigma%28x%29%29) it provides a convenient way to compute gradients, helping neural networks learn efficiently.

💡 **Personal Helper Note: Why is ![e](https://latex.codecogs.com/png.latex?e) (Euler's Number) used in the Sigmoid Function?** Euler's number ![e ≈ 2.718](https://latex.codecogs.com/png.latex?e%20%5Capprox%202.718) plays a key role in the sigmoid function due to its natural mathematical properties:
1. **Exponential Growth and Decay**
    - The ![e^-x](https://latex.codecogs.com/png.latex?e%5E%7B-x%7D) term ensures a **smooth, and gradual transition** between 0 and 1.
    - As ![x](https://latex.codecogs.com/png.latex?x) increases, ![e^-x](https://latex.codecogs.com/png.latex?e%5E%7B-x%7D) shrinks exponentially, making σ(x) approach 1.
    - As ![x](https://latex.codecogs.com/png.latex?x) decreases, ![e^-x](https://latex.codecogs.com/png.latex?e%5E%7B-x%7D) grows exponentially, making σ(x) approach 0.
2. **Log-Odds Interpretation (Logistic Function)**
    - The sigmoid function is derived from **logistic regression**, where it models probabilities using **log-odds**.
    - The transformation: ![sigmoid](https://latex.codecogs.com/png.latex?%5Csigma%28x%29%20%3D%20%5Cfrac%7B1%7D%7B1%20%2B%20e%5E%7B-x%7D%7D) comes from the logistic equation used in statistics: ![log-odds](https://latex.codecogs.com/png.latex?%5Clog%5Cleft%28%5Cfrac%7Bp%7D%7B1-p%7D%5Cright%29%20%3D%20x) where ![p](https://latex.codecogs.com/png.latex?p)
 is the probability of an event occurring.

**Output Interpretation:** If \( y > 0.5 \), classify as **spam (1)**; otherwise, classify as **not spam (0)**.

![classification rule](https://latex.codecogs.com/png.latex?y%20%3E%200.5%20%5CRightarrow%20%5Cmathbf%7BSpam%20%281%29%7D%2C%20%5Ctext%7Belse%7D%20%5CRightarrow%20%5Cmathbf%7BNot%20Spam%20%280%29%7D)

---

### **2. Multi-Class Classification**
**Example Task:** Recognizing handwritten digits (0-9).  
**Input Representation:** Flattened grayscale image pixels, where each value represents the intensity of a pixel.  
\[
X = [0, 0, 0.6, 0.8, 1, 0.5, 0, 0, 0]
\]
**Output Representation:** A probability distribution over all classes (0-9), where the sum of probabilities equals 1.  
\[
y = \text{softmax}(WX + b)
\]
\[
y = [0.01, 0.02, 0.05, 0.70, 0.10, 0.02, 0.03, 0.03, 0.02, 0.02]
\]
**Interpretation:** The model predicts **the digit with the highest probability** (in this case, **3** with 70% confidence).

---

### **3. Multi-Label Classification**
**Example Task:** Identifying multiple objects in an image (e.g., "dog" and "car" in the same image).  
**Input Representation:** Feature vector representing image characteristics (extracted via convolutional layers).  
\[
X = [0.9, 0.3, 0.6, 0.1, 0.5, 0.2]
\]
**Output Representation:** Independent probabilities for each class, where multiple labels can be assigned.  
\[
y = \sigma(WX + b)
\]
\[
y = [0.85, 0.02, 0.90, 0.10, 0.05]
\]
(If threshold \( > 0.5 \), label is predicted as present.)  
**Interpretation:** The model detects **class 1 (dog) and class 3 (car)** since they exceed the threshold.

---

### **4. Regression Task with Unbounded Output**
**Example Task:** Predicting house prices based on features like square footage, number of bedrooms, and location.  
**Input Representation:**  
\[
X = [1500, 3, 2, 1, 5] \quad \text{(square feet, bedrooms, bathrooms, garage, neighborhood rating)}
\]
**Output Representation:** A continuous value with no constraints.  
\[
y = WX + b
\]
\[
y = 420,000
\]
**Interpretation:** The model predicts a house price of **$420,000**.

---

### **5. Regression Task with Bounded Output**
**Example Task:** Predicting customer satisfaction score (0 to 1).  
**Input Representation:**  
\[
X = [0.6, 0.3, 0.9, 0.4] \quad \text{(customer review sentiment scores, response time, etc.)}
\]
**Output Representation:** A single continuous value constrained to a range using a **sigmoid activation function**.  
\[
y = \sigma(WX + b)
\]
\[
y = 0.78
\]
**Interpretation:** The predicted satisfaction score is **78%**.

---

### **Summary Table**
| Task Type | Input Vector Example | Output Activation | Output Vector Example | Interpretation |
|-----------|----------------------|-------------------|----------------------|---------------|
| **Binary Classification** | `[0.2, 0.5, 0.8, 0.1, 0.6]` | Sigmoid | `[0.87]` | Classifies as spam (1) if \( y > 0.5 \) |
| **Multi-Class Classification** | `[0, 0, 0.6, 0.8, 1, 0.5, 0, 0, 0]` | Softmax | `[0.01, 0.02, 0.05, 0.70, 0.10, 0.02, 0.03, 0.03, 0.02, 0.02]` | Predicts the digit 3 |
| **Multi-Label Classification** | `[0.9, 0.3, 0.6, 0.1, 0.5, 0.2]` | Sigmoid | `[0.85, 0.02, 0.90, 0.10, 0.05]` | Detects multiple objects: Dog (1) and Car (3) |
| **Regression (Unbounded)** | `[1500, 3, 2, 1, 5]` | None (Linear) | `[420000]` | Predicts house price: **$420K** |
| **Regression (Bounded)** | `[0.6, 0.3, 0.9, 0.4]` | Sigmoid | `[0.78]` | Predicts customer satisfaction: **78%** |

---

## Functions

### Activation Functions
Activation functions introduce non-linearities into the model, enabling it to learn complex patterns. Different activation functions serve different purposes and are used in specific layers of a neural network:

#### **Types of Activation Functions and Their Uses**
1. **ReLU (Rectified Linear Unit)**
   - **Used in:** Most hidden layers of deep neural networks.
   - **Why?** Efficient, prevents vanishing gradients, and allows deep networks to train faster.
   - **Downside:** Can suffer from the "dying ReLU" problem (neurons stuck at zero output).

2. **Leaky ReLU / Parametric ReLU**
   - **Used in:** Hidden layers, especially in GANs and deep networks.
   - **Why?** Addresses dying ReLU issue by allowing a small slope for negative inputs.

3. **Sigmoid**
   - **Used in:** Output layer for binary classification.
   - **Why?** Outputs values between 0 and 1, making it useful for probability-based predictions.
   - **Downside:** Can cause vanishing gradients in deep networks.

4. **Tanh (Hyperbolic Tangent)**
   - **Used in:** Some hidden layers (historically in RNNs).
   - **Why?** Outputs between -1 and 1, making it useful for zero-centered data.
   - **Downside:** Still suffers from vanishing gradients but is better than sigmoid.

5. **Softmax**
   - **Used in:** Output layer for multi-class classification.
   - **Why?** Converts logits into probabilities that sum to 1.
   - **Downside:** Can be sensitive to large input values, leading to numerical instability.

6. **Linear (No Activation)**
   - **Used in:** Output layer for regression tasks.
   - **Why?** Allows the model to predict continuous values without constraints.

#### **When Some Layers Do Not Use Activation Functions**
- The **output layer of regression models** does not use an activation function to allow unrestricted predictions.
- Sometimes, a **linear transformation layer** (such as the final layer before softmax) does not use activation functions to simplify computations.
- Some architectures, like **residual networks (ResNets)**, may pass information directly through certain layers without activation to preserve gradients.

### Loss Functions
Loss functions measure the discrepancy between predicted and actual outcomes, guiding optimization.

### Optimization Functions
Optimization functions help to reduce the error (loss) by adjusting the model's parameters.

---

## Use Cases
- Applying neural networks in image classification
- Using neural networks for natural language processing

---

## History
The concept of neural networks dates back to the 1940s, with key advancements such as the backpropagation algorithm in the 1980s.


