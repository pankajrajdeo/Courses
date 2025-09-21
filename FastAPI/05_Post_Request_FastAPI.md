# 📘 Lecture Notes: FastAPI – Creating the `Create` Endpoint (CRUD)

## 1. Recap of Previous Videos

Before diving into **Create**, here’s what has been covered so far:

* **Video 1**:

  * Introduction to **APIs** (what they are, why they are useful).
* **Video 2**:

  * Fundamentals of **FastAPI**.
  * Compared FastAPI with Flask.
  * Discussed **strengths and weaknesses**.
* **Video 3+**: Started a **Patient Management System** project.

  * Goal: build an API for **CRUD operations** (Create, Retrieve, Update, Delete).
  * Database: simple **JSON file**.
  * Implemented endpoints:

    1. `/hello` → toy endpoint returning "hello".
    2. `/project` → description endpoint.
    3. `/view` → retrieves all patients from JSON file. (Learned **Path Parameter**)
    4. `/patient/{id}` → retrieves specific patient by ID.
    5. `/sort` → returns patients sorted by height, weight, or BMI using **Query Parameters**.

✅ So far, **Retrieve (R)** in CRUD is complete.
➡️ Today’s lecture: implement **Create (C)**.

---

## 2. Prerequisite: **Pydantic**

* FastAPI depends heavily on two libraries:

  * **Starlette** → handles web requests/responses.
  * **Pydantic** → handles **data validation** and **serialization**.
* **Pydantic** ensures:

  * Correct **data types** (e.g., `age` should be `int`, not `"30"` string).
  * **Constraints** (e.g., age must be > 0).
  * **Computed fields** can be auto-generated (e.g., BMI).

👉 If not familiar with Pydantic, watch a **Pydantic crash course** first.

---

## 3. Plan for the `Create` Endpoint

Steps for implementing:

1. **Client sends data**

   * Sends an **HTTP POST request** (not GET).
   * Includes **Request Body** with patient details in **JSON**.
   * Example request body:

     ```json
     {
       "id": "P006",
       "name": "Rahul Gupta",
       "city": "Hyderabad",
       "age": 19,
       "gender": "male",
       "height": 1.74,
       "weight": 55
     }
     ```

   🔑 Definition:

   * **Request Body** = part of an HTTP request containing data sent from client → server (typically JSON or XML in APIs).
   * Used in **POST** (create) and **PUT** (update).

2. **Validate the data**

   * Use **Pydantic Model** to enforce:

     * Field requirements.
     * Value ranges.
     * Allowed options.
   * Invalid data → raise error automatically.

3. **Save to Database (JSON file)**

   * If **ID already exists** → raise error (`400 Bad Request`).
   * If new → add patient to JSON, auto-compute **BMI** + **Verdict**, and save.

---

## 4. Building the Pydantic Model

### Step 1: Import required classes

```python
from pydantic import BaseModel, Field, computed_field
from typing import Annotated, Literal
```

### Step 2: Define Patient model

```python
class Patient(BaseModel):
    id: Annotated[str, Field(
        description="Unique patient ID",
        example="P01"
    )]
    name: Annotated[str, Field(
        description="Name of the patient"
    )]
    city: Annotated[str, Field(
        description="City where the patient lives"
    )]
    age: Annotated[int, Field(
        gt=0, lt=120,
        description="Age of the patient"
    )]
    gender: Annotated[Literal["male", "female", "others"], Field(
        description="Gender of the patient"
    )]
    height: Annotated[float, Field(
        gt=0,
        description="Height of the patient in meters"
    )]
    weight: Annotated[float, Field(
        gt=0,
        description="Weight of the patient in kilograms"
    )]

    # Computed fields
    @computed_field(return_type=float)
    @property
    def bmi(self) -> float:
        return round(self.weight / (self.height ** 2), 2)

    @computed_field(return_type=str)
    @property
    def verdict(self) -> str:
        if self.bmi < 18.5:
            return "Underweight"
        elif self.bmi < 25:
            return "Normal"
        elif self.bmi < 30:
            return "Overweight"
        else:
            return "Obese"
```

✅ Features:

* Validation:

  * Age between 1–119.
  * Height/Weight > 0.
  * Gender restricted to `male | female | others`.
* Computed fields:

  * **BMI** (Body Mass Index).
  * **Verdict** (Underweight, Normal, Overweight, Obese).

---

## 5. Writing Utility Functions

We need helper functions to **load and save JSON data**.

```python
import json

DB_FILE = "patients.json"

def load_data() -> dict:
    with open(DB_FILE, "r") as f:
        return json.load(f)

def save_data(data: dict):
    with open(DB_FILE, "w") as f:
        json.dump(data, f, indent=4)
```

---

## 6. Implementing the `Create` Endpoint

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import JSONResponse

app = FastAPI()

@app.post("/create")
def create_patient(patient: Patient):
    # Step 1: Load existing data
    data = load_data()

    # Step 2: Check if ID already exists
    if patient.id in data:
        raise HTTPException(status_code=400, detail="Patient already exists")

    # Step 3: Add new patient
    data[patient.id] = patient.model_dump(exclude={"id"})  

    # Step 4: Save updated data
    save_data(data)

    # Step 5: Return success response
    return JSONResponse(
        status_code=201,
        content={"message": "Patient created successfully"}
    )
```

---

## 7. Testing the Endpoint

1. Run the server:

   ```bash
   uvicorn main:app --reload
   ```

2. Open **Swagger Docs** (FastAPI auto-generates):

   ```
   http://127.0.0.1:8000/docs
   ```

3. In docs:

   * `POST /create` appears in **green** (POST).
   * Test with:

     ```json
     {
       "id": "P006",
       "name": "Rahul Gupta",
       "city": "Hyderabad",
       "age": 19,
       "gender": "male",
       "height": 1.74,
       "weight": 55
     }
     ```

4. Possible outcomes:

   * If ID already exists →
     Response: `400 Bad Request`, message: `"Patient already exists"`.
   * If ID is new →
     Response: `201 Created`, message: `"Patient created successfully"`.
     JSON file updated with **BMI & Verdict auto-computed**.

---

## 8. Key Learnings

* **POST vs GET**:

  * GET = retrieve data.
  * POST = send new data (create).
* **Request Body**:

  * JSON sent by client → validated by Pydantic.
* **Pydantic**:

  * Ensures **valid data** before inserting into DB.
  * Supports **computed fields** (BMI & Verdict).
* **FastAPI + Pydantic synergy**:

  * FastAPI automatically:

    * Parses request body.
    * Sends it to Pydantic model.
    * Returns validation errors if any.

---

## 9. Next Steps

* **Upcoming lectures**:

  * Implement `Update` (PUT) endpoint.
  * Implement `Delete` endpoint.

---

✨ With this, CRUD → **C + R implemented**, next: **U + D**.
