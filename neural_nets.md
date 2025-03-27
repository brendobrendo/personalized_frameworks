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

## 🤔 Which Task Type Should I Choose For My Problem?
#### **General Forward Pass Flow for Different Task Types in Neural Networks**

Below is a step-by-step breakdown of the forward pass for different task types, including how the input and output vectors are represented and how they translate into meaningful predictions.

---

### **1. Binary Classification**
**Example Task:** Determining if an email is **spam (1)** or not **spam (0)**.  

**Input Representation:** A feature vector representing email characteristics, such as word frequencies, sender reputation, and presence of specific phrases. 

![Vector X](https://latex.codecogs.com/png.latex?X%20%3D%20%5B0.2%2C%200.5%2C%200.8%2C%200.1%2C%200.6%5D)

**Output Representation:** A single probability value between 0 and 1, [y], indicating how likely the email is spam.  

![Equation](https://latex.codecogs.com/png.latex?y%20%3D%20%5Csigma(WX%20%2B%20b)%2C%20%5Cquad%20%5Csigma(x)%20%3D%20%5Cfrac%7B1%7D%7B1%20%2B%20e%5E%7B-x%7D%7D)
- **WX + b** is the raw, unbounded score (logit).
- **σ(x)** transforms the logit into a probability.
- **y** is the final probability output of the neural network, which determines classification (e.g., spam vs. not spam).

> 💭 **_Why use the Sigmoid Function (σ(x))?_** 
>**Monotonic and Asymptotic Behavior.**: Sigmoid is **monotonic,** meaning as input ![x](https://latex.codecogs.com/png.latex?x) increases, the output σ(x) never decreases. It is **asymptotic** as it approaches **1** as ![Sigmoid Behavior](https://latex.codecogs.com/png.latex?x%5Cto%5Cinfty) and **0** as ![x → −∞](https://latex.codecogs.com/png.latex?x%20%5Cto%20-%5Cinfty). **Smooth and differentiable** Sigmoid is a smooth and continuous function, meaning it allows gradient-based optimization (backpropagation) to work effectively. Since it has a well-defined derivative: ![Sigmoid Derivative](https://latex.codecogs.com/png.latex?%5Csigma%27%28x%29%20%3D%20%5Csigma%28x%29%281%20-%20%5Csigma%28x%29%29) it provides a convenient way to compute gradients, helping neural networks learn efficiently.

> 💭 **_Why is ![e](https://latex.codecogs.com/png.latex?e) (Euler's Number) used in the Sigmoid Function?_** 
>Euler's number ![e ≈ 2.718](https://latex.codecogs.com/png.latex?e%20%5Capprox%202.718) plays a key role in the sigmoid function due to its natural mathematical properties. **Exponential Growth and Decay** The ![e^-x](https://latex.codecogs.com/png.latex?e%5E%7B-x%7D) term ensures a **smooth, and gradual transition** between 0 and 1. As ![x](https://latex.codecogs.com/png.latex?x) increases, ![e^-x](https://latex.codecogs.com/png.latex?e%5E%7B-x%7D) shrinks exponentially, making σ(x) approach 1. As ![x](https://latex.codecogs.com/png.latex?x) decreases, ![e^-x](https://latex.codecogs.com/png.latex?e%5E%7B-x%7D) grows exponentially, making σ(x) approach 0. **Log-Odds Interpretation (Logistic Function)** The sigmoid function is derived from **logistic regression**, where it models probabilities using **log-odds**. 
>
>The transformation: 
>
>![sigmoid](https://latex.codecogs.com/png.latex?%5Csigma%28x%29%20%3D%20%5Cfrac%7B1%7D%7B1%20%2B%20e%5E%7B-x%7D%7D) 
>
>comes from the logistic equation used in statistics: 
>
>![log-odds](https://latex.codecogs.com/png.latex?%5Clog%5Cleft%28%5Cfrac%7Bp%7D%7B1-p%7D%5Cright%29%20%3D%20x) 
>
>where ![p](https://latex.codecogs.com/png.latex?p) is the probability of an event occurring.

**Output Interpretation:** If \( y > 0.5 \), classify as **spam (1)**; otherwise, classify as **not spam (0)**.

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

### Optimization Functions
Optimization functions help to reduce the error (loss) by adjusting the model's parameters.
#### **Types of Optimization Functions and Their Uses**

1. **Gradient Descent (GD)**
    - **Description:**  A first-order optimization algorithm used to minimize a function by iteratively moving in the direction of steepest descent as defined by the negative of the gradient.
    - **Mathematical Expression:**  
    ![GD](https://latex.codecogs.com/png.latex?\theta%20=%20\theta%20-%20\eta%20\nabla_\theta%20J(\theta))
    
    - Where:  
        - ![theta](https://latex.codecogs.com/png.latex?\theta) are the model parameters  
        - ![eta](https://latex.codecogs.com/png.latex?\eta) is the learning rate  
        - ![J(theta)](https://latex.codecogs.com/png.latex?J(\theta)) is the loss function

    - **First Used / Introduced By:** Cauchy, 1847; further formalized by Robbins & Monro, 1951 (stochastic version).

2. **Stochastic Gradient Descent (SGD)**
    - **Description:**  An extension of gradient descent where the parameters are updated for each training example, making it faster and more suitable for large datasets.
    - **Mathematical Expression:**  
    ![SGD](https://latex.codecogs.com/png.latex?\theta%20=%20\theta%20-%20\eta%20\nabla_\theta%20J(\theta;%20x^{(i)},%20y^{(i)}))
    - **First Used / Introduced By:** Robbins & Monro, 1951.

3. **Batch Gradient Descent**
    - **Description:**  A variation of gradient descent where updates are made after calculating the gradient on the entire training dataset
    - **Mathematical Expression:**  
    ![BatchGD](https://latex.codecogs.com/png.latex?\theta%20=%20\theta%20-%20\eta%20\nabla_\theta%20J(\theta;%20X,%20Y))
    - **First Used / Introduced By:** Standard approach attributed to early numerical optimization practices (pre-1980s).

4. **Adam (Adaptive Moment Estimation)**
    - **Description:** Combines the benefits of AdaGrad and RMSProp. It computes adaptive learning rates for each parameter using estimates of first and second moments of the gradients.
    - **Mathematical Expression:**
    ![Adam1](https://latex.codecogs.com/png.latex?m_t%20=%20\beta_1%20m_{t-1}%20+%20(1%20-%20\beta_1)%20\nabla_\theta%20J(\theta))  
    ![Adam2](https://latex.codecogs.com/png.latex?v_t%20=%20\beta_2%20v_{t-1}%20+%20(1%20-%20\beta_2)%20(\nabla_\theta%20J(\theta))^2)  
    ![Adam3](https://latex.codecogs.com/png.latex?\hat{m}_t%20=%20\frac{m_t}{1%20-%20\beta_1^t})  
    ![Adam4](https://latex.codecogs.com/png.latex?\hat{v}_t%20=%20\frac{v_t}{1%20-%20\beta_2^t})  
    ![Adam5](https://latex.codecogs.com/png.latex?\theta%20=%20\theta%20-%20\eta%20\frac{\hat{m}_t}{\sqrt{\hat{v}_t}%20+%20\epsilon})
    - **First Used / Introduced By:** Diederik P. Kingma & Jimmy Ba, 2014.

5. **AdaGrad (Adaptive Gradient Algorithm)**
    - **Description:** Adjusts the learning rate for each parameter based on past gradients, performing smaller updates for frequent parameters and larger updates for infrequent ones.
    - **Mathematical Expression:**  
    ![AdaGrad](https://latex.codecogs.com/png.latex?\theta%20=%20\theta%20-%20\frac{\eta}{\sqrt{G_t%20+%20\epsilon}}%20\nabla_\theta%20J(\theta))
    - **First Used / Introduced By:** John Duchi, Elad Hazan, Yoram Singer, 2011.

6. **RMSProp (Root Mean Square Propagation)**
    - **Description:** Modifies AdaGrad to resolve its radically diminishing learning rates by using a moving average of squared gradients.
    - **Mathematical Expression:**  
    ![RMSProp1](https://latex.codecogs.com/png.latex?E[g^2]_t%20=%20\gamma%20E[g^2]_{t-1}%20+%20(1%20-%20\gamma)%20g_t^2)  
    ![RMSProp2](https://latex.codecogs.com/png.latex?\theta%20=%20\theta%20-%20\frac{\eta}{\sqrt{E[g^2]_t%20+%20\epsilon}}%20g_t)
    - **First Used / Introduced By:** Tieleman & Hinton, 2012 (in a Coursera lecture by Geoffrey Hinton).

7. **Adadelta**
    - **Description:** An extension of AdaGrad that seeks to reduce its aggressive, monotonically decreasing learning rate by limiting the window of accumulated past gradients.
    - **Mathematical Expression:** 
    ![Adadelta](https://latex.codecogs.com/png.latex?\Delta%20\theta_t%20=%20-%20\frac{\sqrt{\Delta%20\theta_{t-1}%20+%20\epsilon}}{\sqrt{E[g^2]_t%20+%20\epsilon}}%20g_t)
    - **First Used / Introduced By:**  Matthew D. Zeiler, 2012.

8. **Nesterov Momentum (NAG)**
    - **Description:** An improved version of momentum-based SGD. It anticipates the future position of the parameters by applying the momentum step before computing the gradient.
    - **Mathematical Expression:**  
    ![NAG1](https://latex.codecogs.com/png.latex?v_{t+1}%20=%20\gamma%20v_t%20+%20\eta%20\nabla_\theta%20J(\theta%20-%20\gamma%20v_t))  
    ![NAG2](https://latex.codecogs.com/png.latex?\theta%20=%20\theta%20-%20v_{t+1})
    - **First Used / Introduced By:** Yurii Nesterov, 1983.

9. **Conjugate Gradient**
    - **Description:** An optimization method for large-scale problems. It solves systems of linear equations iteratively without storing large matrices.
    - **Mathematical Expression:** (Iterative updates based on conjugate directions instead of simple gradients; too lengthy to reduce to a single formula.)
    - **First Used / Introduced By:** Magnus Hestenes and Eduard Stiefel, 1952.

10. **Coordinate Descent**
    - **Description:** An optimization algorithm that minimizes along one coordinate direction at a time (cycling or choosing randomly), making it simpler and efficient for some problems.
    - **Mathematical Expression:** ![CoordDesc](https://latex.codecogs.com/png.latex?\theta_j%20=%20\theta_j%20-%20\eta%20\frac{\partial%20J}{\partial%20\theta_j})
    - **First Used / Introduced By:** Arthur F. Veinott, 1960.

11. **Bayesian Optimization**
    - **Description:** A probabilistic model-based optimization technique that builds a surrogate model to approximate the objective function and chooses samples using an acquisition function.
    - **Mathematical Expression:**  
        - Utilizes Gaussian Processes (or other probabilistic models)  
        - Optimizes acquisition functions like Expected Improvement (EI) or Upper Confidence Bound (UCB)
    - **First Used / Introduced By:** Jonas Mockus et al., 1978.

### Loss Functions
Loss functions measure the discrepancy between predicted and actual outcomes, guiding optimization.

---

## Ways to Modify the Learning Process of a Neural Network
When training neural networks, you can significantly impact learning speed, stability, and final performance by adjusting several components of the learning process. Here are the main levers you can pull:
### 1. Adjusting the Learning Rate

**Description:**  
The learning rate controls how large of a step the optimizer takes in the direction of the negative gradient. It is one of the most sensitive hyperparameters.

#### Strategies:
- **Static Learning Rate:** Set a constant learning rate throughout training.
- **Learning Rate Schedules:** Decrease the learning rate according to a pre-defined schedule (e.g., exponential decay, step decay).
- **Adaptive Learning Rates:** Use optimizers like Adam, RMSProp, or AdaGrad that adjust the learning rate dynamically per parameter.
- **Cyclical Learning Rates (CLR):** Oscillate between a lower and upper bound for the learning rate during training.
- **Warm-up:** Start with a lower learning rate and gradually increase it in the early epochs.

#### Impact:
- A **too high** learning rate can cause divergence or oscillations.
- A **too low** learning rate can slow convergence and get stuck in suboptimal minima.
### 2. Choice of Optimization Algorithm

**Description:**  
Different optimizers use different strategies to update the network's weights and can dramatically affect the training dynamics and convergence.

#### Impact:
- Some optimizers converge faster but generalize worse (e.g., Adam vs SGD in some vision tasks).
- Optimizers with adaptive learning rates can handle sparse or noisy data better.
### 3. Batching Strategy (Mini-batch Size)

**Description:**  
Defines how much data you use to estimate the gradient at each update step.

#### Types:
- **Batch Gradient Descent (Full-batch):** Uses the entire training dataset for each update.  
  - **Pros:** Stable updates.  
  - **Cons:** Can be very slow for large datasets.
  
- **Stochastic Gradient Descent (SGD):** Updates based on a single sample at a time.  
  - **Pros:** Fast updates, more frequent weight adjustments.  
  - **Cons:** High variance; updates may be noisy.

- **Mini-Batch Gradient Descent:** Updates based on a small subset of the dataset (e.g., 32, 64, 128 samples).  
  - **Pros:** A balance between stability and speed. Most commonly used in deep learning.

#### Impact:
- **Small batches** introduce noise, which can help escape local minima but may hurt convergence stability.
- **Large batches** result in smoother, more stable updates but require more memory and may converge to sharp minima.

### 4. Gradient Clipping

**Description:**  
A technique to clip or limit the gradients during backpropagation to avoid exploding gradients, especially in recurrent neural networks (RNNs).

#### Strategies:
- **Norm Clipping:** Clip gradients if their L2 norm exceeds a threshold.
- **Value Clipping:** Clip individual gradient values element-wise.

#### Impact:
- Prevents numerical instability.
- Helps maintain smoother training dynamics.
### 5. Early Stopping

**Description:**  
Stop training when the validation loss stops improving for a specified number of epochs (patience).

#### Impact:
- Prevents overfitting.
- Saves compute time by halting unnecessary training.
### 6. Weight Initialization

**Description:**  
Initializing weights appropriately to avoid vanishing or exploding gradients.

#### Common Schemes:
- **Xavier/Glorot Initialization:** Recommended for tanh and sigmoid activations.
- **He Initialization:** Recommended for ReLU and its variants.
- **Orthogonal Initialization:** Sometimes used for RNNs.

#### Impact:
- Poor initialization can slow down or even prevent convergence.
### 7. Normalization Techniques

**Description:**  
Techniques like **Batch Normalization**, **Layer Normalization**, or **Group Normalization** adjust and scale the activations to stabilize and speed up training.

#### Impact:
- Allows for higher learning rates.
- Reduces internal covariate shift.
- Can improve generalization.
### 8. Regularization Methods

**Description:**  
Used to penalize model complexity to reduce overfitting.

#### Examples:
- **L2 Regularization (Weight Decay):** Adds a penalty proportional to the square of the weights.
- **L1 Regularization:** Promotes sparsity by penalizing absolute values of weights.
- **Dropout:** Randomly zeroes out neurons during training.

#### Impact:
- Helps the model generalize better to unseen data.
### 9. Loss Function Choice

**Description:**  
The choice of loss function directly affects the learning process.

#### Impact:
- Incorrect loss functions can cause poor learning behavior and misalignment with the task.


## Use Cases
- Applying neural networks in image classification
- Using neural networks for natural language processing

---

## History

- 1943 - McCulloch & Pitts introduced the first mathematical model of a neuron, using a simple step function (binary output based on a threshold). The McCulloch-Pitts model hinted at multi-layer concepts but wasn’t used practically. Their goal was to describe how neural activity could produce logical computations.

- 1958 - The basic idea of the perceptron (a basic FFN unit) was introduced by **Frank Rosenblatt**. Early FFNs were limited to shallow architectures. Built upon the McCulloch-Pitts model to create a learning algorithm capable of pattern recognition, aiming to simulate aspects of human learning.

- 1969 - **Minsky and Papert’s book “Perceptrons”** showed the limitations of single-layer perceptrons (e.g., inability to solve XOR), which led to an *AI winter* for neural nets. The book rigorously analyzed the mathematical limits of single-layer perceptrons.
  - They proved that single-layer perceptrons cannot solve problems that are not *linearly separable*, with XOR being the most famous example. A single-layer perceptron can only solve problems where classes can be separated by a *linear boundary* (e.g., AND, OR), but XOR requires a non-linear decision boundary.
  - The book was widely interpreted as a condemnation of neural networks as a whole, even though multi-layer networks could, in theory, solve XOR (but lacked practical training algorithms like backpropagation at that time).
  - As a result, research funding dried up, and attention shifted to symbolic AI approaches for over a decade.

> **Where I’m not following:** What is a *linear decision boundary* and why is that necessary to build a neural network model? I understand linear separability conceptually but haven’t internalized why the perceptron’s computations inherently rely on it.
> **Some more context:** In the 1960s, the single-layer perceptron was one of the only practical neural network models available. When **Minsky and Papert** published *Perceptrons*, they proved that single-layer perceptrons could not solve problems like XOR, which are not linearly separable. Non-linearly separable problems—like XOR—are common in real-world classification tasks (e.g., vision, sensor data interpretation). The findings were overgeneralized as evidence that *all* neural networks were flawed. Researchers lacked effective ways to train multi-layer perceptrons (since backpropagation wasn’t widely adopted until the mid-1980s), leading to the *AI winter*.

- 1970s - Theoretical groundwork for gradient-based learning was developed but not widely applied to neural nets.

- 1970s-1980s - Researchers explored multi-layer structures (2+ layers) but lacked efficient training algorithms.

- 1980 - **Kunihiko Fukushima** introduced the *Neocognitron*, laying the foundation for modern CNNs. It used convolutional and pooling layers but lacked backpropagation.
  - The *Neocognitron* was inspired by the visual cortex and designed to recognize visual patterns hierarchically.
  - Alternated between **S-layers** (similar to convolutional layers) and **C-layers** (similar to pooling layers).
  - Used an unsupervised learning mechanism (winner-takes-all and Hebbian-like updates).

- 1986 - **Rumelhart, Hinton, and Williams** published the landmark paper on backpropagation, *“Learning Representations by Back-Propagating Errors”*.
  - This paper kicked off the modern use of multi-layer perceptrons (MLPs) and deep feed-forward networks and introduced an early concept of Recurrent Neural Networks (RNNs).
  - Led to the adoption of smooth, differentiable activations like *sigmoid* and *tanh*.

- 1989 - **Yann LeCun** applied backpropagation to CNN-like structures for digit recognition using the MNIST dataset.
  - Introduced hierarchical architectures that stacked feature extraction (convolution) layers followed by subsampling (pooling) layers, similar to the Neocognitron.

- 1990 - Jeffrey Elman formalized it with an architecture specifically designed for handling sequences over time.

- 1997 - Long Short-Term Memory networks (LSTMs) were first introduced by Sepp Hochreiter and Jürgen Schmidhuber in their paper titled “Long Short-Term Memory”. LSTMs were a direct response to a major limitation of standard RNNs: the vanishing and exploding gradient problem. Traditional RNNs struggle with learning long-term dependencies because, during backpropagation through time (BPTT), gradients either diminish to near-zero (vanish) or grow exponentially (explode) as they are propagated across many timesteps. Hochreiter and Schmidhuber designed the LSTM architecture with special components, notably memory cells and gating mechanisms (input, output, and forget gates), to regulate the flow of information and gradients across long sequences.

- 1998 - LeCun et al. introduced **LeNet-5**, one of the first successful CNN architectures, for handwritten digit recognition (MNIST).

- 2001 - Echo State Networks (ESNs) were introduced by Herbert Jaeger in his paper titled “The 'Echo State' Approach to Analysing and Training Recurrent Neural Networks”. ESNs were directly influenced by standard RNNs, not LSTMs. The core idea behind ESNs was to address the difficulties of training traditional RNNs, specifically the instability and inefficiency of training all recurrent connections via backpropagation through time (BPTT). Unlike LSTMs, which introduced complex gating mechanisms to solve the vanishing gradient problem, ESNs introduced a simpler idea: (1) Create a large, fixed, and randomly connected recurrent layer (the reservoir). (2) Only train the output weights (from the reservoir to the output layer). This approach kept the recurrent weights unchanged after initialization, making ESNs computationally cheaper to train compared to standard RNNs or LSTMs. ESNs evolved from RNNs, focusing on simplifying training and addressing stability, without the complex memory mechanisms of LSTMs. They are part of a broader class of **reservoir computing methods**.

- 2012 - **AlexNet** by **Alex Krizhevsky, Ilya Sutskever, and Geoffrey Hinton** won the ImageNet competition, drastically outperforming traditional methods.
  - Leveraged deep CNN layers, *ReLU* activations, GPU acceleration, and *dropout* regularization.
  - Triggered the *deep learning revolution* in computer vision.
  - *ReLU* began replacing sigmoid/tanh due to its simplicity and ability to avoid the vanishing gradient problem.
  - *Dropout* is a regularization technique where, during training, you randomly "drop" (set to zero) certain neurons in the network to prevent overfitting and encourage redundancy.

- 2013 - OverFeat was introduced by Pierre Sermanet, David Eigen, Xiang Zhang, Michael Mathieu, Rob Fergus, and Yann LeCun in their paper titled “OverFeat: Integrated Recognition, Localization and Detection using Convolutional Networks”. It was an early end-to-end system that extended CNNs beyond simple image classification to include object localization and detection, all within a unified CNN framework. OverFeat built upon previous CNN architectures (like LeNet and AlexNet) but applied sliding window techniques using CNNs to simultaneously handle Image classification, Bounding box prediction (localization), Object detection. OverFeat was one of the early systems to integrate detection into CNNs, before modern object detectors like YOLO (2016) and Faster R-CNN (2015) became widespread. It leveraged pre-trained CNNs and demonstrated that CNNs could generalize well to not just classification tasks but also spatial prediction tasks like localization and detection.

- 2013 - The Network-in-Network (NiN) framework was introduced in 2013 by Min Lin, Qiang Chen, and Shuicheng Yan in their paper titled "Network In Network." It proposed a modification to traditional convolutional neural networks (CNNs) by replacing the standard linear convolutional filters with small multilayer perceptrons (MLPs), applied at each spatial location using 1x1 convolutions. This made the model more expressive by enabling complex, non-linear transformations within local receptive fields. NiN addressed the limitation of CNNs relying on shallow feature extractors by increasing local modeling power without significantly increasing parameters. It also introduced global average pooling to replace fully connected layers, reducing overfitting and improving generalization. While the original NiN architecture isn't widely used today, its core ideas—especially 1x1 convolutions—became foundational in later models like GoogLeNet and ResNet.

- 2014 - The VGG (Visual Geometry Group) network was first introduced in 2014 by Karen Simonyan and Andrew Zisserman, researchers from the University of Oxford's Visual Geometry Group. It was presented in their paper titled "Very Deep Convolutional Networks for Large-Scale Image Recognition," which was submitted to the International Conference on Learning Representations (ICLR) in 2015. The VGG network became widely recognized for its deep architecture, which featured up to 19 weight layers. It demonstrated that increasing network depth using very small 3x3 convolutional filters could significantly improve performance on image recognition tasks. Among the various configurations proposed, VGG-16 and VGG-19 are the most commonly used models, named for their number of weight layers.

- 2014 - GoogLeNet, based on the Inception architecture, was introduced in 2014 by researchers at Google in the paper *"Going Deeper with Convolutions"* by Christian Szegedy et al. It is a deep convolutional neural network that won the ImageNet Large Scale Visual Recognition Challenge (ILSVRC) 2014 and marked a significant advancement in CNN design. Unlike traditional CNNs that stack convolutional layers sequentially, GoogLeNet introduced the Inception module, which applies multiple filter sizes (1x1, 3x3, 5x5) in parallel to capture features at different spatial scales. To keep the model computationally efficient, it used 1x1 convolutions for dimensionality reduction, an idea inspired by the Network-in-Network framework. This allowed the network to go deeper—22 layers—while using significantly fewer parameters than previous models like AlexNet. GoogLeNet also incorporated global average pooling instead of fully connected layers at the end, reducing overfitting and improving generalization. The architecture demonstrated that CNNs could be made deeper and more expressive without increasing computational cost, influencing many later models in the field.

- 2015 - **ResNets** were introduced by **Kaiming He, Xiangyu Zhang, Shaoquing Ren, and Jian Sun**.
  - Their paper *“Deep Residual Learning for Image Recognition”* won the 2015 ImageNet Large Scale Visual Recognition Challenge.
  - **ResNet-152** significantly outperformed earlier architectures like VGGNet while using fewer parameters in many cases.



