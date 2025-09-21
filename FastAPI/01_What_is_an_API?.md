# 📘 Lecture Notes: Introduction to APIs & Why FastAPI Matters

---

## 1. 📌 Context of the Playlist

* **Creator’s Vision**: Provide complete learning resources around AI.
* Channel already has playlists on:

  * Machine Learning (ML)
  * Deep Learning (DL)
  * Natural Language Processing (NLP)
* \~50% of AI curriculum is covered.
* Missing but crucial topic → **FastAPI**.

### Why FastAPI?

* After building ML/DL/Generative AI models, we must **serve them to users**.
* Serving requires:

  * Websites
  * Mobile apps
  * APIs (as the bridge)
* **FastAPI** is the most adopted framework for creating robust, scalable, industry-grade APIs in ML/AI.
* 9/10 ML product companies use FastAPI for APIs.

---

## 2. 📌 Playlist Structure

The course is divided into **3 parts**:

1. **Fundamentals of FastAPI**

   * Learn FastAPI basics using a small project.
   * Focus: strong foundation before moving to ML use cases.

2. **FastAPI + ML**

   * Take an existing ML model.
   * Build an API around it using FastAPI.
   * Possibly connect the API to a website.

3. **Deployment**

   * Dockerize the FastAPI app.
   * Deploy on **AWS** (industry-standard cloud service).

📅 **Duration**: \~15 videos, covered in \~21–25 days.

---

## 3. 📌 First Concept: What is an API?

### Definition

> **APIs are mechanisms that enable two software components (like frontend & backend) to communicate with each other using defined rules, protocols, and data formats.**

### Key Idea

* **API = Connector** between two software systems.

---

### Example: Udemy Course Search

1. User searches “AI Agents” on Udemy website (frontend).
2. Request goes to API.
3. API passes it to **backend** → backend queries the database.
4. API gets results back, sends them to **frontend**.
5. Frontend displays results.

✅ **Protocols & Formats**:

* Communication protocol: **HTTP**
* Response format: **JSON**

---

### Analogy: Restaurant

* Customer = **Frontend**
* Kitchen (chef) = **Backend**
* Waiter = **API**
* Menu = **Protocol / rules**

The waiter (API) carries customer requests to the kitchen and returns results.

---

## 4. 📌 Pre-API Era: Monolithic Architecture

### Monolithic Workflow

* Database + Backend + Frontend = Single tightly coupled app.
* Example: Early IRCTC website:

  * Database: Train schedules
  * Backend: Function `fetch_trains(from, to, date)`
  * Frontend: Web form
  * All inside **one project folder**.

➡️ Communication worked **without APIs** because everything was in one codebase.

### Problems with Monolithic Apps

1. **Tight coupling** → any change affects entire system.
2. Cannot share data with third-party apps (e.g., MakeMyTrip, Yatra).
3. Multiple versions (web, Android, iOS) meant redundant development & maintenance.

---

## 5. 📌 Why APIs Solved the Problem

### Step 1: Decoupling

* Separate **Backend** and **Frontend**.
* Backend becomes an independent app with functions like `fetch_trains`.

### Step 2: Add API Layer

* Backend functions are **exposed as API endpoints**.
* Example endpoint:

  ```
  GET https://irctc.com/trains?from=DELHI&to=MUMBAI&date=2025-09-20
  ```
* Anyone (frontend, MakeMyTrip, Yatra, etc.) can access this endpoint.
* Backend never directly exposed.

---

### Benefits

1. **Security** – API validates requests, backend not exposed directly.
2. **Reusability** – One backend, multiple frontends.
3. **Standardized communication** – via HTTP & JSON.
4. **Scalability** – Easy to add more consumers (apps/websites).

---

### JSON Format

* Chosen because it’s **universal** (understood by Java, Python, PHP, etc.).
* Example Response:

```json
{
  "trains": [
    {"train_no": "12345", "name": "Rajdhani Express", "time": "09:00"},
    {"train_no": "54321", "name": "Shatabdi Express", "time": "14:00"}
  ]
}
```

---

## 6. 📌 API in Multi-Platform Applications

**Problem**: 1 product → multiple versions (Web, Android, iOS).
Without APIs → 3 backends, 3 databases, triple effort.

**Solution**:

* One **Database**
* One **Backend**
* One **API Layer**
* Multiple **Frontends** (Web, Android, iOS) connect via APIs.

✅ Example: Google, Uber, Zomato follow this pattern.

---

## 7. 📌 API in Machine Learning (ML Perspective)

### Difference from Software Case

* Traditional software: Backend interacts with **Database**.
* ML software: Backend interacts with **ML Model** (stored as file).

### Example: ChatGPT

1. **Model**: GPT (LLM) trained on massive data.
2. **Backend**: Functions like `predict(query)`.
3. **Frontend**: Chat UI where user types.
4. **API Layer**: Exposes endpoints like `/predict`.

---

### ML Workflow (with API)

* User query → API → Backend → Model → Prediction → API → JSON response → Frontend.

### JSON Example for ML

```json
{
  "response": "Hello! How can I help you today?"
}
```

---

### Benefits for ML

1. Allow **external apps** to use the ML model (e.g., Zomato using GPT for chatbots).
2. Same ML model can serve:

   * Website
   * Android app
   * iOS app

✅ Amazon Recommendation System works this way.

---

## 8. 📌 Key Takeaways

* **API = connector** between software components.
* Solves **tight coupling** & enables **sharing** of backend/model logic.
* Communication uses **HTTP protocol** + **JSON format**.
* In ML, API exposes **model predictions** to the world.
* **Modern AI apps (ChatGPT, Amazon Recommender, Zomato bots)** all rely on APIs.

---

## 9. 📌 Preview of Next Lecture

* **Topic**: Introduction to **FastAPI**

  * Benefits over other frameworks.
  * Setup environment.
  * Build first API.

---

# 📂 Code Examples

### Example 1: Simple API with Flask (pre-FastAPI era)

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/trains', methods=['GET'])
def get_trains():
    from_station = request.args.get('from')
    to_station = request.args.get('to')
    date = request.args.get('date')
    
    # mock data (instead of database)
    trains = [
        {"train_no": "12345", "name": "Rajdhani Express", "time": "09:00"},
        {"train_no": "54321", "name": "Shatabdi Express", "time": "14:00"}
    ]
    
    return jsonify({"trains": trains})

if __name__ == "__main__":
    app.run(debug=True)
```

---

### Example 2: Simple ML API (pseudo-backend)

```python
from flask import Flask, request, jsonify
import joblib

app = Flask(__name__)

# Load trained ML model (assume saved with joblib)
model = joblib.load("spam_classifier.pkl")

@app.route('/predict', methods=['POST'])
def predict():
    data = request.get_json()
    text = data['message']
    
    prediction = model.predict([text])[0]
    return jsonify({"prediction": prediction})

if __name__ == "__main__":
    app.run(debug=True)
```

---

✅ In later lectures, these will be done using **FastAPI**, which is faster, cleaner, and more scalable.
