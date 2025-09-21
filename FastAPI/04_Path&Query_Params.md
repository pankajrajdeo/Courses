# Lecture Notes: **Path Parameters & Query Parameters in FastAPI**

---

## 1. Introduction

* Continuing the FastAPI project from the previous lecture.
* Goal of this lecture:

  1. Extend the project functionality.
  2. Learn two important concepts: **Path Parameters** and **Query Parameters**.

---

## 2. Path Parameters

### 2.1 Definition

* **Path parameters** are **dynamic segments** of a URL path used to identify a **specific resource** on the server.
* They make URLs dynamic and flexible.

Example:

```
http://localhost:8000/view
```

→ Shows all patients.

But if we want a **specific patient**:

```
http://localhost:8000/view/3
```

* Here, `3` is a **path parameter** → a dynamic portion of the URL.
* It identifies a **particular patient resource**.

---

### 2.2 Use Cases

* **Retrieve** a specific resource (e.g., patient profile).
* **Update** a specific resource.
* **Delete** a specific resource.

---

### 2.3 Implementing Path Parameters

**Step 1:** Define a new route with a dynamic segment:

```python
from fastapi import FastAPI
from utils import load_data  # utility function to load patients data

app = FastAPI()

@app.get("/patient/{patient_id}")
def view_patient(patient_id: str):
    data = load_data()  # load all patients data
    if patient_id in data:
        return data[patient_id]
    return {"error": "Patient not found"}
```

* `patient_id` is declared as a variable inside `{}`.
* Type is `str` because patient IDs look like `P01`, `P002`, etc.

---

### 2.4 Improving Readability with `Path()`

FastAPI provides a helper:

```python
from fastapi import Path

@app.get("/patient/{patient_id}")
def view_patient(
    patient_id: str = Path(..., description="ID of the patient in the DB", example="P01")
):
    data = load_data()
    if patient_id in data:
        return data[patient_id]
    return {"error": "Patient not found"}
```

* `...` → required parameter.
* Adds **metadata** (description, examples) for API docs.

---

## 3. HTTP Status Codes

### 3.1 Definition

* **HTTP Status Codes** are 3-digit numbers returned by the server along with the response.
* They describe the **result of the client’s request**.

### 3.2 Categories

* **2xx** → Success (e.g., 200 OK, 201 Created, 204 No Content).
* **3xx** → Further action required (redirection).
* **4xx** → Client-side error (e.g., 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found).
* **5xx** → Server-side error (e.g., 500 Internal Server Error, 502 Bad Gateway).

---

### 3.3 Problem in Current Code

* If patient doesn’t exist → returns `{error: "Patient not found"}` with **200 OK**.
* But correct code should be **404 Not Found**.

---

### 3.4 Fixing with `HTTPException`

```python
from fastapi import HTTPException

@app.get("/patient/{patient_id}")
def view_patient(patient_id: str):
    data = load_data()
    if patient_id in data:
        return data[patient_id]
    raise HTTPException(status_code=404, detail="Patient not found")
```

* Raises proper error response:

  ```json
  {
    "detail": "Patient not found"
  }
  ```
* Status code: `404`.

---

## 4. Query Parameters

### 4.1 Definition

* **Query parameters** are optional **key-value pairs** appended to the URL **after `?`**.
* They pass **additional information** to the server without altering the endpoint path.
* Commonly used for:

  * Filtering
  * Sorting
  * Searching
  * Pagination

Example:

```
http://localhost:8000/sort?sort_by=weight&order=descending
```

Here:

* `sort_by` = `weight`
* `order` = `descending`

---

### 4.2 Syntax

* After `?` → `key=value`.
* Multiple parameters → separated by `&`.

Example:

```
/endpoint?city=Delhi&sort_by=age
```

---

### 4.3 Implementing Query Parameters

**Step 1:** Create endpoint with query params.

```python
from fastapi import Query

@app.get("/sort")
def sort_patients(
    sort_by: str = Query(..., description="Sort by: height, weight, or bmi"),
    order: str = Query("ascending", description="Sort order: ascending or descending")
):
    valid_fields = ["height", "weight", "bmi"]
    if sort_by not in valid_fields:
        raise HTTPException(status_code=400, detail=f"Invalid field. Select from {valid_fields}")
    
    if order not in ["ascending", "descending"]:
        raise HTTPException(status_code=400, detail="Invalid order. Select between ascending or descending")
    
    data = load_data()
    
    # Sort logic
    sort_order = True if order == "descending" else False
    sorted_data = sorted(data.values(), key=lambda x: x[sort_by], reverse=sort_order)
    
    return sorted_data
```

---

### 4.4 Behavior

* `/sort?sort_by=height&order=descending`
  → Patients sorted by height, tallest first.

* `/sort?sort_by=bmi`
  → Sorted by BMI, default **ascending**.

* Invalid query (e.g., `/sort?sort_by=age`)
  → Returns 400 Bad Request.

---

## 5. Summary

* **Path Parameters**

  * Required.
  * Part of the URL path (e.g., `/patient/{id}`).
  * Identify specific resources.
  * Useful for **retrieve, update, delete** operations.

* **Query Parameters**

  * Optional.
  * Appended after `?` in URL (e.g., `/sort?sort_by=weight&order=desc`).
  * Useful for **search, filter, sort, pagination**.

* **Improvements Learned**

  1. Enhanced **API documentation** with `Path()` and `Query()`.
  2. Proper **error handling** with `HTTPException` + correct status codes.

* **Next step:**
  Combine **Path** + **Query parameters** in a single endpoint.
