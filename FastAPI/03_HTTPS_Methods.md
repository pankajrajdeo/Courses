# 📘 Lecture Notes: Introduction to FastAPI

## 1. Background & Motivation

* FastAPI is a **modern, high-performance web framework for building APIs with Python**.
* Ideal for **Data Science, AI, and ML developers** who often need to expose models as APIs.
* Existing frameworks (like Flask, Django REST) had **limitations**:

  * **Performance issues** → slower response times, not ideal for large-scale apps.
  * **Boilerplate code** → developers needed to write more repetitive code.

### Why FastAPI?

* Two core philosophies:

  1. **Fast to Run** → APIs handle requests with low latency, support concurrency, scale well.
  2. **Fast to Code** → minimal boilerplate, built-in features like validation & docs.

---

## 2. Internal Architecture of FastAPI

FastAPI is built on top of two major Python libraries:

* **Starlette** → handles HTTP request/response cycle.
* **Pydantic** → handles input validation and type checking.

### Role of Starlette

* Manages how requests are received and responses are sent.
* Example:

  * Client sends an HTTP request → Starlette routes it to the correct endpoint.
  * API returns a response → Starlette formats it as HTTP and sends back.

### Role of Pydantic

* Provides **data validation** and **type enforcement**.
* Example:

  * API expects `station_name: str` and `date: str`.
  * If client sends incorrect types (e.g., int instead of string), Pydantic raises error.
* Prevents invalid data from reaching your business logic.

---

## 3. Core Philosophies of FastAPI

### A. Fast to Run

* **Traditional API request cycle**:

  1. Client sends HTTP request.
  2. Web server receives it.
  3. Server Gateway Interface (SGI) translates request → Python-understandable.
  4. API code executes business logic.
  5. SGI translates Python response → HTTP.
  6. Response sent back to client.

* **Flask** uses:

  * **WSGI (Web Server Gateway Interface)** → synchronous protocol.
  * Server: **Gunicorn**.
  * Library: **Werkzeug** for WSGI.
  * Problem → **Blocking architecture**:

    * Handles one request at a time.
    * Other requests must wait → latency.

* **FastAPI** uses:

  * **ASGI (Asynchronous Server Gateway Interface)** → async protocol.
  * Server: **Uvicorn** (high-performance ASGI server).
  * Library: **Starlette**.
  * Supports **async/await** → can handle multiple requests concurrently.
  * Ideal for:

    * Real-time features (e.g., WebSockets).
    * ML/AI inference (where some requests take longer).

📌 **Analogy**:

* **Flask waiter** → takes one order, waits until kitchen finishes, then goes to next customer.
* **FastAPI waiter** → takes order, passes it to kitchen, immediately serves another customer while waiting.

---

### B. Fast to Code

Three main features:

1. **Automatic Input Validation**

   * Built-in with Pydantic.
   * Define expected types in endpoint → FastAPI validates automatically.

2. **Auto-generated Documentation**

   * Two types:

     * Swagger UI (`/docs`) → interactive API docs.
     * ReDoc (`/redoc`) → alternative documentation view.
   * Developers & users can test APIs without Postman.

3. **Modern Integrations**

   * Seamless with ML & DL libraries → scikit-learn, TensorFlow, PyTorch.
   * Supports OAuth for authentication.
   * Integrates with SQLAlchemy (databases).
   * Works with Docker & Kubernetes for deployment.

---

## 4. Installing and Setting Up FastAPI

### Step 1: Create Project Folder

```bash
mkdir api_tutorials
cd api_tutorials
```

### Step 2: Create Virtual Environment

```bash
python -m venv myenv
myenv\Scripts\activate   # On Windows
source myenv/bin/activate # On Mac/Linux
```

### Step 3: Install Dependencies

```bash
pip install fastapi uvicorn pydantic
```

---

## 5. Writing Your First FastAPI App

### File: `main.py`

```python
from fastapi import FastAPI

app = FastAPI()

# Home Route
@app.get("/")
def hello():
    return {"message": "Hello World"}

# About Route
@app.get("/about")
def about():
    return {"message": "CampusX is an education platform where you can learn AI."}
```

### Run the App

```bash
uvicorn main:app --reload
```

* `main` → filename (`main.py`).
* `app` → FastAPI instance.
* `--reload` → auto-reload server when code changes.

### Access in Browser

* `http://127.0.0.1:8000/` → returns `{ "message": "Hello World" }`.
* `http://127.0.0.1:8000/about` → returns `{ "message": "CampusX is an education platform where you can learn AI." }`.

---

## 6. Exploring Auto-Generated Docs

* Swagger UI: `http://127.0.0.1:8000/docs`
* ReDoc: `http://127.0.0.1:8000/redoc`

Features:

* Lists all endpoints with methods (`GET`, `POST`).
* Shows request/response format.
* Interactive testing → "Try it out" feature.

---

## 7. Summary of Flask vs FastAPI

| Feature        | Flask (WSGI)                      | FastAPI (ASGI)                             |
| -------------- | --------------------------------- | ------------------------------------------ |
| Protocol       | WSGI (sync)                       | ASGI (async)                               |
| Server         | Gunicorn                          | Uvicorn                                    |
| Validation     | Manual (with WTForms/Marshmallow) | Built-in with Pydantic                     |
| Docs           | Manual (Swagger via plugins)      | Auto-generated (`/docs`)                   |
| Concurrency    | Limited (blocking)                | High (async/await)                         |
| Best Use Cases | Simple apps, legacy APIs          | Modern, scalable APIs (ML, real-time apps) |

---

## 8. Key Takeaways

* FastAPI is **faster to run** (async) and **faster to code** (automatic validation + docs).
* Built on **Starlette (requests/responses)** and **Pydantic (validation)**.
* First app can be written in just a few lines of code.
* Great for **modern web APIs, ML model serving, and scalable applications**.

---

✅ Next steps (as per lecture):

* Practice by creating more endpoints.
* Use `/docs` to interact with your API.
* Prepare for upcoming project-based learning.
