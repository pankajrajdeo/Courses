# 📘 Lecture Notes: FastAPI – Patient Management System (CRUD Complete)

## 1. Project Overview

We are building a **Patient Management System API** using **FastAPI**.
The project implements the **CRUD operations**:

* **Create** → Add a new patient ✅ (already built)
* **Retrieve** → Fetch patient details ✅ (already built: all, by ID, sorted by weight/height/BMI)
* **Update** → Edit details of an existing patient 🔄 (built in this lecture)
* **Delete** → Remove a patient from the system ❌ (built in this lecture)

At the end of this lecture, **CRUD is fully implemented**.

---

## 2. Update Endpoint (Edit Patient Data)

### 2.1 Requirements

* **Endpoint name:** `/edit/{patient_id}`
* **HTTP Method:** `PUT`
* **Inputs:**

  1. **Path parameter:** `patient_id` → to identify which patient to update.
  2. **Request body (JSON):** with new field values to update (city, weight, etc.).
* **Key Point:**

  * Client may send **all fields** OR **only a few fields**.
  * Must work in both cases.

---

### 2.2 Why a New Pydantic Model?

* Existing model `Patient` requires **all fields**.
* But for update, only some fields may be sent.
* ✅ Solution → Create a **new Pydantic model `PatientUpdate`**:

  * All fields are optional.
  * Defaults set to `None`.

```python
from pydantic import BaseModel
from typing import Optional

class PatientUpdate(BaseModel):
    name: Optional[str] = None
    city: Optional[str] = None
    age: Optional[int] = None
    gender: Optional[str] = None
    height: Optional[float] = None
    weight: Optional[float] = None
```

* Fields `bmi` and `verdict` are **excluded** since they are computed automatically.

---

### 2.3 Update Endpoint Implementation

#### Step 1: Define the endpoint

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.put("/edit/{patient_id}")
def update_patient(patient_id: str, patient_update: PatientUpdate):
    # Step 1: Load existing data
    data = load_data()   # custom utility function
```

---

#### Step 2: Validate patient exists

```python
    if patient_id not in data:
        raise HTTPException(status_code=404, detail="Patient not found")
```

---

#### Step 3: Extract existing patient info

```python
    existing_patient = data[patient_id]
```

---

#### Step 4: Convert `patient_update` to dictionary

* Only include fields client provided (`exclude_unset=True`).

```python
    updated_fields = patient_update.model_dump(exclude_unset=True)
```

---

#### Step 5: Merge updates into existing patient info

```python
    for key, value in updated_fields.items():
        existing_patient[key] = value
```

---

#### Step 6: Handle dependent fields (BMI & Verdict)

* After updating **weight/height**, **BMI & Verdict must be recalculated**.
* Convert dict → `Patient` object (original full model).
* Recalculate automatically through model logic.

```python
    # Add back ID (required by Patient model)
    existing_patient["id"] = patient_id  

    # Recreate Patient object → auto compute BMI & verdict
    patient_obj = Patient(**existing_patient)  

    # Convert back to dict, exclude ID (already stored as key)
    data[patient_id] = patient_obj.model_dump(exclude={"id"})
```

---

#### Step 7: Save updated data and return response

```python
    save_data(data)
    return {"message": "Patient updated"}
```

---

### 2.4 Example Request & Response

**Request:**

```
PUT /edit/P004
{
  "city": "Mumbai",
  "weight": 90
}
```

**Before:**

```json
{
  "name": "Arjun Verma",
  "city": "Bengaluru",
  "age": 40,
  "gender": "Male",
  "height": 1.8,
  "weight": 95,
  "bmi": 29.32,
  "verdict": "Overweight"
}
```

**After:**

```json
{
  "name": "Arjun Verma",
  "city": "Mumbai",
  "age": 40,
  "gender": "Male",
  "height": 1.8,
  "weight": 90,
  "bmi": 27.78,
  "verdict": "Normal"
}
```

---

### 2.5 Edge Cases Tested

1. ✅ Update multiple fields (city + weight) → Works, BMI recalculated.
2. ✅ Update only one field (name) → Works.
3. ❌ Update with wrong `patient_id` → Returns `404 Patient not found`.

---

## 3. Delete Endpoint

### 3.1 Requirements

* **Endpoint name:** `/delete/{patient_id}`
* **HTTP Method:** `DELETE`
* **Inputs:**

  * `patient_id` (path parameter).
* **Action:** Remove patient record from data store.

---

### 3.2 Implementation

```python
@app.delete("/delete/{patient_id}")
def delete_patient(patient_id: str):
    data = load_data()

    if patient_id not in data:
        raise HTTPException(status_code=404, detail="Patient not found")

    del data[patient_id]    # remove patient
    save_data(data)

    return {"message": "Patient deleted"}
```

---

### 3.3 Example Request & Response

**Request:**

```
DELETE /delete/P006
```

**Before:** Last patient → `P006` exists
**After:** Last patient → now `P005` is last
**Response:**

```json
{ "message": "Patient deleted" }
```

---

### 3.4 Edge Cases Tested

1. ✅ Valid patient\_id → Record deleted.
2. ❌ Invalid patient\_id → `404 Patient not found`.

---

## 4. Final System Features (Summary)

Our **Patient Management API** now supports:

1. **Retrieve all patients** → `GET /patients`
2. **Retrieve by ID** → `GET /patient/{id}`
3. **Retrieve sorted list** → `GET /sort?by=weight|height|bmi&order=asc|desc`
4. **Create new patient** → `POST /create`
5. **Update patient info** → `PUT /edit/{id}`
6. **Delete patient** → `DELETE /delete/{id}`

✅ This completes the **CRUD functionality**.

---

## 5. Next Steps

* In future lectures → use FastAPI to **serve Machine Learning models**.
* Today’s work ensures fundamentals of **CRUD & API structure** are clear.

---

# 📌 Key Takeaways

* **CRUD completed** with FastAPI.
* Learned difference between `POST`, `GET`, `PUT`, `DELETE`.
* Understood why **separate Pydantic models** are useful for update vs create.
* Learned how to recalculate **dependent fields (BMI & verdict)** during updates.
* Proper **error handling (404)** included.

Would you like me to also prepare a **diagram/flowchart of the CRUD API endpoints** to visually revise how each part connects?
