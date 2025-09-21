# 📘 Lecture Notes: Introduction to PyTorch, LLMs, and GenAI

---

## 1. Context: Why Learn PyTorch for Generative AI (GenAI)

* **AI boom**: Currently, there is massive growth in AI, especially around **Generative AI (GenAI)** and **Large Language Models (LLMs)**.
* **Industry demand**: Startups and companies are actively hiring **GenAI engineers**.
* **Core technology**: PyTorch has become the **de facto deep learning framework** used to build GenAI and LLM-based applications.

**Instructor’s goal**:

* To help learners gain enough confidence in PyTorch so they can build GenAI/LLM applications without difficulty.
* Playlist roadmap:

  1. Fundamentals of PyTorch (tensors, computation graphs, autograd, etc.)
  2. Neural networks (ANN, CNN, RNN, etc.)
  3. Advanced topics (TorchScript, ONNX, quantization, deployment).

---

## 2. What is PyTorch?

* **Definition**:
  PyTorch is an **open-source, Python-based deep learning library** for building and training neural networks.
* **Key advantage**: Combines:

  * Torch’s **powerful tensor computations**
  * Python’s **simplicity & ecosystem**

---

## 3. Origin Story of PyTorch

* **2002**: Torch framework introduced.

  * Strength: tensor-based operations on both **CPU & GPU**.
  * Adopted for early deep learning research (e.g., CNN implementations like AlexNet, VGGNet).

* **Problems with Torch**:

  1. Written in **Lua** (an unpopular language).

     * Developers had to learn Lua before using Torch.
  2. Used **static computation graphs** (hard to debug/experiment with).

* **Solution (Meta AI researchers)**:

  * Merge Torch’s tensor power with Python’s popularity.
  * Create a new library: **PyTorch (2017)**.
  * Gave developers:

    * Python-based API
    * Dynamic computation graphs

---

## 4. Release Timeline of PyTorch

### 🔹 **PyTorch 0.1 (2017)**

* **Features**:

  1. Python compatibility (works seamlessly with NumPy, SciPy, etc.).
  2. **Dynamic computation graphs** (unlike TensorFlow/Torch at that time).

**Dynamic vs Static Graphs Example:**

```python
import torch

# Dynamic graph creation at runtime
x = torch.tensor(2.0, requires_grad=True)
y = torch.tensor(3.0, requires_grad=True)

z = x**2 + y**3  # Graph built dynamically here
z.backward()

print(x.grad)  # ∂z/∂x = 2x = 4
print(y.grad)  # ∂z/∂y = 3y^2 = 27
```

* **Impact**: Researchers loved it (easy debugging, flexible experimentation).
* PyTorch became the **default framework for research papers**.

---

### 🔹 **PyTorch 1.0 (2018)**

* Focus: Bridge **research ↔ production gap**.

* **Key additions**:

  1. **TorchScript**:

     * Serialize/optimize models → run them without Python dependency.

     ```python
     import torch

     class SimpleModel(torch.nn.Module):
         def __init__(self):
             super().__init__()
             self.linear = torch.nn.Linear(2, 1)

         def forward(self, x):
             return self.linear(x)

     model = SimpleModel()
     scripted = torch.jit.script(model)   # TorchScript serialization
     scripted.save("model.pt")
     ```

  2. **Merge with Caffe2**:

     * Production-ready, scalable deep learning features integrated into PyTorch.

* **Subsequent improvements**:

  * Distributed training
  * ONNX support (export models to other frameworks)
  * Quantization (reduce model size for deployment)
  * Domain libraries:

    * `torchvision` (vision), `torchtext` (NLP), `torchaudio` (audio).

---

### 🔹 **PyTorch 2.0 (2023)**

* Focus: **Performance & optimization**.
* **Improvements**:

  1. Faster execution via compiler optimizations.
  2. Higher throughput (process more data per unit time).
  3. Better hardware utilization (GPUs, TPUs).

---

## 5. Core Features of PyTorch

