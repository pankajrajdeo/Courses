# 📘 Lecture Notes: Introduction to FastAPI

---

## 1. Background & Motivation

* In the previous video, the instructor introduced:

  * What **APIs** are.
  * Why APIs are needed.
  * Importance of **FastAPI** in fields like *Data Science, AI, ML*.

* Today’s lecture focuses on two things:

  1. **Deep Introduction to FastAPI**

     * What it is.
     * Its strengths.
     * Why use it instead of other frameworks (e.g., Flask).
  2. **Hands-on practice**

     * Setup FastAPI on your machine.
     * Build your **first API**.

---

## 2. What is FastAPI?

**Definition**:

> *FastAPI is a modern, high-performance web framework for building APIs with Python.*

* **Key points**:

  * Written in Python.
  * Used to build **industry-grade, high-performance APIs**.
  * Built on top of two libraries:

    * **Starlette** → Handles HTTP requests and responses.
    * **Pydantic** → Handles data validation.

---

## 3. Internal Components of FastAPI

### 3.1 Starlette

* Role: Manages **request-response cycle**.
* API flow:

  * Receives HTTP requests from clients.
  * Processes them.
  * Sends back HTTP responses.
* Example:

  * Request → FastAPI (via Starlette) → Response.

**One-liner**:

> *Starlette = “traffic controller” between client request and API response.*

---

### 3.2 Pydantic

* Role: **Data validation library**.
* Python is *dynamically typed* → no strict type checking.
* Pydantic ensures:

  * Data received is in the **correct type/format**.
  * Example: If station names should be `string`, Pydantic validates it.
* Saves developers from writing lots of manual validation code.

**One-liner**:

> *Pydantic = “data quality checker” for APIs.*

---

## 4. Core Philosophy of FastAPI

Two main reasons it was created:

1. **Fast to Run**

   * High performance.
   * Can handle concurrent users.
   * Low latency.

2. **Fast to Code**

   * Less boilerplate.
   * Easy, minimal code to build functional APIs.

---

## 5. Why FastAPI is Fast to Run

### 5.1 API Workflow (Behind the Scenes)

When deployed on a cloud service (e.g., AWS), an API has:

1. **API Code** – Implements business logic.
2. **Web Server** – Listens for HTTP requests on machine ports.

**Flow of a request**:

* Client sends an HTTP request.
* Web server receives it.
* **Translator (SGI)** converts between HTTP ↔ Python code.
* API processes it.
* Result converted back to HTTP response.
* Response sent to client.

---

### 5.2 WSGI vs ASGI

* **WSGI** (Web Server Gateway Interface):

  * Used by **Flask**.
  * **Synchronous** → One request at a time.
  * Blocking architecture → Slower with multiple requests.

* **ASGI** (Asynchronous Server Gateway Interface):

  * Used by **FastAPI**.
  * **Asynchronous** → Multiple requests handled concurrently.
  * Best for modern apps needing real-time features (e.g., WebSockets).

---

### 5.3 Components Comparison

| Component   | Flask (WSGI)               | FastAPI (ASGI)               |
| ----------- | -------------------------- | ---------------------------- |
| Protocol    | WSGI (synchronous)         | ASGI (asynchronous)          |
| Library     | Werkzeug                   | Starlette                    |
| Web server  | Gunicorn                   | Uvicorn                      |
| Code style  | Synchronous only           | Supports `async`/`await`     |
| Performance | Slower (blocking requests) | Faster (concurrent requests) |

---

### 5.4 Analogy: Waiter in a Restaurant

* **Flask**:

  * Waiter takes order → waits in kitchen until dish is ready → serves → goes to next customer.
  * Slow.
* **FastAPI**:

  * Waiter takes order → gives to kitchen → meanwhile takes more orders → serves dishes as they’re ready.
  * Efficient, concurrent.

---

## 6. Why FastAPI is Fast to Code

### 6.1 Automatic Input Validation

* Uses **Pydantic**.
* You can declare types, and FastAPI validates automatically.

Example:

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    in_stock: bool
```

* If client sends wrong type (e.g., price = "abc"), FastAPI rejects it.

---

### 6.2 Auto-generated Interactive Documentation

* At `/docs`, FastAPI automatically generates **Swagger UI**.
* Features:

  * Lists all endpoints.
  * Describes input/output.
  * Lets you test APIs directly from browser (no Postman needed).

---

### 6.3 Modern Integrations

* Works seamlessly with modern libraries:

  * ML/DL frameworks: Scikit-learn, TensorFlow, PyTorch.
  * Database: SQLAlchemy.
  * Authentication: OAuth.
  * Deployment: Docker, Kubernetes.

---

## 7. Hands-on: Setting up FastAPI

### 7.1 Environment Setup

```bash
# Create project folder
mkdir api_tutorials
cd api_tutorials

# Create virtual environment
python -m venv myenv

# Activate environment
myenv\Scripts\activate   # Windows
source myenv/bin/activate  # Mac/Linux
```

### 7.2 Install dependencies

```bash
pip install fastapi uvicorn pydantic
```

---

## 8. First FastAPI App (Hello World)

### 8.1 Code (`main.py`)

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def hello():
    return {"message": "Hello World"}

@app.get("/about")
def about():
    return {"message": "CampusX is an education platform where you can learn AI"}
```

---

### 8.2 Run the server

```bash
uvicorn main:app --reload
```

* `main` → filename (`main.py`).
* `app` → FastAPI object.
* `--reload` → Auto-restarts server when code changes.

Server output:

```
Uvicorn running on http://127.0.0.1:8000
```

---

### 8.3 Test Endpoints

* Home endpoint:
  `http://127.0.0.1:8000/`
  → Response: `{"message": "Hello World"}`
* About endpoint:
  `http://127.0.0.1:8000/about`
  → Response: `{"message": "CampusX is an education platform where you can learn AI"}`

---

### 8.4 Interactive Docs

* Open in browser:
  `http://127.0.0.1:8000/docs`

* Features:

  * Lists all endpoints (`/`, `/about`).
  * Allows testing (click “Try it out” → Execute).
  * Shows request/response format.

---

## 9. Summary

* **FastAPI Basics**

  * High-performance web framework for APIs.
  * Built on **Starlette (request handling)** and **Pydantic (validation)**.
  * Two philosophies: *Fast to run* and *Fast to code*.

* **Comparison with Flask**

  * Flask uses **WSGI (sync)** → slower.
  * FastAPI uses **ASGI (async)** → faster, concurrent.

* **Developer Benefits**

  * Auto validation.
  * Interactive docs.
  * Modern integrations.

* **Practical**

  * Setup virtual environment.
  * Install FastAPI + Uvicorn.
  * Build Hello World API.
  * Test with `/docs`.

---

✅ **Next Steps**: In upcoming lectures → build a **small project** with FastAPI to explore advanced features (parameters, POST requests, database integration, authentication, etc.).
