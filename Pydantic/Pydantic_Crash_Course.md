# 📘 Lecture Notes: Pydantic (Beginner → Intermediate)

---

## 1. Why Pydantic?

Python is a **dynamically typed language**:

* You can assign different types to the same variable at runtime.
* Example:

```python
x = 10      # integer
x = "ten"   # string (allowed in Python)
```

✅ Good for beginners, but ❌ dangerous in production.

* No strict type validation.
* Leads to **bugs** (wrong types stored in DB, APIs receiving bad data, etc.).

### Two big problems in Python:

1. **No strict type validation** (types can be ignored).
2. **No built-in data validation** (e.g., email format, age > 0).

👉 **Pydantic** solves both.

* Builds **data models (schemas)**.
* Handles **type validation + data validation**.
* Widely used in **FastAPI**, **YAML/JSON configs**, **ML pipelines**.

---

## 2. Traditional Approaches (before Pydantic)

### Example function (Insert patient data)

```python
def insert_patient_data(name, age):
    print(name, age)
    print("Inserted into database")
```

Problems:

* A junior dev might pass wrong types:

  ```python
  insert_patient_data("Nitesh", "30")  # age should be int!
  ```
* Still runs → wrong data goes to DB.

---

### Attempt #1: Type Hints

```python
def insert_patient_data(name: str, age: int):
    print(name, age)
```

* Signature suggests correct types.
* ❌ But Python ignores hints at runtime.
* `"30"` (string) still works → not enforced.

---

### Attempt #2: Manual Validation

```python
def insert_patient_data(name, age):
    if not isinstance(name, str) or not isinstance(age, int):
        raise TypeError("Invalid data type")
    if age < 0:
        raise ValueError("Age cannot be negative")
    print("Inserted into DB")
```

* ✅ Works for small cases.
* ❌ Not scalable (repeated in every function).
* ❌ Boilerplate code everywhere.

---

## 3. Enter Pydantic 🎉

### How it works (3 Steps)

1. **Define a model** (schema) → using `BaseModel`.
2. **Create an instance** → raw dict → validated object.
3. **Use object** → pass it to functions, guaranteed validated.

---

## 4. First Pydantic Example

### Install Pydantic V2

```bash
pip install pydantic
```

---

### Step 1: Define Model

```python
from pydantic import BaseModel

class Patient(BaseModel):
    name: str
    age: int
```

---

### Step 2: Create Instance

```python
patient_info = {"name": "Nitesh", "age": 30}
patient1 = Patient(**patient_info)
```

* If `"age": "30"` → auto converts to int.
* If `"age": "thirty"` → ❌ ValidationError.

---

### Step 3: Use in Function

```python
def insert_patient_data(patient: Patient):
    print(patient.name, patient.age)
    print("Inserted into DB")

insert_patient_data(patient1)
```

* ✅ Clean, reusable.
* ✅ If you add new fields (e.g., weight), just update `Patient` model.

---

## 5. Bigger Model with Complex Types

### Supported Types

* **Primitives**: `str`, `int`, `float`, `bool`
* **Lists**: `List[str]`, `List[int]`
* **Dictionaries**: `Dict[str, str]`
* **Optional fields**

---

### Example

```python
from typing import List, Dict, Optional
from pydantic import BaseModel

class Patient(BaseModel):
    name: str
    age: int
    weight: float
    married: bool = False
    allergies: Optional[List[str]] = None
    contact_details: Dict[str, str]
```

---

### Input Example

```python
patient_info = {
    "name": "Nitesh",
    "age": 30,
    "weight": 72.5,
    "married": True,
    "allergies": ["dust", "pollen"],
    "contact_details": {
        "email": "abc@gmail.com",
        "phone": "1234567890"
    }
}

patient = Patient(**patient_info)
```

* ✅ Wrong type (e.g., int in allergies list) → ValidationError.

---

## 6. Required vs Optional Fields

* By default, all fields are **required**.
* Use `Optional[]` + `= None` for optional fields.

```python
from typing import Optional

class Patient(BaseModel):
    name: str
    age: int
    allergies: Optional[List[str]] = None
```

---

## 7. Default Values

```python
class Patient(BaseModel):
    married: bool = False
```

* If not provided → defaults to `False`.

---

## 8. Built-in Data Validation (Custom Types)

### Example: Email & URL