1. **Tensor computations**

   * Efficient multi-dimensional arrays (like NumPy but GPU-accelerated).

   ```python
   import torch
   a = torch.tensor([[1, 2], [3, 4]])
   b = torch.tensor([[5, 6], [7, 8]])
   print(a + b)  # Element-wise addition
   ```

2. **GPU acceleration**

   * Easy switching between CPU and GPU.

   ```python
   device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
   x = torch.ones((3,3)).to(device)
   print(x.device)  # cuda or cpu
   ```

3. **Dynamic computation graphs** (flexible debugging & experimentation).

4. **Autograd (Automatic Differentiation)**

   * Module `torch.autograd` computes gradients automatically.

5. **Distributed training**

   * Train across multiple GPUs/nodes.

6. **Interoperability**

   * Works with Python ecosystem (NumPy, SciPy, Pandas).
   * Export models with **ONNX**.

---

## 6. PyTorch vs TensorFlow

| Aspect            | PyTorch                                     | TensorFlow                            |
| ----------------- | ------------------------------------------- | ------------------------------------- |
| Focus (origin)    | Research                                    | Industry/Production                   |
| Graph type        | Dynamic (runtime)                           | Static (later added Eager mode)       |
| Ease of use       | More intuitive (Pythonic)                   | Steeper learning curve                |
| Deployment        | Improving (TorchScript, TorchServe, Mobile) | Stronger (TF Serving, TF Lite, TF.js) |
| Community         | Research-driven                             | Industry-driven                       |
| Customization     | Easier                                      | More abstracted                       |
| Ecosystem         | Hugging Face, Lightning, FastAI             | Keras, TF Hub, TF Extended            |
| Mobile deployment | Limited (Torch Mobile)                      | Mature (TF Lite, TF.js)               |

**Verdict**:

* **Research/LLMs/GenAI → PyTorch**
* **Production/Enterprise apps → TensorFlow**
* Best: Learn PyTorch first → then TensorFlow if needed.

---

## 7. Core PyTorch Modules

* **`torch`** → Core tensor library
* **`torch.nn`** → Neural network layers, activations, loss functions
* **`torch.optim`** → Optimizers (SGD, Adam, RMSProp)
* **`torch.autograd`** → Automatic differentiation
* **`torch.utils.data`** → Dataset and DataLoader classes
* **`torch.distributed`** → Distributed training support
* **`torch.jit`** → TorchScript (serialization)
* **`torch.cuda`** → GPU acceleration
* **`torch.quantization`** → Model compression
* **`torch.onnx`** → Export to ONNX format

**Domain-specific libraries**:

* `torchvision` → Computer Vision
* `torchtext` → NLP
* `torchaudio` → Audio
* `torchserve` → Model deployment

---

## 8. Ecosystem & Community Libraries

* **Hugging Face Transformers** → Pretrained LLMs, NLP models
* **PyTorch Lightning** → High-level API for clean training loops
* **FastAI** → Simplifies deep learning workflows
* **PyTorch Geometric** → Graph Neural Networks (GNNs)
* **PyTorch Forecasting** → Time-series forecasting
* **Skorch** → Scikit-learn + PyTorch integration

---

## 9. Real-World Usage of PyTorch

* **Microsoft Azure**: Many ML services built on PyTorch
* **Tesla Autopilot**: Vision-based systems use PyTorch
* **OpenAI**: GPT models trained with PyTorch
* **Uber**: Uses PyTorch for demand forecasting, routing optimization

---

## 10. Next Steps in Learning Path

* Learn fundamentals:

  * Tensors, computations, autograd, computation graphs
* Build simple neural networks in PyTorch
* Explore architectures: ANN, CNN, RNN, Transformers
* Advanced: TorchScript, ONNX, quantization, deployment

---

✅ **Key Takeaway**:
If your goal is **GenAI / LLM engineering**, **start with PyTorch**. It is the most widely adopted framework in research and by companies building cutting-edge AI systems.
