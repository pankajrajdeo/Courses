# 📘 Lecture Notes: Serving ML Models with FastAPI

---

## 1. Recap of Previous Videos

Before today’s topic, the playlist has covered **FastAPI fundamentals** through a **Patient Management System project**.
We built endpoints for full CRUD:

* **Create** → Add a patient
* **Read** → Retrieve patients (single/all)
* **Update** → Modify patient details
* **Delete** → Remove a patient

Concepts covered so far:

* What APIs are & why they are needed
* HTTP methods (GET, POST, PUT, DELETE)
* Path parameters & Query parameters
* Request body in FastAPI
* **Pydantic** for data validation
* Building confidence in using FastAPI fundamentals

---

## 2. Today’s Agenda

We’ll now learn how to **serve a Machine Learning model** using FastAPI.
The idea: build an API endpoint around a trained ML model so users can send input and get predictions.

The lecture is divided into **three parts**:

1. **Model Building**

   * Problem statement: Insurance Premium Prediction
   * Data processing, feature engineering, training model, exporting model

2. **Serving Model via FastAPI**

   * Build `/predict` endpoint
   * Validate user input using Pydantic
   * Run inference with ML model

3. **Frontend with Streamlit**

   * Create simple UI form for users
   * Connect frontend to API
   * Display prediction results

---

## 3. Flow Diagram (Explained)

```
[Frontend: Streamlit] → sends user inputs →
[FastAPI Endpoint /predict] → validates, computes features →
[ML Model (trained)] → generates prediction →
[FastAPI] → sends JSON response →
[Frontend] → displays result to user
```

---

## 4. Part 1 – Model Building

### 4.1 Problem Statement

We want to predict **Insurance Premium Category** (`High`, `Medium`, `Low`) based on user attributes.

#### Inputs collected:

* Age
* Weight
* Height
* Income (LPA)
* Smoker (Yes/No)
* City
* Occupation

#### Output:

* Insurance Premium Category → `High` / `Medium` / `Low`

---

### 4.2 Feature Engineering

We transform raw input into **model-ready features**:

1. **BMI** = weight / (height²)
2. **Age Group** → categorical bins

   * 0–18 → Young
   * 18–45 → Adult
   * 45–60 → Middle-aged
   * 60+ → Senior
3. **Lifestyle Risk** (from BMI + Smoker)

   * High risk = BMI > 30 and Smoker
   * Medium = BMI 27–30 and Smoker
   * Low otherwise
4. **City Tier** (categorize cities into Tier 1, 2, 3)

We drop old columns (`Age`, `Weight`, `Height`, `City`, `Smoker`) once derived features are ready.

---

### 4.3 ML Algorithm

* Preprocessing:

  * One-hot encoding for categorical features
  * Pass-through for numeric features
* Model: **Random Forest Classifier**
* Pipeline:

  ```
  ColumnTransformer (encode categorical) → RandomForestClassifier
  ```
* Dataset split into train/test, trained, accuracy ≈ 90% (toy dataset).

---

### 4.4 Export Model

Save model as a `.pkl` file:

```python
import pickle

with open("model.pkl", "wb") as f:
    pickle.dump(pipeline, f)
```

---

## 5. Part 2 – Serving Model with FastAPI

We now create an API endpoint around the model.

---

### 5.1 Setup

* Place `model.pkl` in project folder
* Create `app.py` file

Install dependencies:

```bash
pip install fastapi uvicorn pydantic pandas scikit-learn
```

---

### 5.2 Load Model

```python
import pickle
from fastapi import FastAPI

with open("model.pkl", "rb") as f:
    model = pickle.load(f)

app = FastAPI()
```

---

### 5.3 Pydantic Model for Input Validation

