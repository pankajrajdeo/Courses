# Lecture Notes: Limitations of the Perceptron

## 1. Introduction

* **Perceptron Recap**:

  * A perceptron is one of the earliest neural network models.
  * It works well for **linearly separable data**.
  * It fails on **non-linear data** (e.g., XOR).
* **Goal of Lecture**:

  * Show practically (with code and plots) why the perceptron cannot handle non-linear data.
  * Prepare for the motivation behind **multi-layer networks (MLPs)**.

---

## 2. Why the Perceptron Fails on Non-Linear Data

* **Linear Separability**:

  * The perceptron draws a straight line (hyperplane in higher dimensions) to separate classes.
  * Works if a single straight line can classify the dataset.
* **Problem**:

  * Many real-world problems involve data that cannot be separated by a single straight line.
  * Example: **XOR dataset**.
  * In XOR, no matter how the line is drawn, at least one data point will always be misclassified.

---

## 3. Datasets Used in Demonstration

The lecture uses **three toy datasets** to test perceptron performance:

1. **AND Dataset**

   * Truth table: Output is `1` only when both inputs are `1`.
   * Linearly separable.

   | Input X1 | Input X2 | Output |
   | -------- | -------- | ------ |
   | 0        | 0        | 0      |
   | 0        | 1        | 0      |
   | 1        | 0        | 0      |
   | 1        | 1        | 1      |

2. **OR Dataset**

   * Truth table: Output is `1` if either input is `1`.
   * Linearly separable.

   | Input X1 | Input X2 | Output |
   | -------- | -------- | ------ |
   | 0        | 0        | 0      |
   | 0        | 1        | 1      |
   | 1        | 0        | 1      |
   | 1        | 1        | 1      |

3. **XOR Dataset**

   * Truth table: Output is `1` only if inputs are **different**.
   * **Not linearly separable.**

   | Input X1 | Input X2 | Output |
   | -------- | -------- | ------ |
   | 0        | 0        | 0      |
   | 0        | 1        | 1      |
   | 1        | 0        | 1      |
   | 1        | 1        | 0      |

---

## 4. Python Code Demonstration

### Imports and Setup

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import Perceptron

# AND, OR, XOR datasets
X = np.array([[0,0], [0,1], [1,0], [1,1]])
y_and = np.array([0, 0, 0, 1])
y_or  = np.array([0, 1, 1, 1])
y_xor = np.array([0, 1, 1, 0])
```

### Training Perceptrons

```python
# Train on AND dataset
clf_and = Perceptron(max_iter=1000, tol=1e-3)
clf_and.fit(X, y_and)

# Train on OR dataset
clf_or = Perceptron(max_iter=1000, tol=1e-3)
clf_or.fit(X, y_or)

# Train on XOR dataset
clf_xor = Perceptron(max_iter=1000, tol=1e-3)
clf_xor.fit(X, y_xor)
```

### Predictions

```python
print("AND Predictions:", clf_and.predict(X))
print("OR Predictions:", clf_or.predict(X))
print("XOR Predictions:", clf_xor.predict(X))
```

* **Expected Result**:

  * AND → Correct classification
  * OR → Correct classification
  * XOR → **Fails**, misclassifications remain no matter how long it trains

---

## 5. Visualizing the Decision Boundaries

```python
def plot_decision_boundary(clf, X, y, title):
    x_min, x_max = X[:, 0].min()-1, X[:, 0].max()+1
    y_min, y_max = X[:, 1].min()-1, X[:, 1].max()+1
    xx, yy = np.meshgrid(np.arange(x_min, x_max, 0.01),
                         np.arange(y_min, y_max, 0.01))
    
    Z = clf.predict(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)
    
    plt.contourf(xx, yy, Z, alpha=0.2)
    plt.scatter(X[:, 0], X[:, 1], c=y, s=100, edgecolors="k")
    plt.title(title)
    plt.show()

# Plot for each dataset
plot_decision_boundary(clf_and, X, y_and, "Perceptron on AND")
plot_decision_boundary(clf_or, X, y_or, "Perceptron on OR")
plot_decision_boundary(clf_xor, X, y_xor, "Perceptron on XOR")
```

* **Observations**:

  * For **AND** and **OR**, perceptron draws a clean straight line separating classes.
  * For **XOR**, no line can separate the classes → perceptron fails.

---

## 6. Key Takeaways

1. **Perceptron works only for linearly separable data.**

   * Works for AND, OR.
   * Fails for XOR.
2. **Limitation is structural**:

   * A single perceptron cannot represent non-linear decision boundaries.
3. **Motivation for Multi-Layer Perceptrons (MLPs):**

   * By stacking perceptrons (layers) and adding non-linear activation functions, we can approximate **any function**, including XOR.
   * This is why deep learning (multi-layered networks) became powerful and popular.

---

## 7. Next Steps

* This lecture sets the stage for **Multi-Layer Perceptrons (MLPs)**.
* Next topic: how adding **hidden layers** and **non-linear activations** overcomes perceptron limitations.
