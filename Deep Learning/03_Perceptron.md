# Lecture Notes: Introduction to Perceptron

---

## 1. Motivation and Context

* **Why Perceptron first?**

  * It is the **fundamental building block** of Artificial Neural Networks (ANNs).
  * Understanding a single Perceptron makes it easier to understand **Multi-Layer Perceptrons (MLPs)** and deeper architectures.
  * Jumping directly into MLPs without Perceptron weakens conceptual clarity.

* **Perceptron in AI history:**

  * Introduced by **Frank Rosenblatt (1958)**.
  * Early success but had limitations (e.g., inability to solve non-linear problems).
  * Inspired the later development of MLPs and modern deep learning.

---

## 2. What is a Perceptron?

* A **Perceptron is both an algorithm and a mathematical model**.
* Works in **Supervised Learning** (classification tasks).
* Can be seen as:

  * A **linear classifier**.
  * A **mathematical function** that maps inputs to outputs.
  * A **building block of neural networks**.

---

## 3. Perceptron Architecture

### Components:

1. **Inputs**:

   * $x_1, x_2, ..., x_n$ (features such as IQ, CGPA, etc.).
   * An extra input = constant $1$, used for **bias**.

2. **Weights**:

   * $w_1, w_2, ..., w_n$ – learnable parameters controlling importance of each input.

3. **Bias (b)**:

   * A special parameter allowing the decision boundary to shift.

4. **Summation Function**:

   * Computes weighted sum:

   $$
   z = (w_1 \cdot x_1) + (w_2 \cdot x_2) + ... + (w_n \cdot x_n) + b
   $$

5. **Activation Function**:

   * Transforms $z$ into output $y$.
   * Example: **Step function**

     $$
     y = 
     \begin{cases} 
     1 & \text{if } z \geq 0 \\
     0 & \text{if } z < 0
     \end{cases}
     $$

---

## 4. Perceptron Working Example

### Student Placement Prediction

* Inputs:

  * $x_1 = $ IQ
  * $x_2 = $ CGPA

* Output:

  * $y = 1$ → Placement happened
  * $y = 0$ → Placement did not happen

* Weighted sum:

  $$
  z = w_1 \cdot \text{IQ} + w_2 \cdot \text{CGPA} + b
  $$

* Prediction:

  * If $z \geq 0 $, predict placement.
  * If $z < 0 $, predict no placement.

---

### Code Illustration (Prediction)

```python
import numpy as np

# Example weights and bias from training
w1, w2, b = 2, 2, 3  

# New student data
IQ = 100
CGPA = 5.1

# Step function
def step_function(z):
    return 1 if z >= 0 else 0

# Weighted sum
z = w1 * IQ + w2 * CGPA + b
y = step_function(z)

print("Prediction (1 = Placement, 0 = No Placement):", y)
```

---

## 5. Multiple Inputs Extension

* If dataset has 3 features (e.g., IQ, CGPA, 12th marks):

  $$
  z = w_1 x_1 + w_2 x_2 + w_3 x_3 + b
  $$

* General form for $n$ features:

  $$
  z = \sum_{i=1}^{n} w_i x_i + b
  $$

---

## 6. Biological Inspiration

* **Perceptron vs Biological Neuron**

  * **Neuron**: Dendrites (inputs), Nucleus (processing), Axon (output).
  * **Perceptron**: Inputs ($x_i$), Summation + Activation, Output ($y$).

* **Key Differences**:

  1. Biological neuron = highly complex, electrochemical processes.
     Perceptron = simple mathematical model.
  2. Biological neurons have **neuroplasticity** (connections grow/shrink).
     Perceptron weights do not dynamically evolve this way.
  3. Biological neurons connect in billions.
     Perceptron is a simplified abstraction.

---

## 7. Geometrical Interpretation

* Equation of perceptron:

  $$
  z = w_1 x_1 + w_2 x_2 + b
  $$

* This is the **equation of a line** in 2D.

* Generalization:

  * In **3D**, perceptron defines a **plane**.
  * In **nD**, perceptron defines a **hyperplane**.

* **Role of the hyperplane**:

  * Divides the input space into **two regions** (binary classification).
  * Region 1 → Class 1.
  * Region 2 → Class 0.

* **Limitation**:

  * Works only for **linearly separable data**.
  * Fails on non-linear datasets (e.g., XOR problem).

---

### Example Visualization in Python

```python
import matplotlib.pyplot as plt
from sklearn.linear_model import Perceptron
from mlxtend.plotting import plot_decision_regions
import numpy as np

# Example dataset: [CGPA, Resume Score]
X = np.array([
    [8.5, 9], [7, 7.5], [9, 8.5],   # placed
    [4.5, 5], [6, 5.5], [5, 6]      # not placed
])

# Labels: 1 = Placed, 0 = Not Placed
y = np.array([1, 1, 1, 0, 0, 0])

# Perceptron model
clf = Perceptron()
clf.fit(X, y)

# Decision boundary plot
plot_decision_regions(X, y, clf=clf, legend=2)
plt.xlabel("CGPA")
plt.ylabel("Resume Score")
plt.title("Perceptron Decision Boundary")
plt.show()
```

---

## 8. Interpretation of Weights

* Weights show **feature importance**.
* Example:

  * If $w_1 = 2$ (IQ) and $w_2 = 4$ (CGPA),
  * CGPA is **twice as important** as IQ in predicting placement.

---

## 9. Limitations of Perceptron

* Only handles **binary classification**.
* Only works for **linearly separable datasets**.
* Cannot solve problems like **XOR dataset**.
* These limitations led to the invention of **Multi-Layer Perceptrons (MLPs)**.

---

## 10. Summary

* **Perceptron = Linear Classifier + Step Activation.**
* Inspired by biological neurons but simplified.
* Works by creating a line/plane/hyperplane to separate classes.
* Feature importance is encoded in weights.
* **Key limitation**: cannot handle non-linear problems → motivated deeper networks.

---

✅ Next Steps in Lecture Series:

1. Training process of perceptron (weight update rule, learning rate).
2. Limitations in detail (XOR example).
3. Introduction to Multi-Layer Perceptrons.
