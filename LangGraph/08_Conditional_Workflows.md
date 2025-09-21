# Lecture Notes: Conditional Workflows in LangGraph

## 1. Recap of Previous Workflows

Before learning conditional workflows, the instructor reviewed what was already covered in the LangGraph playlist:

* **Sequential Workflows**:
  Tasks execute **one after another in a linear order**.
  Example: Task1 → Task2 → Task3.

* **Parallel Workflows**:
  Multiple tasks execute **simultaneously** after a branching point.
  Example flow:

  ```
  Task1 
    ↘ Task2 
    ↘ Task3  
  (both execute in parallel) → Task4
  ```

---

## 2. Introduction to Conditional Workflows

### Definition

* Similar to parallel workflows but **only one branch executes**, based on a **condition**.
* It’s equivalent to **if–else logic in programming**.

### Example:

```
Task1 → (if condition true → Task2 → Task4)
       (else → Task3 → Task4)
```

* At the decision point, **only one branch is taken**.
* More than two branches are possible (like `if–elif–else`).

### Importance

* Just as `if–else` is fundamental in programming, **conditional workflows are essential in LangGraph**.
* They are often required for building real-world, complex workflows.

---

## 3. First Example: Quadratic Equation Solver Workflow

### Why Quadratic Equations?

* They naturally involve **conditions** based on the discriminant (`b² - 4ac`).
* Good demo of branching logic.

### Quadratic Equation Refresher

* Standard form: **ax² + bx + c = 0**
* **Discriminant**: `D = b² - 4ac`
* Root cases:

  * If `D > 0`: Two distinct real roots

    ```
    root1 = (-b + √D) / 2a
    root2 = (-b - √D) / 2a
    ```
  * If `D = 0`: One repeated root

    ```
    root = -b / 2a
    ```
  * If `D < 0`: No real roots.

### Workflow Plan

1. Input: coefficients `a`, `b`, `c`.
2. Node 1: Show equation.
3. Node 2: Calculate discriminant.
4. Conditional branch:

   * `D > 0` → Real roots node.
   * `D = 0` → Repeated root node.
   * `D < 0` → No real roots node.
5. End.

### State Definition (Code)

```python
class QuadState(TypedDict):
    a: float
    b: float
    c: float
    equation: str
    discriminant: float
    result: str
```

### Step 1: Show Equation Node

```python
def show_equation(state: QuadState):
    equation = f"{state['a']}x² + {state['b']}x + {state['c']}"
    return {"equation": equation}
```

### Step 2: Calculate Discriminant Node

```python
def calculate_discriminant(state: QuadState):
    d = (state['b'] ** 2) - (4 * state['a'] * state['c'])
    return {"discriminant": d}
```

### Step 3: Conditional Branch Functions

* **Real Roots**

```python
def real_roots(state: QuadState):
    d = state["discriminant"] ** 0.5
    root1 = (-state["b"] + d) / (2 * state["a"])
    root2 = (-state["b"] - d) / (2 * state["a"])
    return {"result": f"The roots are {root1} and {root2}"}
```

* **Repeated Root**

```python
def repeated_root(state: QuadState):
    root = -state["b"] / (2 * state["a"])
    return {"result": f"The repeated root is {root}"}
```

* **No Real Roots**

```python
def no_real_roots(state: QuadState):
    return {"result": "No real roots"}
```

### Step 4: Routing Function

```python
def check_condition(state: QuadState):
    d = state["discriminant"]
    if d > 0:
        return "real_roots"
    elif d == 0:
        return "repeated_root"
    else:
        return "no_real_roots"
```

### Connecting Graph with Conditional Edges

```python
graph.add_conditional_edges(
    "calculate_discriminant",
    check_condition
)

graph.add_edge("real_roots", "end")
graph.add_edge("repeated_root", "end")
graph.add_edge("no_real_roots", "end")
```

### Example Run

Input: `a=4, b=-5, c=-4`

* Equation: `4x² - 5x - 4`
* Discriminant: 89
* Result: Two real roots.

---

## 4. Second Example: LLM-Based Customer Support Workflow

### Goal

* Given a **customer review**, decide if it’s **positive** or **negative**.
* If positive → generate a thank-you reply.
* If negative → diagnose issue (type, tone, urgency) and generate an empathetic resolution.

### Workflow Plan

1. Input: `review`.
2. Node 1: Find sentiment (positive/negative).
3. Conditional branch:

   * If positive → Positive response node.
   * If negative → Run diagnosis → Negative response.
4. End.

### State Definition

```python
class ReviewState(TypedDict):
    review: str
    sentiment: Literal["positive", "negative"]
    diagnosis: dict
    response: str
```

### Step 1: Find Sentiment

Schema:

```python
class SentimentSchema(BaseModel):
    sentiment: Literal["positive", "negative"]
```

Function:

```python
def find_sentiment(state: ReviewState):
    prompt = f"What is the sentiment of this review: {state['review']}"
    result = structured_model.invoke(prompt)  # with SentimentSchema
    return {"sentiment": result.sentiment}
```

### Step 2: Conditional Branching

```python
def check_sentiment(state: ReviewState):
    if state["sentiment"] == "positive":
        return "positive_response"
    else:
        return "run_diagnosis"
```

### Step 3: Positive Response

```python
def positive_response(state: ReviewState):
    prompt = f"Write a warm thank you message in response to this review: {state['review']}"
    response = model.invoke(prompt).content
    return {"response": response}
```

### Step 4: Run Diagnosis for Negative Reviews

Schema:

```python
class DiagnosisSchema(BaseModel):
    issue_type: Literal["UI", "performance", "bug", "support", "other"]
    tone: Literal["frustrated", "angry", "neutral"]
    urgency: Literal["low", "medium", "high"]
```

Function:

```python
def run_diagnosis(state: ReviewState):
    prompt = f"Diagnose this review: {state['review']}"
    result = structured_model2.invoke(prompt)  # with DiagnosisSchema
    return {"diagnosis": result.model_dump()}
```

### Step 5: Negative Response

```python
def negative_response(state: ReviewState):
    diag = state["diagnosis"]
    prompt = f"""
    You are a support assistant.
    The user reported an issue of type {diag['issue_type']}.
    The tone is {diag['tone']} and urgency is {diag['urgency']}.
    Write an empathetic, helpful resolution message.
    """
    response = model.invoke(prompt).content
    return {"response": response}
```

### Conditional Edges

```python
graph.add_conditional_edges("find_sentiment", check_sentiment)
graph.add_edge("positive_response", "end")
graph.add_edge("run_diagnosis", "negative_response")
graph.add_edge("negative_response", "end")
```

### Example Run

* **Positive review**:
  “The app interface is incredibly clean and intuitive.”
  → Sentiment = positive → Generates thank-you message.

* **Negative review**:
  “The app keeps freezing on login, unacceptable for basic functionality.”
  → Sentiment = negative → Diagnosed as:

  * Issue type: bug
  * Tone: frustrated
  * Urgency: high
    → Generates empathetic, urgent response.

---

## 5. Key Takeaways

* **Conditional workflows** mirror `if–else` logic in programming.
* Useful whenever you must make decisions in workflows.
* Two examples:

  1. Non-LLM example (Quadratic Equation solver).
  2. LLM example (Customer Support system).
* Core concept:

  * Build **routing functions** that check conditions.
  * Use **`add_conditional_edges`** in LangGraph.

---

✅ After revising these notes, you should be able to:

* Differentiate sequential, parallel, and conditional workflows.
* Implement conditional branching in LangGraph.
* Apply both **mathematical** and **LLM-driven** logic inside workflows.