```python
from pydantic import BaseModel, Field
from typing import Literal

class UserInput(BaseModel):
    age: int = Field(..., gt=0, le=120, description="Age of user")
    weight: float = Field(..., gt=0, description="Weight in kg")
    height: float = Field(..., gt=0, le=2.5, description="Height in meters")
    income_lpa: float = Field(..., gt=0, description="Annual income (LPA)")
    smoker: bool = Field(..., description="Is the user a smoker?")
    city: str
    occupation: Literal["Govt Job", "Private Job", "Business Owner", "Student"]
```

---

### 5.4 Computed Fields

We add derived features inside the model class:

```python
class UserInput(BaseModel):
    # same fields as above...

    @property
    def bmi(self):
        return self.weight / (self.height ** 2)

    @property
    def age_group(self):
        if self.age < 18: return "Young"
        elif self.age < 45: return "Adult"
        elif self.age < 60: return "Middle-Aged"
        else: return "Senior"

    @property
    def lifestyle_risk(self):
        if self.smoker and self.bmi > 30:
            return "High"
        elif self.smoker and self.bmi > 27:
            return "Medium"
        else:
            return "Low"

    @property
    def city_tier(self):
        tier1 = ["Delhi", "Mumbai", "Bangalore"]
        tier2 = ["Pune", "Chennai", "Hyderabad"]
        if self.city in tier1: return 1
        elif self.city in tier2: return 2
        else: return 3
```

---

### 5.5 Prediction Endpoint

```python
import pandas as pd
from fastapi.responses import JSONResponse

@app.post("/predict")
def predict_premium(data: UserInput):
    input_df = pd.DataFrame([{
        "BMI": data.bmi,
        "AgeGroup": data.age_group,
        "LifestyleRisk": data.lifestyle_risk,
        "CityTier": data.city_tier,
        "IncomeLPA": data.income_lpa,
        "Occupation": data.occupation
    }])

    prediction = model.predict(input_df)[0]

    return JSONResponse(
        status_code=200,
        content={"predicted_category": prediction}
    )
```

---

### 5.6 Run Server

```bash
uvicorn app:app --reload
```

Test via Swagger Docs at:
👉 `http://127.0.0.1:8000/docs`

---

## 6. Part 3 – Frontend with Streamlit

Instead of HTML/CSS/JS, we use **Streamlit** for simplicity.

---

### 6.1 Install

```bash
pip install streamlit requests
```

---

### 6.2 Frontend Code (`front.py`)

```python
import streamlit as st
import requests

API_URL = "http://127.0.0.1:8000/predict"

st.title("Insurance Premium Prediction")
st.write("Enter your details below:")

age = st.number_input("Age", min_value=1, max_value=120, value=30)
weight = st.number_input("Weight (kg)", min_value=1.0, value=70.0)
height = st.number_input("Height (m)", min_value=1.0, max_value=2.5, value=1.75)
income = st.number_input("Income (LPA)", min_value=1.0, value=10.0)
smoker = st.selectbox("Are you a smoker?", [True, False])
city = st.text_input("City", "Mumbai")
occupation = st.selectbox("Occupation", ["Govt Job", "Private Job", "Business Owner", "Student"])

if st.button("Predict Premium Category"):
    data = {
        "age": age,
        "weight": weight,
        "height": height,
        "income_lpa": income,
        "smoker": smoker,
        "city": city,
        "occupation": occupation
    }
    response = requests.post(API_URL, json=data)

    if response.status_code == 200:
        st.success(f"Predicted Premium: {response.json()['predicted_category']}")
    else:
        st.error("Error while predicting!")
```

---

### 6.3 Run Frontend

```bash
streamlit run front.py
```

👉 Opens a local web app with input form and prediction results.

---

## 7. Key Takeaways

* **Model building**: Train and export ML model (`.pkl`)
* **FastAPI**: Wrap model in `/predict` endpoint with POST method
* **Pydantic**: Validate raw inputs & compute derived features
* **Streamlit frontend**: Simple UI to interact with API

This flow is the **foundation of ML model deployment** – applicable to real-world projects.

---

✅ With this setup, you now know how to:

1. Train and export ML models
2. Serve them with FastAPI
3. Build frontend to consume the API
