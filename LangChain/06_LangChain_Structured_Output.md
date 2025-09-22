# Lecture Notes: Structured Output in LangChain

## 1. Introduction

* **Goal**: Learn how to make Large Language Models (LLMs) interact with other systems (databases, APIs, tools).
* Normally, LLMs generate **unstructured text** outputs.
* **Structured output** allows us to transform these outputs into formats like JSON, making them useful for integration with external systems.
* This concept is foundational for **agents**—enhanced chatbots that can not only talk but also **perform actions**.

---

## 2. Recap of Previous Lecture

* Last time: **Prompts** in LangChain.

  * Prompts = inputs given to LLMs.
* Today: Focus on **outputs** from LLMs.

  * Specifically: how to **structure** outputs for programmatic use.

---

## 3. What is Structured Output?

### Unstructured Output

* Normal conversation with ChatGPT:

  ```txt
  Q: What is the capital of India?
  A: New Delhi is the capital of India.
  ```
* Output = plain text → **unstructured**.

### Structured Output

* Instead of plain text, output is returned in **JSON/dictionary form**:

  ```json
  [
    {"time": "morning", "activity": "Visit Eiffel Tower"},
    {"time": "afternoon", "activity": "Visit a museum"},
    {"time": "evening", "activity": "Dinner"}
  ]
  ```
* **Advantages**:

  * Easier to parse.
  * Can be stored in databases or passed to APIs.
  * Enables LLM ↔ machine/system communication.

**Definition**:
In LangChain, **structured output** means making the LLM return responses in a **well-defined data format** (like JSON), instead of free text.

---

## 4. Why Do We Need Structured Output?

### Use Case 1: Data Extraction

* Example: Job portal like Naukri.com.
* Extract structured information from resumes (name, last company, grades).
* Save directly into a database using structured output.

### Use Case 2: API Building

* Example: Summarizing Amazon product reviews.
* Extract topics, pros, cons, sentiment.
* Serve structured info via a REST API (Flask, FastAPI).

### Use Case 3: Building Agents

* Agents need **tools** (e.g., calculator).
* Structured output helps extract **numbers, operations, parameters** from text.
* Example: “Find square root of 2” → extract `{ "operation": "sqrt", "number": 2 }`.

---

## 5. Ways to Get Structured Output in LangChain

There are two types of LLMs:

1. **LLMs that natively support structured output** (e.g., OpenAI GPT models).

   * Use `with_structured_output()`.
2. **LLMs that do not support structured output**.

   * Use **Output Parsers** (covered in the next lecture).

### Today’s Focus: `with_structured_output`

---

## 6. The `with_structured_output` Function

* Workflow is same as normal LLM call, but with one change:

  * Call `.with_structured_output(schema)` before invoking the model.
* You must define a **schema** to specify output format.

**3 ways to define schema**:

1. `TypedDict` (Python type hints).
2. `Pydantic` (data validation).
3. `JSON Schema` (cross-language).

---

## 7. Method 1: Using TypedDict

### What is TypedDict?

* A way to define dictionary structure in Python.

* Example:

  ```python
  from typing import TypedDict

  class Person(TypedDict):
      name: str
      age: int
  ```

* Benefits:

  * IDE/type checker knows expected keys & types.

* Limitations:

  * **No runtime validation** (Python won’t stop you if you put wrong type).

### Example: Phone Review Summarizer

```python
from langchain_openai import ChatOpenAI
from typing import TypedDict
from dotenv import load_dotenv

load_dotenv()
model = ChatOpenAI()

# Define schema
class Review(TypedDict):
    summary: str
    sentiment: str

# Create structured model
structured_model = model.with_structured_output(Review)

# Input
review_text = "The phone has great battery life but the display is dull."

result = structured_model.invoke(review_text)

print(result)
print("Summary:", result["summary"])
print("Sentiment:", result["sentiment"])
```

* Output:

  ```json
  {"summary": "Great battery life, poor display", "sentiment": "neutral"}
  ```

---

## 8. Method 2: Using Pydantic

### What is Pydantic?

* Python library for **data validation & parsing**.
* Used in FastAPI.
* Provides runtime **validation**, **default values**, **type coercion**.

### Key Features

* **Validation**:

  ```python
  from pydantic import BaseModel

  class Student(BaseModel):
      name: str

  student = Student(name=32)  # ❌ Error: must be string
  ```
* **Optional fields**:

  ```python
  from typing import Optional

  class Student(BaseModel):
      name: str
      age: Optional[int] = None
  ```
* **Constraints** with `Field`:

  ```python
  from pydantic import BaseModel, Field

  class Student(BaseModel):
      cgpa: float = Field(ge=0, le=10, description="Student CGPA")
  ```

### Example: Review Schema with Pydantic