```python
from pydantic import BaseModel, EmailStr, AnyUrl

class Patient(BaseModel):
    name: str
    email: EmailStr
    linkedin_url: AnyUrl
```

✅ Auto validates:

* `"abc@gmail.com"` ✅
* `"abcgmail.com"` ❌

---

## 9. Field Constraints with `Field`

### Example

```python
from pydantic import BaseModel, Field

class Patient(BaseModel):
    name: str = Field(..., max_length=50)
    age: int = Field(..., gt=0, lt=120)  # range 1–119
    weight: float = Field(..., gt=0, strict=True)
    allergies: Optional[List[str]] = Field(None, max_length=5)
```

* `gt`, `lt` → numerical ranges.
* `max_length` → string/list size.
* `strict=True` → disallow auto type conversion.

---

## 10. Metadata with `Annotated`

```python
from typing import Annotated
from pydantic import BaseModel, Field

class Patient(BaseModel):
    name: Annotated[
        str, Field(max_length=50, title="Patient Name",
                   description="Full name under 50 characters",
                   examples=["Nitesh", "Amit"])
    ]
```

* Useful in **FastAPI docs**.

---

## 11. Field Validators

### Custom Validation

```python
from pydantic import BaseModel, EmailStr, field_validator

class Patient(BaseModel):
    email: EmailStr

    @field_validator("email")
    @classmethod
    def validate_email_domain(cls, value):
        valid_domains = ["hdfc.com", "icici.com"]
        domain = value.split("@")[-1]
        if domain not in valid_domains:
            raise ValueError("Not a valid domain")
        return value
```

✅ Ensures only corporate emails allowed.

---

### Transformation Example

```python
class Patient(BaseModel):
    name: str

    @field_validator("name")
    @classmethod
    def transform_name(cls, value):
        return value.upper()
```

* `"nitesh"` → `"NITESH"`.

---

### Modes: `before` vs `after`

* **before** → runs before type conversion.
* **after** (default) → runs after type conversion.

---

## 12. Model Validators (Cross-field Validation)

```python
from pydantic import BaseModel, model_validator
from typing import Dict

class Patient(BaseModel):
    age: int
    contact_details: Dict[str, str]

    @model_validator(mode="after")
    def validate_emergency_contact(self):
        if self.age > 60 and "emergency" not in self.contact_details:
            raise ValueError("Patients over 60 must have emergency contact")
        return self
```

✅ Allows rules involving **multiple fields**.

---

## 13. Computed Fields

```python
from pydantic import BaseModel, computed_field

class Patient(BaseModel):
    weight: float
    height: float  # meters

    @computed_field
    @property
    def bmi(self) -> float:
        return round(self.weight / (self.height ** 2), 2)
```

* ✅ BMI auto-calculated.
* User doesn’t provide it.

---

## 14. Nested Models

```python
class Address(BaseModel):
    city: str
    state: str
    pincode: str

class Patient(BaseModel):
    name: str
    age: int
    address: Address
```

Input:

```python
patient_info = {
    "name": "Nitesh",
    "age": 30,
    "address": {
        "city": "Gurgaon",
        "state": "Haryana",
        "pincode": "122001"
    }
}
patient = Patient(**patient_info)
```

✅ Auto validates nested models.
✅ Cleaner + reusable (`Address` can be reused in `Employee`, `Student`, etc.).

---

## 15. Exporting Models

### To Dict

```python
data = patient.model_dump()
```

### To JSON

```python
json_data = patient.model_dump_json()
```

---

### Control with `include` / `exclude`

```python
# Include only some fields
patient.model_dump(include={"name", "age"})

# Exclude specific fields
patient.model_dump(exclude={"address"})
```

---

### Exclude Unset Fields

```python
class Patient(BaseModel):
    gender: str = "Male"

patient = Patient(name="Nitesh", age=30)

patient.model_dump(exclude_unset=True)  # "gender" omitted
```

---

# ✅ Summary

Pydantic provides:

1. **Strict type validation**.
2. **Flexible data validation** (`Field`, `EmailStr`, custom validators).
3. **Transformations** (`field_validator`, `model_validator`).
4. **Computed fields** (e.g., BMI).
5. **Nested models** for structured data.
6. **Export utilities** (`model_dump`, `model_dump_json`).

👉 With this, you can write **clean, reusable, and production-grade Python code**.
👉 Essential for **FastAPI**, configs, and ML pipelines.
