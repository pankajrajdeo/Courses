# Lecture 5 Notes – Training the Perceptron (100 Days of Deep Learning)

## 1. Recap from Previous Lecture

* **Perceptron**: A single-layer linear classifier that maps inputs to outputs using weights and a bias.
* **Prediction**:

  * Compute a weighted sum of inputs.
  * Apply a threshold: if sum ≥ 0 → predict **1**, else predict **0**.
* Last lecture: We learned what a perceptron is and how prediction works.
* **Today’s focus**: How to **train** perceptron weights and bias.

---

## 2. Linearly Separable Data

* If data is **linearly separable**, we can find a line (2D) or hyperplane (higher dimensions) to split classes.
* Training = adjusting weights and bias until the decision boundary correctly separates the points.

---

## 3. Decision Boundary

Equation of a line (2D perceptron):

$$
w_0 + w_1x_1 + w_2x_2 = 0
$$

* $w_0$ = bias (intercept)
* $w_1, w_2$ = weights for input features $x_1, x_2$

### Regions:

* If $w_0 + w_1x_1 + w_2x_2 > 0$ → **positive region**
* If $w_0 + w_1x_1 + w_2x_2 < 0$ → **negative region**

**Tip**: To check which side a point lies on, substitute its coordinates into the equation.

---

## 4. Transformations of the Line

Changing coefficients changes the decision boundary:

1. **Change bias $w_0$** → shifts the line up/down (parallel move).
2. **Change weights $w_1, w_2$** → rotates the line.
3. Combination of both → adjusts orientation + position.

---

## 5. Misclassified Points

* If a **negative point lies in positive region** → misclassification.
* If a **positive point lies in negative region** → misclassification.

### Strategy:

* Adjust weights **slightly** in the direction of the misclassified point to “pull” the boundary.
* Use a **learning rate (η)** to control step size.

---

## 6. Perceptron Update Rule

For each misclassified point $(x, y)$:

* Prediction:

  $$
  \hat{y} = 
  \begin{cases} 
  1 & \text{if } (w \cdot x) \geq 0 \\
  0 & \text{otherwise}
  \end{cases}
  $$

* Update:

  $$
  w_{\text{new}} = w_{\text{old}} + \eta \cdot (y - \hat{y}) \cdot x
  $$

Where:

* $y$ = true label
* $\hat{y}$ = predicted label
* $\eta$ = learning rate (small positive value, e.g., 0.1)
* $x$ = input vector (with bias term included as $x_0 = 1$)

---

## 7. Training Algorithm (Pseudocode)

```text
Initialize weights w randomly
Choose a learning rate η

for epoch in range(num_epochs):
    Select a random training example (x, y)
    Compute prediction: y_hat = step(w · x)
    Update weights: w = w + η * (y - y_hat) * x
```

* Repeat until convergence (all points correctly classified or epochs complete).

---

## 8. Python Code Example

```python
import numpy as np

def perceptron(X, y, lr=0.1, epochs=1000):
    # Add bias column (x0 = 1)
    X = np.insert(X, 0, 1, axis=1)
    
    # Initialize weights (w0, w1, w2)
    w = np.zeros(X.shape[1])
    
    for _ in range(epochs):
        i = np.random.randint(0, len(y))   # random point
        xi, yi = X[i], y[i]
        y_hat = 1 if np.dot(w, xi) >= 0 else 0
        w = w + lr * (yi - y_hat) * xi
    
    return w

# Example dataset
X = np.array([
    [2, 3],
    [1, 1],
    [2, 1],
    [3, 1]
])
y = np.array([1, 0, 0, 1])

weights = perceptron(X, y)
print("Trained Weights:", weights)
```

---

## 9. Understanding the Output

* Final weights define the **decision boundary**.
* In 2D:
  If equation is $w_0 + w_1x_1 + w_2x_2 = 0$, slope = $-w_1 / w_2$, intercept = $-w_0 / w_2$.

---

## 10. Key Takeaways

* **Perceptron learning** iteratively fixes misclassified points.
* Updates are controlled by the **learning rate** (small steps to avoid overshooting).
* Algorithm guarantees convergence if data is **linearly separable**.
* Foundation for more complex algorithms like logistic regression and neural networks.