```python
from pydantic import BaseModel, Field
from typing import Optional, List, Literal
from langchain_openai import ChatOpenAI

class Review(BaseModel):
    key_themes: List[str] = Field(description="Topics discussed in the review")
    summary: str = Field(description="Short summary of the review")
    sentiment: Literal["positive", "negative"] = Field(description="Overall sentiment")
    pros: Optional[List[str]] = Field(default=None, description="Pros if available")
    cons: Optional[List[str]] = Field(default=None, description="Cons if available")
    name: Optional[str] = Field(default=None, description="Reviewer name")

model = ChatOpenAI().with_structured_output(Review)

result = model.invoke("The battery lasts long, but the screen is dim.")
print(result.dict())
```

---

## 9. Method 3: Using JSON Schema

### When to Use JSON Schema

* Needed when your project uses **multiple languages** (e.g., Python backend, JavaScript frontend).
* JSON Schema is **language-agnostic**.

### Example Schema

```json
{
  "title": "Review",
  "type": "object",
  "properties": {
    "summary": { "type": "string" },
    "sentiment": { "type": "string", "enum": ["positive", "negative"] },
    "pros": { "type": ["array", "null"], "items": { "type": "string" } },
    "cons": { "type": ["array", "null"], "items": { "type": "string" } }
  },
  "required": ["summary", "sentiment"]
}
```

### Usage in LangChain

```python
json_schema = {...}  # paste schema here
model = ChatOpenAI().with_structured_output(json_schema)
result = model.invoke("The battery is great but the phone heats up.")
print(result)
```

---

## 10. Choosing the Right Approach

* **TypedDict** → Simple type hints, no validation. Use if you just need structure hints.
* **Pydantic** → Python-only, with full validation, defaults, constraints.
  → **Recommended** for most projects.
* **JSON Schema** → Cross-language compatibility. Use if multiple languages must share schema.

---

## 11. Important Points

* `with_structured_output` has a parameter `method`:

  * `"json_mode"` → outputs JSON (most common).
  * `"function_calling"` → used for tool/agent execution.
* Some models (like OpenAI GPT) support structured output natively.
* Others (e.g., some HuggingFace models) do not → must use **Output Parsers** (next lecture).

---

## 12. Summary

* Structured output allows LLMs to **communicate with machines** by producing well-defined data formats.
* **3 schema definition methods**: TypedDict, Pydantic, JSON Schema.
* Best practice:

  * Use **Pydantic** for validation-heavy Python projects.
  * Use **JSON Schema** for cross-language scenarios.

---

# 📊 Comparison: TypedDict vs Pydantic vs JSON Schema

| Feature / Aspect              | **TypedDict** (Python)      | **Pydantic** (Python)                             | **JSON Schema** (Language-Agnostic)          |
| ----------------------------- | --------------------------- | ------------------------------------------------- | -------------------------------------------- |
| **Language Scope**            | Python only                 | Python only                                       | Cross-language (universal JSON)              |
| **Validation**                | ❌ No runtime validation     | ✅ Strong runtime validation                       | ✅ Validation via schema rules                |
| **Default Values**            | ❌ Not supported             | ✅ Supported via `Field(default=...)`              | ✅ Can define defaults in schema              |
| **Type Coercion**             | ❌ No                        | ✅ Automatic (e.g., "32" → int 32)                 | ❌ No automatic coercion                      |
| **Constraints** (e.g., range) | ❌ Not supported             | ✅ `Field(ge=0, le=10)`                            | ✅ Supported via schema keywords              |
| **Optional Fields**           | ✅ Supported (`Optional`)    | ✅ Supported (`Optional`)                          | ✅ Supported (`type: [X, null]`)              |
| **Annotations / Hints**       | ✅ With `Annotated`          | ✅ With `Field(description=...)`                   | ✅ With `description`                         |
| **Ease of Use**               | ✅ Simple (type hints)       | ⚖️ Moderate (needs Pydantic library)              | ⚖️ Verbose (JSON syntax heavy)               |
| **Best For**                  | Quick prototyping in Python | Production-grade Python apps (FastAPI, LangChain) | Cross-language projects (backend + frontend) |

---

## ✅ Recommendations

* **Use `TypedDict`**:

  * When you only need **basic type hints**.
  * For **quick experiments**.
  * No validation required.

* **Use `Pydantic`**:

  * When building **production-ready systems** in Python.
  * If you need **validation, defaults, constraints, type coercion**.
  * Best choice for most LangChain + Python projects.

* **Use `JSON Schema`**:

  * When working with **multiple languages** (Python + JavaScript/Java/etc.).
  * When schemas must be **shared across systems**.
  * Suitable for APIs and interoperability.

---

⚡ This table gives you a **cheat sheet** when deciding which approach to use.

